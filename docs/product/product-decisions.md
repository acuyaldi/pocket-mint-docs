# PD-001 — Canonical Net Worth Definition

Status

Pending

Priority

P0

Problem

The Product RFC defines net worth as assets minus debts, while the current frontend, backend, tests, and repository-local guidance use asset balances only. Pocket Mint therefore has two incompatible meanings for its most prominent financial metric.

Current Situation

The implemented asset-only value excludes debt wallets and is presented as net worth. Debt is shown separately. Changing the calculation to match the RFC would materially change displayed values; retaining the calculation without renaming it would preserve a nonstandard and potentially misleading definition.

Options

## Option A — Conventional Net Worth

Define net worth as total assets minus total debts. Show total assets and total debts as supporting figures.

Pros

- Uses the widely understood financial definition.
- Gives users a complete view of financial position.
- Aligns with the current Product RFC and the product principle that every number must be explainable.

Cons

- Changes the value currently displayed to users.
- Requires existing asset-only labels and tests to be reconciled.

## Option B — Asset Position Only

Keep the current calculation and continue calling it net worth.

Pros

- Preserves current displayed values.
- Matches the active calculation paths.

Cons

- Uses a nonstandard meaning for net worth.
- Can overstate a user's financial position when debt is significant.
- Conflicts with the Product RFC.

## Option C — Separate Net Worth and Total Assets

Use conventional net worth as the primary metric and retain the current asset-only calculation under the explicit name “Total Assets” or “Asset Position.”

Pros

- Preserves the useful asset-only figure without mislabeling it.
- Gives users both liquidity context and complete financial position.
- Makes the relationship between assets, debts, and net worth explicit.

Cons

- Adds another headline metric to the interface.
- Requires clear hierarchy to avoid information overload.

Recommendation

Option C — Separate Net Worth and Total Assets.

Reasoning

Pocket Mint should use the conventional definition for a term with an established financial meaning. Retaining the asset-only figure under an accurate label preserves useful information and avoids presenting debt-excluding data as a complete financial position.

Documentation Impact

- `product-rfc.md`
- `design-system.md`
- `screen-spec.md`
- `component-spec.md`
- `pocket-mint-fe/skills/financial-logic.skill.md`

Affected Repositories

- `pocket-mint-docs`
- `pocket-mint-fe`
- `pocket-mint-be`

Implementation Impact

Frontend

High — Primary dashboard and wallet summary labels, hierarchy, and displayed calculations are affected.

Backend

Medium — The canonical summary contract and calculation semantics are affected.

Database

None — The decision does not require a persistence-model change.

Automation

Low — Any automation consuming summary values must distinguish net worth from total assets.

Migration Required

No

Implementation Complexity

High

Decision Owner

Product

---

# PD-002 — Owner-First Authentication Policy

Status

Pending

Priority

P0

Problem

The Product RFC prohibits public registration and describes a single owner by default, while the current authentication experience exposes email signup and Google OAuth. The product also needs explicit public-route boundaries because required health and authentication routes cannot follow the RFC's literal “every request requires authentication” wording.

Current Situation

Financial API routes require verified Supabase identities, but account creation may remain publicly available unless external provider settings reject it. The database can represent multiple users, and authenticated frontend routes do not all receive consistent server-side protection.

Options

## Option A — Permanent Single Owner

Allow only the first owner account and prohibit all additional accounts.

Pros

- Strongly matches the simplest self-hosted privacy model.
- Minimizes account-administration concepts.
- Provides the smallest public attack surface.

Cons

- Leaves the existing multi-user schema capability unused.
- Prevents controlled household or future multi-user use.
- Makes owner recovery and replacement policy more consequential.

## Option B — First Owner with Controlled Access

Allow initial owner bootstrap, then require owner-issued invitations or an allowlist for additional users. Keep only landing, login, authentication callback, password recovery, and health endpoints public.

Pros

- Preserves owner-first and no-public-registration principles.
- Supports deliberate multi-user expansion without open signup.
- Makes public and authenticated boundaries explicit.

Cons

- Introduces an access-administration policy.
- Requires bootstrap, invitation or allowlist, and recovery semantics to remain coherent.

## Option C — Open Multi-User Registration

Permit public email and OAuth registration while maintaining per-user data isolation.

Pros

- Matches the currently exposed signup experience.
- Has the lowest barrier for new users on a shared deployment.

Cons

- Conflicts with the owner-first, no-public-registration RFC.
- Expands abuse and deployment-administration risks.
- Changes Pocket Mint from a closed self-hosted workspace into a public multi-user service.

