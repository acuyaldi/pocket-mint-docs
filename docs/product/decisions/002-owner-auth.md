---
id: PD-002
decision_number: 002
title: Owner-First Access Policy
version: 1.0
status: Draft
priority: P0
owner: Product
created: 2026-07-14
updated: 2026-07-14

related:
  rfc: docs/product/product-rfc.md#authentication
  design: docs/product/design-system.md
  screen: docs/product/screen-spec.md
  component: docs/product/component-spec.md

affected_repositories:
  - pocket-mint-docs
  - pocket-mint-fe
  - pocket-mint-be

tags:
  - authentication
  - ownership
  - privacy

ai_context:
  required_before:
    - docs/product/product-rfc.md
    - docs/development/reconciliation-report.md#pm-auth-001--owner-first-authentication
  update_after:
    - docs/product/product-rfc.md
    - docs/product/screen-spec.md
---

# Summary

Make Pocket Mint a closed, owner-controlled product with no public account registration.

---

# Problem

The product promises owner-first access, but the current sign-in experience exposes public signup paths. The intended audience and access boundary are unclear.

---

# Current Situation

## Current Implementation

Financial APIs require verified identities, but the frontend offers email signup and Google OAuth. Several authenticated screens are not consistently redirected before loading.

## Current Documentation

The Product RFC states no public registration, single owner by default, schema support for multiple users, and authentication for every request.

## Current User Impact

Self-hosters cannot tell whether additional people may create accounts, and public auth flows appear inconsistent with the privacy-first promise.

---

# Options

## Option A — Closed owner-controlled access

Allow initial owner setup, then permit additional accounts only through an owner-approved invitation or allowlist. Keep login, callback, recovery, and health routes public.

Pros

- Matches the privacy-first, owner-first proposition.
- Supports optional future multi-user use without public registration.

Cons

- Requires an explicit onboarding and invitation policy.
- Adds friction for multi-user households.

## Option B — Single-owner only

Allow exactly one account per installation and expose no invitation path.

Pros

- Simplest policy for self-hosting.
- Minimizes account-management complexity.

Cons

- Makes existing multi-user schema support unusable.
- Limits future household use.

## Option C — Open registration

Allow anyone who can reach the deployment to register.

Pros

- Lowest onboarding friction.
- Matches the currently exposed signup experience.

Cons

- Conflicts with owner-first positioning.
- Increases privacy and administration risk for self-hosted deployments.

---

# Recommendation

Adopt Option A. Pocket Mint should be closed by default: the first owner initializes the installation, and every later account requires owner approval.

Evidence: Reconciliation finding PM-AUTH-001 (P0) identifies a direct conflict between the RFC's no-public-registration rule and active email and OAuth signup paths; it also confirms that public login, callback, reset, and health routes are necessary exceptions.

Reasoning: Closed owner control preserves the product promise while retaining deliberate multi-user growth.

---

# Final Decision

Status

Draft

Decision

Use first-owner initialization followed by owner-approved access. Public self-registration is not permitted.

Approved By

Pending

Approved On

Pending

Rationale

This policy balances privacy-first ownership with optional multi-user support.

---

# Documentation Impact

- [x] product-rfc.md
- [ ] design-system.md
- [x] screen-spec.md
- [x] component-spec.md

---

# Implementation Impact

Frontend

Onboarding, login, and protected-screen behavior must reflect closed access.

Backend

Account creation must enforce owner approval and preserve authenticated ownership for financial APIs.

Database

Owner and invitation state may need explicit representation.

Automation

Automated access must remain owner-authorized.

Migration Required

Existing non-owner accounts require a product migration policy.

Estimated Effort

Medium

---

# Risks

An unclear first-owner recovery process could lock users out. Existing deployments may already contain multiple valid accounts.

---

# Follow-up Tasks

- Define first-owner setup and recovery journeys.
- Define owner-approved invitation and account-removal policies.
- List the public and authenticated product routes in the screen specification.

---

# History

| Date | Status | Notes |
|------|--------|------|
| 2026-07-14 | Draft | Initial creation |
