# Backend Phase 1 Execution Plan — Financial Core

> Source: [Backend Engineering Backlog](./backend-backlog.md) Phase 1 tasks (P1-01 … P1-09), sequenced per the [Implementation Roadmap](./implementation-roadmap.md).
> Audience: implementation agents (Claude Code, Codex) and reviewers.
> This plan defines execution order, file surface, verification, and rollback for each Phase 1 task. It contains no implementation, no time estimates, and no assignments. It adds no work beyond the backlog.

---

## Ground Rules for Every Batch

- **Entry gate (applies before Batch 1):**
  - The verification suite must be green. The audit recorded 3 failing JWT success-path tests (backlog task P2-04, Immediate Sprint order 1). Do not start Phase 1 batches against a red suite — Phase 1 correctness claims depend on it.
  - P0-01 (PD-001 guidance/test-description sync) must be complete before Batch 2.
  - P0-02 (documented contracts for summary, exact money, idempotency, correction/deletion) must be complete before Batches 3, 4, 5, and 7 respectively for their governed contract.
- **Working-backend invariant:** every batch merges with `npx tsc --noEmit` clean, `npx vitest run` fully green, `npx prisma validate` passing, and no behavioral regression outside the batch's declared contract change.
- **Preserved behavior (from `COMPLIANT` audit findings):** JWT-only authentication, User-scoped route/service filters, Express-free services, atomic `$transaction` mutation paths, the shared Decimal domain helpers in `src/domain/transactionBalance.ts` and `src/domain/installment.ts`, redacting logger, and central error forwarding. A batch that weakens any of these fails review regardless of its own goal.
- **Migrations:** additive and reviewed; replay each new migration on disposable PostgreSQL before merge. Never edit committed migration history.
- **Draft PD boundary:** no batch implements PD-003 … PD-008 behavior. Credit Limit enforcement, lifecycle, fees, reporting surface, Transfer-model retirement stay out of Phase 1.

Standard verification gate referenced below as **GATE**:

```bash
npx tsc --noEmit
npx vitest run
npx prisma validate
git diff --check
```

---

## Batch 1 — Ownership Integrity Hotfix

Single task. No documentation dependency. Mergeable immediately after the entry gate. Smallest possible diff; ships first because it closes a live cross-User data exposure.

### P1-05 — Category ownership validation on Transaction update

- **Task ID:** P1-05
- **Goal:** Transaction update validates `{ id: categoryId, userId }` for a changed `categoryId` before entering the mutation transaction, matching the invariant already enforced on create.
- **Why it exists:** Audit BE-AUTHZ-002 / BE-TX-002 (Critical): update writes a requested Category without ownership validation, so a caller who learns another User's Category ID can attach it and read cross-User Category metadata through relation serialization. BE-DB-002 confirms no database constraint catches this.
- **Dependencies:** None (entry gate only).
- **Files likely affected:**
  - `pocket-mint-be/src/services/transaction.service.ts` (update path; mirror the create-path ownership check)
  - `pocket-mint-be/src/services/transaction.errors.ts` (reuse or extend the existing typed category-ownership error)
  - `pocket-mint-be/test/transactionService.test.ts` (regression: cross-User Category assignment rejected on update)
- **Verification:** GATE. New regression test proves update with another User's `categoryId` returns the typed ownership error and performs no write; existing create-path tests unchanged and green.
- **Rollback considerations:** Pure service-layer validation addition; no schema or contract change. Single revert commit restores prior behavior. Reverting reopens the vulnerability — treat revert as a security regression requiring sign-off.
- **Definition of Done:** Update enforces the same ownership invariant as create for every changed relationship identifier; regression test exists and passes; no other behavior changed; GATE green.

---

## Batch 2 — PD-001 Net Worth

Single task. Depends on P0-01 (guidance and test descriptions already synchronized to PD-001). This batch intentionally changes the documented meaning of `netWorth` in responses — that is its contract change, coordinated with the frontend as a consumer note, not hidden.

### P1-01 — Implement PD-001 Net Worth calculation

