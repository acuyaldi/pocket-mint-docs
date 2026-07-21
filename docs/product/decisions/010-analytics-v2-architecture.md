---
id: PD-010
decision_number: "010"
title: Analytics v2 Architecture
version: "1.0"
status: Draft
priority: P1
owner: Product + Engineering
created: 2026-07-22
updated: 2026-07-22

related:
  rfc: docs/product/product-rfc.md
  design: docs/product/decisions/006-reporting.md

affected_repositories:
  - pocket-mint-be
  - pocket-mint-fe
  - pocket-mint-docs

tags:
  - analytics
  - reporting
  - architecture
  - aggregation
  - financial-data

ai_context:
  required_before:
  update_after:
---

# Summary

Analytics v2 derives authoritative metrics from canonical transactional data using live aggregation against the Ledger (the `Transaction` table) and Budget domain. No materialized views, precomputed snapshots, or caching layer are introduced at this scale. The existing `GET /dashboard/summary` and `GET /transactions/summary` endpoints are supplemented — not replaced — by a dedicated, user-scoped `/api/v1/analytics` module that provides period-scoped overview, trends, category/wallet breakdowns, budget performance, and drill-down.

This ADR also documents the terminology reconciliation between the product's chosen operative term ("Analytics") and the earlier unapproved PD-006 ("Financial Reporting Surface"), which proposed "Reports" as the canonical term before the implementation shipped.

---

# Problem

The existing Analytics page in `pocket-mint-fe` (`/analytics`) fetched every transaction to the browser (`GET /transactions/all`) and computed cash flow, category breakdowns, and text insights client-side using the `app/(app)/analytics/period.ts` helper module. Key metrics were missing entirely: previous-period comparison, trend bucketing beyond a fixed 6-month window, per-wallet income/expense breakdown (only asset/liability composition existed), budget performance in context of analytics, and any drill-down from a metric to its constituent transactions.

Users could not answer:
1. How did my cash flow change compared with the previous period?
2. Which wallets drive the most income and spending?
3. How is my actual spending performing against active budgets?
4. Which transactions produced a particular reported number?

A validated, consistent, backend-authoritative implementation was required, with all figures derived from the same inclusion rules (transfer exclusion, uncategorized handling, half-open date boundaries, Decimal precision) as the existing `GET /transactions/summary` and Budget domain.

---

# Current Situation

## Current Implementation

- `GET /dashboard/summary` — wallet-balance-based net worth (wallet-scoped, not period-scoped).
- `GET /transactions/summary?month=YYYY-MM` — single-period income/expense/net (groupBy, Decimal-safe).
- `GET /budgets` — per-budget usage with `computeBudgetUsage` (pure function, Decimal-safe, reusable).
- `src/domain/reportingTime.ts` — half-open, DST-safe reporting-calendar primitives plus `getPreviousReportingMonthRange` (implemented, unused by any consumer prior to Analytics v2).
- `pocket-mint-fe` `/analytics` page — client-side full-transaction-dump computation. No chart library; hand-rolled CSS-bar charts. Period filter limited to `month | quarter | six-months`. No drill-down, no budget-performance display.

## Current Documentation

- PD-006 — Financial Reporting Surface (Draft, unapproved) proposed a "Reports" surface separate from the Dashboard. The shipped product uses the term "Analytics" (route `/analytics`, nav label, i18n namespace `"analytics"`) — see §Terminology Reconciliation below.
- The Budgeting Calculation Specification and API Contract establish the documentation precedent reused by the Analytics v2 docs (`analytics-v2-calculation-spec.md`, `analytics-v2-api-contract.md`).

## Current User Impact

Analytics data was incomplete (no previous-period context, no wallet-income view, no budget-performance context) and architecturally fragile (every metric depended on a full uncapped transaction fetch to the browser). The page was a real, shipped feature — not a stub — but its answers were limited and its approach would not scale past small transaction volumes.

---

# Options

## Option A — Live aggregation from canonical Ledger facts (chosen)

Backend computes every metric from live `Prisma groupBy`/`aggregate` queries scoped to the authenticated user, using the same inclusion rules and Decimal-safe arithmetic as the existing monthly summary and Budget domain. Frontend fetches only the pre-computed response, never the raw transaction set.

