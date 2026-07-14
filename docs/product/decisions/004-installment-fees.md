---
id: PD-004
decision_number: 004
title: Installment Fee Treatment
version: 1.0
status: Draft
priority: P1
owner: Product
created: 2026-07-14
updated: 2026-07-14

related:
  rfc: docs/product/product-rfc.md#installment-calculator
  design: docs/product/design-system.md
  screen: docs/product/screen-spec.md
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-fe
  - pocket-mint-be

tags:
  - installments
  - fees
  - calculation

ai_context:
  required_before:
    - docs/product/product-rfc.md
    - docs/development/reconciliation-report.md#pm-ins-003--forward-and-reverse-installment-calculator
  update_after:
    - docs/product/product-rfc.md
    - docs/product/screen-spec.md
    - docs/product/component-spec.md
---

# Summary

Include a declared administrative fee in the total installment obligation and support either a flat amount or a percentage.

---

# Problem

The current preview shows an administrative-fee estimate that is not part of the saved installment contract. Users can see one total and incur another.

---

# Current Situation

## Current Implementation

The forward preview estimates an administrative fee, but submitted installment data does not preserve that fee as part of the obligation.

## Current Documentation

The Product RFC defines calculator inputs and front-loaded deduction but does not define fee types, fee basis, or whether fees belong in the total.

## Current User Impact

Users cannot verify whether the shown fee affects debt, monthly payment, or reports.

---

# Options

## Option A — Include flat or percentage fees in the obligation

Calculate a declared fee once at creation, show its basis, and include it in the total obligation and monthly payment.

Pros

- Matches real installment products.
- Makes the preview, saved obligation, and reports reconcilable.

Cons

- Adds fields and explanation to installment entry.
- Percentage fees require an explicit calculation basis.

## Option B — Support flat fees only

Allow one currency amount and include it in the total obligation.

Pros

- Simple to understand and calculate.
- Covers common administrative charges.

Cons

- Cannot accurately represent percentage-based provider fees.

## Option C — Exclude fees

Remove fee entry and fee estimates from the product.

Pros

- Keeps installment calculations simple.

Cons

- Understates obligations that include real fees.
- Forces users to record fees elsewhere.

---

# Recommendation

Adopt Option A. A fee may be a flat amount or a percentage of principal, is calculated once when the installment is created, and is included in the total obligation before the monthly payment is derived.

Evidence: Reconciliation finding PM-INS-003 (P1) shows that the frontend displays an administrative-fee estimate that is excluded from the submitted contract; the report explicitly requires a decision on fee inclusion and end-to-end FLAT and PERCENT support.

Reasoning: Persisting one transparent fee rule prevents the preview, debt balance, payment schedule, and reports from disagreeing.

---

# Final Decision

Status

Draft

Decision

Include declared flat or principal-based percentage fees in the installment's total obligation.

Approved By

Pending

Approved On

Pending

Rationale

The saved obligation must match what the user reviews before creation.

---

# Documentation Impact

- [x] product-rfc.md
- [ ] design-system.md
- [x] screen-spec.md
- [x] component-spec.md

---

# Implementation Impact

Frontend

Entry and review states must show fee type, basis, amount, and resulting total.

Backend

The canonical calculation must include and return the declared fee.

Database

Fee type, input, and calculated amount must remain auditable.

Automation

Imported installments must use the same fee policy.

Migration Required

Existing installments without fee data remain fee-free unless the user explicitly revises them.

Estimated Effort

Medium

---

# Risks

Users may confuse provider interest with administrative fees. A percentage basis that is not clearly shown may produce unexpected totals.

---

# Follow-up Tasks

- Approve labels and explanations for flat and percentage fees.
- Define how edited installments present recalculated totals.
- Add fee details to installment screen and component specifications.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-14 | Draft | Initial creation |
