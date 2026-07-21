# Pocket Mint Budgeting Readiness Audit

> Audit date: 2026-07-21
> Audit scope: `pocket-mint-be`, `pocket-mint-fe`, `pocket-mint-docs`
> Method: read-only static inspection of schema, services, routes, frontend source, and documentation. No source, schema, or dependency changes were made.
> Purpose: establish, with repository evidence, what exists today that a Budgeting feature (roadmap phase after the current stable release) can build on, and what is genuinely missing.
> Related: [PD-009 — Budgeting Scope and MVP Definition](../product/decisions/009-budgeting-scope.md), [Budgeting Calculation Specification](./budgeting-calculation-spec.md)

This document records **current behavior only**. Recommendations and product decisions live in PD-009, not here. Where behavior could not be confirmed from the repositories inspected, it is marked `UNVERIFIED` rather than assumed.

---

## Executive Summary

Pocket Mint's backend already has almost every primitive Budgeting needs: a user-scoped `Category` model, a canonical `Transaction.categoryId` relation, a reporting-timezone-aware month-boundary helper (`src/domain/reportingTime.ts`), a DB-`groupBy`-based monthly summary query, and an idempotent threshold/notification pattern already proven by the recurring-reminder engine. No Budgeting model, endpoint, or UI exists yet.

Two gaps matter most:

1. **No spend-by-category aggregation query exists.** Only a spend-by-type (`INCOME`/`EXPENSE`) monthly `groupBy` exists. A category-scoped version must be added.
2. **The RFC and Vision documents currently list "a budgeting-envelope application" as an explicit Non-Goal.** A Budgeting feature must be scoped so it does not contradict that boundary, and the Non-Goals language must be clarified by an approved Product Decision before implementation proceeds. See PD-009.

Everything else below is either directly reusable or a small, well-understood extension of an existing pattern.

---

## 1. Transaction Types — Current Behavior

**Confirmed** (`pocket-mint-be/prisma/schema.prisma:124-128`, `130-163`):

- `TransactionType` enum: `INCOME`, `EXPENSE`, `TRANSFER`. No `INSTALLMENT` transaction type exists.
- An installment purchase is an `EXPENSE` transaction with `isInstallment: true` and `installmentId` set. The wallet balance is debited the full `grandTotal` at creation (front-loaded, per PD-001-adjacent installment model), but the **persisted `Transaction.amount` is `monthlyAmount`**, not `grandTotal` (`financial-logic.skill.md:199-206`, confirmed by backend research).
- An installment repayment is a `TRANSFER` (asset wallet → debt wallet), never an `EXPENSE` (`pocket-mint-be/src/services/installment-payment.service.ts`; `financial-logic.skill.md:131-139`).
- A debt repayment / credit-card payment is likewise a `TRANSFER`, excluded from `INCOME`/`EXPENSE` aggregation entirely (`src/domain/reportingEffect.ts:40`, `getAggregateCashFlowEffect` returns 0 for `TRANSFER`).

**Confirmed constraint:** the existing monthly summary (`transaction-query.service.ts:132-155`) sums the **persisted `amount` column** via `groupBy(['type'])`. For an installment `EXPENSE` row this is `monthlyAmount`, which already disagrees with the `grandTotal` used to debit the wallet at creation. This is a pre-existing nuance in Pocket Mint's reporting semantics, not something Budgeting introduces — Budgeting must follow the same convention to stay consistent with the numbers users already see on Dashboard/Analytics (see PD-009, Decision C).

---

## 2. Categorization — Current Behavior

**Confirmed:**

- `Category` is a real Prisma model, **user-owned** (`userId` FK, cascade delete), with `type: CategoryType` (`INCOME` | `EXPENSE`) (`prisma/schema.prisma:101-119`).
- `@@unique([userId, name, type])` — a user cannot have two categories with the same name and type.
- Default categories (7 `EXPENSE`, 5 `INCOME`, Indonesian names) are lazily seeded per-user via `upsert` every time `listCategories` runs (`src/services/category.service.ts:4-24`).
- **No public endpoint exists to create a custom category.** `src/routes/categoryRoutes.ts` wires only `GET /`. Category creation is backend-internal/seed-only from the exposed API surface.
- `Transaction.categoryId` is a nullable FK (`onDelete: SetNull`), but the service layer **requires** a category for `INCOME`/`EXPENSE` at creation and update time (`transaction.service.ts:376-378`, `'categoryId wajib diisi untuk pemasukan dan pengeluaran'`). `TRANSFER` forbids a category (`373-375`).
- Category `type` must match transaction `type` (`transaction.service.ts:386-388`).
- **Practical consequence:** a genuinely uncategorized `EXPENSE`/`INCOME` transaction can only occur if its linked `Category` row is later deleted (`onDelete: SetNull`) — it is not a normal state produced by the create/update flow.
- Frontend: `src/features/categories/hooks/useCategories.ts` fetches categories via `GET /categories` (React Query) and feeds a `Select` in `AddTransactionModal.tsx`. No category management (create/edit/delete) screen exists in the frontend. `financial-logic.skill.md` (frontend) states explicitly: **"Category is metadata — not used in core calculations."** — true today; Budgeting is the first feature to make categories calculation-relevant.

