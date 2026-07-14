# Pocket Mint Implementation Roadmap

---

## Purpose

This roadmap translates Pocket Mint's approved product and architecture documentation into an implementation dependency graph across `pocket-mint-docs`, `pocket-mint-be`, and `pocket-mint-fe`.

It defines what must become stable before dependent work begins, why that order protects financial correctness and privacy, which repository owns each responsibility, and where unresolved Product Decisions stop implementation. It does not authorize behavior governed by a Draft Product Decision. It is not a feature list, sprint plan, delivery estimate, or assignment plan.

Implementation follows the documentation authority order. Within an approved scope, documentation is reconciled first, backend contracts and Canonical Calculations are implemented second, and frontend presentation is implemented last.

---

## Current Documentation Status

### Approved Product Decisions

The individual Product Decision files are authoritative for status.

| Decision | Status | Implementation consequence |
|---|---|---|
| [PD-001 — Canonical Net Worth Definition](../product/decisions/001-net-worth.md) | Approved | Net Worth, Total Assets, Total Outstanding Debt, Negative Net Worth, and same-Reporting-Cutoff rules may be implemented. |
| [PD-002 — Owner-First Access Policy](../product/decisions/002-owner-auth.md) | Approved | First Owner with Controlled Access, explicit public exceptions, User isolation, and installation-level Owner authority may be implemented. |

### Draft Product Decisions

Draft decisions may guide architecture exploration but do not authorize their recommended product behavior.

| Decision | Status | Blocks |
|---|---|---|
| [PD-003 — Installment Lifecycle Policy](../product/decisions/003-installment-lifecycle.md) | Draft | Automatic term progression, Reporting Timezone behavior, catch-up, settlement, payoff, cancellation, and Installment Payment state. |
| [PD-004 — Installment Fee Treatment](../product/decisions/004-installment-fees.md) | Draft | Administrative Fee types, fee basis, persistence, and fee inclusion in Total Obligation. |
| [PD-005 — Debt Utilization Policy](../product/decisions/005-debt-policy.md) | Draft | Hard Credit Limit enforcement, Debt Ratio thresholds, over-limit handling, and PayLater risk semantics. |
| [PD-006 — Financial Reporting Surface](../product/decisions/006-reporting.md) | Draft | Dedicated Reports scope, Dashboard boundary, historical presentation, and report navigation. |
| [PD-007 — Canonical Transfer Representation](../product/decisions/007-transfer.md) | Draft | Retirement of the separate Transfer model and adoption of a Transaction with a destination wallet as the sole representation. |
| [PD-008 — Canonical Design Language](../product/decisions/008-design-language.md) | Draft | Canonical colors, typography, financial-number styling, radii, and token normalization. |

### Completed Documentation

- The [Vision](../product/vision.md) defines product purpose, values, boundaries, non-goals, and authority order.
- The [Canonical Glossary](../product/glossary.md) defines approved terminology and marks Draft-dependent terms as Provisional.
- The [System Architecture](../architecture/system-architecture.md) defines repository and layer ownership, data flow, trust boundaries, financial invariants, and future integration boundaries.
- The [Screen Specification](../product/screen-spec.md) defines current product surfaces, required states, data dependencies, navigation, and Product Decision gates.
- Individual Product Decision documents exist for PD-001 through PD-008, with PD-001 and PD-002 approved.
- The [Implementation Reconciliation Report](./reconciliation-report.md) records implementation evidence and known gaps across all three repositories.
- [AI Project Context](../ai/project-context.md), [AI Coding Rules](../ai/coding-rules.md), and the [Product Decision Approval Workflow](../ai/product-decision-approval-workflow.md) define authority-preserving AI and documentation workflows.

### Outstanding Documentation

- PD-003 through PD-008 require explicit approval and authority-order synchronization before their governed behavior is implemented.
- The Product RFC remains `Draft — Requires Implementation Reconciliation` and contains stale related-document status and roadmap language.
- The Design System remains Draft, contains internal token and prose conflicts, and is governed in part by PD-008.
- `product-decisions.md` is a stale aggregate that labels decisions Pending; it must not override the individual decision files.
- `component-spec.md` is empty, so reusable component contracts, variants, financial-number presentation, and interaction states are not implementation-ready.
- `user-flows.md` is empty, so cross-screen journeys and state transitions are not yet specified there.
- Screen sections governed by PD-003 through PD-008 are explicitly Provisional and cannot be treated as approved requirements.
- Backend API, persistence, migration, error, and idempotency contracts must be documented in the appropriate repository before dependent frontend or automation work.

