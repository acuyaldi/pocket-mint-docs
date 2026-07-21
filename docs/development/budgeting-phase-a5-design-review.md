# Budgeting Phase A.5 — UX and API Contract Review

> Audit date: 2026-07-21
> Scope: `pocket-mint-be`, `pocket-mint-fe`, `pocket-mint-docs`
> Method: read-only static inspection of the implemented Phase A backend, existing frontend conventions, and current documentation. **No runtime code was written or changed.** No Prisma schema/migration, route, controller, React component, or dependency was added.
> Related: [PD-009](../product/decisions/009-budgeting-scope.md) (Approved, Phase A only), [Budgeting Readiness Audit](./budgeting-readiness-audit.md), [Budgeting Calculation Specification](./budgeting-calculation-spec.md), [Budgeting API Contract](./budgeting-api-contract.md), [Budgeting User Journeys](./budgeting-user-journeys.md), [Screen Specification — Budgets](../product/screen-spec.md#budgets)

---

## 1. Executive Summary

Phase A shipped a correct, well-tested domain foundation: a `Budget` Prisma model, a pure calculation function, and a read-only query service — all matching PD-009 and the calculation spec exactly, with no drift found. Nothing in Phase A needs to change.

This iteration (Phase A.5) does not touch that code. It closes the gap between "the domain model is correct" and "a controller/route/UI implementer has an unambiguous spec to build against," by:

- Finalizing five previously-undecided UX policies (category reassignment, archive vs. delete, dashboard integration timing, period semantics, detail-screen justification).
- Producing an implementation-ready API contract (`budgeting-api-contract.md`) that reuses existing Pocket Mint conventions exactly (response envelope, `BudgetError`, auth helpers).
- Producing 15 implementation-ready user journeys.
- Updating `screen-spec.md`'s Budgets/Budget Detail/Create Budget sections from Draft-era language to Approved, adding a missing Edit Budget section, and removing a Delete action the finalized policy no longer includes.
- Confirming no domain gap requires a schema change.

**Budgeting remains in progress, not released.** No routes, controllers, frontend code, or dashboard integration exist after this iteration — only documentation changed.

---

## 2. Implemented-Domain Findings

Verified directly against source, not solely against the prior audit:

| Area | File | Behavior |
|---|---|---|
| Prisma model | `pocket-mint-be/prisma/schema.prisma:352-368` | `Budget { id, userId, categoryId, amount Decimal(15,2), isArchived, createdAt, updatedAt }`, `@@unique([userId, categoryId])`, `@@index([userId, isArchived])`, both FKs `onDelete: Cascade`. |
| Migration | `pocket-mint-be/prisma/migrations/20260721012908_add_budget/migration.sql` | Single additive migration, matches the schema exactly, no backfill. |
| Calculation | `pocket-mint-be/src/domain/budget.ts` | `computeBudgetUsage(amount, spent, isArchived)` — pure, `Prisma.Decimal` throughout, no floating point. Evaluation order `ARCHIVED → EXCEEDED(>100) → REACHED(==100) → APPROACHING(>=75) → HEALTHY`. `remaining` never clamped. Defensive `amount<=0` branch documented as unreachable (write-time validation enforces `amount > 0`). |
| Query service | `pocket-mint-be/src/services/budget-query.service.ts` | `getBudgetUsage` (single, ownership-scoped `findFirst`, 404 via `BudgetError`) and `listActiveBudgetUsage` (grouped `transaction.groupBy(['categoryId'])`, one query regardless of budget count, empty list short-circuits before any transaction query). Period resolved via `getReportingMonthRange`, half-open, timezone-aware. |
| Types | `pocket-mint-be/src/services/budget-query.types.ts` | `BudgetQueryPrismaClient = Pick<PrismaClient, 'budget'\|'transaction'>`; explicit comment confirms no controller/route consumes this yet. |
| Errors | `pocket-mint-be/src/services/budget.errors.ts` | `BudgetError` mirrors `savingGoal.errors.ts`/`installment.errors.ts` exactly; structurally recognized by `forwardError` with no registration step. |
| Tests | `pocket-mint-be/test/budget.test.ts`, `test/budgetQueryService.test.ts` | Table-driven boundary tests at every status transition (74.9999/75/99.9999/100/100.0001/150%), exact-Decimal precision assertions, archived-overrides-all assertion, N+1 assertion (`groupBy` called once for 3 budgets/3 categories), cross-user lookup structurally scoped, unexpected DB errors propagate untyped. |
| Routes/controllers | — | **Confirmed absent.** `src/controllers/`, `src/routes/`, `src/models/` contain no `budget*` file; `src/routes/index.ts` does not wire one. Matches PD-009's explicit non-authorization of Phase B. |

No mismatch was found between PD-009, the calculation spec, the roadmap, and the actual Phase A implementation — all four agree on the five-state model, the uniqueness rule, and the Decimal/timezone handling. The only documentation gap found was in `screen-spec.md` itself, which still used Draft-era "Provisional"/PD-009-Draft language after PD-009 was approved for Phase A — corrected in this iteration (§5).

---

## 3. Domain/Documentation Mismatches

None found in the domain logic. One pre-existing, unrelated drift was noted for awareness, not fixed (out of scope): the frontend nav (`app-sidebar.tsx`/`bottom-nav.tsx`) already has 6 items where `skills/design.md` documents 5 canonical labels — a Budgets nav entry will be the 7th, following the code's existing (not the stale doc's) pattern.

