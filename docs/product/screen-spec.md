# Dashboard

## Purpose

Give the User a trustworthy current-state overview of their financial position and the Financial Events that need attention.

---

## Primary Goal

Understand current Net Worth, what contributes to it, and where to go next.

---

## Information Hierarchy

1. Financial Summary: Net Worth, Total Assets, Total Outstanding Debt, and Reporting Cutoff.
2. Debt Wallet status: Outstanding Balance, Available Credit, and Debt Ratio.
3. Income and Expense for the current Reporting Period.
4. Active Installments, Monthly Payment, and next Installment Due Date.
5. Recent Financial Events.
6. Trend information only when supported by Historical Records.

---

## Required Data

- User.
- Financial Summary and Reporting Cutoff.
- Asset Wallet and Debt Wallet balances.
- Income and Expense for the current Reporting Period.
- Active Installments and their Installment Schedules.
- Recent Financial Events.
- Historical Records when a Trend is shown.

---

## User Actions

- Open Wallet List or a Wallet Detail.
- Open Transaction List or a Transaction Detail.
- Open Installments or an Installment Detail.
- Open Reports.
- Start Create Wallet, Create Transaction, Create Installment, or Transfer.
- Inspect the meaning and source of a Derived Metric.

---

## Navigation

Users arrive after successful Authentication or by returning from another authenticated screen.

Users can continue to Wallet List, Wallet Detail, Transaction List, Transaction Detail, Create Transaction, Transfer, Installments, Installment Detail, Create Installment, Reports, Settings, or Profile.

---

## Empty State

When the User has no wallets or Financial Events, show a zero Financial Summary with a clear Reporting Cutoff, explain that no financial activity is recorded, and direct the User to Create Wallet. Do not show a fabricated Trend.

---

## Loading State

Identify each business concept being prepared without presenting incomplete values as current facts. Previously known values may appear only when their Reporting Cutoff remains visible.

---

## Error State

State which financial information is unavailable, avoid combining values from different Reporting Cutoffs, and let the User retry or continue to an unaffected screen.

---

## Success State

Show one internally consistent Financial Summary with explainable supporting values and clear paths to the underlying wallets, Financial Events, Installments, and Reports.

---

## Business Rules

- Follow the Product RFC sections **Product Principles**, **Financial Rules**, and **User Experience Principles**.
- PD-001, **Canonical Net Worth Definition** (Approved), governs Net Worth, Total Assets, Total Outstanding Debt, Negative Net Worth, and Reporting Cutoff.
- PD-005, **Debt Utilization Policy** (Draft), provisionally governs Credit Limit enforcement and Debt Ratio states.
- PD-006, **Financial Reporting Surface** (Draft), provisionally separates current-state Dashboard information from historical Reports.
- Every Derived Metric must use its Canonical Calculation. Synthetic financial facts and fabricated Trends are prohibited.

---

## Notes

The Dashboard is a current-state overview, not a substitute for Reports. The final Dashboard and Reports boundary remains provisional under PD-006 (Draft).

---

# Wallet List

## Purpose

Give the User one place to understand every tracked Asset Wallet and Debt Wallet.

---

## Primary Goal

Find a wallet and understand its contribution to Total Assets or Total Outstanding Debt.

---

## Information Hierarchy

1. Total Assets and Total Outstanding Debt at a Reporting Cutoff.
2. Asset Wallets and their Wallet Balances.
3. Debt Wallets and their Outstanding Balances.
4. Available Credit and Debt Ratio for each Debt Wallet.

---

## Required Data

- User-owned Asset Wallets and Debt Wallets.
- Wallet Balance for every wallet.
- Outstanding Balance, Credit Limit, Available Credit, and Debt Ratio for each Debt Wallet.
- Total Assets, Total Outstanding Debt, and Reporting Cutoff.

---

## User Actions

- Open Wallet Detail.
- Start Create Wallet.
- Start Create Transaction for a selected wallet.
- Start Transfer.

---

## Navigation

Users arrive from Dashboard or authenticated product navigation.

Users can continue to Wallet Detail, Create Wallet, Create Transaction, Transfer, Dashboard, Reports, Settings, or Profile.

---

## Empty State

Explain that no wallets are tracked and direct the User to Create Wallet. Do not imply that an empty wallet set represents the User's complete financial life.

---

## Loading State

