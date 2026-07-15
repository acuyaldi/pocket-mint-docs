# Pocket Mint Wallets Package

Version: 1.0

Status: AI Package

Owner: Pocket Mint

Category: Google Stitch Generation Package

---

# Purpose

This document provides Google Stitch with the essential requirements for generating the Pocket Mint Wallets screen.

It summarizes the relevant Pocket Mint design foundation, Wallets Specification, and Wallets Composition into one concise generation package.

This package is not the source of truth.

The source documents take precedence when a conflict exists.

---

# Product Identity

Pocket Mint is a **Private Financial Workspace**.

It helps users understand:

- what they own
- what they owe
- where their money is stored
- what requires management

Pocket Mint is not:

- an expense tracker
- a banking application
- a crypto dashboard
- an investment trading platform
- an analytics dashboard
- a budgeting game

The interface must feel:

- calm
- professional
- trustworthy
- organized
- information-first
- privacy-first

Never playful.

Never gamified.

Never marketing-heavy.

---

# Screen Identity

Screen name:

**Dompet**

Primary navigation:

- Dashboard
- Dompet
- Transaksi
- Cicilan
- Akun

Do not rename the screen.

Do not replace “Dompet” with “Wallets” inside the Indonesian product interface.

---

# Screen Goal

The Dompet screen is the user's financial inventory and wallet-management workspace.

It should answer one primary question:

> Where is my money, and what do I currently own or owe?

Users should quickly understand:

- total assets
- total liabilities
- available wallets
- the balance of each wallet
- which wallets represent debt
- which wallet may require management

---

# User Mental Model

Before opening:

> “I want to see where my money is stored.”

After opening:

> “I understand my total assets, liabilities, and the balance of each wallet.”

Before leaving:

> “I know which wallet I need to inspect or manage.”

---

# Primary Focal Point

The primary focal point is the **Asset and Liability Summary**.

It should communicate:

- Total Assets
- Total Outstanding Debt
- Total Wallet Count or Active Wallet Count, when useful

The summary must remain concise.

Do not turn it into an analytics hero.

Do not add charts, trends, forecasts, or comparisons.

---

# Primary User Questions

The screen should immediately answer:

1. How much do I own?
2. How much do I owe?
3. Where is my money stored?
4. Which wallets are assets?
5. Which wallets are liabilities?
6. What is the current balance of each wallet?
7. Which wallet should I manage next?

---

# Information Priority

Use this hierarchy:

1. Asset and Liability Summary
2. Asset and Liability Separation
3. Wallet List
4. Wallet Detail or Selected Wallet Context
5. Supporting actions and metadata

This hierarchy must remain consistent across Desktop, Tablet, and Mobile.

---

# Reading Flow

Use this reading order:

Asset and Liability Summary

↓

Wallet Categories or Segmentation

↓

Wallet List

↓

Selected Wallet Detail

↓

Wallet Management Actions

Do not reorder the flow.

Users should understand the inventory before seeing management controls.

---

# Main Sections

## 1. Page Header

Display:

- Dompet
- optional supporting context, such as wallet count or last updated information
- Add Wallet as the primary action

Do not use:

- greetings
- marketing copy
- promotional content
- motivational statements

---

## 2. Asset and Liability Summary

Purpose:

Provide an immediate overview of financial ownership and obligations.

Display:

- Total Assets
- Total Outstanding Debt
- optional Active Wallet Count

Requirements:

- financial values receive strong emphasis
- assets and liabilities remain semantically distinct
- liabilities must never appear as positive financial indicators
- summary should remain calm and compact
- no charts or decorative analytics

Priority:

Highest

---

## 3. Wallet Segmentation

Purpose:

Help users distinguish financial ownership from debt.

Recommended categories:

- Aset
- Kewajiban
- Diarsipkan, when relevant

Possible presentation:

- tabs
- segmented control
- clearly separated sections

Requirements:

- assets and liabilities must not be mixed without clear distinction
- the control should remain compact
- segmentation must support the wallet list, not dominate it

Priority:

High

---

## 4. Wallet List

Purpose:

Display the user's active financial accounts and wallets.

Each wallet item or card should include:

- Wallet Name
- Wallet Type
- Institution, when available
- Current Balance
- Status, when relevant
- supporting icon

Possible wallet types:

- Cash
- Bank Account
- Digital Wallet
- Savings
- Investment
- Credit Card
- Loan or liability account

Requirements:

- wallet balance is the strongest information inside each item
- wallet name is immediately recognizable
- metadata remains secondary
- negative balances remain visible
- debt wallets use appropriate semantic treatment
- cards remain consistent in structure and height where possible

Priority:

High

---

## 5. Selected Wallet Detail

Purpose:

Provide additional context after a wallet is selected.

Possible information:

- current balance
- wallet type
- institution
- recent transactions
- linked installments
- notes
- wallet status

Possible actions:

- Edit Wallet
- Transfer
- View Transactions
- Archive
- Delete

Requirements:

- detail remains secondary to the wallet inventory
- do not show a large empty detail panel before a wallet is selected
- management actions should appear only when relevant
- destructive actions should not receive primary visual emphasis

Priority:

Medium

---

## 6. Empty State

When no wallet exists, show:

- a concise explanation
- one clear Add Wallet action
- optional restrained illustration

Suggested Indonesian copy:

**Belum ada dompet**

“Tambahkan dompet pertama untuk mulai mencatat posisi keuangan Anda.”

Do not display:

- fake balances
- sample wallets
- fabricated analytics
- oversized illustrations

---

# Primary Action

Primary action:

**Tambah Dompet**

The action should be clearly visible without dominating the page.

Desktop:

- place near the page header or main wallet-management region

Mobile:

