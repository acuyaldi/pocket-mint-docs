# PD-016 — Telegram Interactive Clarification & Confirmation (Phase 26A)

**Status:** Accepted
**Date:** 2026-07-27
**Author:** Agent (Phase 26A implementation)

## Context

PD-014 (Phase 25) deliberately deferred Telegram-native interactive actions as "Scope B" — every clarification and financial-draft result the Assistant produced was rendered as a fixed web-handoff message, regardless of how simple the underlying choice was. PD-015 (Phase 26B) then made the *text* path durable (inbox/outbox, operation-identity guard, claim/lease workers) but explicitly left Scope B untouched.

Phase 26A closes that gap for the bounded set of actions the existing domain services already support end-to-end: selecting a clarification option, cancelling a clarification, and confirming or cancelling a pending financial draft. It does not add a new capability to the Assistant, the Persistent Clarification Engine, or the Pending Financial Draft lifecycle — it adds a second, safe way to invoke the exact same authoritative operations those systems already expose to the web client.

Discovery for this phase confirmed:

- Telegram update parsing (`telegram/schema.ts`) only ever accepted `message`; any `callback_query` update was silently dropped as unsupported.
- `ClarificationOption.tokenDigest` and `AssistantFinancialDraft` confirm/cancel are called by the HTTP controller through `application.service.ts` (clarification) and `financial-draft.service.ts` (drafts) directly — never through `provider-runtime.ts`, which exposed only `sendMessage`.
- `ChannelOutboundDelivery` was 1:1 with `ChannelInboundJob` and had no concept of an action other than "send this text."
- `ChannelAssistantOperation`'s primary key (`channel:<provider>:<jobId>`) already encodes "one job, one durable result" generically — nothing about it is specific to an Assistant turn.

## Decision

**Selected: opaque `ChannelCallbackToken` (digest-only, mirrors `ClarificationOption.tokenDigest`) + a provider-neutral `interaction.service.ts` orchestration boundary + minimal, additive extensions to the existing durable channel schema.** No new pipeline, no second worker, no parallel financial logic.

### Callback ingestion

- `telegram/schema.ts` gained a sibling parser, `parseTelegramCallbackQuery`, alongside the existing `parseTelegramUpdate` — same allowlist discipline (private chat only, bounded `data` ≤256 chars, structural validation, nothing else leaves the file). `envelope.ts` normalizes either shape into the same `InboundChannelMessage` envelope, now carrying an optional `kind: 'MESSAGE' | 'CALLBACK'` discriminator plus (for callbacks) `callbackQueryId` and `callbackMessageId` — the latter taken directly from `callback_query.message.message_id`, which is exactly the message the pressed inline keyboard is attached to, so keyboard cleanup never needs a separate provider-message lookup.
- `ChannelInboundJob` gained `kind` (default `MESSAGE`, so every existing row is unaffected), `callbackQueryId`, `callbackMessageId`. The `(provider, externalUpdateId)` unique constraint — the dedup guarantee — is unchanged and covers callback updates identically.
- `telegram.service.ts` (the webhook-facing service) durably inserts the job first, then — for a callback update only — synchronously calls `client.answerCallbackQuery(...)` with a neutral acknowledgment. Acknowledgment failure is swallowed and only logged (`channel.telegram.callback_acknowledged`, outcome `failed`); the job is already committed, so a failed ack never undoes durable acceptance. The webhook still never resolves ownership, selects a clarification, confirms a draft, invokes the Assistant, or edits a Telegram message — it does exactly what PD-015 already established for text updates, plus one bounded, best-effort HTTP call.

### `ChannelCallbackToken`

A new, provider-neutral model (`callbackToken.service.ts`): `id, provider, tokenDigest, connectionId, conversationId, interactionType (CLARIFICATION_SELECT | CLARIFICATION_CANCEL | DRAFT_CONFIRM | DRAFT_CANCEL), clarificationRequestId?, clarificationOptionId?, financialDraftId?, actionSecret?, status (PENDING | CONSUMED | EXPIRED | CANCELLED | STALE), expiresAt, consumedAt, createdAt, updatedAt`. `@@unique([provider, tokenDigest])`.

