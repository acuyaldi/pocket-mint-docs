# Pocket Mint Canonical Glossary

> Version: 1.0
> Status: Active
> Last Updated: 2026-07-14
> Scope: Canonical business terminology for all Pocket Mint repositories and documents
>
> Related documents:
> - [`product-rfc.md`](./product-rfc.md)
> - [`design-system.md`](./design-system.md)
> - [`decisions/`](./decisions/)

## Purpose

This glossary is the single source of truth for business terminology in Pocket Mint.

Every product document, frontend, backend, API, automation workflow, AI prompt, and future documentation MUST use these definitions consistently.

If any documentation conflicts with this glossary, the glossary wins until a Product Decision explicitly supersedes it. When a Product Decision changes a term, this glossary must be updated in the same change.

Terms whose definitions depend on a Product Decision that is still in **Draft** status are marked **Provisional**. Provisional definitions reflect the current recommendation and may change when the decision is approved.

---

# Terminology Rules

1. **One concept has exactly one canonical term.** Every business concept is named by exactly one entry in this glossary.
2. **Avoid synonyms unless explicitly listed.** If a synonym is not listed under a term's entry or in Naming Conventions, do not use it.
3. **UI labels may differ slightly, but internal documentation must use canonical terms.** A screen may shorten "Total Outstanding Debt" for space; the specification describing that screen must not.
4. **Product Decisions may supersede glossary entries.** An approved Product Decision is the only mechanism that changes a canonical definition.
5. **API names may differ for backward compatibility, but documentation must reference the canonical business term.** An API field name is an implementation detail, never a definition.
6. **Provisional terms must be flagged.** Any document relying on a Provisional term must note the governing Draft Product Decision.
7. **Prefer business language over implementation language.** This glossary defines what things mean to the user, not how they are stored or transported.

---

# Canonical Terms

## Net Worth

**Definition**
The user's complete financial position: Total Assets minus Total Outstanding Debt, evaluated at the same Reporting Cutoff. Net Worth represents only assets and debts tracked by Pocket Mint and may be negative.

**Why it exists**
Net Worth is the product's headline metric. It uses the conventional financial definition so the most prominent number is complete, comparable, and explainable without special interpretation. Approved by PD-001.

**Related Terms**
Total Assets, Total Outstanding Debt, Reporting Cutoff, Negative Net Worth, Canonical Calculation

**Not To Be Confused With**
Total Assets. An assets-only value must be labeled Total Assets and must never be presented as Net Worth.

**Used In**
Dashboard, Reports, Financial Summary, automation outputs

**Examples**
Total Assets of 10,000,000 and Total Outstanding Debt of 3,000,000 gives a Net Worth of 7,000,000.

## Total Assets

**Definition**
The combined Wallet Balance of all Asset Wallets tracked by Pocket Mint at a given Reporting Cutoff.

**Why it exists**
Users need a clear view of what they hold, separately from what they owe. Total Assets is a supporting figure for Net Worth and the accurate name for any assets-only value. Approved by PD-001.

**Related Terms**
Net Worth, Asset Wallet, Wallet Balance, Reporting Cutoff

**Not To Be Confused With**
Net Worth. Total Assets excludes debt entirely.

**Used In**
Dashboard, Wallets, Reports, Financial Summary

**Examples**
Cash 2,000,000 + Bank 7,000,000 + E-Wallet 1,000,000 = Total Assets 10,000,000.

## Total Outstanding Debt

**Definition**
The combined Outstanding Balance of all Debt Wallets tracked by Pocket Mint at a given Reporting Cutoff. An Installment obligation already represented in a Debt Wallet's Outstanding Balance must not be counted again.

**Why it exists**
Users need one number for everything they owe. The double-counting rule keeps Net Worth deterministic when Installments are front-loaded into Debt Wallets. Approved by PD-001.

**Related Terms**
Net Worth, Debt Wallet, Outstanding Balance, Installment

**Not To Be Confused With**
Outstanding Balance, which is the debt of a single wallet. Also not "Total Debt" or "Outstanding Amount" — see Naming Conventions.

**Used In**
Dashboard, Wallets, Reports, Financial Summary

**Examples**
Credit Card Outstanding Balance 2,000,000 + PayLater Outstanding Balance 1,000,000 = Total Outstanding Debt 3,000,000.

## Debt Wallet

**Definition**
A wallet whose type represents money owed to a provider: Credit Card or PayLater. Debt classification is derived by application logic from the wallet type.

