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

### Approved Product Decisions (continued)

| Decision | Status | Implementation consequence |
|---|---|---|
| [PD-009 — Budgeting Scope and MVP Definition](../product/decisions/009-budgeting-scope.md) | Approved | Category-level monthly spend-limit tracking, the five-state Budget Status model, one-persistent-Budget-per-category uniqueness, and the notification decision (not yet implemented) may be implemented. Phase A (domain foundation), Phase B1 (command service), and Phase B2 (HTTP API, controllers, routes, DTO mapping) are implemented; frontend, dashboard integration, and notification integration remain pending. |

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
- Any undefined AI Transaction Extraction product contract.

**Enables**

- The documented future AI Transaction Extraction direction within the established trust boundary.
- Assistant Core (Phase 21), whose architecture is now defined in [Assistant Core Architecture](../architecture/assistant-core-architecture.md); implementation has not started.

**Risk**

- **High:** private-data disclosure, hallucinated financial facts, opaque mutation, or probabilistic calculations would violate core product values.

### Phase 10 — Budgeting

**Purpose**

Introduce per-category monthly spending limits, calculated dynamically from posted `EXPENSE` transactions, so Users can see how much they may spend, have spent, and have remaining per Category.

**Status**

[PD-009](../product/decisions/009-budgeting-scope.md) is **Approved**. **Phase A (domain foundation) is implemented**: the `Budget` Prisma model and migration, and the backend calculation/query service (`src/services/budget-query.service.ts`, `src/domain/budget.ts`), following the five-state Budget Status model (Decision E) and one-persistent-Budget-per-category uniqueness (Decision L). **Phase A.5 (UX and API contract review) is complete**: the [Budgeting API Contract](./budgeting-api-contract.md), [Budgeting User Journeys](./budgeting-user-journeys.md), and [Phase A.5 Design Review](./budgeting-phase-a5-design-review.md) finalize the implementation-ready design for Phase B, and `screen-spec.md`'s Budgets/Budget Detail/Create Budget/Edit Budget sections were updated from Draft-era language to Approved. **Phase B1 (command service and validation) is implemented**: `src/services/budget.service.ts` (`createBudget`, `updateBudgetAmount`, `archiveBudget`, `restoreBudget`), reusing the existing `BudgetError` class with the API contract's exact codes (`BAD_REQUEST`, `INVALID_AMOUNT`, `CATEGORY_NOT_FOUND`, `CATEGORY_NOT_EXPENSE`, `BUDGET_ALREADY_EXISTS`, `NOT_FOUND`, `ALREADY_ARCHIVED`, `ALREADY_ACTIVE`), with unit tests in `test/budgetService.test.ts`. Category reassignment and delete remain unsupported (no `categoryId` accepted by the update command; no delete command exists).

**Phase B2 (HTTP API, controllers, routes, DTO mapping, and contract tests) is implemented**:
- Controllers: `src/controllers/budget.controller.ts` — `BudgetController` static class (list, getOne, create, update, archive, restore), composing the command service for mutations and the query service for current-month usage through a single canonical `toBudgetDto` mapper.
- Routes: `src/routes/budgetRoutes.ts` — mounted at `/api/v1/budgets` in `src/routes/index.ts`, with `requireUser` on all routes and `mutationLimiter` on mutating routes.
- Endpoints: `GET /api/v1/budgets` (active list, `?status=archived` filter), `GET /api/v1/budgets/:id`, `POST /api/v1/budgets`, `PATCH /api/v1/budgets/:id`, `POST /api/v1/budgets/:id/archive`, `POST /api/v1/budgets/:id/restore`.
- DTO: `BudgetDto` centralized in `toBudgetDto()` — Decimal → `number`, status backend-computed, period bounds exposed as ISO 8601, category (`id`/`name`/`type`) included via a single Prisma `include` per query.
- Request validation: `categoryId` rejected with `CATEGORY_NOT_EDITABLE` on PATCH; `status` query param validated to `"active"`|`"archived"`; body fields are allowlisted (no spread); business validation stays in `budget.service.ts`.
- Error mapping: `BudgetError` forwarded through `forwardError` (structural `isOperational` recognition); `NOT_FOUND` covers both missing and cross-user; `CATEGORY_NOT_FOUND` covers both missing and cross-user categories; raw Prisma P2002 never reaches the client.
- Tests: `test/budgetController.test.ts` (111 boundary tests covering auth, CRUD, DTO serialization, error forwarding, DTO contract); integration tests added to `test/prismaAdapter.integration.test.ts` (command-service P2002 translation, full acceptance path: create→list→spend→verify→update→archive→verify absent→restore→verify active).
- No schema changes, no new migrations, no new dependencies, no frontend changes.