**Pros:**
- Single source of truth for every metric (Ledger + Budget domain).
- Same Decimal-precision, half-open-period, andtransfer-exclusion rules as every existing financial calculation.
- No new infrastructure; no snapshot drift.
- Immediately consistent with `GET /budgets`, `GET /dashboard/summary`, and `GET /transactions/summary`.
- A frontend bug cannot misrepresent a metric (the browser receives an answer, not raw data to reinterpret).

**Cons:**
- Per-request aggregation cost grows with user transaction volume.
- If a user accumulates millions of transactions, aggregation latency could become noticeable.

**Mitigation:** an `EXPLAIN ANALYZE` pass against a seeded 100k-row dataset confirmed existing `[userId, date]` / `[walletId, date]` indexes are sufficient (sub-2ms, no sequential scans). The measured threshold for revisiting this decision is when a single user's `Transaction` table row count exceeds ~500k, at which point a targeted composite index on `[userId, type, date]` (no migration required in this iteration) would be audited first before evaluating materialized views.

## Option B — Mix dashboard widgets and analytics

Add more cards to the existing Dashboard page rather than extend the dedicated Analytics page.

**Rejected:** PD-001 and the design system explicitly define Dashboard as current-state (`Financial Position → Needs Attention → Quick Actions → Wallet Overview → Current Period Summary → Recent Activity`), not a trend-analysis surface. The already-live `/analytics` route and its existing nav entry is the correct home for period-scoped aggregation and trend reporting.

## Option C — Export-only

Offer only CSV/PDF export of raw transaction data and let users perform their own analysis.

**Rejected by PD-006's own recommendation** (Option A — Dedicated Reports surface), and `pocket-mint-fe`'s `/analytics` page is already a real, live feature at that surface — the question is how to make it correct and complete, not whether it should exist.

---

# Recommendation

**Option A — Live aggregation from canonical Ledger facts.**

The backend implementation confirms this is practical: every aggregation query uses existing indexes, Decimal-safe arithmetic, and the single `reportingTime.ts` period-resolution pipeline. The Budget domain's `computeBudgetUsage` is reused verbatim — no separate formula was written for Analytics v2. The complexity budget is already absorbed (six focused endpoints, one dedicated analytics module with internal separation, ~2k lines including tests, no new infrastructure).

---

# Final Decision

| Field | Value |
|---|---|
| **Status** | Draft |
| **Decision** | Analytics v2 derives authoritative metrics from canonical transactional data using live aggregation. No materialized views, precomputed snapshots, or caching layer are introduced at this scale. Budget calculations reuse the existing Budget domain's computeBudgetUsage verbatim. |
| **Approved By** | TBD |
| **Approved On** | TBD |
| **Rationale** | Existing indexes confirmed sufficient via EXPLAIN ANALYZE; live aggregation avoids snapshot drift and infrastructure complexity; the Budget domain's formulas are already validated and reused; the dedicated `/api/v1/analytics` module preserves the correct boundaries between Dashboard (current-state overview, per PD-001) and Analytics (period-scoped trends and breakdowns, per the product name already live in navigation). |

---

# Terminology Reconciliation — Analytics vs Reports vs PD-006

The docs repo's glossary (`docs/product/glossary.md`) and PD-006 (`docs/product/decisions/006-reporting.md`, Draft, 2026-07-14) proposed "Report" / "Reports" as the canonical term for a dedicated reporting surface separate from the Dashboard — before any implementation of that surface existed.

The product's shipped implementation (route `/analytics`, navigation label, i18n namespace `"analytics"`, EN "Analytics" / ID "Analitik") uses "Analytics" as the operative term. This is the name users see, the route the navigation points to, and the name the engineering team used for this initiative ("Analytics v2").

**This ADR's position:** "Analytics" is the product's chosen operative term for this surface. It satisfies the intent of PD-006's scope (a dedicated reporting destination separate from the Dashboard, with period-aggregated Ledger-derived figures). PD-006 should be updated to reflect "Analytics" as the canonical term in the glossary and decision, and its status should be advanced to Approved (or superseded/replaced by this decision, which captures the same architectural conclusion). A dated History entry to that effect has been added to PD-006's History table — this ADR does **not** unilaterally mark PD-006 Approved itself, per this repo's convention of product-owner sign-off on status changes.

---

# Documentation Impact

