---
id: PD-008
decision_number: 008
title: Canonical Design Language
version: 1.0
status: Draft
priority: P1
owner: Product
created: 2026-07-14
updated: 2026-07-14

related:
  rfc: docs/product/product-rfc.md#user-experience-principles
  design: docs/product/design-system.md
  screen: docs/product/screen-spec.md
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-fe

tags:
  - design-language
  - design-tokens
  - accessibility

ai_context:
  required_before:
    - docs/product/design-system.md
    - docs/development/reconciliation-report.md#pm-ds-002--design-token-adoption
    - docs/development/reconciliation-report.md#pm-ds-003--design-system-internal-consistency
  update_after:
    - docs/product/design-system.md
    - docs/product/screen-spec.md
    - docs/product/component-spec.md
---

# Summary

Use one light, token-led design language with mint as the primary action color, amber for warnings, and tabular Inter for financial figures.

---

# Problem

The design documentation gives conflicting answers for colors, typography, radii, and primary controls. Product teams cannot apply a single visual contract consistently.

---

# Current Situation

## Current Implementation

The product ships a light palette with mint primary actions and mostly Inter typography. Token adoption is incomplete, raw colors remain, and financial-number styling varies.

## Current Documentation

The design system conflicts on background color, amber semantics, primary-button color, font families, and radius aliases.

## Current User Impact

Inconsistent visual signals reduce trust and make similar financial states look unrelated across screens.

---

# Options

## Option A — Reconcile around the shipped light system

Keep light-only presentation, mint primary actions, `#f8f9ff` background, amber warning semantics, Inter typography with tabular figures for financial values, and the documented 4/8/12/16/24px radius scale.

Pros

- Preserves the recognizable current product.
- Resolves semantic conflicts with limited user disruption.
- Keeps financial values aligned without introducing another font.

Cons

- Requires documentation and token cleanup.
- Defers dark mode.

## Option B — Rebuild around the prose design direction

Use slate primary buttons, `#f8fafc` background, and separate display and mono fonts.

Pros

- Follows the narrative design guidance closely.
- Creates stronger typographic contrast.

Cons

- Changes the shipped brand expression.
- Adds font and token complexity.

## Option C — Preserve mixed conventions

Allow each component to select from documented or implemented variants.

Pros

- Requires little immediate change.

Cons

- Keeps the design system non-authoritative.
- Continues visual drift and inconsistent financial semantics.

---

# Recommendation

Adopt Option A. The canonical language is light-only, uses mint for primary actions and positive growth, amber for warnings and upcoming states, coral red for debt and destructive states, Inter throughout, and tabular figures for every financial value.

Evidence: Reconciliation finding PM-DS-001 confirms a coherent light baseline. PM-DS-002 (P1) finds incomplete token and financial-number adoption, while PM-DS-003 (P1) identifies direct conflicts in background, color semantics, buttons, fonts, and radii.

Reasoning: Reconciliation around the shipped system restores one source of truth without an unnecessary visual reset.

---

# Final Decision

Status

Draft

Decision

Pocket Mint uses the reconciled light, mint-led, Inter-based token system defined in Option A.

Approved By

Pending

Approved On

Pending

Rationale

This direction is coherent, accessible to maintain, and closest to the current product experience.

---

# Documentation Impact

- [ ] product-rfc.md
- [x] design-system.md
- [x] screen-spec.md
- [x] component-spec.md

---

# Implementation Impact

Frontend

Components must use the canonical semantic tokens and tabular financial-number style.

Backend

No impact.

Database

No impact.

Automation

Generated product interfaces must use the same design contract.

Migration Required

No data migration; visual normalization is required.

Estimated Effort

Medium

---

# Risks

Token cleanup may expose contrast or hierarchy issues. Deferring dark mode may disappoint users who expect theme choice.

---

# Follow-up Tasks

- Approve the canonical semantic color glossary.
- Define financial-number, radius, and component-state usage in product documentation.
- Record dark mode as a separate future product decision only if demand is validated.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-14 | Draft | Initial creation |