**Why it exists**
Separating debts from assets is the foundation of Net Worth, Available Credit, and Debt Ratio. Users managing credit products need debts tracked as first-class wallets, not as negative afterthoughts.

**Related Terms**
Asset Wallet, Credit Card, PayLater, Outstanding Balance, Credit Limit

**Not To Be Confused With**
An Asset Wallet with a temporarily negative balance. Debt classification comes from wallet type, not the sign of the balance.

**Used In**
Wallets, Dashboard, Transactions, Installments, Reports

**Examples**
A PayLater wallet with a 5,000,000 Credit Limit and 1,500,000 Outstanding Balance is a Debt Wallet.

## Asset Wallet

**Definition**
A wallet whose type represents money the user holds: Cash, Bank, or E-Wallet.

**Why it exists**
Asset Wallets are the positive side of the user's financial position and the source of Total Assets. Multi-wallet consolidation across asset types is a core product capability.

**Related Terms**
Debt Wallet, Wallet Balance, Total Assets

**Not To Be Confused With**
Debt Wallet. Also not an investment account — Pocket Mint is not an investment platform.

**Used In**
Wallets, Dashboard, Transactions, Transfers, Reports

**Examples**
A Bank wallet holding 7,000,000 is an Asset Wallet contributing 7,000,000 to Total Assets.

## Wallet Balance

**Definition**
The current stored value of a single wallet. For Asset Wallets it is the money held; for Debt Wallets it is the debt position from which the Outstanding Balance is derived.

**Why it exists**
The Wallet Balance is the atomic unit every canonical financial figure aggregates from. Every balance change must come from a recorded Financial Event so the balance is always explainable.

**Related Terms**
Asset Wallet, Debt Wallet, Outstanding Balance, Ledger

**Not To Be Confused With**
Available Credit, which is a derived headroom figure for Debt Wallets, not a balance.

**Used In**
Wallets, Dashboard, Transactions, Financial Summary

**Examples**
Recording an Expense of 50,000 from a Cash wallet reduces that wallet's Wallet Balance by exactly 50,000.

## Available Credit

**Definition**
The remaining spending headroom of a Debt Wallet: Credit Limit minus Outstanding Balance.

**Why it exists**
Users of Credit Cards and PayLater need to see how much credit remains before reaching their limit, without doing mental arithmetic.

**Related Terms**
Credit Limit, Outstanding Balance, Debt Ratio, Debt Wallet

**Not To Be Confused With**
Wallet Balance. Available Credit is derived headroom, not money the user holds.

**Used In**
Wallets, Dashboard, Transactions (debt validation), Reports

**Examples**
Credit Limit 5,000,000 with Outstanding Balance 1,500,000 gives Available Credit of 3,500,000.

## Credit Limit

**Definition**
The maximum debt a Debt Wallet may carry, configured by the user to match the provider's limit.

**Provisional** — PD-005 (Draft) defines the Credit Limit as a hard boundary: debt-creating operations that would exceed it are rejected until the limit is raised.

**Why it exists**
The Credit Limit anchors Available Credit and Debt Ratio, and gives the product a deterministic rule for rejecting obligations the user's real credit product could not carry.

**Related Terms**
Available Credit, Debt Ratio, Outstanding Balance, Debt Wallet

**Not To Be Confused With**
A budget or spending target. The Credit Limit mirrors the provider's contractual limit.

**Used In**
Wallets, Transactions, Installments, Reports

**Examples**
With a Credit Limit of 5,000,000 and Outstanding Balance of 4,800,000, a new 500,000 debt Expense is rejected.

## Debt Ratio

**Definition**
The utilization of a Debt Wallet: Outstanding Balance divided by Credit Limit.

**Provisional** — PD-005 (Draft) defines the presentation thresholds: normal below 30%, warning from 30% to below 80%, danger at 80% or above, applied identically to all Debt Wallets.

**Why it exists**
A single ratio communicates debt risk at a glance and gives every screen the same, consistent risk signal.

**Related Terms**
Outstanding Balance, Credit Limit, Available Credit

**Not To Be Confused With**
A debt-to-income ratio or any personalized financial advice. Thresholds are product guidance only.

**Used In**
Wallets, Dashboard, Reports

**Examples**
Outstanding Balance 4,000,000 against Credit Limit 5,000,000 is a Debt Ratio of 80% — danger state.

## Outstanding Balance

**Definition**
The amount currently owed on a single Debt Wallet: the absolute value of that wallet's debt balance.