Recommendation

Option B — First Owner with Controlled Access.

Reasoning

Controlled access best preserves Pocket Mint's privacy-first, self-hosted position while respecting the schema's deliberate multi-user capability. It also provides a precise replacement for the RFC's overly broad authentication language: public system and authentication routes are exceptions, while every user-financial API route remains authenticated and owner-scoped.

Documentation Impact

- `product-rfc.md`
- `screen-spec.md`
- Backend authentication and API documentation

Affected Repositories

- `pocket-mint-docs`
- `pocket-mint-fe`
- `pocket-mint-be`

Implementation Impact

Frontend

High — Signup visibility, access states, route protection, and owner administration are affected.

Backend

High — Account admission and explicit public-route policy are affected.

Database

Low — Existing user support remains applicable, but controlled-access state may require persistence depending on the approved admission mechanism.

Automation

Medium — Automated clients must authenticate as admitted users and cannot rely on public account creation.

Migration Required

No

Implementation Complexity

High

Decision Owner

Product

---

# PD-003 — Installment Lifecycle and Monthly Reporting

Status

Pending

Priority

P0

Problem

Pocket Mint locks the full installment liability against a debt wallet at creation, but it has no approved lifecycle for monthly reporting, term progression, missed processing, settlement, cancellation, or early payoff.

Current Situation

An installment is created with `currentTerm` set to 1 and an active status. The wallet balance is adjusted once for the full total, but no later monthly reporting records are produced and no inspected application path advances the term or status. Visible installment actions do not have an approved product contract.

Options

## Option A — Automatic Calendar Lifecycle

Pocket Mint advances installments on their due schedule, creates a monthly reporting representation without deducting the wallet again, catches up missed periods after downtime, and settles the installment after its final term. Processing uses the owner's reporting timezone and must be idempotent.

Pros

- Fulfils the front-loaded balance-deduction model and monthly reporting promise.
- Keeps progress and reports current without repeated user action.
- Supports deterministic recovery after self-hosted downtime.

Cons

- Requires precise rules for timing, retries, catch-up, and corrections.
- Automatic progression can be inaccurate if a real-world payment is missed or rescheduled.

## Option B — Manual Payment Lifecycle

Advance a term only when the user confirms a payment. Reporting follows confirmed payments rather than the calendar.

Pros

- Keeps the user in control of the real payment state.
- Avoids treating a scheduled due date as proof of payment.
- Reduces reliance on background scheduling.

Cons

- Increases recurring bookkeeping effort.
- Progress and reports become stale when the user does not confirm payments.
- Weakens the automation goal.

## Option C — Scheduled Lifecycle with User Correction

Advance reporting automatically on the due schedule, while allowing the user to mark a term unpaid, correct its date, cancel the plan, or settle it early. Corrections preserve an explainable history and never repeat the initial wallet deduction.

Pros

- Combines low-effort automation with user control.
- Handles downtime and real-world exceptions.
- Supports explainable reporting, cancellation, and payoff behavior.

Cons

- Has the broadest lifecycle semantics.
- Requires clear distinction between scheduled, paid, corrected, cancelled, and settled states.

Recommendation

Option C — Scheduled Lifecycle with User Correction.

Reasoning

This option best matches the principles that automation assists the user, financial mutations are deterministic, and the full liability is deducted only once. The approved lifecycle should use the owner's reporting timezone, prevent duplicate term processing, catch up missed schedules after downtime, preserve monthly reporting entries, and define cancellation and payoff as auditable corrections rather than silent deletion.

Documentation Impact

- `product-rfc.md`
- `screen-spec.md`
- `component-spec.md`
- Backend installment and API documentation

Affected Repositories

- `pocket-mint-docs`
- `pocket-mint-fe`
- `pocket-mint-be`

Implementation Impact

Frontend

High — Installment progress, detail, correction, cancellation, and payoff states are affected.

Backend

High — Lifecycle authority, monthly reporting semantics, idempotency, catch-up, and status transitions are affected.

Database

High — The approved lifecycle may require durable reporting and processing identity beyond the current installment record.

Automation

High — A reliable schedule owner and missed-run policy are central to the decision.

Migration Required

Yes

Implementation Complexity

High

Decision Owner

Product

---

# PD-004 — Installment Fees and Total Liability

Status

Pending

Priority

P0

Problem

The installment interface can display an administration-fee estimate, but the submitted and persisted installment contract does not consistently include that fee. It is therefore unclear whether `grandTotal` represents the user's full liability and whether flat and percentage fees are supported product features.

Current Situation