Indicate that wallet information is being prepared without showing provisional totals as confirmed Total Assets or Total Outstanding Debt.

---

## Error State

Explain that wallet information is unavailable, preserve the distinction between missing information and a zero Wallet Balance, and let the User retry.

---

## Success State

Every wallet is clearly classified as an Asset Wallet or Debt Wallet, and all displayed totals reconcile with the visible wallet values at the same Reporting Cutoff.

---

## Business Rules

- Follow the Product RFC sections **Wallet Types**, **Net Worth**, **Outstanding**, **Available Credit**, **Debt Ratio**, and **Business Rules**.
- PD-001, **Canonical Net Worth Definition** (Approved), governs Total Assets and Total Outstanding Debt.
- PD-005, **Debt Utilization Policy** (Draft), provisionally governs Credit Limit enforcement, Debt Ratio thresholds, and PayLater debt semantics.
- Cash, Bank, and E-Wallet are Asset Wallet types. Credit Card and PayLater are Debt Wallet types.
- Wallet classification comes from wallet type, not from the sign of the Wallet Balance.

---

## Notes

This screen summarizes wallets. Financial Events belong in Transaction List, and historical analysis belongs in Reports under PD-006 (Draft).

---

# Wallet Detail

## Purpose

Explain the current state and recorded activity of one User-owned wallet.

---

## Primary Goal

Understand the Wallet Balance and the Financial Events that produced it.

---

## Information Hierarchy

1. Wallet type and Wallet Balance.
2. For a Debt Wallet: Outstanding Balance, Credit Limit, Available Credit, and Debt Ratio.
3. Financial Events associated with the wallet, ordered from most recent.
4. Related Installments when the wallet is a Debt Wallet.

---

## Required Data

- User ownership.
- Wallet type and Wallet Balance.
- Debt Wallet metrics when applicable.
- Related Financial Events and Categories.
- Related Installments when applicable.

---

## User Actions

- Start Create Transaction for the wallet.
- Start Transfer using the wallet.
- Open Transaction Detail.
- Open Installment Detail when related.
- Correct wallet information.
- Request wallet deletion with a clear explanation of its effect on Historical Records and financial consistency.

---

## Navigation

Users arrive from Wallet List, Dashboard, Transaction Detail, Transfer, or Installment Detail.

Users can continue to Create Transaction, Transfer, Transaction Detail, Installment Detail, Wallet List, Dashboard, or Reports.

---

## Empty State

When the wallet has no Financial Events, show its current Wallet Balance and explain that no recorded activity is available. Offer Create Transaction or Transfer where valid.

---

## Loading State

Keep the wallet type visible when known, identify which financial sections are still loading, and do not present partial debt calculations as complete.

---

## Error State

If the wallet cannot be found or does not belong to the User, do not expose financial information. Otherwise identify unavailable sections and allow retry or return to Wallet List.

---

## Success State

The Wallet Balance, debt metrics, and visible Financial Events reconcile and give the User a clear explanation of the wallet's current state.

---

## Business Rules

- Follow the Product RFC sections **Wallet Types**, **Financial Rules**, **Business Rules**, and **Error Handling**.
- PD-005, **Debt Utilization Policy** (Draft), provisionally governs Credit Limit enforcement and Debt Ratio states.
- Every wallet belongs to exactly one User.
- Any correction or deletion must preserve financial consistency and keep Historical Records explainable.

---

## Notes

The final policy for wallet deletion and Historical Records is unresolved. The screen must not promise retention or deletion behavior that has not been approved.

---

# Create Wallet

## Purpose

Allow the User to begin tracking one Asset Wallet or Debt Wallet with an explicit starting financial state.

---

## Primary Goal

Create an accurately classified wallet with a Wallet Balance the User understands.

---

## Information Hierarchy

1. Wallet type.
2. Asset Wallet or Debt Wallet classification.
3. Wallet Balance at creation.
4. Credit Limit when creating a Credit Card or PayLater Debt Wallet.
5. Confirmation of the financial meaning before creation.

---

## Required Data

- User.
- One wallet type: Cash, Bank, E-Wallet, Credit Card, or PayLater.
- Wallet Balance at creation.
- Credit Limit for a Debt Wallet.

---

## User Actions

- Choose a wallet type.
- Enter the Wallet Balance at creation.
- Enter the Credit Limit when required.
- Review the resulting Asset Wallet or Debt Wallet classification.
- Confirm creation or cancel.

