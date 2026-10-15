# PD-024 — Draft Correction Completion (Phase 34)

**Status:** Accepted
**Date:** 2026-10-15
**Author:** Agent (Phase 34 implementation)

## Context

A read-only inspection at the start of this phase found that Assistant draft editing already exists end-to-end: `POST /drafts/:draftId/confirm` already accepts optional `amount`/`walletId`/`categoryId`/`description`/`date` overrides, already merges them with the pending draft and commits atomically, and already runs that merged write through the same ownership check, expiry check, status check, advisory lock, and idempotency-key protection as an unmodified confirm. The frontend already has complete edit UI for amount, wallet, category, date, and description, already holds edits locally until the user presses Confirm, and already hides Confirm/Cancel while editing. There is no separate edit endpoint, and this phase does not introduce one.

What inspection found instead were four real, narrow gaps:

1. The confirm-time override contract was implemented but undocumented — neither `pocket-mint-be`'s `docs/api/assistant-conversations.md` nor the architecture doc described it.
2. The backend override-merge behavior (per-field overrides, combinations, invalid wallet/category rejection, re-inference on description change, concurrent confirms, overrides against expired/cancelled/committed drafts) had no automated test coverage.
3. The frontend edit-to-confirm wiring had no automated coverage.
4. The draft preview DTOs (initial preparation, clarification-continuation, recovery-state projection) exposed only a human-readable `category` string and a `date` string — no raw `categoryId` and no unambiguous ISO business date — so the existing edit UI could not faithfully preselect the category control or the date control; `app/(app)/assistant/page.tsx`'s `handleStartEditDraft` was found seeding `categoryId: undefined` and `date: undefined` with comments stating the data wasn't available.

This phase's job is to close these four gaps — document, test, and supply the missing preview fields — not to build new editing capability that does not already exist.

## Decision

**Selected: document and test the override contract as it already exists; add `categoryId` and `transactionDate` additively to all three draft preview DTOs so the existing edit UI can prefill correctly.**

- **Edit and confirm remain one atomic operation.** There is no standalone edit endpoint, no `PATCH`/`PUT /drafts/:draftId`, and none is introduced. An override is only ever applied in the same transaction that commits the draft.
- **Overrides are local in the frontend until final confirmation.** This phase does not change that UX — it fixes the one place it was broken (category/date not preselecting) by extracting the seed logic into `buildDraftEditOverrides()`, reading the new raw `categoryId`/`transactionDate` preview fields instead of the display-only `category`/`date` strings.
- **Existing protections remain authoritative and unwidened.** Ownership, expiry, draft status, the per-draft advisory lock, and idempotency-key handling all still gate a confirm call carrying overrides exactly as they gate one without — overrides merge onto the draft only after every one of those checks passes, so they cannot be used to bypass any of them.
- **`categoryId` and ISO `transactionDate` are additive preview fields, not new data exposure.** Both draft-preview paths (initial preparation and clarification-continuation) share one `preview()` function in `financial-draft.service.ts`, fixed once; the recovery-state projection in `clarification.service.ts` is fixed separately since it builds its own DTO. All three are already owner-scoped by existing `{conversationId, userId}` / draft-ownership queries — the new fields sit inside that same boundary, never returned to anyone but the draft's own owner. `transactionDate` is computed via the existing `formatReportingDate` helper against `reportingConfig.timezone`; the existing `date` field in the recovery-state projection is left untouched for backward compatibility even though it was found to be a naive UTC slice that is off by one calendar day in `Asia/Jakarta` — the new field is correct, the old one is preserved as-is rather than silently changed under existing consumers.
- **No schema migration, no persisted edit state, no revision/audit history.** Nothing about a draft's shape in Postgres changes; "overrides" remain request-scoped values in a single confirm call, never written to any table of their own.
- **Frontend scope stays a prefill/wiring fix, not a new form.** No full Add Transaction form was built inside the Assistant surface, and no Telegram editing UI was added — both were explicit non-goals and remain so.
- **Why this is a usability completion, not a new feature:** every piece of user-facing capability described above (edit fields, hold-until-confirm, atomic commit, revalidation, re-inference) was already live before this phase. The product gap was that the capture flow's own preview data couldn't drive its own edit UI correctly, and that the contract backing it was invisible to anyone reading the docs or the test suite. Closing that gap is what this phase does — it does not add the ability to edit a draft, which already existed.