**Phase B3 (frontend screens, forms, navigation, and API integration) is implemented** in `pocket-mint-fe`:
- Types/API/hooks: `src/types/budget.ts` (`BudgetDto` mirror, backend status union, `isArchived` kept authoritative), `src/features/budgets/hooks/useBudgets.ts` (`useBudgets`, `useBudget`, `useCreateBudget`, `useUpdateBudget`, `useArchiveBudget`, `useRestoreBudget`; pessimistic mutations; `invalidateBudgetDependents` invalidates `['budgets']` only — corrected in Phase B4, see below). `useTransactions.ts`'s `invalidateTransactionDependents` now also invalidates `['budgets']`.
- Routes: `app/(app)/anggaran/page.tsx` (Budgets — active/archived toggle on one screen) and `app/(app)/anggaran/[id]/page.tsx` (Budget Detail — figures plus the contributing-transaction list, derived client-side from the existing `useTransactions()` current-month hook filtered by `categoryId`, not a new endpoint). Create and Edit are modals (`CreateBudgetModal`, `EditBudgetModal`), matching the finalized Phase A.5 UX decision; Archive/Restore are confirmation modals (`ArchiveBudgetModal`, `RestoreBudgetModal`).
- Navigation: Budgets added as the 7th entry to both `app-sidebar.tsx` and `bottom-nav.tsx` (icon: `Gauge`), matching the design review's acknowledged existing 6-item deviation from the 5-label canon.
- Status/progress: centralized `BudgetStatusPill`/`BudgetProgressBar` — label and status always backend-computed (`status` field, never derived from `percentUsed`); progress bar visual width and `aria-valuenow` are capped at 100 (an ARIA range fix made in Phase B4 — `aria-valuenow` above `aria-valuemax` is invalid), while the true unrounded `percentUsed`, including values above 100, is exposed via `aria-valuetext` and the on-screen percentage text.
- Errors: `budgets.errors.*` localization keys keyed on the backend's stable `code` (`BAD_REQUEST`, `INVALID_AMOUNT`, `CATEGORY_NOT_FOUND`, `CATEGORY_NOT_EXPENSE`, `CATEGORY_NOT_EDITABLE`, `BUDGET_ALREADY_EXISTS`, `NOT_FOUND`, `ALREADY_ARCHIVED`, `ALREADY_ACTIVE`), never the English message string.
- Localization: `nav.budgets`, `budgets.*`, `budgetModals.*`, `budgetDetail.*` added to both `messages/en.json` and `messages/id.json`, key-parity verified by the existing `tests/i18n.test.ts`.
- Tests: `tests/budgets.test.ts` (source-contract assertions only — `readFileSync` + string-marker checks under a `node` vitest environment; this repo has no `@testing-library`/jsdom installed, so nothing here renders a component or simulates interaction) plus the full existing suite and `tsc --noEmit`/`next build` all green.
- Not implemented (deferred per task scope): dashboard Budget summary slot, notification/threshold UI, historical period browsing, delete, category reassignment, a new Playwright E2E spec (no Playwright test-runner config exists yet in this repo — only ad-hoc screenshot scripts — so no new E2E harness was introduced for this feature alone).

