---
name: Pocket Mint
colors:
  surface: '#f9f9f8'
  surface-dim: '#dadad9'
  surface-bright: '#f9f9f8'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f4f3f3'
  surface-container: '#eeeeed'
  surface-container-high: '#e8e8e7'
  surface-container-highest: '#e2e2e2'
  on-surface: '#1a1c1c'
  on-surface-variant: '#414848'
  inverse-surface: '#2f3130'
  inverse-on-surface: '#f1f1f0'
  outline: '#717978'
  outline-variant: '#c0c8c7'
  surface-tint: '#406564'
  primary: '#001414'
  on-primary: '#ffffff'
  primary-container: '#002b2b'
  on-primary-container: '#6e9493'
  inverse-primary: '#a7cecd'
  secondary: '#505f76'
  on-secondary: '#ffffff'
  secondary-container: '#d0e1fb'
  on-secondary-container: '#54647a'
  tertiary: '#230a02'
  on-tertiary: '#ffffff'
  tertiary-container: '#3c1e11'
  on-tertiary-container: '#b08370'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#c2eae9'
  primary-fixed-dim: '#a7cecd'
  on-primary-fixed: '#002020'
  on-primary-fixed-variant: '#274d4c'
  secondary-fixed: '#d3e4fe'
  secondary-fixed-dim: '#b7c8e1'
  on-secondary-fixed: '#0b1c30'
  on-secondary-fixed-variant: '#38485d'
  tertiary-fixed: '#ffdbcd'
  tertiary-fixed-dim: '#efbba7'
  on-tertiary-fixed: '#2f1408'
  on-tertiary-fixed-variant: '#623e2f'
  background: '#f9f9f8'
  on-background: '#1a1c1c'
  surface-variant: '#e2e2e2'
  mint-positive: '#2DD4BF'
  amber-warning: '#F59E0B'
  coral-error: '#F87171'
  slate-neutral: '#0F172A'
typography:
  financial-display:
    fontFamily: Inter
    fontSize: 32px
    fontWeight: '600'
    lineHeight: 40px
    letterSpacing: -0.02em
  financial-display-mobile:
    fontFamily: Inter
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Inter
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 28px
  body-lg:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-md:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  label-sm:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
    letterSpacing: 0.02em
  metadata:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '400'
    lineHeight: 16px
rounded:
  sm: 0.25rem
  DEFAULT: 0.75rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  base: 8px
  internal: 16px
  card-padding: 24px
  section: 32px
  desktop-safe-area: 40px
  tablet-safe-area: 32px
  mobile-margin: 20px
  max-width: 1280px
---

# Pocket Mint — DESIGN.md

Version: 2.0

Status: Authoritative

Purpose: Google Stitch Design Brief

---

# Product

Name

Pocket Mint

Category

Private Financial Workspace

Platform

Responsive Web Application

Primary Language

Bahasa Indonesia

Target Devices

- Desktop
- Tablet
- Mobile

---

# Product Identity

Pocket Mint is a Private Financial Workspace.

It helps users answer:

• What do I own?

• What do I owe?

• What requires attention?

• What happened recently?

Pocket Mint is designed for people who want clarity over complexity.

The product should always feel:

- Calm
- Professional
- Trustworthy
- Organized
- Private
- Information-first

Never playful.

Never marketing-heavy.

Never gamified.

---

# Product Positioning

Pocket Mint is NOT:

- Expense Tracker
- Banking Application
- Budgeting Game
- Crypto Dashboard
- Investment Platform
- Generic Admin Dashboard

Pocket Mint IS:

A personal financial workspace that provides a clear picture of someone's financial position.

---

# Design Philosophy

Pocket Mint follows one principle:

Financial clarity over visual decoration.

Every component exists to improve understanding.

Never add visual elements that do not improve comprehension.

Whitespace is functional.

Typography is functional.

Color communicates meaning.

---

# Visual Language

Visual style should feel closer to:

- Linear
- Notion
- Raycast
- 1Password

Avoid visual inspiration from:

- Mobile Banking Apps
- Crypto Exchanges
- Trading Platforms
- Consumer Fintech Marketing Sites

The interface should feel premium through restraint rather than decoration.

---

# Layout Philosophy

Pocket Mint follows a clean application layout.

Desktop

Sidebar

↓

Page Header

↓

Content

Tablet

Compact Navigation

↓

Content

Mobile

Top App Bar

↓

Content

↓

Bottom Navigation

---

# Responsive Philosophy

Desktop is the master breakpoint.

Tablet and Mobile inherit the Desktop visual language.

Responsive layouts may adapt:

- Width
- Columns
- Stacking

Responsive layouts must never change:

- Component style
- Typography hierarchy
- Border radius
- Elevation
- Color system
- Card design
- Icon style
- Spacing scale

Every breakpoint should feel like the same product.

Composition by device

Desktop

- Persistent sidebar navigation
- Page header above the content region
- 12-column grid within a maximum content width of 1280px
- 40px desktop safe area
- Desktop is the master composition and establishes content order

Tablet

- Compact navigation above or beside the content without changing labels
- 8-column grid
- 32px safe area
- Reduce column count or stack regions while preserving component appearance and content order

Mobile

- Top app bar
- Single reading column within a 4-column grid
- 20px horizontal margin
- Bottom navigation
- Stack desktop regions in the established reading order; do not hide primary financial information

Across breakpoints

- Change width, column count, and stacking only
- Keep the same component tokens, content, typography hierarchy, radius, elevation, color meaning, icon treatment, and interaction behavior
- Do not create mobile-only or tablet-only visual variants of shared components
- Keep the page itself free of horizontal overflow
- Preserve a minimum 44px touch target without visually inflating compact controls

---

# Information Hierarchy

Every screen should communicate information in this order:

Primary

↓

Supporting

↓

Actions

↓

History

Users should understand before interacting.

---

# Spacing System

Base spacing

8px rhythm

Major section spacing

32px

Card padding

24px

Internal spacing

16px

Desktop safe area

40px

Tablet safe area

32px

Mobile horizontal margin

20px

Avoid cramped layouts.

Avoid excessive whitespace.

---

# Grid

Desktop

12 columns

Tablet

8 columns

Mobile

4 columns

Maximum desktop content width

1280px

---

# Color Philosophy

Color communicates financial meaning.

Primary

Mint

Assets

Income

Positive values

Secondary

Slate

Navigation

Neutral interface

Typography

Warning

Amber

Upcoming

Needs Attention

Pending

Error

Coral Red

Debt

Negative balances

Critical financial states

Never use color only for hierarchy.

Typography should establish importance first.

---

# Typography

Font

Inter

Use tabular figures for all financial numbers.

Hierarchy

Financial Values

↓

Section Titles

↓

Body

↓

Metadata

Numbers always receive the strongest emphasis.

Labels should never compete with values.

---

# Elevation

Use minimal elevation.

Cards separate information.

Not decoration.

Canvas

↓

Cards

↓

Modal

Avoid heavy shadows.

Avoid glassmorphism.

Avoid neumorphism.

---

# Shape

Default corner radius

12px

Large containers

16px

Pills

999px

Keep shapes soft but professional.

---

# Components

Every shared component has only one visual definition.

The following components must remain visually identical across Desktop, Tablet and Mobile:

- Hero Card
- Wallet Card
- Summary Card
- Quick Action
- Activity Row
- Installment Card
- Buttons
- Inputs
- Section Header

Responsive layouts adapt composition only.

Never redesign components for different breakpoints.

---

# Component States

Every interactive component must define the following states without changing its visual identity.

Hover

- Apply only on devices that support hover
- Use a restrained surface, border, or text change
- Never shift layout, add a glow, or introduce a new color meaning

Focus Visible

- Show a persistent, high-contrast outline or ring around the control
- The indicator must remain distinguishable from hover and selected states
- Never remove the browser focus indicator without an accessible replacement

Pressed

- Provide immediate tactile feedback through a restrained surface or border change
- Preserve the control's size, label, and position

Selected

- Use more than color: pair the semantic color with a border, indicator, icon, checkmark, or text treatment
- Expose the selected state programmatically

Disabled

- Keep the label legible while reducing emphasis
- Remove hover and pressed feedback
- Prevent activation and expose the disabled state programmatically

Loading

- Preserve the control's dimensions and, where possible, its action label
- Prevent duplicate submission
- Never imply success before the operation succeeds

Skeleton

- Match the dimensions and hierarchy of the content being loaded
- Use neutral surfaces only
- Do not display fabricated values, labels, charts, or financial trends
- Respect reduced-motion preferences

Empty

- Explain that no records exist and provide only an action already supported by the product
- Never replace a loading or error state with an empty state

Error

- State what failed in clear Bahasa Indonesia
- Place field errors beside the relevant field and page errors near the affected region
- Offer retry only when retry is supported
- Never convert unavailable financial data to zero

No Search Result

- Preserve the current query and filters
- State that no matching results were found
- Offer only supported ways to clear or adjust the search

Destructive Confirmation

- Name the affected item and state the consequence before confirmation
- Use Coral only for the destructive action and critical message
- Keep the safe cancel action clear and available
- Never preselect or automatically trigger the destructive action

Overdue

- Use Coral with an explicit overdue label and due date
- Never rely on color alone