## Rejected alternatives

- **Add a dedicated `PATCH /drafts/:draftId` edit endpoint.** Rejected: the existing confirm-time override contract already lets a client submit a corrected set of fields in the same call that commits them, with no intermediate persisted state to reconcile, expire, or reconcile against a second confirm. A separate edit endpoint would add a new state transition and a new set of ownership/expiry edge cases for no behavior the product does not already have.
- **Fix the recovery-state projection's naive-UTC-slice `date` field in place.** Rejected: that field is an existing, already-depended-on wire value; changing its computation silently would be a breaking change to any consumer relying on its current (if occasionally off-by-one) value. Adding the correctly-computed `transactionDate` alongside it, and documenting the discrepancy, achieves the same user-facing correctness (faithful edit prefill) without an undocumented behavior change to a field already in use.
- **Build a persisted draft-edit/revision history so a user could see what was changed before confirming.** Rejected as scope creep — never requested, and inconsistent with the deliberately ephemeral, single-shot nature of a financial draft (expires on its own short timer, is never an audit record).
- **Extend editing into the Telegram channel.** Rejected: out of scope for this phase and not requested; Telegram's interactive workflows (PD-016) do not currently expose field-level editing, and adding it is an independent product decision with its own UX design, not a byproduct of fixing Web preview data.

## Consequences

**Positive:**

- The confirm-time override contract is now documented in both `pocket-mint-be`'s `docs/api/assistant-conversations.md` and `Assistant Core Architecture §15.12`, closing the gap between what the system does and what a reader can learn about it.
- New backend integration tests (per-field overrides, combinations, invalid wallet/category rejection with no mutation, description-triggered re-inference, five-way concurrent confirm, overrides against expired/cancelled/committed drafts, preview/recovery projection contents) pin the existing override-merge behavior against a real database, so a future change to `confirm()`'s merge logic will fail a test instead of silently changing behavior.
- The frontend edit UI now faithfully preselects category and date on entering edit mode, and a Storybook interaction test plus unit tests pin that an edit to one field does not clear the others — closing the one real user-facing bug this phase found.
- Zero schema or endpoint surface added — the entire change is additive preview fields plus tests and docs.

**Costs:**

- None beyond the diff itself. The recovery-state projection now carries two date fields (`date`, kept for compatibility; `transactionDate`, newly correct) until a future phase decides it is safe to retire the old one.

## Related documents

- [Assistant Core Architecture §15.12](../../architecture/assistant-core-architecture.md#1512-draft-correction-completion--preview-data--confirm-time-overrides-phase-34)
- [Implementation Roadmap — Phase 34](../../development/implementation-roadmap.md)
- `pocket-mint-be`'s `docs/api/assistant-conversations.md` — draft-preview and confirm sections, expanded by this phase
- [PD-017 — Assistant Request-Level Idempotency & Turn Recovery](017-assistant-request-level-idempotency.md) — the idempotency protection confirmed unwidened by overrides
- [PD-023 — Conversation History Retention & Deletion Semantics](023-conversation-history-retention-deletion-semantics.md) — most recent prior Assistant-lifecycle phase

## Non-goals / explicitly out of scope

- No new draft CRUD endpoint, no `PATCH`/`PUT /drafts/:draftId`.
- No persisted draft-edit state, no draft revision/audit-history model.
- No multi-draft conversation support.
- No background draft-expiry worker.
- No editing of already-confirmed transactions, no bypass of confirmation.
- No change to duplicate-transaction behavior.
- No new Telegram editing UI or financial logic.
- No financial-advice or insight features.
- No full Add Transaction form built inside the Assistant surface — the existing field-level edit UI is fixed, not replaced.

## Follow-up Tasks

- If the recovery-state projection's old naive-UTC-slice `date` field is ever confirmed to have no remaining consumer, it can be retired in favor of `transactionDate` alone — that is a follow-up cleanup, not part of this phase.
- If Telegram-channel draft editing becomes a real product need, it should be evaluated as its own decision with its own UX review, consistent with how Phase 30–33's channel-specific work was each scoped independently.