---

## 4. Final MVP UX Decisions

Summarized here; full screen-by-screen detail is in `screen-spec.md` and the rationale for each contested decision is in §10–§11 below.

- **Overview:** current-month-only, active list by default, an Archived toggle on the same screen (not a separate screen), no sort/search/filter beyond that (the category set is small — 7 seeded expense categories — so these add no value yet).
- **Create:** modal (matches `CreateWalletModal`/`CreateSavingGoalModal`), not a page or drawer — no drawer component exists in the codebase to justify introducing one.
- **Edit:** amount only; category is read-only. New "Edit Budget" screen-spec section added (previously only implied by a Create Budget footnote).
- **Detail:** kept, justified specifically by the contributing-transaction list (not by CRUD convention).
- **Archive/Delete:** archive/restore only; delete removed from MVP entirely.
- **Dashboard integration:** deferred past the first UI release.
- **Status presentation:** backend-owned `status`, capped progress-bar width but true `percentUsed` in text/`aria-valuenow`.
- **Period:** current-month-only, no historical browsing, no schema change.

---

## 5. Final Screen Specification

Updated in place: [`docs/product/screen-spec.md`](../product/screen-spec.md#budgets) (Budgets, Budget Detail, Create Budget sections revised; Edit Budget section added). Not duplicated here — screen-spec.md is the single source per the "prefer updating existing screen-spec.md" instruction.

---

## 6. Final User Journeys

See [`budgeting-user-journeys.md`](./budgeting-user-journeys.md) — 15 journeys, entry point through accessibility considerations, no AI/forecasting/notification content.

---

## 7. Final API Endpoint Set

See [`budgeting-api-contract.md`](./budgeting-api-contract.md). Summary:

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/budgets?status=active\|archived` | List with usage |
| GET | `/api/v1/budgets/:id` | One Budget with usage |
| POST | `/api/v1/budgets` | Create |
| PATCH | `/api/v1/budgets/:id` | Update amount only |
| POST | `/api/v1/budgets/:id/archive` | Archive |
| POST | `/api/v1/budgets/:id/restore` | Restore |

No `DELETE`, no separate archived-list endpoint, no eligible-categories endpoint — each was evaluated and rejected as unnecessary given the existing `GET /categories` and the two list calls already required (see the contract doc's per-endpoint rationale).

---

## 8. Request/Response DTO Contracts

One `BudgetDto` (list, detail, and every mutation response) — see contract doc for the full shape, field-by-field serialization rules, and the explicit reasoning for not forking into Summary/Detail/Mutation variants. `amount`/`spent`/`remaining` serialize as JS `number` (matching the existing `savingGoal` convention); the precision ceiling this implies is called out explicitly as a pre-existing, shared risk across every money-serializing endpoint, not something Budgeting should fix in isolation. `percentUsed` serializes unrounded; `status` is always backend-computed.

---

## 9. Validation and Error Contract

Full table in the contract doc. Highlights: `422 CATEGORY_NOT_EDITABLE` on any `PATCH` that includes `categoryId` (explicit rejection, not silent ignore); `409 BUDGET_ALREADY_EXISTS` covers both active and archived duplicates (Decision L); `404` never distinguishes "not found" from "not yours," for both Budget and Category lookups; `409 ALREADY_ARCHIVED`/`ALREADY_ACTIVE` on redundant archive/restore calls rather than silent no-ops.

---

## 10. Archive/Delete/Category Reassignment Decisions

**Category reassignment: forbidden.** `PATCH /budgets/:id` rejects any `categoryId` in the body. Rationale: uniqueness is `(userId, categoryId)` (Decision L); allowing reassignment means re-running Create's entire duplicate-check against the *target* category on every edit, for a use case ("I want this budget to track a different category") that is already fully served by archive-old + create-new — at zero additional backend complexity and with an unambiguous audit trail (the old Budget's history isn't silently repurposed to mean something else).

**Delete: omitted from MVP entirely**, not "for a never-used Budget only." Reasoning against the task's own decision menu:
- Archive/restore already answers every "I don't want this anymore" journey without a second destructive action.
- PD-009 Decision J already establishes hard-delete as the eventual behavior *if* a delete endpoint is ever added, and that a deleted Budget's uniqueness slot would immediately free up — a materially different outcome from archiving (which deliberately does *not* free the slot, per Decision L). Exposing both in the same MVP creates exactly the "two destructive actions without a clear distinction" the task warns against: a user could not intuit from the UI alone why "Archive" keeps the category reserved but "Delete" doesn't.
- No journey in §6 requires delete. Adding it now would be scope not justified by an approved MVP journey — the same bar the domain-gap review (§12) applies to fields.

**Archive/restore is sufficient and is the only destructive-adjacent action in v1.**

---

## 11. Period and Historical-View Decision

**Chosen: Option 1 — current-month-only MVP, historical views deferred.** This is not a new decision; it is PD-009 Decision J restated for the UX/API layer, confirmed still correct:

- The domain has no `BudgetPeriod` snapshot model, and Phase A's query service only ever resolves "the current Reporting Period."
- Because v1 never displays a past period, editing a Budget's amount has no retroactive UI to reconcile — screen-spec.md's Notes sections were updated this iteration to state this explicitly ("editing a Budget's amount changes the limit used for every future read; it is never retroactive") so no future copywriter or implementer accidentally implies otherwise.
- A schema change (`BudgetPeriod` snapshots) is confirmed **not required** for this MVP and should only be introduced when historical browsing is actually scheduled — building it now would be exactly the speculative field the domain-gap review below rejects.

---

## 12. Domain Gaps and Whether Any Schema Change Is Required

**No schema change is required.** Walked through the task's evaluation list against the approved journeys (§6) and finalized UX (§4):

| Candidate field | Verdict | Reasoning |
|---|---|---|
| Custom name / notes | Not added | No journey references a user-supplied name distinct from the Category's own name. |
| Sort order | Not added | List is small (≤7 categories); `createdAt` ordering (already the query service's behavior) is sufficient. |
| Status override | Not added | Status is fully derived; an override would contradict "never computed independently" by introducing a second, conflicting source. |
| Period fields / start month | Not added | Confirmed unnecessary — Budget is a recurring definition, not an instance (§11). |
| Category alias | Not added | No journey needs a Budget-specific label distinct from `Category.name`. |
| Soft delete | Not added | No delete endpoint exists in MVP (§10); archive already serves the "hide but keep" need via `isArchived`. |
| Immutable historical amounts | Not added | Only relevant once historical browsing exists (§11); not required now. |

### A. Budget origin metadata

**Recommendation: defer.** No `origin`/`createdBy` field is added in this iteration. Automation (Roadmap Phase 7) is several phases away and has no approved product contract yet; adding a field for a consumer that doesn't exist is exactly the speculative-future-need pattern the review explicitly warns against. When Automation is scoped, the minimum honest addition at that time is a nullable `origin` enum/string on `Budget` (mirroring how `RecurringReminderEvent` already distinguishes user- vs. system-originated rows) — not a general audit system, which would be over-scoped for a single field's need.

### B. Service boundaries

**Confirmed:** `budget-query.service.ts` remains Budget-specific — its `BudgetQueryPrismaClient = Pick<PrismaClient, 'budget'|'transaction'>` type already structurally prevents it from silently absorbing unrelated entities. A future Analytics v2 (Roadmap Phase 6, "Reporting") should reuse the **lower-level primitive** the readiness audit and calculation spec both point to — the `db.transaction.groupBy(['categoryId'], ...)` pattern itself — not call into `budget-query.service.ts` directly or grow it into a general reporting service. This boundary is recorded here so Phase B's implementer does not "helpfully" generalize the service while adding CRUD.

### C. Historical truth

**Confirmed avoided.** See §11 — current-month-only UX structurally avoids the reproducibility problem a snapshot-less design would otherwise create, exactly per PD-009 Decision J's own reasoning. No minimum necessary domain change exists because none is needed yet.

---

## 13. Frontend Implementation Contract

No code written. Structure below follows the confirmed existing convention (`src/features/wallets/hooks/useWallets.ts`, `app/(app)/wallets/`) — see the parallel Explore findings; not re-derived independently here.

```
pocket-mint-fe/
  src/types/budget.ts                              # BudgetDto mirror, BudgetStatus union
  src/features/budgets/hooks/useBudgets.ts          # API fns + useBudgets/useCreateBudget/useUpdateBudget/useArchiveBudget/useRestoreBudget + invalidateBudgetDependents
  app/(app)/anggaran/page.tsx                       # Budgets list + Archived toggle
  app/(app)/anggaran/components/CreateBudgetModal.tsx
  app/(app)/anggaran/components/EditBudgetModal.tsx
  app/(app)/anggaran/[id]/page.tsx                  # Budget Detail (category, figures, contributing transactions via existing transaction hook filtered by categoryId+period)
  components/layout/app-sidebar.tsx                 # + Budgets nav item
  components/layout/bottom-nav.tsx                  # + Budgets nav item (must match sidebar exactly)
  messages/en.json, messages/id.json                # + budgets / budgetModals.create / budgetModals.edit / budgetModals.archive / nav.budgets keys
```

Reused, not reinvented: `AppModal`/`ModalCancelButton`/`ModalSubmitButton`, `FormField`/`FormErrorMessage`, `toast()`, the hand-rolled progress-bar `<div>` pattern (`tagihan`/`target-tabungan` precedent — no chart library, no new Progress primitive introduced solely for Budgeting), the hand-rolled status-pill `<span>` pattern (no new Badge primitive). No new shared component is introduced prematurely; if a second and third feature converge on the identical pill/bar markup after Budgeting ships, extracting a shared primitive becomes a justified, evidence-based follow-up — not a Phase B deliverable.

**React Query behavior:**
- List query: `['budgets', status]`, `staleTime: 5 * 60 * 1000` (matches every other feature).
- Mutations: **pessimistic**, matching the app-wide convention (no `onMutate`/optimistic writes exist anywhere in the codebase today — introducing one for Budgeting alone would be a new, unrequested pattern). `isPending` disables the submit button and swaps its label, per `ModalSubmitButton`'s existing prop.
- Invalidation: `invalidateBudgetDependents(queryClient)` invalidates `['budgets']` and `['dashboard']` on every Budget mutation. **Cross-feature touch point:** `useTransactions.ts`'s existing `invalidateTransactionDependents` must add `['budgets']` to its list — a Transaction mutation can change a Budget's live usage even though it never touches the `Budget` table. This is a one-line addition to an existing array, not a new invalidation mechanism, and is explicitly Phase B/C scope, not done in this documentation-only iteration.
- Retry/rollback: default React Query retry; no rollback logic needed since nothing is optimistic.

---

## 14. Accessibility and Localization Decisions

Indonesian/English copy already approved by PD-009 Decision K is restated here for implementer convenience (source of truth remains PD-009):

| English | Indonesian |
|---|---|
| Budget | Anggaran |
| Budget Limit | Batas Anggaran |
| Spent | Terpakai |
| Remaining | Tersisa |
| Reached | Tercapai |
| Approaching | Mendekati Batas |
| Exceeded | Melebihi Anggaran |
| Archived | Diarsipkan |
| Create Budget | Buat Anggaran |
| Edit Budget | Ubah Anggaran |
| Archive | Arsipkan |
| Restore | Pulihkan |

No-budget empty state, duplicate-budget error, invalid-amount error, and category-eligibility error copy are specified per-journey in `budgeting-user-journeys.md` (Journeys 1–3) rather than duplicated here as standalone strings, since their exact wording depends on the triggering context.

Accessibility requirements (detailed per-journey in `budgeting-user-journeys.md`): progress bars carry `role="progressbar"` with `aria-valuenow`/`aria-valuemin`/`aria-valuemax` and a status-inclusive `aria-label`; status is never color-only (text label always accompanies the pill, matching the existing nav `aria-current` non-color-cue convention); `FormField`'s existing `aria-invalid`/`aria-describedby` wiring is reused for Create/Edit validation errors; `AppModal`'s existing focus-trap/restore behavior is reused, not reimplemented; Archive/Restore controls carry explicit `aria-label`s naming the affected Budget; no interaction relies on hover (matches the existing app-wide convention); exceeded values (>100%) are never silently clamped in accessible text even where the visual bar is capped (Journey 7).

---

## 15. Future Test Matrix

**Backend contract tests** (Phase B, once routes exist): create; duplicate (active and archived); invalid amount (`<=0`, `>2` decimals); wrong-user category; non-expense category; `PATCH` with `categoryId` present → `422`; list active with usage; list archived with usage; archive; archive-already-archived → `409`; restore; restore-already-active → `409`; update amount; ownership isolation (cross-user `id`/`categoryId` never distinguishable from not-found); status boundary re-assertion at the HTTP layer (75/100/100.0001, reusing the existing `budget.test.ts` fixture table); response serialization (Decimal→number, `percentUsed` unrounded); timezone/current-month behavior (reuse the existing Jakarta half-open fixture from `budgetQueryService.test.ts`); no N+1 regression at the controller layer (assert `groupBy` still called once for a multi-budget list request).

**Frontend tests:** empty state; list states (loading/error/success); exact 75%/100%/>100% rendering (assert text and `aria-valuenow`, not just visual width); create success; create duplicate (`409` handling); edit; archive; restore; validation error display; localization (`en`/`id` key presence); mobile layout; accessibility (progress bar roles/labels, form field associations); query invalidation (`['budgets']`, `['dashboard']`, and the cross-feature `['budgets']` addition to `invalidateTransactionDependents`).

**E2E acceptance scenario** (no notification assertions): open Budgeting with no Budget → create a Food Budget → appears at zero usage → a matching Expense changes usage → editing that transaction recalculates usage → reaches exactly 100% → exceeds 100% → archive the Budget → restore it.

---

## 16. Documentation Files Created or Updated

| File | Change |
|---|---|
| `docs/product/screen-spec.md` | Updated Budgets, Budget Detail, Create Budget sections from Draft/Provisional language to Approved; added Edit Budget section; removed Delete action; added Archived-toggle and dashboard-deferral language. |
| `docs/development/budgeting-api-contract.md` | **New.** Full Phase B API contract. |
| `docs/development/budgeting-user-journeys.md` | **New.** 15 implementation-ready journeys. |
| `docs/development/budgeting-phase-a5-design-review.md` | **New.** This document. |
| `docs/development/implementation-roadmap.md` | Phase 10 status note updated to record Phase A.5 completion. |
| `docs/product/decisions/009-budgeting-scope.md` | History table appended noting Phase A.5 review completed; no decision content changed — no new ADR was needed (see below). |

**No new ADR was created.** Every UX/API decision finalized in this iteration (category-reassignment policy, delete omission, dashboard-integration timing, single-`BudgetDto` shape) is an *implementation-level* elaboration of policy PD-009 already approved (Decisions E, J, L), not a new product-scope decision — none of them changes what Budgeting *is*, only how its already-approved rules are exposed. Per the task's own instruction ("a new durable decision is required" only when necessary), a new PD was judged unnecessary; PD-009's History table is appended instead to record that this elaboration happened and where.

**Phase status:** Phase A — complete. Phase A.5 (this iteration) — complete. Phase B — next, not started. Budgeting overall — in progress, not released.

---

## 17. Refined Phase B Implementation Plan

**Recommended division: Option 2 (B1/B2/B3), not Option 1.** Reasoning: the repository's own `agent-rules.skill.md` requires `tsc --noEmit`, `npm run build`, `vitest run`, `prisma validate`, and a committed `dist/` rebuild as the done-bar for *every* backend task — bundling command-service + controller + routes + tests into one PR makes that verification gate (and any review) cover a much larger, harder-to-isolate diff than the existing `savingGoal`/`installment` features were built with. The existing codebase's own service/controller/route file split (three files per feature, not one) already implies the smaller-task division is the house style.

### B1 — Budget command service and validation

- **Scope:** `createBudget`, `updateBudgetAmount`, `archiveBudget`, `restoreBudget` in a new `src/services/budget.service.ts` (command side, separate from the existing read-only `budget-query.service.ts` — matches the `transaction.service.ts` vs. `transaction-query.service.ts` split already used elsewhere). Validation per `budgeting-api-contract.md` §"Validation" (amount `>0`, ≤2 decimals; category ownership + `EXPENSE` type; duplicate check across active+archived; `422 CATEGORY_NOT_EDITABLE` guard belongs in the controller's request-mapping layer, not the service, matching `savingGoal.controller.ts`'s allowlist-mapper pattern).
- **Expected files:** `src/services/budget.service.ts`, `src/services/budget.types.ts` (or extending `budget-query.types.ts`), unit tests `test/budgetService.test.ts` (mocked Prisma client, same pattern as `budgetQueryService.test.ts`).
- **Acceptance criteria:** every validation/error row in the API contract's error table has a passing test; duplicate check covers both `isArchived: true` and `false` existing rows; no route/controller/model file touched.
- **Validation commands:** `npx tsc --noEmit`, `npx vitest run test/budgetService.test.ts`, `npx prisma validate` (no schema change expected, run as a guard).
- **Non-goals:** no HTTP layer, no request parsing, no rate limiting, no route registration.

### B2 — Controller, routes, request DTOs

- **Scope:** `src/models/budget.model.ts` (`CreateBudgetDto`, `UpdateBudgetDto`), `src/controllers/budget.controller.ts` (mirrors `savingGoal.controller.ts`'s shape: allowlist mappers, `serialize()` matching `BudgetDto`, `sendSuccess`/`forwardError`), `src/routes/budgetRoutes.ts` (`requireUser` on all, `mutationLimiter` on mutating routes), wiring into `src/routes/index.ts`.
- **Expected files:** the three above, plus `test/budgetRoutes.test.ts` or equivalent HTTP-level integration test (matching the existing route test convention for `savingGoal`/`installment`).
- **Acceptance criteria:** every endpoint in `budgeting-api-contract.md` §"Endpoints" implemented exactly as specified (paths, status codes, error codes); `npm run build` succeeds and `dist/` is rebuilt/committed per `agent-rules.skill.md`; contract tests from §15 above pass.
- **Validation commands:** full backend gate — `npx tsc --noEmit`, `npm run build`, `npx vitest run`, `npx prisma validate`, `git diff --check`.
- **Non-goals:** no frontend changes, no dashboard integration, no notification/threshold work (still Decision F's own later phase).

### B3 — Frontend integration

- **Scope:** exactly the file list in §13 above — types, hooks, Budgets/Budget Detail pages, Create/Edit modals, nav entries (both sidebar and bottom-nav), localization keys, and the one-line `invalidateTransactionDependents` addition.
- **Expected files:** listed in §13.
- **Acceptance criteria:** all 15 journeys in `budgeting-user-journeys.md` manually verifiable in a running dev instance; existing Vitest "source contract" tests extended for the new components (matching the codebase's existing test style — `readFileSync` + string-marker assertions, not rendered-DOM tests, per the Readiness Audit's confirmed convention); no chart library, form library, or new shared UI primitive added.
- **Validation commands:** frontend's existing `npm run build`/`npm run test`/lint gate (not enumerated here — owned by `pocket-mint-fe`'s own agent instructions, not read in full during this backend-focused review; confirm against that repo's CLAUDE.md/AGENTS.md before starting B3).
- **Non-goals:** dashboard summary slot (explicitly deferred, §4), notification UI (Decision F's separate phase), any historical/month-navigation UI (§11).

---

## 18. Exact Recommended Next Implementation Task

**B1 — Budget command service and validation**, in `pocket-mint-be`, on a new `feature/budgeting-command-service` branch cut from `dev`, implementing exactly the scope in §17/B1 above and validated against `budgeting-api-contract.md`'s Validation table. This is the smallest task that unblocks B2 without touching HTTP surface area, and it is the one piece of Phase B with zero frontend or routing dependency, so it can start immediately.

---

## 19. Validation Performed

- Read the actual Phase A source (schema, migration, domain calculation, query service, types, errors, both test files) directly — not solely the prior Readiness Audit — confirmed no mismatch (§2).
- Read PD-009, the calculation spec, the readiness audit, the roadmap, and the glossary in full for this iteration.
- Read the existing `savingGoal` controller/route/response/error-forwarding stack as the API-contract template.
- Explored frontend navigation, routing, API client, React Query conventions, UI primitives, localization, and accessibility conventions via a dedicated frontend audit pass.
- `git status` confirms only documentation files (`.md`) under `pocket-mint-docs/docs/` were changed by this iteration.
- No `npx prisma validate`, `npx tsc`, `npm run build`, or `npx vitest run` was run — no backend source changed, so the backend verification gate in `agent-rules.skill.md` does not apply to this documentation-only iteration.
- No migration was run. No commit was made. No push was made.

---

## 20. Git Diff Summary

Documentation-only changes, all within `pocket-mint-docs`:

- Modified: `docs/product/screen-spec.md` (Budgets/Budget Detail/Create Budget sections revised, Edit Budget section added).
- Modified: `docs/development/implementation-roadmap.md` (Phase 10 status note).
- Modified: `docs/product/decisions/009-budgeting-scope.md` (History table entry appended).
- Added: `docs/development/budgeting-api-contract.md`.
- Added: `docs/development/budgeting-user-journeys.md`.
- Added: `docs/development/budgeting-phase-a5-design-review.md` (this file).

No file under `pocket-mint-be/` or `pocket-mint-fe/` was modified. No Prisma schema, migration, route, controller, component, or `package.json` changed.

---

## 21. Git Status for All Three Repositories

Recorded at the point this document was written — `pocket-mint-docs` shows the six files above as pending changes (not committed, per instruction not to commit); `pocket-mint-be` and `pocket-mint-fe` are unchanged by this iteration and reflect whatever state they were in before this review began.
