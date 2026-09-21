# PD-020 — Channel Source Attribution & Cross-Channel UX (Phase 30)

**Status:** Accepted
**Date:** 2026-09-17
**Author:** Agent (Phase 30 implementation)

## Context

Phase 25 ([PD-014](014-telegram-channel-foundation.md)) let an Assistant conversation be continued from either the Web app or a linked Telegram chat — a `ChannelConnection` points at the user's "current" conversation, and either surface can add turns to it. Phases 27–29 hardened reliability and gave operators visibility and remediation tooling for that shared conversation state, but none of them touched what the conversation/turn DTOs actually expose: `ConversationSummaryDto`, the `getOwnedConversation` response, and `AssistantMessage`/`AssistantTurn` carry no channel/source concept at all. A user reading their Web Assistant history has no way to tell that a given turn was actually sent from Telegram — the audit that proposed this phase named this directly: reliability and operator safety are addressed, and the next useful step is making that already-existing cross-channel state transparent, not adding new channel behavior.

## Decision

**Selected: a small, additive `AssistantTurn.channel` column, stamped after the fact by the channel inbound worker, surfaced through the existing conversation read paths as `WEB` / `TELEGRAM` — no new endpoint, no field ever threaded through the Web/Telegram-shared application/provider-runtime code.**

- **Schema:** `enum AssistantChannel { WEB TELEGRAM }`; `AssistantTurn.channel AssistantChannel @default(WEB)`. A turn defaults to `WEB` at creation — `beginTurn` and its `BeginTurnInput` are untouched.
- **Write path — stamped, not threaded.** `src/channels/workers/inbound.worker.ts` already learns a turn's id at the end of both the MESSAGE path (`processJob`) and the CALLBACK path (`processCallbackJob`, via `renderApplicationResult`'s `turnId` in the returned `CallbackActionResult`) — after the fact, it updates that one turn's `channel` to `TELEGRAM`. `application.service.ts`, `provider-runtime.ts`, and `financial-draft.service.ts` — the code shared by both the Web HTTP path and every Telegram interaction (message and callback) — are **not modified**. The stamp is best-effort: a failed update is caught and logged, never blocking reply delivery or job success, because channel attribution is metadata, not a correctness-load-bearing write.
- **`renderApplicationResult` gains an optional `turnId` field** on its return shape (`CallbackActionResult`), populated from the underlying `AssistantApplicationResult.turnId` this function already receives. This is the only change to shared channel-adapter code, and it is purely additive (an optional field), not a behavior change to any existing caller.
- **Read path — reuses `conversation.service.ts`, adds no endpoint.** `getOwnedConversation`'s existing `turns` select gains `channel`; the conversation object gains a derived `sourceChannels: AssistantChannel[]` (the distinct set of its turns' channels, in a fixed `[WEB, TELEGRAM]` display order). `listOwnedConversations` gains the same `sourceChannels` per summary row via one additional `assistantTurn.groupBy(['conversationId', 'channel'])` query, matching the existing pattern of `listOwnedConversations`'s separate title-lookup query.
- **One-time backfill, not an ongoing derivation.** The migration also runs `UPDATE assistant_turns SET channel = 'TELEGRAM' FROM channel_inbound_jobs WHERE assistant_turn_id = id AND provider = 'TELEGRAM'` — a best-effort retag of turns whose originating `ChannelInboundJob` row has not yet been retention-purged (`src/channels/retention.ts`, default 7 days success / 28 days terminal). Turns whose job has already been purged, and turns from a callback interaction (which was never linked via `assistantTurnId` before this phase — see Rejected alternatives), keep the `WEB` default. This is a one-time, bounded, documented imprecision in historical data — every turn created after this phase ships is stamped correctly and permanently, independent of retention.
- **Frontend (`pocket-mint-fe`):** `AssistantConversationSummary.sourceChannels?` and `AssistantTurn.channel?` are added as optional fields (absent on an older backend response is treated as unknown, never assumed `WEB`). A subtle badge (small icon + `sr-only`/`title` text, no color-only signal) appears in the conversation history list when `sourceChannels` includes `TELEGRAM`, and the same subtle marker appears next to a message's role label in the timeline when its turn's `channel` is `TELEGRAM` — nothing renders for the default `WEB` case, keeping the common path visually unchanged.

## Rejected alternatives

