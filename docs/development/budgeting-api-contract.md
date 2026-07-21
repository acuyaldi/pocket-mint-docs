# Pocket Mint Budgeting API Contract

> Status: Finalized design, **not implemented**. Governed by [PD-009](../product/decisions/009-budgeting-scope.md) (Approved) and the [Budgeting Phase A.5 Design Review](./budgeting-phase-a5-design-review.md). Written to unblock Phase B (`src/controllers/budget.controller.ts`, `src/routes/budgetRoutes.ts`). No route or controller exists in `pocket-mint-be` yet — confirmed absent from `src/controllers/`, `src/routes/`, `src/models/` as of this review.
>
> Conventions followed (see `pocket-mint-be/src/controllers/savingGoal.controller.ts`, `src/routes/savingGoal.routes.ts`, `src/utils/response.ts`, `src/http/forwardError.ts`, `src/http/authContext.ts`): static controller class, `sendSuccess`/`sendError` envelope, `forwardError` + `BudgetError` (already implemented at `src/services/budget.errors.ts`), `requireUser` on every route, `mutationLimiter` on every mutating route, `getAuthenticatedUserId(req)` — never a client-supplied `userId`.

---

## Response Envelope

Success: `{ "success": true, "data": T, "message": string }`
Error: `{ "success": false, "error": { "code": string, "statusCode": number, "message": string } }`

This is the existing app-wide envelope (`src/utils/response.ts`). No Budgeting-specific envelope is introduced.

---

## Endpoints

### `GET /api/v1/budgets`

List the caller's Budgets with current-month usage.

- **Auth:** `requireUser`.
- **Query parameters:**
  - `status` — `active` (default) | `archived`. Selects which set of Budgets to return; there is no combined view, matching the Budgets screen's explicit Archived toggle (never both lists mixed in one response).
  - `month`, `year` — optional integers. Omitted defaults to the current Reporting Period. **Present only for parity with the existing query-service signature (`GetBudgetUsageInput`/`ListActiveBudgetUsageInput` already accept `month`/`year`)** — the Budgets screen never sends them in v1 (current-month-only UX, see the Period decision). Accepting but not requiring them avoids a breaking contract change if historical browsing ships later.