**Confirmed constraint:** Categorization is sufficient for a category-scoped Budgeting MVP as-is (every user already has categorized `EXPENSE` rows against a stable, user-scoped `Category` id). The absence of a category-creation endpoint limits users to the seeded defaults; this is a UX limitation, not a blocker (see PD-009, Decision D).

---

## 3. Dates and Periods — Current Behavior

**Confirmed** (`pocket-mint-be/src/domain/reportingTime.ts`, 141 lines):

- `Transaction.date` is a full `DateTime`, not a date-only column (`schema.prisma:144`).
- `REPORTING_TIMEZONE` env var, default `Asia/Jakarta`, resolved once via `reportingConfig.timezone` (`src/config/index.ts`).
- `getReportingMonthRange(value, zone)` returns a half-open `{ startInclusive, endExclusive }` pair — the canonical way to bound "this calendar month" in the reporting timezone. `getPreviousReportingMonthRange` also exists.
- All existing period-bounded queries use `gte`/`lt` (never `lte`) on the end bound (`transaction-query.service.ts:104`), and use `Transaction.date` (business date), never `createdAt`, for period membership (`financial-logic.skill.md:346-350`).
- Timezone math is done via `Intl.DateTimeFormat` + iterative UTC-offset resolution — **no date library** (`date-fns`/`luxon`/`dayjs`) is a dependency.
- **No special handling exists anywhere for "future-dated" transactions** — a transaction dated in month M counts toward month M's aggregate regardless of when "today" is or when the row was created. There is no rejection of future dates found in transaction validation during this audit (`UNVERIFIED` — not explicitly disproven, but no such check was located in `transaction.service.ts`).
- **No soft-delete column exists on `Transaction`.** Deletes are hard deletes (`transaction.service.ts:446-498`, `tx.transaction.delete(...)`), with the wallet balance effect reversed first. A deleted transaction is therefore automatically absent from any query run after the deletion — no special "deleted" handling is needed by a consumer that queries live data.

**Confirmed constraint:** `getReportingMonthRange` is the correct, existing, reusable primitive for Budgeting's monthly period boundaries. Building a new date-boundary implementation would violate the "one canonical path" architecture principle and duplicate a helper that already exists.

---

## 4. Transaction Mutations — Current Behavior

**Confirmed** (`transaction.service.ts`):

- Editable after creation: `type`, `amount`, `description`, `date`, `categoryId`, `walletId`, `toWalletId` (update path, lines 294-438).
- Installment-linked transactions (`isInstallment: true`) cannot be edited directly (409 `'Transaksi cicilan tidak bisa diubah langsung'`).
- A regular transaction cannot be converted into an installment via update (400).
- Every mutation (create/update/delete) is wrapped in `db.$transaction(...)`.
- Because there is no soft delete and no cached "spent" value anywhere in the schema, **any budget usage figure that is computed live from `Transaction` rows automatically reflects edits, category re-assignment, deletions, and backdating with zero extra invalidation logic.** This is a significant simplification for Budgeting: there is nothing to invalidate.

---

## 5. Existing Analytics/Aggregation — Current Behavior

**Confirmed:**

- `dashboard-query.service.ts` (`getSummary`) computes Net Worth from wallet balances only — **not date-scoped, no category breakdown.**
- `transaction-query.service.ts` (`getSummary`, lines 132-155) computes the **only existing "monthly totals" query**: `db.transaction.groupBy({ by: ['type'], where: { userId, type: {in:['INCOME','EXPENSE']}, date: {gte, lt} }, _sum: {amount:true} })`. This is scoped by type only, **not by category.**
- **No `groupBy(['categoryId'])` query exists anywhere in the codebase** (confirmed by grep across `src/`). A category-scoped aggregation must be added new.
- The codebase has an explicit, documented preference for DB-level `groupBy` over in-memory summation for money aggregation (`transaction-query.service.ts:118-131` code comment: "In-memory reporting-effect aggregation would fetch every row for the same result, so DB aggregation is preferred.") — Budgeting's spend aggregation should follow the same pattern.
- Frontend: `app/(app)/analytics/page.tsx` already renders a **category breakdown** (income/expense by category, likely calling a backend endpoint not enumerated by this audit pass — `UNVERIFIED` which exact endpoint backs it; flag for implementation-time verification) as plain CSS width-percentage bars, confirming there is frontend appetite/precedent for category-level breakdowns even though the backend `groupBy(['categoryId'])` primitive itself was not found. This should be reconciled before Phase B backend work begins.