---

## Navigation

Users arrive from Wallet List or Dashboard.

After cancellation, users return to the originating screen. After success, users continue to Wallet Detail or Wallet List.

---

## Empty State

Begin with no assumed wallet type or financial value. Explain the distinction between Asset Wallet and Debt Wallet before the User commits a choice.

---

## Loading State

Do not allow confirmation until the required wallet choices and their business meaning are available.

---

## Error State

Explain which business rule prevented creation, preserve valid user-entered information, and allow correction or cancellation.

---

## Success State

Confirm the created wallet, its classification, and its Wallet Balance. For a Debt Wallet, also confirm the Credit Limit and derived Available Credit.

---

## Business Rules

- Follow the Product RFC sections **Money Precision**, **Wallet Types**, **Validation Rules**, and **Business Rules**.
- PD-005, **Debt Utilization Policy** (Draft), provisionally governs whether a Debt Wallet may begin above its Credit Limit.
- Wallet classification is determined by wallet type.
- Financial formatting must never change the value the User entered.

---

## Notes

Create Wallet defines one wallet only. Financial activity after creation is recorded through Financial Events.

---

# Transaction List

## Purpose

Provide an ordered record of the User's Financial Events.

---

## Primary Goal

Find a Financial Event and understand its type, amount, timing, and financial context.

---

## Information Hierarchy

1. Reporting Period and ordered Financial Events.
2. Financial Event type: Income, Expense, Transfer, Installment creation, or Installment Payment.
3. Amount, Financial Event date, and related wallet or wallets.
4. Category and related Installment when applicable.

---

## Required Data

- User-owned Financial Events.
- Reporting Period and Reporting Timezone.
- Income, Expense, Transfer, Installment, or Installment Payment classification.
- Related wallets and Category.
- Related Installment when applicable.

---

## User Actions

- Open Transaction Detail.
- Start Create Transaction.
- Start Transfer.
- Open Wallet Detail or Installment Detail from a related Financial Event.
- Move between available Reporting Periods.

---

## Navigation

Users arrive from Dashboard, Wallet Detail, Reports, or authenticated product navigation.

Users can continue to Transaction Detail, Create Transaction, Transfer, Wallet Detail, Installment Detail, Dashboard, or Reports.

---

## Empty State

Explain that no Financial Events exist for the selected Reporting Period. Offer Create Transaction or Transfer without fabricating activity.

---

## Loading State

Keep the selected Reporting Period visible and do not present an incomplete sequence as the complete Ledger for that period.

---

## Error State

Explain that Financial Events could not be retrieved, distinguish failure from an empty Reporting Period, and allow retry.

---

## Success State

Show a complete, ordered set of Financial Events for the selected Reporting Period with clear links to related business concepts.

---

## Business Rules

- Follow the Product RFC sections **Business Rules**, **Financial Rules**, and **Error Handling**.
- PD-003, **Installment Lifecycle Policy** (Draft), provisionally governs Installment Payment creation and Reporting Period assignment.
- PD-007, **Canonical Transfer Representation** (Draft), provisionally establishes one Transfer as one Financial Event with source and destination wallets.
- A Transfer is neither Income nor Expense.
- An Installment Payment must not deduct the Wallet Balance again when the Total Obligation was front-loaded.

---

## Notes

The Transaction List presents Ledger facts. It must distinguish Imported, Scheduled, Projected, Confirmed, and Paid states according to the glossary.

---

# Transaction Detail

## Purpose

Explain one Financial Event and its effect on the User's financial position.

---

## Primary Goal

Understand exactly what happened, when it happened, and which financial values changed.

---

## Information Hierarchy

1. Financial Event type, amount, and Financial Event date.
2. Related wallet or source and destination wallets.
3. Effect on Wallet Balance, Outstanding Balance, or both.
4. Category and related Installment when applicable.
5. Origin and confirmation state when the Financial Event was Imported or created by Automation.

---

## Required Data

- User ownership.
- Financial Event type and amount.
- Financial Event date, Reporting Period, and Reporting Timezone.
- Related wallet or wallets.
- Category when applicable.
- Related Installment when applicable.
- Recorded origin and confirmation state when applicable.

---

## User Actions

