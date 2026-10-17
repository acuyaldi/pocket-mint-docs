# PD-027 — Wallet Alias Management & Resolution Reliability (Phase 37)

**Status:** Accepted
**Date:** 2026-10-17
**Author:** Agent (Phase 37 implementation)

## Context

The Assistant's deterministic Wallet Resolver (§15.1 of the Assistant Core Architecture) already derives exact-match aliases from a wallet's canonical name — "BCA Debit" yields the derived tokens `bca`/`debit`. That derivation is useful but fixed: a User cannot teach the resolver a nickname that isn't a literal substring-by-word of the wallet's own name — "rekening utama", "dompet jajan", or any other personal shorthand simply never resolves, and the only workaround is renaming the wallet itself, which is destructive to the canonical display name the rest of the product shows.

This phase closes that gap with an explicit, User-managed alias the resolver treats as a first-class exact candidate — without touching anything else about how resolution, confirmation, or financial execution already work.

## Decision

**Selected: add a `WalletAlias` model as a per-user, exclusive nickname namespace, fed into the existing production Wallet Resolver as additional exact candidates alongside the existing name-derived tokens — no new resolver, no new entity type, no fuzzy matching.**

### 1. Namespace: per-user, not per-wallet, not global

An alias is unique within `(userId, aliasNormalized)` — never globally unique, and never shared or visible across Users. The same alias text is free to use again by a different User, or (once deleted) by the same User on a different wallet. This mirrors the existing `MerchantMapping` namespace shape (`(userId, normalizedMerchant)`) rather than inventing a new convention.

### 2. Collision policy: exact, symmetric, no silent resolution