**Why it exists**
Debt balances need an unambiguous, always-positive figure that users can read directly, and that feeds Available Credit, Debt Ratio, and Total Outstanding Debt.

**Related Terms**
Debt Wallet, Total Outstanding Debt, Available Credit, Debt Ratio

**Not To Be Confused With**
Total Outstanding Debt (the sum across all Debt Wallets), or the Total Obligation of a single Installment.

**Used In**
Wallets, Dashboard, Installments, Reports

**Examples**
Creating an Installment with a Total Obligation of 1,200,000 on a PayLater wallet immediately raises that wallet's Outstanding Balance by 1,200,000.

## Installment

**Definition**
A purchase or loan repaid in fixed monthly payments over a set duration. On creation, the full Total Obligation is applied to the Debt Wallet once (front-loaded), and the Outstanding Balance immediately reflects the entire debt.

**Why it exists**
Installment purchases via PayLater and Credit Card are central to the target user's financial life. Front-loading makes the true debt visible immediately instead of drip-feeding it month by month.

**Related Terms**
Installment Schedule, Installment Payment, Total Obligation, Principal, Interest, Administrative Fee, Monthly Payment

**Not To Be Confused With**
Installment Payment (a single monthly event within an Installment), or an ordinary Expense.

**Used In**
Installments, Wallets, Dashboard, Reports

**Examples**
A 12-month phone purchase of 12,000,000 creates one Installment; the Debt Wallet's Outstanding Balance rises by the full Total Obligation on day one.

## Installment Schedule

**Definition**
The ordered sequence of Installment Payments an Installment will produce: one per term, from the first Installment Due Date to the final one, after which the Installment is settled.

**Provisional** — PD-003 (Draft) defines the lifecycle: terms advance automatically by due month in the Reporting Timezone, missed terms are caught up exactly once, and payoff or cancellation require explicit user action.

**Why it exists**
Users need to see where they are in a repayment plan — how many terms are done, what is due next, and when the debt ends.

**Related Terms**
Installment, Installment Payment, Installment Due Date, Reporting Timezone

**Not To Be Confused With**
The Ledger. The schedule is a plan; the Ledger records what actually happened.

**Used In**
Installments, Dashboard, Reports, Automation

**Examples**
A 12-month Installment started in January has an Installment Schedule of 12 Installment Payments, January through December.

## Installment Payment

**Definition**
A single monthly term of an Installment, recorded as a reporting entry when its term becomes due. An Installment Payment never deducts the Wallet Balance again — the full obligation was already deducted at creation.

**Provisional** — governed by PD-003 (Draft).

**Why it exists**
Monthly reports must show the monthly repayment reality even though the debt was front-loaded. One reporting entry per due term keeps Reporting Periods accurate without double-deducting.

**Related Terms**
Installment, Installment Schedule, Monthly Payment, Reporting Period

**Not To Be Confused With**
The initial front-loaded deduction at Installment creation, or a manual Expense the user records separately.

**Used In**
Installments, Reports, Reporting Periods

**Examples**
In month 3 of a 12-month plan, the third Installment Payment appears in that month's report; the Wallet Balance is unchanged by it.

## Installment Due Date

**Definition**
The calendar date on which an Installment Payment's term becomes due, evaluated in the Reporting Timezone.

**Provisional** — governed by PD-003 (Draft).

**Why it exists**
Due dates drive term progression, upcoming and overdue states, and the timing of monthly reporting entries.

**Related Terms**
Installment Schedule, Installment Payment, Reporting Timezone

**Not To Be Confused With**
The Reporting Cutoff, which bounds a metric evaluation, or the date the user actually paid the provider.

**Used In**
Installments, Dashboard, Reports, Automation

**Examples**
An Installment started on 15 January has its second Installment Due Date on 15 February in the Reporting Timezone.

## Principal

**Definition**
The base amount borrowed or financed by an Installment, before Interest and Administrative Fees.

**Why it exists**
Principal is the input every Installment calculation starts from, in both forward and reverse calculation modes.

**Related Terms**
Interest, Administrative Fee, Total Obligation, Monthly Payment

**Not To Be Confused With**
Total Obligation, which includes Interest and Administrative Fees.

**Used In**
Installments, Installment calculator, Reports

**Examples**
Financing a 10,000,000 purchase over 12 months has a Principal of 10,000,000 regardless of Interest.

## Interest

