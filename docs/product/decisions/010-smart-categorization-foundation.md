# PD-010 — Smart Categorization Foundation

**Status:** Approved
**Date:** 2026-07-22
**Author:** Agent (Phase 18 implementation)

## Context

Pocket Mint users manually assign categories to every INCOME and EXPENSE transaction. The existing category picker is a simple dropdown that requires scrolling through a flat list. As users accumulate transaction history, the manual selection becomes repetitive — the same merchant descriptions map to the same categories every time.

Two later roadmap phases are planned:

- **Phase 19 — Merchant Mapping:** per-user merchant-name → category aliases
- **Phase 20 — Rule Engine:** user-defined categorization rules

This phase (Phase 18) builds the foundation that Merchant Mapping and Rule Engine extend.

## Decision

### Deterministic matching precedes Merchant Mapping

Phase 18 implements **deterministic keyword matching** using a static keyword → category mapping. This provides immediate value (suggestions for common Indonesian merchants) with zero user configuration.

When Phase 19 (Merchant Mapping) lands, the suggestion engine is extended as follows:

1. Check user-defined merchant aliases first (highest priority)
2. Fall back to the static keyword mapping (current implementation)
3. When Phase 20 (Rule Engine) lands, user rules override both

This ordering ensures each phase adds capability without redesigning the engine.

### Automatic categorization is intentionally deferred

Suggestions are **advisory only**. The user always selects a category explicitly. The system never:

- Auto-assigns a category without user confirmation
- Changes a transaction's category after creation
- Learns from user corrections
- Adjusts confidence based on correction history

This preserves the principle that financial data must be intentionally entered. Automation belongs to a later phase with explicit user opt-in.

### How Rule Engine will extend this architecture

The current architecture is designed for extension:

```
User input (description)
  → [Rule Engine] (Phase 20, future — highest priority)
  → MerchantNormalizer (Phase 18)
  → [Merchant Alias Lookup] (Phase 19, future)
  → KeywordMatcher (Phase 18)
  → ConfidenceCalculator (Phase 18)
  → Ranked suggestions
```

> **Correction (2026-07-22, [PD-012](./012-rule-engine.md)):** this diagram originally placed `[Rule Engine]` between `KeywordMatcher` and `ConfidenceCalculator` — a copy-paste error that contradicted this document's own prose below ("user rules override both") and PD-011's. Rule Engine runs **first**, as shown above. See PD-012 for the corrected pipeline and rationale.

Each stage is a pure function with no side effects. Adding a stage means inserting a call in the pipeline — no existing code is rewritten.

### Confidence model

Confidence is **deterministic and explainable**, never arbitrary:

| Confidence | Condition | Example |
|---|---|---|
| HIGH | Exact match: normalized description equals a keyword | `"indomaret"` → Belanja |
| MEDIUM | Contains match: description contains a keyword substring | `"alfamart serpong"` → Belanja (contains `"alfamart"`) |
| LOW | Token match: an individual word matches a keyword | `"makan bakso enak"` → Makanan (token `"makan"`) |

No match → no suggestion. The system never fabricates confidence values.

### Merchant normalization

Normalization is deterministic: lowercase, whitespace collapse, payment prefix stripping, trailing ID removal, separator normalization. No fuzzy matching, no Levenshtein, no embeddings.

### API design

The suggestion endpoint reuses the existing `/api/v1/categories` route prefix:

```
GET /api/v1/categories/suggestions?description=INDOMARET+%23123&type=EXPENSE
```

This is consistent with the existing convention of resource-scoped sub-routes. It reuses existing authentication (JWT Bearer token, `requireUser` middleware).

### Keyword mapping

The initial keyword set covers common Indonesian merchants and payment descriptions across all default categories. Keywords are mapped to category **names**, not IDs — the service resolves names to the user's actual category IDs at query time, so the mapping works with any category set.

## Consequences

- Users receive category suggestions immediately after typing 3+ characters in the description field
- Suggestions are deterministic — same input always produces same output
- Zero user configuration required
- Merchant Mapping (Phase 19) adds per-user aliases as a higher-priority lookup before keyword matching
- Rule Engine (Phase 20) adds user-defined rules as the highest-priority stage
- Automatic categorization remains a separate, later decision requiring its own PD
