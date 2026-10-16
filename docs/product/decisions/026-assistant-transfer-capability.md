# PD-026 — Assistant Transfer Capability (Phase 36B)

**Status:** Accepted
**Date:** 2026-10-16
**Author:** Agent (Phase 36B implementation)

## Context

PD-007 (ratified alongside this decision) settles the canonical transfer *data model*: a transfer is one `Transaction` row with `walletId` (source) and `toWalletId` (destination) — no separate transfer identity. That model, and the underlying `transactionService.createTransaction()`/`transactionBalance.ts` domain logic, already fully supported `TRANSFER` before this phase. The gap PD-007 did not cover — and this decision does — is entirely in the Assistant layer: the `transaction.create` tool contract, the `AssistantFinancialDraft` persistence shape, the Persistent Clarification Engine, and the Telegram channel renderer were all `INCOME`/`EXPENSE`-only. PD-025 (Phase 35) explicitly listed "no transfer support" as out of scope. This decision opens natural-language transfer capture on top of the PD-007 model, without touching that model itself.

Three things made an internal transfer a distinct case rather than a third copy of the existing INCOME/EXPENSE path:

1. **Two wallets, not one.** Every existing Assistant entity-resolution and clarification path (wallet, merchant, category) resolves exactly one ambiguous reference per turn. A transfer has two independent wallet references (source, destination) that can each be ambiguous, not-found, or identical to the other.
2. **No category, ever.** `transactionService.createTransaction()` already rejects a category on a TRANSFER row. The draft layer (`AssistantFinancialDraft.categoryId`) had no way to represent "no category" — the column was `NOT NULL`.
3. **Higher risk, no new mechanism needed.** `transaction.create` is already registered `riskLevel: HIGH`, `confirmationPolicy: EXPLICIT` (draft + explicit confirm) — this predates this phase and already matches the product's stated intent that an internal transfer is HIGH-risk. `STEP_UP` exists in the `ConfirmationPolicy` type but is set by zero tools in the registry and is explicitly documented (PD-016) as unimplemented/dead; building on it for transfers would mean implementing unfinished infrastructure for its first real caller, not reusing something proven.

## Decision

**Selected: extend the existing `transaction.create` tool, draft, and clarification machinery additively for `TRANSFER`, with independent source/destination wallet resolution and source-first sequential clarification — reusing `EXPLICIT`/`DRAFT_AND_CONFIRM` confirmation (never `STEP_UP`) and the existing fixed Telegram web-handoff for the confirmation step.**

### 1. Tool contract

`transaction.create`'s `type` enum gains `TRANSFER` alongside `INCOME`/`EXPENSE`. A new optional provider argument, `toWalletReference`, is the destination's **text** reference — resolved exactly like `walletReference`, through the same deterministic entity-resolution pipeline and the same `WALLET_TRANSACTION_CREATE_CONSTRAINTS`. The model is never offered a `toWalletId` key — identical to how it was never offered `walletId` — so a destination wallet identifier can never come from the model. `categoryReference`/`categoryId` are rejected outright when `type` is `TRANSFER`. The tool description's language-mapping instructions, which previously told the model that "transfer untuk membayar" (transfer-to-pay phrasing) means `EXPENSE`, are rewritten: that signal is removed, and `TRANSFER` is given its own explicit instruction and examples, so transfer-sounding language is never routed to `EXPENSE` or `INCOME`. The same fix is made in the deterministic provider-miss recovery fallback (`transaction-type-recovery.ts`), which only ever recovers `INCOME`/`EXPENSE` from an omitted `type` and must not treat transfer language as an expense signal either.

### 2. Draft persistence

`AssistantFinancialDraft` gains a nullable `toWalletId` and its `categoryId` becomes nullable (migration, additive-only — see Backend Implementation Summary). Existing INCOME/EXPENSE rows are unaffected: `categoryId` stays populated for them, exactly as before, and `toWalletId` stays `null`. A TRANSFER row always has `toWalletId` populated and `categoryId` always `null` — enforced at the service boundary, not left to convention.

### 3. Independent two-wallet resolution and sequential clarification

Source and destination are resolved independently, through the same wallet entity-resolution reused unchanged (same candidate pool, same constraints) — not a parallel resolution path. Both resolved and unambiguous → the draft is created directly. Either one not found (or owned by another user) → a terminal rejection message, no draft. Either one ambiguous → a clarification is created.