- **Digest-only, exactly like `ClarificationOption.tokenDigest`.** `generateCallbackToken()` = `'cbk_' + randomBytes(24).base64url()`; the plaintext is returned to the caller once (to embed as Telegram `callback_data`) and never persisted. Telegram `callback_data` therefore never contains a user ID, connection ID, conversation ID, clarification ID, draft ID, option ID, action name, amount, merchant, wallet, category, or token digest — only this one opaque handle.
- **`actionSecret` — the one deliberate wrinkle, and the answer to "can clarification option tokens be wrapped or must they stay internal."** `clarification.service.ts`'s `select()` needs the *raw* `clarify_...` token (it compares its digest against the persisted `ClarificationOption.tokenDigest`), but that raw token is only ever returned once, at clarification-creation time — long before a Telegram button is pressed. Rather than exposing it in `callback_data` (forbidden) or re-deriving it later (impossible — it's never persisted), `ChannelCallbackToken.actionSecret` wraps it: populated only for `CLARIFICATION_SELECT`, read server-side to call `selectClarification`, and cleared to `null` the moment the token is claimed. It is never serialized to Telegram, never logged, and is optional/unused for the other three interaction types (`DRAFT_CONFIRM`/`DRAFT_CANCEL` only need the draft ID already on the row plus a deterministic idempotency key; `CLARIFICATION_CANCEL` only needs the clarification ID).
- **One-time claim is the cross-press race gate.** `claimCallbackToken` does the same conditional `updateMany({ where: { id, status: 'PENDING' }, data: { status: 'CONSUMED', actionSecret: null } })` pattern as `ClarificationRequest`. Two independent button presses on the same token (two different `ChannelInboundJob` rows, since Telegram gives each press its own `update_id`) race on this claim; only the winner ever calls the authoritative service. The underlying domain services' own guards (clarification's conditional claim, the draft's `pg_advisory_xact_lock` + idempotency record) are a second, independent layer — not the only one, as they would be if the callback token merely observed rather than gated.
- **No FK to `ClarificationRequest`/`ClarificationOption`/`AssistantFinancialDraft`.** These stay plain string columns validated at the application layer inside `interaction.service.ts` (ownership re-derivation), the same trade-off already accepted for `ChannelInboundJob.channelConnectionId` vs. its own text fields. Adding those FKs would touch three existing, heavily-tested models for referential integrity the application layer already enforces before ever reading the ID.

### Operation identity — extended, not duplicated

`ChannelAssistantOperation` gained `kind (ASSISTANT_TURN | CALLBACK_INTERACTION, default ASSISTANT_TURN)`, `callbackTokenId?`, `terminal_status?`. The primary key is still `channel:<provider>:<jobId>` — one inbound job still maps to at most one operation row, now either kind. `beginCallbackOperation`/`completeCallbackOperation` mirror `beginAssistantOperation`/`completeAssistantOperation` exactly (insert-first-wins, `'new' | 'replay' | 'ambiguous'`), so a reclaimed callback job replays its persisted `terminalStatus`/`renderedText` verbatim and never repeats the callback action — the same crash-window C guarantee PD-015 established for Assistant turns, now covering callback interactions too. This was extending one existing model with a closed enum, not building a second parallel operation table that would have falsely implied every callback is an Assistant turn.

### `interaction.service.ts` — the one orchestration boundary

A new, provider-neutral `src/channels/interaction.service.ts` is the *only* place a callback job's button press becomes an authoritative action. `processCallback(...)` re-derives every piece of ownership from scratch, in this order, never trusting the Telegram payload for anything but the opaque token: resolve token by digest → verify `PENDING` and unexpired → load `ChannelConnection` → verify `ACTIVE` → verify `connection.externalUserId === job.externalSenderId` → verify `token.conversationId === connection.conversationId` → atomically claim the token → dispatch to `providerRuntime.selectClarification` / `.cancelClarification` / `.confirmDraft` / `.cancelDraft`. Any failure at any step returns a bounded terminal status (`not_found`, `consumed`, `expired`, `cancelled`, `stale`, `connection_revoked`, `restart_required`, or a mapped `AssistantError` code) without ever creating a new clarification/draft or reactivating a terminal one.