---

## Implementation Philosophy

### Documentation First

Product ambiguity is resolved through a Product Decision. Once approved, the Product RFC, Glossary, Architecture, Design System, Screen Specification, Component Specification, and repository guidance are synchronized in authority order before implementation begins.

### Backend Before Frontend

The Backend owns authentication enforcement, authorization, Business Services, Canonical Calculations, persistence, financial mutations, and deterministic error contracts. The Frontend consumes those contracts and never becomes an alternate source of financial truth.

### Financial Correctness Before UI

Exact money handling, User ownership, atomicity, idempotency, Ledger integrity, Reporting Cutoffs, and explainable calculations are prerequisites for presenting financial results. A polished screen cannot compensate for an ambiguous or incorrect financial contract.

### Stable Contracts Before Features

Schema meaning, service behavior, command and query contracts, provenance, time boundaries, and error states must be stable before consumers are built. Frontend, Automation, and AI must not infer missing fields, states, or rules.

### One Canonical Path

Interactive, automated, imported, and future client requests pass through the same authenticated Backend API and Business Services. No caller may bypass ownership, validation, transactions, idempotency, or required User control.

### Recorded Facts Before Derived Views

The User-scoped Ledger and supporting records are established before Financial Summaries, Reports, Trends, or AI explanations. Derived Metrics remain reproducible from their inputs and never replace Historical Records.

### Approved Scope Before Optimization

Repository cleanup, visual normalization, Automation, and AI follow the stable financial and access foundations they consume. Draft policy remains replaceable and is not embedded into clients or persistence prematurely.

---

## Engineering Phases

The phases are ordered by dependency. A later phase may begin only when its required predecessors and Product Decision gates are complete. Work within a phase still follows `pocket-mint-docs` → `pocket-mint-be` → `pocket-mint-fe` where those repositories are affected.

### Phase 0 — Documentation and Repository Preparation

**Purpose**

Establish an implementation-ready authority baseline, record existing repository state, and remove documentation ambiguity before product behavior is changed.

**Repositories**

- `pocket-mint-docs` owns Product Decision approval and synchronization, canonical terminology, architecture, screen and component contracts, and this roadmap.
- `pocket-mint-be` owns repository-local backend guidance and the documentation of API, persistence, migration, error, and idempotency contracts.
- `pocket-mint-fe` owns repository-local frontend guidance aligned with higher-authority documentation.

**Dependencies**

- Vision and documentation authority order.
- Individual Product Decision status.
- Reconciliation findings as implementation evidence, not product authority.

**Blocked By**

- Any conflict between higher-authority documents.
- Any phase-specific behavior whose governing Product Decision remains Draft.
- Any edit that would overwrite unrelated working-tree changes.

**Enables**

- A stable scope for every later phase.
- Reviewable backend contracts and frontend consumption work.
- Clear separation between approved work and deferred policy.

**Risk**

- **High:** treating aggregate status, reconciliation findings, or existing implementation as approval would propagate incorrect policy through every repository.

### Phase 1 — Financial Core

**Purpose**

Create the backend-owned financial foundation: exact Decimal semantics, User-scoped Ledger facts, atomic financial mutations, deterministic Canonical Calculations, provenance, Reporting Cutoff consistency, and safe correction behavior within approved policy.

**Repositories**

- `pocket-mint-docs` defines approved financial terms and calculations, especially PD-001.
- `pocket-mint-be` owns money parsing and storage, Ledger integrity, Business Services, transactions, ownership validation, Canonical Calculations, provenance, and tests.
- `pocket-mint-fe` consumes canonical results only; phase work is limited to removing or isolating competing client calculations where required by an approved contract.

**Dependencies**

- Phase 0.
- PD-001.
- Architecture rules for exact money, Backend ownership of financial logic, atomicity, determinism, and explainability.

**Blocked By**

- Any calculation or Historical Record behavior not defined by approved documentation.
- Any attempt to use binary floating point as authoritative money input.

**Enables**

- Wallet, Transaction, Installment, Reporting, Automation, and AI phases to share one financial truth.
- Canonical Financial Summary contracts for Net Worth, Total Assets, and Total Outstanding Debt.

**Risk**

- **High:** precision loss, partial writes, duplicated effects, mismatched Reporting Cutoffs, or client-side calculations can corrupt every downstream financial view.

