# PD-012 — Rule Engine (Phase 20)

**Status:** Approved
**Date:** 2026-07-22
**Author:** Agent (Phase 20 implementation)

## Context

[PD-010](./010-smart-categorization-foundation.md) (Phase 18) and [PD-011](./011-merchant-mapping.md) (Phase 19) built a deterministic categorization pipeline and reserved a slot for Phase 20's Rule Engine. Both documents' **prose** is consistent — PD-010: *"When Phase 20 (Rule Engine) lands, user rules override both [merchant mapping and keyword matching]"*; PD-011: *"a matching user-defined rule will short-circuit ahead of a Merchant Mapping hit, the same way a Merchant Mapping hit today short-circuits ahead of keyword matching."* Both say rules run **first**.

However, both documents' **diagram** places `[Rule Engine]` between `KeywordMatcher` and `ConfidenceCalculator` — i.e. last, not first:

```
MerchantNormalizer → Merchant Alias Lookup (Ph.19) → KeywordMatcher (Ph.18) → [Rule Engine] (Ph.20) → ConfidenceCalculator
```

This is a copy-paste error that was never corrected across two documents. This ADR resolves the contradiction: **the prose was correct, the diagram was wrong.** Rules run first.

## Decision

### Rule Engine runs first and short-circuits everything else

The categorization pipeline's real order, as implemented in `categorization.service.ts::getSuggestions()`, is now:

```
User input (description, type)
  → Rule Engine (Phase 20 — this document, highest priority)
  → Merchant Normalizer + Merchant Mapping lookup (Phase 19)
  → KeywordMatcher (Phase 18)
  → ConfidenceCalculator (Phase 18)
  → Ranked suggestions
```

If any enabled rule matches, the service returns a single HIGH-confidence suggestion immediately — Merchant Mapping, keyword matching, and confidence scoring never run for that request. This mirrors the existing Merchant Mapping short-circuit exactly; no existing stage was rewritten, only a new check was inserted before all of them, consistent with PD-010's "each stage is a pure function with no side effects" principle.

**Why rules run first, not last:** a rule is the most explicit, most deliberate signal a user can give — more explicit than a remembered merchant alias, which is itself more explicit than a static keyword. Ordering by decreasing explicitness (rule → merchant mapping → keyword) is the only order where a user configuring a rule can trust it will actually apply, which is the entire point of building a rule engine. An engine whose rules could be silently overridden by an inferred keyword match would not be trustworthy enough to build a UI around.

### Why first-match wins, with no further evaluation

Rules are ordered per-user by an explicit `priority` (ascending — lower runs first). The first enabled rule that matches wins; no other rule is evaluated afterward, and no attempt is made to combine, rank, or blend multiple rule matches.

This was chosen over evaluating all rules and picking the "best" match because:

- Rules are boolean/deterministic (contains/equals/starts-with/ends-with on a specific field), not scored — there is no natural notion of "better" between two rule matches the way HIGH/MEDIUM/LOW confidence works for keyword matches.
- First-match-wins is the same mental model as `switch`/`if-else if` chains, which the priority-ordered UI (move up/down) directly exposes to the user: a rule's position in the list *is* its precedence, with no hidden tie-breaking logic to reason about.
- It keeps the rule model itself intentionally simple — no AND/OR groups, no nested conditions, no weights — per the roadmap's explicit exclusion of an expression language for this phase.

### Rules only ever suggest — manual override is always possible

A rule match returns exactly the same `CategorySuggestion` shape the rest of the pipeline already returns (`categoryId`, `categoryName`, `confidence: 'HIGH'`, `reason: 'Matched by rule: "<name>"'`, ...). It is surfaced through the existing `GET /api/v1/categories/suggestions` endpoint and the existing `CategorySuggestionList` UI component — there is no new "locked category" concept, no auto-fill that bypasses the transaction form, and no code path where a rule mutates a transaction directly. The user must still click the suggestion (or leave the category field for manual selection) to accept it, exactly as with merchant-mapping and keyword suggestions today. This preserves PD-010's "advisory-only, automatic categorization is intentionally deferred" principle without exception.

