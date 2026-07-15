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
  DEFAULT: 0.5rem
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

# End DataStore Snapshot