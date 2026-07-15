# Installments Specification

Version: 1.0

Status: Draft

Owner: Pocket Mint

Category: Screen Specification

---

# Dependencies

Depends On

- DESIGN.md
- design-philosophy.md
- design-principles.md
- information-hierarchy.md
- navigation-system.md
- interaction-principles.md
- responsive-strategy.md
- content-strategy.md
- layout-spacing.md
- app-shell.md

Related Documents

- installments-composition.md

---

# Purpose

The Installments screen helps users monitor and manage ongoing financial obligations.

Unlike Transactions, which record completed events, Installments represent commitments that continue over time.

The screen should help users answer one primary question:

"What financial obligations do I still have?"

---

# Product Role

The Installments screen is the obligation management workspace.

Users should be able to:

- monitor installment progress
- understand upcoming payments
- review payment history
- manage installment plans
- identify overdue obligations

The screen prioritizes awareness before management.

---

# User Goals

Users visit this screen to:

Review active installments.

Understand remaining balances.

Identify upcoming due dates.

Mark payments as completed.

Review installment history.

Manage installment information.

---

# Primary User Questions

The screen should answer:

What installments are still active?

Which payment is due next?

How much remains unpaid?

How much has already been paid?

Is anything overdue?

---

# Information Priority

Highest

Upcoming Payments

↓

Active Installments

↓

Installment Progress

↓

Completed Installments

↓

Supporting Information

---

# Primary Sections

## Upcoming Payments

Purpose

Highlight installments requiring immediate attention.

Displayed Information

- Installment Name
- Due Date
- Remaining Balance
- Payment Amount

Priority

Highest

---

## Active Installments

Purpose

Display every ongoing installment.

Displayed Information

- Name
- Remaining Balance
- Monthly Payment
- Progress
- Due Date

Priority

High

---

## Installment Detail

Purpose

Display complete installment information.

Information

- Total Amount
- Remaining Amount
- Installment Length
- Payment History
- Notes

Priority

Medium

---

## Completed Installments

Purpose

Provide historical reference.

Completed installments should remain visually secondary.

---

# Available Actions

Primary

Add Installment

Secondary

Search

Filter

Sort

Contextual

Edit Installment

Mark as Paid

Archive

Delete

---

# Data Requirements

Required

Installment Name

Remaining Balance

Monthly Payment

Due Date

Status

Optional

Notes

Institution

Reference Number

Attachments

Payment Schedule

---

# Business Rules

Installments must calculate remaining balances accurately.

Overdue installments always receive higher priority.

Completed installments should not appear with active installments.

Payment history should never be editable retrospectively.

---

# Interaction Rules

Users should be able to:

Review installment status.

Open installment detail.

Mark payment completed.

Edit installment information.

Archive completed installments.

Delete installment plans.

Interactions should minimize unnecessary confirmation.

---

# States

Normal

Loading

Empty

Partial

Error

Completed

Overdue

Archived

---

# Navigation Behaviour

Dashboard

↓

Installments

↓

Installment Detail

↓

Back to Installments

Navigation depth remains shallow.

---

# Responsive Behaviour

Desktop

Multiple-column layout.

Tablet

Reduced columns.

Mobile

Stacked installment cards.

Hierarchy remains unchanged.

---

# Accessibility

Installment progress should remain understandable.

Status should never rely solely on color.

Touch targets remain accessible.

Progress indicators should have textual meaning.

---

# Privacy

Future support may include:

Hidden balances.

Masked payment values.

Privacy Mode.

Installment structure should remain understandable regardless of visibility settings.

---

# Performance Considerations

Prioritize loading:

Upcoming Payments

↓

Active Installments

↓

Installment Progress

↓

Completed Installments

Critical obligations should always appear first.

---

# Success Criteria

Users can immediately understand:

What is due.

What remains unpaid.

How much progress has been made.

Which obligations require attention.

---

# Things To Avoid

Do not:

Hide overdue payments.

Mix completed installments with active ones.

Display fake progress.

Turn progress into gamification.

Use decorative charts.

Overcomplicate payment management.

---

# AI Generation Notes

Google Stitch should prioritize:

- obligation awareness
- clear payment progress
- chronological due dates
- readable financial values
- calm financial presentation

Google Stitch should avoid:

- progress gamification
- achievement badges
- celebratory visuals
- crypto aesthetics
- analytics dashboards
- oversized progress graphics

This screen inherits every Pocket Mint foundation document.

---

# Definition of Done

The Installments screen is complete when:

✓ Upcoming obligations are obvious.

✓ Progress is understandable.

✓ Remaining balances are easy to scan.

✓ Payment actions are easy to access.

✓ Responsive behaviour follows the Pocket Mint Design System.

This document serves as the functional specification for the Installments screen.