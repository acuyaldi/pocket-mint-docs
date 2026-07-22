# Pocket Mint Analytics v2 API Contract

> Status: Implemented in `pocket-mint-be` on branch `feature/analytics-v2`, PR open against `dev`, **not merged**. Governed by [ADR-010 — Analytics v2 Architecture](../product/decisions/010-analytics-v2-architecture.md). Companion to the [Analytics v2 Calculation Specification](./analytics-v2-calculation-spec.md), which defines the underlying formulas this contract exposes.
> Conventions followed (mirrors [Budgeting API Contract](./budgeting-api-contract.md)): static controller class (`AnalyticsController`, `src/controllers/analytics.controller.ts`), `sendSuccess`/`sendError` envelope, `forwardError` + typed `AnalyticsError`, `requireUser` on every route, `getAuthenticatedUserId(req)` — never a client-supplied `userId`.
> Ground truth: `src/routes/analyticsRoutes.ts`, `src/controllers/analytics.controller.ts`, `src/services/analytics-period.ts`, `src/services/analytics-overview.service.ts`, `src/services/analytics.errors.ts` (read directly).

---

## Response Envelope

Success: `{ "success": true, "data": T, "message": string }`
Error: `{ "success": false, "error": { "code": string, "statusCode": number, "message": string } }`

The existing app-wide envelope (`src/utils/response.ts`). No Analytics-specific envelope is introduced.

---

## Auth and Rate Limiting

All six endpoints are mounted at `/api/v1/analytics` (`src/routes/index.ts`: `router.use('/v1/analytics', analyticsRouter)`) and are **read-only `GET`s**, each gated by `requireUser`. `401 UNAUTHORIZED` if there is no authenticated caller. Every result is scoped to `getAuthenticatedUserId(req)` — no cross-user data is ever reachable.

No `mutationLimiter` is applied — these are queries, not mutations. `generalLimiter` (applied globally in `app.ts`) already covers them, matching the precedent set by `GET /dashboard/summary`, `GET /transactions/summary`, and `GET /budgets`.

---

## Shared Query Parameters (`overview`, `trends`, `categories`, `wallets`, `transactions`)

| Param | Type | Default | Notes |
|---|---|---|---|
| `period` | `current-month \| previous-month \| last-3-months \| last-6-months \| current-year \| custom` | `current-month` | See [Calculation Spec §4](./analytics-v2-calculation-spec.md#4-period-and-timezone-policy). |
| `startDate` | `YYYY-MM-DD` | — | Required with `endDate` when `period=custom`. |
| `endDate` | `YYYY-MM-DD` | — | Required with `startDate` when `period=custom`. |

`GET /budget-performance` accepts **no period parameter at all** — it is always scoped to the current reporting month (see Calculation Spec §7).

### Error behavior for period resolution

Every endpoint that resolves a period routes through the single `resolvePeriodOrThrow` translation point, so these are identical across all of them:

| Condition | Status | Code |
|---|---|---|
| Unrecognized `period` value | 400 | `BAD_REQUEST` |
| `period=custom` missing `startDate` or `endDate` | 400 | `BAD_REQUEST` |
| `startDate`/`endDate` not `YYYY-MM-DD` | 400 | `BAD_REQUEST` |
| `startDate` after `endDate` | 400 | `BAD_REQUEST` |
| Custom range exceeds 400 days | 400 | `BAD_REQUEST` |
| No authenticated user | 401 | `UNAUTHORIZED` |

No `AnalyticsError` code is more granular than `BAD_REQUEST` for period problems — the message text (not a machine code) distinguishes the specific validation failure, matching this endpoint family's read-only, low-stakes nature (unlike Budgeting's mutation-path error codes, which the frontend keys UI copy on).

---

## `GET /api/v1/analytics/overview`

Current-period income/expense/net-cash-flow totals plus a comparison against the immediately preceding period of equal duration.

