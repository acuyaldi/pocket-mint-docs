# Installments Composition

Version: 1.0

Status: Draft

Owner: Pocket Mint

Category: Screen Composition

---

# Dependencies

Depends On

- installments-specification.md
- design-philosophy.md
- design-principles.md
- information-hierarchy.md
- visual-language.md
- layout-spacing.md
- app-shell.md

---

# Purpose

This document defines the visual composition of the Installments screen.

The composition should prioritize financial obligations before financial history.

Users should immediately understand what requires attention today before reviewing completed payments.

---

# Composition Philosophy

The Installments screen is an obligation workspace.

It should communicate responsibility.

Not pressure.

Users should immediately understand:

What needs to be paid.

When.

How much remains.

The composition should reduce uncertainty rather than create urgency.

---

# User Mental Model

Before Opening

"I know I have several ongoing installments."

↓

After Opening

"I immediately understand:

- Which payment is due next.
- Which installment is overdue.
- How much remains.
- My overall repayment progress."

↓

Before Leaving

"I know exactly what payment should happen next."

---

# Reading Flow

Desktop

Upcoming Payments

↓

Active Installments

↓

Installment Detail

↓

Completed Installments

↓

History

---

Mobile

Upcoming Payments

↓

Active Installments

↓

Installment Detail

↓

Completed Installments

↓

History

Reading should naturally move from urgency toward reference.

---

# Visual Hierarchy

Highest

Upcoming Payments

↓

High

Active Installments

↓

Medium

Installment Detail

↓

Low

Completed Installments

↓

Supporting Information

Only one urgency section should dominate the page.

---

# Visual Weight

Upcoming Payments

★★★★★

Active Installments

★★★★☆

Installment Detail

★★★☆☆

Completed Installments

★★☆☆☆

Supporting Information

★☆☆☆☆

Urgency should feel visible.

Not overwhelming.

---

# Focal Point

Upcoming Payments is always the visual anchor.

It communicates:

Next Payment

↓

Due Date

↓

Amount

↓

Remaining Balance

Users should understand this before reading anything else.

---

# Desktop Composition

Desktop prioritizes overview.

Recommended composition

Upcoming Payments

↓

Active Installments

↓

Installment Detail

↓

Completed Installments

↓

History

Avoid splitting urgency across multiple columns.

Urgent information should remain visually unified.

---

# Tablet Composition

Tablet reduces parallel information.

Installment Detail may appear as:

Slide-over

or

Stacked section.

Upcoming Payments remains fixed at the top.

---

# Mobile Composition

Mobile prioritizes sequential understanding.

Upcoming Payments

↓

Active Installments

↓

Progress

↓

Detail

↓

History

Users should never scroll through completed installments before seeing upcoming payments.

---

# Desktop Wireframe Narrative

Upcoming Payments (Hero)

↓

Active Installments

↓

Selected Installment Detail

↓

Completed Installments

↓

History

Desktop should communicate the user's obligations within one screen.

---

# Mobile Wireframe Narrative

Upcoming Payments

↓

Installment Cards

↓

Selected Detail

↓

Completed List

↓

Actions

The experience should feel guided.

Not dense.

---

# Section Relationships

Upcoming Payments

supports

Active Installments

↓

Active Installments

support

Installment Detail

↓

Installment Detail

supports

Payment Actions

↓

Completed Installments

support

History

Every section answers the next financial question.

---

# Progressive Disclosure

Upcoming Payment

↓

Installment Overview

↓

Installment Detail

↓

Payment History

↓

Management

Users should never see detailed management before understanding their obligations.

---

# Information Density

Installments follow medium density.

Higher than Dashboard.

Lower than Transactions.

Cards should remain comfortable to scan.

Progress should remain understandable.

---

# Content Balance

Upcoming Payments occupies the highest emphasis.

Active Installments occupy the largest space.

Completed Installments remain visually quiet.

History should never dominate.

---

# Empty Space Strategy

Whitespace separates sections.

Not payment rows.

Installment cards should feel grouped.

Avoid excessive vertical gaps.

Avoid compressed repayment information.

---

# Component Relationships

Upcoming Payments

↓

Active Installments

↓

Progress

↓

Installment Detail

↓

Payment Actions

↓

History

Each component supports financial awareness.

---

# Visual Rhythm

Urgency

↓

Overview

↓

Progress

↓

Detail

↓

History

The rhythm should naturally reduce urgency as users scroll.

---

# Emphasis Rules

Visual emphasis comes from:

Hierarchy

Grouping

Typography

Financial Values

Progress

Avoid emphasis through:

Bright colors

Large icons

Heavy shadows

Decorative graphics

Progress indicators should remain informative.

Never playful.

---

# Progress Representation

Progress communicates repayment.

Not achievement.

Progress bars should remain neutral.

Avoid:

Gamification

Confetti

Celebration

Badges

Progress exists to reduce uncertainty.

---

# Responsive Consistency

Desktop

Tablet

Mobile

should communicate the same repayment journey.

Only layout changes.

The repayment narrative remains identical.

---

# AI Generation Notes

Google Stitch should prioritize:

- Upcoming payment visibility
- Clear repayment progress
- Calm obligation management
- Readable financial values
- Structured installment cards

Google Stitch should avoid:

- Gamification
- Achievement systems
- Decorative progress circles
- Analytics dashboards
- KPI widgets
- Large illustrations
- Crypto-inspired layouts

This screen inherits every Pocket Mint foundation document.

---

# Anti Patterns

Do not:

Prioritize completed installments.

Hide overdue payments.

Decorate progress indicators.

Create multiple urgency cards.

Display repayment analytics.

Use celebratory visuals.

Compete with Dashboard hierarchy.

Turn repayments into achievements.

---

# Acceptance Criteria

The composition is successful when:

✓ Users immediately identify the next payment.

✓ Repayment progress is understandable.

✓ Active installments dominate.

✓ Completed installments remain secondary.

✓ Desktop feels balanced.

✓ Mobile feels guided.

✓ Financial obligations remain the focus.

---

# Definition of Done

The Installments composition is complete when:

✓ Reading flow is obvious.

✓ Upcoming Payments become the visual anchor.

✓ Repayment progress feels understandable.

✓ Payment actions remain discoverable.

✓ Responsive behaviour preserves hierarchy.

✓ Composition follows the Pocket Mint Design System.

This document serves as the authoritative composition reference for the Installments screen.