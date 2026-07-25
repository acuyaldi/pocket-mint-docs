# PD-012 — Persistent Clarification Engine and Deterministic Continuation (Phase 22.5)

**Status:** Accepted
**Date:** 2026-07-25
**Author:** Agent (Phase 22.5 implementation)

## Context

Phases 22.1–22.4 gave `transaction.create` deterministic, owner-scoped wallet/merchant/category resolution, but an `ambiguous` or missing result simply terminated the Assistant's execution: the User had to restate the entire request from scratch, and there was no server-side memory of which candidates had been shown. That is unsafe to fix casually, because:

- Provider output is untrusted. Whatever mechanism lets a User "pick option 2" must not let the provider (or a replayed/forged request) choose on the User's behalf, and must not re-run the provider to reinterpret a selection.
- User authority over a financial entity choice must be explicit and unambiguous — a candidate ID, array index, or free-text guess is either guessable or requires re-deriving intent from natural language, which is exactly the ambiguity this phase exists to remove.
- Finance mutations require a deterministic, replay-safe boundary: two callers racing to select the same clarification, or one caller retrying after a transient failure, must never produce two children, two drafts, or a double financial effect.
- An ephemeral (in-memory or conversation-JSON) clarification is unsafe: it does not survive a process restart, cannot be revalidated against current ownership at selection time, and has no natural expiry or concurrency story.
- Persisting a raw selection token would be unsafe: any read access to the row (backup, replica, admin query, log) would leak a live bearer credential for a pending financial choice.

## Decision