### Phase 2 — Authentication and Controlled Access

**Purpose**

Implement PD-002 as the trusted access boundary before financial capabilities are exposed: atomic first-Owner initialization, closed public registration after initialization, Owner-approved admission, authenticated User identity, resource isolation, recovery boundaries, and explicit public exceptions.

**Repositories**

- `pocket-mint-docs` owns the approved access policy and required journeys.
- `pocket-mint-be` owns authentication verification, atomic Owner creation, admission enforcement, authorization, User scoping, safe disclosure, recovery authority, and persistence or migration required by PD-002.
- `pocket-mint-fe` owns initialization, authentication, invitation or allowlist completion, recovery, protected navigation, Profile, and Settings presentation after backend contracts are stable.

**Dependencies**

- Phase 0.
- PD-002.
- Phase 1 ownership and transactional foundations for protected financial resources.

**Blocked By**

- An unspecified backend admission representation or migration path for existing installations.
- A recovery or Owner-replacement implementation that would transfer or expose another User's financial resources.

**Enables**

- Safe exposure of Wallet, Transaction, Installment, Reporting, Automation, and AI capabilities.
- Explicitly authorized, narrowly User-scoped service access in later phases.

**Risk**

- **High:** a bootstrap race, public signup path, authorization gap, identity disclosure, or Owner-as-financial-superuser behavior violates the privacy-first boundary.

### Phase 3 — Wallet Engine

**Purpose**

Provide canonical Asset Wallet and Debt Wallet state as the base aggregation boundary for later financial operations.

**Repositories**

- `pocket-mint-docs` owns wallet terminology, approved PD-001 aggregation rules, and any future PD-005 synchronization.
- `pocket-mint-be` owns wallet persistence, User ownership, type classification, Wallet Balance, Outstanding Balance, Total Assets, Total Outstanding Debt, Available Credit, Debt Ratio calculations, and authoritative command/query contracts.
- `pocket-mint-fe` owns Wallet List, Wallet Detail, Create Wallet, formatting, and interaction states using backend-returned values.

**Dependencies**

- Phases 0 through 2.
- PD-001 for Total Assets, Total Outstanding Debt, and Net Worth contributions.

**Blocked By**

- PD-005 for hard Credit Limit enforcement, utilization thresholds, over-limit migration behavior, and PayLater risk presentation.
- Unapproved deletion or Historical Record semantics for wallet removal.

**Enables**

- Transaction source and destination ownership checks.
- Installment attachment to Debt Wallets.
- Financial Summary and future Report aggregation.

**Risk**

- **High:** incorrect wallet classification, ownership, sign handling, or debt aggregation causes incorrect Net Worth and cross-User exposure.

### Phase 4 — Transaction and Transfer Engine

**Purpose**

Establish Income, Expense, Category, and Transfer behavior as atomic, User-scoped Financial Events whose wallet effects are reversible and explainable.

**Repositories**

- `pocket-mint-docs` owns Financial Event terminology and the product meaning of Income, Expense, Category, and Transfer.
- `pocket-mint-be` owns transaction validation, Category ownership, wallet effects, rollback, correction, idempotency where required, and the canonical Transfer contract after PD-007 approval.
- `pocket-mint-fe` owns Transaction List, Transaction Detail, Create Transaction, Transfer intent collection, consequence review, and confirmed-result presentation.

**Dependencies**

- Phases 1 through 3.
- Stable Wallet commands and queries.
- Authentication and User ownership enforcement.

**Blocked By**

- PD-007 for making Transaction-with-destination the sole Transfer representation and retiring the unused Transfer model.
- Unapproved general deletion and Historical Record policy where a requested action depends on it.

**Enables**

- A trustworthy Ledger for Financial Summaries and Reports.
- Installment creation and Installment Payment records.
- Automation and AI proposals to use the same financial command path.

**Risk**

- **High:** partial Transfer effects, cross-User Category or wallet references, incorrect reversal, or duplicate submission can break Ledger and Wallet Balance integrity.

### Phase 5 — Installment Engine

**Purpose**

Implement the approved Installment contract from creation through Total Obligation, schedule, monthly reporting, progression, settlement, payoff, and cancellation without applying the same debt effect twice.

**Repositories**

