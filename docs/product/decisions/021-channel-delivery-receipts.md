# PD-021 — Channel Delivery Receipts & User-Facing Send State (Phase 31)

**Status:** Accepted
**Date:** 2026-09-21
**Author:** Agent (Phase 31 implementation)

## Context

Phase 30 ([PD-020](020-channel-source-attribution.md)) made it visible, in the Web Assistant, that a given turn originated from Telegram — but attribution stops at "where it came from." The channel worker pipeline (Phase 26A/26B, [PD-015](015-durable-channel-processing.md)/[PD-016](016-telegram-interactive-workflows.md)) already tracks the durable lifecycle of the *outbound* reply itself, through `ChannelOutboundDelivery.status` (`PENDING → SENDING → SENT`, or `FAILED_RETRYABLE`/`FAILED_TERMINAL` with backoff and a bounded retry schedule), and Phase 29 ([PD-019](019-safe-operator-remediation.md)) gave operators a safe remediation path for the terminal-failure case. None of that state reaches the user: a Telegram-originated turn shows the same Assistant reply bubble whether the message was actually delivered, is still retrying, or terminally failed to reach Telegram. This phase closes that gap — a small, additive, privacy-safe delivery-status layer, reusing state the channel worker already persists.

## Decision

**Selected: a computed (not stored) `turn.deliveryStatus` field, aggregated at read time from the existing `ChannelOutboundDelivery` row for a TELEGRAM-channel turn, surfaced through the same `getOwnedConversation` read path Phase 30 already extended — no new endpoint, no schema change, no write-path change.**

- **No migration.** Unlike Phase 30's `channel` (a fact about the past, which needed a durable stamp precisely because it must never change), delivery status is live, dynamic state — it is expected to change between `PENDING` and a terminal outcome over the following seconds. A stored column would need to be kept in sync with every worker transition for no benefit over reading the already-authoritative `ChannelOutboundDelivery` row directly. `getOwnedConversation` (`src/assistant/conversation.service.ts`) gains one additional bounded query — `channelInboundJob.findMany({ where: { assistantTurnId: { in: telegramTurnIds } }, select: { assistantTurnId: true, deliveries: { where: { kind: 'SEND_MESSAGE' }, select: { status: true } } } })` — scoped to the page's TELEGRAM-channel turns only, mirroring the cost profile of the `toolExecutions` select already on the same query and Phase 30's `sourceChannels` groupBy. `listOwnedConversations` is unchanged: delivery status is a per-turn concept, and the summary list does not return turns.
- **Closed, safe status enum:** `'NOT_APPLICABLE' | 'PENDING' | 'PROCESSING' | 'DELIVERED' | 'FAILED'`, mapped from the internal `ChannelDeliveryStatus`:

  | Internal `ChannelDeliveryStatus` | Safe `AssistantDeliveryStatus` | Why |
  |---|---|---|
  | `PENDING` | `PENDING` | Queued, not yet attempted. |
  | `SENDING` | `PROCESSING` | An attempt is in flight. |
  | `SENT` | `DELIVERED` | Telegram accepted the message. |
  | `FAILED_RETRYABLE` | `PROCESSING` | Still within the bounded retry/backoff schedule — showing "failed" here would be misleading since it may yet succeed. |
  | `FAILED_TERMINAL` | `FAILED` | Exhausted retries; this is the state Phase 29's operator remediation already exists to act on. |
  | *(WEB turn — no channel delivery to report)* | `NOT_APPLICABLE` | Explicit, not omitted — a WEB turn is not "unknown," it simply has no outbound channel delivery. |
  | *(TELEGRAM turn, delivery row not found — retention-purged, or a rare pre-delivery-row race)* | *(field absent)* | Distinct from every other state: never guess success or failure for data that no longer exists. Bounded, same posture as Phase 30's documented historical-attribution gap. |

- **Placement — the turn, not the message.** Mirrors Phase 30's `channel`: both a turn's `USER` and `ASSISTANT` messages share one channel and one delivery outcome (the outbound reply this turn produced), so the field belongs where `channel` already lives, keeping the DTO shape and the frontend's existing turn-id-keyed lookup pattern unchanged in kind. The frontend renders the status only next to the `ASSISTANT` message, never the `USER` one — delivery is about the reply reaching the user, not their own outgoing instruction.
- **Frontend (`pocket-mint-fe`):** `AssistantTurn.deliveryStatus?: AssistantDeliveryStatus` — optional, absent on an older backend response (same pattern as `channel`). `AssistantMessage` renders a small icon + `sr-only`/`title` label next to the role badge, restrained to exactly three visible states (`PENDING`/`PROCESSING` both read as "delivering", `DELIVERED` as "sent", `FAILED` as "failed"); `NOT_APPLICABLE`, an absent field, or a WEB turn render nothing, keeping the default case visually unchanged, same as the Phase 30 badge.

## Rejected alternatives