- **Task ID:** P1-01
- **Goal:** `calculateNetWorth` and every consumer return Net Worth = Total Assets − Total Outstanding Debt at one Reporting Cutoff, with `totalAssets` and `totalOutstandingDebt` preserved as separate labeled components.
- **Why it exists:** Audit BE-WAL-002 / BE-FIN-001 (Critical, top-ranked risk): the shared helper sets `netWorth = totalAset`, so every Financial Summary overstates Net Worth whenever tracked debt exists; tests currently assert the contradiction as intended.
- **Dependencies:** P0-01.
- **Files likely affected:**
  - `pocket-mint-be/src/utils/financial.ts` (canonical calculation)
  - `pocket-mint-be/src/services/wallet-query.service.ts`, `pocket-mint-be/src/services/wallet-query.types.ts`
  - `pocket-mint-be/src/services/dashboard-query.service.ts`, `pocket-mint-be/src/services/dashboard-query.types.ts`
  - `pocket-mint-be/src/controllers/account.controller.ts`, `pocket-mint-be/src/controllers/dashboard.controller.ts` (serialization of the corrected fields)
  - Tests asserting the old semantics: `pocket-mint-be/test/dashboardQueryService.test.ts`, `pocket-mint-be/test/reportingEffect.test.ts`, `pocket-mint-be/test/walletQueryService.test.ts`, plus any controller-boundary test pinning the old value
- **Verification:** GATE. Tests assert assets-minus-debt (including a negative Net Worth case and a zero-debt case where Net Worth equals Total Assets); no endpoint returns assets-only as `netWorth`; `totalAssets` still present and correct.
- **Rollback considerations:** Calculation-and-tests change in one commit; revert restores prior (incorrect) semantics cleanly. Consumers reading `netWorth` see a value change on merge — release note must state the correction; rollback needs the same coordination in reverse.
- **Definition of Done:** PD-001 semantics live in the shared helper and all consumers; components remain separately reported; all updated tests assert the approved calculation; GATE green.

---

## Batch 3 — Exact Money at the Input Boundary

Single task. Depends on the exact-money-transport contract from P0-02. Merges independently: input parsing changes only; response shapes untouched (that is Batch 4).

### P1-02 — Decimal parsing of authoritative monetary inputs

- **Task ID:** P1-02
- **Goal:** All authoritative monetary inputs (balance, Credit Limit, Interest, Administrative Fee, transaction amounts) parse directly into Decimal at the service boundary; malformed, empty-string, and non-finite values are rejected with typed operational errors before validation, calculation, or persistence.
- **Why it exists:** Audit BE-MONEY-001 (Critical) and BE-VAL-001 (High): wallet and transaction commands coerce money through JavaScript `Number` before constructing Decimal, risking silent precision loss on large integers or long decimals and unexpected 500s on malformed input.
- **Dependencies:** P0-02 (documented accepted-input contract: which string/number forms are valid).
- **Files likely affected:**
  - New shared exact-money parser (single helper; placed per backend architecture docs, e.g. `pocket-mint-be/src/domain/` or `pocket-mint-be/src/utils/`)
  - `pocket-mint-be/src/services/wallet.service.ts` (create/update coercions)
  - `pocket-mint-be/src/services/transaction.service.ts` (create/update `numAmount` path)
  - `pocket-mint-be/src/services/wallet.types.ts`, `pocket-mint-be/src/services/transaction.types.ts` (input types)
  - `pocket-mint-be/src/services/wallet.errors.ts`, `pocket-mint-be/src/services/transaction.errors.ts` (typed validation errors)
  - `pocket-mint-be/src/controllers/account.controller.ts`, `pocket-mint-be/src/controllers/transaction.controller.ts` (body allowlists pass raw values through to the parser)
  - Tests: `pocket-mint-be/test/walletService.test.ts`, `pocket-mint-be/test/walletUpdate.test.ts`, `pocket-mint-be/test/transactionService.test.ts`, controller-boundary tests for deterministic 4xx on malformed bodies
- **Verification:** GATE. Regression tests cover: long-decimal and large-integer inputs preserved exactly to persistence; `NaN`/`Infinity`/empty-string/object inputs rejected with typed errors (never 500, never silent zero); accepted values produce identical stored results as before for previously-valid inputs.
- **Rollback considerations:** No schema change. Accepted-input surface may narrow (previously-coerced garbage now rejected) — that narrowing is the documented intent; revert is a single commit but reintroduces precision loss. Keep parser adoption in one batch so a revert cannot leave paths half-migrated.
- **Definition of Done:** No `Number` coercion precedes Decimal construction on any Wallet or Transaction command path; one shared parser owns acceptance rules; rejection is deterministic and typed; GATE green.

---

## Batch 4 — Canonical Response Contracts

Two related tasks: what summaries mean (P1-03) and how money travels in responses (P1-04). Both are additive response-contract work over the same serializers; batch merges as one reviewable contract change. P1-04 may trail P1-03 within the batch but both must respect backward compatibility.

