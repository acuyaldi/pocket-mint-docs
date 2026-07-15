# Pocket Mint AI Context Template

Version: 1.0

Status: Template

Owner: Pocket Mint

Category: AI Context

---

# Purpose

This document provides an AI-optimized summary of one Pocket Mint screen.

Unlike the source documentation, this document is intentionally concise.

Its purpose is to provide Google Stitch (or other AI design tools) with only the information required to generate one screen consistently.

This document is NOT the source of truth.

The Core Design System and Screen Documentation always take precedence.

---

# Screen

<Name>

---

# Product Identity

Pocket Mint is a Private Financial Workspace.

The product exists to help users understand:

• What do I own?

• What do I owe?

• What requires attention?

• What happened recently?

The interface should feel:

- Calm
- Professional
- Trustworthy
- Information-first
- Privacy-first

Avoid:

- Expense Tracker appearance
- Banking App appearance
- Crypto Dashboard aesthetics
- Analytics Dashboard layouts
- Gamification
- Marketing-heavy interfaces

---

# Screen Goal

Describe the single purpose of this screen.

One screen should answer one primary user question.

---

# Primary User Questions

List 3–5 questions that users should immediately answer after opening the screen.

---

# Reading Flow

Describe the reading order.

Example

Hero

↓

Summary

↓

Cards

↓

List

↓

Details

Never reorder unless explicitly required.

---

# Primary Focal Point

Define the visual anchor.

Only one focal point should dominate.

Everything else supports it.

---

# Information Priority

Highest

↓

High

↓

Medium

↓

Low

The hierarchy should remain identical across every breakpoint.

---

# Main Sections

List every major section.

For each section include:

Purpose

Priority

Displayed Information

Primary Action

---

# Layout Rules

Follow Pocket Mint Layout & Spacing.

Desktop

40px Safe Area

Tablet

32px

Mobile

20px

Section Gap

32px

Cards use internal padding.

Never increase spacing using large empty gaps.

---

# Navigation

Desktop

Persistent Sidebar

Mobile

Bottom Navigation

Navigation labels:

Dashboard

Dompet

Transaksi

Cicilan

Akun

Never rename navigation.

---

# Composition Rules

Describe:

Reading rhythm

Section relationships

Progressive disclosure

Whitespace strategy

Visual balance

Composition should prioritize understanding.

Not decoration.

---

# Visual Language

Professional

Quiet

Minimal

Soft elevation

Subtle borders

Semantic colors

Financial values receive highest emphasis.

Icons support information.

Never decorate.

---

# Responsive Behaviour

Desktop

Overview

Tablet

Balanced

Mobile

Sequential

Hierarchy never changes.

Only layout changes.

---

# AI Generation Instructions

Google Stitch should:

- Preserve Pocket Mint identity.
- Follow Information Hierarchy.
- Follow Layout Spacing.
- Preserve Navigation.
- Use calm composition.
- Prioritize readability.
- Produce Desktop and Mobile layouts.
- Use consistent card structures.
- Preserve semantic colors.

---

# AI Must Avoid

Do not generate:

Fake charts

Analytics dashboards

Banking dashboards

Crypto layouts

Trading terminals

Decorative gradients

Large hero illustrations

Marketing banners

Gamification

Greeting headers

Oversized floating buttons

Multiple focal points

Competing cards

Visual clutter

---

# Expected Deliverables

Generate:

Desktop

Tablet

Mobile

All layouts should communicate the same information.

Only composition should change.

---

# Acceptance Criteria

The generated screen should:

✓ Preserve Pocket Mint identity.

✓ Answer the intended user questions.

✓ Have one focal point.

✓ Follow layout spacing.

✓ Preserve reading flow.

✓ Maintain visual hierarchy.

✓ Feel calm.

✓ Feel professional.

✓ Feel implementation-ready.

---

# References

Core

- DESIGN.md
- design-philosophy.md
- design-principles.md
- visual-language.md
- information-hierarchy.md
- navigation-system.md
- interaction-principles.md
- responsive-strategy.md
- content-strategy.md
- layout-spacing.md

Screen

- specification.md
- composition.md

This template serves as the standard AI Context format for every Pocket Mint screen.