Principal, interest, duration, monthly amount, fee type, and fee value appear in parts of the current model or interface, but fee behavior is not reconciled end to end. A displayed fee can imply a liability that is not reflected in the stored installment total or debt balance.

Options

## Option A — No Administration Fees

Remove administration fees from the product contract and define `grandTotal` as principal plus interest only.

Pros

- Produces the simplest and clearest calculation.
- Eliminates misleading partial fee support.
- Reduces rounding and validation decisions.

Cons

- Cannot accurately represent installment products with real administration fees.
- Requires users to account for fees separately.

## Option B — Flat Fees Only

Support a fixed administration fee and include it in `grandTotal`, outstanding liability, and the repayment schedule.

Pros

- Covers a common fee model with straightforward explanation.
- Ensures displayed and stored liability agree.
- Has simpler rounding semantics than percentage fees.

Cons

- Does not represent percentage-based fees.
- Preserves only part of the apparent current fee model.

## Option C — Flat and Percentage Fees

Support both fee types and include the resolved fee amount in `grandTotal`, outstanding liability, monthly calculation, previews, and reporting.

Pros

- Represents both fee types already implied by the current domain and interface.
- Makes total debt and available credit reflect the complete obligation.
- Avoids requiring separate manual fee transactions.

Cons

- Requires explicit percentage basis and rounding rules.
- Adds validation and explanation complexity.

Recommendation

Option C — Flat and Percentage Fees.

Reasoning

A finance product should represent the user's complete contractual liability. Supporting both fee types is justified by the current product surface, but approval must define percentage fees as a percentage of principal and require the resolved fee amount to be included exactly once in `grandTotal`. The same total must drive preview, persistence, outstanding debt, available credit, and reporting.

Documentation Impact

- `product-rfc.md`
- `screen-spec.md`
- `component-spec.md`
- Backend installment and API documentation

Affected Repositories

- `pocket-mint-docs`
- `pocket-mint-fe`
- `pocket-mint-be`

Implementation Impact

Frontend

Medium — Fee input, preview, labels, and calculation explanations are affected.

Backend

High — Canonical fee validation and installment totals are affected.

Database

Medium — Existing fee fields must be reconciled with the approved total semantics.

Automation

Medium — Lifecycle reporting must carry the approved fee-inclusive amounts consistently.

Migration Required

Yes

Implementation Complexity

High

Decision Owner

Product

---

# PD-005 — Debt Limit, Utilization Thresholds, and PayLater Semantics

Status

Pending

Priority

P0

Problem

Pocket Mint has inconsistent debt-utilization thresholds, no canonical server-side policy for transactions that exceed a credit limit, and conflicting visual semantics for PayLater debt.

Current Situation

The frontend can reject a debt expense that exceeds available credit, but the backend does not enforce the same rule. Wallet cards use warning at 30% and danger at 80%, while another wallet summary switches to danger above 50%. The Design System assigns red to debt and amber to pending or upcoming installments, while PayLater cards currently use amber.

Options

## Option A — Strict Limit with Unified Debt Semantics

Reject debt-creating operations above the wallet's credit limit. Use normal below 30%, warning from 30% to below 80%, and danger at 80% or above. Present both Credit Card and PayLater wallet debt in red; reserve amber for pending or upcoming installment states.

Pros

- Gives all clients one deterministic rule.
- Prevents the stored debt state from exceeding the configured limit.
- Keeps wallet type and installment status colors semantically distinct.

Cons

- Cannot represent provider-approved over-limit activity without changing the wallet limit first.
- Requires a clear policy for wallets without a configured limit.

## Option B — Allow Over-Limit Debt with Warnings

Allow debt to exceed the configured limit and use escalating warning and danger states. Keep PayLater amber as a distinct debt subtype.

Pros

- Can record real-world over-limit balances.
- Preserves the current PayLater visual distinction.

Cons

- Makes the meaning of credit limit advisory rather than enforceable.
- Uses amber for both a wallet type and temporal installment states.
- Conflicts with the current frontend rejection behavior.

## Option C — Configurable Per Wallet

Let each debt wallet choose strict or advisory limit behavior and define wallet-type-specific utilization colors.

Pros

- Supports different provider behaviors.
- Gives advanced users maximum flexibility.

Cons

- Adds configuration and explanation burden.
- Makes the same utilization value behave differently across wallets.
- Weakens predictable financial validation.

Recommendation

Option A — Strict Limit with Unified Debt Semantics.

Reasoning