- Open related Wallet Detail.
- Open related Installment Detail.
- Request a correction while preserving an explainable Historical Record.
- Request deletion only under approved deletion rules.
- Return to Transaction List or the originating screen.

---

## Navigation

Users arrive from Transaction List, Dashboard, Wallet Detail, Installment Detail, or Reports.

Users can continue to related Wallet Detail, related Installment Detail, Transaction List, Dashboard, or Reports.

---

## Empty State

This screen has no valid empty state. A missing or inaccessible Financial Event is handled as an Error State.

---

## Loading State

Identify that the Financial Event is being prepared without showing unverified financial effects as confirmed facts.

---

## Error State

If the Financial Event does not exist or does not belong to the User, reveal no financial details and provide a safe return path. For partial failure, identify unavailable supporting information.

---

## Success State

The User can trace the Financial Event to its wallet effects, Category, Reporting Period, and any related Installment.

---

## Business Rules

- Follow the Product RFC sections **Business Rules**, **Error Handling**, and **User Experience Principles**.
- PD-003, **Installment Lifecycle Policy** (Draft), applies when the Financial Event is an Installment Payment.
- PD-007, **Canonical Transfer Representation** (Draft), applies when the Financial Event is a Transfer.
- Corrections and deletions must preserve financial consistency and keep Historical Records explainable.

---

## Notes

This screen explains a recorded fact. It must never substitute a Derived Metric or Projected value for the Financial Event itself.

---

# Create Transaction

## Purpose

Allow the User to record one Income or Expense as a Financial Event.

---

## Primary Goal

Record an accurate Financial Event with the correct wallet, amount, Category, and Financial Event date.

---

## Information Hierarchy

1. Income or Expense classification.
2. Amount and related wallet.
3. Category.
4. Financial Event date and resulting Reporting Period.
5. Review of the expected Wallet Balance or Outstanding Balance effect.

---

## Required Data

- User.
- Income or Expense classification.
- Amount.
- User-owned Asset Wallet or Debt Wallet.
- User-owned Category.
- Financial Event date and Reporting Timezone.
- Current Wallet Balance and applicable Debt Wallet metrics.

---

## User Actions

- Choose Income or Expense.
- Choose a wallet and Category.
- Enter the amount and Financial Event date.
- Review the financial effect.
- Confirm creation or cancel.

---

## Navigation

Users arrive from Dashboard, Transaction List, Wallet List, or Wallet Detail.

After cancellation, users return to the originating screen. After success, users continue to Transaction Detail, Transaction List, or Wallet Detail.

---

## Empty State

Begin with no assumed Financial Event type, wallet, Category, amount, or date. If the User has no wallet, direct them to Create Wallet before recording activity.

---

## Loading State

Do not allow confirmation until required wallet and Category choices and the expected financial effect are available.

---

## Error State

Explain the violated business rule, preserve valid user-entered information, and allow correction or cancellation. Never silently ignore a failed Financial Event.

---

## Success State

Confirm the recorded Income or Expense, its wallet effect, Category, and Reporting Period, then provide access to Transaction Detail.

---

## Business Rules

- Follow the Product RFC sections **Money Precision**, **Business Rules**, **Validation Rules**, and **Error Handling**.
- PD-005, **Debt Utilization Policy** (Draft), provisionally governs Credit Limit enforcement for an Expense recorded against a Debt Wallet.
- Amount must be greater than zero.
- The wallet and Category must belong to the User.
- Financial formatting must never change the recorded amount.

---

## Notes

Transfer and Installment creation have dedicated screens and must not be presented as ordinary Income or Expense entry paths.

---

# Transfer

## Purpose

Allow the User to move money between two different User-owned wallets without treating the movement as Income or Expense.

---

## Primary Goal

Complete one Transfer with clear and symmetric effects on both wallets.

---

## Information Hierarchy

1. Source wallet and its Wallet Balance.
2. Destination wallet and its Wallet Balance or Outstanding Balance.
3. Transfer amount.
4. Expected effect on both wallets.
5. Confirmation that the movement is neither Income nor Expense.

---

## Required Data

- User.
- Source wallet and destination wallet.
- Current Wallet Balance for both wallets.
- Outstanding Balance when a Debt Wallet is involved.
- Transfer amount and Transfer date.

---

## User Actions

- Choose a source wallet.
- Choose a different destination wallet.
- Enter the Transfer amount and Transfer date.
- Review both wallet effects.
- Confirm the Transfer or cancel.