`interaction.service.ts` reaches the Assistant only through `AssistantProviderRuntime`, extended with four thin passthroughs (`selectClarification`, `cancelClarification`, `confirmDraft`, `cancelDraft`, `getAssistantState`) that forward to the exact same `application.service.ts` / `financial-draft.service.ts` methods `assistant.controller.ts` already calls for the web path — no parallel financial logic, no new validation, no direct transaction creation. `test/channels/telegramAdapterBoundary.test.ts` was extended (recursively, from `src/telegram/` to also cover `src/channels/`) to assert this: no file in either directory may import `transaction.service`, `wallet.service`, `category.service`, `merchant.service`, `clarification.service`, `financial-draft.service`, or `application.service` directly, and the only Assistant-namespace imports allowed anywhere are `provider-runtime` and the structural `errors` module.

### Outbound: keyboards and cleanup

`ChannelOutboundDelivery` gained `kind (SEND_MESSAGE | EDIT_REPLY_MARKUP, default SEND_MESSAGE)`, `replyMarkup (Json?)`, `targetMessageId`. The prior `inboundJobId @unique` became `@@unique([inboundJobId, kind])` — a callback job can now produce both a result-text delivery and a keyboard-cleanup delivery, while each individual kind is still create-once-per-job (P2002 on reclaim), preserving the exact idempotency property PD-015 relied on.

- **Rendering** is split cleanly: `interaction.service.ts` (and the equivalent code path in `inbound.worker.ts` for a fresh message-path clarification/draft) produces a provider-neutral `InteractionKeyboard` (`{ rows: [[{ label, discriminator?, token }]] }`) — no Telegram types. `telegram/keyboardRenderer.ts` is the only place that maps it to Telegram's `{ inline_keyboard: [[{ text, callback_data }]] }` JSON, truncating button text to Telegram's 64-character limit.
- **Cleanup** targets `callback_message_id` captured straight from the callback payload — no separate provider-message-ID lookup is needed for this path. `outbound.worker.ts` dispatches on `delivery.kind`: `SEND_MESSAGE` calls `client.sendMessage(chatId, text, replyMarkup?)` (now capturing `message_id` from Telegram's response into `providerMessageId`, which the client previously discarded entirely); `EDIT_REPLY_MARKUP` calls `client.editMessageReplyMarkup(chatId, messageId)` with no markup argument, which clears the keyboard. A failed cleanup delivery retries/fails independently of the domain action it followed — it can never roll back or repeat the underlying clarification/draft mutation, only the keyboard edit itself.
- **`answerCallbackQuery` is synchronous and never the authoritative result.** It runs once, in the webhook, with a neutral message ("Processing…" or no text) — success or failure of this call has no bearing on whether the durable job is accepted or how the callback action resolves.

### Message-path buttons (not just callback follow-ups)

The very first time a text message's Assistant turn produces a `clarification_required` result with options, or a freshly created draft, `inbound.worker.ts` now builds real `ChannelCallbackToken` rows and an inline keyboard for it directly (reusing `interaction.service.ts`'s `renderApplicationResult`), instead of unconditionally falling back to the fixed web-handoff message `replyRenderer.ts` established in PD-014. That fallback message is still used for every case Telegram genuinely cannot handle (see Exclusions) and for a reclaimed/replayed job, which resends its persisted text without rebuilding buttons — rebuilding would mint duplicate token rows for a state that may have already moved on.

## Rejected alternatives