- `pocket-mint-docs` owns the approved lifecycle, fee, debt-limit, screen, user-flow, and component contracts.
- `pocket-mint-be` owns forward and reverse Canonical Calculations, Total Obligation, atomic creation, one-time Debt Wallet effect, durable schedule identity, lifecycle state, idempotency, catch-up, and migration.
- `pocket-mint-fe` owns Installments, Installment Detail, Create Installment, review and confirmation, schedule presentation, and approved manual actions.

**Dependencies**

- Phases 1 through 4.
- Stable Debt Wallet and Financial Event contracts.
- Approved definitions for lifecycle state, fees, limits, Reporting Timezone, rounding, and impossible reverse-calculation inputs.

**Blocked By**

- PD-003.
- PD-004.
- PD-005 where Credit Limit behavior affects creation.

**Enables**

- Installment-aware Reporting.
- Scheduled lifecycle Automation.
- Explainable installment status in Dashboard and future AI explanations.

**Risk**

- **High:** duplicate balance deductions, duplicate term processing, incorrect catch-up, ambiguous payment state, fee drift, or unsafe migration can misstate debt and history.

### Phase 6 — Reporting

**Purpose**

Produce Financial Summaries, period aggregates, and Trends exclusively from Ledger facts and Historical Records with explicit Reporting Cutoffs, Reporting Periods, Reporting Timezone, provenance, and missing-history gaps.

**Repositories**

- `pocket-mint-docs` owns approved Report scope, Dashboard boundary, terminology, screens, and components.
- `pocket-mint-be` owns canonical report queries, period boundaries, historical aggregation, provenance, partial-failure contracts, and consistent cutoffs.
- `pocket-mint-fe` owns Dashboard summary presentation and the approved Reports surface without recalculating canonical values.

**Dependencies**

- Phases 1 through 5 for complete financial facts.
- PD-001 for Financial Summary semantics.
- Stable Historical Record and provenance behavior.

**Blocked By**

- PD-006 for the dedicated Reports surface and final Dashboard boundary.
- PD-003 for Reporting Timezone and Installment Payment reporting.
- Missing or incomplete Ledger history; it must be shown as missing rather than synthesized.

**Enables**

- Explainable historical views.
- Backend-derived summaries for Automation and AI.
- Removal of fabricated growth, Trend, and principal/interest presentation.

**Risk**

- **High:** synthetic history, mixed cutoffs, incomplete totals presented as complete, or frontend recomputation directly undermines user trust.

### Phase 7 — Automation

**Purpose**

Add approved scheduled and externally triggered work through the same authenticated Backend and Business Services used by interactive clients.

**Repositories**

- `pocket-mint-docs` owns approved automation behavior, User-control boundaries, origin terminology, and any required Product Decisions.
- `pocket-mint-be` owns authorized automation-facing commands and queries, idempotency, origin and audit evidence, and committed outcomes.
- `pocket-mint-fe` owns User-visible authorization, confirmation, status, retry, and correction experiences where approved.

**Dependencies**

- Phases 1 through 6.
- Explicit service authorization and narrowly defined User scope from Phase 2.
- Stable commands, queries, error contracts, and provenance.

**Blocked By**

- PD-003 for scheduled Installment progression and catch-up.
- Any unapproved confirmation, correction, webhook, import, or notification policy.
- Missing idempotency identity or inability to distinguish Scheduled, Projected, Imported, Confirmed, and Paid states.

**Enables**

- The documented future Secure Webhooks, n8n Integration, WhatsApp Parsing, and other Automation adapters after their product contracts are approved.
- A controlled boundary for AI-assisted interpretation.

**Risk**

- **High:** retries, delayed jobs, duplicate delivery, excessive authority, or direct persistence can create duplicate or unauthorized Financial Events.

### Phase 8 — AI

**Purpose**

Introduce bounded probabilistic interpretation only after deterministic financial and automation paths are stable. AI may extract, classify, propose, or explain; it may not authenticate, authorize, calculate canonical values, confirm, persist, or invent history.

**Repositories**

- `pocket-mint-docs` owns approved AI scope, disclosure, confirmation, provenance, and privacy rules.
- `pocket-mint-be` owns minimized authorized read models, structured proposal validation, canonical explanations, and the mutation boundary.
- `pocket-mint-fe` owns proposal review, uncertainty and origin presentation, explicit User control, and safe error states.

**Dependencies**

- Phases 1 through 7.
- Stable authenticated Backend contracts and Canonical Calculations.
- Approved Automation origin, confirmation, idempotency, and audit behavior.

**Blocked By**

- Any request for AI to decide financial truth, infer missing history, bypass validation, or act with reusable User authority.
- Any undefined AI Transaction Extraction or AI Assistant product contract.

