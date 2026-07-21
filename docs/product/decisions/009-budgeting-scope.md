---
id: PD-009
decision_number: 009
title: Budgeting Scope and MVP Definition
version: 1.1
status: Approved
priority: P1
owner: Product
created: 2026-07-21
updated: 2026-07-21

related:
  rfc: docs/product/product-rfc.md#budgeting
  design: docs/product/design-system.md
  screen: docs/product/screen-spec.md#budgets
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-fe
  - pocket-mint-be

tags:
  - budgeting
  - category
  - reporting
  - notifications

ai_context:
  required_before:
    - docs/product/product-rfc.md
    - docs/product/vision.md
    - docs/product/glossary.md
    - docs/development/budgeting-readiness-audit.md
    - docs/development/budgeting-calculation-spec.md
  update_after:
    - docs/product/product-rfc.md
    - docs/product/vision.md
    - docs/product/glossary.md
    - docs/product/screen-spec.md
    - docs/development/implementation-roadmap.md
---

# Summary

Introduce Budgeting as a per-category monthly spending limit, calculated dynamically from posted `EXPENSE` transactions against the user's existing `Category` model. No envelope allocation, no rollover, no forecasting, no AI. This narrows and supersedes the "budgeting-envelope application" Non-Goal in the Product RFC and Vision, which must be reworded to distinguish spend-limit tracking from envelope budgeting.

---

# Problem

Pocket Mint's roadmap names Budgeting as the next product phase after the current stable release. No Budget concept exists in the schema, backend, frontend, or documentation today. The Product RFC (`Non Goals`) and Vision (`Non-goals`) both currently list "a budgeting-envelope application" as something Pocket Mint explicitly is not, and several design documents warn against resembling a "budgeting game." Implementing any feature literally named "Budgeting" without first resolving this contradiction would either violate stated product boundaries or require silently reinterpreting them — both prohibited by the documentation authority order in `docs/ai/project-context.md`.

Separately, no aggregation query, notification type, or data model exists yet to answer the user-facing questions this feature must answer: how much may I spend, how much have I spent, how much remains, and when am I approaching or exceeding a limit.

---

# Current Situation

## Current Implementation

- `Category` (`pocket-mint-be/prisma/schema.prisma:101-119`) is a real, user-scoped model with a `type` (`INCOME`/`EXPENSE`) and default-seeded rows per user. No `Budget` model exists.
- `Transaction.categoryId` links transactions to categories; `EXPENSE`/`INCOME` transactions require a category at creation.
- The only existing monthly aggregation (`transaction-query.service.ts:getSummary`) groups by `type`, not `category`.
- The notification-equivalent model (`RecurringReminderEvent`) has a proven idempotent-threshold pattern (unique constraint + upsert) but no percentage-threshold concept.
- Full evidence is recorded in [Budgeting Readiness Audit](../../development/budgeting-readiness-audit.md).

## Current Documentation

- `product-rfc.md` Non Goals: "a budgeting-envelope application."
- `vision.md` Non-goals: "A budgeting-envelope system."
- `glossary.md` has no `Budget` term.
- `screen-spec.md` has no Budget screen.
- `implementation-roadmap.md` Engineering Phases end at Phase 9 (UX Polish); no Budgeting phase exists.

## Current User Impact

Users cannot set or see a spending limit anywhere in the product today. Category data exists but is inert metadata (confirmed by `pocket-mint-fe` financial-logic guidance: "Category is metadata — not used in core calculations").

---

# Options

## Option A — Category-scoped monthly spend limits, dynamically calculated, no envelope allocation

One optional `Budget` per user per `Category`, expressed as a fixed monthly amount. Usage is always computed live from posted `EXPENSE` transactions in the current calendar month (Reporting Timezone) — never stored or cached. No fund allocation, no rollover, no wallet-specific or overall budgets in v1.

Pros

- Directly answers the five product-intent questions with the smallest possible model.
- Reuses existing `Category`, `Transaction`, and `reportingTime.ts` primitives almost entirely.
- Clearly distinguishable from "envelope budgeting" (which allocates and moves finite funds between categories) — resolves the Non-Goals conflict by narrowing scope rather than ignoring it.
- No snapshot/cache invalidation logic needed, because nothing is cached.