### P1-03 — Financial Summary contract with Reporting Cutoff and provenance

- **Task ID:** P1-03
- **Goal:** Dashboard and wallet-mutation summary responses carry exact components, one shared Reporting Cutoff, calculation identity, and provenance, per the contract documented in P0-02.
- **Why it exists:** Audit BE-FIN-003 (High): summaries return bare numeric triples; consumers cannot prove components share a cutoff or explain a headline value. Roadmap Phase 1 requires provenance and Reporting Cutoff consistency before Reports, Automation, and AI consume summaries.
- **Dependencies:** P0-02, P1-01 (contract wraps the corrected calculation, not the wrong one).
- **Files likely affected:**
  - `pocket-mint-be/src/services/dashboard-query.service.ts`, `pocket-mint-be/src/services/dashboard-query.types.ts`
  - `pocket-mint-be/src/services/wallet-query.service.ts`, `pocket-mint-be/src/services/wallet-query.types.ts`
  - `pocket-mint-be/src/services/wallet.service.ts` (mutation-response snapshot totals)
  - `pocket-mint-be/src/controllers/dashboard.controller.ts`, `pocket-mint-be/src/controllers/account.controller.ts`
  - Tests: `pocket-mint-be/test/dashboardQueryService.test.ts`, `pocket-mint-be/test/dashboardControllerBoundary.test.ts`, `pocket-mint-be/test/walletQueryControllerBoundary.test.ts`, `pocket-mint-be/test/walletControllerBoundary.test.ts`
  - Backend contract documentation (per P0-02 location, e.g. `pocket-mint-be/docs/`)
- **Verification:** GATE. Tests assert: all components in one summary response share one Reporting Cutoff value; response identifies its calculation and time boundary; existing numeric fields remain present and unchanged in meaning (additive change).
- **Rollback considerations:** Additive fields only — existing consumers unaffected; revert removes the new fields without breaking prior contract. Do not let any consumer become load-bearing on the new fields until the batch has survived a release cycle.
- **Definition of Done:** Documented summary contract implemented additively across Dashboard and wallet snapshots; cutoff shared and asserted; provenance present; GATE green.

### P1-04 — Backward-compatible exact-money response representation

- **Task ID:** P1-04
- **Goal:** Responses expose an exact representation of authoritative money (per the P0-02 transport contract, e.g. decimal strings) alongside the existing JSON-number fields, which are documented as lossy display conveniences.
- **Why it exists:** Audit BE-MONEY-002 (High): controllers serialize Decimal via `parseFloat`/`Number`; consumers have no lossless transport for values outside JavaScript's exact range.
- **Dependencies:** P0-02. Independent of P1-03 in content, batched together because both touch the same serializers.
- **Files likely affected:**
  - `pocket-mint-be/src/controllers/account.controller.ts`, `pocket-mint-be/src/controllers/transaction.controller.ts`, `pocket-mint-be/src/controllers/dashboard.controller.ts`, `pocket-mint-be/src/controllers/installment.controller.ts` (serializers)
  - `pocket-mint-be/src/utils/response.ts` (if shared serialization helpers live there)
  - Controller-boundary tests for each surface
  - Backend contract documentation (compatibility status of numeric fields)
- **Verification:** GATE. Boundary tests assert exact representation matches persisted Decimal for boundary-magnitude values; existing numeric fields byte-identical to pre-change output for representative payloads.
- **Rollback considerations:** Strictly additive; revert removes exact fields. Numeric fields never change in this task — if a diff shows a numeric field changing, the task has exceeded its scope.
- **Definition of Done:** Exact representation available on every authoritative money field in responses; numeric fields unchanged and documented as compatibility conveniences; GATE green.

---

## Batch 5 — Mutation Idempotency

Single task, schema-bearing. Depends on the idempotency contract from P0-02. Largest batch; keep it isolated so its migration and middleware review are not entangled with calculation changes.

### P1-08 — Persistent request idempotency for financial mutations

