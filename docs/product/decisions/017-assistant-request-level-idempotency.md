# PD-017 — Assistant Request-Level Idempotency & Turn Recovery (Phase 27)

**Status:** Accepted
**Date:** 2026-09-13
**Author:** Agent (Phase 27 implementation)

## Context

The post-Phase-26B Assistant/Telegram/conversation-history audit found the highest-priority remaining reliability gap in the Assistant lifecycle: `POST /assistant/messages` and `POST /assistant/execute` have no request-level idempotency contract. Telegram is already protected — `assistantOperationGuard` (PD-015, §15.5) binds one inbound job to at most one Assistant turn, with an explicit doc-comment noting "the Assistant module itself has no request-level idempotency contract, so this lives entirely in the channel layer instead of touching it." Web has no equivalent: a client retry, a double submit, a dropped connection followed by a resubmit, or a naive "resend on timeout" can each create a second turn and, for `transaction.create`, a second pending draft — from what the user experienced as one action.

`POST /drafts/:draftId/confirm` already has a proven idempotency pattern (Phase 21.4): a caller-supplied `Idempotency-Key`, `(userId, key)` database uniqueness, a PostgreSQL advisory transaction lock scoped to the draft, and exact replay of the committed result. Discovery for this phase confirmed that pattern does not transfer directly:

- `AssistantIdempotencyRecord.draftId` was a required, non-nullable foreign key — the record is inherently draft-scoped. `/messages` and `/execute` may never produce a draft (an analytics intent never does; `transaction.create` only does after successful entity resolution), so there is no draft to scope the record to at claim time.
- Draft confirm's advisory lock (`pg_advisory_xact_lock`, held for the whole request inside one Prisma transaction) works because the entire operation — validate, create the confirmation turn, call the transaction service, commit the draft — is fast and fully database-local. `/messages` is not: it invokes an external LLM provider (`invokeProvider`) for up to `ASSISTANT_PROVIDER_TIMEOUT_MS` (default 15,000 ms) with no Prisma transaction open around that call, by design (§15 Provider Adapter Boundary — the provider call must never happen inside a database transaction). Holding an advisory lock across that call was rejected outright: it would tie up a pooled database connection for the provider's entire round trip, is not what advisory locks are for, and does not compose with `invokeProvider`'s existing timeout/abort behavior.
- Draft confirm's "in-flight" case is handled implicitly — a second concurrent request simply blocks on the lock until the first transaction commits, then proceeds through the normal committed-replay path. There is no persisted "still processing" status anywhere in that flow. The audit explicitly asked for a real "still-processing" response for `/messages`/`/execute`, which requires a persisted status this pattern doesn't produce.

## Decision

**Selected: extend `AssistantIdempotencyRecord` with a nullable `turnId`, a `RUNNING | COMPLETED` status, and a stored response snapshot; claim a key with the same insert-first-wins pattern `assistantOperationGuard` already uses for Telegram, not draft-confirm's whole-request advisory lock.** One table continues to serve both draft-confirm (`draftId` populated, status effectively always `COMPLETED` because its own row is only ever visible after its owning transaction commits) and turn-level idempotency (`turnId` populated, status starts `RUNNING`), rather than a second parallel table.

### Claim / resolve / release, not lock-and-hold

`conversation.service.ts` gains three functions:

