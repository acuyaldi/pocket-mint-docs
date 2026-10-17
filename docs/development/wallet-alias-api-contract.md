# Pocket Mint Wallet Alias API Contract

> Status: Implemented (Phase 37). Governed by [PD-027](../product/decisions/027-wallet-alias-management.md) (Accepted). Implemented at `pocket-mint-be/src/controllers/walletAlias.controller.ts`, `src/routes/walletRoutes.ts`, `src/services/walletAlias.service.ts`.
>
> Conventions: same as the Merchant Mapping API (`docs/development/merchant-mapping-api-contract.md`) — static controller class, `sendSuccess`/`sendError` envelope, `forwardError` + `WalletAliasError`, `requireUser` on every route, `mutationLimiter` on every mutating route, `getAuthenticatedUserId(req)` — never a client-supplied `userId`.
>
> See [Assistant Core Architecture §15.15](../architecture/assistant-core-architecture.md#1515-explicit-wallet-aliases-phase-37) for how a created alias affects Assistant transaction-capture resolution — this document covers only the CRUD HTTP contract.

---

## Response Envelope

Success: `{ "success": true, "data": T, "message": string }`
Error: `{ "success": false, "error": { "code": string, "statusCode": number, "message": string } }`

The existing app-wide envelope (`src/utils/response.ts`). No Wallet Alias-specific envelope.

## WalletAliasDto

```json
{
  "id": "clxalias001",
  "walletId": "clxwallet001",
  "aliasDisplay": "Rekening Utama",
  "createdAt": "2026-10-17T00:00:00.000Z"
}
```

Only the display text is returned — never `aliasNormalized` or `userId`, neither of which any client needs.

## Wallet read DTO addition

`GET /api/v1/wallets` additively returns each wallet's aliases on the existing wallet object:

```json
{
  "id": "clxwallet001",
  "name": "BCA Debit",
  "...": "... every existing wallet field, unchanged ...",
  "aliases": [
    { "id": "clxalias001", "aliasDisplay": "Rekening Utama" }
  ]
}
```

`aliases` is always present (an empty array for a wallet with none) on `GET /wallets`. It is **not** present on the `POST`/`PUT /wallets` create/rename response bodies — those mutations don't touch the alias table and don't re-fetch it, so they simply omit the key rather than claim a stale or empty list.

---

## Endpoints

### `POST /api/v1/wallets/:walletId/aliases`

Create an explicit alias for an owned wallet.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Body:** `{ aliasDisplay: string }`.
- **Behavior:** `aliasDisplay` is trimmed and normalized via the identical `normalizeEntityReference` the Assistant Wallet Resolver uses (see [§15.1](../architecture/assistant-core-architecture.md#151-deterministic-entity-resolution)) — no second normalizer. The create runs inside a user-scoped advisory-locked transaction so it cannot race a concurrent wallet create/rename targeting the same normalized name (see PD-027 §4).
- **Status codes:**
  - `201` on success.
  - `400 BAD_REQUEST` — `aliasDisplay` missing/empty, or exceeds the shared per-alias byte budget (128 UTF-8 bytes, the same ceiling the resolver enforces on every candidate alias).
  - `422 INVALID_ALIAS` — `aliasDisplay` normalizes to nothing (e.g. `"###"`).
  - `404 NOT_FOUND` — `walletId` does not exist or is not owned by the caller.
  - `409 ALIAS_LIMIT_REACHED` — the wallet already has the maximum aliases (10 — see PD-027 §4's "Costs").
  - `409 CONFLICT` — the normalized alias collides with another of the caller's aliases, any of the caller's canonical wallet names, or this wallet's own canonical name; also the fallback for a concurrent-create race the pre-check didn't catch (`P2002` translated to the same conflict).

### `DELETE /api/v1/wallets/:walletId/aliases/:aliasId`

Delete an owned alias.

- **Auth:** `requireUser`. **Rate-limited:** `mutationLimiter`.
- **Behavior:** Ownership is checked in one scoped lookup — `(id: aliasId, walletId, userId)` — so an alias belonging to a different wallet than the one named in the route, or to another User, is one indistinguishable `404`, never a separate "wrong wallet" signal.
- **Status codes:** `200` on success (`data: null`); `404 NOT_FOUND` per the lookup above.
- There is no `PATCH`/`PUT` alias endpoint. Renaming an alias is delete-then-recreate — an intentional simplification (PD-027), not an oversight.
- Deleting an alias never touches the wallet it belonged to, any transaction history, or any in-flight Assistant draft/clarification; it only removes that one exact candidate from future resolution.

---

## Resolution integration (read path, not exposed as a separate endpoint)

Once created, an alias is picked up by the Assistant Wallet Resolver on its very next candidate load — no cache to invalidate, no separate activation step. It applies identically to normal `INCOME`/`EXPENSE` wallet resolution and to both the source and destination sides of a `TRANSFER` (see [§15.14](../architecture/assistant-core-architecture.md#1514-assistant-transfer-capability-phase-36b)), and identically across the Web and Telegram channels, because all of them call the same resolver. See [§15.15](../architecture/assistant-core-architecture.md#1515-explicit-wallet-aliases-phase-37) for the full resolution-tier and ambiguity contract.
