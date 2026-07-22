---
id: PD-006
decision_number: 006
title: Financial Reporting Surface
version: 1.0
status: Draft
priority: P1
owner: Product
created: 2026-07-14
updated: 2026-07-14

related:
  rfc: docs/product/product-rfc.md#product-overview
  design: docs/product/design-system.md#layout--spacing
  screen: docs/product/screen-spec.md
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-fe
  - pocket-mint-be

tags:
  - reporting
  - analytics
  - navigation

ai_context:
  required_before:
    - docs/product/product-rfc.md
    - docs/development/reconciliation-report.md#pm-scr-001--implemented-and-missing-screens
    - docs/development/reconciliation-report.md#pm-ui-003--explainable-financial-presentation
  update_after:
    - docs/product/product-rfc.md
    - docs/product/screen-spec.md
    - docs/product/component-spec.md
---

# Summary

Provide a dedicated Reports surface; keep the dashboard focused on current financial status and shortcuts.

---

# Problem

Financial reporting is a core capability, but the product currently offers only dashboard summaries. Some visible trends are placeholders rather than ledger-derived facts.

---

# Current Situation

## Current Implementation

The dashboard shows current summaries and recent activity. There is no reports route, and several trend or composition figures are synthetic or fixed.

## Current Documentation

The Product RFC names financial reporting as a core capability but does not define its scope. No authoritative screen inventory currently resolves the gap.

## Current User Impact

Users cannot explore or compare financial performance over time, and placeholder analytics may appear factual.

---

# Options

## Option A — Dedicated Reports surface

Add a primary reporting destination for ledger-derived period summaries, trends, debt, and installment reporting. Keep the dashboard as a current-state overview.

Pros

- Makes the core capability discoverable and expandable.
- Separates quick status from deeper analysis.

Cons

- Adds another primary destination.
- Requires clear reporting scope before delivery.

## Option B — Dashboard-only reporting

Treat dashboard widgets as the complete reporting experience.

Pros

- Keeps navigation compact.
- Reuses the current information architecture.

Cons

- Limits historical comparison and detail.
- Crowds the dashboard as reporting grows.

## Option C — Export-only reporting

Provide downloadable data and leave analysis to external tools.

Pros

- Supports advanced user-controlled analysis.

Cons

- Does not provide in-product financial clarity.
- Creates friction for routine questions.

---

# Recommendation

Adopt Option A. Reports should be a dedicated product surface backed only by ledger-derived data; dashboard widgets remain concise summaries that link into deeper reporting.

Evidence: Reconciliation finding PM-SCR-001 (P1) finds no dedicated reports route despite reporting being a core RFC capability. PM-UI-003 (P0) finds fabricated growth, trend, and principal/interest figures that undermine explainability.

Reasoning: A separate surface gives reporting enough room to be useful while preventing unsupported analytics from filling dashboard gaps.

---

# Final Decision

Status

Draft

Decision

Financial reporting is a dedicated product surface; the dashboard remains a current-state overview.

Approved By

Pending

Approved On

Pending

Rationale

This makes a core capability explicit and keeps every displayed figure explainable.

---

# Documentation Impact

- [x] product-rfc.md
- [x] design-system.md
- [x] screen-spec.md
- [x] component-spec.md

---

# Implementation Impact

Frontend

Navigation, dashboard links, reporting states, and historical views require a defined product contract.

Backend

Reports require canonical period aggregates and provenance for derived values.

Database

Historical reporting needs review against available ledger dates and installment events.

Automation

Exports and scheduled summaries should use the same report definitions.

Migration Required

No user-data migration is expected.

Estimated Effort

Large

---

# Risks

An oversized first release could delay useful reporting. Incomplete historical data may create gaps that need explicit explanation.

---

# Follow-up Tasks

- Prioritize the first report questions and date ranges.
- Define empty, partial-data, and unavailable-history states.
- Approve the Reports route and navigation role in the screen specification.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-14 | Draft | Initial creation |
| 2026-07-22 | Draft | Analytics v2 implemented in `pocket-mint-be`/`pocket-mint-fe` (branches `feature/analytics-v2`, PRs open against `dev`). The product's operative term is "Analytics" (route `/analytics`, i18n namespace `"analytics"`, EN "Analytics" / ID "Analitik"), satisfying PD-006's intent of a dedicated reporting destination separate from the Dashboard. See [ADR-010](./010-analytics-v2-architecture.md) for the terminology reconciliation and architecture decision. Status left at Draft pending product-owner review of the Analytics vs Reports terminology. |