- **Response:** `data: BudgetDto[]`, ordered by `createdAt` ascending (matches `listActiveBudgetUsage`'s existing order — archived-list ordering follows the same query shape with `isArchived: true`).
- **Status codes:** `200`. Empty list is `200` with `data: []`, never `404`.
- **Archived records:** included only when `status=archived`.
- **Period/timezone:** resolved server-side via `getReportingMonthRange(now, REPORTING_TIMEZONE)`, identical to the existing `budget-query.service.ts` behavior.

```json
GET /api/v1/budgets?status=active

{
  "success": true,
  "data": [
    {
      "id": "clx1budget001",
      "category": { "id": "clxcat001", "name": "Makan", "type": "EXPENSE" },
      "amount": 2000000,
      "spent": 1500000,
      "remaining": 500000,
      "percentUsed": 75,
      "status": "APPROACHING",
      "isArchived": false,
      "periodStart": "2026-06-30T17:00:00.000Z",
      "periodEnd": "2026-07-31T17:00:00.000Z",
      "createdAt": "2026-06-01T02:00:00.000Z",
      "updatedAt": "2026-06-01T02:00:00.000Z"
    }
  ],
  "message": "Retrieved budgets"
}
```

No separate `GET /api/v1/budgets/archived` endpoint — one endpoint with `status` avoids two near-duplicate list handlers for what the query service already treats as one filter (`isArchived`).

---

### `GET /api/v1/budgets/:id`

Get one Budget with current-month usage. Backs Budget Detail's header figures; the contributing-transaction list is fetched separately (see below) rather than embedded, so this endpoint stays budget-specific and the transaction list is never duplicated behind two different queries.

- **Auth:** `requireUser`.
- **Path parameters:** `id` — Budget id.
- **Response:** `data: BudgetDto`.
- **Status codes:** `200`; `404 NOT_FOUND` if the Budget does not exist **or** is not owned by the caller (identical lookup shape to the existing `getBudgetUsage`'s `findFirst({ where: { id, userId } })` — a cross-user id never distinguishes "not found" from "not yours").
- **Ownership isolation:** enforced by the same `{ id, userId }` lookup; no separate authorization check, no `403` (avoids leaking existence of another user's Budget id).
- **Archived records:** included (Budget Detail must still work for an archived Budget reached from the Archived filter).

The **contributing-transaction list** for Budget Detail reuses the existing transaction list endpoint (not enumerated by this document — owned by the Transaction feature) filtered by `categoryId=<budget.category.id>` and the same current-month date range the `BudgetDto` reports (`periodStart`/`periodEnd`), so the two can never disagree about which period they describe. No new endpoint is added for this.

---

### `POST /api/v1/budgets`

Create a Budget.

- **Auth:** `requireUser`, `mutationLimiter`.
- **Request body:**

```json
{ "categoryId": "clxcat001", "amount": 2000000 }
```

- **Validation:**
  - `amount` — required, numeric, `> 0`, at most 2 decimal places (matches `Decimal(15,2)`).
  - `categoryId` — required, non-empty string.
  - `categoryId` must reference a `Category` owned by the caller with `type = EXPENSE`. Enforced application-side (no FK-level type constraint exists — same pattern already used by `transaction.service.ts` for category/type matching).
  - No existing `Budget` row (active or archived) may exist for `(userId, categoryId)`.
- **Response:** `data: BudgetDto` for the newly created Budget (zero spend, `HEALTHY`, current period).
- **Status codes:** `201` on success.
- **Errors:**
  - `400 BAD_REQUEST` — missing/malformed `amount` or `categoryId`.
  - `422 INVALID_AMOUNT` — `amount <= 0` or more than 2 decimal places.
  - `404 CATEGORY_NOT_FOUND` — `categoryId` does not exist or is not owned by the caller. Identical response whether the category doesn't exist or belongs to another user (never `403`, never distinguishes the two).
  - `422 CATEGORY_NOT_EXPENSE` — `categoryId` resolves to an `INCOME` category.
  - `409 BUDGET_ALREADY_EXISTS` — a Budget (active or archived) already exists for this category. Message must not imply the row is retrievable by the caller through this endpoint; it directs the user to Budgets to find/restore the existing one.
- **Idempotency:** none. A retried `POST` after a network timeout that actually succeeded server-side surfaces as `409 BUDGET_ALREADY_EXISTS` on the client's retry — acceptable because the error is actionable (the Frontend can refetch the list and show the existing Budget) and no financial mutation risk exists (unlike a Transaction, a duplicate `POST` cannot double-count money).

---

### `PATCH /api/v1/budgets/:id`

Update a Budget's amount. Category and archive state are not editable through this endpoint.

- **Auth:** `requireUser`, `mutationLimiter`.
- **Request body:**

```json
{ "amount": 2500000 }
```

- **Validation:**
  - `amount` — required, numeric, `> 0`, at most 2 decimal places.
  - If the request body includes `categoryId` at all (even matching the current value), reject with `422 CATEGORY_NOT_EDITABLE` — an explicit rejection, not a silent ignore, so a client bug attempting reassignment is caught immediately rather than silently no-op'd.
  - A no-op update (`amount` unchanged) is accepted and returns `200` with the unchanged `BudgetDto` — not rejected, not specially flagged.
  - Updating an archived Budget's amount is **allowed** (the row still exists and may be restored later with the new amount already applied) — the response's `status` remains `ARCHIVED` regardless of the new amount.
- **Response:** `data: BudgetDto`, recalculated against the current period.
- **Status codes:** `200`.
- **Errors:** `400 BAD_REQUEST`, `422 INVALID_AMOUNT`, `422 CATEGORY_NOT_EDITABLE`, `404 NOT_FOUND` (not found or not owned — same pattern as `GET /budgets/:id`).
- **Optimistic concurrency:** none — no existing Pocket Mint mutation endpoint uses one (confirmed by `savingGoal`/`transaction` update patterns), so Budgeting does not introduce a new concept unrequested by the rest of the app. Last write wins.

---

### `POST /api/v1/budgets/:id/archive`

- **Auth:** `requireUser`, `mutationLimiter`.
- **Request body:** none.
- **Response:** `data: BudgetDto` with `isArchived: true`, `status: "ARCHIVED"`.
- **Status codes:** `200`.
- **Errors:** `404 NOT_FOUND` (not found/not owned); `409 ALREADY_ARCHIVED` if already archived — an explicit conflict, not a silent no-op, so the UI never shows an "Archive" action succeeding twice without the caller knowing its click was a no-op.

### `POST /api/v1/budgets/:id/restore`

- **Auth:** `requireUser`, `mutationLimiter`.
- **Request body:** none.
- **Response:** `data: BudgetDto` with `isArchived: false`, status recalculated against the current period's live usage (an archived Budget can be restored directly into `EXCEEDED` if spend already happened this period against that category via a different mechanism — unlikely but not structurally impossible, and the response must reflect it honestly rather than defaulting to `HEALTHY`).
- **Status codes:** `200`.
- **Errors:** `404 NOT_FOUND`; `409 ALREADY_ACTIVE` if not archived.

### No `DELETE /api/v1/budgets/:id`

Hard delete is **not exposed** in v1. See the [Archive/Delete decision](./budgeting-phase-a5-design-review.md#10-archivedeletecategory-reassignment-decisions) — archive/restore is sufficient, and PD-009 Decision L already means archiving never blocks reuse the way a delete-and-recreate cycle would. Deferred, not rejected — a delete endpoint may be added later for a never-used Budget if a validated need emerges (no core journey requires it now).

### No dedicated "eligible categories" endpoint

Create Budget's Category selector is populated by the Frontend from the existing `GET /api/v1/categories` (all of the user's categories) filtered client-side to `type === 'EXPENSE'` and with `category.id` in `GET /api/v1/budgets?status=active` **and** `?status=archived` removed (uniqueness applies to both — Decision L). No new backend endpoint, per the "avoid endpoints not required by the final journeys" instruction — the existing `GET /categories` and the two Budget list calls (already needed for the Budgets screen) fully support this without a race-prone third call pattern. If category counts grow materially in the future, this can move server-side without changing the response shape Create Budget consumes.

---

## `BudgetDto`

One DTO for list, detail, and every mutation response — not split into Summary/Detail/Mutation variants. Every field the UI needs (list row, detail header, post-mutation confirmation) is the same shape; a detail-only field (the contributing transaction list) is deliberately **not** embedded here (see `GET /budgets/:id` above), so there is no reason to fork the type.

```ts
interface BudgetDto {
  id: string;
  category: {
    id: string;
    name: string;
    type: 'EXPENSE'; // always EXPENSE for a Budget's category
  };
  amount: number;        // Budget Limit, Decimal(15,2) serialized as a number
  spent: number;         // Budget Spent for the current period
  remaining: number;     // amount - spent; may be negative, never clamped
  percentUsed: number | null; // exact (unrounded) percentage; null only in the defensive amount<=0 branch, unreachable via the API since amount > 0 is enforced at write time
  status: 'HEALTHY' | 'APPROACHING' | 'REACHED' | 'EXCEEDED' | 'ARCHIVED';
  isArchived: boolean;
  periodStart: string;   // ISO 8601, half-open period start (inclusive)
  periodEnd: string;     // ISO 8601, half-open period end (exclusive)
  createdAt: string;     // ISO 8601
  updatedAt: string;     // ISO 8601
}
```

### Decimal serialization

`amount`, `spent`, `remaining` are `Prisma.Decimal` at the domain/service layer and are serialized to **JavaScript `number`** at the HTTP boundary — matching the existing convention (`savingGoal.controller.ts`'s `serialize()`: `parseFloat(decimal.toString())`). This is a deliberate consistency choice, not an oversight:

- **Precision risk, called out explicitly:** `Decimal(15,2)` values up to roughly 9 × 10¹² (IDR) round-trip exactly through a JS `number` (`Number.MAX_SAFE_INTEGER` ≈ 9 × 10¹⁵), so realistic Budget amounts are safe. If Pocket Mint ever needs amounts near that ceiling, every existing money-serializing endpoint (`savingGoal`, `transaction`, `wallet`) has the same exposure — fixing it for Budgeting alone would create the exact per-feature drift the app's calculation-spec documents warn against. Not fixed here; flagged as a pre-existing, shared, low-probability risk rather than a Budgeting-specific one.
- `percentUsed` is serialized as a `number` at full precision (e.g. `74.9999`), **not pre-rounded** — the calculation spec's rule that "the stored/compared value... uses the unrounded ratio" extends to what the API returns, so a Frontend that needs the exact figure (e.g. for an accessible `aria-valuenow`) never has to re-derive it from `spent`/`amount`. Rounding to a whole-number display string (`"75%"`) is a presentation-only concern owned entirely by the Frontend (see the API contract's sibling doc, [User Journeys](./budgeting-user-journeys.md)) and must never feed back into any client-side status decision — `status` is always the field of record.

### `status` is always backend-computed

No Frontend code computes `HEALTHY`/`APPROACHING`/`REACHED`/`EXCEEDED`/`ARCHIVED` from `percentUsed`. This mirrors the calculation spec's explicit rule ("never computed independently by the frontend") and is the reason `status` is a first-class `BudgetDto` field rather than something derived client-side from the boundary table.

---

## Error Code Summary

| Code | Status | Meaning |
|---|---|---|
| `BAD_REQUEST` | 400 | Malformed request body (missing/wrong-typed field). |
| `UNAUTHORIZED` | 401 | No authenticated user. |
| `NOT_FOUND` | 404 | Budget does not exist or is not owned by the caller. |
| `CATEGORY_NOT_FOUND` | 404 | `categoryId` does not exist or is not owned by the caller. |
| `INVALID_AMOUNT` | 422 | `amount` is not a positive number with ≤2 decimal places. |
| `CATEGORY_NOT_EXPENSE` | 422 | `categoryId` resolves to an `INCOME` category. |
| `CATEGORY_NOT_EDITABLE` | 422 | Update request included `categoryId`. |
| `BUDGET_ALREADY_EXISTS` | 409 | A Budget (active or archived) already exists for this category. |
| `ALREADY_ARCHIVED` | 409 | Archive requested on an already-archived Budget. |
| `ALREADY_ACTIVE` | 409 | Restore requested on an already-active Budget. |
| `TOO_MANY_REQUESTS` | 429 | `mutationLimiter` rate limit hit. |

All codes follow the existing `sendError(res, message, statusCode, code)` shape (`src/utils/response.ts`) — no new envelope field. `BudgetError` (`src/services/budget.errors.ts`, already implemented) carries `message`, `statusCode`, `code` and is recognized automatically by `forwardError`'s structural `isOperational` check — no registration needed for these new codes.

No error ever discloses whether a `categoryId` or Budget `id` belongs to another user versus not existing at all.