---

## 6. Notifications — Current Behavior

**Confirmed:**

- There is no separate `Notification` model. `RecurringReminderEvent` (`schema.prisma:281-306`) is the notification-equivalent model today, scoped to recurring-transaction and installment reminders only.
- Read/unread: `readAt: DateTime?` (null = unread).
- **Duplicate prevention pattern** (directly reusable): DB-level composite unique constraints (`@@unique([templateId, occurrenceDate, offsetDays])`) combined with `upsert(..., update: {})` in the engine — re-running the evaluation for the same occurrence is a no-op, not a duplicate insert.
- Creation is **event-driven / on-demand** (`notification.service.ts:refreshNotifications`), invoked when a user opens the notification menu/page. Whether a cron/scheduler also invokes `recurringReminderEngine.service.ts:evaluateReminders` could not be confirmed in this pass (`src/scripts/` was not opened) — **`UNVERIFIED`**. This must be confirmed before assuming budget-threshold notifications can rely on a background scheduler; the safe assumption for MVP is refresh-on-read/refresh-on-mutation, matching the confirmed pattern.
- No amount/percentage threshold logic exists anywhere today (PD-005 Debt Ratio thresholds are Draft, not implemented).

---

## 7. Recurring Transactions and Bills — Current Behavior

**Confirmed:**

- `RecurringTransactionTemplate` is **template-only**. Per its own code comment: "Phase 1 only: create/read/update/delete the template row. No due-date computation, no transaction generation." Templates are **never** auto-materialized into `Transaction` rows.
- The only path from template → real posted `Transaction` is a **user-initiated** confirmation of a `RecurringReminderEvent` (`notification.service.ts:confirmReminder`), which atomically creates the `Transaction` and marks the reminder `completedAt` + `generatedTransactionId`.
- **Conclusion: only posted `Transaction` rows currently affect any financial total in Pocket Mint.** Recurring templates and their reminders have zero effect on Net Worth, wallet balances, or the monthly summary until confirmed. Budgeting inherits this same rule for free — no new "planned vs. posted" branching is needed if Budgeting only reads `Transaction` rows (see PD-009, Decision H).

---

## 8. Installments — Current Behavior

Covered in §1 and §5. Restated for clarity: the credit purchase is an `EXPENSE` (counts toward budgets via its persisted `monthlyAmount`), the repayment is a `TRANSFER` (excluded from budgets, like all transfers).

---

## 9. Multi-Currency — Current Behavior

**Confirmed:** no `currency` field exists on `User`, `Wallet`, or `Transaction` (grep across `src/` and `prisma/schema.prisma` returned no matches outside unrelated comments). IDR is implicitly assumed everywhere; `Decimal(15,2)` is the universal money scale. Budgeting must not introduce any per-wallet or per-user currency concept — there is nothing to aggregate against.

---

## 10. Frontend Architecture — Current Behavior

**Confirmed:**