**Definition**
The cost charged by a provider for an Installment, expressed as a rate applied to the Principal and included in the Total Obligation.

**Why it exists**
Interest is what makes an Installment cost more than its Principal; showing it explicitly keeps every financial number explainable.

**Related Terms**
Principal, Administrative Fee, Total Obligation, Monthly Payment

**Not To Be Confused With**
Administrative Fee, which is a separate declared charge, not a rate-based financing cost.

**Used In**
Installments, Installment calculator, Reports

**Examples**
A 10,000,000 Principal at 12% total Interest adds 1,200,000 to the Total Obligation.

## Administrative Fee

**Definition**
A declared provider charge on an Installment, either a flat amount or a percentage of Principal, calculated once at creation and included in the Total Obligation.

**Provisional** — PD-004 (Draft) governs fee types, basis, and inclusion in the Total Obligation.

**Why it exists**
Real installment products charge administrative fees. Including the declared fee in the Total Obligation keeps the preview, the stored obligation, the Outstanding Balance, and reports in agreement.

**Related Terms**
Principal, Interest, Total Obligation, Installment

**Not To Be Confused With**
Interest. Also not "Admin Fee" or "Service Fee" — see Naming Conventions.

**Used In**
Installments, Installment calculator, Reports

**Examples**
A 1% Administrative Fee on a 10,000,000 Principal adds 100,000 to the Total Obligation, once, at creation.

## Total Obligation

**Definition**
The complete amount an Installment commits the user to repay: Principal plus Interest plus Administrative Fee. This is the amount front-loaded into the Debt Wallet's Outstanding Balance at creation.

**Provisional** — fee inclusion is governed by PD-004 (Draft).

**Why it exists**
Users must see and be held to exactly one number for what an Installment truly costs. The saved obligation must match what the user reviewed before creating it.

**Related Terms**
Principal, Interest, Administrative Fee, Monthly Payment, Outstanding Balance

**Not To Be Confused With**
Total Outstanding Debt (a portfolio-level figure), or Principal (the pre-cost base amount).

**Used In**
Installments, Wallets, Reports

**Examples**
Principal 10,000,000 + Interest 1,200,000 + Administrative Fee 100,000 = Total Obligation 11,300,000.

## Monthly Payment

**Definition**
The fixed amount due for each term of an Installment, derived from the Total Obligation and the duration. In reverse calculation, the Monthly Payment is an input from which the plan is derived.

**Why it exists**
The Monthly Payment is what the user actually experiences each month and is the figure recorded by each Installment Payment.

**Related Terms**
Total Obligation, Installment Payment, Installment Schedule

**Not To Be Confused With**
Installment Payment, which is the recorded monthly event; the Monthly Payment is its amount.

**Used In**
Installments, Installment calculator, Reports

**Examples**
A Total Obligation of 11,300,000 over 12 months gives a Monthly Payment of roughly 941,667.

## Transfer

**Definition**
The movement of money between two different wallets owned by the same user. A Transfer decreases the source wallet and increases the destination wallet atomically, and cannot use the same wallet for both sides.

**Provisional** — PD-007 (Draft) establishes that a Transfer is represented as a single Transaction with a source and destination wallet; no separate Transfer entity exists in the product contract.

**Why it exists**
Users constantly move money between their own wallets. Treating a Transfer as one atomic event keeps both balances consistent and prevents it from being mistaken for Income or Expense.

**Related Terms**
Wallet Balance, Ledger, Financial Event

**Not To Be Confused With**
Income or Expense. A Transfer changes where money sits, not how much the user has.

**Used In**
Transactions, Wallets, Reports

**Examples**
Moving 1,000,000 from a Bank wallet to an E-Wallet is one Transfer; Total Assets is unchanged.

## Category

**Definition**
A user-owned label that classifies a transaction by purpose (for example food, transport, salary). Every Category belongs to exactly one User.

**Why it exists**
Categories power spending analysis, reporting breakdowns, and future automation such as AI transaction extraction.

**Related Terms**
Income, Expense, Reports, User

**Not To Be Confused With**
Wallet type. A Category describes what a transaction was for; a wallet type describes where money lives.

**Used In**
Transactions, Reports, Dashboard, Automation

**Examples**
An Expense of 50,000 tagged with the "Food" Category appears under Food in the Reporting Period's breakdown.

## Income

**Definition**
A Financial Event that increases the user's financial position by adding money to a wallet from outside the tracked wallets.