- `claimIdempotencyKey(userId, key, operation)` — attempts `assistantIdempotencyRecord.create({ ..., status: 'RUNNING' })`. Success means this request is the first with this key: proceed normally. A `P2002` unique-constraint violation on `(userId, key)` means another request already claimed it: re-read the row. If its `operation` doesn't match, the key was reused across `/messages` and `/execute` (or against a draft-confirm key) — reject with `409 ASSISTANT_IDEMPOTENCY_CONFLICT`, the same code draft-confirm already uses for cross-draft key reuse. If its status is still `RUNNING`, the original request hasn't finished — reject with `409 ASSISTANT_REQUEST_IN_PROGRESS` (new). Otherwise the row is terminal — replay its stored `responseStatus`/`responseBody` verbatim, executing nothing.
- `resolveIdempotencyKey(userId, key, { httpStatus, response, turnId })` — called once the underlying operation actually completes (successfully or with a handled business-level error; both are "terminal" from idempotency's point of view). Updates the same row to `status: 'COMPLETED'` with the response snapshot and, when available, the turn it produced.
- `releaseIdempotencyKey(userId, key)` — called instead of `resolveIdempotencyKey` when the wrapped operation throws before producing any terminal result (e.g. `assertContinuable` rejecting an already-archived conversation before a turn exists). Deletes the row only if it is still `RUNNING`, so it can never clobber a terminal row written by a concurrent request, then the caller rethrows the original error unchanged. This is what keeps a synchronous business-logic failure from being indistinguishable from a stuck claim — only an actual process crash between claim and resolve/release leaves a key `RUNNING` forever (see Crash window, below).

The claim step is a single fast insert (or a fast failed-insert-then-read), not a lock held across the request — it adds one round trip before the real work starts, not a held connection for the work's duration. This mirrors `assistantOperationGuard.beginAssistantOperation` almost exactly (see PD-015): "insert-first-wins... a conflict means a previous attempt on this exact job already started." Phase 27 generalizes that same shape from the channel layer into the Assistant module itself, for the one caller (Web) that needed it and didn't have it.

### Wrapping, not rewriting, `execute` and `sendMessage`

`application.service.ts`'s `execute()` and `provider-runtime.ts`'s `sendMessage()` each gained an optional trailing `idempotencyKey?: string` parameter. When provided, the function claims the key, delegates to the original (renamed, otherwise-untouched) implementation, and resolves the key with the result before returning it. When omitted — every existing caller, including `sendMessage`'s own internal call into `execute()` for a resolved intent plan, and every Telegram code path — behavior is byte-for-byte identical to before this phase. This is why a Telegram-originated call is unaffected: it never supplies this parameter, so `claimIdempotencyKey`/`resolveIdempotencyKey` are never invoked on that path at all; Telegram keeps using `assistantOperationGuard` exclusively, as it already did.

Because `sendMessage`'s own key governs the *whole* message turn (provider call plus any resulting `execute()`), its internal call to `execute()` deliberately does not forward a key — a second, redundant claim on the same logical operation would be wrong, not just unnecessary.

### What gets replayed

The stored `responseBody` is the same `{ httpStatus, response }` shape `execute()`/`sendMessage()` already return internally — not a re-derived or reconstructed object, and not the full HTTP envelope (`{ success, data }` / `{ success: false, error }`) the controller wraps it in. The controller applies its normal status/envelope branching to a replayed result exactly as it does to a fresh one, so replay reuses 100% of the existing response-shaping logic; nothing about the client-visible envelope needed to change for replay to work.

### Recovery-state gains `activeTurn`

`GET /assistant/conversations/:conversationId/recovery-state` (Phase 23.5, §15.3) already projects `activeClarification`/`pendingDraft`/`latestTerminalClarification` from already-persisted state. Phase 27 adds a fourth, symmetric field: `activeTurn` (`turnId`, `intent`, `startedAt`), populated from the same conversation's most recent `RUNNING` `AssistantTurn` row, queried via the existing `@@index([status, startedAt])`. This exists independently of whether the original request used an `Idempotency-Key` — a client that lost its Idempotency-Key (or never sent one) still has an authoritative way to discover "something is still running here" before deciding whether to resubmit, rather than guessing from a stale local view.

## Rejected alternatives

- **Extending draft-confirm's advisory-lock pattern verbatim to `/messages`/`/execute`.** Rejected: it would require holding a lock across an outbound LLM call with no fixed upper bound below the configured timeout, tying up a pooled database connection for the entire round trip. This is the central reason this phase's concurrency guard differs from the one it's explicitly asked to reuse — see Context above.
- **A second, parallel table (e.g. `AssistantRequestIdempotencyRecord`) instead of extending `AssistantIdempotencyRecord`.** Rejected for the same reason PD-016 rejected a second `ChannelInteractionOperation` table: one row shape, one place, no risk of two idempotency concepts drifting out of sync on what "one key → one outcome" means. The two use cases (draft-scoped, turn-scoped) are distinguished by which of `draftId`/`turnId` is populated, not by which table the row lives in.
- **Blocking (lock-and-wait) for an in-flight duplicate instead of returning `409 ASSISTANT_REQUEST_IN_PROGRESS`.** Rejected: blocking is exactly the pattern being avoided for `/messages` (see Context), and even for `/execute` — which has no external call — a blocked HTTP request ties up a server connection for an unbounded duration with no client-visible signal that anything unusual is happening. A fast, explicit "still processing" response lets the client decide (poll recovery-state, wait, or show a spinner) rather than silently stalling.
- **A TTL-based staleness fallback that reclaims a `RUNNING` row after a crash.** Rejected as out of scope for this phase, matching the same fail-closed posture `assistantOperationGuard`'s `ambiguous` outcome already accepts for Telegram: a `RUNNING` row after a process crash stays `RUNNING` indefinitely, with no dead-letter tooling or recovery worker. This mirrors an explicit instruction for this phase, not an oversight — automatic recovery is deferred to a later phase if it proves necessary in practice.
- **Migrating Web `/messages`/`/execute` onto the async channel inbox/outbox pipeline (§15.5) instead of adding synchronous dedup.** Rejected: that pipeline exists to make Telegram's fire-and-forget webhook model durable across a process restart; Web's request/response model has a live client connection waiting for a synchronous answer, which the inbox/outbox pipeline is not designed to serve. Out of scope for this phase by explicit instruction.

## Delivery guarantees

- **Key claim:** at-most-one `RUNNING` row per `(userId, key)`, enforced by the same database-unique constraint draft-confirm already relies on (`P2002` is the race-fallback, not merely a validation nicety — the insert itself is the serialization point).
- **Duplicate-turn prevention:** exactly one underlying `execute()`/`sendMessage()` invocation ever runs to completion per claimed key. A concurrent duplicate observes either `409 ASSISTANT_REQUEST_IN_PROGRESS` (claim lost the race while the first is still running) or an exact replay (claim lost the race after the first finished) — never a second turn.
- **Cross-operation safety:** a key claimed for `/execute` cannot be silently reused for `/messages`, or for draft confirm — mismatched `operation` returns `409 ASSISTANT_IDEMPOTENCY_CONFLICT` rather than a wrong-shaped replay.
- **Crash window:** a process crash between claiming a key and resolving it leaves that key's row `RUNNING` forever in this phase — identical in kind to `assistantOperationGuard`'s `ambiguous` outcome, and to draft-confirm's own unrecovered-`FAILED`-turn case. `activeTurn` on recovery-state gives the client a way to notice this independently of the key itself.
- **Backward compatibility:** omitting `Idempotency-Key` reproduces pre-Phase-27 behavior exactly — no claim, no replay, no new row. No existing caller (including every Telegram code path, which never supplies this parameter) changes behavior.

## Observability

Extends the existing `logEvent`/`AssistantLogEvent` taxonomy (already closed/allowlisted, no index signature) with the existing `idempotencyOutcome`/`idempotentReplay` fields — the same ones draft confirm already populates — now also populated by `assistant.message.received`/`assistant.message.completed` on both `/messages` and `/execute`. `ASSISTANT_REQUEST_IN_PROGRESS` is mapped into the existing `idempotency` `ErrorCategory` bucket alongside `ASSISTANT_IDEMPOTENCY_CONFLICT`. The idempotency key value itself is never logged, matching the existing redaction discipline verified for draft confirm.

## Consequences

**Positive:**

- Web now has the same "no channel can silently create two turns/drafts from one user action" guarantee Telegram already had, closing the gap the audit ranked highest-priority.
- `AssistantIdempotencyRecord` and its proven `(userId, key)` uniqueness are reused rather than duplicated; the only schema change is additive columns, all nullable or defaulted, so `draft-confirm`'s existing code path is unmodified and untested-behavior-preserving.
- `activeTurn` is a small, additive projection reusing an existing index — no new query pattern, no new table.

**Costs:**

- `AssistantIdempotencyRecord` now serves two distinct use cases (draft-scoped vs. turn-scoped) distinguished by which nullable FK is populated — a future reader must know both shapes exist in one table, an explicit trade-off against a second parallel table (see Rejected alternatives).
- A crashed process leaves a claimed key permanently `RUNNING` with no automatic recovery in this phase — an accepted, documented limitation, not a hidden gap.

## Related documents

- [PD-012 — Persistent Clarification Engine](012-persistent-clarification-engine.md)
- [PD-015 — Durable Channel Processing](015-durable-channel-processing.md)
- [Assistant Core Architecture § 15.3, § 15.5, § 15.7](../../architecture/assistant-core-architecture.md)
- `docs/api/assistant-conversations.md` (backend repository — canonical API-contract source for the `Idempotency-Key` header and `activeTurn` response shape)
- [Implementation Roadmap — Phase 27](../../development/implementation-roadmap.md)

## Non-goals / explicitly out of scope

- Dead-letter tooling or a recovery worker for a request stuck `RUNNING` after a crash.
- Migrating Web `/messages`/`/execute` to the async channel inbox/outbox pipeline.
- Channel badges or channel-identity UI in the frontend.
- Cross-worker or cross-request ordering guarantees beyond single-key deduplication.
- Any change to Telegram's existing `assistantOperationGuard` behavior — it is untouched by this phase.