**Enables**

- The documented future AI Transaction Extraction and AI Assistant directions within the established trust boundary.

**Risk**

- **High:** private-data disclosure, hallucinated financial facts, opaque mutation, or probabilistic calculations would violate core product values.

### Phase 9 — UX Polish and Design-System Convergence

**Purpose**

Normalize the completed product around one approved visual and component contract without changing financial meaning or introducing unsupported analytics.

**Repositories**

- `pocket-mint-docs` owns the approved Design System, Screen Specification, Component Specification, User Flows, and accessibility expectations.
- `pocket-mint-be` has no design ownership; it supports only stable data, state, and error contracts needed by the experience.
- `pocket-mint-fe` owns tokens, typography, financial-number styling, components, responsive layout, accessibility, loading, empty, partial, success, and error states.

**Dependencies**

- Stable behavior and contracts from all applicable earlier phases.
- Complete Component Specification and User Flows.

**Blocked By**

- PD-008 for canonical visual tokens and typography.
- PD-005 for debt-risk presentation.
- PD-006 for final Dashboard and Reports composition.
- Any unresolved component behavior that would require inventing product policy.

**Enables**

- Consistent and accessible presentation across every approved surface.
- Safe removal of raw values, misleading variants, inert controls, and fabricated visuals.

**Risk**

- **Medium:** visual changes can obscure state meaning or create apparent inconsistencies if applied before product and data contracts stabilize.

---

## Cross Repository Order

```text
pocket-mint-docs
        ↓
   pocket-mint-be
        ↓
   pocket-mint-fe
```

For each approved behavior:

1. `pocket-mint-docs` resolves product policy, terminology, architecture, screens, components, user flows, dependencies, and acceptance boundaries.
2. `pocket-mint-be` implements the trusted contract, Business Services, persistence, migrations, Canonical Calculations, authorization, transactions, provenance, and deterministic errors.
3. `pocket-mint-fe` consumes the stable backend contract, collects explicit User intent, and presents confirmed results and states.

The order is a dependency rule, not a requirement to change every repository. A documentation-only decision stops after documentation. A backend-only contract does not create unnecessary frontend work. Frontend exploration may occur against replaceable mocks, but no Draft policy or invented contract becomes production behavior.

---

## Product Decision Dependencies

```text
PD-001 ──→ Phase 1 Financial Core ──→ Phase 3 Wallet Engine ──→ Phase 6 Reporting

PD-002 ──→ Phase 2 Authentication ──→ Phases 3–8 protected capabilities

PD-003 ──→ Phase 5 Installment Engine ──→ Phase 6 Reporting ──→ Phase 7 Automation

PD-004 ──→ Phase 5 Installment Engine ──→ Phase 6 Reporting

PD-005 ──→ Phase 3 Wallet Engine ──→ Phase 4 Transaction Engine
                                  └──→ Phase 5 Installment Engine

PD-006 ──→ Phase 6 Reporting ──→ Phase 9 UX Polish

PD-007 ──→ Phase 4 Transaction and Transfer Engine ──→ Phase 7 Automation

PD-008 ──→ Phase 9 UX Polish and Design-System Convergence
```

| Product Decision | Affected phases | Dependency meaning |
|---|---|---|
| PD-001 — Canonical Net Worth Definition | 1, 3, 6, 8 | Establishes canonical summary calculations and terminology consumed by wallets, reports, and explanations. |
| PD-002 — Owner-First Access Policy | 0, 2, 3–8 | Establishes who may access every financial capability and how Automation or AI receives authority. |
| PD-003 — Installment Lifecycle Policy | 0, 5, 6, 7, 9 | Must be approved before lifecycle, Reporting Timezone, schedule states, catch-up, or related UI becomes authoritative. |
| PD-004 — Installment Fee Treatment | 0, 5, 6, 9 | Must be approved before fee-inclusive Total Obligation, persistence, reports, or fee presentation is implemented. |
| PD-005 — Debt Utilization Policy | 0, 3, 4, 5, 6, 9 | Must be approved before hard limits, thresholds, over-limit behavior, or debt-risk presentation is standardized. |
| PD-006 — Financial Reporting Surface | 0, 6, 7, 9 | Must be approved before final Reports scope, navigation, Dashboard boundary, or scheduled report consumption is implemented. |
| PD-007 — Canonical Transfer Representation | 0, 4, 7 | Must be approved before the storage contract is declared sole, the unused model is retired, or integrations depend on it. |
| PD-008 — Canonical Design Language | 0, 9 | Must be approved before token, typography, radius, and semantic-color convergence is treated as canonical. |