- **Task ID:** P1-08
- **Goal:** Consequential Wallet, Transaction, Transfer, and Installment mutations accept a stable idempotency key; a repeated accepted request produces exactly one financial effect and a deterministic response; deduplication evidence is durably persisted.
- **Why it exists:** Audit BE-IDEM-001 (Critical): no route accepts an idempotency key and no table records mutation deduplication, so client retries after timeouts duplicate Income, Expense, Transfer, or Installment effects. Prerequisite for Phase 5 lifecycle and Phase 7 Automation.
- **Dependencies:** P0-02 (key transport, scope, replay-response, and retention rules). Applies on top of Batch 3's exact parsing (same command paths).
- **Files likely affected:**
  - `pocket-mint-be/prisma/schema.prisma` (idempotency evidence model) + new migration under `pocket-mint-be/prisma/migrations/`
  - New middleware or service component for key handling (per contract; e.g. `pocket-mint-be/src/middleware/`)
  - `pocket-mint-be/src/routes/walletRoutes.ts`, `pocket-mint-be/src/routes/transaction.routes.ts`
  - `pocket-mint-be/src/services/wallet.service.ts`, `pocket-mint-be/src/services/transaction.service.ts` (dedup check inside the same `$transaction` boundary as the effect)
  - `pocket-mint-be/src/controllers/account.controller.ts`, `pocket-mint-be/src/controllers/transaction.controller.ts`
  - New test file for idempotency behavior; extensions to `pocket-mint-be/test/transactionService.test.ts`, `pocket-mint-be/test/walletService.test.ts`
- **Verification:** GATE plus migration replay on disposable PostgreSQL. Tests prove: same key + same intent = one persisted effect and a deterministic replay response; same key raced concurrently = exactly one effect; requests without a key behave per the documented contract (unchanged unless the contract says otherwise). Existing atomicity tests (BE-ATOM-001 surface) stay green.
- **Rollback considerations:** Schema-bearing — migration must be additive (new table only) so rollback is "stop reading the table," not a destructive down-migration. If key handling is middleware-gated, a revert of application code leaves the table inert and harmless. Do not enforce mandatory keys in this batch if the contract allows a compatibility window; enforcement toggles are easier to roll back than route rejections.
- **Definition of Done:** Documented contract implemented on all consequential mutation routes; durable evidence persisted; retry and race tests green; migration replayed clean on disposable PostgreSQL; GATE green.

---

## Batch 6 — Validation Integrity

Two tasks: enforcement (P1-06) then structural assessment (P1-07). P1-07 produces a document, and only optionally a migration — the batch is mergeable with the assessment alone.

### P1-06 — Complete approved cross-field validation

- **Task ID:** P1-06
- **Goal:** Business Services enforce the approved cross-field invariants: wallet type changes revalidate required Debt Wallet fields; Category type matches Income/Expense on create and update; related-resource consistency is enforced on every mutation path. PD-005-dependent Credit Limit rules remain explicitly deferred.
- **Why it exists:** Audit BE-VAL-002 (High): structurally valid rows can contradict business meaning — a wallet retyped without Debt fields, an Expense filed under an Income Category.
- **Dependencies:** P1-05 (ownership checks are the base the type checks extend); benefits from Batch 3's typed-error surface.
- **Files likely affected:**
  - `pocket-mint-be/src/services/wallet.service.ts` (type-change revalidation)
  - `pocket-mint-be/src/services/transaction.service.ts` (Category type ↔ transaction type on create and update)
  - `pocket-mint-be/src/services/wallet.errors.ts`, `pocket-mint-be/src/services/transaction.errors.ts`
  - Tests: `pocket-mint-be/test/walletUpdate.test.ts`, `pocket-mint-be/test/walletService.test.ts`, `pocket-mint-be/test/transactionService.test.ts`
- **Verification:** GATE. One test per invariant (accept and reject cases); deferred PD-005 rules named in test comments or docs as deferred, not silently absent.
- **Rollback considerations:** Service-layer validation only; single revert. Note: rows violating the new invariants may already exist — this task rejects new writes, it does not migrate old data; any data cleanup is P1-09/P1-07 territory or later, so rollback has no data implications.
- **Definition of Done:** Listed invariants enforced with typed errors and per-invariant tests; PD-005 rules documented as blocked; GATE green.

### P1-07 — Structural same-User integrity assessment

- **Task ID:** P1-07
- **Goal:** Written assessment of structural reinforcement options (e.g., composite foreign keys on `(id, userId)`) for same-User relationship integrity, with a recommendation; any adopted constraint ships as a deliberate reviewed migration.
- **Why it exists:** Audit BE-DB-002 (High): foreign keys check row existence, not `userId` equality — a missed service check can persist cross-User links. Application fixes (P1-05, P1-06) close known gaps; this task decides whether the database should backstop them.
- **Dependencies:** P1-05, P1-06 (assess after application-level gaps are closed).
- **Files likely affected:**
  - New assessment document (per P0-02 documentation location, e.g. `pocket-mint-be/docs/`)
  - Only if adopted: `pocket-mint-be/prisma/schema.prisma` + new migration