When **both** are ambiguous, they are **not** clarified in parallel and not merged into one clarification: the source is clarified first; once selected, the destination is re-resolved from its original text reference and, if still ambiguous, clarified as a **child** of the source clarification (`parentId` set, reusing the existing parent-chain mechanism). This sequencing is additive, not a new entity type: both clarifications use the existing `entityType: 'wallet'`, discriminated internally by an additive `pendingWalletRole: 'wallet' | 'toWallet'` field carried in the clarification's `trustedContext` — never by a new value in the entity-resolution `EntityType` union, which stays `wallet | merchant | category`. The selected candidate for a `'wallet'`-role clarification only ever fills the source slot; a `'toWallet'`-role clarification only ever fills the destination slot. The other, already-resolved slot is carried forward in the same `trustedContext` and is never re-derived or overwritten by the next selection. Cancelling or letting either clarification expire terminates the chain with no draft and no financial mutation — the existing clarification cancel/expiry mechanics (atomic claim, database-time expiry) are reused unchanged.

### 4. Draft preview, confirm, and overrides

The draft preview and the recovery-state projection expose `toWalletId`/a destination display label for a TRANSFER draft, and omit `categoryId`/`category` entirely — additive fields on the existing projections; the INCOME/EXPENSE shape is unchanged. `DraftConfirmOverride` gains an optional `toWalletId`, added to the controller's existing fail-closed allowlist. Inside the existing atomic confirm transaction (advisory lock, idempotency replay, expiry, ownership — all unchanged), an override is re-validated exactly as `walletId`/`categoryId` already are: destination ownership, source wallet type (`CASH`/`BANK`/`E_WALLET` — the existing transfer-source allowlist, reused, not duplicated), and same-wallet rejection, all re-checked whenever either side is overridden. A `categoryId` override against a TRANSFER draft, or a `toWalletId` override against a non-TRANSFER draft, is rejected. Confirm calls the same `transactionService.createTransaction()` with `type: 'TRANSFER'`, source `walletId`, and destination `toWalletId` — no new balance logic, no new endpoint, no second confirmation path.

### 5. Telegram exclusion

A TRANSFER draft's confirmation step always renders the existing fixed "continue in the Pocket Mint web app" message — the same fallback Telegram already uses for a capability it cannot safely render inline (free-form clarification, or a future `STEP_UP` tool). It never renders the inline Confirm/Cancel keyboard that an INCOME/EXPENSE draft gets. Wallet clarification *within* a transfer flow is unaffected by this exclusion — it is still `entityType: 'wallet'` and still eligible for Telegram's existing bounded-option keyboard, the same as any other wallet clarification.

## Rejected alternatives

- **A separate `transaction.transfer` tool.** Rejected: would duplicate the entire tool-registration, policy, and audit-redaction machinery for what is the same canonical operation (`transaction.create`) with a different `type` — exactly the model PD-007 already settled for the data layer. One tool, one contract, an additive `type` value.
- **A distinct clarification entity type (e.g. `transfer_source_wallet` / `transfer_destination_wallet`) instead of a role discriminator.** Rejected in favor of the additive-discriminator form this decision explicitly allows: a new `EntityType` value would propagate into the entity-resolution registry, the resolver dispatch, the Telegram clarification-kind logging allowlist, and every switch keyed on `EntityType` — all for a distinction that only matters to the Assistant's own continuation logic, not to entity resolution itself (both roles resolve against the identical wallet candidate pool and constraints). The role lives in `trustedContext`, where it is only ever read by the transfer continuation path.
- **`STEP_UP` confirmation for transfers.** Rejected: `STEP_UP` is declared in the type system but implemented by zero tools, and the executor's own handling of a non-`EXECUTE_IMMEDIATELY`/`DRAFT_AND_CONFIRM` policy action is explicitly marked "not yet supported." Building a transfer's safety on it would mean finishing unfinished infrastructure as a side effect of this phase, not reusing a proven mechanism. `EXPLICIT`/`DRAFT_AND_CONFIRM` — already the policy `transaction.create` resolves to — combined with a dedicated, explicit two-wallet preview before confirm, is the existing, working, higher-risk-tier path.
- **Letting the destination be a trusted `toWalletId` from the model, mirroring neither income/expense's `walletId` fallback nor requiring a reference.** Rejected: the model is already never trusted with `walletId` for the source; extending that same boundary to the destination is the only decision consistent with "never accept trusted wallet IDs from the model."
- **Resolving both wallets in parallel and failing/merging if both are ambiguous.** Rejected: a single combined clarification would need to represent two independent choices in one answer, which the existing one-answer-per-clarification (`select` by a single token) contract cannot express without a new, more complex selection shape. Sequential resolution reuses the existing parent-chain mechanism (already proven for wallet → merchant → category) with no new selection contract.