---

## Risk Matrix

| Level | Risk areas | Required control |
|---|---|---|
| **High** | Money precision; Net Worth and debt calculation; User isolation; Owner initialization; financial mutation atomicity; Transfer symmetry; Installment double-counting; lifecycle idempotency; Historical Record integrity; Reporting Cutoff consistency; Automation authority; AI privacy and hallucination | Backend-owned Canonical Calculations and authorization, exact Decimal semantics, transactional writes, migrations, idempotency evidence, provenance, regression tests, and approved Product Decisions before dependent implementation. |
| **Medium** | API and schema compatibility; existing-installation migration; report query completeness; partial failures; recovery journeys; design semantics; accessibility; loading and error-state consistency | Stable documented contracts, compatibility review, explicit migration and rollback strategy, state-specific tests, and frontend consumption only after backend stabilization. |
| **Low** | Documentation navigation, relative links, non-behavioral copy alignment, and presentation cleanup that does not alter financial or access meaning | Documentation validation, terminology review, focused diffs, and repository-local formatting or link checks. |

Risk level does not authorize scope. A Low-risk task governed by a Draft Product Decision remains blocked.

---

## Suggested Repository Order

1. pocket-mint-docs

2. pocket-mint-be

3. pocket-mint-fe

---

## Exit Criteria

### Documentation Is Implementation-Ready When

- Every Product Decision required by the target phase is individually Approved.
- Each approved decision is synchronized through all affected documents in authority order.
- The Product RFC no longer contains stale or contradictory policy for the target phase.
- Canonical Glossary terms are approved, consistent, and no required implementation term remains Provisional.
- System Architecture names the owning layer and preserves the Backend, Automation, AI, and Database boundaries.
- Screen Specification, Component Specification, and User Flows define all affected interactions, permissions, data dependencies, loading, empty, partial, success, error, and consequence-confirmation states.
- Backend contracts, migrations, error semantics, provenance, Reporting Cutoffs or Periods, and idempotency requirements are documented where applicable.
- Draft-governed behavior is clearly deferred rather than silently assumed.
- Relative links, headings, Markdown structure, status references, and cross-document terminology are valid.
- Historical reconciliation findings are preserved and any superseding evidence is appended transparently.

### Backend Is Complete for a Phase When

- Every implemented behavior is authorized by approved documentation.
- Authentication, User ownership, authorization, validation, and disclosure rules are enforced at the Backend boundary.
- Canonical Calculations and financial state transitions reside in Business Services and use exact Decimal semantics.
- Multi-record financial mutations are atomic and consequential retries are idempotent.
- Database changes use deliberate migrations with compatibility and data-impact review.
- Commands return canonical outcomes and queries return the provenance and time boundaries required to interpret them.
- Deterministic errors distinguish rejection, unavailability, rollback, and unknown outcome without exposing another User's information.
- Required tests cover affected calculations, ownership, authorization, rollback, idempotency, migrations, and contracts, including the Product RFC minimum areas.
- No alternate Frontend, Automation, integration, or AI path bypasses Business Services.
- The relevant backend checks pass and remaining risks or unavailable checks are recorded accurately.

### Frontend Is Complete for a Phase When

- It consumes stable backend contracts and does not recreate Canonical Calculations, balances, thresholds, lifecycle rules, or financial truth.
- It presents canonical terminology and preserves Reporting Cutoff, Reporting Period, Reporting Timezone, provenance, origin, uncertainty, and state distinctions supplied by the Backend.
- It covers approved navigation and interaction flows, including loading, empty, partial, success, error, permission, recovery, and destructive or consequential confirmation states.
- It does not present synthetic Financial Events, fabricated Trends, placeholder growth, or unsupported principal and Interest splits as facts.
- It does not mark a mutation successful until the Backend confirms the outcome.
- It protects private navigation for usability while relying on Backend authorization as the security boundary.
- It meets the approved Design System, Component Specification, responsive behavior, financial-number formatting, and accessibility requirements.
- Affected interaction, rendering, contract, and accessibility checks pass and remaining risks or unavailable checks are recorded accurately.

The product is not implementation-complete merely because all screens exist. Completion requires the dependency graph to be satisfied from approved documentation through backend financial truth to frontend presentation.