- Nav items are declared in two duplicated arrays that must both be updated for a new nav entry: `pocket-mint-fe/components/layout/app-sidebar.tsx:30-37` (desktop) and `components/layout/bottom-nav.tsx:32-39` (mobile). Both read labels from `messages/{en,id}.json` under the `nav` key.
- **No charting library is installed** (`package.json` has no recharts/chart.js/d3/visx). The existing category breakdown on `app/(app)/analytics/page.tsx` is rendered as plain CSS width-percentage `<div>` bars. `skills/design.md` explicitly forbids adding a chart/sparkline. Budget progress must use the same div-bar convention.
- **No form library** (`react-hook-form`, `zod`, `yup`) is installed anywhere in the frontend. Forms use `useState` + manual validation (`AddTransactionModal.tsx` pattern). A Budget create/edit form must follow the same convention — introducing a form library for one feature would violate the existing pattern and add an unrequested dependency.
- **No shared `EmptyState`/`Skeleton`/`ErrorState` component exists.** Every page hand-rolls these inline (`animate-pulse` divs, local `EmptyPanel` components). Budget screens should do the same, not invent a new shared abstraction unrequested by this task.
- Data fetching: TanStack React Query v5 throughout, one shared axios instance (`lib/api.ts`) with a bearer-token interceptor. `src/features/wallets/hooks/useWallets.ts` is the confirmed template for a new `src/features/budgets/hooks/useBudgets.ts`.
- API envelope: `{ success, data, message }` on success, `{ success: false, error: { code, statusCode, message } }` on error (`pocket-mint-be/src/utils/response.ts`) — the single documented exception is the bare `GET /dashboard/summary` response, preserved for compatibility. A new Budgets module must use the standard envelope.
- Dashboard section order in code (`app/(app)/dashboard/page.tsx`): Hero Card → inline stat bar → Quick Actions → Wallet Overview (capped at 4) → Recent Activity (capped at 5). There is **no separate "Current Period Summary" section** in code today — `netSavings`/`monthDelta` is folded into the Hero Card itself, which is itself a pre-existing deviation from the documented Hero Card contract (`skills/design.md`) unrelated to Budgeting. The best insertion point for a Budget summary is between Wallet Overview and Recent Activity.
- Testing: Vitest "source contract" tests (`tests/*.test.ts`) that `readFileSync` component source and assert string markers — not rendered DOM tests. No Playwright e2e specs exist yet; Playwright is only used for authenticated screenshot capture.

---

## 11. Documentation Conflict — Must Be Resolved Before Implementation

**Confirmed, critical:**

- `pocket-mint-docs/docs/product/product-rfc.md:88` (Non Goals): *"a budgeting-envelope application"*
- `pocket-mint-docs/docs/product/vision.md:126` (Non-goals): *"A budgeting-envelope system."*
- `docs/product/stictch/core/design-principles.md:13,118`, `docs/product/design-system.md:187`, `docs/product/stictch/core/DESIGN.md:187`, `docs/product/stictch/core/design-philosophy.md:211`, and three `stictch/packages/*.md` files all list "budgeting game(s)" as a pattern to avoid resembling.
- `docs/product/glossary.md` has no `Budget` or `Anggaran` term today.

