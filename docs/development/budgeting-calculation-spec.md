# Pocket Mint Budgeting Calculation Specification

> Status: Approved — governed by [PD-009](../product/decisions/009-budgeting-scope.md) (Approved). Phase A (domain foundation) implements this spec's calculation service; API, frontend, and notification integration remain pending.
> Scope: deterministic formulas for backend implementation and shared backend/frontend test fixtures
> Reuses: `pocket-mint-be/src/domain/reportingTime.ts` (`getReportingMonthRange`), the DB-`groupBy` aggregation pattern from `transaction-query.service.ts`

This document is deterministic enough that a backend unit test and a frontend contract test can assert the same expected numbers for the same fixture data. It defines no new business rule beyond what [PD-009](../product/decisions/009-budgeting-scope.md) already decided — it makes those rules computable.

---

## 1. Period Resolution

```
period = getReportingMonthRange(now, REPORTING_TIMEZONE)
       = { startInclusive, endExclusive }
```

- `startInclusive` = local midnight of the 1st of the current calendar month in `REPORTING_TIMEZONE`.
- `endExclusive` = local midnight of the 1st of the following calendar month in `REPORTING_TIMEZONE`.
- A transaction belongs to the period iff `startInclusive <= transaction.date < endExclusive`.
- Never use `lte` on the end bound. Never construct the boundary with `new Date(y, m, 1)` (server-timezone dependent); always call `getReportingMonthRange`.

**Example** (REPORTING_TIMEZONE = `Asia/Jakarta`, UTC+7):

- `now` = 2026-07-21T10:00:00Z (17:00 WIB)
- `startInclusive` = 2026-06-30T17:00:00Z (2026-07-01T00:00:00 WIB)
- `endExclusive` = 2026-07-31T17:00:00Z (2026-08-01T00:00:00 WIB)
- A transaction dated `2026-07-31T23:00:00+07:00` (still July 31 in WIB) is **included**.
- A transaction dated `2026-08-01T00:30:00+07:00` is **excluded** — belongs to August's period.

---

## 2. Spent Amount

```
Budget Spent(budget, period)
  = SUM(t.amount)
    FOR t IN Transaction
    WHERE t.userId = budget.userId
      AND t.categoryId = budget.categoryId
      AND t.type = 'EXPENSE'
      AND t.date >= period.startInclusive
      AND t.date <  period.endExclusive
```