- **A stored `AssistantTurn.deliveryStatus` column, updated by the channel workers on every transition.** Rejected: this is dynamic state that already has one authoritative source (`ChannelOutboundDelivery.status`, itself already durable and retry-aware). Duplicating it onto `AssistantTurn` would require touching `inbound.worker.ts` and `outbound.worker.ts` on every status transition for a field that read-time aggregation already answers correctly, adding write-path risk to the channel worker pipeline for no benefit — unlike Phase 30's `channel`, where the persisted stamp was required specifically to survive `ChannelInboundJob` retention purges and a callback-path gap that read-time derivation could not close.
- **`message.channelDeliveryStatus` instead of `turn.deliveryStatus`.** Considered per the phase brief's own alternative framing. Rejected in favor of the turn-level field: delivery status is a property of the turn's one outbound reply, not of any individual persisted message row, and placing it on the turn keeps it consistent with `channel`'s existing placement and the frontend's existing `turnId`-keyed lookup — a message-level field would either duplicate the same value onto both the USER and ASSISTANT rows of a turn, or require the client to special-case which message it applies to, for no benefit.
- **A user-facing retry/resend action when `FAILED_TERMINAL`.** Explicitly out of scope for this phase (see Non-goals) — Phase 29's operator remediation (`requeue-outbound`) is the only remediation path today, and this phase does not add a new user-triggered mutation surface for it.
- **Exposing `providerMessageId` or `attempt` count to the client.** Rejected: neither is needed to render "delivering / sent / failed," and both are provider/internal implementation detail — `attempt` in particular could hint at retry internals with no product value, while `providerMessageId` is a Telegram-assigned identifier with no safe reason to leave the backend.

## Consequences

**Positive:**

- Closes the remaining gap between what the channel worker pipeline already knows (durable, retry-aware delivery lifecycle) and what the user can see, without introducing any new state, write path, or endpoint.
- Zero migration risk: purely a read-time projection over data that already exists and is already correctly maintained by the Phase 26A/26B worker pipeline.
- A `FAILED_RETRYABLE` delivery reads as "delivering," not "failed," so a transient provider hiccup that resolves on retry never shows the user a false failure.
- Consistent with Phase 30's placement and optionality conventions — no new mental model for frontend consumers of `AssistantTurn`.

**Costs:**

- One additional bounded query per `getOwnedConversation` call when the page contains at least one TELEGRAM-channel turn — same cost class as Phase 30's `sourceChannels` groupBy, bounded by page size.
- The delivery-row-purged edge case (a TELEGRAM turn old enough that its `ChannelOutboundDelivery`/`ChannelInboundJob` rows were retention-purged, default 7/28 days) resolves to an absent field rather than a real historical answer — a documented, bounded gap, not an ongoing one, and it only affects conversations a user reopens long after the fact.

## Related documents

- [PD-015 — Durable Channel Processing](015-durable-channel-processing.md)
- [PD-016 — Telegram Interactive Workflows](016-telegram-interactive-workflows.md)
- [PD-018 — Assistant Operations Visibility](018-assistant-operations-visibility.md)
- [PD-019 — Safe Operator Remediation](019-safe-operator-remediation.md)
- [PD-020 — Channel Source Attribution & Cross-Channel UX](020-channel-source-attribution.md)
- [Assistant Core Architecture §15.9](../../architecture/assistant-core-architecture.md#159-channel-delivery-receipts-phase-31)
- [Implementation Roadmap — Phase 31](../../development/implementation-roadmap.md)

## Non-goals / explicitly out of scope

- No Telegram `externalChatId`, `externalUserId`, or `externalSenderId` exposure through any DTO, at any point.
- No `providerMessageId`, `destinationChatId`, rendered outbound text beyond what is already visible as the Assistant message's own `content`, reply markup, or callback token exposure.
- No user-triggered resend/retry action or button — Phase 29's operator remediation (`opsRemediate.ts requeue-outbound`) remains the only remediation path for a terminally failed delivery.
- No automatic replay or remediation of any kind.
- No queue redesign, and no change to the Phase 26A/26B worker retry/backoff behavior itself.
- No admin or operator UI or endpoint.
- No new channel provider.
- No broad Assistant UI redesign — the only frontend surface is a subtle icon/label next to a Telegram-originated Assistant reply's role badge, following the exact placement pattern the Phase 30 channel badge already established.
- No `deliveryStatus` on `listOwnedConversations` summaries — this is a per-turn concept, and the list endpoint does not return turns.

## Follow-up Tasks

- If a real product need for user-initiated resend ever appears, it would need its own decision record — this phase deliberately stops at visibility, matching Phase 30's own precedent of separating "make state visible" from "let the user act on it."
- If the retention-purged-delivery gap proves to matter in practice (a user reopening a very old Telegram-originated conversation), the same bounded, documented-gap posture Phase 30 already accepted for historical channel attribution applies here — no new mechanism is proposed until a concrete need is shown.