Rules are evaluated only during transaction create/edit — never against historical transactions, never via a background job. There is no bulk-recategorize action; adding or editing a rule never touches previously-saved transactions.

### Rules are strictly user-scoped

Every rule is owned by exactly one user (`userId` foreign key). There is no shared/global rule set and no cross-user fallback, matching the same ownership model as Merchant Mapping (PD-011), Budgets, and every other per-user domain in Pocket Mint.

### Future Automation integration point

This phase deliberately does not implement Automation, background jobs, or any execution without direct user interaction — those are out of scope per the roadmap and deferred to the future Automation phase. Because a rule match is exposed as a pure, side-effect-free suggestion (`RuleCandidate[] → RuleMatch | null`, `src/domain/rules/ruleMatcher.ts`), a future Automation phase can reuse the exact same `matchRules()` function against a background job or webhook without any change to the Rule Engine itself — Automation would only add a new *caller* of this pure function, not a new rule-matching implementation. This is the intended extension point.

## Rule model

```
Rule
  id          String        @id
  userId      String
  name        String
  enabled     Boolean       @default(true)
  priority    Int           // ascending — lower runs first; service-managed, not client-set directly
  matchType   RuleMatchType // DESCRIPTION | MERCHANT | TRANSACTION_TYPE
  operator    RuleOperator  // CONTAINS | EQUALS | STARTS_WITH | ENDS_WITH
  value       String
  categoryId  String
  createdAt   DateTime
  updatedAt   DateTime

  @@index([userId, enabled, priority])
```

- `matchType: MERCHANT` compares against the same `normalizeMerchant()` output Merchant Mapping uses, so a rule on "Steam" matches "STEAM ###" the same way a merchant mapping would.
- `matchType: TRANSACTION_TYPE` only ever means equality — the stored `operator` is present for schema uniformity but ignored for this `matchType`, both in the matcher (`src/domain/rules/ruleMatcher.ts`) and validated server-side (`rule.service.ts` rejects a `TRANSACTION_TYPE` rule whose `value` isn't `INCOME`/`EXPENSE`/`TRANSFER`).
- `priority` has no uniqueness constraint. New rules are appended after the user's current lowest-priority rule; reordering is an explicit, all-or-nothing operation (`PATCH /rules/reorder`) that must supply the user's complete rule-id set or is rejected — a partial or foreign list would silently corrupt ordering.

## API

`GET|POST|PATCH|DELETE /api/v1/rules`, `PATCH /api/v1/rules/reorder` — ownership-scoped CRUD + explicit reordering, reusing the existing JWT `requireUser` auth and `{ success, data, message }` / `{ success, error }` response envelope. Enable/disable is a `PATCH /api/v1/rules/:id` with `{ enabled }`, not a separate endpoint — mirrors how every other partial update in this codebase works.

## Consequences

- Users gain a deterministic, trustworthy override for categorization that nothing else in the pipeline can silently outrank.
- PD-010 and PD-011's stale diagrams are superseded by this document's corrected diagram; their prose was already accurate and required no change.
- Merchant Mapping and keyword matching are otherwise completely unaffected — both remain the fallback path when no rule matches, exactly as PD-011 already specified for the Merchant-Mapping-vs-keyword-matching relationship.
- A future Automation phase has a ready-made, pure-function extension point (`matchRules()`) rather than needing to reimplement condition matching.

## Explicitly out of scope (deferred to later phases or not planned)

- Automation, background jobs, execution without direct user interaction
- Regex, nested condition groups, AND/OR boolean expressions, an expression language
- AI/LLM/embeddings-based matching, fuzzy matching
- Shared/global rules, import/export, OCR, CSV import
- Bulk recategorization of historical transactions