**Phase B4 (database integration verification, live UI QA, and release stabilization) is partially implemented** — see [Budgeting Readiness Audit](./budgeting-readiness-audit.md#phase-b4-verification-2026-07-21) for the full account. Summary:
- Backend CI (`pocket-mint-be/.github/workflows/ci.yml`) now runs a `postgres:18` service container, applies `prisma migrate deploy`, and runs the full `vitest run` (including the previously-never-executed Prisma/Budget integration suite) — this had never actually run against a real database in CI before B4, despite `docs/database-testing.md` describing it as already wired up.
- The same integration suite (32 tests) was run locally against a disposable `embedded-postgres` instance and passed, including duplicate-active/duplicate-archived Budget rejection, P2002 → `BUDGET_ALREADY_EXISTS` translation, ownership isolation, and cascade delete. The GitHub Actions run itself remains pending until this branch/PR actually executes in CI.
- Fixed: `invalidateBudgetDependents` was invalidating `['dashboard']` even though the Dashboard reads no budget data — removed the speculative coupling; it now invalidates `['budgets']` only.
- Fixed: `BudgetProgressBar`'s `aria-valuenow` could exceed `aria-valuemax` (invalid ARIA range) for exceeded budgets — it's now capped at 100, with the true percentage exposed via `aria-valuetext`.
- Audited: the Budget Detail contributing-transaction list. `useTransactions()` applies no `limit`/pagination parameter and is scoped to the same Asia/Jakarta reporting-month range (`getReportingMonthRange`) the Budget's own `periodStart`/`periodEnd` come from — so the client-side category filter yields the complete per-period set, not a truncated batch. Copy ("Transactions this period" / "Transaksi periode ini") was already honestly scoped and needed no change.
- Live-browser/manual UI verification was performed in a later session (Phase B5, below) — the B4-era gap noted here is closed.

**Phase B5 (authenticated browser QA) is complete.** Desktop (1440×900), tablet (768×1024), and mobile (390×844, 375×812) QA passed for the full authenticated Budget flow (create, spend, recalculate, edit, archive, restore). Runtime request payloads matched the approved API contract, and accessibility verification passed for progress values above 100%. Two defects were found during this pass and are tracked/fixed separately from Budgeting scope (see Phase B6): a duplicate-DOM-id bug in the shared `FormField` component, and a missing app-wide route guard that let anonymous sessions reach the authenticated shell.

**Phase B6 (release closure and route-guard hardening) — this session (2026-07-21).** Scope was deliberately split from Budgeting: the route guard is an app-wide prerequisite fix, not Budget functionality, and is prepared as an independently reviewable change.

- Root cause of the missing route guard: `lib/supabase/middleware.ts` (invoked via `proxy.ts` — Next.js 16 renamed `middleware.ts` to `proxy.ts`; the proxy **was** running) used a hardcoded `protectedRoutes` allow-list that was never extended for newer routes. `/anggaran`, `/cicilan`, `/target-tabungan`, and `/notifications` were absent from it, so anonymous requests to those paths were never redirected. Fixed by inverting the check to a small public-path allow-list (`/`, `/login`, `/changelog`, `/auth/*`) with every other route protected by default, so a new `(app)` route is protected automatically instead of requiring a manual addition.
- The guard already used `supabase.auth.getUser()` (server-verified), not `getSession()`, and already ran at the proxy layer before rendering — no double client-side guard was added.
- No return-path/redirect-query parameter exists or was introduced; the anonymous redirect target is always `/login` (YAGNI — this fix scope did not need one).
- Regression coverage added: `tests/route-guard.test.ts` (behavioral — invokes the real `updateSession` function against a mocked Supabase client and a real `NextRequest`, covering anonymous-redirect, authenticated-passthrough, public-route-passthrough, and no-redirect-loop cases across the full route matrix) and `tests/private-route-auth.test.ts` was rewritten from asserting the old hardcoded list to asserting the new allow-list shape and rejecting a regression back to a hardcoded list.
- `components/ui/form-field.tsx`'s existing duplicate-id guard (skip cloning `id`/`aria-*` onto a plain layout `<div>` child) was verified against all 10 `FormField` usages in the frontend; every non-`<div>` child is a native `<input>`/`MoneyInput`/`Select`, so the guard does not suppress `aria-invalid`/`aria-describedby` propagation anywhere else. Added `tests/form-field-id.test.ts`, a behavioral test via `react-dom/server`'s `renderToStaticMarkup` (no new dependency — `react-dom` is already installed), which actually renders the component and parses the HTML output, unlike the source-contract convention used elsewhere in this suite.
- Full local validation passed: backend `prisma validate`, `tsc --noEmit`, `vitest run` (642 passed, 32 integration tests correctly skipped without `TEST_DATABASE_URL`), `npm run build`; frontend `vitest run` (530 passed), `tsc --noEmit`, `eslint .` (no issues), `npm run build`.
- Not performed this session: an actual GitHub Actions run of the backend CI Postgres integration gate (requires a real push/PR), and disposable-Postgres integration re-run locally (no Docker/local Postgres available in this environment — no backend source was changed this session, so no regression risk was introduced to the integration-tested paths).

Budgeting is **CONDITIONALLY READY**, not release-verified — it remains gated on an actual GitHub Actions run of the Postgres integration workflow. Historical periods, delete, category reassignment, the dashboard summary slot, and threshold notifications remain deferred by product decision/task scope, not by omission.

**Repositories**

- `pocket-mint-docs` owns [PD-009](../product/decisions/009-budgeting-scope.md) (Approved), the [Budgeting Readiness Audit](./budgeting-readiness-audit.md), the [Budgeting Calculation Specification](./budgeting-calculation-spec.md), and the reworded Non-Goals language (synchronized in `product-rfc.md` and `vision.md`).
- `pocket-mint-be` owns the `Budget` model and migration (done, Phase A), the calculation/query service (done, Phase A), the command service and validation (done, Phase B1 — `src/services/budget.service.ts`), and the HTTP API layer (done, Phase B2 — `src/controllers/budget.controller.ts`, `src/routes/budgetRoutes.ts`). Remaining: B3 (frontend screens and navigation), and — in a later, separately scoped phase — the `BudgetThresholdEvent` model and threshold evaluation (PD-009 Decision F), which must not fail or roll back a valid transaction mutation.
- `pocket-mint-fe` owns the Budgets navigation entry, list/detail/create/edit screens (done, Phase B3), and — in later, separately scoped phases — the dashboard summary slot and threshold notification presentation.

**Dependencies**

- Phases 1 through 4 (Financial Core, Authentication, Wallet Engine, Transaction and Transfer Engine) for a stable, User-scoped `Category` and `Transaction` model.
- [PD-009](../product/decisions/009-budgeting-scope.md) approval, including its specific reworded Non-Goals text — satisfied.

**Blocked By**

- Nothing for Phase A, Phase B1, or Phase B2 (all complete). Phase B3 (frontend) is not blocked by any open Product Decision, only by scheduling.
- The notification-integration phase (Decision F) requires its own explicit review of the failure-isolation requirement before implementation.

**Enables**

- A Budget-aware Dashboard summary section (pending Phase B/frontend).
- A foundation for a later overall (all-category) budget, if validated.
- A category-scoped aggregation query (`budget-query.service.ts`'s grouped `transaction.groupBy`) that Analytics v2 may also reuse.

**Risk**

- **Medium:** the installment `monthlyAmount` vs `grandTotal` nuance must be applied identically in every consumer or budget figures will silently disagree with the Dashboard's existing monthly summary — `budget-query.service.ts` reads the persisted `Transaction.amount` as-is, consistent with `transaction-query.service.ts`'s `getSummary`, so this is satisfied for Phase A.

### Phase 11 — Analytics v2

**Purpose**

Upgrade Pocket Mint's existing analytics and reporting capability into a reliable, useful, explainable, and reusable Analytics v2 experience: backend-authoritative aggregation (not client-side full-transaction-dump computation), period-scoped overview with previous-period comparison, contiguous trend bucketing, category and wallet breakdowns, budget performance reuse, and transaction drill-down — all governed by one documented metric policy.

**Status**

**Backend is implemented** (`pocket-mint-be`, branch `feature/analytics-v2`, PR [#61](https://github.com/acuyaldi/pocket-mint-backend/pull/61) open against `dev`, not merged): a dedicated `analytics` module with internal separation for period resolution (`src/domain/analyticsPeriod.ts`, built on `reportingTime.ts`), metric aggregation (`src/services/analytics-*.service.ts`), validation (`src/services/analytics-period.ts`, `src/services/analytics.errors.ts`), response mapping (`src/controllers/analytics.controller.ts`), and routes (`src/routes/analyticsRoutes.ts`, mounted at `/api/v1/analytics`). Six endpoints: overview (with previous-period comparison and zero-baseline `{ value: null, reason: "ZERO_BASELINE" }` handling), trends (daily/monthly bucketing, zero-filled), categories (by type, explicit Uncategorized group), wallets (per-wallet income/expense/net), budget-performance (verbatim `budgetQueryService.listActiveBudgetUsage` reuse — numerically identical to `GET /budgets`), and transactions (drill-down, paginated, capped at 200/page, reusing the canonical transaction shape). All endpoints are `requireUser`-gated, scoped to the authenticated user. No new migration; the existing `[userId, date]` and `[walletId, date]` indexes were confirmed sufficient by `EXPLAIN ANALYZE` against a 100k-row seeded dataset (sub-2ms, no sequential scans). Full test suite: 712 unit/boundary tests + 42 integration tests (real Postgres) covering Decimal precision, cross-user isolation, uncategorized handling, transfer exclusion, zero-baseline percentage, half-open date boundaries, and budget-performance numeric agreement with `GET /budgets`.

**Frontend is implemented** (`pocket-mint-fe`, branch `feature/analytics-v2`, PR open against `dev`, not merged): the existing `/analytics` page is rewired from client-side full-transaction-dump computation (`useAllTransactions` + `buildMonthlyFlow`/`filterTransactionsByPeriod`) to backend-authoritative aggregation via dedicated `useAnalytics*` TanStack Query hooks (`src/features/analytics/hooks/useAnalytics.ts`). Period state is URL-persisted via `parseAnalyticsSearchParams`/`serializeAnalyticsSearchParams` (`src/features/analytics/period.ts`). A single chart library was added: Recharts 3.10.0, used for the cash-flow trend (`CashFlowTrend` — ComposedChart with income/expense bars and net line), category breakdown (`CategoryBreakdown` — horizontal bar chart with accessible data table), and wallet breakdown (`WalletBreakdown` — semantic HTML table, no chart). Every chart has an accessible non-color-only fallback (screen-reader summaries and/or accessible data tables in the DOM). The period selector supports six presets plus a custom date range (`AnalyticsPeriodSelector`), all with native `<input type="date">` inputs. Overview cards (`AnalyticsSummaryCard`) display income/expense/net/transaction-count with previous-period comparison and explicit "no comparison" state for zero-baseline percentage. Budget performance reuses `BudgetStatusPill`/`BudgetProgressBar` from `app/(app)/anggaran/components/` for visual consistency. Transaction drill-down (`TransactionDrillDown`) is a paginated table with type-filter, category-filter, and wallet-filter support. The i18n `"analytics"` namespace in both `en.json`/`id.json` was replaced with new keys for period-selector, overview, trends, categories, wallets, budget-performance, and drill-down sections — no hardcoded strings. Storybook build passes; source-contract tests updated to check for the new hooks and export patterns rather than the old `useAllTransactions`/client-side computation. The existing CSV export button is preserved, mapped from the new period presets to the prior export endpoint's values where compatible.

**Docs is implemented** (`pocket-mint-docs`, branch `feature/analytics-v2`, not merged): [Analytics v2 Calculation Specification](./analytics-v2-calculation-spec.md) (metric definitions, period/timezone policy, previous-period comparison rules, zero-baseline policy, trend bucketing, budget performance reuse policy, known limitations), [Analytics v2 API Contract](./analytics-v2-api-contract.md) (endpoint contracts, auth, error codes, response shapes), and [ADR-010 — Analytics v2 Architecture](../product/decisions/010-analytics-v2-architecture.md) (decision: live aggregation from canonical Ledger facts; terminology reconciliation between "Analytics" and PD-006's "Reports"). PD-006's History table updated with a dated entry noting the implementation.

**Repositories**

- `pocket-mint-be` owns the Analytics v2 module (`src/domain/analyticsPeriod.ts`, `src/services/analytics-*.service.ts`, `src/services/analytics.errors.ts`, `src/controllers/analytics.controller.ts`, `src/routes/analyticsRoutes.ts`) and its tests (unit + integration + isolation).
- `pocket-mint-fe` owns the rewired `/analytics` page (`app/(app)/analytics/page.tsx`), its components (`components/AnalyticsPeriodSelector.tsx`, `AnalyticsSummaryCard.tsx`, `CashFlowTrend.tsx`, `CategoryBreakdown.tsx`, `WalletBreakdown.tsx`, `BudgetPerformance.tsx`, `TransactionDrillDown.tsx`), the analytics hooks and types (`src/features/analytics/`, `src/types/analytics.ts`), and the Recharts 3.10.0 dependency.
- `pocket-mint-docs` owns [ADR-010](../product/decisions/010-analytics-v2-architecture.md), the [Calculation Specification](./analytics-v2-calculation-spec.md), and the [API Contract](./analytics-v2-api-contract.md).

**Dependencies**

- Phases 1–5 (Financial Core, Authentication, Wallet Engine, Transaction Engine, Installment Engine) for the canonical `Transaction`/`Category`/`Wallet`/`Budget` models and `reportingTime.ts` primitives.
- Phase 10 (Budgeting) for the `computeBudgetUsage` pure function and `budgetQueryService.listActiveBudgetUsage` that Analytics v2's budget-performance endpoint reuses verbatim.

**Blocked By**

- Nothing. All prerequisite phases and product decisions are complete.

**Enables**

- A consistent, documented, and test-verified metric policy that all future features (Smart Categorization, Rule Engine, Automation) can consume through the same `/api/v1/analytics` contracts.
- A drill-down pathway from any metric to its constituent transactions.
- A single chart library (Recharts) with established accessible-fallback patterns usable by other product areas.

**Risk**

- **Low:** aggregation latency at very large per-user transaction volumes. Existing indexes confirmed sufficient; a targeted `[userId, type, date]` composite index and/or materialized-view evaluation is gated behind a measured production threshold (~500k rows per user, p95 >500ms for the overview endpoint), not a speculative pre-build.
- **Low:** Recharts dependency. The accessible data-table fallback for every chart ensures no information is lost during any migration period.

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

### Phase 21 — Assistant Core

**Purpose**

Build the conversational Assistant boundary defined in [Assistant Core Architecture](../architecture/assistant-core-architecture.md): a provider-neutral orchestration layer that queries and proposes mutations through the existing Backend and Business Services, never around them.

**Repositories**

- `pocket-mint-docs` owns the Assistant Core ADR, canonical terminology, risk/policy definitions, and lifecycle contracts.
- `pocket-mint-be` owns the Assistant boundary itself: Conversation Manager, Intent Resolver, Bounded Workflows, Execution Engine, Risk and Policy Gate, Tool Router, Tool Registry, Draft Store, Conversation Store, Audit Log, and the provider adapter. It reuses existing domain services unchanged.
- `pocket-mint-fe` owns the conversational surface, draft/preview presentation, and explicit confirmation controls.

**Dependencies**

- Phases 1 through 8.
- Stable authenticated Backend contracts and Canonical Calculations.

**Sub-Phases**

- **21.1 — Documentation and Contracts:** ADR (complete), canonical Assistant types, initial tool contract, risk/confirmation enums, lifecycle state definitions. ✅ **Complete.** Module at `src/assistant/` in `pocket-mint-be`. Canonical contracts: `ToolContract`, `ExecutionContext`, `PolicyResult`, `ToolRegistry`, `evaluatePolicy`. First tool: `analytics.monthly-spending-summary` (LOW risk, read-only, no confirmation). Validation via minimal internal validators (no Zod/Joi — compatible with adopting them later).
- **21.2 — Read-Only Assistant Foundation:** ✅ **Implemented.** Provider-neutral canonical request/response, correlation ID middleware, deterministic intent resolver, consolidated executor/tool-router, tool handler wired to existing `transactionQueryService` + `analyticsCategoriesService`, deterministic Indonesian response renderer, HTTP endpoint `POST /v1/assistant/execute`, structured audit logging via existing logger. **No LLM provider integrated.** Conversation persistence and draft persistence remain deferred. Durable audit persistence remains deferred.
- **21.3 — Conversation Persistence:** ✅ **Implemented.** User-owned conversations, canonical messages with source provenance, explicit request turns, minimized durable tool-execution history, bounded ownership-scoped retrieval, and idempotent archive. Automatic expiration and permanent deletion remain deferred pending an approved retention policy.
- **21.4 — First Financial Draft Flow:** ✅ **Implemented.** `transaction.create` creates a 15-minute owner-scoped draft only; dedicated confirm/cancel endpoints enforce explicit confirmation, database-backed idempotency, PostgreSQL draft locking, atomic reuse of the existing transaction service, and minimized lifecycle audit history. Transfers, installments, providers, channels, frontend work, and automatic stale-draft cleanup remain out of scope.
- **21.5 — Assistant Memory, Context Assembly, Conversation Retrieval, Prompt Context Engine:** ✅ **Implemented.** Owner-scoped `AssistantContextService`; deterministic oldest-to-newest DTO assembly from bounded newest-first reads; configurable 40-message, 20-turn, 10-tool, one-draft, and 64 KiB limits; protected current request/pending draft/latest Assistant response; ISO timestamps, canonical decimals, stable serialization, and hidden-field filtering. Four bounded Prisma reads avoid N+1 work. `AssistantApplicationService.prepareProviderExecution` is internal-only and read-only; the Phase 21.4 execute path does not invoke it, and no endpoint was added.
- **21.6 — Provider Runtime:** ✅ **Implemented.** Explicit natural-language `POST /api/v1/assistant/messages`; provider-neutral request/response/error/usage contracts; deterministic registry-derived capability catalog and prompt assembly; strict structured plan validation; one official Gemini adapter with retries disabled, bounded output, timeout/cancellation, and safe error mapping; minimized provider audit; and delegation to existing read/draft execution paths. Initial `transaction.create` still produces only a pending draft and requires the unchanged explicit confirmation endpoint. Duplicate message submissions remain independent requests and may create independent drafts.
- **Later — Proactive Domain-Event Workflows:** deferred until provider-backed conversational request/response behavior is production-ready.

**Blocked By**

- Any request to give the Assistant a database path that bypasses the Tool Router, Business Services, or existing authorization.
- Any request to commit a financial write without a Draft → Preview → Explicit Confirmation → Commit cycle.

**Enables**

- Conversational read access to a User's own financial data.
- Conversational initiation of financial writes under the same authorization and confirmation rules as every other client.

**Risk**

- **High:** an under-validated tool contract or a weakened confirmation path would let untrusted model output affect financial state.

### Phase 21.5 Implementation Note (2026-07-23)

The Context Engine treats persisted conversations as authoritative and `AssistantContext` as a disposable derived object. Ownership is checked before child history reads; unknown and cross-User IDs remain indistinguishable, and archived conversations cannot be prepared for continued execution. After ownership succeeds, a bounded owner-scoped messages query joins turn metadata and includes the latest Assistant response in one SQL statement, while one unexpired pending draft and recent tool executions are retrieved as the other two parallel reads. Together with the ownership lookup this is exactly four SQL statements, independent of conversation size. Expired drafts are omitted without a lazy status update.

DTOs expose conversation timestamps/state, canonical messages and turns, only the public draft projection, safe tool summaries, and the explicit current request. `conversationId` is deliberately classified as a public Assistant identifier; owner/database/internal IDs other than `conversationId` and `draftId` are omitted. Recursive explicit normalized-key filtering removes policy, risk, Prisma, correlation, stack, SQL, lock, idempotency, audit, raw payload/argument/output, execution metadata, reasoning, credential/token/secret, balance, and prototype fields while preserving legitimate keys that merely contain those words. Serialization uses UTC ISO timestamps, decimal strings with redundant trailing zeroes removed, stable object keys, and a UTF-8 64 KiB ceiling. Whole optional turns or tool entries are trimmed oldest first; the pending draft is immediately before the current request, and protected content is never truncated.

`AssistantApplicationService.prepareProviderExecution` is deliberately not called by `execute`. It accepts an unpersisted current User request, delegates exactly once to ContextService, and returns the deterministic DTO; callers must not persist that same request first. Invalid limits, unsupported safe-summary data, persistence/serialization failures, and protected-content overflow produce safe deterministic Assistant errors and no writes. Phase 21.6 now provides its first production consumer through the separate `/assistant/messages` runtime; Phase 21.5 itself introduced no LLM SDK, provider call, prompt execution, tool change, mutation, cache, embedding, semantic search, worker, streaming, or channel integration.

### Phase 21.6 Implementation Note (2026-07-23)

The provider runtime is opt-in and does not change the canonical `/assistant/execute` path. With `ASSISTANT_PROVIDER=gemini`, `ASSISTANT_MODEL`, and `GEMINI_API_KEY`, `/assistant/messages` establishes or ownership-validates a conversation, calls `prepareProviderExecution` exactly once while the current request remains unpersisted, builds a stable system instruction and registry-derived capability catalog, and invokes Gemini outside any Prisma transaction. The official `@google/genai` adapter requests closed-schema JSON, caps generation at 4,096 output tokens, forwards cancellation, applies the configured 15-second default timeout, and sets retry attempts to one.

The backend accepts only `intent`, `clarification`, or `unsupported` plans with exact fields and bounded text/object depth. It rejects non-terminal responses, mixed-case or compatibility-normalized forbidden claims, secret-seeking clarifications, links, executable markup, and control characters. Registry lookup, existing input validators, and existing policy run again before delegation. Known intent output and all authoritative financial text remain deterministic. Provider calls have their own minimized `AssistantProviderExecution` lifecycle rather than masquerading as tool executions; no prompt, context, raw response, financial arguments, hidden reasoning, headers, credentials, or raw SDK error is stored.

The current request is persisted once after planning through the existing execution lifecycle or one non-tool terminal turn. A model cannot confirm a draft, select an authenticated User, assert ownership/authorization, or mark a mutation successful. `/messages` has no request-idempotency contract: duplicate or concurrent submissions can create independent turns and drafts, while confirmation idempotency remains scoped to one draft. A metadata-only audit-finalization failure after durable execution can leave the provider audit `STARTED`, but it does not replace the durable outcome with a retry-triggering error. Frontend chat, external channels, provider failover/routing, automatic retry, streaming, tool loops, multi-step autonomous planning, embeddings, semantic retrieval, background recovery, transfers, installments, recurring mutations, and direct natural-language confirmation remain deferred.

### Phase 21.1 Implementation Note (2026-07-22, corrected 2026-07-22)

**Module path:** `src/assistant/` in `pocket-mint-be` — a self-contained module following existing flat-layered repo conventions. Internal structure: `types.ts` (canonical types), `errors.ts` (domain errors), `policy.ts` (deterministic policy evaluator), `registry.ts` (static tool registry), `tools.ts` (tool contract definitions), `bootstrap.ts` (startup wiring), `executor.ts` (consolidated executor + tool router), `intent.ts` (deterministic intent resolver), `renderer.ts` (Indonesian response renderer), `handlers/` (tool implementations), `index.ts` (barrel).

**Corrections applied after initial 21.1 review:**

- **Risk-policy mapping:** `evaluatePolicy` now maps by `confirmationPolicy` rather than inferring from `riskLevel` alone. HIGH risk defaults to EXPLICIT (draft + confirm), not STEP_UP. STEP_UP is an optional strengthening available to any tier. VERY_HIGH always maps to UNAVAILABLE in v1.
- **Registry invariants:** MEDIUM and HIGH both require at least EXPLICIT. HIGH no longer requires STEP_UP. VERY_HIGH requires DISABLED.
- **Removed `_context` parameter from `evaluatePolicy`** — unused in the current deterministic implementation; preference memory remains deferred beyond Phase 21.3.
- **Money representation:** Tool boundary uses `number` via `Number(decimal.toString())`, matching the existing controller convention. No financial arithmetic is performed with JS numbers inside Assistant Core.

### Phase 21.2 Implementation Note (2026-07-22)

**Canonical request/response:**

- Request: `{ intent: string, arguments: unknown, conversationId?: string, locale?: string }`
- Response: `{ status: 'success' | 'clarification_required' | 'rejected' | 'error', renderedText, data, correlationId }`

**HTTP route:** `POST /api/v1/assistant/execute` — authenticated via existing `requireUser` middleware. Only the allow-listed intent `analytics.monthly-spending-summary` is accepted. Callers cannot send arbitrary tool IDs.

**Correlation ID:** Fresh UUIDv4 generated per request via `correlationMiddleware`. Returned in `X-Correlation-Id` response header and in the response body. Caller-supplied IDs are not accepted.

**Intent resolver:** Deterministic allow-list (`resolveIntent`). One supported intent: `analytics.monthly-spending-summary`. A future provider adapter will translate NL into this canonical structure.

**Executor / Tool Router:** Consolidated into one component (`executeTool`). Resolves tool → checks enabled → evaluates policy → rejects non-immediate → validates input → resolves handler → executes with timeout → validates output → returns structured result. Logs execution metadata with correlation ID via existing logger.

**Tool handler:** `handleMonthlySpendingSummary` calls `transactionQueryService.getSummary` (P&L) and `analyticsCategoriesService.getCategoryBreakdown` (top expense categories). Parses `YYYY-MM` → Jakarta calendar month via existing `getReportingMonthRange`. Serializes `Prisma.Decimal` → `number` via `Number(decimal.toString())`. Uses `userId` from `ExecutionContext` — never from tool input.

**Renderer:** `renderMonthlySpendingSummary` produces deterministic Indonesian text. Handles: positive/negative net savings, zero transactions, empty categories. Uses `Intl.NumberFormat('id-ID')` for rupiah formatting.

**Audit logging:** Structured log entries via existing `logger.info('assistant_tool_execution', ...)` with correlation ID, userId, toolId, status, durationMs. No durable database audit persistence — deferred per ADR.

**Explicitly NOT implemented:**
- No LLM provider adapter (OpenAI, Anthropic, Gemini, etc.)
- No prompt engineering
- No frontend chat
- No conversation/message Prisma tables
- No draft persistence
- No transaction creation through the Phase 21.2 read-only intent; Phase 21.4 adds only explicitly confirmed draft commits
- No general planner or workflow DAG
- No semantic memory
- No preference persistence
- No proactive events
- No Redis, message queues, WebSocket, or streaming
- No general arbitrary-tool execution endpoint

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
