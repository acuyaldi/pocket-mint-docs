# Pocket Mint AI Coding Rules

These rules supplement [Pocket Mint AI Project Context](./project-context.md). Read that document first for product scope, authority, provisional areas, architectural boundaries, and task classification. This rulebook governs engineering behavior; it does not redefine product policy or coding style.

---

## General Principles

- Work on one clearly scoped task at a time.
- Change only the repository and files required by the task.
- Preserve existing behavior unless the task explicitly and authoritatively changes it.
- Make the smallest coherent change that satisfies the approved requirement.
- Do not refactor, rename, reformat, migrate, or clean up unrelated code.
- Preserve unrelated working-tree changes. Never discard or overwrite work without explicit authorization.
- Prefer small, reviewable commits with one purpose.
- Stop when product intent, contract ownership, or required behavior is unresolved.
- Treat passing tests as evidence, not as permission to violate documentation.
- Treat existing implementation as evidence, not automatic product authority.

---

## Documentation First

Resolve what the system should do before changing how it does it.

```text
Vision
  ↓
Product RFC
  ↓
Approved Product Decisions
  ↓
Canonical Glossary
  ↓
System Architecture
  ↓
Implementation
```

- Start with [project-context.md](./project-context.md), then read the minimum documents required for the task type.
- Apply the complete authority order defined in project context when lower-level specifications or repository guidance are relevant.
- Use each individual file in [`docs/product/decisions/`](../product/decisions/) as the source for that Product Decision's current status.
- Do not treat a recommendation, aggregate summary, reconciliation finding, checked impact box, or implemented behavior as approval.
- If an aggregate document disagrees with individual Product Decision front matter, report the conflict and follow the individual decision status. Do not silently synchronize documents.
- Implement an Approved Product Decision only within its decided scope.
- Treat every Draft Product Decision as provisional. Stop and request approval when its unresolved choice affects the task.
- Report documentation-versus-implementation conflicts before making dependent changes.
- Do not edit approved product documentation merely to legitimize buggy or accidental implementation behavior.

---

## Repository Rules

### `pocket-mint-docs`

- Keep product policy, terminology, architecture, design, screen behavior, and implementation evidence in their established document layers.
- Create or change product policy through a Product Decision before updating dependent documents.
- Use canonical terms from the Glossary; do not introduce synonyms or new business concepts casually.
- Update only documents affected by the approved change and preserve authority relationships between them.
- Keep reconciliation reports as historical audit records. Do not rewrite them to hide resolved findings.
- Verify relative links, headings, status references, and contradictions after every documentation change.

### `pocket-mint-fe`

- Read repository-local instructions before editing and reconcile them with higher-authority documentation.
- Consume backend contracts and canonical results; do not create a competing source of financial truth.
- Reuse existing routes, components, hooks, types, formatters, and design tokens before adding new ones.
- Keep authoritative business validation and lifecycle decisions out of UI components.
- Keep loading, empty, partial, success, and error states distinct.
- Preserve Reporting Cutoff, provenance, and state labels supplied by the Backend.
- Never access the Database directly.

### `pocket-mint-be`

- Read repository-local instructions, schema history, service boundaries, and relevant tests before editing.
- Keep API boundary handling, Business Services, and persistence responsibilities separate.
- Reuse existing services, domain functions, validators, types, and error contracts before adding new ones.
- Route financial commands and queries through the established service layer.
- Make schema changes only through a deliberate migration with compatibility and data impact reviewed.
- Keep Automation and future clients on the same authenticated Backend and Business Service paths as the Frontend.
- Preserve deterministic, User-scoped, transactional, and explainable financial behavior.

---

## Before Writing Code

Complete this checklist before modifying implementation:

- [ ] Classify the task as Product, Documentation, Frontend, Backend, Architecture, or Automation work.
- [ ] State the exact requested outcome and the repository in scope.
- [ ] Read [project-context.md](./project-context.md) and the minimum authoritative documents for the task type.
- [ ] Open every relevant Product Decision and confirm its individual front-matter status.
- [ ] Identify any Draft decision, policy conflict, contract ambiguity, or unresolved reconciliation finding.
- [ ] Inspect repository-local instructions and the current working-tree status.
- [ ] Search for existing functions, services, routes, types, components, validators, formatters, tokens, and tests.
- [ ] Trace the current call path and identify which layer owns the behavior.
- [ ] Identify existing tests that protect the behavior and record any baseline failures.
- [ ] Produce a short plan naming files, behavior, verification, migrations, and policy dependencies.
- [ ] Stop for clarification if the task requires new product policy, a new business concept, or a choice governed by a Draft decision.

---

## Backend Rules

- Put business rules and Canonical Calculations in Business Services or established backend domain modules.
- Keep controllers and API handlers focused on boundary concerns; do not duplicate service logic there.
- Validate authentication, User ownership, resource relationships, input constraints, and current state on every command path.
- Never trust frontend validation, hidden controls, route guards, identifiers, calculated values, or claimed ownership.
- Parse and preserve authoritative money values with exact decimal semantics from the request boundary onward.
- Never pass authoritative money through binary floating point before validation, calculation, or persistence.
- Use one database transaction for every operation whose records or balance effects must succeed together.
- Make retryable or externally triggered consequential operations idempotent.
- Never bypass the service layer with an alternate controller, automation, script, or integration path.
- Return canonical outcomes and deterministic business errors; do not require clients to reconstruct financial effects.
- Preserve audit context and provenance for consequential, corrected, imported, or automated operations.
- Add a deliberate migration for every schema change; do not edit schema state without a migration and data-impact review.
- Add or update tests for calculations, ownership, authorization, rollback, idempotency, and contract changes affected by the task.

---

## Frontend Rules

- Treat backend-confirmed data as financial truth.
- Never calculate Net Worth, Total Assets, Total Outstanding Debt, balances, debt metrics, installment obligations, or period aggregates independently in UI components.
- Never hardcode business thresholds, lifecycle rules, financial formulas, fabricated history, or placeholder financial facts.
- Keep components presentational where possible; isolate data access and orchestration in established application boundaries.
- Reuse shared API clients, types, components, formatters, and tokens instead of creating local variants.
- Use local validation only to improve usability; always handle authoritative backend rejection.
- Do not mark a mutation successful until the Backend confirms it.
- Display Reporting Cutoff, provenance, uncertainty, and state distinctions when required by the returned contract.
- Preserve the difference between unavailable data and zero, and between Scheduled, Projected, Imported, Confirmed, and Paid.
- Never expose another User's data or resource existence through content, routing, caching, or error messages.
- Do not invent an endpoint, field, enum, error shape, or state transition. Confirm the backend contract first.
- Update affected interaction, loading, empty, success, error, and accessibility tests.

---

## Documentation Rules

- Use a Product Decision to propose any new or changed product behavior, financial rule, business concept, or canonical term.
- Do not represent a Draft Product Decision as approved behavior.
- After a Product Decision is Approved, update the Product RFC and Glossary when its behavior or terminology changes them.
- Review Architecture, Design System, Screen Specification, Component Specification, repository guidance, and implementation for dependent updates.
- Do not mark documentation-impact checkboxes complete until the referenced document has actually been reconciled.
- Use canonical terminology and link to authoritative definitions instead of copying them into lower-level documents.
- Keep implementation details out of product policy unless they are necessary constraints.
- Record implementation evidence in reconciliation or development documentation without allowing it to silently override product authority.
- Preserve historical findings and decision history; append or supersede transparently.
- Report empty, missing, stale, or internally contradictory specifications rather than inventing their contents.

---

## AI Behavior Rules

