---
id: PD-002
decision_number: 002
title: Owner-First Access Policy
version: 1.0
status: Approved
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

Before approval, the Product RFC stated no public registration, single owner by default, schema support for multiple Users, and authentication for every request. This decision resolves the public exceptions and access boundaries that wording left ambiguous.

## Current User Impact

Self-hosters cannot tell whether additional people may create accounts, and public auth flows appear inconsistent with the privacy-first promise.

---

# Options

## Option A — Closed owner-controlled access

Allow initial owner setup, then permit additional accounts only through an owner-approved invitation or allowlist. Keep login, callback, recovery, and health routes public.

Pros

- Matches the privacy-first, owner-first proposition.
- Supports deliberate multi-user use without public registration.

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

Approved

Decision

Pocket Mint adopts First Owner with Controlled Access.

An uninitialized installation permits one narrowly scoped initialization journey. The first account to complete that journey becomes the installation's single active Owner. Owner creation must be atomic so concurrent attempts cannot create multiple Owners.

After initialization, public self-registration is prohibited. Additional accounts require explicit Owner approval through an invitation or allowlist. Owner approval authorizes admission only; it does not authenticate the person, create financial ownership for them, or grant access before identity verification is complete.

Invitation is the preferred admission experience because it can be identity-bound, expiring, revocable, and single-use. An Owner-managed allowlist is an acceptable self-hosting alternative when it identifies the permitted account precisely and follows the same verification, revocation, and privacy rules. A pending invitation or allowlist entry grants no product access.

Pocket Mint supports additional Users as a current controlled-access capability. This does not make Pocket Mint a collaborative finance product. Each User owns isolated financial resources, and no User may see or affect another User's resources. Shared wallets, household financial summaries, delegated financial access, and collaborative ownership are outside this decision.

The Owner is also a User, but Owner authority is limited to installation-level responsibilities such as admission and installation settings. Owner authority must never bypass User resource isolation, impersonate another User, or confer access to another User's financial data.

Only non-financial landing content, first-Owner initialization while uninitialized, authentication, approved authentication callbacks, invitation or allowlist admission completion, account recovery, and minimal health capabilities may be reachable without an authenticated session. These public capabilities must reveal no private financial information and must not disclose whether an account or invitation exists beyond what is necessary for a valid flow.

Every user-financial capability requires an authenticated identity. The Backend must enforce User scope and resource authorization for every financial query and mutation regardless of whether the caller is the Frontend, Automation, an integration, or a future client. Frontend route protection improves privacy and navigation but is not an authorization boundary.

Every User may recover their own account through an approved recovery path that does not disclose account existence. Owner recovery restores the existing Owner identity when possible. If the Owner is permanently unavailable, replacement requires an explicit installation-controlled recovery procedure with proof of installation control and an auditable transfer of installation authority. No User is promoted automatically. Owner replacement does not transfer, expose, or merge the former Owner's financial resources.

Automation and service identities require explicit authorization and narrowly defined User scope. They use the same Backend and Business Services boundaries as interactive clients and receive no financial authority from being approved by the Owner. The Owner cannot authorize Automation to access another User's resources on that User's behalf.

Existing installations must transition away from public signup without treating the next post-transition registrant as Owner. New registration must close, one legitimate account must be deliberately established as Owner, and existing accounts must be reviewed under the controlled-admission policy. Existing financial resources must remain attached to their current User identities and must not be silently transferred, merged, exposed, or deleted during transition.

Approved By

Product

Approved On

2026-07-14

Rationale

A closed, Owner-controlled installation is consistent with Pocket Mint's privacy-first and self-hosting goals while allowing deliberate household deployment. Separating installation authority from User resource ownership prevents the Owner role from becoming a financial superuser. Explicit public exceptions make authentication, recovery, health checks, and safe initialization possible without weakening the protected financial boundary.

---

# Documentation Impact

- [x] product-rfc.md
- [ ] design-system.md
- [x] screen-spec.md
- [ ] component-spec.md
- [x] glossary.md
- [x] system-architecture.md
- [x] project-context.md

---

# Implementation Impact

Frontend

Onboarding, login, and protected-screen behavior must reflect closed access.

Backend

Account creation must enforce owner approval and preserve authenticated ownership for financial APIs.

Database

Owner and invitation state may need explicit representation.

Automation

Automated access must remain explicitly authorized and User-scoped. Owner approval cannot bypass another User's authorization.

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
| 2026-07-14 | Approved | Adopted First Owner with Controlled Access and explicit public-access exceptions. |