**Why it exists**
Income is one half of cash-flow reporting and the counterpart to Expense in every Reporting Period summary.

**Related Terms**
Expense, Transfer, Category, Reporting Period

**Not To Be Confused With**
A Transfer, which moves money between the user's own wallets and is not Income.

**Used In**
Transactions, Dashboard, Reports

**Examples**
A monthly salary of 8,000,000 recorded into a Bank wallet is Income.

## Expense

**Definition**
A Financial Event that decreases the user's financial position: money leaving an Asset Wallet, or new debt added to a Debt Wallet.

**Why it exists**
Expenses are the primary thing users track daily; fast expense entry is a core product priority.

**Related Terms**
Income, Transfer, Category, Outstanding Balance

**Not To Be Confused With**
A Transfer, or an Installment Payment (which records repayment of already-counted debt).

**Used In**
Transactions, Dashboard, Reports

**Examples**
Paying 50,000 for lunch from a Cash wallet is an Expense; charging 500,000 to a Credit Card is an Expense that raises the Outstanding Balance.

## Financial Event

**Definition**
Any recorded occurrence that affects the user's financial position or its reporting: Income, Expense, Transfer, Installment creation, or an Installment Payment.

**Why it exists**
Pocket Mint requires that every financial number be explainable. Financial Event is the umbrella term for the recorded facts that explanations trace back to.

**Related Terms**
Ledger, Income, Expense, Transfer, Installment Payment

**Not To Be Confused With**
A Derived Metric, which is calculated from Financial Events rather than recorded.

**Used In**
Ledger, Reports, Automation, audit discussions

**Examples**
"Every Wallet Balance is the result of its Financial Events" — no balance may change without one.

## Reporting Cutoff

**Definition**
The single point in time at which a financial metric is evaluated. All components of one metric — for example the Total Assets and Total Outstanding Debt inside Net Worth — must be evaluated at the same Reporting Cutoff.

**Why it exists**
Comparing assets from one moment with debts from another produces a meaningless number. The cutoff rule makes every canonical metric deterministic and reproducible. Established by PD-001.

**Related Terms**
Net Worth, Reporting Period, Reporting Timezone

**Not To Be Confused With**
Reporting Period, which is a span of time; the cutoff is an instant.

**Used In**
Dashboard, Reports, Financial Summary, Automation

**Examples**
A Net Worth figure dated 31 January uses Total Assets and Total Outstanding Debt both as of 31 January.

## Reporting Period

**Definition**
A bounded span of time — typically a calendar month in the Reporting Timezone — over which Income, Expenses, Installment Payments, and other Financial Events are aggregated for reporting.

**Provisional** — reporting scope is governed by PD-006 (Draft).

**Why it exists**
Users understand their finances month by month. Reporting Periods give trends, comparisons, and monthly summaries a consistent boundary.

**Related Terms**
Reporting Cutoff, Reporting Timezone, Report, Trend

**Not To Be Confused With**
Reporting Cutoff (an instant, not a span).

**Used In**
Reports, Dashboard, Installments, Automation

**Examples**
The February Reporting Period contains all Financial Events dated in February in the Reporting Timezone.

## Reporting Timezone

**Definition**
The Owner's configured timezone, used to determine calendar boundaries for Reporting Periods, Installment Due Dates, and scheduled processing.

**Provisional** — governed by PD-003 (Draft).

**Why it exists**
Self-hosted deployments run anywhere. Without one declared timezone, "which month does this belong to" becomes nondeterministic — violating the rule that calculations are deterministic.

**Related Terms**
Reporting Period, Reporting Cutoff, Installment Due Date, Owner

**Not To Be Confused With**
The server's system timezone or a browser's local timezone.

**Used In**
Reports, Installments, Automation

**Examples**
An event at 23:30 on 31 January in the Reporting Timezone belongs to January, regardless of where the server runs.

## Owner

**Definition**
The single active User with installation-level authority. The first account to complete initialization becomes the Owner. The Owner controls installation admission and installation-level settings but cannot view, modify, export, recover, or impersonate another User's financial resources.

**Why it exists**
Pocket Mint is a privacy-first, self-hosted product. Owner-first access is what makes a deployment closed and private by default.

**Related Terms**
User, Invited User, Reporting Timezone

**Not To Be Confused With**
User, the general term. Every Owner is a User; not every User is the Owner. Also distinct from resource-level ownership ("every resource belongs to exactly one user").

**Used In**
Authentication, onboarding, settings, documentation about access policy