Pocket Mint prioritizes deterministic backend financial rules and predictable presentation. A strict canonical limit prevents clients from producing contradictory balances, while the 30% and 80% thresholds retain the more graduated existing policy. Red should communicate debt as a wallet classification; amber should remain available for pending, upcoming, or cautionary installment states.

Documentation Impact

- `product-rfc.md`
- `design-system.md`
- `screen-spec.md`
- `component-spec.md`
- `pocket-mint-fe/skills/financial-logic.skill.md`
- `pocket-mint-fe/skills/ui-system.skill.md`

Affected Repositories

- `pocket-mint-docs`
- `pocket-mint-fe`
- `pocket-mint-be`

Implementation Impact

Frontend

Medium — Validation messages, utilization states, and PayLater presentation are affected.

Backend

High — Credit-limit authority and canonical computed debt values are affected.

Database

Low — Existing limits remain usable; existing over-limit records may require review.

Automation

Medium — Automated transaction creation must follow the same limit policy.

Migration Required

No

Implementation Complexity

Medium

Decision Owner

Product

---

# PD-006 — Financial Reporting Product Surface

Status

Pending

Priority

P1

Problem

Financial reporting is a stated core capability, but Pocket Mint has no authoritative definition of whether current dashboard widgets fulfil that promise or whether a dedicated reports surface is required.

Current Situation

The dashboard presents net worth, wallets, recent transactions, monthly profit and loss, and active installments. There is no dedicated reports route or authoritative screen inventory. Some trend and composition values are synthetic or hard-coded rather than derived from ledger history, and the Design System conflicts with the current choice of dashboard right-rail content.

Options

## Option A — Dashboard Reporting Only

Treat corrected dashboard summaries and drill-down links as the complete reporting capability for the current product scope.

Pros

- Keeps reporting close to the primary daily workflow.
- Avoids a separate navigation destination.
- Has the smallest product surface.

Cons

- Limits historical exploration and comparison.
- Makes the dashboard carry both monitoring and analysis responsibilities.
- May not satisfy the long-term planning goal.

## Option B — Dedicated Reports Surface

Keep the dashboard focused on current status and provide a separate reports screen for ledger-derived historical trends, period comparisons, categories, cash flow, debt, and installments.

Pros

- Clearly fulfils financial reporting as a core capability.
- Separates quick status from deeper analysis.
- Provides a natural home for explainable historical data when available.

Cons

- Adds a primary product surface and navigation destination.
- Depends on canonical historical aggregates and reporting semantics.

## Option C — Export-Led Reporting

Keep the dashboard summaries and treat downloadable data exports as the primary mechanism for deeper analysis.

Pros

- Supports advanced external analysis without a large in-app reporting surface.
- Fits a privacy-first, user-controlled data model.

Cons

- Moves core insight work outside Pocket Mint.
- Provides a weaker experience for users who expect in-app reports.
- Does not resolve the need for explainable dashboard trends.

Recommendation

Option B — Dedicated Reports Surface.

Reasoning

Financial reporting is explicitly a core capability and supports the long-term planning goal. The dashboard should remain a concise current-state view, with debt ratio as its canonical risk widget and active installments available as a focused operational widget. A reports surface should contain only ledger-derived or clearly sourced values; synthetic trends, fixed growth rates, and arbitrary principal or interest splits should not be presented as financial facts.

Documentation Impact

- `product-rfc.md`
- `design-system.md`
- `screen-spec.md`
- `component-spec.md`

Affected Repositories

- `pocket-mint-docs`
- `pocket-mint-fe`
- `pocket-mint-be`

Implementation Impact

Frontend

High — Navigation, dashboard scope, reports presentation, and financial provenance states are affected.

Backend

High — Canonical historical aggregates and report contracts are affected.

Database

Medium — Historical reporting may require durable derived records or queries over existing ledger data.

Automation

Low — Reporting data may be consumed by future automation, but automation is not required to define the product surface.

Migration Required

No

Implementation Complexity

High

Decision Owner

Product

---

# PD-007 — Canonical Transfer Representation

Status

Pending

Priority

P2

Problem

Pocket Mint exposes two apparent transfer representations: the active application uses a Transaction row with `toWalletId`, while the Prisma schema also contains an unused Transfer model.

Current Situation

The active one-row Transaction representation validates both wallets, applies both balance effects atomically, and is covered by transfer consistency and rollback tests. No inspected service or controller uses the separate Transfer model. Keeping both leaves future API, migration, and reconciliation behavior ambiguous.

Options

## Option A — Transaction Is Canonical

Declare Transaction with `toWalletId` as the sole transfer representation and retire the unused Transfer model.

Pros

