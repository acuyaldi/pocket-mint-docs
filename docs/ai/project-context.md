# Pocket Mint AI Project Context

## Project Summary

Pocket Mint is a privacy-first, self-hostable personal finance system for understanding tracked assets, debt, Financial Events, Installments, Reports, and Net Worth. It records User-scoped financial facts and derives explainable summaries through deterministic Canonical Calculations.

Pocket Mint does not move money, provide banking services, prescribe financial decisions, or infer missing financial history. Its financial position covers only information recorded in Pocket Mint, and automation remains subordinate to User control.

---

## Repository Map

| Repository | Responsibility |
|---|---|
| `pocket-mint-docs` | Authoritative product, architecture, design, screen, component, decision, and reconciliation documentation. |
| `pocket-mint-fe` | Frontend presentation, navigation, accessibility, interaction state, input assistance, and collection of explicit User intent. It consumes canonical backend results. |
| `pocket-mint-be` | Backend API, Business Services, persistence, authentication and authorization enforcement, Canonical Calculations, transactional financial mutations, and automation-facing capabilities. |

Do not move responsibilities between repositories without an approved architectural or product change.

---

## Documentation Authority

Use this authority order when sources disagree:

1. [Vision](../product/vision.md)
2. [Product RFC](../product/product-rfc.md)
3. [Approved Product Decisions](../product/decisions/)
4. [Canonical Glossary](../product/glossary.md)
5. [System Architecture](../architecture/system-architecture.md)
6. [Design System](../product/design-system.md)
7. [Screen Specification](../product/screen-spec.md)
8. [Component Specification](../product/component-spec.md)
9. Repository-local implementation guidance
10. Implementation

- Approved documents override implementation assumptions.
- Approved Product Decisions override lower-level documents within their decided scope.
- Draft Product Decisions are provisional recommendations, not settled policy.
- Implementation does not silently override approved product policy.
- Report conflicts before making changes. Do not resolve them by choosing the most convenient source.

---

## Required Reading by Task Type

Read only the minimum relevant context, then follow links when the task exposes a dependency or conflict.

