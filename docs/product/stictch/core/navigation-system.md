# Pocket Mint Navigation System

Version: 1.0

Status: Foundation

Owner: Pocket Mint

Category: Navigation System

---

# Purpose

This document defines the navigation architecture used throughout Pocket Mint.

It establishes a consistent navigation experience across Desktop, Tablet, and Mobile.

Navigation should help users move confidently through the application.

Navigation is never the primary content.

Navigation supports content.

---

# Navigation Philosophy

Navigation should feel invisible.

Users should always know:

- where they are
- where they can go
- how to return

Navigation should never become the visual focus of a screen.

The content is always more important.

---

# Navigation Structure

Pocket Mint uses a shallow navigation hierarchy.

Primary Navigation

↓

Screen

↓

Section

↓

Detail

Avoid creating unnecessary navigation depth.

Users should reach important information within one or two interactions.

---

# Primary Navigation

Pocket Mint contains five primary destinations.

Dashboard

Dompet

Transaksi

Cicilan

Akun

These destinations should remain consistent across every platform.

Do not rename them.

Do not reorder them without an approved product decision.

---

# Desktop Navigation

Desktop uses a persistent sidebar.

Sidebar remains visible during navigation.

Users should always understand their current location.

Sidebar should never collapse automatically.

---

## Sidebar Contents

Application Logo

↓

Primary Navigation

↓

Spacer

↓

Secondary Actions

↓

Application Version (Optional)

---

## Sidebar Principles

Compact.

Predictable.

Quiet.

Sidebar should never compete with page content.

Visual emphasis belongs to the active page.

---

## Active Navigation

Only one navigation item should be active.

The active item should communicate:

Current Location

Not interaction.

---

## Navigation Density

Navigation should remain compact.

Avoid oversized navigation rows.

Avoid excessive spacing.

Users should scan navigation quickly.

---

# Tablet Navigation

Tablet follows desktop behavior where possible.

When horizontal space becomes limited:

Navigation may collapse.

Maintain clear access to primary destinations.

---

# Mobile Navigation

Mobile uses Bottom Navigation.

Bottom Navigation contains:

Dashboard

Dompet

Transaksi

Cicilan

Akun

Never exceed five primary items.

---

## Bottom Navigation Principles

Simple.

Stable.

Thumb-friendly.

Always visible.

Never scroll horizontally.

---

## Active State

Only one item is active.

Active state should be immediately recognizable.

Inactive items should remain visible.

---

# Page Header

Every authenticated screen contains a page header.

The page header provides context.

Not branding.

---

## Page Header Contents

Page Title

↓

Supporting Information

↓

Optional Contextual Actions

---

## Avoid

Welcome messages

Greetings

Marketing copy

Quotes

Promotional banners

The page header exists for orientation.

---

# Page Titles

Page titles should remain simple.

Examples

Dashboard

Dompet

Transaksi

Cicilan

Akun

Avoid creative naming.

Avoid decorative subtitles.

---

# Context Information

Supporting information may include:

Reporting Cutoff

Last Updated

Current Filter

Wallet Name

Transaction Count

Context should support understanding.

Not decoration.

---

# Back Navigation

Back navigation appears only when entering a detail screen.

Examples

Wallet Detail

Transaction Detail

Installment Detail

Settings

Primary pages should never display a Back button.

---

# Breadcrumbs

Pocket Mint does not use breadcrumbs.

Navigation depth is intentionally shallow.

Breadcrumbs add unnecessary complexity.

---

# Floating Action Button (FAB)

FAB is optional.

Only use when one action is overwhelmingly dominant.

Examples

Add Transaction

Add Wallet

Never use multiple FABs.

---

# Primary CTA

Every screen should have only one primary action.

Examples

Dashboard

Add Transaction

Wallet

Add Wallet

Transaction

New Transaction

Installment

Add Installment

Landing

Get Started

Avoid competing primary actions.

---

# Secondary Actions

Secondary actions should remain visually quieter.

Examples

Filter

Sort

Export

Share

View All

Secondary actions support workflows.

They should never interrupt the primary journey.

---

# Contextual Actions

Contextual actions appear only when relevant.

Examples

Delete Wallet

Mark Installment Paid

Edit Transaction

Rename Wallet

Avoid displaying contextual actions globally.

---

# Navigation Consistency

Navigation placement should remain consistent.

Users should not relearn navigation between screens.

Consistency reduces cognitive effort.

---

# Navigation Priority

Primary Navigation

↓

Primary Action

↓

Secondary Action

↓

Contextual Action

↓

Overflow

Priority should remain consistent across the application.

---

# Search

Search appears only when users manage large collections.

Examples

Transactions

Wallets

Installments

Dashboard should not prioritize Search.

---

# Filters

Filters belong to management screens.

Examples

Transactions

Wallets

Installments

Filters should not dominate page layout.

---

# Deep Navigation

Users should move naturally:

Dashboard

↓

Wallet

↓

Wallet Detail

or

Dashboard

↓

Transaction

↓

Transaction Detail

Navigation depth should remain predictable.

---

# Navigation Feedback

Every interaction should provide immediate feedback.

Hover

Pressed

Focused

Selected

Disabled

Loading

Feedback should be subtle.

Not distracting.

---

# Navigation Accessibility

Navigation should support:

Keyboard navigation

Screen readers

Touch interaction

High contrast

Visible focus states

Accessibility should never be optional.

---

# Responsive Navigation

Desktop

Persistent Sidebar

Tablet

Adaptive Sidebar

Mobile

Bottom Navigation

Navigation identity should remain consistent.

Only presentation changes.

---

# Anti Patterns

Do not:

- hide primary navigation
- move navigation between screens
- introduce more than five primary destinations
- use multiple primary CTAs
- overload page headers
- create inconsistent navigation positions
- mix navigation with marketing content
- place unrelated actions in navigation

Navigation should always support clarity.

---

# Acceptance Criteria

Navigation is successful when:

✓ Users always know where they are.

✓ Users know where to go next.

✓ Navigation remains consistent.

✓ Primary actions are easy to find.

✓ Navigation never competes with content.

✓ Desktop, Tablet, and Mobile preserve the same navigation logic.

---

# Definition of Done

The navigation system is complete when every Pocket Mint screen follows the same navigation architecture regardless of platform.

This document serves as the authoritative navigation reference for all future Pocket Mint interfaces.