- [x] `analytics-v2-calculation-spec.md` — metric definitions, inclusion rules, period/timezone policy, zero-baseline behavior
- [x] `analytics-v2-api-contract.md` — endpoint contracts, auth, error codes, response shapes
- [x] `implementation-roadmap.md` — Phase 11 (Analytics v2) added
- [ ] `docs/product/decisions/006-reporting.md` — History entry noting Analytics implementation; status left at Draft pending product-owner review
- [ ] `docs/product/glossary.md` — "Analytics" entry to be added when PD-006's terminology is finalized

---

# Implementation Impact

- **Frontend:** `pocket-mint-fe` branch `feature/analytics-v2` rewires the existing `/analytics` page from client-side `useAllTransactions()` computation to six dedicated `useAnalytics*` TanStack Query hooks calling the new `/api/v1/analytics` endpoints. Adds Recharts 3.10.0 as the single chart library dependency, with accessible data-table fallbacks for every visualization. Extends the existing i18n `"analytics"` namespace in both `en.json`/`id.json` with period-selector, overview, trends, categories, wallets, budget-performance, and drill-down translation keys. Reuses `BudgetStatusPill`/`BudgetProgressBar` from `app/(app)/anggaran/components/` for visual consistency.

- **Backend:** `pocket-mint-be` branch `feature/analytics-v2` adds a focused `analytics` module (`src/domain/analyticsPeriod.ts`, `src/services/analytics-*.service.ts`, `src/controllers/analytics.controller.ts`, `src/routes/analyticsRoutes.ts`, mounted at `/api/v1/analytics`). Six endpoints: overview (with previous-period comparison and zero-baseline percentage-change handling), trends (daily/monthly bucketing, zero-filled), categories (by type with explicit Uncategorized group), wallets (per-wallet income/expense/net), budget-performance (verbatim `budgetQueryService.listActiveBudgetUsage` reuse — no separate formula), and transactions (drill-down with pagination, reusing the canonical transaction shape). A test asserts numeric parity between `GET /budgets` and `GET /analytics/budget-performance` for the same data.

- **Database:** No migration. No new indexes. The existing `[userId, date]` and `[walletId, date]` indexes were confirmed sufficient by `EXPLAIN ANALYZE` against a 100k-row seeded dataset.

- **Automation:** None introduced.

- **Migration Required:** No.

- **Estimated Effort:** Implemented — backend 4 commits (period resolution, overview/trends, categories/wallets/budget/drill-down, integration tests), frontend in progress on same-named branch, docs in progress.

---

# Risks

- **Aggregation latency at very large scale:** if a single user exceeds ~500k transaction rows, per-request groupBy latency may become noticeable. The defined trigger is when real production queries consistently exceed 500ms p95 for the overview/categories endpoints. Mitigation: audit a targeted `[userId, type, date]` composite index first; only evaluate materialized-view or snapshot approaches if indexing alone can't bring latency below the threshold. There is no evidence this is a near-term concern.
- **Frontend chart-library dependency:** Recharts 3.10.0 was added as the single chart library. If Recharts proves incompatible with a future React or Next.js major version, the accessible data-table fallback ensures no information is lost during any migration period.
- **Zero-baseline consumer discipline:** the API contract's `{ value: null, reason: "ZERO_BASELINE" }` shape requires every consumer to handle `null` — a consumer that ignores `reason` and renders "Infinity%" is a consumer bug, not a spec ambiguity. The backend test suite covers this explicitly; frontend components (`AnalyticsSummaryCard`) render the translated "no comparison" state when `value === null`.

---

# Follow-up Tasks

1. Product owner to review and advance PD-006 status (Approved or superseded) and reconcile terminology (Analytics vs Reports) in the glossary.
2. When real user transaction volumes approach 500k rows per user, audit `EXPLAIN ANALYZE` for the aggregation queries against production data and evaluate `[userId, type, date]` composite index before considering materialized views.
3. Future roadmap phases (Smart Categorization, Rule Engine) that add derived/classified data should document how their new fields are consumed by the Analytics v2 aggregation queries — the `src/domain/analyticsPeriod.ts` + `src/services/analytics-*.service.ts` files are the integration points to extend, not replace.
4. Design review: the Analytics page's CSV export button maps new period presets to the prior export endpoint's `month | quarter | six-months` values — a future dedicated export endpoint supporting all Analytics v2 period presets would close this gap.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-22 | Draft | Initial creation — documents the implemented Analytics v2 architecture and terminology reconciliation. |