**Examples**
The first person to initialize a Pocket Mint installation becomes its Owner; later accounts require the Owner's approval through an invitation or allowlist.

## User

**Definition**
An authenticated human account admitted to a Pocket Mint installation. Every financial resource belongs to exactly one User, and every financial operation is scoped to the requesting User. A User has no installation-level authority unless that User is the Owner.

**Why it exists**
Per-user ownership is the product's core isolation rule: no User may ever see or affect another User's financial data.

**Related Terms**
Owner, Invited User

**Not To Be Confused With**
Owner (a specific privileged User). "User" alone never implies installation-level control.

**Used In**
All product surfaces, business rules, API documentation

**Examples**
A wallet created by one User is invisible to every other User in the same installation.

## Invited User

**Definition**
A non-Owner User admitted to an installation through the Owner's explicit approval using an invitation or allowlist, rather than through public registration. A pending invitation or allowlist entry is not a User and grants no financial access.

**Why it exists**
Pocket Mint supports deliberate multi-user use (for example a household) without opening public signup.

**Related Terms**
Owner, User

**Not To Be Confused With**
A publicly self-registered account — public registration is not permitted.

**Used In**
Authentication, onboarding, access-policy documentation

**Examples**
The Owner invites a partner; the partner becomes an Invited User with their own fully isolated financial data.

## Automation

**Definition**
Any capability where Pocket Mint acts on the user's behalf without a manual step: scheduled installment progression, webhooks, external integrations (such as n8n), message parsing, and AI transaction extraction. Automation assists the user and never replaces user control.

**Why it exists**
Minimizing manual bookkeeping is a stated product goal, bounded by the principle that the user always stays in control.

**Related Terms**
Installment Schedule, Financial Event, Ledger

**Not To Be Confused With**
Canonical Calculation. Calculations are deterministic rules; Automation is scheduled or triggered behavior that applies them.

**Used In**
Roadmap, Installments lifecycle, integrations documentation

**Examples**
Monthly Installment progression is Automation; cancelling an Installment is a user decision Automation must never make.

## Ledger

**Definition**
The complete, ordered record of a User's Financial Events. The Ledger is the factual basis from which every balance, metric, Report, and Trend is derived.

**Why it exists**
"Every financial number must be explainable" requires one authoritative record of what happened. Reports may only present ledger-derived values — never synthetic or placeholder figures.

**Related Terms**
Financial Event, Historical Record, Derived Metric, Report

**Not To Be Confused With**
A Report (a presentation of ledger data) or the Installment Schedule (a plan, not a record).

**Used In**
Reports, Dashboard, audit and reconciliation discussions

**Examples**
A Trend shown in Reports must trace back to Ledger entries; if the Ledger has no history for a period, the Trend shows a gap, not an invented line.

## Historical Record

**Definition**
Ledger data belonging to past Reporting Periods, preserved unchanged so that past Reports and metrics remain reproducible.

**Why it exists**
Trends, comparisons, and audits are only trustworthy if the past does not silently change. Corrections are recorded as new Financial Events, keeping history explainable.

**Related Terms**
Ledger, Reporting Period, Report, Trend

**Not To Be Confused With**
Archived data (a Reserved Word for items hidden from active use). Historical Records remain part of reporting.

**Used In**
Reports, Trends, audit discussions

**Examples**
January's Historical Record still supports January's Report in December, even after wallets have changed.

## PayLater

**Definition**
A Debt Wallet type representing buy-now-pay-later credit from a provider. PayLater wallets carry a Credit Limit, an Outstanding Balance, and commonly host Installments.

**Why it exists**
PayLater services are a major debt instrument for the target audience and behave differently enough from cash to need first-class debt tracking.

**Related Terms**
Debt Wallet, Credit Card, Installment, Credit Limit

**Not To Be Confused With**
Credit Card. Both are Debt Wallets, but they are distinct wallet types. Visually, PayLater debt uses debt semantics, not installment-timing semantics (PD-005, Draft).

**Used In**
Wallets, Transactions, Installments, Reports

**Examples**
A 12-month phone Installment charged to a PayLater wallet raises that wallet's Outstanding Balance by the Total Obligation.

## Credit Card

**Definition**
A Debt Wallet type representing revolving credit-card debt, with a Credit Limit, Outstanding Balance, Available Credit, and Debt Ratio.

**Why it exists**
Credit-card tracking is a core capability; users need card debt reflected in Total Outstanding Debt and Net Worth, not hidden outside the system.

