# Dashboard Specification

Version: 1.0

Status: Draft

Owner: Pocket Mint

Category: Product Design Specification

Last Updated: 2026

---

# Dependencies

This specification depends on:

- DESIGN.md
- design-principles.md
- layout-spacing.md
- app-shell.md
- stitch-editing-rules.md

Related documents:

- dashboard-composition.md

---

# 1. Purpose

The Dashboard is the primary workspace of Pocket Mint.

It represents the user's current financial position and serves as the starting point for all financial decisions.

Unlike analytics dashboards, the Dashboard is not intended to provide deep analysis, historical reports, or business intelligence.

Its primary responsibility is to answer the user's most important financial questions immediately after opening the application.

The Dashboard must feel:

- calm
- trustworthy
- professional
- information-first
- privacy-first

It should encourage clarity rather than exploration.

---

# 2. Product Role

Within Pocket Mint, the Dashboard functions as the user's financial home.

It summarizes the current state of the user's finances rather than exposing every available piece of information.

The Dashboard should reduce cognitive load by presenting only the information that supports immediate awareness and decision making.

It is intentionally designed to answer questions before encouraging deeper exploration.

Users who require more detail should navigate to Wallets, Transactions, or Installments.

The Dashboard should never become a reporting page.

---

# 3. Primary User Goals

When opening the Dashboard, users are expected to achieve the following goals within a few seconds.

## Goal 1

Understand current financial position.

## Goal 2

Identify urgent financial items.

## Goal 3

Determine whether immediate action is required.

## Goal 4

Know where money currently exists.

## Goal 5

Understand recent financial activity.

The Dashboard is considered successful when users can achieve all five goals without opening another page.

---

# 4. Primary User Questions

The Dashboard must answer these questions immediately.

## Financial Position

What is my financial position today?

## Attention

Is there anything requiring my attention?

## Action

What is the next thing I probably want to do?

## Assets

Where is my money currently stored?

## Activity

What happened recently?

Every section on the Dashboard should contribute to answering one or more of these questions.

If a section does not answer one of these questions, it should not exist on the Dashboard.

---

# 5. Dashboard Philosophy

Pocket Mint intentionally avoids becoming an analytics application.

The Dashboard should never resemble:

- business intelligence software
- stock trading software
- crypto exchanges
- marketing dashboards
- expense tracker summaries

Instead, it should resemble a professional financial workspace.

Information should be actionable rather than analytical.

Users should feel informed, not overwhelmed.

---

# 6. Information Priority

Information must appear in the following order.

1. Financial Position

2. Needs Attention

3. Quick Actions

4. Wallet Overview

5. Current Period Summary

6. Recent Activity

This hierarchy is fixed.

Do not reorder sections without an explicit product decision.

---

# 7. Information Hierarchy

## Level 1

Financial Position

Primary financial summary.

Largest visual emphasis.

## Level 2

Needs Attention

Items requiring immediate awareness.

## Level 3

Quick Actions

Most common user actions.

## Level 4

Wallet Overview

Current location of assets and liabilities.

## Level 5

Current Period Summary

High-level income and expense overview.

## Level 6

Recent Activity

Chronological transaction history.

Information importance decreases from top to bottom.

---

# 8. Navigation Context

The Dashboard belongs to the authenticated application.

Primary navigation:

- Dashboard
- Dompet
- Transaksi
- Cicilan
- Akun

The Dashboard is always the default destination after authentication.

---

# 9. Page Identity

Page title:

Dashboard

Do not use:

- Welcome
- Hello
- Good Morning
- Personalized greetings

The Dashboard should immediately communicate financial context.

Supporting information may include:

- Reporting Cutoff
- Last Updated
- Data Timestamp

Never replace the page title with marketing copy.

---

# 10. Primary Sections

The Dashboard consists of six primary sections.

## Financial Position

Purpose:

Provide a complete snapshot of the user's financial standing.

Displayed information:

- Net Worth
- Total Assets
- Total Outstanding Debt
- Reporting Cutoff

Primary importance:

Highest

---

## Needs Attention

Purpose:

Highlight items requiring immediate awareness.

Examples:

- Upcoming installment
- Overdue installment
- Negative wallet balance
- Financial warning

Primary importance:

High

---

## Quick Actions

Purpose:

Reduce interaction cost for common financial tasks.

Examples:

- Add Transaction
- Transfer
- Scan Receipt
- More Actions

Primary importance:

Medium

---

## Wallet Overview

Purpose:

Provide a quick overview of current asset locations.

Examples:

- Cash Wallet
- Bank Account
- Credit Card
- Digital Wallet
- Investment Wallet

Primary importance:

Medium

---

## Current Period Summary

Purpose:

Provide a lightweight overview of the current reporting period.

Displayed metrics:

- Income
- Expense

This section is informational only.

It should never become an analytics dashboard.

---

## Recent Activity

Purpose:

Display the latest financial events.

Items should be chronological.

Users should immediately understand:

- what happened
- where it happened
- when it happened
- financial impact

Only recent items belong here.

Historical exploration belongs to the Transactions page.

---

# 11. Data Freshness

Financial information shown on the Dashboard should represent the latest synchronized data.

Display:

- Reporting Cutoff
or
- Last Updated

Users should understand how recent the information is.

Never imply real-time synchronization if it does not exist.

---

# 12. Content Strategy

The Dashboard prioritizes relevance over completeness.

Only surface information that helps users answer immediate financial questions.

Avoid displaying information solely because it is available.

Every displayed metric should have a clear purpose.

If a metric does not influence user understanding or decision making, it should not appear on the Dashboard.

---

# 13. Progressive Disclosure

The Dashboard provides summaries.

Detailed exploration belongs to dedicated pages.

Examples:

Wallet Overview

↓

Dompet page

Recent Activity

↓

Transactions page

Installment Warning

↓

Installments page

The Dashboard should encourage navigation without replacing deeper workflows.

---

# 14. Functional Boundaries

The Dashboard must not become:

- a reporting center
- a budgeting tool
- an investment analysis tool
- a statistics dashboard
- a chart gallery

Historical analysis belongs to future reporting features, not the Dashboard.

---

# 15. Dashboard Principles

Every design decision should reinforce the following principles.

Information first.

Action second.

Analysis later.

The Dashboard should provide confidence, not curiosity.

Users should leave the Dashboard with a clear understanding of their financial situation before navigating elsewhere.

---

# 16. Financial Position Specification

## Purpose

The Financial Position section is the primary information block of the Dashboard.

It represents the user's financial condition at the current reporting cutoff.

This section should immediately answer:

> "How am I doing financially today?"

This is the visual anchor of the entire page.

---

## Priority

Highest

This section should receive the greatest visual emphasis.

No other dashboard section should compete with its importance.

---

## Required Information

The Financial Position section must display:

- Net Worth
- Total Assets
- Total Outstanding Debt
- Reporting Cutoff or Last Updated

Optional:

- Currency indicator (when multiple currencies are supported)

---

## Net Worth

Definition:

Net Worth = Total Assets − Total Outstanding Debt

Requirements:

- Largest number on the page
- Always displayed using tabular figures
- Uses semantic financial formatting
- Never abbreviated unless required by responsive constraints

---

## Total Assets

Definition:

Sum of all positive financial holdings.

Examples:

- Cash
- Bank Accounts
- Digital Wallets
- Investments
- Savings

Should always use positive semantic styling.

---

## Total Outstanding Debt

Definition:

Total remaining liabilities.

Examples:

- Credit Cards
- Loans
- Installments
- Mortgage

Should always use warning semantic styling.

Never display debt as a positive financial indicator.

---

## Reporting Context

Display:

- Reporting Cutoff
or
- Last Updated

Purpose:

Users must understand the freshness of the displayed financial information.

---

## Functional Rules

Do not display:

- charts
- historical trends
- month-over-month comparison
- performance indicators
- investment performance

This section represents a snapshot, not an analysis.

---

# 17. Needs Attention Specification

## Purpose

Surface financial items that require immediate awareness.

This section exists to reduce the chance that users miss important financial obligations.

---

## Priority

High

Only Financial Position has higher priority.

---

## Display Rules

Display only actionable items.

Examples:

Upcoming Installment

Overdue Installment

Negative Wallet Balance

Payment Reminder

Financial Warning

---

## Ordering Rules

Highest urgency first.

Recommended order:

1. Overdue

2. Due Today

3. Due Soon

4. Financial Warning

5. Informational Reminder

---

## Item Structure

Each attention item should contain:

- Status Icon
- Title
- Short Description
- Due Information
- Optional CTA

---

## Maximum Items

Desktop:

3 visible items

Mobile:

2 visible items

Additional items should be accessed through:

"Lihat Semua"

---

## Empty State

When there are no urgent items:

Display a positive confirmation.

Example:

"Tidak ada hal yang memerlukan perhatian."

Do not fabricate reminders.

---

# 18. Quick Actions Specification

## Purpose

Reduce friction for the most common financial tasks.

Quick Actions should support immediate interaction.

---

## Priority

Medium

Actions should never visually overpower Financial Position.

---

## Recommended Actions

Primary:

- Add Transaction

Secondary:

- Transfer

Optional:

- Scan Receipt

Overflow:

- More

---

## Interaction Rules

Quick Actions should always be immediately accessible.

Do not hide primary actions inside menus.

---

## Design Rules

Quick Actions should prioritize recognition over density.

Icons must remain readable.

Touch targets must follow accessibility standards.

---

# 19. Wallet Overview Specification

## Purpose

Provide a high-level overview of where financial assets and liabilities are currently stored.

This section answers:

"Where is my money?"

---

## Priority

Medium

---

## Display Rules

Display only summary information.

Each wallet should contain:

- Wallet Name
- Wallet Type
- Current Balance

Optional:

- Institution
- Wallet Icon

---

## Wallet Ordering

Recommended:

1. Primary Cash Wallet

2. Bank Accounts

3. Digital Wallets

4. Investments

5. Credit Cards

Negative balances should remain visible.

Do not hide liabilities.

---

## Maximum Visible Wallets

Desktop:

4–6

Mobile:

2–3

Additional wallets should be accessible through:

"Lihat Semua"

---

## Empty State

If no wallet exists:

Encourage wallet creation.

Do not generate placeholder balances.

---

# 20. Current Period Summary Specification

## Purpose

Provide a lightweight overview of the current reporting period.

This section is intentionally minimal.

---

## Display Information

Income

Expense

Optional:

Reporting Period

---

## Functional Rules

Do not display:

- trends
- graphs
- analytics
- comparisons
- forecasting

This section communicates totals only.

---

## Empty State

Display:

"No transactions recorded for this period."

---

# 21. Recent Activity Specification

## Purpose

Display the latest financial events.

Users should immediately understand recent financial movement.

---

## Display Rules

Each activity item should contain:

- Transaction Title
- Wallet
- Date
- Amount

Optional:

Category

---

## Ordering

Newest first.

---

## Maximum Items

Desktop:

5

Mobile:

4

Older transactions should be viewed through the Transactions page.

---

## Amount Formatting

Income:

Positive semantic styling.

Expense:

Negative semantic styling.

Transfers:

Neutral semantic styling.

---

## Empty State

Display:

"No recent activity."

Do not fabricate transactions.

---

# 22. Primary Actions

The Dashboard supports only lightweight actions.

Examples:

- Add Transaction
- Open Wallet
- View Transactions
- Open Installment
- View Wallet List

The Dashboard should never become a transaction editor.

---

# 23. Navigation Behaviour

The Dashboard serves as the application's home.

Users may navigate to:

- Wallets
- Transactions
- Installments
- Account

Each section should naturally lead to its dedicated page.

---

# 24. Cross-Page Relationships

Dashboard summarizes.

Dedicated pages provide detail.

Relationship mapping:

Financial Position

↓

Wallets

Needs Attention

↓

Installments

Recent Activity

↓

Transactions

The Dashboard should never duplicate the functionality of these pages.

---

# 25. Business Rules

The Dashboard must never display fabricated information.

Never generate:

- fake charts
- fake analytics
- estimated balances
- projected income
- predicted spending
- synthetic financial insights

Every displayed value must originate from actual user financial data.

If data is unavailable, display an appropriate empty state instead.

---

# 26. Dashboard States

The Dashboard should support multiple operational states.

---

## Normal State

All dashboard sections are available.

The user can immediately understand:

- financial position
- attention items
- wallets
- current period summary
- recent activity

This is the default state.

---

## Empty State

Purpose:

Support new users who have not yet entered financial data.

The Dashboard should remain informative.

Examples:

Financial Position

"Belum ada data keuangan."

Wallet Overview

"Tambahkan dompet pertama Anda."

Recent Activity

"Belum ada transaksi."

Needs Attention

"Tidak ada hal yang memerlukan perhatian."

Do not display placeholder financial values.

Do not fabricate demo data.

---

## Partial Data State

Some sections contain data while others do not.

Example:

Wallets exist

↓

No transactions yet

↓

No installments

The Dashboard should gracefully display available information without creating visual imbalance.

---

## Loading State

While data is loading:

Display skeleton placeholders.

Skeletons should preserve:

- layout
- spacing
- hierarchy

Do not display loading spinners for the entire page unless absolutely necessary.

Loading should feel progressive.

---

## Error State

If dashboard data cannot be retrieved:

Display a concise explanation.

Example:

"Gagal memuat data Dashboard."

Provide:

- Retry
- Refresh

Do not expose technical errors.

---

# 27. Responsive Behaviour

The Dashboard must preserve information hierarchy across every device.

Only the composition changes.

The hierarchy never changes.

---

## Desktop

Primary focus:

Financial Position

Use multiple columns where appropriate.

The Dashboard should feel balanced.

Never stretch content purely to fill available width.

---

## Tablet

Maintain desktop hierarchy.

Reduce the number of simultaneous columns.

Preserve comfortable reading width.

---

## Mobile

Stack sections vertically.

Preserve the same reading order.

Never introduce horizontal scrolling.

Scrolling is expected.

Compression is not.

---

# 28. Accessibility

The Dashboard should be usable by a broad range of users.

---

## Typography

Maintain readable font sizes.

Financial values must remain legible.

Avoid reducing typography simply to fit more content.

---

## Contrast

Follow semantic color usage.

Positive

Negative

Neutral

Warning

Maintain sufficient contrast.

Never rely on color alone.

---

## Icons

Icons support recognition.

Icons must never become the sole indicator of meaning.

---

## Touch Targets

Interactive components should meet recommended touch target sizes.

Quick Actions

Navigation

Wallet Cards

Recent Activity

All interactive areas should be easy to activate.

---

## Screen Readers

Meaningful labels should exist for:

Financial Position

Wallet balances

Quick Actions

Recent Activity

Warnings

---

# 29. Performance Considerations

The Dashboard should feel immediately available.

Prioritize loading:

1. Financial Position

2. Needs Attention

3. Wallet Overview

4. Current Period Summary

5. Recent Activity

Lower-priority content should never delay the appearance of critical financial information.

---

# 30. Data Integrity

Every financial value displayed on the Dashboard must originate from real user data.

Never calculate speculative values.

Never estimate.

Never infer balances.

Never fabricate trends.

The Dashboard reflects reality.

Not prediction.

---

# 31. Privacy

Pocket Mint is a Private Financial Workspace.

The Dashboard should respect financial privacy.

Avoid unnecessary exposure of sensitive information.

Future implementations may support:

- Hidden balances
- Privacy mode
- Sensitive value masking

The Dashboard structure should accommodate these features.

---

# 32. Future Extensibility

The Dashboard should remain lightweight.

Future features should be added only when they reinforce the Dashboard's primary purpose.

Possible future additions:

- Smart Insights
- Financial Health Indicators
- AI Recommendations
- Goal Progress

These should never replace the existing information hierarchy.

---

# 33. Success Criteria

The Dashboard is successful when users can answer the following questions within five seconds.

What is my financial position today?

Is there anything requiring my attention?

What action do I most likely want to perform next?

Where is my money?

What happened recently?

If users cannot answer these questions immediately, the Dashboard should be reconsidered.

---

# 34. Things to Avoid

Do not introduce:

- fake charts
- decorative analytics
- stock market visualizations
- crypto exchange layouts
- marketing banners
- promotional cards
- gamification
- achievement systems
- badges
- unnecessary notifications

Do not overload the Dashboard with information.

Do not compete for visual attention between sections.

Do not replace clarity with decoration.

---

# 35. Design Checklist

Before approving any Dashboard design, verify the following.

Financial Position is the primary visual anchor.

Needs Attention is immediately visible.

Quick Actions are accessible.

Wallet Overview clearly answers where money is stored.

Current Period Summary remains lightweight.

Recent Activity is easy to scan.

No fabricated information exists.

No analytics dominate the interface.

Spacing follows the Layout & Spacing Specification.

Composition follows the Dashboard Composition document.

The Application Shell remains unchanged.

Visual language follows the Design Principles.

---

# 36. Definition of Done

The Dashboard Specification is considered complete when:

- every section has a clearly defined purpose
- every displayed metric has a business justification
- hierarchy is fixed
- navigation relationships are defined
- responsive behavior is documented
- accessibility expectations are documented
- loading, empty, partial, and error states are documented
- prohibited patterns are explicitly listed
- success criteria are measurable

This document serves as the authoritative functional specification for the Pocket Mint Dashboard.

All future design iterations and frontend implementations should conform to this specification unless superseded by an approved product decision.