## Consequences

**Positive:**

- Users can capture an internal transfer by natural language ("transfer 500rb dari BCA ke GoPay") with the same draft-review-then-confirm safety as every other Assistant-created transaction, instead of being forced to EXPENSE/INCOME or out to the manual form.
- Transfer language is never silently misclassified as income or expense at the tool boundary — removing a real, previously-existing mapping (`transfer untuk membayar` → `EXPENSE`) that this phase found and corrected.
- No new financial mutation surface: the same `transactionService.createTransaction()`, the same balance-effect domain, the same advisory-locked, idempotent, ownership-checked confirm transaction. A transfer draft is exactly as inert as an income/expense draft until explicit confirmation.
- The sequential-clarification design generalizes the existing single-entity clarification chain to two wallets without adding a second underlying mechanism — no new entity-resolution dispatch, no new Telegram rendering branch beyond reusing the existing "can't render this inline" fallback.

**Costs:**

- `AssistantFinancialDraft.categoryId` is now nullable at the schema level for every draft, not just transfer drafts — a one-time, backward-compatible loosening (existing INCOME/EXPENSE rows keep their non-null `categoryId`; application code still requires it for those types).
- The confirm-time override validation branches on `isTransfer` in a few more places (source type, destination ownership, same-wallet, category prohibition) — a small, bounded increase in that function's branching, not a new confirmation path.
- `pendingWalletRole` is an Assistant-internal field with no equivalent in the generic single-entity clarification case; a future reader of `CanonicalContext` needs to know it only applies when `type === 'TRANSFER'`.

## Related documents

- [PD-007 — Canonical Transfer Representation](007-transfer.md) — the data-model decision this phase builds the Assistant capability on top of, ratified alongside this decision
- [Assistant Core Architecture §15.14](../../architecture/assistant-core-architecture.md#1514-assistant-transfer-capability-phase-36b) — the draft contract, two-wallet clarification, and Telegram exclusion, documented in full
- [Implementation Roadmap — Phase 36](../../development/implementation-roadmap.md) — files touched per repository
- [PD-016 — Telegram Interactive Workflows](016-telegram-interactive-workflows.md) — the existing fixed web-handoff fallback this phase reuses for a TRANSFER draft's confirmation step, and the `STEP_UP`-is-unimplemented finding this phase relies on
- [PD-025 — Capture Grounding & First-Pass Reliability](025-capture-grounding-first-pass-reliability.md) — the immediately prior Assistant-capture phase, whose non-goals explicitly deferred transfer support to a later decision

## Non-goals / explicitly out of scope

- No change to the canonical transfer data model — that is PD-007's scope, not this one.
- No external bank transfer, bank credentials, payment rail, or transfer between different users.
- No currency conversion, no recurring/scheduled/split transfer.
- No new transaction API endpoint, no new balance algorithm, no second confirmation path.
- No `PATCH`/`PUT` draft endpoint; no editing of an already-confirmed transfer through the Assistant.
- No `STEP_UP` implementation — explicitly rejected above, not deferred.
- No Telegram-native transfer confirm/cancel keyboard, and no Telegram transfer edit UI.
- No merchant or category concept introduced for transfers — a transfer never has either.
- No weakening of confirmation, ownership, idempotency, expiry, or concurrency guarantees anywhere in the existing draft/confirm machinery.
- No financial advice, recommendation, or proactive messaging — unchanged product boundary.

## Follow-up Tasks

- If a future phase wants to restrict TRANSFER destination wallet type (today only the source is restricted to `CASH`/`BANK`/`E_WALLET` — see `financial-logic.skill.md`'s "TRANSFER destination hardening note"), that is a transaction-service-level hardening decision, not an Assistant-layer one, and applies to every transfer creation path, not just the Assistant's.
- If product wants a distinct user-facing prompt wording for "wallet sumber transfer" vs. "wallet tujuan transfer" beyond the current role-aware label already added, that is a copy change, not an architecture change.