| Task type | Minimum documents to read first |
|---|---|
| Product behavior | [Vision](../product/vision.md), [Product RFC](../product/product-rfc.md), [Glossary](../product/glossary.md), relevant [Product Decisions](../product/decisions/) |
| Financial logic | [Product RFC](../product/product-rfc.md), [Glossary](../product/glossary.md), [PD-001](../product/decisions/001-net-worth.md), [System Architecture](../architecture/system-architecture.md), relevant Product Decisions |
| Frontend screen | [Screen Specification](../product/screen-spec.md), [Design System](../product/design-system.md), [Glossary](../product/glossary.md), relevant Product Decisions |
| UI component | [Component Specification](../product/component-spec.md), [Design System](../product/design-system.md), [Glossary](../product/glossary.md), relevant Product Decisions |
| Backend command or query | [Product RFC](../product/product-rfc.md), [Glossary](../product/glossary.md), [System Architecture](../architecture/system-architecture.md), relevant Product Decisions |
| Authentication | [Product RFC](../product/product-rfc.md), [System Architecture](../architecture/system-architecture.md), [PD-002](../product/decisions/002-owner-auth.md), relevant repository guidance |
| Installment | [Product RFC](../product/product-rfc.md), [Glossary](../product/glossary.md), [PD-001](../product/decisions/001-net-worth.md), [PD-003](../product/decisions/003-installment-lifecycle.md), [PD-004](../product/decisions/004-installment-fees.md) |
| Transfer | [Product RFC](../product/product-rfc.md), [Glossary](../product/glossary.md), [System Architecture](../architecture/system-architecture.md), [PD-007](../product/decisions/007-transfer.md) |
| Reporting | [Product RFC](../product/product-rfc.md), [Glossary](../product/glossary.md), [PD-001](../product/decisions/001-net-worth.md), [PD-006](../product/decisions/006-reporting.md), [Screen Specification](../product/screen-spec.md) |
| Budgeting | [Product RFC](../product/product-rfc.md#budgeting), [Glossary](../product/glossary.md), [PD-009](../product/decisions/009-budgeting-scope.md), [Budgeting Readiness Audit](../development/budgeting-readiness-audit.md), [Budgeting Calculation Specification](../development/budgeting-calculation-spec.md), [Screen Specification](../product/screen-spec.md#budgets) |
| Automation | [Product RFC](../product/product-rfc.md), [System Architecture](../architecture/system-architecture.md), relevant Product Decisions and repository guidance |
| AI integration | [Vision](../product/vision.md), [System Architecture](../architecture/system-architecture.md), [Glossary](../product/glossary.md), relevant Product Decisions |
| Documentation change | [Vision](../product/vision.md), the documents above the target in the authority order, relevant Product Decisions, [Reconciliation Report](../development/reconciliation-report.md) when implementation evidence matters |

---

## Canonical Product Rules

These rules are approved or stable. Follow the linked sources for complete definitions.

- Pocket Mint is privacy-first and self-hostable. Minimize disclosure and keep Users in control. ([Vision](../product/vision.md))
- Every resource and operation is User-scoped. Ownership and authorization are enforced by the Backend. ([Product RFC](../product/product-rfc.md), [System Architecture](../architecture/system-architecture.md))
- Pocket Mint uses First Owner with Controlled Access: the first initialized account becomes Owner, public registration closes after initialization, and additional Users require explicit Owner approval. ([PD-002](../product/decisions/002-owner-auth.md))
- Owner authority is installation-level and never bypasses per-User financial resource isolation. Every user-financial capability requires authentication; only the public exceptions approved by PD-002 may remain unauthenticated. ([PD-002](../product/decisions/002-owner-auth.md))
- Business rules and Canonical Calculations belong to backend Business Services, not clients, automation, prompts, or formatters. ([System Architecture](../architecture/system-architecture.md))
- The same facts, policy version, Reporting Cutoff or Reporting Period, Reporting Timezone, and money precision must produce the same result. ([System Architecture](../architecture/system-architecture.md))
- Money uses exact decimal representation. Binary floating point is not authoritative for financial decisions. ([Product RFC](../product/product-rfc.md))
- Multi-record financial mutations are atomic: every effect commits or none does. ([System Architecture](../architecture/system-architecture.md))
- Financial facts and Derived Metrics must remain explainable through their inputs, calculation, provenance, and time boundary. ([Vision](../product/vision.md), [Glossary](../product/glossary.md))
- Missing financial facts and Historical Records remain missing. Never fabricate history or present projections as facts. ([Vision](../product/vision.md))
- **Net Worth = Total Assets − Total Outstanding Debt**, evaluated at the same Reporting Cutoff. It covers only assets and debts tracked by Pocket Mint and may be negative. ([PD-001](../product/decisions/001-net-worth.md))
- **Total Assets** is the combined balance of tracked Asset Wallets at the Reporting Cutoff. ([PD-001](../product/decisions/001-net-worth.md))
- **Total Outstanding Debt** is the combined Outstanding Balance of tracked Debt Wallets at the Reporting Cutoff. ([PD-001](../product/decisions/001-net-worth.md))
- Values from different Reporting Cutoffs must not be combined in one Financial Summary. ([PD-001](../product/decisions/001-net-worth.md))
- An Installment obligation already represented in a Debt Wallet must not be counted or applied again through later Installment Payments. ([PD-001](../product/decisions/001-net-worth.md))
- A Transfer is neither Income nor Expense and must not create or remove value from the tracked financial position. Its storage representation remains provisional under PD-007. ([System Architecture](../architecture/system-architecture.md))

---

## Provisional Areas

The following Product Decisions are **Draft**. Their recommendations may guide exploration, but must not be implemented or described as settled policy without explicit approval.

| Decision | Provisional area |
|---|---|
| [PD-003 — Installment Lifecycle Policy](../product/decisions/003-installment-lifecycle.md) | Automatic progression, catch-up, settlement, payoff, cancellation, due states, and Reporting Timezone behavior |
| [PD-004 — Installment Fee Treatment](../product/decisions/004-installment-fees.md) | Administrative Fee types, basis, persistence, and inclusion in Total Obligation |
| [PD-005 — Debt Utilization Policy](../product/decisions/005-debt-policy.md) | Hard credit limits, utilization thresholds, risk states, and over-limit handling |
| [PD-006 — Financial Reporting Surface](../product/decisions/006-reporting.md) | Dedicated Reports surface, report scope, navigation, and historical presentation |
| [PD-007 — Canonical Transfer Representation](../product/decisions/007-transfer.md) | Whether a Transfer is represented by a Financial Event with a destination wallet or a separate entity |
| [PD-008 — Canonical Design Language](../product/decisions/008-design-language.md) | Light-only design, color semantics, typography, financial-number styling, tokens, and radii |
| [PD-009 — Budgeting Scope and MVP Definition](../product/decisions/009-budgeting-scope.md) | Whether Budgeting exists at all, its category-only scope, calendar-month period, spend basis, status thresholds, notification policy, rollover exclusion, and the reworded Non-Goals text it depends on |

PD-001 and PD-002 are Approved. PD-003 through PD-009 are Draft as of this document's creation. Always check current decision front matter before relying on status.

---

## Architectural Boundaries

| Layer | Owns | Does not own |
|---|---|---|
| Frontend | Presentation, interaction state, accessibility, intent collection, non-authoritative input assistance, canonical result formatting | Financial truth, Canonical Calculations, authoritative balances, ownership decisions, persistence |
| Backend API | Trusted boundary, authentication context, request scoping, boundary validation, command and query contracts, idempotency context, safe responses | Alternate financial rules outside Business Services |
| Business Services | Domain behavior, Canonical Calculations, ownership checks, invariants, lifecycle rules, transactions, provenance, final acceptance or rejection | Presentation or delivery-channel-specific behavior |
| Database | Durable User-scoped facts, Ledger, relationships, exact values, transactions, structural constraints, idempotency evidence | Independent product meaning or client contracts |
| Automation | Approved scheduling and triggers, retry, catch-up, deduplication, origin tracking, operational state | Financial authority, direct persistence, independent calculations, bypassing validation or confirmation |
| AI Services | Bounded probabilistic extraction, classification, proposal, or explanation | Authentication, authorization, calculations, validation, persistence, confirmation, or historical truth |

- Frontend does not own financial truth.
- Automation cannot bypass the Backend.
- AI output is untrusted input.
- Frontend, Automation, and AI never access the Database directly.
- Canonical Calculations belong to Business Services.

---

## AI Task Workflow

0. **Classify the task** as Product, Documentation, Frontend, Backend, Architecture, or Automation work.
1. **Read the minimum required documents** for that task type and resolve the exact scope.
2. **Inspect existing implementation** before proposing or duplicating anything.
3. **Detect conflicts or provisional Product Decisions** and stop when clarification is required.
4. **Produce a short implementation plan** naming the repository, files, checks, and policy dependencies.
5. **Modify only the required repository** and files; backend changes precede dependent frontend changes.
6. **Verify changes** with relevant tests, checks, document validation, and diff inspection.
7. **Report evidence**: inspected files, modified files, checks and results, assumptions, conflicts, and provisional decisions involved.

---

## When to Stop

Stop and request product clarification when:

- multiple Product Decisions conflict;
- a Draft Product Decision affects the requested work;
- the requested implementation would change approved product behavior;
- documentation and implementation disagree without an approved resolution; or
- the requested change would introduce a new business concept.

Do not resolve these conditions through implementation assumptions, copied behavior, or new policy hidden in code.

---

## Change Rules

- Work on one task at a time.
- Do not refactor unrelated code.
- Do not duplicate an existing function, route, type, component, calculation, or rule.
- Backend changes precede dependent frontend changes.
- Database changes require a deliberate, reviewable migration.
- Product policy changes require a Product Decision.
- Do not update approved documentation merely to match buggy implementation.
- Do not present provisional policy as approved.
- Do not invent missing financial data or Historical Records.
- Do not hardcode values that belong to canonical data, configuration, or design tokens.
- Preserve existing unrelated work in a dirty worktree.

---

## Financial Safety Rules

- Never use binary floating point as authoritative money input.
- Never calculate canonical financial metrics independently in UI components.
- Never partially apply a multi-record financial operation.
- Never combine values from different Reporting Cutoffs.
- Never count one Installment obligation twice.
- Never treat Scheduled, Projected, Imported, Confirmed, and Paid as synonyms.
- Never hide, clamp, or relabel Negative Net Worth as zero or Total Assets.
- Never expose another User's data or resource existence.

---

## Documentation Change Workflow

```text
Product ambiguity
        ↓
Product Decision
        ↓
Approval
        ↓
Product RFC or Glossary update
        ↓
Architecture, Design System, or Screen Specification update
        ↓
Implementation
```

- Resolve product ambiguity at the Product Decision layer before dependent documentation or implementation.
- Update lower-level documents only after the governing decision is approved.
- Reconciliation reports are historical audit records. Append or supersede findings transparently; never rewrite them to hide resolved findings.

---

## Verification Expectations

Every completion report must include:

- [ ] Files inspected
- [ ] Files modified
- [ ] Tests or checks run
- [ ] Verification result, including failures or skipped checks
- [ ] Assumptions made
- [ ] Unresolved conflicts
- [ ] Provisional Product Decisions involved

Never claim completion without fresh verification evidence.

---

## Quick Reference

- [Vision](../product/vision.md)
- [Product RFC](../product/product-rfc.md)
- [Canonical Glossary](../product/glossary.md)
- [Product Decisions](../product/decisions/)
- [Design System](../product/design-system.md)
- [Screen Specification](../product/screen-spec.md)
- [System Architecture](../architecture/system-architecture.md)
- [Implementation Reconciliation Report](../development/reconciliation-report.md)