```json
GET /api/v1/analytics/overview?period=current-month

{
  "success": true,
  "data": {
    "period": "current-month",
    "periodStart": "2026-06-30T17:00:00.000Z",
    "periodEnd": "2026-07-31T17:00:00.000Z",
    "income": 8000000,
    "expense": 5200000,
    "netCashFlow": 2800000,
    "transactionCount": 42,
    "previousPeriod": {
      "periodStart": "2026-05-31T17:00:00.000Z",
      "periodEnd": "2026-06-30T17:00:00.000Z",
      "income": 7500000,
      "expense": 6000000,
      "netCashFlow": 1500000
    },
    "change": {
      "income": 500000,
      "expense": -800000,
      "netCashFlow": 1300000
    },
    "percentageChange": {
      "income": { "value": 6.6667, "reason": null },
      "expense": { "value": -13.3333, "reason": null },
      "netCashFlow": { "value": 86.6667, "reason": null }
    }
  },
  "message": "Retrieved analytics overview"
}
```

If a previous-period value is zero, its `percentageChange` entry is `{ "value": null, "reason": "ZERO_BASELINE" }` (see Calculation Spec §5). `transactionCount` counts `INCOME`+`EXPENSE` rows only (`TRANSFER` excluded, per Calculation Spec §2).

---

## `GET /api/v1/analytics/trends`

Zero-filled, contiguous income/expense/net-cash-flow buckets covering the resolved period, at daily or monthly granularity (Calculation Spec §6).

```json
GET /api/v1/analytics/trends?period=last-6-months

{
  "success": true,
  "data": {
    "period": "last-6-months",
    "periodStart": "2026-02-01T00:00:00.000Z",
    "periodEnd": "2026-08-01T00:00:00.000Z",
    "granularity": "month",
    "buckets": [
      { "start": "2026-02-01T00:00:00.000Z", "end": "2026-03-01T00:00:00.000Z", "income": 8000000, "expense": 5200000, "netCashFlow": 2800000 },
      { "start": "2026-03-01T00:00:00.000Z", "end": "2026-04-01T00:00:00.000Z", "income": 0, "expense": 0, "netCashFlow": 0 }
    ]
  },
  "message": "Retrieved analytics trends"
}
```

A bucket with no matching transactions still appears with all-zero values — never omitted.

---

## `GET /api/v1/analytics/categories`

Category breakdown for one transaction type in the resolved period.

- **Query:** `type` — `EXPENSE` (default) | `INCOME`. Plus the shared period params.

```json
GET /api/v1/analytics/categories?period=current-month&type=EXPENSE

{
  "success": true,
  "data": {
    "period": "current-month",
    "periodStart": "2026-06-30T17:00:00.000Z",
    "periodEnd": "2026-07-31T17:00:00.000Z",
    "type": "EXPENSE",
    "total": 5200000,
    "categories": [
      { "categoryId": "clxcat001", "name": "Makan", "amount": 1500000, "transactionCount": 12, "percentage": 28.8462 },
      { "categoryId": null, "name": "Uncategorized", "amount": 200000, "transactionCount": 2, "percentage": 3.8462 }
    ]
  },
  "message": "Retrieved category breakdown"
}
```

`percentage` is `amount / total * 100`, exact (unrounded) `Decimal` division, `null` only in the defensive `total === 0` branch (empty period). A `categoryId: null` row is the explicit `"Uncategorized"` group (localized name resolved by the caller/frontend from the `null` id, not embedded server-side as a magic string) — never silently dropped (Calculation Spec §3).

---

## `GET /api/v1/analytics/wallets`

Per-wallet income/expense/net-cash-flow breakdown for the resolved period.

```json
GET /api/v1/analytics/wallets?period=current-month

{
  "success": true,
  "data": {
    "period": "current-month",
    "periodStart": "2026-06-30T17:00:00.000Z",
    "periodEnd": "2026-07-31T17:00:00.000Z",
    "wallets": [
      { "id": "clxwallet001", "name": "BCA", "income": 8000000, "expense": 3000000, "netCashFlow": 5000000, "transactionCount": 20 }
    ]
  },
  "message": "Retrieved wallet breakdown"
}
```