- **Free-form Telegram text as clarification input.** Rejected outright — this would require conversational state tracking ("what is this next message a reply to?") this phase doesn't build, and would reintroduce exactly the un-trusted, client-driven identifier problem opaque tokens exist to prevent. Free-form clarification (e.g. typing a new merchant name) remains web-only; Telegram states this limitation via the same web-handoff message.
- **Embedding the option/action identifiers directly in `callback_data`, relying on server-side ownership checks alone.** Rejected: it would work functionally, but violates the explicit "opaque handles only" requirement and creates an enumeration/tampering surface (a modified `callback_data` referencing another user's draft ID) that digest-only lookup removes by construction — a forged token simply doesn't resolve to any row.
- **A second, parallel `ChannelInteractionOperation` table instead of extending `ChannelAssistantOperation`.** Rejected in favor of extending the existing model with a closed `kind` enum — same reasoning PD-015 used for evolving `ChannelUpdateDedup` into `ChannelInboundJob` in place: one row, one place, no risk of the two operation concepts drifting out of sync on what "one job → one result" means.
- **Loosening `ChannelOutboundDelivery`'s per-job uniqueness entirely (dropping it) instead of scoping it to `(inboundJobId, kind)`.** Rejected: dropping it outright would remove the exact idempotency property (create-once-per-job, P2002-safe reclaim) PD-15 relies on for the pre-existing text-send path. Scoping to `(inboundJobId, kind)` is the minimal change that both preserves that guarantee per-action and allows the new multi-delivery-per-job case.
- **Implementing STEP_UP confirmation inside Telegram.** No tool in the registry currently uses the `STEP_UP` confirmation policy (only `EXPLICIT`, via `DRAFT_AND_CONFIRM`) — there is nothing to wire up yet, and building speculative infrastructure for a policy tier with zero current callers was rejected. When the first `STEP_UP` tool exists, Telegram renders the existing web-handoff message and does not offer a Confirm button, exactly as free-form clarification does today.
- **Adding a fifth `ANSWER_CALLBACK` delivery kind for queued/retryable acknowledgment.** Rejected — `answerCallbackQuery` is synchronous and happens exactly once, in the webhook, per spec; queuing it would blur the line between "synchronous, non-authoritative ack" and the durable outbox, which the spec explicitly separates.

## Security resolution — precisely, in order

For every `CALLBACK` job, `interaction.service.ts` performs, and only in this order:

1. Resolve `ChannelCallbackToken` by `sha256(rawCallbackData)` digest — an unknown digest is indistinguishable from any other terminal state to the end user (`not_found`).
2. Reject if `status !== 'PENDING'` (already consumed/expired/cancelled/stale) — reports the *actual* terminal state, not a generic error, so a legitimate reclaim can still make forward progress.
3. Reject and lazily transition to `EXPIRED` if `expiresAt` has passed.
4. Load the bound `ChannelConnection`; reject if missing or not `ACTIVE` (`connection_revoked`) — a delivered keyboard becomes unusable the moment the connection is revoked, with no separate cleanup step required.
5. Reject if `connection.externalUserId !== job.externalSenderId` (`restart_required`, deliberately generic — never reveals that this was specifically an identity mismatch to the Telegram side).
6. Reject if `token.conversationId !== connection.conversationId` (`stale` — the conversation moved on since this keyboard was rendered).
7. Atomically claim the token (`PENDING → CONSUMED`) — the cross-press race gate.
8. Only now call the authoritative clarification/draft service, with `userId` taken from step 4's connection — never from anything in the Telegram payload.

Telegram identity (`callback_query.from.id`) is used **exclusively** as a value to compare against the already-resolved connection's `externalUserId` — it is never itself trusted as a Pocket Mint user identifier, matching PD-014's original linking-security posture.

## Delivery guarantees

Precisely, and without overstating any of them:

- **Callback acceptance:** durable before acknowledgment, identical in mechanism to PD-015's message-update acceptance — the job is committed in the same transaction that would report a duplicate, before `answerCallbackQuery` is ever called.
- **Update deduplication:** unchanged mechanism (`(provider, externalUpdateId)` unique constraint), now covering callback updates.
- **Callback action execution:** at-most-once per `ChannelCallbackToken`, enforced by that token's own one-time claim — independent of, and in addition to, the underlying domain idempotency (clarification's conditional claim; the draft's advisory lock + idempotency record). A reclaimed job with an already-`ambiguous` operation fails terminally for manual review, the same crash-window-C posture PD-015 established for Assistant turns.
- **Clarification consumption:** unchanged from the Persistent Clarification Engine — one-time, digest-compared, database-time-bounded expiry. Nothing about this phase weakens or duplicates that mechanism.
- **Financial mutation:** unchanged from the Pending Financial Draft lifecycle — advisory-locked, idempotency-keyed, at-most-once. A confirm-vs-cancel race on the same draft has exactly one authoritative winner, decided entirely inside `financial-draft.service.ts`, not by which callback token happened to be claimed first.
- **Outbound Telegram send/edit: at-least-once / best-effort, not exactly-once** — identical posture to PD-015. A crash after Telegram accepts a send/edit but before this backend records `SENT` can, in principle, produce a duplicate user-visible message or a keyboard that isn't cleared until the next successful edit attempt. This can never duplicate a clarification selection or a financial mutation — only possibly duplicate what the user sees.
- **`answerCallbackQuery` failure:** never undoes durable job acceptance; Telegram may show a spinner timeout, and the durable worker still resolves the button press normally.

## Observability

Extends the existing closed, allowlisted `AssistantLogEvent` taxonomy (`src/utils/logger.ts`) with `interactionType` and `terminalStatus` — both bounded, closed-set string labels, structurally incapable of carrying token plaintext, digests, message text, option labels, financial preview data, or raw Telegram payloads (the interface has no index signature). New events: `channel.callback.terminal`, `channel.callback.duplicate`, `channel.callback.token_consumed`, `channel.callback.token_expired`, `channel.callback.completed`, `channel.callback.rejected`, `channel.telegram.callback_acknowledged`, `channel.telegram.keyboard_cleanup_scheduled`, `channel.telegram.keyboard_cleanup_completed`, `channel.telegram.keyboard_cleanup_failed`.

## Retention

`cleanupChannelRecords` (already the sole retention mechanism, run opportunistically on each worker poll tick per PD-015) gained a `channelCallbackToken` pass: only rows in a terminal status (`CONSUMED | EXPIRED | CANCELLED | STALE`) past the same retention window as `ChannelOutboundDelivery`/`ChannelInboundJob` are deleted. `PENDING` rows — including ones past `expiresAt` that haven't yet been lazily transitioned by a read — are never touched, so a still-processing or not-yet-resolved job can never have its token pulled out from under it.

## Consequences

**Positive:**

- The three most common Assistant follow-ups (pick a wallet/merchant/category, confirm or cancel a draft) now resolve inside Telegram, without the user leaving the chat — while every financial mutation and every clarification consumption still flows through exactly the same authoritative, already-audited services the web client uses.
- The provider-neutral `interaction.service.ts` boundary means a second interactive channel (were one ever added) reuses this orchestration without duplicating the security-resolution logic.
- `providerMessageId` is now actually captured (it existed as a column since PD-015 but was never populated) — both because keyboard cleanup needs it in principle, and because the callback payload turned out to make that lookup unnecessary for the one thing this phase needed it for.

**Costs:**

- Two additional durable rows per interactive prompt (the callback token(s) plus, on resolution, a second outbound delivery for keyboard cleanup) — bounded by the same retention policy as everything else in the durable channel schema, not unbounded growth.
- `ChannelAssistantOperation` and `ChannelOutboundDelivery` now each carry a `kind`/discriminator dimension a future reader must account for — an explicit, documented trade against introducing a second parallel table for each.
- STEP_UP and free-form clarification remain explicitly web-only; a user who needs either still gets redirected out of Telegram, same as before this phase.

## Related documents

- [PD-014 — Telegram Channel Foundation](014-telegram-channel-foundation.md)
- [PD-015 — Durable Channel Processing](015-durable-channel-processing.md)
- [Telegram Security Architecture](../../architecture/telegram-security.md)
- [Assistant Core Architecture](../../architecture/assistant-core-architecture.md)
- [Telegram Deployment Runbook](../../development/telegram-deployment-runbook.md)
- [Implementation Roadmap — Phase 26A](../../development/implementation-roadmap.md)

## Non-goals / explicitly out of scope

- Step-up authentication inside Telegram — remains web-only until a `STEP_UP`-tiered tool exists.
- Free-form/arbitrary-text clarification responses in Telegram — remains web-only.
- Telegram Web Apps, media, voice, OCR.
- Discord, WhatsApp, n8n, or any other channel.
- Direct financial mutation outside the existing draft confirm/cancel services — none was added.
- New AI/NL parsing of any kind — callback actions are bounded, closed-enum button presses, never interpreted as language.
- Frontend changes — none were required; no Telegram-specific status, history, or control was added to the web application.
- A queue dashboard or any new operational tooling beyond the existing worker loops.
- Exactly-once Telegram UI delivery — not claimed, for the same reason PD-015 didn't claim it for text replies.
