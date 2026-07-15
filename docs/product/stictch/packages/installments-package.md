# Pocket Mint Installments Package

Version: 1.0

Status: AI Package

Owner: Pocket Mint

Category: Google Stitch Generation Package

---

# Purpose

This document provides Google Stitch with the essential requirements for generating the Pocket Mint Installments screen.

It summarizes the relevant Pocket Mint design foundation, Installments Specification, and Installments Composition into one concise generation package.

This package is not the source of truth.

The source documents always take precedence.

---

# Product Identity

Pocket Mint is a **Private Financial Workspace**.

The Installments screen helps users understand their financial obligations.

It exists to answer:

- what do I still owe?
- what payment is due next?
- how much remains unpaid?
- which obligation requires attention?

The interface should feel:

- calm
- trustworthy
- structured
- predictable
- information-first

Never feel like:

- a loan calculator
- a banking portal
- an achievement tracker
- a budgeting game
- an analytics dashboard

---

# Screen Identity

Screen name

**Cicilan**

Primary Navigation

- Dashboard
- Dompet
- Transaksi
- Cicilan
- Akun

Never rename the screen.

---

# Screen Goal

The Cicilan screen helps users manage ongoing financial commitments.

The screen should prioritize awareness before management.

Users should immediately understand:

- upcoming payments
- overdue payments
- remaining balances
- repayment progress

---

# User Mental Model

Before Opening

"I want to know what I still need to pay."

↓

After Opening

"I immediately know which payment comes next."

↓

Before Leaving

"I know exactly what requires my attention."

---

# Primary User Questions

The screen should answer:

- Which installment is due next?
- Is anything overdue?
- How much remains?
- How much has already been paid?
- What is my repayment progress?
- Which installment should I manage?

---

# Primary Focal Point

The **Upcoming Payments** section is always the visual anchor.

It should immediately communicate:

- Installment Name
- Due Date
- Payment Amount
- Remaining Balance

This section should receive the greatest visual emphasis.

---

# Information Priority

1. Upcoming Payments
2. Active Installments
3. Repayment Progress
4. Installment Detail
5. Completed Installments

Never reverse this hierarchy.

---

# Reading Flow

Upcoming Payments

↓

Active Installments

↓

Progress

↓

Installment Detail

↓

Completed Installments

↓

History

Users should understand urgency before history.

---

# Main Sections

## Page Header

Display

- Cicilan
- Add Installment

Avoid greetings.

Avoid marketing copy.

---

## Upcoming Payments

Purpose

Highlight obligations requiring immediate attention.

Display

- Installment Name
- Due Date
- Monthly Payment
- Remaining Balance

Requirements

- visually dominant
- concise
- immediately understandable

---

## Active Installments

Display

- Installment Name
- Monthly Payment
- Remaining Balance
- Due Date
- Progress

Requirements

- consistent cards
- chronological urgency
- readable values

---

## Repayment Progress

Purpose

Communicate repayment status.

Requirements

- neutral
- informative
- non-gamified

Progress exists to reduce uncertainty.

Never to celebrate completion.

---

## Installment Detail

Possible Information

- Total Amount
- Remaining Amount
- Monthly Payment
- Due Date
- Payment History
- Notes

Possible Actions

- Edit
- Mark as Paid
- Archive
- Delete

Remain secondary.

---

## Completed Installments

Purpose

Historical reference.

Should never compete with Active Installments.

---

## Empty State

Display

- concise explanation
- Add Installment button
- optional restrained illustration

Never generate fake installment data.

---

# Primary Action

Primary Action

Tambah Cicilan

Visible.

Easy to discover.

Not visually aggressive.

---

# Secondary Actions

Search

Filter

Sort

Remain secondary.

---

# Contextual Actions

- Edit
- Mark as Paid
- Archive
- Delete

Only visible when relevant.

---

# Layout Rules

Desktop

- 40px safe area
- clear section spacing
- summary first
- detail optional

Tablet

- reduced columns
- stacked detail

Mobile

- vertical flow
- one obligation at a time

Avoid horizontal scrolling.

---

# Composition Rules

The interface communicates

Urgency

↓

Obligation

↓

Progress

↓

Detail

↓

History

Users should always know what to pay first.

---

# Information Density

Medium.

Denser than Dashboard.

Lighter than Transactions.

Cards remain comfortable.

---

# Visual Language

Professional.

Quiet.

Soft surfaces.

Subtle borders.

Minimal shadows.

Semantic colors.

Typography-first hierarchy.

Financial values dominate.

---

# Financial Semantics

Upcoming

Attention semantic.

Overdue

Warning semantic.

Completed

Neutral semantic.

Do not rely on color alone.

---

# Responsive Behaviour

Desktop

Overview.

Tablet

Balanced.

Mobile

Sequential.

Hierarchy never changes.

---

# Interaction Expectations

Users should

- review installments
- inspect details
- mark payment completed
- edit
- archive
- delete

Interactions remain calm and predictable.

---

# Required States

Generate

- normal
- loading
- empty
- overdue
- completed
- archived
- selected installment
- privacy mode

Loading should preserve layout.

---

# Privacy Requirements

Support

- hidden balances
- masked repayment values
- privacy mode

The interface remains understandable without exposing sensitive values.

---

# Google Stitch Instructions

Generate

Desktop

Tablet

Mobile

Preserve

- Pocket Mint identity
- repayment hierarchy
- urgency-first reading flow
- spacing
- navigation
- semantic colors

Generate one coherent design.

Do not redesign the workflow.

---

# Google Stitch Must Avoid

Do not generate

- gamification
- achievement badges
- celebratory animations
- decorative progress circles
- analytics dashboards
- KPI widgets
- crypto layouts
- banking portals
- fake repayment data
- glowing effects
- glassmorphism
- oversized illustrations

---

# Expected Deliverables

Generate

Desktop Cicilan

Tablet Cicilan

Mobile Cicilan

All layouts should communicate identical information.

Only composition changes.

---

# Acceptance Criteria

The generated Cicilan screen passes when

- users immediately identify the next payment
- overdue installments are obvious
- repayment progress is understandable
- active installments dominate
- completed installments remain secondary
- Desktop feels balanced
- Tablet feels readable
- Mobile feels focused
- no gamification appears
- the screen feels implementation-ready

---

# References

Core

- DESIGN.md
- design-philosophy.md
- design-principles.md
- information-hierarchy.md
- navigation-system.md
- visual-language.md
- interaction-principles.md
- responsive-strategy.md
- content-strategy.md
- layout-spacing.md
- app-shell.md
- stitch-editing-rules.md

Screen

- installments-specification.md
- installments-composition.md

This package is optimized for Google Stitch generation.