---

## `GET /api/v1/analytics/budget-performance`

Current-month Budget usage — **numerically identical** to `GET /budgets` for the same user at the same instant (Calculation Spec §7). No `period` param.

```json
GET /api/v1/analytics/budget-performance

{
  "success": true,
  "data": [
    {
      "id": "clx1budget001",
      "category": { "id": "clxcat001", "name": "Makan", "type": "EXPENSE" },
      "limit": 2000000,
      "spent": 1500000,
      "remaining": 500000,
      "percentUsed": 75,
      "status": "APPROACHING",
      "isArchived": false,
      "periodStart": "2026-06-30T17:00:00.000Z",
      "periodEnd": "2026-07-31T17:00:00.000Z"
    }
  ],
  "message": "Retrieved budget performance"
}
```

Note the field is `limit` here (not `amount`, as in `BudgetDto`) — a naming difference in the serialization only; the underlying value and formula are the same (see [Calculation Spec §7](./analytics-v2-calculation-spec.md#7-budget-performance-reuse-policy)). Only active (non-archived) budgets are returned, matching `GET /budgets?status=active`'s default.

---

## `GET /api/v1/analytics/transactions` (drill-down)

Paginated transaction list scoped to a resolved period, reusing the canonical transaction shape from `transaction.controller.ts`'s `serializeTransaction` — **no second transaction DTO.**

- **Query:** shared period params, plus:
  - `type` — `INCOME | EXPENSE | TRANSFER`, optional.
  - `categoryId` — optional.
  - `walletId` — optional.
  - `page` — optional, default `1`, clamped to a minimum of `1`.
  - `limit` — optional, default `20`, clamped to `[1, 200]`. **200 is the hard cap regardless of the requested value** — a client requesting `limit=10000` silently receives `limit=200`, not an error.

```json
GET /api/v1/analytics/transactions?period=current-month&type=EXPENSE&page=1&limit=20

{
  "success": true,
  "data": {
    "period": "current-month",
    "periodStart": "2026-06-30T17:00:00.000Z",
    "periodEnd": "2026-07-31T17:00:00.000Z",
    "transactions": [ /* canonical Transaction shape, see transaction.controller.ts */ ],
    "pagination": { "page": 1, "limit": 20, "total": 42, "totalPages": 3 }
  },
  "message": "Retrieved transactions"
}
```

`totalPages` is `max(ceil(total / limit), 1)` — never `0`, even for an empty result set, so a client can always render "Page 1 of 1" rather than dividing by zero or hiding pagination inconsistently.

---

## Error Code Summary

| Code | Status | Meaning |
|---|---|---|
| `BAD_REQUEST` | 400 | Malformed/invalid `period`, missing or invalid custom-range bounds, inverted range, or range exceeding 400 days. Message text distinguishes the specific cause. |
| `UNAUTHORIZED` | 401 | No authenticated user. |
| `TOO_MANY_REQUESTS` | 429 | `generalLimiter` rate limit hit (shared, not Analytics-specific). |

All codes follow the existing `sendError(res, message, statusCode, code)` shape. `AnalyticsError` (`src/services/analytics.errors.ts`) carries `message`, `statusCode`, `code` and is recognized automatically by `forwardError`'s structural `isOperational` check, mirroring `BudgetError`/`TransactionError` — no new registration mechanism.

---

## Frontend Consumption Status

`pocket-mint-fe`'s `/analytics` page (shared-named branch `feature/analytics-v2`, PR open against `dev`) now calls all six endpoints documented above. The page is rewritten from client-side full-transaction-dump aggregation (`useAllTransactions` + `buildMonthlyFlow`/`filterTransactionsByPeriod`) to backend-authoritative aggregation via the dedicated `useAnalytics*` TanStack Query hooks. See the [Implementation Roadmap's Phase 11 section](./implementation-roadmap.md#phase-11--analytics-v2) and the [Calculation Spec's Known Limitations](./analytics-v2-calculation-spec.md#8-known-limitations-out-of-scope-for-this-phase) for the full account.