---

## Navigation

Users arrive from Dashboard, Wallet List, Wallet Detail, or Transaction List.

After cancellation, users return to the originating screen. After success, users continue to Transaction Detail, Wallet Detail, or Transaction List.

---

## Empty State

If fewer than two eligible User-owned wallets exist, explain that a Transfer requires two different wallets and direct the User to Create Wallet.

---

## Loading State

Do not allow confirmation until both wallets and their current financial states are available.

---

## Error State

Explain whether the Transfer failed because of ownership, identical wallets, an invalid amount, or another financial rule. Confirm that no partial movement occurred and allow correction or cancellation.

---

## Success State

Confirm one Transfer and show the resulting financial state of both wallets. The movement must appear once in the Ledger and must not appear as Income or Expense.

---

## Business Rules

- Follow the Product RFC sections **Business Rules**, **Validation Rules**, and **Error Handling**.
- PD-007, **Canonical Transfer Representation** (Draft), provisionally establishes one Transfer as one Financial Event with source and destination wallets.
- Source and destination wallets must be different and belong to the User.
- Both wallet effects must succeed or fail together.
- A Transfer does not create or remove value from the User's tracked financial position.

---

## Notes

The Transfer screen defines the user's business action. The internal representation remains governed by PD-007 (Draft).

---

# Installments

## Purpose

Give the User a consolidated view of tracked Installments, their obligations, schedules, and current states.

---

## Primary Goal

Understand what is owed, what is due next, and how each Installment is progressing.

---

## Information Hierarchy

1. Total Outstanding Debt attributable to tracked Debt Wallets.
2. Active Installments and their Total Obligations.
3. Monthly Payment, current term, and next Installment Due Date.
4. Installment Schedule progress and state.
5. Settled or cancelled Installments as Historical Records.

---

## Required Data

- User-owned Installments.
- Related Debt Wallet and Outstanding Balance.
- Principal, Interest, Administrative Fee when applicable, and Total Obligation.
- Monthly Payment and Installment Schedule.
- Installment Due Dates, term progress, and lifecycle state.
- Reporting Timezone.

---

## User Actions

- Open Installment Detail.
- Start Create Installment.
- Open the related Wallet Detail.
- Review active and historical Installments.

---

## Navigation

Users arrive from Dashboard, Wallet Detail, Reports, or authenticated product navigation.

Users can continue to Installment Detail, Create Installment, Wallet Detail, Dashboard, Transaction Detail, or Reports.

---

## Empty State

Explain that no Installments are tracked and direct the User to Create Installment. Do not show projected obligations as recorded facts.

---

## Loading State

Identify which Installment information is being prepared and do not present incomplete schedule progress as current.

---

## Error State

Explain that Installment information is unavailable, distinguish failure from an empty set, and allow retry or navigation to unaffected screens.

---

## Success State

Every Installment has an explainable Total Obligation, related Debt Wallet, Monthly Payment, schedule position, and lifecycle state.

---

## Business Rules

- Follow the Product RFC sections **Installment Model**, **Installment Calculator**, and **Business Rules**.
- PD-003, **Installment Lifecycle Policy** (Draft), provisionally governs term progression, Installment Payments, settlement, payoff, cancellation, Reporting Timezone, and catch-up behavior.
- PD-004, **Installment Fee Treatment** (Draft), provisionally governs Administrative Fee and Total Obligation.
- The Total Obligation is reflected in the Debt Wallet once. Future Installment Payments must not deduct the Wallet Balance again.

---

## Notes

Payoff and cancellation actions must not be presented as active behavior until their financial and Historical Record effects are approved under PD-003 (Draft).

---

# Installment Detail

## Purpose

Explain one Installment from its original obligation through its schedule and current lifecycle state.

---

## Primary Goal

Understand the complete cost, monthly commitment, progress, and relationship to the Debt Wallet.

---

## Information Hierarchy

1. Total Obligation and related Debt Wallet.
2. Principal, Interest, and Administrative Fee when applicable.
3. Monthly Payment, current term, and Installment Due Date.
4. Installment Schedule and recorded Installment Payments.
5. Current lifecycle state and Historical Record.

---

## Required Data

- User ownership.
- Related Debt Wallet and Outstanding Balance.
- Principal, Interest, Administrative Fee when applicable, and Total Obligation.
- Monthly Payment and duration.
- Installment Schedule, Installment Due Dates, and Installment Payments.
- Reporting Timezone and lifecycle state.

