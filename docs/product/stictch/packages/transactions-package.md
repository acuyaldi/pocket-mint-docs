# Pocket Mint Transactions Package

Version: 1.0

Status: AI Package

Owner: Pocket Mint

Category: Google Stitch Generation Package

---

# Purpose

This document provides Google Stitch with the essential requirements for generating the Pocket Mint Transactions screen.

It summarizes the relevant Pocket Mint design foundation, Transactions Specification, and Transactions Composition into one concise generation package.

This package is not the source of truth.

The source documents always take precedence.

---

# Product Identity

Pocket Mint is a **Private Financial Workspace**.

The Transactions screen is a financial journal.

It exists to help users understand:

- what happened
- when it happened
- where it happened
- how much money was involved

The interface must feel:

- calm
- professional
- trustworthy
- organized
- efficient
- information-first

Never feel like:

- an accounting ERP
- a spreadsheet
- an expense tracker
- a banking statement
- an analytics dashboard

---

# Screen Identity

Screen name:

**Transaksi**

Primary Navigation

- Dashboard
- Dompet
- Transaksi
- Cicilan
- Akun

Never rename the screen.

---

# Screen Goal

The Transactions screen is the user's financial history workspace.

It should help users quickly review and manage historical financial records.

The emphasis is understanding financial history before editing it.

---

# User Mental Model

Before Opening

"I want to check my financial history."

↓

After Opening

"I immediately understand my latest financial activity."

↓

Before Leaving

"I found the transaction I needed."

---

# Primary User Questions

The screen should immediately answer:

- What happened?
- When did it happen?
- Which wallet was used?
- Which category does it belong to?
- How much money was involved?
- Is there anything that requires correction?

---

# Primary Focal Point

The **Transaction List** is always the visual anchor.

Everything else supports the list.

Search and filters help users find information.

They should never dominate the page.

---

# Information Priority

1. Transaction List
2. Search
3. Filters
4. Transaction Detail
5. Supporting Information

The hierarchy remains identical across every breakpoint.

---

# Reading Flow

Search

↓

Quick Filters

↓

Transaction List

↓

Transaction Detail

↓

Contextual Actions

Never reverse this order.

---

# Main Sections

## Page Header

Display

- Transaksi
- Search
- Add Transaction

Avoid greetings or marketing copy.

---

## Search

Purpose

Quickly locate financial history.

Requirements

- always visible
- compact
- lightweight
- easy to access

Do not oversize the search field.

---

## Quick Filters

Possible filters

- Wallet
- Category
- Transaction Type
- Date
- Amount

Requirements

- secondary visual priority
- compact
- collapsible on smaller devices

---

## Transaction List

Purpose

Display chronological financial history.

Each row should include

- Title
- Amount
- Wallet
- Category
- Date
- Transaction Type

Requirements

- newest first
- balance easy to scan
- title immediately recognizable
- metadata remains secondary
- rows remain visually consistent

---

## Transaction Detail

Purpose

Provide complete information about one transaction.

Possible Information

- amount
- wallet
- category
- date
- notes
- attachments
- reference
- created time

Possible Actions

- Edit
- Delete
- Duplicate

Remain secondary to the list.

---

## Empty State

Display

- concise explanation
- Add Transaction button
- optional restrained illustration

Avoid fake transaction history.

---

# Primary Action

Primary Action

Tambah Transaksi

Should be immediately discoverable.

Do not compete with Search.

---

# Secondary Actions

Search

Filter

Sort

Export (future)

Remain visually quieter than the primary action.

---

# Contextual Actions

- Edit
- Delete
- Duplicate
- View Detail

Display only after selecting a transaction.

---

# Layout Rules

Desktop

- 40px safe area
- search and filters above list
- transaction list occupies most space
- optional detail panel

Tablet

- reduced columns
- collapsible filters
- optional slide-over detail

Mobile

- vertical composition
- search first
- quick filters
- transaction list
- detail page

No horizontal scrolling.

---

# Composition Rules

The interface communicates:

Find

↓

Review

↓

Understand

↓

Manage

Search and filters support discovery.

The transaction list remains dominant.

Whitespace separates groups.

Rows remain compact but breathable.

---

# Information Density

Highest density in Pocket Mint.

However,

Never sacrifice readability.

Rows should remain comfortable.

Avoid spreadsheet compression.

---

# Visual Language

Professional

Minimal

Neutral

Soft elevation

Subtle borders

Consistent cards

Semantic colors

Typography-first hierarchy

Amounts receive strongest emphasis.

---

# Financial Semantics

Income

Positive semantic color.

Expense

Negative semantic color.

Transfer

Neutral semantic color.

Status should never rely only on color.

---

# Responsive Behaviour

Desktop

Efficient overview.

Tablet

Balanced interaction.

Mobile

Fast scanning.

Information hierarchy never changes.

---

# Interaction Expectations

Users should be able to

- search
- filter
- sort
- inspect
- edit
- delete
- duplicate

Interaction should remain immediate.

---

# Required States

Generate

- normal
- loading
- empty
- filtered empty
- error
- offline
- selected transaction
- privacy mode

Loading should preserve layout using skeletons.

---

# Privacy Requirements

Support

- hidden amount
- masked values
- privacy mode

The list remains understandable without exposing sensitive values.

---

# Google Stitch Instructions

Generate

Desktop

Tablet

Mobile

Preserve

- Pocket Mint identity
- transaction hierarchy
- reading flow
- navigation
- spacing
- semantic colors

Do not invent additional features.

Do not redesign the workflow.

Generate only one coherent design.

---

# Google Stitch Must Avoid

Do not generate

- spreadsheets
- ERP layouts
- banking statements
- fake analytics
- KPI cards
- charts
- graphs
- oversized filters
- multiple search bars
- floating widgets
- crypto UI
- glowing effects
- glassmorphism
- decorative gradients
- marketing banners

---

# Expected Deliverables

Generate

Desktop Transactions

Tablet Transactions

Mobile Transactions

All layouts must communicate the same workflow.

Only composition changes.

---

# Acceptance Criteria

The generated Transactions screen passes when

- transaction history is immediately understandable
- search remains lightweight
- filters remain secondary
- transaction rows are easy to scan
- amounts dominate visually
- detail supports the list
- Desktop feels efficient
- Tablet feels balanced
- Mobile feels comfortable
- no fake analytics appear
- the interface feels implementation-ready

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

- transactions-specification.md
- transactions-composition.md

This package is optimized for Google Stitch generation.