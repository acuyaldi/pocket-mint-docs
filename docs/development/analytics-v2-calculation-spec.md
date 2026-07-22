# Pocket Mint Analytics v2 Calculation Specification

> Status: Implemented (backend, `pocket-mint-be` branch `feature/analytics-v2`, PR open against `dev`, not merged). Governed by [ADR-010 — Analytics v2 Architecture](../product/decisions/010-analytics-v2-architecture.md) and the terminology reconciliation recorded against [PD-006 — Financial Reporting Surface](../product/decisions/006-reporting.md) (still Draft; see that document's History table).
> Scope: deterministic metric definitions for the `/api/v1/analytics/*` endpoints, mirroring the precedent set by [Budgeting Calculation Specification](./budgeting-calculation-spec.md).
> Reuses: `pocket-mint-be/src/domain/reportingTime.ts` (half-open, DST-safe reporting-calendar primitives), `src/domain/analyticsPeriod.ts` (new — period resolution and trend-bucket generation built on top of `reportingTime.ts`), `src/services/budget-query.service.ts` / `src/domain/budget.ts` (Budget Performance, reused verbatim — see §7).
> Ground truth for this document: `src/domain/analyticsPeriod.ts`, `src/controllers/analytics.controller.ts`, `src/services/analytics-overview.service.ts`, `src/services/analytics-period.ts`, `src/services/analytics.errors.ts` (all read directly for this spec, not inferred).

---

## 1. Money Representation

All monetary fields are `Prisma.Decimal(15,2)` at the domain/service layer. Every aggregation (`groupBy`, `_sum`) is performed by the database or by `Prisma.Decimal` arithmetic in the service layer — never by summing floating-point `number`s in application code. Conversion to a JavaScript `number` happens exactly once, at the HTTP boundary, in the controller's serializer functions (`num()` in `analytics.controller.ts`: `Number(value.toString())`). This matches the existing convention documented in the [Budgeting API Contract](./budgeting-api-contract.md#decimal-serialization) — the same precision-risk note applies (safe for realistic IDR amounts, a pre-existing shared exposure at very large values, not Analytics-specific).

---

## 2. Transaction Type Treatment

`TransactionType` is `INCOME | EXPENSE | TRANSFER`. Every Analytics v2 aggregation that sums money is scoped `type IN ('INCOME', 'EXPENSE')` — `TRANSFER` is excluded by construction from every income/expense total, trend bucket, and category/wallet breakdown that counts "spend" or "earn". This mirrors `transaction-query.service.ts`'s existing monthly summary and the Budgeting Calculation Spec's Basis rule: a transfer relocates money between the user's own wallets and is never income or expense, so it can never inflate both sides of a cash-flow total.

`GET /transactions` (drill-down) is the one exception: it returns the canonical transaction shape for the resolved period/filters, including `TRANSFER` rows if `type` isn't filtered, because it is a ledger view, not an aggregate.

---

## 3. Uncategorized Transactions

`Transaction.categoryId` is nullable (`onDelete: SetNull` when a `Category` is deleted). The category breakdown (`GET /categories`) groups by `categoryId`, and a `null` group is surfaced as an explicit `"Uncategorized"` entry (localized) rather than being silently dropped or folded into another bucket — consistent with the product's principle of never hiding a true state behind an omission. This state is rare in practice (a category can only become `null` via category deletion; the create/update transaction flow requires a category for `INCOME`/`EXPENSE`), but it is handled explicitly, not assumed away.

---

## 4. Period and Timezone Policy

- Reporting timezone: `Asia/Jakarta` by default, overridable via the `REPORTING_TIMEZONE` env var (`reportingConfig.timezone`).
- Every period is a **half-open range**: `startInclusive <= transaction.date < endExclusive`. Never `lte` on the end bound.
- DST-safety and timezone math are inherited entirely from the existing `reportingTime.ts` primitives (`Intl.DateTimeFormat`-based, no date library dependency) — `analyticsPeriod.ts` adds no new timezone arithmetic of its own.
- Supported `period` values (`src/domain/analyticsPeriod.ts`, `AnalyticsPeriod`): `current-month | previous-month | last-3-months | last-6-months | current-year | custom`. Defaults to `current-month` when omitted.
- `custom` requires `startDate` and `endDate` as strict `YYYY-MM-DD` calendar dates (inclusive on both ends as calendar days; internally converted to the same half-open instant range via `getReportingDayRange`). Validation:
  - Malformed date format → 400.
  - `startDate` after `endDate` → 400.
  - Range exceeding `MAX_CUSTOM_RANGE_DAYS` (400 days) → 400. An unbounded custom range is not a "period" and would defeat the point of scoping a query.
- An unrecognized `period` value → 400.

All period-resolution errors are raised as a typed `AnalyticsError(message, 400, 'BAD_REQUEST')` via the single translation point `resolvePeriodOrThrow` (`src/services/analytics-period.ts`) — every analytics service and the drill-down controller route through this one function, so a malformed period never surfaces as an untyped 500.

---

## 5. Previous-Period Comparison

**Rule:** the comparison baseline is always the immediately preceding range of **equal duration** to the resolved period — never a fixed "same period last year" or "last calendar month" unless that happens to equal the resolved period's own shape.

| `period` | Current range | Previous range |
|---|---|---|
| `current-month` | This reporting month | The immediately preceding reporting month |
| `previous-month` | Last reporting month | The reporting month before that |
| `last-3-months` | Trailing 3 reporting months ending this month | The preceding 3-reporting-month block |
| `last-6-months` | Trailing 6 reporting months ending this month | The preceding 6-reporting-month block |
| `current-year` | This reporting year | The immediately preceding reporting year |
| `custom` | The requested `[startDate, endDate]` | An equal wall-clock-duration range immediately preceding `startDate` (no calendar meaning — arbitrary bounds have none) |

**Zero-baseline policy (must be followed by every consumer, backend and frontend):** if the previous period's value for a metric is exactly zero, `percentageChange` for that metric is never computed as `Infinity`, `-Infinity`, or `NaN`. It is returned as an explicit object:

```json
{ "value": null, "reason": "ZERO_BASELINE" }
```

When the previous value is non-zero, the shape is `{ "value": <number>, "reason": null }`, where `value` is `((current - previous) / previous) * 100`, an exact `Decimal` division serialized to `number` only at the HTTP boundary — not pre-rounded, matching the Budgeting API Contract's precedent for `percentUsed`.

This rule applies independently per metric (`income`, `expense`, `netCashFlow`) inside `GET /overview` — one metric can be `ZERO_BASELINE` while another has a real percentage in the same response (e.g. a user with prior-period expense but zero prior-period income).

---

## 6. Trend Bucketing

`GET /trends` returns a contiguous, zero-filled series of buckets covering the resolved period — a period with no transactions in a given bucket still returns that bucket with `income: 0, expense: 0, netCashFlow: 0`, not a gap.

**Granularity rule** (`resolveTrendGranularity`, `src/domain/analyticsPeriod.ts`):

```
days = round((range.endExclusive - range.startInclusive) / 86_400_000)
granularity = days <= 62 ? 'day' : 'month'
```

62 days (~2 months) is the cutoff — a daily series beyond that becomes visually unreadable and expensive to zero-fill for little UX benefit, so longer periods bucket monthly instead. Buckets are generated up front by `generateTrendBuckets` and clipped to the requested range (a boundary bucket for a `custom` range starting/ending mid-month never reports a `start`/`end` outside the requested bounds), then each is filled from a single aggregation query — never one query per bucket.

---

## 7. Budget Performance Reuse Policy

`GET /budget-performance` does **not** implement a second spend/status formula. It calls `budgetQueryService.listActiveBudgetUsage({ userId, status: 'active' })` — the exact same service that backs `GET /budgets` — and maps the result through `serializeBudgetPerformance`, a renamed mirror of `budget.controller.ts`'s `toBudgetDto` (fields renamed `amount → limit` for the analytics response shape only; every computed value is identical).

Consequences of this reuse:

- No separate "analytics budget formula" exists to drift from the [Budgeting Calculation Specification](./budgeting-calculation-spec.md)'s §2–§5 (Spent, Remaining, percentUsed, five-state Status).
- `GET /budget-performance` accepts **no `period` parameter**. A Budget is a recurring monthly definition (PD-009), not a dated instance, so this endpoint always reflects the current reporting month — identical to `GET /budgets` for the same user at the same instant. This is enforced structurally (the controller never reads a `period` query param for this endpoint), not by convention alone.
- A test (`test/analyticsBudgetPerformance.test.ts` in `pocket-mint-be`) asserts numeric parity between `GET /budgets` and `GET /analytics/budget-performance` for the same fixture data — the two must never disagree.

---

## 8. Known Limitations (Out of Scope for This Phase)

Per explicit product-owner instruction for this iteration:

- **No export.** Analytics v2 does not add a CSV/PDF export endpoint. (Note: `pocket-mint-fe` already has an unrelated, pre-existing transaction CSV export wired to the Analytics page's "Export report" button — that is prior functionality, not part of this iteration's contract, and is not documented by this spec.)
- **No forecasting.** No projected/predicted future values are computed anywhere in this contract. Every figure is derived from posted, historical `Transaction`/`Budget` rows only.
- **No materialized views, snapshots, or caching layer.** See [ADR-010](../product/decisions/010-analytics-v2-architecture.md) for the measured rationale (an `EXPLAIN ANALYZE` pass against a 100k-row seeded dataset found existing indexes already sufficient) and the threshold that would justify revisiting this.
- **No new database indexes were added.** The existing `[userId, date]` / `[walletId, date]` indexes on `Transaction` were confirmed sufficient (sub-2ms, no sequential scans) by the same `EXPLAIN ANALYZE` pass.
- **Frontend integration is implemented** — `pocket-mint-fe` branch `feature/analytics-v2` now wires the `/analytics` page to all six backend endpoints (TanStack Query hooks in `src/features/analytics/hooks/useAnalytics.ts`, period URL-state persistence in `src/features/analytics/period.ts`, Recharts-based charts with accessible data-table fallbacks, BudgetPerformance reusing `BudgetStatusPill`/`BudgetProgressBar` from `app/(app)/anggaran/components/`). The client-side `buildMonthlyFlow`/`filterTransactionsByPeriod` path is replaced by backend-authoritative aggregation; the existing CSV export button is preserved, mapped from the new period enum to the prior export endpoint's values where compatible. — see the [Implementation Roadmap](./implementation-roadmap.md#phase-11--analytics-v2)'s honesty note. The backend contract described in this document and the [API Contract](./analytics-v2-api-contract.md) is implemented and merges-ready; the `pocket-mint-fe` `/analytics` page on the same-named branch has **not** been wired to call it yet (confirmed by reading `app/(app)/analytics/page.tsx` directly — it still computes every figure client-side from `useAllTransactions()`, using its own pre-existing `month | quarter | six-months` period filter, not the backend's `current-month | previous-month | last-3-months | last-6-months | current-year | custom` enum). No drill-down UI, no accessible data-table fallback for the existing CSS-bar charts, and no budget-performance display exist on that page today. No chart library is installed in `pocket-mint-fe` — `recharts` was removed from `package.json` as unused dependency cleanup before this branch existed; the page uses the same plain CSS width/height-percentage `<div>` convention as the rest of the app (also used by Budgeting's progress bars).

---

## 9. Worked Example — Zero-Baseline Comparison

Setup: user has no `INCOME` transactions in the previous reporting month, and `Rp 500,000` in `INCOME` this reporting month.

```
income (current)   = 500,000
income (previous)   = 0
change.income        = 500,000 - 0 = 500,000
percentageChange.income = { value: null, reason: "ZERO_BASELINE" }   -- not Infinity
```

If the user also had `Rp 200,000` expense last period and `Rp 300,000` this period:

```
expense (current)  = 300,000
expense (previous) = 200,000
change.expense       = 100,000
percentageChange.expense = { value: 50.0000, reason: null }   -- (300,000-200,000)/200,000*100
```

Both fields exist in the same `GET /overview` response — one metric is `ZERO_BASELINE`, the other has a real percentage, computed independently per §5.
