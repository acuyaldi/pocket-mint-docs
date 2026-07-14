---
id: PD-005
decision_number: 005
title: Debt Utilization Policy
version: 1.0
status: Draft
priority: P0
owner: Product
created: 2026-07-14
updated: 2026-07-14

related:
  rfc: docs/product/product-rfc.md#available-credit
  design: docs/product/design-system.md#wallet-cards
  screen: docs/product/screen-spec.md
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-fe
  - pocket-mint-be

tags:
  - debt
  - credit-limit
  - utilization

ai_context:
  required_before:
    - docs/product/product-rfc.md
    - docs/product/design-system.md
    - docs/development/reconciliation-report.md#pm-wal-002--debt-calculations-and-credit-limit-enforcement
  update_after:
    - docs/product/product-rfc.md
    - docs/product/design-system.md
    - docs/product/component-spec.md
---

# Summary

Apply one debt-utilization policy to Credit Card and PayLater wallets, with a hard credit limit and consistent risk states.

---

# Problem

Debt calculations exist, but the product can accept obligations beyond the configured limit and presents inconsistent risk colors for the same utilization.

---

# Current Situation

## Current Implementation

Outstanding, available credit, and utilization are calculated. The frontend may reject over-limit entries, but the backend does not. Warning and danger thresholds differ across screens.

## Current Documentation

The Product RFC defines debt formulas and backend authority. The design system gives debt and warning colors but no utilization thresholds.

## Current User Impact

Users can receive conflicting risk signals, and entries made outside the main interface may exceed the stated credit limit.

---

# Options

## Option A — Hard limit with three utilization states

Reject obligations above 100%. Show normal below 30%, warning from 30% through 79%, and danger from 80% upward. Use debt red for both Credit Card and PayLater debt; reserve amber for upcoming installment timing.

Pros

- Creates one predictable policy across all entry points and screens.
- Separates debt severity from payment timing.

Cons

- Blocks users from recording a real over-limit balance without first raising the limit.
- Thresholds are product guidance rather than personalized advice.

## Option B — Soft limit with warnings

Allow entries above the limit but warn the user.

Pros

- Can record real provider over-limit states.
- Avoids blocking data entry.

Cons

- Weakens the configured limit as a control.
- Makes automation and API behavior harder to predict.

## Option C — Display utilization without risk states

Show the percentage and remaining credit without thresholds or semantic colors.

Pros

- Avoids prescriptive financial guidance.

Cons

- Reduces at-a-glance clarity.
- Does not resolve over-limit behavior.

---

# Recommendation

Adopt Option A. Use the configured credit limit as a hard boundary and one utilization scale: normal below 30%, warning at 30–79%, and danger at 80% or above. Treat PayLater as debt red; use amber only for installment timing states.

Evidence: Reconciliation finding PM-WAL-002 (P0) shows missing authoritative credit-limit enforcement and conflicting 30%, 50%, and 80% display thresholds. PM-UI-001 shows that PayLater uses amber despite the design system's debt-red rule.

Reasoning: One policy gives users consistent, explainable feedback and makes the configured limit meaningful everywhere.

---

# Final Decision

Status

Draft

Decision

Debt obligations cannot exceed the configured limit; utilization uses the 30% warning and 80% danger thresholds across all debt wallets.

Approved By

Pending

Approved On

Pending

Rationale

Consistent enforcement and presentation protect financial clarity.

---

# Documentation Impact

- [x] product-rfc.md
- [x] design-system.md
- [x] screen-spec.md
- [x] component-spec.md

---

# Implementation Impact

Frontend

All debt views and entry points must share the same states and over-limit message.

Backend

Every debt-changing operation must apply the canonical limit policy.

Database

No schema change is expected; existing limits require data review.

Automation

Automated entries must obey the same limit and return a clear rejection.

Migration Required

Existing over-limit wallets require a user-resolution policy.

Estimated Effort

Medium

---

# Risks

A hard limit may prevent users from entering accurate provider balances. Static thresholds may be mistaken for personalized financial advice.

---

# Follow-up Tasks

- Define the user journey for an existing over-limit wallet.
- Approve utilization labels and non-advisory explanatory copy.
- Align debt and installment color meanings in product documentation.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-14 | Draft | Initial creation |
