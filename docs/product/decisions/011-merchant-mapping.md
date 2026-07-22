# PD-011 — Merchant Mapping (Phase 19)

**Status:** Approved
**Date:** 2026-07-22
**Author:** Agent (Phase 19 implementation)

## Context

[PD-010](./010-smart-categorization-foundation.md) (Phase 18) shipped a deterministic keyword-matching suggestion engine and explicitly reserved a higher-priority slot for per-user merchant aliases:

```
User input (description)
  → MerchantNormalizer (Phase 18)
  → [Merchant Alias Lookup] (Phase 19, future)
  → KeywordMatcher (Phase 18)
  → [Rule Engine] (Phase 20, future — highest priority)
  → ConfidenceCalculator (Phase 18)
  → Ranked suggestions
```

Phase 19 implements that reserved stage: **Merchant Mapping**, a per-user table of `merchantName → categoryId` aliases.

## Decision

### Merchant Mapping precedes keyword matching

When a suggestion request arrives, the categorization service now:

1. Normalizes the description (existing `MerchantNormalizer`, unchanged).
2. Looks up an exact `(userId, normalizedMerchant)` match, scoped to the requested category `type` (INCOME/EXPENSE), in the user's Merchant Mapping table.
3. **If found:** returns that category immediately as a single HIGH-confidence suggestion and skips keyword matching entirely.
4. **If not found:** falls through to the unchanged Phase 18 keyword-matching pipeline.

This is a strict precedence order, not a merge or a weighted blend — a user's explicit mapping always wins over the static keyword list. This matches PD-010's stated rationale: each phase adds a stage without redesigning the engine, and existing keyword-matching code is untouched (`keywordMatcher.ts`, `confidenceCalculator.ts` have zero changes in this phase).

### Mappings are strictly user-scoped

Every mapping is owned by exactly one user (`userId` foreign key, `@@unique([userId, normalizedMerchant])`). There is:

- No global or shared merchant database.
- No cross-user fallback — a lookup for user A never considers user B's mappings, even for an identical merchant string. Two different users can map `"warung bu siti"` to two entirely different categories.
- No learning from other users' behavior.

This preserves the same per-user data ownership model as every other Pocket Mint domain (Wallets, Categories, Budgets) and avoids the product and privacy complexity of a shared merchant knowledge base — explicitly out of scope per the roadmap.

### Mappings require explicit user confirmation

Merchant Mapping is created only through:

1. The dedicated management screen (Akun → Merchant Mapping): explicit create/edit/delete.
2. An opt-in "Ingat merchant ini" ("Remember this merchant") checkbox on the transaction form, unchecked by default. A transaction is never mapped automatically, and checking the box only takes effect after the transaction itself is successfully saved — a mapping-save failure never blocks or rolls back the transaction.

No mapping is ever created as a side effect of a suggestion being accepted, a transaction being categorized, or any background process. This continues PD-010's "automatic categorization is intentionally deferred" principle: the system may suggest, but the user always decides what gets remembered.

### How Rule Engine (Phase 20) will extend this architecture

Phase 20 is unaffected by this change and will slot in above Merchant Mapping, per PD-010's original diagram:

```
User input (description)
  → MerchantNormalizer (Phase 18)
  → Merchant Alias Lookup (Phase 19 — this document)
  → KeywordMatcher (Phase 18)
  → [Rule Engine] (Phase 20, future — highest priority)
  → ConfidenceCalculator (Phase 18)
  → Ranked suggestions
```

When Phase 20 lands, a matching user-defined rule will short-circuit ahead of a Merchant Mapping hit, the same way a Merchant Mapping hit today short-circuits ahead of keyword matching. No existing stage is rewritten to add it — only a new check is inserted before the others, consistent with the "each stage is a pure function with no side effects" principle from PD-010.

## Data model

```
MerchantMapping
  id                  String   @id
  userId              String
  merchantName        String   // as entered by the user, display form
  normalizedMerchant  String   // MerchantNormalizer output, used for lookup
  categoryId          String
  createdAt           DateTime
  updatedAt           DateTime

  @@unique([userId, normalizedMerchant])
  @@index([userId])
```

`normalizedMerchant` is stored once at write time (not recomputed on every read) using the exact same `normalizeMerchant()` function the suggestion pipeline already uses, so a mapping created from `"INDOMARET #123"` and a later transaction described as `"indomaret #456"` resolve to the same normalized key.

## API

`GET|POST|PATCH|DELETE /api/v1/merchant-mappings` — ownership-scoped CRUD, reusing the existing JWT `requireUser` auth and `{ success, data, message }` / `{ success, error }` response envelope. See `docs/development/merchant-mapping-api-contract.md` for the full contract.

## Consequences

- Suggestion quality improves immediately for any merchant a user has explicitly mapped, with zero change to keyword-matching behavior for everything else.
- No automatic categorization is introduced — advisory-only remains true for both Phase 18 and Phase 19 sources.
- Rule Engine (Phase 20) has a well-defined, unchanged insertion point.
- Cross-user data isolation is preserved; no shared merchant knowledge exists anywhere in the system.

## Explicitly out of scope (deferred to later phases or not planned)

- Rule Engine (Phase 20)
- Automatic/AI-based categorization, embeddings, fuzzy matching
- Shared/global merchant database, learning from other users
- Import/export, background jobs, OCR, CSV import
- Learning from user corrections (still deferred per PD-010)