- **Persistent request/options.** `ClarificationRequest` and `ClarificationOption` are additive Prisma models (see [Assistant Core Architecture § 15.2](../../architecture/assistant-core-architecture.md#152-persistent-clarification-engine)) that survive across HTTP requests, scoped to `userId` and `conversationId`.
- **Digest-only token storage.** Each option's selection token is a random 32-byte value issued once, at creation. Only its SHA-256 digest is persisted; the raw token is never logged, snapshotted, or stored.
- **Scoped one-time selection.** Selection requires both the clarification ID (from the URL) and the raw token (from the body). A valid token atomically transitions `PENDING → CONSUMED`; any other outcome is a specific, deterministic terminal error.
- **Deterministic entity resolvers.** Selection revalidates the selected candidate and any already-resolved prerequisite (wallet, then merchant) against the authenticated User's current data before continuing — displayed option data is never treated as standing authorization.
- **Parent-child chain.** A selection that leaves the next entity in the wallet → merchant → category sequence ambiguous creates exactly one child clarification (`parentId` set), with its own fresh 15-minute expiry and the original immutable context (amount, type, date, description) carried forward unchanged.
- **Database-backed expiry.** Every clarification's 15-minute TTL is computed from database time, not the application clock, and checked before token matching. Expiry is represented as `STALE` with `terminalCode = 'expired'` — no separate `EXPIRED` enum value.
- **Transaction-aware continuation.** The atomic claim, candidate/prerequisite revalidation, child-or-draft creation, and Assistant lifecycle persistence run inside one shared interactive transaction, so a downstream failure rolls back the claim and leaves the token retryable.
- **Provider exclusion.** The provider is never invoked during continuation, and there is no provider-facing clarification-selection tool.
- **Pending draft, not direct `Transaction`.** A fully-resolved chain produces a `PENDING_CONFIRMATION` draft exactly like the existing Phase 21.4 flow; explicit confirmation remains the only path to the Transaction Service.
- **Strict HTTP endpoints.** `POST .../clarifications/:id/select` accepts exactly `{ optionToken: string }`; `POST .../clarifications/:id/cancel` accepts an empty body. Both reject any other shape before touching the database, and both treat ownership/conversation mismatch as indistinguishable from not-found.

## Alternatives considered

- **Rerun the provider with the User's selection.** Rejected: reintroduces provider authority over a financial entity choice and a second untrusted-output validation pass for no benefit over a deterministic token match.
- **Store raw tokens (plaintext or reversibly encrypted).** Rejected: any read path becomes a live bearer-credential leak. Digest-only storage means even a full row dump discloses nothing usable.
- **Select by candidate ID or array index.** Rejected: an ID is a stable, guessable, replayable reference across requests; an index depends on response ordering the client must not be trusted to have preserved correctly. A single-use random token tied to one specific option avoids both.
- **Keep clarification only in memory (conversation JSON or process-local state).** Rejected: does not survive a restart, cannot express a database-time expiry independent of the application clock, and has no transactional concurrency story for concurrent selection attempts.
- **Directly create a financial `Transaction` on selection.** Rejected: collapses the existing Draft → Preview → Explicit Confirmation → Commit boundary (§ 13 of the architecture doc) that every other write path uses; a clarification is still just resolving *what* the User meant, not authorizing the mutation.
- **Separate transactions for the claim and downstream continuation.** Rejected: a claim that commits before a failed child/draft creation would non-atomically consume a token with no compensating result — the User would see an error but the clarification would already be unusable. One shared transaction makes claim-and-continue succeed or roll back together.
- **Let merchant mapping silently authorize category.** Rejected: `MerchantMapping.categoryId` is an advisory suggestion (unchanged since Phase 19/22.3), not resolution authority; letting it silently populate the category step would make an unrelated one-directional suggestion feature into a hidden authorization path.

## Consequences

**Positive:**

- Deterministic behavior: every selection outcome is one of a small, enumerated set of terminal results, never inferred from prose.
- Replay safety: same-token retries, select/select races, select/cancel races, and expiry races all resolve to exactly one winner, verified against real PostgreSQL concurrency.
- Auditability: the persisted lifecycle (`PENDING/CONSUMED/CANCELLED/STALE` plus `terminalCode`) is a complete, inspectable record of what happened to a clarification.
- Financial safety: zero `Transaction` rows and zero balance changes are possible from clarification creation, selection, or cancellation alone.
- Channel-independent foundation: the HTTP contract has no dependency on the web frontend, a specific chat surface, or the Gemini provider adapter, so a future channel only needs its own authentication and transport.

**Costs:**

- Two additive migrations and a new aggregate add schema and query surface that must be maintained alongside the existing draft lifecycle.
- A lost or discarded raw token cannot be recovered or reissued by design — the User must restart the ambiguity from a fresh request.
- Continuation logic must thread a single transaction client through resolver revalidation, child creation, and lifecycle persistence, which is a stricter constraint on future changes to those code paths than a non-transactional call would be.
- Concurrency correctness (same-token races, select/cancel races, expiry races) requires its own dedicated PostgreSQL integration coverage beyond ordinary unit tests.
- Expired/cancelled/consumed rows are retained, not cleaned up — Phase 22.5 has no automatic pruning worker, matching the existing accepted draft-retention posture.

## Related documents

- [Assistant Core Architecture § 15.2 — Persistent Clarification Engine](../../architecture/assistant-core-architecture.md#152-persistent-clarification-engine)
- [Implementation Roadmap — Phase 22.5](../../development/implementation-roadmap.md)
- Backend API contract: `docs/api/assistant-conversations.md` (`pocket-mint-be` repository), § Clarification selection and cancellation
- Implementation plan: `docs/superpowers/plans/2026-07-24-persistent-clarification-engine-foundation.md` (`pocket-mint-be` repository, untracked working artifact)

## Explicitly out of scope (deferred to later phases or not planned)

- Frontend clarification UI.
- Telegram/Discord/WhatsApp or other external-channel integration.
- n8n or other external orchestration.
- External-channel identity mapping.
- Natural-language ("the second one") selection.
- Conversation-aware reference resolution (Phase 22.6, e.g. "wallet yang tadi").
- Raw-token recovery or reissue.
