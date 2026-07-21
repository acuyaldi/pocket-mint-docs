# Pocket Mint Merchant Mapping API Contract

> Status: Implemented (Phase 19). Governed by [PD-011](../product/decisions/011-merchant-mapping.md) (Approved). Implemented at `pocket-mint-be/src/controllers/merchantMapping.controller.ts`, `src/routes/merchantMappingRoutes.ts`, `src/services/merchantMapping.service.ts`.
>
> Conventions: same as the Budgeting API (`docs/development/budgeting-api-contract.md`) — static controller class, `sendSuccess`/`sendError` envelope, `forwardError` + `MerchantMappingError`, `requireUser` on every route, `mutationLimiter` on every mutating route, `getAuthenticatedUserId(req)` — never a client-supplied `userId`.

---

## Response Envelope

Success: `{ "success": true, "data": T, "message": string }`
Error: `{ "success": false, "error": { "code": string, "statusCode": number, "message": string } }`

The existing app-wide envelope (`src/utils/response.ts`). No Merchant Mapping-specific envelope.

## MerchantMappingDto

```json
{
  "id": "clxmapping001",
  "merchantName": "Warung Bu Siti",
  "normalizedMerchant": "warung bu siti",
  "categoryId": "clxcat001",
  "createdAt": "2026-07-22T00:00:00.000Z",
  "updatedAt": "2026-07-22T00:00:00.000Z"
}
```

---

## Endpoints

### `GET /api/v1/merchant-mappings`

List the caller's mappings, ordered by `merchantName` ascending.

- **Auth:** `requireUser`.
- **Query parameters:** `search` — optional, case-insensitive substring filter on `merchantName`.
- **Response:** `data: MerchantMappingDto[]`. Empty list is `200` with `data: []`, never `404`.

### `POST /api/v1/merchant-mappings`

Create a mapping.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Body:** `{ merchantName: string, categoryId: string }`.
- **Behavior:** `merchantName` is trimmed and normalized via the same `normalizeMerchant()` the suggestion pipeline uses. `categoryId` must belong to the caller.
- **Status codes:**
  - `201` on success.
  - `400 BAD_REQUEST` — `merchantName` missing/empty.
  - `422 INVALID_MERCHANT_NAME` — `merchantName` normalizes to nothing (e.g. `"###"`).
  - `404 CATEGORY_NOT_FOUND` — `categoryId` does not exist or is not owned by the caller.
  - `409 DUPLICATE_MERCHANT` — a mapping with the same normalized merchant already exists for this user (pre-check plus a `P2002` fallback for the concurrent-create race).

### `PATCH /api/v1/merchant-mappings/:id`

Update a mapping's merchant name and/or category. Both fields are optional and independent — either or both may be sent.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Body:** `{ merchantName?: string, categoryId?: string }`.
- **Status codes:**
  - `200` on success.
  - `404 NOT_FOUND` — mapping does not exist or is not owned by the caller (identical lookup shape to Budget's `findFirst({ where: { id, userId } })` — a cross-user id never distinguishes "not found" from "not yours").
  - `409 DUPLICATE_MERCHANT` — renaming into a normalized merchant already used by another of the caller's mappings.
  - `404 CATEGORY_NOT_FOUND` — new `categoryId` not owned by the caller.

### `DELETE /api/v1/merchant-mappings/:id`

Delete a mapping.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Status codes:** `200` on success (`data: null`); `404 NOT_FOUND` per the same ownership-scoped lookup.
- Deleting a mapping never touches existing transactions — only future suggestions for that merchant, which fall back to keyword matching (or nothing, if the merchant matches no keyword).

---

## Pipeline integration (read path, not exposed as a separate endpoint)

`GET /api/v1/categories/suggestions?description=&type=` ([PD-010](../product/decisions/010-smart-categorization-foundation.md)) now checks the caller's Merchant Mapping table first — see [PD-011](../product/decisions/011-merchant-mapping.md) for the full precedence rationale. No request/response shape change: a mapping hit returns the same `CategorySuggestion[]` shape as a keyword match, just as a single HIGH-confidence entry with `reason: 'Merchant mapping: "<merchantName>"'`.