Upcoming Payment

- Use Amber with an explicit upcoming or due-soon label and due date
- Do not style an upcoming payment as overdue

---

# Hero Card

Purpose

Communicate the user's financial position.

Contains only:

- Net Worth
- Total Assets
- Total Outstanding Debt
- Reporting Cutoff

Never include:

- Charts
- Sparklines
- Trend graphs
- Decorative analytics
- KPI dashboards

The Hero Card is always the primary visual anchor.

---

# Wallet Cards

Each wallet displays:

- Wallet Name
- Wallet Type
- Balance

Optional:

- Institution

Assets use Mint semantic accents.

Debt wallets use Coral semantic accents.

Balance is always the dominant information.

---

# Transaction Rows

Each transaction contains:

- Category Icon
- Title
- Metadata
- Amount

Income

Mint

Expense

Dark neutral

Debt

Coral

Amounts use tabular figures.

Financial value behavior

- Format values using the Indonesian locale and the product's established currency notation
- Preserve the complete value; never abbreviate, round, or truncate a financial amount in a way that changes its meaning
- Long values may use responsive type sizing or wrap as one readable amount, but labels must not overtake the value hierarchy
- Negative asset balances and debt values use a minus sign and Coral with a text label or surrounding context that explains the state
- Ordinary expense transaction amounts remain dark neutral and must not become Coral merely because they are expenses
- Zero values use the neutral text color and must not imply positive, warning, or destructive status
- All financial values, including zero and negative values, use tabular figures

---

# Installment Cards

Display

- Installment Name
- Monthly Payment
- Remaining Balance
- Due Date

Progress indicators communicate repayment only.

Never gamify repayment.

---

# Buttons

Primary

Solid Slate

Secondary

Outlined

Danger

Coral

Buttons remain compact.

Never oversized.

---

# Navigation

Desktop

Sidebar

Mobile

Bottom Navigation

Navigation labels

Dashboard

Dompet

Transaksi

Cicilan

Akun

Never rename navigation.

---

# Dashboard Philosophy

The Dashboard is not an analytics page.

The Dashboard answers:

What is my financial position today?

What requires my attention?

What should I do next?

Where is my money?

What happened recently?

Reading order

Financial Position

↓

Needs Attention

↓

Quick Actions

↓

Wallet Overview

↓

Current Period Summary

↓

Recent Activity

Never change this hierarchy.

---

# Wallet Philosophy

The Wallet screen is an inventory.

Users immediately understand:

Assets

Liabilities

Balances

Wallet organization

---

# Transactions Philosophy

The Transactions screen is a financial journal.

It prioritizes:

Search

History

Editing

Never analytics.

---

# Installments Philosophy

The Installments screen prioritizes obligations.

Users immediately understand:

Upcoming payments

Overdue payments

Remaining balance

Repayment progress

---

# Landing Philosophy

The Landing page builds trust.

Story flow

Identity

↓

Trust

↓

Features

↓

Privacy

↓

Call To Action

Never become a marketing-heavy landing page.

---

# Accessibility

Maintain sufficient contrast.

Touch targets

Minimum 44px

Never communicate status using color alone.

Support keyboard navigation.

Keyboard behavior

- Use a logical focus order that follows the visible reading order
- All controls must be reachable and operable with a keyboard
- Enter or Space activates buttons and selectable controls according to native semantics
- Escape closes a dismissible modal and restores focus to its trigger
- Do not use non-semantic clickable containers in place of buttons, links, inputs, or table controls

Modal behavior

- Desktop modals are centered, contained dialogs above the canvas
- Tablet and Mobile use the same modal styling, content order, radius, elevation, and actions; only width, wrapping, and action stacking may adapt
- Keep Mobile modals within the viewport and 20px horizontal safe area, with internal scrolling when content is long
- Trap focus inside an open modal, provide an accessible name, make the background inert, and restore focus on close
- Keep destructive confirmation separate from ordinary edit or create actions

Tables on narrow screens

- Preserve the table's information order, semantic headers, financial alignment, and shared row styling
- Do not redesign table rows into decorative cards
- When all columns cannot fit, contain horizontal scrolling inside the labelled table region; never make the page itself scroll horizontally
- Keep the primary label and financial amount understandable without relying on color alone

---

# AI Generation Rules

Treat this document as the authoritative design brief.

Generate production-ready interfaces.

Do not generate concept art.

Do not generate multiple visual directions.

Do not redesign shared components.

Do not invent new product features.

Do not invent additional sections.

Do not generate fake charts.

Do not generate decorative analytics.

Do not generate placeholder branding.

Always use:

Pocket Mint

The goal is implementation quality.

Not exploration.

Not experimentation.

Not concept generation.