Cons

- Users cannot set an overall (all-category) limit in v1.
- No historical period browsing in v1 (see Decision J).
- Requires an approved wording change to two Non-Goals statements.

## Option B — Full envelope budgeting (allocate total income across category "envelopes")

Pros

- Matches the literal meaning of "budgeting" in some finance apps.

Cons

- Directly contradicts the existing, unrevised Non-Goals in both `product-rfc.md` and `vision.md`.
- Requires tracking allocated-vs-available funds per envelope, a materially larger and riskier data model.
- Explicitly out of the product's stated calm, non-gamified identity per `design-principles.md`.

## Option C — Defer Budgeting entirely until Analytics v2 ships

Pros

- Avoids any Non-Goals conflict for now.

Cons

- Contradicts the explicit roadmap ordering (Budgeting precedes Analytics v2) provided for this iteration.
- Does not answer why the audit and design work were commissioned this iteration.

---

# Recommendation

Adopt Option A. It is the option repository evidence directly supports, the option that best matches the "deliberately narrow MVP" instruction, and the option that resolves rather than ignores the Non-Goals conflict.

Evidence: [Budgeting Readiness Audit](../../development/budgeting-readiness-audit.md) §§1–10 show every needed primitive already exists except a category-scoped aggregation query and a threshold-notification type, both small, well-understood additions. §11 shows the Non-Goals conflict is real and must be resolved explicitly, not silently.

Reasoning: A spend-limit tracked against recorded transactions is categorically different from envelope budgeting (which allocates a finite pool of funds and typically supports rollover between envelopes). Naming that distinction explicitly in the Non-Goals text lets Budgeting proceed without abandoning the product's stated identity.

---

# Detailed Product Decisions

## A. Budget Scope — Category budgets only

**Decision:** MVP supports one budget per user per expense `Category`. No overall (all-category) budget, no wallet-specific budget, in v1.

**Rationale:** Category budgets alone answer every question in the product intent (how much may I spend / have I spent / remains / approaching / exceeding / which categories). An overall budget is a straightforward future addition (sum of category budgets, or a distinct `categoryId: null` row) once the category-scoped mechanism is proven; building both simultaneously doubles the uniqueness, notification, and UI surface for no validated user need. Wallet-specific budgets are deferred indefinitely — Pocket Mint's mental model is category-based spending, not wallet-based, and no evidence in the frontend or backend suggests wallet-scoped limits are expected.

**Deferred:** overall monthly budget (fast-follow candidate), wallet-scoped budgets (not currently justified).

## B. Budget Period — Calendar month only, no rollover

**Decision:** A `Budget` is a **recurring monthly definition** — not a period-specific row. Its usage window for "the current period" is always `getReportingMonthRange(now, REPORTING_TIMEZONE)`. No weekly, custom-range, or one-time budgets in v1.

**Rationale:** Matches the existing Reporting Period convention used everywhere else in the product (dashboard, analytics). Reusing `getReportingMonthRange` means budgets automatically "renew" every month with zero scheduled job, because nothing is materialized per period.

**Deferred:** weekly periods, custom date ranges.

## C. Budget Basis — What counts as spend

**Decision:** Budget Spent = the sum of the persisted `amount` field of a user's posted `EXPENSE` transactions whose `categoryId` equals the budget's category and whose `date` falls within the current Reporting Period (half-open `[start, end)` in `REPORTING_TIMEZONE`).

Explicitly:

| Record type | Counts toward Budget Spent? |
|---|---|
| Posted `EXPENSE` transaction, matching category | Yes |
| Posted `EXPENSE` transaction, installment purchase (`isInstallment: true`) | Yes, at its persisted `monthlyAmount` — **not** `grandTotal`. Consistent with the existing monthly summary (`transaction-query.service.ts`) so a user never sees two different "this month's expense" numbers for the same data. |
| `INCOME` | No |
| `TRANSFER` (including installment repayments and debt/credit-card payments) | No — transfers relocate money between the user's own wallets and are never spend, per the existing Financial Event model. |
| Recurring transaction template not yet confirmed into a real `Transaction` | No — only posted transactions ever affect any total in Pocket Mint today (Readiness Audit §7). |
| Refund / negative expense | Not applicable — Pocket Mint's schema and validation (`amount > 0`) have no negative-expense or refund concept. If a user needs to reverse an expense, they edit or delete the original transaction. |
| Deleted transaction | No — hard delete means it is simply absent from the next live query. No special "deleted" handling is implemented or needed. |
| Backdated transaction (dated in the current period, entered late) | Yes, counted in the period its `date` belongs to, regardless of `createdAt`. |
| Future-dated transaction (dated in a later period than today) | Counted only in **its own** period's aggregate, never the current one — no special-casing required because aggregation is purely date-range based. |
| Uncategorized `EXPENSE` (`categoryId: null`, only reachable via category deletion today) | Not counted by any category budget. Out of scope for an "Uncategorized" bucket in v1 — this state is rare (Readiness Audit §2) and category budgets are inherently category-specific. |

## D. Category Dependency

**Decision:** The current `Category` model is sufficient to implement Budgeting. No schema or endpoint change to `Category` is required to ship the MVP; users budget against their existing (seeded-default) categories.

**Decision (clarified):** A custom category-creation endpoint (`POST /categories`) is not a prerequisite for Budgeting MVP and is **not implemented** in Phase A. It remains a candidate non-blocking future improvement — since users today can only budget against the 7 seeded default expense categories — but is explicitly deferred, not scheduled. This is explicitly **not** Smart Categorization — no merchant mapping, no rule engine, no learned classification. Those remain a distinct, later roadmap phase and must not be pulled forward.