Within one User's namespace, an alias must not collide — after the identical NFKC/lowercase/punctuation normalization the resolver itself uses — with: another of that User's explicit aliases, any of that User's canonical wallet names, or the canonical name of the wallet the alias is being attached to. Symmetrically, creating or renaming a wallet is rejected if the candidate canonical name collides with that User's existing alias. Every collision is a typed conflict response (409, the repository's existing validation/conflict shape) — never a silent overwrite, rename, or wallet selection on either side.

A pre-existing duplicate canonical wallet name (two wallets already named identically, from before this phase) is left exactly as it was: still ambiguous at resolution time, as it always has been. No retroactive cleanup or backfill migration was performed — that is a separate, riskier decision this phase does not make.

### 3. Resolution tier: exact alias, same tier as a derived token

A stored alias is merged into the resolver's existing per-candidate alias list, explicit aliases ordered first so the existing 16-alias-per-candidate cap never silently drops a User's intentional alias ahead of a derived token. It is matched at the identical `alias_exact` tier (score 950) a derived token already occupies in the existing scoring table — canonical exact (1000) remains strongest, and no new score tier or fuzzy/substring/edit-distance matching was introduced. Ambiguity (two candidates tying within the existing margin) still resolves to `ambiguous`, never a silent pick, whether the tie involves an alias, a derived token, or both.

### 4. Concurrency: a user-scoped advisory lock, not a new mechanism class

Because the namespace spans two tables (`Wallet.name`, `WalletAlias.aliasNormalized`) that no single database constraint can cover together, both the alias-create path and a name-changing wallet create/rename open a `pg_advisory_xact_lock` keyed on `userId` (distinctly salted from the existing per-draft lock used elsewhere in the Assistant layer) for exactly the check-then-write window. This reuses the same lock *mechanism* the codebase already relies on for a different purpose, rather than introducing a new concurrency-safety idiom. The `(userId, aliasNormalized)` unique constraint remains the final guarantee beneath the lock.

### 5. API and ownership: the repository's existing conventions, not new ones

`POST /wallets/:walletId/aliases` and `DELETE /wallets/:walletId/aliases/:aliasId` — no update endpoint; delete-and-recreate is the intended edit flow, matching how the product treats a handful of other small owned sub-resources. Ownership is re-verified per request from the authenticated caller only; a wallet or alias belonging to another User is indistinguishable from `404 NOT_FOUND`, the same convention already used throughout this backend.

## Rejected alternatives

- **Teach the Assistant to learn aliases automatically from conversation.** Rejected: this would make natural-language interpretation output influence a durable, User-visible naming decision without an explicit User action — a different and much larger trust boundary than the deterministic, User-initiated flow this phase builds. It also reintroduces exactly the "provider/LLM-sourced trusted data" risk the entity-resolution layer was built to keep out.
- **A new, distinct `alias_exact_explicit` score tier above derived-token `alias_exact` but below canonical exact.** Rejected: there is no product signal that a User's explicit alias should outrank a wallet's own name-derived token in a genuine tie — both are exact, deterministic, User-controlled-in-origin matches. Reusing the existing tier is simpler and avoids a score table that would need to justify a new rank nobody asked for.
- **A single globally-unique alias namespace (one alias string usable by only one User account, system-wide).** Rejected: wallet names and nicknames are personal ("rekening utama" is an extremely common phrase); a global namespace would make the feature fail unpredictably and would leak a (admittedly meaningless) cross-user signal — whether some *other* account already used a given phrase. Per-user namespacing avoids this entirely and matches every other owned-resource namespace in the product.
- **Store the alias only, and derive `aliasNormalized` at query time instead of persisting it.** Rejected: the uniqueness constraint and the collision checks both need to query by the normalized form; persisting it is what makes `(userId, aliasNormalized)` a real database-level guarantee instead of an application-level race condition.
- **An Assistant-owned alias table, scoped to Assistant use only.** Rejected outright by the originating product requirements: aliases belong to the wallet/account domain the rest of the product already owns, not to the Assistant layer that merely consumes them at resolution time.

## Consequences

**Positive:**

- A User can teach transaction capture a personal nickname for any wallet without renaming it, closing a real first-pass-reliability gap the fixed name-derived-token behavior could not.
- The namespace guarantee (no alias can shadow a wallet name, or vice versa, for that User) is enforced at write time, not left as a resolution-time ambiguity for the User to puzzle over later.
- Zero new resolution mechanism: the production Wallet Resolver, its normalization, its scoring tiers, its ambiguity policy, and its owner-scoped query discipline are all reused exactly as-is. Transfer source/destination resolution (§15.14) gained alias support for free, with no code change to the transfer path itself.
- No change to confirmation, idempotency, the clarification lifecycle, or any financial mutation path.

**Costs:**

- A wallet create/rename now opens a transaction (and a brief advisory lock) in the one case — a canonical name is actually being set or changed — that didn't need one before; every other wallet mutation (balance-adjacent fields, archive, color/icon) stays the original single write.
- The per-wallet alias cap (10) exists solely so a User's explicit aliases can never, combined with a wallet's own name-derived tokens, exceed the resolver's existing 16-alias-per-candidate ceiling — an implementation constant to track if that resolver-level ceiling is ever revisited.
- The frontend's wallet read DTO grows one additive field (`aliases`); every existing consumer of that DTO that doesn't care about aliases is unaffected, but it is one more field future wallet-DTO changes need to keep in mind.

## Related documents

- [Assistant Core Architecture §15.1](../../architecture/assistant-core-architecture.md#151-deterministic-entity-resolution) — the production Wallet Resolver, normalization policy, and scoring table this phase extends, updated in place
- [Assistant Core Architecture §15.15](../../architecture/assistant-core-architecture.md#1515-explicit-wallet-aliases-phase-37) — the full namespace, concurrency, and resolution-tier contract for this phase
- [Implementation Roadmap — Phase 37](../../development/implementation-roadmap.md) — files touched per repository
- [Wallet Alias API Contract](../../development/wallet-alias-api-contract.md) — the alias CRUD endpoint documentation
- [Glossary — Wallet Alias](../glossary.md#wallet-alias)

## Non-goals / explicitly out of scope

- No fuzzy, substring, edit-distance, or semantic wallet matching, for aliases or otherwise.
- No automatic alias creation from an Assistant message or any provider/LLM output.
- No cross-user alias discovery or sharing; no global alias namespace.
- No silent selection on ambiguity — unchanged from existing resolver behavior.
- No alias history or audit system.
- No dedicated alias-management route in the frontend; it lives inside the existing wallet-edit modal.
- No Assistant-owned alias model or table.
- No change to confirmation, idempotency, the clarification lifecycle, transfer balance logic, or transaction lifecycle semantics.
- No financial advice or recommendation behavior.
- No new channel, queue, or Telegram-specific alias logic — Telegram benefits automatically through the shared resolver, exactly like Web.
- No retroactive cleanup migration for pre-existing duplicate canonical wallet names.

## Follow-up Tasks

- If a future phase wants aliases discoverable/searchable from the wallets list itself (beyond the compact display this phase adds), that is a UI-scope decision, not a resolver or namespace change.
- If the per-candidate alias ceiling (16) or the per-wallet alias cap (10) ever needs raising for an unrelated reason, re-derive the relationship between the two documented in this decision rather than changing either in isolation.