Implementation: a single `db.transaction.groupBy({ by: ['categoryId'], where: {...}, _sum: { amount: true } })` call per user (all of a user's active budgets resolved in one query), mirroring `transaction-query.service.ts`'s existing pattern. Never sum in application code by fetching every row (Readiness Audit §5).

**Installment purchases:** `t.amount` for an installment `EXPENSE` row is the persisted `monthlyAmount`, not `grandTotal` (Readiness Audit §1, PD-009 Decision C). This formula uses the persisted column as-is — no special-casing, no lookup of the related `Installment.grandTotal`.

**Transfers, income, unconfirmed recurring templates:** never enter this query (`type = 'EXPENSE'` filter excludes transfers and income by construction; unconfirmed templates never produce a `Transaction` row at all).

**Refunds:** not representable in the current schema (no negative-expense concept, `amount > 0` validated at write time). Not part of this formula. If introduced later, the formula becomes `SUM(t.amount * sign)` and must be re-specified in a new PD, not assumed here.

---

## 3. Remaining Amount

```
Budget Remaining(budget, period) = budget.amount - Budget Spent(budget, period)
```

- May be negative. A negative Remaining is the deterministic signal for `EXCEEDED` (see §5) and must be displayed as a negative number, never clamped to zero (consistent with Pocket Mint's existing Negative Net Worth precedent: never hide a true negative value).

---

## 4. Percentage Used

```
percentUsed(budget, period) =
    IF budget.amount == 0 THEN undefined   -- amount is validated > 0 at write time; this branch should be unreachable in practice
    ELSE (Budget Spent(budget, period) / budget.amount) * 100
```

- Computed as an exact `Decimal` division, then rounded to the nearest integer **only for display**. The stored/compared value for threshold evaluation (§5) uses the unrounded ratio, so a spend of `749,999 / 1,000,000 = 74.9999%` does not round up to a false `75%` `APPROACHING` state.
- `percentUsed` may exceed 100.

**Example:**

- `budget.amount = 1,000,000`, `Budget Spent = 750,000` → `percentUsed = 75.0000` → displayed `75%`, status `APPROACHING` (boundary is inclusive, see §5).
- `budget.amount = 1,000,000`, `Budget Spent = 1,250,000` → `percentUsed = 125.0000` → displayed `125%`, `Budget Remaining = -250,000`, status `EXCEEDED`.

---

## 5. Budget Status

```
status(budget, percentUsed) =
    IF budget.isArchived THEN 'ARCHIVED'
    ELSE IF percentUsed > 100 THEN 'EXCEEDED'
    ELSE IF percentUsed == 100 THEN 'REACHED'
    ELSE IF percentUsed >= 75  THEN 'APPROACHING'
    ELSE 'HEALTHY'
```

Five deterministic states ([PD-009](../product/decisions/009-budgeting-scope.md) Decision E, Approved). `EXCEEDED` is strictly greater than 100 — exactly 100% is its own state, `REACHED`, never reported as `EXCEEDED` or `APPROACHING`. Evaluated in this exact order (`ARCHIVED`, then `EXCEEDED`, then `REACHED`, then `APPROACHING`, else `HEALTHY`).

**Superseded rule:** an earlier Draft of this spec defined only four states with `EXCEEDED` at `percentUsed >= 100` (no `REACHED`). That rule is superseded by PD-009's Approved five-state model above; no implementation may use the four-state rule.

**Boundary test table:**

| Budget Spent | Budget Amount | percentUsed | Status |
|---|---|---|---|
| 0 | 1,000,000 | 0% | `HEALTHY` |
| 749,999 | 1,000,000 | 74.9999% | `HEALTHY` |
| 750,000 | 1,000,000 | 75.0000% | `APPROACHING` |
| 999,999 | 1,000,000 | 99.9999% | `APPROACHING` |
| 1,000,000 | 1,000,000 | 100.0000% | `REACHED` |
| 1,000,001 | 1,000,000 | 100.0001% | `EXCEEDED` |
| 1,500,000 | 1,000,000 | 150.0000% | `EXCEEDED` |
| (any) | (any), `isArchived: true` | — | `ARCHIVED` |

---

## 6. Threshold Notification Evaluation

> Not implemented in Phase A ([PD-009](../product/decisions/009-budgeting-scope.md) Decision F). Recorded here for the later notification-integration phase; no `BudgetThresholdEvent` model or evaluation code exists yet.

Two thresholds only — 75% and 100% — not one event per Budget Status. `REACHED` and `EXCEEDED` are two distinct statuses (§5) but share one 100% `thresholdType`, so crossing from 100.0000% to a higher percentage never fires a second event.

Evaluated whenever a mutation could change a user's Budget Spent for the current period (transaction create, update that changes `categoryId`/`amount`/`date`/`type` to or from a match, or delete of a previously-matching transaction).

```
FOR EACH active (non-archived) budget affected by the mutation:
    percentUsed = recompute per §4
    newStatus   = recompute per §5

    IF newStatus IN ('REACHED', 'EXCEEDED'):
        UPSERT BudgetThresholdEvent
            WHERE (budgetId, periodStart = period.startInclusive, thresholdType = 'EXCEEDED')
            ON CONFLICT DO NOTHING   -- idempotent; never re-fires within the same period,
                                     -- even as percentUsed keeps climbing past 100%
    ELSE IF newStatus == 'APPROACHING':
        UPSERT BudgetThresholdEvent
            WHERE (budgetId, periodStart = period.startInclusive, thresholdType = 'APPROACHING')
            ON CONFLICT DO NOTHING
```

- A budget crossing directly from `HEALTHY` to `EXCEEDED` in one mutation (e.g. a single large expense) creates **both** the `APPROACHING` and `EXCEEDED`(100%) `BudgetThresholdEvent` rows in the same evaluation pass — a user should still see they crossed 75% on the way to 100%, not just the final state.
- Dropping back below a threshold (edit/delete reduces spend) does **not** delete or retract an already-created `BudgetThresholdEvent` (PD-009 Decision F).
- No event is created for `HEALTHY` or `ARCHIVED`.
- **Failure isolation:** a failure creating/upserting a `BudgetThresholdEvent` must never fail or roll back the transaction mutation that triggered evaluation (PD-009 Decision F).

---

## 7. Dashboard/Summary Aggregation

```
activeBudgetCount = COUNT(Budget WHERE userId = X AND isArchived = false)
totalBudgetAmount = SUM(Budget.amount WHERE userId = X AND isArchived = false)
totalBudgetSpent  = SUM(Budget Spent(b, currentPeriod) FOR EACH active budget b)
budgetsExceeded   = COUNT(active budgets WHERE status IN ('REACHED', 'EXCEEDED'))
```

These are simple aggregates over the per-budget results from §2–§5, computed in the same request, not a separately cached figure. If `activeBudgetCount == 0`, the dashboard section is not shown (empty state, not a zero-value section) per the existing Pocket Mint convention of distinguishing empty from zero.

---

## 8. Worked End-to-End Example

Setup: user has one active Budget for category "Makan" (Food), `amount = 2,000,000`, `REPORTING_TIMEZONE = Asia/Jakarta`. Current date: 2026-07-21.

Transactions in July (WIB dates):

| Date | Type | Category | Amount | Counts? |
|---|---|---|---|---|
| 2026-07-02 | EXPENSE | Makan | 300,000 | Yes |
| 2026-07-05 | EXPENSE | Transport | 100,000 | No — different category |
| 2026-07-10 | EXPENSE | Makan | 500,000 | Yes |
| 2026-07-12 | TRANSFER | — | 1,000,000 | No — transfer |
| 2026-07-15 | EXPENSE | Makan (installment purchase, `monthlyAmount`) | 700,000 | Yes, at 700,000 (not `grandTotal`) |
| 2026-07-18 | EXPENSE | Makan | 300,000 (later deleted by user) | No — deleted before period end, absent from live query |
| 2026-08-01 | EXPENSE | Makan | 900,000 | No — belongs to August's period |

```
Budget Spent    = 300,000 + 500,000 + 700,000 = 1,500,000
Budget Remaining = 2,000,000 - 1,500,000       = 500,000
percentUsed      = 1,500,000 / 2,000,000 * 100 = 75.0000%
status           = APPROACHING (>= 75, < 100)
```

`BudgetThresholdEvent` created: `(budgetId, periodStart = 2026-07-01T00:00:00+07:00, thresholdType = APPROACHING)`.

If the user then deletes the 2026-07-10 transaction (500,000):

```
Budget Spent    = 300,000 + 700,000 = 1,000,000
percentUsed      = 50.0000%
status           = HEALTHY
```

The `APPROACHING` `BudgetThresholdEvent` created earlier is **not deleted** (§6, PD-009 Decision F) — it remains as a historical fact that the threshold was crossed on 2026-07-21, even though current spend has since dropped. Budget Detail always shows the live `HEALTHY` numbers regardless.

---

## Risk Register

| Risk | Likelihood | Impact | Mitigation | Blocks MVP? |
|---|---|---|---|---|
| Non-Goals conflict ("budgeting-envelope") not resolved before implementation begins | Medium | High — implementation would violate stated product boundary | PD-009 must be reviewed and approved, with the specific reworded Non-Goals text, before Phase B backend work starts. | **Yes**, until PD-009 is approved. |
| Installment `monthlyAmount` vs `grandTotal` ambiguity re-litigated differently in frontend vs backend | Medium | Medium — a budget could show a different "spent" figure than the Dashboard's monthly summary for the same data | §2 of this spec is the single source; both backend and frontend must cite it, never re-derive independently. | No, if this spec is followed. |
| Category model insufficient (no custom category creation) frustrates users who want to budget a category that doesn't exist | Low | Low — workaround is picking the nearest seeded category | Explicitly scoped as a non-blocking, optional Phase A improvement in PD-009 Decision D. | No. |
| Assumed background scheduler for threshold evaluation that doesn't actually exist | Medium | Medium — notifications silently never fire if design assumes cron | §6 evaluates synchronously inside the mutation path, not a scheduled job; scheduler existence is `UNVERIFIED` per the Readiness Audit and must not be assumed. | No, if synchronous evaluation is implemented as specified. |
| Slow aggregation as transaction volume grows | Low | Medium | §2 mandates DB `groupBy`, the same pattern already proven at existing scale; add `@@index([userId, categoryId, date])` consideration at implementation time if query plans show it's needed (existing `Transaction` indexes already cover `[userId,type]`, `[categoryId]`). | No. |
| Duplicated threshold notifications | Low | Low | §6's unique-constraint + upsert pattern is a proven idempotency mechanism reused verbatim from `RecurringReminderEvent`. | No. |
| Frontend/backend calculation drift (percentage rounding, boundary inclusivity) | Medium | Medium | §4–§5's boundary table is the shared fixture; both backend unit tests and frontend contract tests must assert against the same table. | No. |
| Historical values appearing to change when a budget's amount is edited | Low (v1) | Medium if it occurred | Avoided structurally in v1 by never displaying historical periods (PD-009 Decision J) — the risk only becomes live when historical browsing is added, at which point a `BudgetPeriod` snapshot must be introduced. | No, for v1 scope only. |
| Over-scoping rollover or custom periods into v1 | Low | Medium — delays ship date | PD-009 Decisions B and G explicitly defer both. | No, if PD-009 is followed as scoped. |
| Mixing forecast (recurring template) amounts into "spent" | Low | High if it occurred — misleads the user about actual spend | §2's formula only ever reads posted `Transaction` rows; recurring templates structurally cannot contribute (Readiness Audit §7). | No. |