**This is not a blocker to designing the feature, but it is a blocker to implementing it without an approved Product Decision.** Pocket Mint's documentation authority order (`docs/ai/project-context.md`) requires the Vision and RFC's Non-Goals language to be reconciled through an approved Product Decision before Budgeting behavior is treated as settled policy. PD-009 (this iteration's companion document) proposes the precise scoping language that distinguishes "budget = a spend-limit tracked against recorded transactions" from "budgeting-envelope = zero-based allocation of unspent funds across categories," and recommends the specific Non-Goals wording change. **PD-009 is Draft, not Approved, in this iteration** — no Non-Goals or Vision text has been changed by this audit.

---

## Readiness Classification Summary

| Area | Classification | Notes |
|---|---|---|
| Category model | `SUFFICIENT_FOR_MVP` | User-scoped, typed, seeded defaults exist. No custom-category creation endpoint (non-blocking gap). |
| Transaction date/period semantics | `REUSABLE` | `getReportingMonthRange` is a direct fit. |
| Transaction mutation handling | `REUSABLE` | No soft delete, no cache — dynamic queries need no invalidation logic. |
| Category-scoped spend aggregation | `MISSING` | Must be built new; only a by-type `groupBy` exists today. |
| Notification/threshold idempotency pattern | `REUSABLE` | `RecurringReminderEvent` unique-constraint + upsert pattern is a proven template. |
| Recurring transaction materialization | `CONFIRMED_BEHAVIOR` | Only posted transactions ever affect totals; no planned/actual mixing risk. |
| Multi-currency | `NOT_APPLICABLE` | No currency concept exists; single-currency IDR assumption is safe. |
| Frontend charting | `CONSTRAINT` | No charting library; must use existing CSS-bar convention. |
| Frontend forms | `CONSTRAINT` | No form library; must use existing `useState` convention. |
| Product documentation (Non-Goals) | `CONFLICT_REQUIRES_PD` | Must be resolved by an approved Product Decision before implementation, not by this audit. |
| Backend cron/scheduler existence | `UNVERIFIED` | `src/scripts/` not inspected in this pass; confirm before assuming background threshold evaluation is available. |
| Backend category-breakdown endpoint used by `app/(app)/analytics/page.tsx` | `UNVERIFIED` | Exact backend source not identified in this pass; reconcile before Phase B. |

---

## Phase B4 Verification (2026-07-21)

Ran after Phases A–B3 (domain, command service, HTTP API, frontend) were implemented, to close verification gaps before Dashboard or notification work starts. Findings are evidence-based (actual files read, actual commands executed), not carried forward from earlier summaries.

**Backend integration-test harness — real, already well-built:**
- `scripts/run-integration-tests.mjs` boots a disposable `embedded-postgres` instance (or reuses an existing `TEST_DATABASE_URL`), runs `prisma migrate deploy`, runs `test/prismaAdapter.integration.test.ts` + `test/notificationRefreshE2E.integration.test.ts`, then tears down. `assertTestDatabaseUrl` blocks Supabase/RDS hosts and the `postgres` database name at test-collection time.
- `docs/database-testing.md` claimed `.github/workflows/ci.yml` already ran a `postgres:18` service — **false**. The actual workflow had no `services:` block, no `TEST_DATABASE_URL`, and no migration step; the integration suite's `describe.skipIf(!TEST_DATABASE_URL)` meant it silently never ran in CI. Corrected in this pass: `ci.yml` now runs a `postgres:18` service container, applies `prisma migrate deploy` against it, and sets `TEST_DATABASE_URL` only for the `Test` step.
- Local run via `npm run test:integration` against the embedded-postgres harness: **32/32 tests passed**, including duplicate-active-Budget rejection, duplicate-archived-Budget rejection (no silent restore), Prisma `P2002` → `BUDGET_ALREADY_EXISTS` translation (both direct-race and pre-check paths), FK/ownership isolation (`CATEGORY_NOT_FOUND` for cross-user categories), cascade delete (user → Budget, category → Budget), and the full create→list→spend→verify→update→archive→verify→restore→verify acceptance path. All pre-existing (non-Budget) integration tests passed alongside.
- The GitHub Actions run of the updated workflow itself has **not** executed yet — that requires an actual push/PR run, which is outside a local session. CI workflow YAML was inspected for correctness (service definition, health check, migration step scoped before `Test`, `TEST_DATABASE_URL` scoped to the `Test` step only) but not exercised on GitHub's runners.
- Full backend validation also run and passed: `prisma validate`, `tsc --noEmit`, `vitest run` (642 passed, 32 skipped — the integration suite, correctly gated without `TEST_DATABASE_URL`), `npm run build`, and a clean `git status`/`git diff --check` after build (no stray `dist/` drift beyond what Phase A–B3 already introduced).

**Contributing-transactions audit — implementation is correct, no completeness bug:**
- `useTransactions()` (`GET /transactions`) is called with no `limit`/`categoryId` params. The backend's `resolveTake` treats an absent `limit` as "no cap" (unbounded), and the controller only supports `walletId`/`type`/`month`/`year`/`limit` filters — no `categoryId` filter exists.
- Both `useTransactions()` and the Budget query service derive their month boundary from the exact same `getReportingMonthRange`/`formatReportingDate` utility (`reportingConfig.timezone`, Asia/Jakarta) — so the client-side `tx.type === "EXPENSE" && tx.categoryId === budget.category.id` filter operates over the complete current-period transaction set, not a truncated or mismatched-period one.
- Copy was already honest ("Transactions this period" / "Transaksi periode ini") — no relabeling needed. No backend endpoint change made.

**Cross-feature invalidation — one real bug found and fixed:**
- `invalidateBudgetDependents` invalidated both `['budgets']` and `['dashboard']`, but the Dashboard reads no budget data anywhere (`grep -i budget src/features/dashboard` — no matches). This was speculative coupling to a deferred feature. Fixed: it now invalidates `['budgets']` only. `invalidateTransactionDependents` correctly still invalidates `['budgets']` (a transaction mutation can change Budget usage).

**Accessibility — one real ARIA bug found and fixed:**
- `BudgetProgressBar` set `aria-valuenow` to the true (possibly >100) `percentUsed` while declaring `aria-valuemax={100}` — an invalid ARIA range (`aria-valuenow` must not exceed `aria-valuemax`). Fixed: `aria-valuenow` is now capped at 100; the true value is exposed via `aria-valuetext` (previously carried on `aria-label`, which named the whole element rather than its value).

**Amount input precision — audited, no fix needed:**
- Both Create/Edit Budget modals hold the amount as a raw digit string (`value.replace(/\D/g, "")`) until submit; `Number()` is only ever applied to that digit string, and only at submit time. No decimal separator is accepted (Rupiah has no displayed sub-unit, matching the rest of the app), so "more than two decimal places" cannot occur by construction. Zero/negative rejected via `parsedAmount <= 0`. Values up to `Decimal(15,2)`'s 13-digit integer part are well within `Number`'s safe-integer range (2^53), so no precision loss.

**Mobile navigation — audited, not materially broken:**
- `bottom-nav.tsx` now has 8 destinations (7 nav entries + Account) rendered through `DockMorph`, an icon-only dock where labels appear only as a hover/focus/active tooltip, not persistent text — so the 8-item count does not translate into 8 text labels competing for width. Estimated total width at the 34–36px icon size comfortably fits within the `max-w-[min(100%,22rem)]` (352px) cap on a 375px viewport. No code change made (per phase scope, "do not redesign navigation merely because it exceeds a generic five-item guideline").
- Minor note, not fixed: the icon touch target at the sub-`sm` breakpoint measures roughly 34px icon + 4px inline padding on each side ≈ 42px, 2px under the 44px minimum in `ui-system.skill.md`. This is a pre-existing `DockMorph` sizing detail (not Budgeting-specific) and was not changed here — flagged for a future cross-nav a11y pass rather than a Budgeting-scoped fix.

**Live-browser verification — not performed, remains an open gate:**
- No browser/screenshot/automation tool was available in this verification session (only file-editing, shell, and text-fetch tools). Desktop/tablet/mobile visual QA of `/anggaran`, `/anggaran/[id]`, the Create/Edit/Archive/Restore modals, loading/empty/error states, EN/ID copy, keyboard focus, modal overflow, long category names, large currency values, and >100% progress bars was **not performed** and must not be reported as passing. This is the primary remaining release gate before Budgeting can be considered release-verified.

**Frontend tests strengthened:**
- `tests/budgets.test.ts` remains source-contract testing only (`readFileSync` + string assertions under a `node` vitest environment — no `@testing-library`/jsdom installed in this repo). Added a file-level note stating this limitation explicitly. Added/updated tests for: the corrected `['budgets']`-only invalidation (plus a regression guard that `"dashboard"` no longer appears in the hook source), the corrected ARIA `aria-valuenow`/`aria-valuetext` behavior, the amount-input digit-only/zero-guard pattern, and the contributing-transaction list's period-scoped (not all-time) labeling.

**Net status (superseded by Phase B5/B6 below):** at the end of Phase B4, Budgeting was **not release-verified**. Two real defects (dashboard over-invalidation, invalid ARIA range) were found and fixed in this pass; the CI Postgres integration gate is now wired but has not yet run on GitHub Actions; live-browser QA has never been performed and remains the largest open gap.

---

## Phase B5 — Authenticated Browser QA (2026-07-21)

Live-browser QA (the Phase B4 gap) was performed in a later session: desktop (1440×900), tablet (768×1024), and mobile (390×844, 375×812) all passed for the full authenticated Budget flow (create, spend, recalculate, edit, archive, restore). Runtime request payloads matched the approved API contract; accessibility verification passed for progress values above 100%.

Two defects surfaced during this pass, **neither of them Budgeting-specific**, and both are closed out in Phase B6 as app-wide/shared-component fixes rather than Budget changes:

1. **`FormField` duplicate DOM id.** `components/ui/form-field.tsx` cloned `id`/`aria-invalid`/`aria-describedby` onto whatever its immediate child was, including a plain layout `<div>` (the currency-prefix wrapper used by the Budget amount field), while the real `<input>` inside kept its own hardcoded id — two DOM elements sharing one id, silently breaking the label's `for` association. A narrow guard (skip cloning onto a bare `<div>`) was already applied by the time this audit resumed; Phase B6 added the missing regression test and verified the guard against every other `FormField` usage in the app.
2. **Missing app-wide route guard.** An anonymous browser could navigate directly to any `(app)` route, including `/anggaran`: the authenticated shell rendered, API calls 401'd, and the page stayed stuck in a loading state instead of redirecting to `/login`. This is app-wide, not Budgeting-specific — see Phase B6 for the root cause and fix.

## Phase B6 — Release Closure and Route-Guard Hardening (2026-07-21)

**Route guard root cause:** the proxy file (`pocket-mint-fe/proxy.ts` → `lib/supabase/middleware.ts`'s `updateSession`) was in fact running on every request — Next.js 16 renamed the `middleware.ts` convention to `proxy.ts`, and this repo had already migrated. The bug was not "no middleware exists"; it was a hardcoded `protectedRoutes` allow-list inside `updateSession` that had never been extended as new `(app)` routes shipped. `/anggaran`, `/cicilan`, `/target-tabungan`, and `/notifications` were all absent from it, so requests to those paths were never classified as protected and never redirected.

**Fix:** inverted the check to a small public-path allow-list (`/`, `/login`, `/changelog`, `/auth/*`) with every other route protected by default — a new `(app)` route is now protected automatically rather than depending on a manual list update. The guard already called `supabase.auth.getUser()` (server-verified) rather than `getSession()`, and already ran at the proxy layer before any authenticated UI rendered, so no second guard (middleware + layout) was added, and no return-path/redirect-query parameter was introduced (not needed for this fix; anonymous requests always land on `/login`).

**Regression coverage:** `tests/route-guard.test.ts` is a genuine behavioral test — it imports and calls the real `updateSession` against a mocked `@supabase/ssr` client and a real `next/server` `NextRequest`, covering: anonymous redirect for every protected path (including the four previously-missed ones), authenticated passthrough for the same paths, passthrough for every public path, and an authenticated `/login → /dashboard → /dashboard` sequence proving no redirect loop. `tests/private-route-auth.test.ts` (a pre-existing source-contract test) was rewritten from asserting the old hardcoded list to asserting the new allow-list shape, plus a negative assertion that a hardcoded `protectedRoutes` list is not reintroduced.

**FormField verification across usages:** all 10 `FormField` call sites in the frontend were enumerated. Every immediate child that is not a plain `<div>` is a native `<input>`, `MoneyInput`, or `Select` — none of them lose `aria-invalid`/`aria-describedby`/`id` propagation under the existing guard. `tests/form-field-id.test.ts` adds a behavioral test via `react-dom/server`'s `renderToStaticMarkup` (no new dependency; `react-dom` is already installed by Next.js) that renders both the layout-wrapper case (mirroring the Budget amount field) and the direct-child case, and asserts on the actual HTML output — a stronger check than this repo's usual source-text-contract convention, which is retained elsewhere for components this pass didn't need to touch.

**Full validation, this session:**

- Backend (no source changed — CI workflow reviewed only): `prisma validate` ✅, `tsc --noEmit` ✅, `vitest run` — 642 passed, 32 integration tests correctly skipped (no `TEST_DATABASE_URL`; no Docker/local Postgres available in this environment, so the integration suite was not re-run here — it was last verified 32/32 in Phase B4), `npm run build` ✅.
- Frontend: `vitest run` — 530 passed (500 pre-existing + 30 new: `route-guard.test.ts`, `private-route-auth.test.ts` additions, `form-field-id.test.ts`), `tsc --noEmit` ✅, `eslint .` — no issues, `npm run build` ✅ (build output confirms `Proxy (Middleware)` is registered and every `(app)` route, including `/anggaran`, `/cicilan`, `/target-tabungan`, `/notifications`, is present).
- Not performed: an actual GitHub Actions run of the backend CI Postgres workflow (requires a real push/PR, out of scope for a local session) — do not read the workflow-source review above as an executed CI pass.

**Net status (end of Phase B6):** Budgeting is **CONDITIONALLY READY** — full local validation and authenticated browser QA both pass, but release readiness is still gated on an actual GitHub Actions run of the Postgres integration workflow. The route-guard fix is an app-wide prerequisite, prepared as an independently reviewable change, not part of Budgeting's own scope.

---

## Phase B7 — Release Branches, PRs, and Real CI Verification (2026-07-21)

All four workstreams from Phase B6 were isolated into their own clean
worktrees off the current `origin/dev` (frontend/backend) or `origin/main`
(docs — this repository has no `dev` branch), pushed, and opened as separate,
independently reviewable PRs. The auth-route, backend Budgeting, and frontend
Budgeting PRs have since been merged to `dev` (this document records that
merge below); this docs PR is the last of the four to merge.

- **`fix/protect-app-routes`** → `pocket-mint-frontend` PR #49 — route-guard
  fix only. Anonymous-context browser smoke test re-run against a live local
  dev server (real Chromium via Playwright): `/anggaran`, `/dashboard`,
  `/cicilan`, `/target-tabungan`, `/notifications` all redirect to `/login`;
  `/`, `/login`, `/changelog` stay accessible; `/auth/callback` is left to
  the app's own callback route handler (not bounced to bare `/login` by the
  guard, confirmed via direct request inspection — no loop). Authenticated
  browser context (login → protected pages → refresh → logout) was not
  re-verified in this pass — no test credentials were available in this
  environment — and is instead covered by `route-guard.test.ts`'s
  mocked-Supabase behavioral assertions for the authenticated branch. Full
  Vitest (488/488), `tsc --noEmit`, ESLint, and `next build` all pass in the
  isolated worktree.
- **`feat/budgeting`** (backend) → `pocket-mint-backend` PR #59 — two
  commits (Budget domain/API, then the Postgres CI gate). Local
  `describe.skipIf(!TEST_DATABASE_URL)` integration suite could not be
  re-run in this environment (no Docker/local Postgres available), so it was
  validated by the **real GitHub Actions run instead**: run
  [`29847807971`](https://github.com/acuyaldi/pocket-mint-backend/actions/runs/29847807971)
  on PR #59 completed **green** — `postgres:18` service started and passed
  its health check, `prisma migrate deploy` applied the `add_budget`
  migration to the disposable database, and the Test step executed with
  `TEST_DATABASE_URL` set (not skipped): **674/674 tests passed, 55/55 test
  files, zero skipped** — confirming the Budget integration suite (including
  the 32 tests referenced in Phase B4/B6) actually ran against a real
  Postgres instance in CI, not just locally. Typecheck, build, and the
  generated-file/`dist` consistency gate (`git diff --exit-code` after
  `npm run build`) also passed in that same run.
- **`feat/budgeting`** (frontend) → `pocket-mint-frontend` PR #50 — three
  commits (Budgeting UI, then the FormField and DockMorph fixes as
  independent regression fixes, each with its own test). Full Vitest
  (502/502), `tsc --noEmit`, ESLint, i18n key-parity (EN/ID), and
  `next build` all pass in the isolated worktree; build output confirms
  `/anggaran`, `/anggaran/[id]`, and `Proxy (Middleware)` are registered.
- **`docs/budgeting-readiness`** → `pocket-mint-docs` PR — this document and
  the other Budgeting documentation.

**Net status (end of Phase B7): READY FOR RELEASE PREPARATION.** The
condition that kept Budgeting at CONDITIONALLY READY in Phase B6 — an
unobserved GitHub Actions Postgres run — has been satisfied: the real run on
the backend PR is green, with the integration suite confirmed executed
(not skipped) and an exact passing count recorded above.

## Phase B8 — Merge to `dev` and Post-Merge Validation (2026-07-21)

All three implementation PRs merged to `dev`, in the required order (auth
route guard and backend Budgeting API before the frontend Budgeting PR that
depends on both), each with a standard merge commit and green checks at
merge time:

- `pocket-mint-frontend` PR #49 (route-guard fix) → merged to `dev`,
  commit `94d92f7b`.
- `pocket-mint-backend` PR #59 (Budget domain/API) → merged to `dev`,
  commit `74e26db1`. The merged CI run (`29847807971`) matched the PR's
  head SHA at merge time; no new run was required.
- `pocket-mint-frontend` PR #50 (Budgeting UI) → updated against `dev`
  post-auth-merge via a non-destructive merge commit (no rebase, no force
  push), revalidated (typecheck, ESLint, i18n parity, full Vitest, build all
  green on the combined state), then merged to `dev`, commit `dc38f1ef`.

Post-merge validation on the resulting `dev` branches:

- Backend `dev`: `prisma validate` ✅, `tsc --noEmit` ✅, `vitest run` —
  642/642 unit tests passed; the 32 Postgres integration tests could not be
  re-run locally (no Docker/local Postgres in this environment) but were
  already confirmed 674/674 (including these 32) against a real disposable
  Postgres in CI run `29847807971` on the exact merged head. `npm run build`
  ✅, `git diff --check` ✅.
- Frontend `dev`: `tsc --noEmit` ✅, `eslint .` ✅, i18n EN/ID key parity ✅,
  `npm run build` ✅ (`/anggaran`, `/anggaran/[id]`, and `Proxy (Middleware)`
  all present in the route output), `git diff --check` ✅. `vitest run` —
  528/530 passed; the 2 failures are a local Windows `core.autocrlf`
  checkout artifact in a source-text-matching test (CRLF vs the LF the test
  literal expects), not a real regression — the same suite passed in this
  PR's own CI run (Linux/LF) after the merge.
- A live backend+frontend integration smoke test (server start, login,
  `/anggaran` data fetch) was not performed: it requires real Supabase
  credentials, which are not safely available in this environment — same
  constraint as the authenticated browser re-verification below. This
  remains a pre-release/deployment verification item, not fabricated as
  passed.

Remaining before an actual release: merge this docs PR, then an
authenticated-context browser re-verification with real credentials at
whatever point real credentials become available — the mocked behavioral
test coverage (`route-guard.test.ts`) is a substitute, not a replacement,
for that verification. Production deployment (Railway/Vercel) and the
production Prisma migration have **not** been performed and are not in
scope of this document.
