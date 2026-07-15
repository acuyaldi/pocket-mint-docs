# Wallets Composition

Version: 1.0

Status: Draft

Owner: Pocket Mint

Category: Screen Composition

---

# Dependencies

Depends On

- wallets-specification.md
- design-philosophy.md
- design-principles.md
- information-hierarchy.md
- visual-language.md
- layout-spacing.md
- app-shell.md

---

# Purpose

This document defines how the Wallets screen should be visually composed.

It establishes:

- reading flow
- visual hierarchy
- page composition
- section proportions
- visual balance

This document intentionally does not define spacing values or business logic.

---

# Composition Philosophy

The Wallets screen should feel like a financial inventory.

Users should feel organized.

Not analytical.

Not overwhelmed.

The interface should communicate ownership before management.

---

# Reading Flow

Desktop

Asset Summary

↓

Wallet Categories

↓

Wallet List

↓

Wallet Detail (Optional)

Mobile

Asset Summary

↓

Wallet Categories

↓

Wallet List

↓

Wallet Detail

Users should naturally move from understanding to management.

---

# Visual Hierarchy

Highest

Asset Summary

↓

High

Wallet List

↓

Medium

Wallet Categories

↓

Medium

Wallet Detail

↓

Low

Supporting Information

The Asset Summary should always be the primary visual anchor.

---

# Visual Weight

Asset Summary

★★★★★

Wallet List

★★★★☆

Wallet Categories

★★★☆☆

Wallet Detail

★★★☆☆

Supporting Information

★★☆☆☆

Only one section should dominate the page.

---

# Focal Point

The Asset Summary is the visual anchor.

It should communicate:

Current Assets

Current Liabilities

Wallet Count

Immediately.

No wallet card should visually compete with this summary.

---

# Desktop Composition

Desktop should prioritize overview.

Recommended structure

Asset Summary

↓

Wallet Categories

↓

Wallet List + Wallet Detail

Wallet Detail should appear only after a wallet has been selected.

Avoid empty detail panels.

---

# Tablet Composition

Tablet should simplify desktop.

Reduce simultaneous columns.

Maintain Asset Summary at the top.

Wallet Detail may become a slide-over panel or stacked section.

---

# Mobile Composition

Mobile should prioritize sequential scanning.

Asset Summary

↓

Wallet Categories

↓

Wallet List

↓

Wallet Detail

Only one major task should exist at any moment.

---

# Section Relationships

Asset Summary

supports

Wallet Categories

Wallet Categories

support

Wallet List

Wallet List

supports

Wallet Detail

Every section answers the next logical question.

---

# Progressive Disclosure

Users should first understand:

Overall Assets

↓

Wallet Types

↓

Individual Wallets

↓

Wallet Detail

↓

Management

Avoid exposing detailed settings immediately.

---

# Information Density

The Wallets screen should have medium density.

Higher than Dashboard.

Lower than Transactions.

Users should feel informed without feeling crowded.

---

# Content Balance

The Wallet List occupies most of the visual space.

Asset Summary occupies the highest visual importance.

Wallet Detail should feel supportive.

Never dominant.

---

# Empty Space Strategy

Whitespace should separate major sections.

Wallet cards should remain visually connected.

Avoid large empty regions.

Avoid overly compressed wallet lists.

---

# Component Relationships

Asset Summary

↓

Wallet Categories

↓

Wallet Cards

↓

Wallet Detail

↓

Actions

Every component contributes to financial management.

---

# Visual Rhythm

Overview

↓

Grouping

↓

Inventory

↓

Details

↓

Management

The Wallets screen should feel methodical.

---

# Emphasis Rules

Visual emphasis should come from:

Composition

Grouping

Scale

Hierarchy

Never from:

Decorative gradients

Strong shadows

Large icons

Bright colors

Wallet balances should receive the greatest emphasis inside each card.

---

# Responsive Consistency

Desktop

Tablet

Mobile

should communicate the same inventory.

Only layout changes.

The management flow remains identical.

---

# Anti Patterns

Do not:

Create multiple summary cards.

Make Wallet Detail larger than Wallet List.

Mix assets and liabilities visually.

Use analytics as the focal point.

Display decorative charts.

Use oversized wallet cards.

Allow inconsistent wallet spacing.

Create visual competition between wallet cards.

---

# Acceptance Criteria

The composition is successful when:

✓ Users immediately understand where their money is stored.

✓ Asset Summary dominates the page.

✓ Wallet List feels easy to scan.

✓ Wallet Detail supports rather than competes.

✓ Desktop feels balanced.

✓ Mobile feels focused.

✓ Assets and liabilities remain visually separated.

---

# Definition of Done

The Wallets composition is complete when:

✓ Reading flow is obvious.

✓ Asset Summary becomes the visual anchor.

✓ Wallet inventory feels organized.

✓ Wallet management feels natural.

✓ Responsive behaviour preserves hierarchy.

✓ Composition follows the Pocket Mint Design System.

This document serves as the authoritative composition reference for the Wallets screen.