**Related Terms**
Debt Wallet, PayLater, Credit Limit, Available Credit

**Not To Be Confused With**
PayLater (a distinct Debt Wallet type), or a Bank Asset Wallet at the same institution.

**Used In**
Wallets, Transactions, Installments, Reports

**Examples**
A Credit Card with Credit Limit 10,000,000 and Outstanding Balance 2,500,000 shows Available Credit of 7,500,000.

## Dashboard

**Definition**
The primary current-state overview surface: Net Worth as the dominant element, wallets, recent transactions, monthly summary, and debt/installment status, with shortcuts into deeper surfaces.

**Provisional** — the Dashboard/Reports boundary is governed by PD-006 (Draft): the Dashboard shows current status; historical analysis belongs to Reports.

**Why it exists**
Users need one calm, immediate answer to "how are my finances right now" without navigating.

**Related Terms**
Report, Financial Summary, Net Worth, Trend

**Not To Be Confused With**
Reports. The Dashboard is current-state monitoring, not historical analysis.

**Used In**
Product navigation, screen specifications

**Examples**
The Dashboard shows today's Net Worth; comparing this quarter to last quarter happens in Reports.

## Report

**Definition**
A dedicated presentation of ledger-derived financial data over Reporting Periods: period summaries, Trends, category breakdowns, cash flow, debt, and installment reporting. Every figure in a Report must be ledger-derived or clearly sourced.

**Provisional** — governed by PD-006 (Draft).

**Why it exists**
Financial reporting is a core product capability supporting long-term planning, and needs its own surface so the Dashboard stays a concise overview.

**Related Terms**
Reporting Period, Ledger, Trend, Dashboard, Historical Record

**Not To Be Confused With**
The Dashboard, or a raw data export.

**Used In**
Reports surface, automation outputs, documentation

**Examples**
A monthly cash-flow Report showing Income versus Expenses per Reporting Period across six months.

## Financial Summary

**Definition**
The consolidated set of canonical headline figures for a User at a Reporting Cutoff: Net Worth, Total Assets, Total Outstanding Debt, and their supporting components, all produced by Canonical Calculations.

**Why it exists**
Every surface — Dashboard, Reports, Automation — must present the same numbers from one authoritative source, never recomputed independently per screen.

**Related Terms**
Net Worth, Total Assets, Total Outstanding Debt, Canonical Calculation, Reporting Cutoff

**Not To Be Confused With**
A Report (which covers periods and history); the Financial Summary is a snapshot of headline metrics.

**Used In**
Dashboard, Reports, Automation, API documentation (as a business concept)

**Examples**
The Dashboard hero and an automated monthly message both present the same Financial Summary for the same Reporting Cutoff.

## Trend

**Definition**
The change of a metric across consecutive Reporting Periods, derived exclusively from the Ledger and Historical Records. Where history is missing, a Trend shows the gap explicitly.

**Why it exists**
Users judge progress by direction over time. Trends are only trustworthy if every point is a ledger-derived fact — synthetic or placeholder trends are prohibited.

**Related Terms**
Reporting Period, Ledger, Historical Record, Report, Derived Metric

**Not To Be Confused With**
A projection or forecast. A Trend describes recorded history only.

**Used In**
Reports, Dashboard (summary-level), Automation

**Examples**
A Net Worth Trend plots the ledger-derived Net Worth at each month-end Reporting Cutoff for the last six months.

## Derived Metric

**Definition**
Any value calculated from recorded data rather than entered by the user: Net Worth, Total Assets, Total Outstanding Debt, Available Credit, Debt Ratio, Trends, and period aggregates.

**Why it exists**
Distinguishing derived values from recorded facts enforces the explainability rule: a Derived Metric must always be traceable to the Financial Events and formula that produced it.

**Related Terms**
Canonical Calculation, Financial Event, Ledger, Trend

**Not To Be Confused With**
A Financial Event (recorded fact). Derived Metrics are never stored as independent truths that can drift from the Ledger.

**Used In**
Reports, Dashboard, Financial Summary, documentation

**Examples**
Available Credit is a Derived Metric: it exists only as Credit Limit minus Outstanding Balance.

## Canonical Calculation

**Definition**
The single authoritative business rule that produces a Derived Metric. Each metric has exactly one Canonical Calculation, owned by the backend, deterministic, and used by every surface and automation.

**Why it exists**
Business rules belong in the backend and calculations must be deterministic. One formula per metric prevents different screens from showing different "truths" for the same number.