---

## User Actions

- Open related Wallet Detail.
- Open related Transaction Detail.
- Review the Installment Schedule and Installment Payments.
- Provisional under PD-003 (Draft): request payoff or cancellation, review the full consequence, and explicitly confirm or abandon the action.
- Return to Installments.

---

## Navigation

Users arrive from Installments, Dashboard, Wallet Detail, Transaction Detail, or Reports.

Users can continue to related Wallet Detail, related Transaction Detail, Installments, Dashboard, or Reports.

---

## Empty State

This screen has no valid empty state. Missing schedule information is an Error State and must not be replaced with invented progress.

---

## Loading State

Keep known Installment identity visible, distinguish recorded facts from information still loading, and do not infer lifecycle progress from dates alone.

---

## Error State

If the Installment does not exist or does not belong to the User, reveal no financial details. If schedule information is incomplete, identify the gap and avoid presenting a false state.

---

## Success State

The User can reconcile Total Obligation, Monthly Payment, recorded Installment Payments, schedule progress, and the Debt Wallet's Outstanding Balance without double-counting.

---

## Business Rules

- Follow the Product RFC sections **Installment Model**, **Installment Calculator**, **Business Rules**, and **User Experience Principles**.
- PD-003, **Installment Lifecycle Policy** (Draft), provisionally governs schedule progression, settlement, payoff, cancellation, and Installment Payment behavior.
- PD-004, **Installment Fee Treatment** (Draft), provisionally governs Administrative Fee and Total Obligation.
- An Installment Payment is distinct from the initial front-loaded application of Total Obligation.

---

## Notes

Scheduled, Projected, Paid, Confirmed, Settled, and cancelled states must retain their glossary meanings. Scheduled or due status is not proof of payment.

---

# Create Installment

## Purpose

Allow the User to create one Installment with a complete, reviewable obligation and schedule.

---

## Primary Goal

Understand and confirm the Total Obligation before it is applied to a Debt Wallet.

---

## Information Hierarchy

1. Related Debt Wallet and Available Credit.
2. Forward or reverse calculation mode.
3. Principal, Interest or Monthly Payment, and duration.
4. Administrative Fee when applicable.
5. Total Obligation, Monthly Payment, and Installment Schedule preview.
6. Effect on Outstanding Balance before confirmation.

---

## Required Data

- User-owned Debt Wallet.
- Credit Limit, Outstanding Balance, and Available Credit.
- Principal and duration.
- Interest for forward calculation.
- Monthly Payment for reverse calculation.
- Administrative Fee when applicable.
- Total Obligation, Installment Schedule, first Installment Due Date, and Reporting Timezone.

---

## User Actions

- Choose a Debt Wallet.
- Choose forward or reverse calculation.
- Enter the required calculation values.
- Enter an Administrative Fee when applicable.
- Review Principal, Interest, Administrative Fee, Total Obligation, Monthly Payment, schedule, and debt effect.
- Confirm creation or cancel.

---

## Navigation

Users arrive from Installments, Dashboard, Wallet Detail, or Transaction List.

After cancellation, users return to the originating screen. After success, users continue to Installment Detail, Installments, Wallet Detail, or Transaction Detail.

---

## Empty State

Begin with no assumed Debt Wallet or calculation values. If no Debt Wallet exists, explain that an Installment requires one and direct the User to Create Wallet.

---

## Loading State

Do not allow confirmation until current Debt Wallet values and all Canonical Calculations required for the review are available.

---

## Error State

Explain invalid values or violated financial rules, preserve valid user-entered information, and show no Total Obligation that cannot be explained by its inputs.

---

## Success State

Confirm the Installment, Total Obligation, Monthly Payment, related Debt Wallet, first Installment Due Date, and the one-time change to Outstanding Balance.

---

## Business Rules

- Follow the Product RFC sections **Money Precision**, **Installment Model**, **Installment Calculator**, **Validation Rules**, and **Business Rules**.
- PD-003, **Installment Lifecycle Policy** (Draft), provisionally governs Installment Schedule creation, Reporting Timezone, and lifecycle behavior.
- PD-004, **Installment Fee Treatment** (Draft), provisionally governs Administrative Fee types, basis, and inclusion in Total Obligation.
- PD-005, **Debt Utilization Policy** (Draft), provisionally governs Credit Limit enforcement.
- The full Total Obligation affects Outstanding Balance once at creation.