**Category applicability enforcement:** `Category` does not encode budget-eligibility (i.e. "may this category be budgeted") at the database level beyond its existing `type` (`INCOME`/`EXPENSE`) discriminator. No schema redesign is introduced to add one. Expense-category eligibility (a Budget's `categoryId` must reference an `EXPENSE`-type `Category` owned by the same user) is enforced by the application service layer at Budget create/update time, using the existing `Category.type` and `Category.userId` columns — the same pattern already used elsewhere in the codebase to enforce cross-model ownership without a foreign-key-level constraint.

## E. Budget Status

**Decision:** Five deterministic states, evaluated backend-side, never computed independently by the frontend:

| Status | Condition |
|---|---|
| `HEALTHY` | `percentUsed < 75` |
| `APPROACHING` | `75 <= percentUsed < 100` |
| `REACHED` | `percentUsed == 100` (exact) |
| `EXCEEDED` | `percentUsed > 100` |
| `ARCHIVED` | Budget is archived; not evaluated against the current period at all |

`REACHED` is split out from `EXCEEDED` as its own boundary state: hitting the limit exactly is a distinct, deterministic condition from going over it, and collapsing the two would make "exactly 100%" ambiguous between two different UI treatments. Evaluation order is `ARCHIVED` first, then `EXCEEDED` (`> 100`), then `REACHED` (`== 100`), then `APPROACHING` (`>= 75`), else `HEALTHY`, so a value at exactly 100% is never misreported as `APPROACHING` or `EXCEEDED`.

No `UPCOMING` or `COMPLETED` state exists in v1 because a Budget is a recurring definition, not a dated instance — it is either active (evaluated against the current month) or archived. Historical period states are out of scope in v1 (see Decision J).

**Correction to the original Draft:** the initial Draft of this decision defined only four states with `EXCEEDED` at `percentUsed >= 100`, matching [budgeting-calculation-spec.md](../../development/budgeting-calculation-spec.md) §5 at the time. This Approved decision supersedes that boundary rule; the calculation spec §5 is updated to the five-state table above in the same iteration that implements the calculation service (Phase A), so no consumer ever implements the superseded four-state rule.

## F. Notifications

**Decision:** Two thresholds only — 75% and 100% — evaluated after any transaction mutation (create/update/delete) that changes a categorized `EXPENSE` total for the current period, reusing the exact idempotency pattern already proven by `RecurringReminderEvent`: a composite unique constraint (`budgetId`, `periodStart`, `thresholdType`) plus `upsert(..., update: {})`, so re-evaluating never creates a duplicate.

- **No separate repeated "exceeded" event per increase above 100%.** Once the 100% threshold has fired for a period, further spend that pushes `percentUsed` higher (e.g. 120%, then 180%) does not fire additional events — the idempotent upsert on `(budgetId, periodStart, thresholdType)` structurally prevents it. `REACHED` (exactly 100%) and `EXCEEDED` (over 100%) are two distinct **statuses** (Decision E) but the notification layer still recognizes only the one 100% threshold crossing, not one event per status.
- **Failure isolation is a hard requirement, not an implementation detail.** Future notification evaluation must never make an otherwise-valid financial transaction mutation fail or roll back merely because notification/threshold-event creation fails. Whatever mechanism evaluates thresholds (synchronous post-commit step, outbox, or later background job) must be structured so a notification failure is logged/isolated and never unwinds or blocks the transaction mutation that triggered it.
- **Not implemented in this iteration.** Phase A (this decision's implementation scope) ships only the `Budget` model and calculation service. No `BudgetThresholdEvent` model, no threshold-evaluation code path, and no notification integration are implemented in Phase A — this section records the decision for a later, explicitly scoped notification phase.
- **No 90% tier** — two tiers is enough signal without becoming spammy; a third tier between 75% and 100% adds noise without adding a decision the user can act on differently.
- **No retraction.** A threshold notification is a historical fact ("you crossed 75% on this date"), not live state. If a later edit/deletion drops spend back under a threshold, the notification is not deleted or unsent — the live Budget Detail screen always shows the correct current numbers regardless of past notifications. This mirrors how `RecurringReminderEvent` never un-fires a reminder once created.
- **Period reset is automatic** — because the unique constraint is keyed by `periodStart`, next month's crossing of 75%/100% is a new, distinct row and fires again independently.
- **No `BudgetNotificationPreference` model in v1** — no per-user notification toggle exists yet for any Pocket Mint notification type today, so adding one exclusively for Budgeting would be a new, unscoped concept. Deferred until a general notification-preferences need is validated.
- **Evaluation trigger:** confirmed backend cron/scheduler existence is `UNVERIFIED` (Readiness Audit §6, §11). MVP must not assume one exists. Evaluate thresholds synchronously inside the same transaction-mutation service call that could change a budget's usage (mirrors the existing pattern where notification refresh happens on explicit user action, not a background job).

## G. Rollover

**Decision:** No rollover in v1. Each Reporting Period starts at zero spend against the same monthly `budgetAmount`, unconditionally.

**Rationale:** Rollover requires deciding what happens to overspend (carried as new debt against next month? forgiven?) and underspend (saved? forfeited?) — both meaningfully increase UX and accounting complexity for a feature whose stated design goal is to remain calm and precise, not to become a savings/allocation system. No repository evidence supports the complexity. Deferred.

## H. Planned vs. Actual

**Decision:** Budget Spent reflects **posted transactions only**. Recurring transaction templates and unconfirmed reminders never affect a budget's spent amount, matching the confirmed rule that only posted `Transaction` rows affect any total in Pocket Mint today.

**Deferred:** showing "committed" recurring amounts as a separate forecast figure inside Budget Detail. If added later, it must be visually and numerically distinct from Budget Spent, never summed into it.

## I. Multi-Currency

**Decision:** Budgets are implicitly single-currency (IDR), matching every other monetary concept in Pocket Mint today. No currency field, selector, or conversion logic is introduced.

## J. Deletion and Historical Integrity

**Decision:** No `BudgetPeriod` snapshot model in v1. A `Budget` is calculated purely from its current definition (`budgetAmount`, `categoryId`) plus live `Transaction` history for the **current** Reporting Period only. Historical period browsing (viewing a past month's budget performance) is explicitly **out of scope for v1** and deferred to a fast-follow.

**Why this is safe:** because v1 never displays a past period, editing a budget's amount or category has no retroactive effect to reconcile — it simply changes the limit compared against from this point forward. This directly avoids the reproducibility problem that a snapshot-less design would otherwise create (Pocket Mint's Historical Record principle requires past evaluations to remain reproducible; the only way to honor that without a snapshot is to not display history yet).

Specific rules:

- **Editing amount/category during an active period:** takes effect immediately; the live calculation uses the new definition on the next read. No confirmation step, because nothing is retroactively rewritten (only the current, undisplayed-as-history period is affected).
- **Archiving:** an archived budget is excluded from the active list, the dashboard summary, and threshold evaluation.
- **Deleting:** hard delete, consistent with the codebase's existing no-soft-delete convention for `Transaction`. A future `BudgetThresholdEvent` (Decision F, not implemented in Phase A) would cascade-delete with the budget when that model is introduced.
- **When historical period browsing is added later** (not in this decision's scope), that is exactly the point at which a lightweight, lazily-created `BudgetPeriod` snapshot should be introduced — once a period has fully elapsed, not before. This decision explicitly defers that model rather than building it speculatively now.

## K. Naming and Information Architecture

**Decision:** Canonical English term is **Budget**. Approved Indonesian UI label is **Anggaran**, following the existing convention where documentation uses the English canonical term and the UI localizes it (the same convention already used for "Cicilan"/"Installment").

| Canonical (English, documentation) | Indonesian UI label |
|---|---|
| Budget | Anggaran |
| Budget Limit | Batas Anggaran |
| Budget Spent | Terpakai |
| Budget Remaining | Tersisa |
| Budget Period | Periode Anggaran |
| Budget Status: Healthy / Approaching / Reached / Exceeded / Archived | Sehat / Mendekati Batas / Mencapai Batas / Melebihi Batas / Diarsipkan |

"Budget Limit" (not "Limit" alone) is used to avoid collision with the already-canonical "Credit Limit" term in the Glossary.

## L. Budget Uniqueness (Clarified)

**Decision:** MVP uses one persistent `Budget` definition per user and category, full-stop — not one *active* budget per user and category.

- Unique on `(userId, categoryId)`, enforced as an ordinary (non-partial) database unique constraint.
- Archiving a Budget does **not** free its category for a second Budget. There is never more than one `Budget` row per user+category, archived or not.
- To resume budgeting a category whose Budget was archived, the user restores (un-archives) or edits the existing row — they do not create a new one.
- **Rationale:** this supersedes the original Draft's proposal of a partial unique index (`WHERE is_archived = false`) scoped to active rows only. A partial unique index requires database-level enforcement Prisma cannot express directly, plus an interim application-layer check to cover the gap — two enforcement paths for one invariant. A single ordinary unique constraint on `(userId, categoryId)` is simpler, is fully expressible in the Prisma schema, has no interim-enforcement gap, and still lets a user reuse a category's budget history by restoring the archived row instead of losing it to a hard delete-and-recreate cycle.

---

# Proposed Data Model

## Budget

Purpose: one recurring monthly spend limit for one user's expense category.

| Field | Type | Notes |
|---|---|---|
| `id` | `String @id @default(cuid())` | |
| `userId` | `String` | FK → `User`, required. |
| `categoryId` | `String` | FK → `Category`, required. Must be a `CategoryType.EXPENSE` category owned by the same `userId`. |
| `amount` | `Decimal(15,2)` | The monthly Budget Limit. Must be `> 0`. |
| `isArchived` | `Boolean @default(false)` | Archive flag; archived budgets are excluded from active evaluation. |
| `createdAt` / `updatedAt` | `DateTime` | Standard timestamps. |

Constraints:

- `@@unique([userId, categoryId])` — one `Budget` per user and category, full-stop, per Decision L. An ordinary unique constraint, not a partial index; archiving does not free the category for a second Budget.
- `@@index([userId, isArchived])` for the active-list query.
- No `period` field — the model is a definition, not an instance. "Overlapping periods" (Decision J) is therefore structurally impossible: there is only ever one current period, computed on read.

No caching of `spent`, `remaining`, or `percentUsed` on this model — all three are derived at read time (see [Calculation Specification](../../development/budgeting-calculation-spec.md)). Caching is not introduced because no performance evidence justifies it yet (Readiness Audit §5 shows DB `groupBy` is already the established, fast pattern for this exact shape of aggregation).

## BudgetThresholdEvent (deferred — not part of Phase A)

Purpose: idempotent record of a threshold crossing, powering notifications; mirrors `RecurringReminderEvent`. Shape sketched here for forward reference only.

| Field | Type | Notes |
|---|---|---|
| `id` | `String @id @default(cuid())` | |
| `budgetId` | `String` | FK → `Budget`, `onDelete: Cascade`. |
| `userId` | `String` | Denormalized for query convenience, matches `Budget.userId`. |
| `periodStart` | `DateTime` | The Reporting Period's `startInclusive`, from `getReportingMonthRange`. Identifies which month this event belongs to. |
| `thresholdType` | `BudgetThresholdType` (`APPROACHING` \| `EXCEEDED`) | |
| `triggeredAt` | `DateTime @default(now())` | |
| `readAt` | `DateTime?` | Matches the existing read/unread convention. |

Constraints (for the later notification phase):

- `@@unique([budgetId, periodStart, thresholdType])` — the exact idempotency mechanism from Decision F.
- `@@index([userId, readAt])` for the notification list/unread-count query.

**Not created by this decision's Phase A implementation.** This model, its enum, and all threshold-evaluation code are explicitly out of scope until a dedicated notification-integration phase is scheduled (see Decision F).

**Not introduced:** `BudgetPeriod` (deferred per Decision J), `BudgetNotificationPreference` (deferred per Decision F). Both are explicitly named in the audit brief as concepts to evaluate; both are rejected for v1 with the reasoning above, not silently omitted.

---

# Final Decision

Status

Approved

Decision

Decisions A–L above are approved as the MVP scope for Budgeting. Budgeting is category-level monthly spend-limit tracking, derived live from posted `EXPENSE` transactions against the existing `Category` model — not an envelope-allocation system. This document, together with the [Budgeting Readiness Audit](../../development/budgeting-readiness-audit.md) and [Budgeting Calculation Specification](../../development/budgeting-calculation-spec.md) (updated in the same iteration to the five-state status table in Decision E), constitutes the implementation-ready design.

Approving this decision authorizes the domain foundation only: the `Budget` Prisma model and migration, and the backend calculation/query service (Phase A of the Implementation Roadmap's Budgeting phase). It does **not** authorize HTTP routes/controllers, frontend UI, dashboard integration, or notification integration — those remain separately scoped, later implementation tasks gated on their own review. Budgeting as a whole is **not** implemented by this decision; only its domain foundation is.

Approved By

Product (this iteration)

Approved On

2026-07-21

Rationale

Option A (category-scoped, dynamically-calculated spend limits) is the option repository evidence directly supports (Budgeting Readiness Audit §§1–10) and the option that resolves rather than ignores the Non-Goals conflict identified in the Problem section. The required clarifications — the five-state status model (Decision E), notification-failure isolation (Decision F), one-persistent-Budget-per-category uniqueness (Decision L), and the explicit deferral of custom category creation (Decision D) — close the remaining ambiguities that would otherwise have been decided implicitly during implementation. The reworded Non-Goals text in `product-rfc.md` and `vision.md` (see Documentation Impact) is approved as written there.

---

# Documentation Impact

- [x] `product-rfc.md` — reworded Non Goals to distinguish spend-limit Budgeting from "budgeting-envelope" allocation; Financial Rules subsection already present, confirmed current with Decision E's five-state model.
- [x] `vision.md` — reworded Non-goals identically.
- [x] `glossary.md` — Budget, Budget Limit, Budget Spent, Budget Remaining, Budget Period, Budget Status terms updated from Provisional to reflect this Approved decision, including the added `Reached` state.
- [ ] `screen-spec.md` — Budget screens remain Provisional; not updated this iteration (no frontend work in Phase A).
- [ ] `component-spec.md` — Budget-specific component contracts (progress bar, status badge) still needed once frontend implementation is scheduled.

---

# Implementation Impact

Frontend

Not implemented this iteration. Future scope (unchanged from the original proposal): new `src/features/budgets/` module (hooks, components) following the `wallets` feature template; new nav entry (`components/layout/app-sidebar.tsx`, `bottom-nav.tsx`); new `messages/{en,id}.json` keys; Budget summary slot on Dashboard between Wallet Overview and Recent Activity.

Backend

Phase A (this iteration): `Budget` Prisma model and migration; `src/services/budget-query.service.ts` (calculation/aggregation, read-only). No routes or controllers.

Deferred to a later phase: `src/controllers/budget.controller.ts`, `src/routes/budgetRoutes.ts`, Budget CRUD command service, and threshold evaluation hooked into the transaction mutation path.

Database

Phase A migration adds `Budget` only (see Data Model above). `BudgetThresholdEvent` and its enum are deferred to the notification-integration phase (Decision F) and are not part of this migration. No changes to existing tables.

Automation

None required for Phase A. Future threshold evaluation runs synchronously inside existing request paths, not a scheduled job (see Decision F), and must not fail or roll back a valid transaction mutation.

Migration Required

Yes, for Phase A — additive only (`Budget` table, indexes, FKs), no backfill of existing data required (a user with no budgets simply has none).

Estimated Effort

Phase A (this iteration): Small — one model, one migration, one calculation/query service, tests. Full Budgeting feature (routes, CRUD, frontend, notifications): Medium, most primitives reused, following the Implementation Roadmap's phased order.

---

# Risks

See the [Risk Register](../../development/budgeting-calculation-spec.md#risk-register) for the full, categorized list. Summary: the Non-Goals conflict (resolved by this Approval), the installment `monthlyAmount` vs `grandTotal` nuance (must be documented so frontend/backend never drift), and the unconfirmed scheduler existence (must not be assumed by future threshold evaluation design).

---

# Follow-up Tasks

- Implement Phase A: `Budget` Prisma model, migration, and calculation/query service (this iteration).
- Schedule Phase B: Budget CRUD command service, routes, and controllers.
- Schedule the frontend Budgets feature module and dashboard integration, gated on Phase B.
- Schedule the notification-integration phase implementing Decision F (`BudgetThresholdEvent`, threshold evaluation), with explicit failure-isolation review.
- Revisit `POST /categories` (custom category creation) as an independent, non-blocking candidate improvement (Decision D) if user need is validated.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-21 | Draft | Initial creation from the Budgeting Readiness Audit and product-intent brief. Not yet reviewed or approved. |
| 2026-07-21 | Approved | Approved with required clarifications: five-state Budget Status (added `REACHED`, Decision E), notification failure-isolation and single-100%-threshold semantics (Decision F), category-creation deferral and application-layer expense-category enforcement (Decision D), and one-persistent-Budget-per-category uniqueness superseding the original partial-unique-index proposal (Decision L). Non-Goals text finalized in `product-rfc.md` and `vision.md`. Authorizes Phase A (domain foundation) only; Budgeting overall remains in progress. |
| 2026-07-21 | Approved (Phase A.5 review) | Phase A.5 (UX and API contract review) completed against this decision's existing scope: [Budgeting API Contract](../../development/budgeting-api-contract.md) and [Budgeting User Journeys](../../development/budgeting-user-journeys.md) added; `screen-spec.md` updated from Draft-era language to Approved (checklist item now satisfied); Edit Budget section added. No decision content in this document changed — category-reassignment (forbidden), delete (omitted from MVP), and dashboard-integration timing (deferred) are implementation-level elaborations of already-approved Decisions E/J/L, not new product scope, recorded in full in the [Phase A.5 Design Review](../../development/budgeting-phase-a5-design-review.md). Still authorizes Phase A (domain foundation) only; Phase B (routes/controllers/CRUD) and the frontend phase remain unimplemented. |
| 2026-07-21 | Approved (Phase B1–B3 implemented) | Phase B1 (command service), B2 (HTTP API/routes/DTO), and B3 (frontend screens, forms, navigation, API integration in `pocket-mint-fe`) are implemented per this decision's existing scope — no new decision content. B3 delivers `app/(app)/anggaran` (Budgets list with Active/Archived toggle, Budget Detail with contributing transactions), Create/Edit modals, Archive/Restore confirmations, sidebar/bottom-nav entries, and `en`/`id` localization, following Decisions E (five-state status, backend-computed), J (current-period-only, no retroactive edit), and L (one Budget row per category, archive doesn't free the slot) exactly. Dashboard integration and threshold notifications (Decision F) remain deferred to later, separately scoped phases. Budgeting remains in progress, not released. |