- Matches the active, tested behavior.
- Keeps transfers in the same ledger and query model as other transactions.
- Removes ambiguity between two domain representations.

Cons

- Requires a deliberate schema migration.
- Gives transfers no independent top-level entity.

## Option B — Separate Transfer Is Canonical

Make Transfer the authoritative entity and derive or associate its source and destination ledger effects.

Pros

- Gives transfers an explicit domain identity.
- Can support transfer-specific metadata independently of ordinary transactions.

Cons

- Conflicts with the active implementation.
- Introduces reconciliation between transfer identity and ledger effects.
- Has no current product requirement that needs a separate entity.

## Option C — Retain Both Representations

Keep Transaction transfers for current behavior and reserve the Transfer model for a future use.

Pros

- Avoids immediate schema removal.
- Preserves optionality for future transfer features.

Cons

- Leaves the domain contract ambiguous.
- Risks two sources of truth and divergent future behavior.
- Retains an entity with no defined product purpose.

Recommendation

Option A — Transaction Is Canonical.

Reasoning

The one-row Transaction model already provides atomic and tested transfer behavior. No reconciled product requirement justifies a second representation, and retaining one only for unspecified future use would undermine the goal of an explainable ledger.

Documentation Impact

- `product-rfc.md`
- Backend architecture, database, and API documentation

Affected Repositories

- `pocket-mint-docs`
- `pocket-mint-be`

Implementation Impact

Frontend

Low — Existing transfer behavior already follows the recommended representation.

Backend

Medium — Domain and schema contracts are affected, while the active service path remains aligned.

Database

Medium — The unused model and its relations are affected.

Automation

Low — Integrations receive one unambiguous transfer contract.

Migration Required

Yes

Implementation Complexity

Medium

Decision Owner

Product

---

# PD-008 — Canonical Visual Token Contract

Status

Pending

Priority

P2

Problem

The Design System, repository-local frontend guidance, and active CSS disagree about font families, financial-number typography, radius aliases, semantic color names, background color, and primary-button color.

Current Situation

The Design System primarily specifies Inter and a documented radius scale, while local guidance names Hanken Grotesk and JetBrains Mono and the active CSS resolves heading and mono utilities to Inter. The token table and prose use different background values, amber is described as Accent but represented as tertiary, and the documented slate primary button conflicts with the implemented mint primary button.

Options

## Option A — Adopt the Design System Token Table

Use Inter for all typography with tabular figures for financial data, retain the documented radius scale, use the token table's background value, map amber to a warning or tertiary role, and use mint for primary actions.

Pros

- Uses the most complete existing product-level token definition.
- Minimizes font complexity while preserving numerical alignment.
- Closely matches the active light palette and primary-action behavior.

Cons

- Requires conflicting prose and repository-local guidance to change.
- Does not use the loaded display and monospace font families.

## Option B — Adopt Repository-Local Font Guidance

Use Hanken Grotesk for headings, Inter for body text, and JetBrains Mono for financial values, while reconciling colors and radii around the current frontend conventions.

Pros

- Gives headings and financial values distinct visual roles.
- Uses font families already referenced by the frontend.

Cons

- Increases typographic complexity and loading cost.
- Monospace currency can feel more technical than calm or approachable.
- Conflicts with the product-level typography definition.

## Option C — Freeze the Current Rendered Contract

Treat the currently resolved CSS values and aliases as authoritative, including Inter-backed heading and mono utilities, then rewrite documentation to match.

Pros

- Minimizes immediate visual change.
- Records the behavior users currently see.

Cons

- Preserves misleading utility names and accidental inconsistencies.
- Does not resolve raw values or unclear semantic roles by design.
- Makes implementation history, rather than product intent, authoritative.

Recommendation

Option A — Adopt the Design System Token Table.

Reasoning

Inter with tabular figures supports legible, aligned financial data without introducing a separate monospace voice. The documented radius scale provides intentional hierarchy, while mint primary actions and amber warning semantics align the token contract with Pocket Mint's growth and caution language. The canonical light background should be `#f8f9ff`; amber should be named warning or tertiary rather than accent, and third-party brand colors should remain explicit exceptions.

Documentation Impact

- `design-system.md`
- `screen-spec.md`
- `component-spec.md`
- `pocket-mint-fe/skills/ui-system.skill.md`

Affected Repositories

- `pocket-mint-docs`
- `pocket-mint-fe`

Implementation Impact

Frontend

High — Global tokens, typography utilities, radii, raw values, and financial-number styling are affected.

Backend

None

Database

None

Automation

None

Migration Required

No

Implementation Complexity

Medium

Decision Owner

Product

---
