# Pocket Mint Rule Engine API Contract

> Status: Implemented (Phase 20). Governed by [PD-012](../product/decisions/012-rule-engine.md) (Approved). Implemented at `pocket-mint-be/src/controllers/rule.controller.ts`, `src/routes/ruleRoutes.ts`, `src/services/rule.service.ts`.
>
> Conventions: same as the Merchant Mapping API (`docs/development/merchant-mapping-api-contract.md`) — static controller class, `sendSuccess`/`sendError` envelope, `forwardError` + `RuleError`, `requireUser` on every route, `mutationLimiter` on every mutating route, `getAuthenticatedUserId(req)` — never a client-supplied `userId`.

---

## Response Envelope

Success: `{ "success": true, "data": T, "message": string }`
Error: `{ "success": false, "error": { "code": string, "statusCode": number, "message": string } }`

The existing app-wide envelope (`src/utils/response.ts`). No Rule-specific envelope.

## RuleDto

```json
{
  "id": "clxrule001",
  "name": "Gopay → Transport",
  "enabled": true,
  "priority": 0,
  "matchType": "DESCRIPTION",
  "operator": "CONTAINS",
  "value": "GOPAY",
  "categoryId": "clxcat001",
  "createdAt": "2026-07-22T00:00:00.000Z",
  "updatedAt": "2026-07-22T00:00:00.000Z"
}
```

`matchType` is one of `DESCRIPTION | MERCHANT | TRANSACTION_TYPE`. `operator` is one of `CONTAINS | EQUALS | STARTS_WITH | ENDS_WITH` — present but ignored when `matchType` is `TRANSACTION_TYPE` (always equality; see PD-012).

---

## Endpoints

### `GET /api/v1/rules`

List the caller's rules, ordered by `priority` ascending (evaluation order).

- **Auth:** `requireUser`.
- **Response:** `data: RuleDto[]`. Empty list is `200` with `data: []`, never `404`.

### `POST /api/v1/rules`

Create a rule.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Body:** `{ name: string, matchType: RuleMatchType, operator: RuleOperator, value: string, categoryId: string, enabled?: boolean }`.
- **Behavior:** `priority` is never client-set — the new rule is appended after the caller's current lowest-priority rule (or `0` for the caller's first rule). `categoryId` must belong to the caller.
- **Status codes:**
  - `201` on success.
  - `400 BAD_REQUEST` — `name`/`value` missing or empty, or `matchType`/`operator` not a recognized value.
  - `422 INVALID_RULE_VALUE` — `matchType: TRANSACTION_TYPE` with a `value` that isn't `INCOME`, `EXPENSE`, or `TRANSFER`.
  - `404 CATEGORY_NOT_FOUND` — `categoryId` does not exist or is not owned by the caller.

### `PATCH /api/v1/rules/:id`

Update a rule's name, condition, target category, and/or enabled state. All fields are optional and independent. This is also how enable/disable is toggled (`{ enabled: false }`) — there is no separate enable/disable endpoint.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Body:** `{ name?: string, matchType?: RuleMatchType, operator?: RuleOperator, value?: string, categoryId?: string, enabled?: boolean }`.
- **Behavior:** changing `matchType` alone re-validates the rule's existing `value` against the new `matchType` (e.g. switching to `TRANSACTION_TYPE` without also sending a new `value` fails with `INVALID_RULE_VALUE` unless the existing value happens to already be `INCOME`/`EXPENSE`/`TRANSFER`).
- **Status codes:**
  - `200` on success.
  - `404 NOT_FOUND` — rule does not exist or is not owned by the caller (identical ownership-scoped lookup shape to Merchant Mapping's).
  - `422 INVALID_RULE_VALUE`, `404 CATEGORY_NOT_FOUND` — same as create.
- **Priority is never updated here** — see `PATCH /rules/reorder` below.

### `PATCH /api/v1/rules/reorder`

Rewrite the caller's rule priorities from an explicit new ordering.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Body:** `{ ruleIds: string[] }` — must be the caller's **complete** current set of rule ids, in the desired evaluation order.
- **Behavior:** rewrites `priority` to each id's index in the array. Not wrapped in a database transaction — see the `ponytail:` comment in `rule.service.ts` for the known limitation and upgrade path.
- **Status codes:**
  - `200` on success, `data: RuleDto[]` (the caller's rules in their new order).
  - `400 INVALID_PRIORITY_ORDER` — `ruleIds` contains a duplicate, is missing one of the caller's rules, or contains an id not owned by the caller. Rejected outright rather than partially applied.
- Registered before `PATCH /rules/:id` in the router so the literal path segment `reorder` is never captured as an `:id`.

### `DELETE /api/v1/rules/:id`

Delete a rule.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Status codes:** `200` on success (`data: null`); `404 NOT_FOUND` per the same ownership-scoped lookup.
- Deleting a rule never touches existing transactions — only future suggestions, which fall back to Merchant Mapping / keyword matching.

---

## Pipeline integration (read path, not exposed as a separate endpoint)

`GET /api/v1/categories/suggestions?description=&type=` ([PD-010](../product/decisions/010-smart-categorization-foundation.md)) now checks the caller's enabled rules first, in ascending `priority` order, before Merchant Mapping — see [PD-012](../product/decisions/012-rule-engine.md) for the full precedence rationale. No request/response shape change: a rule match returns the same `CategorySuggestion[]` shape as a merchant-mapping or keyword match, as a single HIGH-confidence entry with `reason: 'Matched by rule: "<name>"'`.