---

## Notes

Reverse calculation behavior is required by the Product RFC, but rounding and impossible-input behavior remain unresolved in the reconciliation report. The screen must not conceal that ambiguity in product documentation.

---

# Reports

## Purpose

Help the User understand financial activity and change across Reporting Periods using only Ledger-derived information.

---

## Primary Goal

Answer a financial question about a Reporting Period or Trend with explainable evidence.

---

## Information Hierarchy

1. Selected Reporting Period or Reporting Cutoff.
2. Financial Summary for the relevant cutoff.
3. Income and Expense across the selected period.
4. Category breakdowns.
5. Debt Wallet and Installment reporting.
6. Trends supported by Historical Records.
7. Supporting Financial Events.

---

## Required Data

- User.
- Reporting Period, Reporting Cutoff, and Reporting Timezone.
- Ledger and Historical Records.
- Financial Summary and Canonical Calculations.
- Income, Expense, Transfer, Category, Debt Wallet, and Installment information relevant to the selected period.
- Trend data derived from consecutive Reporting Periods.

---

## User Actions

- Choose an available Reporting Period or Reporting Cutoff.
- Review Financial Summary, Income, Expense, debt, Installment, Category, and Trend information.
- Open supporting Transaction Detail, Wallet Detail, or Installment Detail.
- Inspect the meaning and source of a Derived Metric.

---

## Navigation

Users arrive from Dashboard, Wallet List, Wallet Detail, Transaction List, Transaction Detail, Installments, or Installment Detail.

Users can continue to supporting Transaction Detail, Wallet Detail, Installment Detail, Dashboard, or another Reporting Period.

---

## Empty State

When a Reporting Period has no Financial Events, state that no recorded activity exists for that period. Where Historical Records are incomplete, show an explicit gap rather than a fabricated Trend.

---

## Loading State

Keep the selected Reporting Period or Reporting Cutoff visible. Do not combine partially loaded values into a Financial Summary or Trend.

---

## Error State

Identify unavailable report sections, avoid presenting partial totals as complete, and let the User retry or inspect unaffected supporting records.

---

## Success State

Every displayed figure is traceable to a Canonical Calculation and supporting Financial Events for the selected Reporting Period or Reporting Cutoff.

---

## Business Rules

- Follow the Product RFC sections **Product Principles**, **Financial Rules**, **User Experience Principles**, and **Error Handling**.
- PD-001, **Canonical Net Worth Definition** (Approved), governs Financial Summary and Net Worth at each Reporting Cutoff.
- PD-003, **Installment Lifecycle Policy** (Draft), provisionally governs Installment Payment reporting and Reporting Timezone.
- PD-006, **Financial Reporting Surface** (Draft), provisionally establishes Reports as a dedicated product surface and Dashboard as a current-state overview.
- Reports must use Ledger and Historical Records. Synthetic or placeholder financial facts are prohibited.

---

## Notes

The final reporting scope and the Dashboard boundary remain provisional under PD-006 (Draft). Missing history must remain visible as missing history.

---

# Settings

## Purpose

Give the Owner one place to review and manage approved product-level preferences and access policy.

---

## Primary Goal

Understand and deliberately change a product-level preference that affects financial behavior or access.

---

## Information Hierarchy

1. Reporting Timezone and its effect on Reporting Periods and Installment Due Dates.
2. Owner identity and access policy.
3. Owner-controlled admission and Invited User access.
4. Consequences of any proposed change.

---

## Required Data

- Owner.
- Reporting Timezone.
- User and Invited User access information.
- Business consequences of changing a setting.

---

## User Actions

- Review the Reporting Timezone.
- Provisional under PD-003 (Draft): change the Reporting Timezone after reviewing its effect on Reporting Periods and Installment Due Dates.
- Review Owner-controlled admission and Invited User status.
- Confirm or cancel a permitted change.

---

## Navigation

Users arrive from Profile, Dashboard, or authenticated product navigation.

Users can continue to Profile, Dashboard, or the screen affected by a confirmed setting change.

---

## Empty State

If no product-level setting is approved for the User, explain that no settings are available. Do not expose controls governed only by an unresolved Draft Product Decision as approved behavior.