- **Deriving channel purely by joining `AssistantTurn` to `ChannelInboundJob.assistantTurnId` at read time, with no schema change.** Rejected: this was the first design considered specifically because it needed no migration, but two problems ruled it out. First, `ChannelInboundJob` rows are retention-purged (7/28 days, `src/channels/retention.ts`) — attribution would silently decay from `TELEGRAM` back to `WEB` the moment the backing job row aged out, which is the opposite of the transparency this phase exists to provide. Second, and more seriously: a CALLBACK-job-originated turn (created by selecting a clarification option or otherwise pressing an inline-keyboard button) was **never linked back to its `ChannelInboundJob` via `assistantTurnId`** in the existing code — only the MESSAGE path sets that field. A pure-derivation approach would have silently misattributed every Telegram callback-driven turn as `WEB`, forever, not just after a retention window. The persisted-column design fixes both: it is durable (no retention dependency) and it is stamped from both the MESSAGE and CALLBACK worker paths.
- **Threading an explicit `channel` parameter through `application.service.ts` (`execute`, `selectClarification`, `submitGuidedClarification`, `cancelClarification`, `finalizeTransactionDraft`) and `provider-runtime.ts` (`sendMessage` and its four callback passthroughs).** Rejected: discovery found these are the exact set of methods both the Web HTTP path and every Telegram interaction share by design — `provider-runtime.ts` even documents its callback passthroughs as "no parallel clarification/draft logic here." Threading a new parameter through this financial/clarification-sensitive surface for pure attribution metadata is a materially larger, riskier diff than stamping the one row that already needs no other change, for no benefit over the stamp-after-the-fact approach.
- **`AsyncLocalStorage`-based ambient channel context**, set once per request/job and read wherever a turn is created. Rejected: nothing in this codebase uses ambient async context — every cross-cutting value (`correlationId`, `userId`) is threaded explicitly as a parameter. Introducing a second, ambient-context paradigm solely for this one field would be a genuinely new abstraction for a problem the stamp-after-the-fact approach already solves without one.
- **A `conversation.sharedWithTelegram` boolean** derived from whether a `ChannelConnection` currently points its `conversationId` at this conversation (i.e., "is this the chat's active conversation right now," as distinct from "did a turn ever come from Telegram"). Deferred, not built: `sourceChannels` already answers the concrete gap the audit named (turns with no visible attribution); a live "currently linked" indicator is a different, narrower question with no product ask behind it yet. Recorded here as the natural extension if a future need for it appears — see Follow-up.
- **A new `/assistant/conversations/:id/channels` (or similar) endpoint.** Rejected: both fields fit naturally as additive projections on the existing `GET /assistant/conversations` and `GET /assistant/conversations/:id` responses; a dedicated endpoint would be unrequested surface for two small derived fields.

## Consequences

**Positive:**

- A user can now see, in the Web Assistant, that a given past turn came from Telegram — closing the exact transparency gap the audit named — without any new provider identifier ever reaching the client.
- The write-path change is one small, well-isolated addition (a post-hoc `assistantTurn.update` in the channel worker, plus one optional field on `CallbackActionResult`) — `application.service.ts`, `provider-runtime.ts`, and `financial-draft.service.ts` are untouched, so this phase carries essentially no risk to financial or clarification logic.
- Attribution is durable going forward: unlike a pure-derivation design, it does not decay when `ChannelInboundJob` rows are retention-purged.
- Both known-missed-attribution cases from the previous phases' data (retention-purged jobs, and every historical callback-originated turn) are now closed for all *future* turns, and partially closed for existing data via the one-time backfill.

**Costs:**

- The one-time backfill cannot recover turns whose `ChannelInboundJob` has already been purged, or any pre-Phase-30 callback-originated turn — those remain labeled `WEB` permanently. This is a known, bounded, one-time data-quality gap, not an ongoing one.
- The channel stamp is best-effort (caught and logged on failure) — in the rare case that update fails, that one turn's attribution silently stays `WEB` even though it came from Telegram. Reply delivery and job success are intentionally never blocked on this, since attribution is presentation metadata, not a financial or delivery guarantee.
- Two additional read queries (`groupBy` for the conversation list, one extra `select` field for the detail view) — both bounded by page size / conversation turn count, matching the cost profile of the existing `listOwnedConversations` title lookup.

## Related documents

- [PD-014 — Telegram Channel Foundation](014-telegram-channel-foundation.md)
- [PD-015 — Durable Channel Processing](015-durable-channel-processing.md)
- [PD-016 — Telegram Interactive Workflows](016-telegram-interactive-workflows.md)
- [PD-017 — Assistant Request-Level Idempotency & Turn Recovery](017-assistant-request-level-idempotency.md)
- [PD-018 — Assistant Operations Visibility](018-assistant-operations-visibility.md)
- [PD-019 — Safe Operator Remediation](019-safe-operator-remediation.md)
- [Assistant Core Architecture §15.8](../../architecture/assistant-core-architecture.md#158-channel-source-attribution-phase-30)
- [Implementation Roadmap — Phase 30](../../development/implementation-roadmap.md)

## Non-goals / explicitly out of scope

- No Telegram `externalChatId`, `externalUserId`, or `externalSenderId` exposure through any DTO, at any point.
- No callback token or raw Telegram/provider payload exposure.
- No new channel provider implementation — `AssistantChannel` has exactly the two values the system already has (`WEB`, `TELEGRAM`); nothing about Telegram integration itself changes.
- No durable queue redesign — the channel stamp reuses the existing worker call sites and their existing best-effort/idempotent posture.
- No admin or operator UI or endpoint of any kind.
- No automatic cross-channel conflict resolution — this phase makes cross-channel state visible, it does not change how a shared conversation is continued or arbitrated between channels.
- No broad Assistant UI redesign — the only frontend change is a subtle badge in the existing history list and timeline components.
- No live "currently shared with Telegram" indicator (`sharedWithTelegram`/`telegramLinked`) — see Rejected alternatives; may be revisited as a follow-up if a concrete product need for it appears.

## Follow-up Tasks

- If a future need arises to show whether a conversation is the Telegram connection's *current* target (not just whether it has historical Telegram turns), add a derived `conversation.telegramLinked: boolean` from `ChannelConnection.conversationId`/`status`, following the same additive-DTO pattern as this phase.
- If historical accuracy for pre-Phase-30 data becomes a real requirement (rather than a documented one-time gap), a wider backfill would need to be sourced from Telegram-side logs/audit records outside this repository, since the source-of-truth `ChannelInboundJob` rows for old, retention-purged messages no longer exist.