- Never guess product intent, missing implementation, API behavior, schema meaning, migration safety, or test expectations.
- Inspect before proposing; search before creating; trace before changing.
- State assumptions before they influence implementation.
- Explain material tradeoffs and recommend one option when a choice is authorized.
- Stop and request clarification when multiple authoritative sources conflict or required policy is provisional.
- Never remove existing behavior, compatibility, validation, or tests without evidence that removal is required.
- Never silently change an API, database, event, type, route, error, or UI-state contract.
- Never broaden task scope to include adjacent cleanup.
- Never claim a file, route, service, test, or feature exists without inspecting it.
- Never claim success based only on code changes or intuition.
- Report failed, skipped, blocked, or unavailable verification honestly.
- Report assumptions, unresolved conflicts, provisional decisions, and remaining risks in the final handoff.

---

## Required Verification

Before claiming completion:

- [ ] Re-read the request and verify every requirement against the diff.
- [ ] Inspect the final working-tree status and confirm only intended files changed.
- [ ] Run the narrowest relevant tests first, then the repository's required broader checks.
- [ ] Run formatting, linting, type checking, build, migration, link, or document checks relevant to the changed files.
- [ ] Confirm new tests fail without the intended implementation when regression proof is required.
- [ ] Investigate unexpected failures; do not label them pre-existing without baseline evidence.
- [ ] Check that no Draft Product Decision was implemented as settled policy.
- [ ] Check that no financial rule was duplicated in a consumer layer.

Every completion report must list:

- files inspected;
- files modified;
- tests and checks run;
- the result of each verification step;
- assumptions and baseline failures;
- unresolved issues and remaining risks; and
- Product Decisions involved, including their current statuses.

No completion claim is valid without fresh verification evidence.

---

## Forbidden Actions

- Do not bypass required documentation or repository-local instructions.
- Do not implement a Draft Product Decision as approved behavior.
- Do not invent product policy, financial history, API contracts, schemas, routes, types, components, or data.
- Do not duplicate an existing function, service, route, type, component, validator, formatter, token, or business rule.
- Do not create alternate financial calculations outside Business Services.
- Do not access the Database directly from Frontend, Automation, or AI code.
- Do not bypass authentication, authorization, ownership checks, validation, transactions, idempotency, or required User confirmation.
- Do not rewrite unrelated modules or perform opportunistic refactors.
- Do not silently change or remove public contracts or persisted meaning.
- Do not make destructive Git or filesystem changes without explicit authorization.
- Do not overwrite unrelated working-tree changes.
- Do not weaken, delete, skip, or hide failing tests to make verification appear successful.
- Do not claim completion while required checks are failing, blocked, skipped without explanation, or not run.

---

## Commit Philosophy

- One commit has one purpose.
- One feature, fix, documentation reconciliation, migration, or approved Product Decision should be independently reviewable.
- Do not mix unrelated cleanup with functional work.
- Keep commits small enough that a reviewer can understand the intent, evidence, and rollback impact.
- Commit backend contract changes before dependent frontend changes when separate commits are appropriate.
- Keep a schema migration with the backend change that requires it.
- Name commits by the outcome, not the activity performed.
- Do not commit generated files, local configuration, secrets, or unrelated working-tree changes.
- A Product Decision approval and its implementation may be separate commits; implementation must not precede approval.

---

## Pull Request Philosophy

Every pull request should make one coherent change and answer:

### Why

- What user, product, integrity, or engineering problem does this solve?
- Which authoritative document or Approved Product Decision permits the change?

### What Changed

- Which repositories, layers, contracts, migrations, and user-visible behaviors changed?
- What intentionally did not change?

### Evidence

- Which files, current implementation paths, decisions, and reconciliation findings support the approach?
- How was duplication avoided and existing behavior preserved?

### Verification

- Which tests and checks ran, and what were their exact results?
- Which scenarios prove ownership, financial consistency, rollback, compatibility, and UI states where applicable?

### Risks

- What migration, compatibility, privacy, financial, operational, or rollback risks remain?
- Which assumptions, Draft Product Decisions, follow-up work, or known failures remain unresolved?

Do not hide scope, failures, tradeoffs, or follow-up requirements in review notes. Make them explicit in the pull request description.