---

## Loading State

Do not permit changes until the current setting and its business consequences are known.

---

## Error State

Explain which setting could not be retrieved or changed, preserve the last confirmed value, and allow retry or cancellation.

---

## Success State

Confirm the new setting, when it takes effect, and which product behavior it changes.

---

## Business Rules

- Follow the Product RFC sections **Authentication**, **Business Rules**, and **User Experience Principles**.
- PD-002, **Owner-First Access Policy** (Approved), governs Owner-controlled admission and Invited User access.
- PD-003, **Installment Lifecycle Policy** (Draft), provisionally governs Reporting Timezone.
- Only the Owner may control installation-level access and settings described by the glossary.

---

## Notes

This screen must expose only settings supported by approved product policy. Draft policy may be described as provisional but must not be presented as settled.

---

# Authentication

## Purpose

Establish User identity and protect access to private financial information.

---

## Primary Goal

Enter Pocket Mint as an authenticated User under the approved Owner-first access policy.

---

## Information Hierarchy

1. Pocket Mint identity and privacy expectation.
2. Authentication requirements.
3. Access recovery.
4. First Owner or Invited User admission when applicable.

---

## Required Data

- User identity required for Authentication.
- Access state.
- Owner, User, or Invited User relationship when applicable.
- Recovery state when requested.

---

## User Actions

- Authenticate.
- Request access recovery.
- Initialize the first Owner while the installation is uninitialized or complete Owner-approved admission as an Invited User.
- Cancel recovery and return to Authentication.

---

## Navigation

Unauthenticated users arrive when opening a protected product screen or following an approved access or recovery path.

After successful Authentication, users continue to Dashboard. Failed or cancelled recovery returns to Authentication.

---

## Empty State

Present a neutral Authentication state with no assumed identity and no financial information.

---

## Loading State

Make the pending Authentication or recovery state clear and prevent repeated consequential submission while the result is unknown.

---

## Error State

Explain that Authentication or recovery failed without exposing whether another User exists. Preserve privacy and offer a safe retry or recovery path.

---

## Success State

Confirm authenticated access and continue to Dashboard without exposing another User's financial information.

---

## Business Rules

- Follow the Product RFC section **Authentication** and the privacy and ownership rules in **Business Rules**.
- PD-002, **Owner-First Access Policy** (Approved), governs first Owner initialization, Invited User admission, public access exceptions, and the prohibition on public registration.
- Public self-registration is prohibited after initialization.
- Only non-financial landing content, first-Owner initialization while uninitialized, authentication, approved authentication callbacks, invitation or allowlist admission completion, account recovery, and minimal health capabilities may be reached without an authenticated User.
- Every protected financial screen requires an authenticated User.
- Authentication must never grant access to another User's resources.

---

## Notes

Authentication presents only the access path applicable to the installation and admission state. It must not expose another User's account, invitation, or financial information.

---

# Profile

## Purpose

Let the authenticated User review their identity and relationship to the Pocket Mint installation.

---

## Primary Goal

Confirm which User is active and understand whether that User is the Owner or an Invited User.

---

## Information Hierarchy

1. User identity.
2. Owner or Invited User relationship when applicable.
3. Access status.
4. Paths to Settings and Authentication exit.

---

## Required Data

- Authenticated User identity.
- Owner or Invited User relationship when applicable.
- Current access status.

---

## User Actions

- Review User identity and access relationship.
- Open Settings.
- End the authenticated session and return to Authentication.

---

## Navigation

Users arrive from Dashboard, Settings, or authenticated product navigation.

Users can continue to Settings, Dashboard, or Authentication after ending the session.

---

## Empty State

An authenticated Profile has no valid empty identity state. If no authenticated User exists, continue to Authentication.

---

## Loading State

Do not infer Owner or Invited User status while identity information is unavailable.

---

## Error State

Explain that Profile information is unavailable without exposing another User's identity. Allow retry, return to Dashboard, or end the session.

---

## Success State

The User can verify their identity, understand their access relationship, and deliberately continue to Settings or end the session.

---

## Business Rules

- Follow the Product RFC sections **Authentication** and **Business Rules**.
- PD-002, **Owner-First Access Policy** (Approved), governs Owner and Invited User relationships.
- A User may see only their own financial resources.

---

## Notes

Profile is about User identity and access context. Product-level preferences belong in Settings.
