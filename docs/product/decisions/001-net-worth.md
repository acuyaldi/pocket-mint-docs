---
id: PD-001
decision_number: 001
title: Canonical Net Worth Definition
version: 1.0
status: Approved
priority: P0
owner: Product
created: 2026-07-14
updated: 2026-07-14

related:
  rfc: docs/product/product-rfc.md#net-worth
  design: docs/product/design-system.md#net-worth-summary-widgets
  screen: docs/product/screen-spec.md
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-fe
  - pocket-mint-be

tags:
  - net-worth
  - financial-semantics
  - reporting

ai_context:
  required_before:
    - docs/product/product-rfc.md
    - docs/development/reconciliation-report.md#pm-fin-001--net-worth-definition
  update_after:
    - docs/product/product-rfc.md
    - docs/product/screen-spec.md
    - docs/product/component-spec.md
---

# Summary

Define net worth as total assets minus total debts. Show asset total separately when useful.

---

# Problem

The Product RFC uses the conventional definition, while the current product labels total assets as net worth. Users cannot reliably interpret the headline financial figure.

---

# Current Situation

## Current Implementation

Backend summaries, dashboard calculations, wallet summaries, and tests calculate net worth as asset total only. Debt is displayed separately.

## Current Documentation

The Product RFC defines net worth as assets minus debts. Repository-local frontend guidance describes the assets-only behavior.

## Current User Impact

Users may overestimate their financial position because the metric called net worth does not include debt.

---

# Options

## Option A — Use conventional net worth

Net worth equals assets minus debts, with assets and debts available as supporting figures.

Pros

- Matches common financial language.
- Makes the headline figure complete and comparable.

Cons

- Changes the value currently shown to users.
- Requires clear communication during the transition.

## Option B — Keep the assets-only metric and rename it

Retain the current calculation under a name such as Total Assets or Asset Position.

Pros

- Preserves the current calculation.
- Keeps asset visibility prominent.

Cons

- Leaves the product without a true net-worth metric.
- Weakens the stated goal of consolidated financial visibility.

## Option C — Keep assets-only under the net-worth label

Formalize the current calculation without renaming it.

Pros

- Avoids a visible value change.

Cons

- Uses a nonstandard and potentially misleading definition.
- Conflicts with the Product RFC.

---

# Recommendation

Adopt Option A. Net worth should equal assets minus debts, while Total Assets and Total Debt remain visible as supporting figures.

Evidence: Reconciliation finding PM-FIN-001 (P0) shows that the Product RFC defines `assets - debts`, while all active calculation paths currently use assets only and may mislead users.

Reasoning: Conventional terminology best supports financial clarity and makes the product's most prominent number understandable without special explanation.

---

# Final Decision

Status

Approved

Decision

Pocket Mint Net Worth equals Total Assets minus Total Outstanding Debt.

Total Assets is the combined balance of all asset wallets tracked by Pocket Mint.

Total Outstanding Debt is the combined outstanding amount of all debt wallets tracked by Pocket Mint.

Assets and debts must be evaluated at the same reporting cutoff. An installment obligation already represented in a debt wallet's outstanding balance must not be deducted again.

An assets-only value must be labeled Total Assets and must not be presented as Net Worth.

Approved By

Product

Approved On

2026-07-14

Rationale

This definition follows standard financial meaning, aligns with the Product RFC, and gives dashboards, wallets, reports, installments, and automation one explainable financial rule.

---

# Documentation Impact

- [x] product-rfc.md
- [x] design-system.md
- [x] screen-spec.md
- [x] component-spec.md

---

# Implementation Impact

Frontend

Headline and supporting labels must reflect the canonical definition.

Backend

Summary responses must provide the canonical net-worth figure and its components.

Database

No stored-data change is expected.

Automation

Any financial summary must use the same definition.

Migration Required

No data migration; a user-facing metric transition is required.

Estimated Effort

Medium

---

# Risks

Existing users may perceive the lower figure as a loss. Historical comparisons may be inconsistent unless the definition is clearly disclosed.

---

# Follow-up Tasks

- Approve user-facing names for Net Worth, Total Assets, and Total Debt.
- Define transition copy explaining the corrected metric.
- Align product, screen, and component documentation after approval.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-14 | Draft | Initial creation |
| 2026-07-14 | Approved | Approved canonical Net Worth definition and synchronized the Product RFC. |