- use a clearly visible action in the page flow or one restrained floating action button when appropriate

Do not create multiple competing primary actions.

---

# Secondary Actions

Possible secondary actions:

- Search
- Filter
- Sort
- View Archived Wallets

These actions should remain visually quieter than Add Wallet.

Do not display advanced controls before they are needed.

---

# Contextual Actions

Possible contextual actions:

- Edit Wallet
- Rename Wallet
- Transfer
- Archive Wallet
- Delete Wallet
- View Transactions

Show contextual actions only after selecting or opening a wallet.

Do not display destructive actions globally.

---

# Layout Rules

Follow Pocket Mint Layout & Spacing.

## Desktop

- 40px application safe area
- compact persistent sidebar
- clear gutter between sidebar and content
- 32px separation between major sections
- 24px standard card padding
- use a structured content width rather than stretching across the entire viewport

Recommended composition:

Page Header

↓

Asset and Liability Summary

↓

Wallet Segmentation

↓

Wallet List + Optional Selected Wallet Detail

The wallet list should occupy the majority of usable content space.

The detail region should only appear when useful.

---

## Tablet

- 32px outer safe area
- reduced simultaneous columns
- maintain summary at the top
- wallet detail may use a slide-over, drawer, or stacked region
- preserve touch-friendly controls

Do not force Desktop density into Tablet.

---

## Mobile

- 20px horizontal safe area
- clear separation below the application bar
- vertical reading flow
- one primary task at a time
- no horizontal dashboard scrolling

Recommended composition:

Page Header

↓

Asset and Liability Summary

↓

Wallet Segmentation

↓

Wallet List

↓

Selected Wallet Detail or Detail Screen

Scrolling is preferred over compressed content.

---

# Composition Rules

The screen should feel like an organized financial inventory.

Use composition to communicate:

Overview

↓

Grouping

↓

Inventory

↓

Detail

↓

Management

The Asset and Liability Summary is the visual anchor.

The Wallet List occupies the largest functional area.

Wallet Detail supports the list and must not visually compete with it.

Whitespace should separate major regions.

Related information inside wallet items should remain compact.

---

# Information Density

Desired density:

**Medium**

The Dompet screen should be:

- denser than Dashboard
- less dense than Transactions
- comfortable for repeated management
- easy to scan

Avoid both oversized wallet cards and spreadsheet-like compression.

---

# Visual Language

Use Pocket Mint’s established visual language:

- sage and mint palette
- calm neutral surfaces
- restrained semantic colors
- subtle borders
- minimal shadows
- consistent corner radius
- one consistent icon family
- typography-led hierarchy
- tabular financial figures

Financial values should receive the strongest visual emphasis.

Icons support recognition.

Icons never replace labels.

---

# Financial Semantics

Assets:

- positive or neutral-positive semantic treatment

Liabilities:

- warning or negative semantic treatment

Neutral metadata:

- restrained neutral treatment

Never invert these meanings.

Never hide liabilities to make the screen appear healthier.

---

# Responsive Behaviour

Desktop, Tablet, and Mobile must communicate the same wallet inventory.

Only the composition changes.

The following must not change:

- information hierarchy
- asset and liability separation
- wallet balance emphasis
- primary action
- navigation labels
- visual language

---

# Interaction Expectations

Users should be able to:

- add a wallet
- inspect a wallet
- edit wallet information
- transfer between wallets
- view wallet transactions
- archive a wallet
- delete a wallet

Interactions should feel:

- predictable
- immediate
- calm
- reversible where possible

Use confirmation only for destructive actions.

---

# Required States

Generate or account for:

- normal state
- loading state
- empty state
- partial-data state
- error state
- archived state
- selected-wallet state
- hidden-balance or privacy state

Loading should preserve the layout using skeletons.

Do not use a full-page spinner when partial loading is possible.

---

# Privacy Requirements

The composition should support:

- hidden balances
- masked financial values
- privacy mode

The screen must remain understandable when financial values are hidden.

Do not expose unnecessary sensitive details.

---

# Google Stitch Instructions

Use this package as the generation authority for the Dompet screen.

Generate one coherent screen family:

- Desktop
- Tablet
- Mobile

Preserve:

- Pocket Mint identity
- approved application shell
- navigation
- information hierarchy
- layout rules
- semantic colors
- component consistency

Do not invent new product features.

Do not change the screen hierarchy.

Do not generate alternative concepts.

Do not create duplicate screens or V2 versions.

---

# Google Stitch Must Avoid

Do not generate:

- decorative charts
- wallet analytics
- spending graphs
- investment-performance panels
- crypto wallet aesthetics
- banking promotional cards
- card carousels with horizontal scrolling
- oversized wallet illustrations
- excessive gradients
- glowing effects
- glassmorphism
- gamification
- rewards or badges
- multiple primary actions
- fake wallet balances
- fake institutions
- fake financial insights
- empty detail panels
- unrelated dashboard widgets

---

# Expected Deliverables

Generate:

1. Desktop Dompet screen
2. Tablet Dompet screen
3. Mobile Dompet screen

All three must feel like the same product and represent the same information hierarchy.

Only layout and composition may adapt.

---

# Acceptance Criteria

The generated Dompet screen passes when:

- users immediately understand total assets and liabilities
- assets and liabilities are clearly separated
- wallet balances are easy to scan
- the wallet list is the primary working area
- the summary remains the visual anchor
- the primary action is obvious but restrained
- wallet details support rather than dominate
- Desktop feels balanced
- Tablet feels readable
- Mobile feels focused
- no fake analytics or fabricated information appears
- the design feels calm, professional, and implementation-ready

---

# References

Core documents:

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

Screen documents:

- wallets-specification.md
- wallets-composition.md

This package is optimized for Google Stitch generation.