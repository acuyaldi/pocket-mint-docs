---
id: PD-007
decision_number: 007
title: Canonical Transfer Representation
version: 1.0
status: Draft
priority: P2
owner: Product
created: 2026-07-14
updated: 2026-07-14

related:
  rfc: docs/product/product-rfc.md#domain-model
  design: docs/product/design-system.md
  screen: docs/product/screen-spec.md
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-be

tags:
  - transfer
  - transaction
  - financial-model

ai_context:
  required_before:
    - docs/product/product-rfc.md
    - docs/development/reconciliation-report.md#pm-trf-001--transfer-atomicity
    - docs/development/reconciliation-report.md#pm-trf-002--separate-transfer-model
  update_after:
    - docs/product/product-rfc.md
---

# Summary

Represent a transfer as one transaction with a source wallet and destination wallet.

---

# Problem

Documentation and the data model imply a separate Transfer entity, while the active product uses a Transaction with a destination wallet. Two representations create ambiguity.

---

# Current Situation

## Current Implementation

Transfers use one Transaction row, update both wallet balances atomically, and reverse both effects on change or deletion. The separate Transfer model is unused.

## Current Documentation

The Product RFC lists Transfer as a primary entity and also states that the Prisma schema is the implementation source of truth.

## Current User Impact

Current transfers behave consistently, but future reporting or migration work could treat the unused model as authoritative and create duplicate meanings.

---

# Options

## Option A — Transaction with destination wallet

Use one transaction record to represent the source, destination, amount, and transfer identity.

Pros

- Matches the proven atomic behavior.
- Keeps transfers in the same ledger and reporting model as other movements.

Cons

- Requires clear transfer-specific rules within the transaction model.

## Option B — Separate Transfer entity

Move active transfer behavior to a dedicated transfer record.

Pros

- Gives transfers an explicit domain object.
- Could hold future transfer-only metadata.

Cons

- Replaces working behavior without a current product need.
- Risks duplicate or split ledger records.

## Option C — Keep both representations

Use Transaction for ledger effects and Transfer for metadata.

Pros

- Allows specialized transfer metadata.

Cons

- Creates synchronization and reconciliation risk.
- Adds complexity without an approved use case.

---

# Recommendation

Adopt Option A. A transfer is one Transaction with a destination wallet; the separate Transfer concept should not remain part of the product contract.

Evidence: Reconciliation finding PM-TRF-001 confirms the one-row Transaction path is atomic and tested. PM-TRF-002 (P2) confirms that the separate Transfer model has no active reader or writer and makes the domain ambiguous.

Reasoning: The existing representation already meets the user need with fewer competing concepts.

---

# Final Decision

Status

Draft

Decision

Transaction with a destination wallet is the sole product representation of a transfer.

Approved By

Pending

Approved On

Pending

Rationale

It preserves proven atomic behavior and removes an unused competing model.

---

# Documentation Impact

- [x] product-rfc.md
- [ ] design-system.md
- [ ] screen-spec.md
- [ ] component-spec.md

---

# Implementation Impact

Frontend

No product behavior change is expected.

Backend

Transfer contracts and terminology must consistently use the transaction representation.

Database

The unused transfer representation requires a deliberate retirement path.

Automation

Integrations must create and read transfers through the transaction contract.

Migration Required

Schema migration is required only to retire the unused representation.

Estimated Effort

Small

---

# Risks

Unknown external consumers may rely on the unused representation. Future transfer-specific needs could require extending the transaction contract.

---

# Follow-up Tasks

- Confirm that no planned product capability requires a separate transfer identity.
- Align transfer terminology in product and API documentation.
- Define the product expectation for any legacy transfer data before retirement.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-14 | Draft | Initial creation |
