---
id: PD-003
decision_number: 003
title: Installment Lifecycle Policy
version: 1.0
status: Draft
priority: P0
owner: Product
created: 2026-07-14
updated: 2026-07-14

related:
  rfc: docs/product/product-rfc.md#installment-model
  design: docs/product/design-system.md#installment-progress-bars
  screen: docs/product/screen-spec.md
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-fe
  - pocket-mint-be

tags:
  - installments
  - lifecycle
  - reporting

ai_context:
  required_before:
    - docs/product/product-rfc.md
    - docs/development/reconciliation-report.md#pm-ins-001--installment-model-a
    - docs/development/reconciliation-report.md#pm-ins-002--installment-scheduler-and-lifecycle
  update_after:
    - docs/product/product-rfc.md
    - docs/product/screen-spec.md
    - docs/product/component-spec.md
---

# Summary

Use an automatic monthly lifecycle with explicit user controls for payoff and cancellation.

---

# Problem

Installments are created with debt locked up front, but terms never advance and later months receive no reporting entries. Progress and status therefore become unreliable.

---

# Current Situation

## Current Implementation

Creation deducts the full obligation once and starts an active installment at term one. No scheduled progression or lifecycle command updates terms, settlement, or monthly reporting.

## Current Documentation

The Product RFC selects front-loaded balance deduction but does not define monthly progression, catch-up, payoff, cancellation, or reporting behavior.

## Current User Impact

Installments can remain permanently active at their first term. Due dates, progress, settlement, and monthly reports may be wrong.

---

# Options

## Option A — Automatic monthly lifecycle with manual controls

Advance each due term automatically in the owner's reporting timezone. Create one reporting entry per due term without changing the wallet balance again. Catch up missed terms once, make processing repeat-safe, settle after the final term, and allow explicit payoff or cancellation.

Pros

- Keeps progress and monthly reporting current.
- Preserves user control for exceptional cases.
- Recovers after self-hosted downtime.

Cons

- Requires clear rules for catch-up and manual actions.
- Background processing adds operational responsibility.

## Option B — Manual term confirmation

Advance a term only when the user marks it paid.

Pros

- Maximizes direct user control.
- Avoids reliance on background processing.

Cons

- Creates recurring bookkeeping work.
- Reports become stale when users forget.

## Option C — Calendar projection only

Derive progress from dates without recording monthly lifecycle events.

Pros

- Simple to explain and operate.

Cons

- Cannot distinguish projected from completed terms.
- Does not support auditable reporting, payoff, or cancellation.

---

# Recommendation

Adopt Option A. Each due month should create one reporting event, advance progress once, and never deduct the wallet balance again. Missed months should be caught up after downtime, and payoff or cancellation should require explicit user action with a recorded outcome.

Evidence: Reconciliation findings PM-INS-001 and PM-INS-002 (both P0) show that front-loaded deduction works, but no later reporting rows, progression, settlement, scheduler, or lifecycle commands exist. PM-INS-004 also shows that visible payoff and cancellation controls have no defined behavior.

Reasoning: Automatic progression delivers accurate low-effort reporting, while explicit manual controls preserve the product principle that automation never replaces user control.

---

# Final Decision

Status

Draft

Decision

Installments progress automatically by due month, catch up missed terms exactly once, and support explicit payoff and cancellation outcomes.

Approved By

Pending

Approved On

Pending

Rationale

This completes the front-loaded model without duplicating balance deductions and keeps reports trustworthy.

---

# Documentation Impact

- [x] product-rfc.md
- [x] design-system.md
- [x] screen-spec.md
- [x] component-spec.md

---

# Implementation Impact

Frontend

Progress, due state, payoff, cancellation, and exception states need defined behavior.

Backend

Lifecycle processing must own repeat-safe progression, catch-up, settlement, and manual outcomes.

Database

Lifecycle events must be identifiable so a due term cannot be recorded twice.

Automation

Monthly processing runs in the owner's reporting timezone and catches up after downtime.

Migration Required

Existing active installments require reconciliation to their correct current term.

Estimated Effort

Large

---

# Risks

Incorrect catch-up may duplicate reports. Timezone changes may shift due dates. Cancellation and payoff may be misunderstood unless their financial effects are explicit.

---

# Follow-up Tasks

- Define product copy for upcoming, due, overdue, settled, paid off, and cancelled states.
- Define the owner reporting-timezone experience.
- Define payoff and cancellation confirmation rules and audit expectations.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-14 | Draft | Initial creation |