- **Verification:** Assessment reviewed and a decision recorded. If a constraint is adopted: `npx prisma validate`, migration replay on disposable PostgreSQL, existing-data violation scan documented before apply, GATE green.
- **Rollback considerations:** Document-only outcome has no rollback surface. A constraint migration must be preceded by a violation scan of existing data; rollback of a constraint is a follow-up migration dropping it — plan both directions in the assessment.
- **Definition of Done:** Assessment exists with explicit adopt/defer decision; if adopted, constraint merged via reviewed, replayed migration; GATE green.

---

## Batch 7 — Historical Record Integrity

Single task, policy-gated and schema-bearing. Last in Phase 1 because its documentation half (correction/deletion policy from P0-02) has the longest decision path, and its runtime change touches deletion semantics other batches do not.

### P1-09 — Deletion cannot destroy or orphan Ledger facts

- **Task ID:** P1-09
- **Goal:** With correction/archival/deletion policy documented first: `DELETE /wallets/:id?force=true` no longer cascades away Transactions and Installments contrary to policy, and destination-wallet deletion no longer produces self-undescribing destination-less Transfers via `SetNull`.
- **Why it exists:** Audit BE-WAL-004 (High) and BE-DB-004 (High): current force-delete plus `onDelete: Cascade`/`SetNull` rules can erase Financial Events and Installments or convert a valid Transfer into an unresolved legacy state, making past balances and Reports irreproducible.
- **Dependencies:** P0-02 (policy documented before any runtime change — the audit explicitly orders documentation first).
- **Files likely affected:**
  - Policy documentation in `pocket-mint-docs` (from P0-02) and backend contract docs
  - `pocket-mint-be/src/services/wallet.service.ts` (force-delete path)
  - `pocket-mint-be/src/services/wallet.errors.ts` (typed rejection or alternative-outcome errors)
  - `pocket-mint-be/prisma/schema.prisma` (`onDelete` referential actions) + new migration
  - `pocket-mint-be/src/controllers/account.controller.ts` (deletion response semantics if policy changes them)
  - Tests: `pocket-mint-be/test/walletDeletion.test.ts`, plus transfer-destination cases in `pocket-mint-be/test/transactionService.test.ts`
- **Verification:** GATE plus migration replay on disposable PostgreSQL. Tests prove: deletion with dependent Financial Events follows the documented policy (reject, archive, or whatever the approved policy states) and never silently erases Ledger facts; destination-wallet deletion cannot leave an unexplainable Transfer state.
- **Rollback considerations:** Highest-risk rollback in Phase 1: referential-action changes affect the database, not just code. Sequence the migration so it is behavior-tightening (removing destructive automatic actions), which is safe to hold even if application code reverts. If the policy outcome changes API behavior (e.g., `force=true` now rejected), document the client-facing change; rolling that back re-enables history destruction and needs the same sign-off as a security regression.
- **Definition of Done:** Policy documented before runtime change; force-delete and referential actions match the policy; no path silently destroys or orphans Financial Events; migration replayed clean; GATE green.

---

## Batch Summary and Merge Order

| Batch | Tasks | Depends on (external) | Schema change | Contract change |
|---|---|---|---|---|
| 1 — Ownership Integrity Hotfix | P1-05 | Entry gate | No | No (bug fix) |
| 2 — PD-001 Net Worth | P1-01 | P0-01 | No | Yes — `netWorth` value corrected |
| 3 — Exact Money Input | P1-02 | P0-02 | No | Input acceptance narrows to documented forms |
| 4 — Canonical Response Contracts | P1-03, P1-04 | P0-02, Batch 2 | No | Additive fields only |
| 5 — Mutation Idempotency | P1-08 | P0-02, Batch 3 | Yes (additive table) | Additive key handling |
| 6 — Validation Integrity | P1-06, P1-07 | Batch 1 | Only if P1-07 adopts | Rejections added per documented invariants |
| 7 — Historical Record Integrity | P1-09 | P0-02 (policy) | Yes (referential actions) | Deletion semantics per approved policy |

Each batch is independently mergeable in this order; later batches assume earlier ones only where the table says so. Phase 1 is complete when all seven batches are merged, the full verification gate (backlog XC-05 shape) has run at the phase boundary, and no Draft-PD behavior was implemented along the way.