**Related Terms**
Derived Metric, Financial Summary, Net Worth, Debt Ratio

**Not To Be Confused With**
A display convenience (formatting, rounding for presentation), which must never alter the calculated value.

**Used In**
Product rules, backend documentation, test requirements

**Examples**
Net Worth = Total Assets − Total Outstanding Debt is a Canonical Calculation; no surface may substitute its own variant.

## Negative Net Worth

**Definition**
A valid Net Worth state where Total Outstanding Debt exceeds Total Assets. Pocket Mint displays Negative Net Worth honestly; it is never clamped to zero or hidden.

**Why it exists**
Users with significant PayLater or Credit Card debt may genuinely owe more than they hold. Hiding that would contradict the product's clarity and explainability principles. Established by PD-001.

**Related Terms**
Net Worth, Total Assets, Total Outstanding Debt

**Not To Be Confused With**
An error state or a negative Wallet Balance on a single wallet.

**Used In**
Dashboard, Reports, Financial Summary, design specifications (negative-state presentation)

**Examples**
Total Assets 3,000,000 with Total Outstanding Debt 5,000,000 gives a Negative Net Worth of −2,000,000.

---

# Naming Conventions

Canonical names are used exactly as written in this glossary. Alternatives are prohibited in documentation unless a Product Decision explicitly allows them.

**Use:** Net Worth
**Avoid:** Asset Position, Wealth, Total Balance — and never label an assets-only figure as Net Worth.

**Use:** Total Assets
**Avoid:** Asset Total, Assets, Total Wallets.

**Use:** Total Outstanding Debt
**Avoid:** Total Debt, Debt Total, Outstanding, Outstanding Amount.

**Use:** Outstanding Balance
**Avoid:** Outstanding (alone), Debt Balance, Amount Owed.

**Use:** Administrative Fee
**Avoid:** Admin Fee, Service Fee, Processing Fee.

**Use:** Total Obligation
**Avoid:** Grand Total, Total Liability, Full Amount. (API names such as `grandTotal` may persist for compatibility; documentation must say Total Obligation.)

**Use:** Available Credit
**Avoid:** Remaining Credit, Credit Left, Headroom.

**Use:** Debt Ratio
**Avoid:** Utilization, Credit Utilization, Usage Percentage — unless a Product Decision approves them as UI labels.

**Use:** Installment
**Avoid:** Cicilan in documentation (UI and routes may localize; documentation uses Installment), Loan, Plan.

**Use:** Installment Payment
**Avoid:** Monthly Installment, Term Payment.

**Use:** Transfer
**Avoid:** Move, Wallet-to-Wallet Transaction.

**Use:** Debt Wallet / Asset Wallet
**Avoid:** Liability Wallet, Credit Wallet, Money Wallet.

**Use:** Owner
**Avoid:** Admin, Administrator, Superuser.

**Use:** Reporting Timezone
**Avoid:** User Timezone, System Timezone.

---

# Reserved Words

These words have special meaning inside Pocket Mint and must not be used casually or with alternative meanings.

- **Outstanding** — Only in Outstanding Balance and Total Outstanding Debt. Never alone as a noun.
- **Settlement / Settled** — The completion of an Installment after its final term is processed. Not a synonym for "paid" in general.
- **Scheduled** — A future Installment Payment or Automation run that is planned but has not occurred. Scheduled is not proof of payment.
- **Paid** — A term or obligation the user has actually satisfied. Distinct from Scheduled and Projected. (Provisional — lifecycle states governed by PD-003, Draft.)
- **Confirmed** — Explicitly acknowledged by the user. Automation may propose; only the user Confirms.
- **Projected** — A forward-looking estimate, never a ledger fact. Projected values must always be labeled as such and must never appear in a Trend.
- **Archived** — Hidden from active use but retained. Archived is never deletion.
- **Deleted** — Removed by explicit user action under the product's deletion rules. Financial consistency must be preserved; partial deletion is prohibited.
- **Imported** — A Financial Event created by Automation or an external source rather than manual entry. Imported events follow every rule manual events follow.
- **Provisional** — A glossary term whose definition depends on a Draft Product Decision.
- **Canonical** — The single authoritative definition, term, or calculation. Nothing may be called canonical unless this glossary or a Product Decision makes it so.

---

# Future Terms

Approved terminology that does not yet have a full entry is added here first, then promoted to Canonical Terms.

_(No future terms recorded yet.)_
