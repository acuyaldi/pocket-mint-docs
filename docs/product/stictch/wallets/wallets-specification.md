# Wallets Specification

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

- wallets-composition.md

---

# Purpose

The Wallets screen is the user's financial inventory.

It answers one primary question:

"Where is my money and what do I currently own or owe?"

Unlike the Dashboard, which summarizes financial position, the Wallets screen focuses on financial assets and liabilities.

Users should understand the distribution of their finances before managing individual wallets.

---

# Product Role

The Wallets screen is the primary asset management workspace.

It allows users to:

- review financial assets
- review liabilities
- organize wallets
- manage balances
- inspect wallet details

The Wallets screen is not intended to become a reporting interface.

---

# User Goals

Users visit the Wallets screen to:

Understand where money is stored.

Review balances.

Manage financial accounts.

Create new wallets.

Inspect wallet details.

Edit existing wallets.

---

# Primary User Questions

The Wallets screen should answer:

Where is my money?

Which wallets do I own?

Which wallets represent debt?

What is the balance of each wallet?

What wallet should I manage next?

---

# Information Priority

Highest

Asset Summary

↓

Wallet Categories

↓

Wallet List

↓

Wallet Details

↓

Supporting Information

---

# Primary Sections

## Asset Summary

Purpose

Provide a high-level overview of the user's financial assets and liabilities.

Displayed Information

- Total Assets
- Total Debt
- Total Wallet Count

Primary Action

None

Priority

Highest

---

## Wallet Categories

Purpose

Separate financial assets from liabilities.

Examples

Assets

Liabilities

Archived

Priority

High

---

## Wallet List

Purpose

Display all active wallets.

Each wallet should communicate:

Wallet Name

Wallet Type

Institution

Current Balance

Status

Priority

High

---

## Wallet Details

Purpose

Provide additional information after selecting a wallet.

Information may include:

Current Balance

Recent Transactions

Notes

Linked Installments

Wallet Settings

Priority

Medium

---

# Available Actions

Primary

Add Wallet

Secondary

Search

Sort

Filter

Contextual

Rename Wallet

Archive Wallet

Delete Wallet

Transfer Balance

---

# Data Requirements

Required

Wallet Name

Wallet Type

Current Balance

Wallet Status

Optional

Institution

Description

Color

Icon

Notes

Archived Status

---

# Business Rules

Wallet balances must always reflect actual financial data.

Assets and liabilities must remain separated.

Archived wallets should not appear by default.

Negative balances must never be hidden.

Wallet ordering should remain predictable.

---

# Interaction Rules

Users should be able to:

Create wallets.

Open wallet details.

Edit wallets.

Archive wallets.

Delete wallets.

Transfer between wallets.

Primary interactions should require minimal effort.

---

# States

Normal

Loading

Empty

Partial

Error

Archived

---

# Navigation Behaviour

Dashboard

↓

Wallets

↓

Wallet Details

↓

Back to Wallets

Wallet navigation should remain shallow.

---

# Responsive Behaviour

Desktop

Multi-column asset overview.

Tablet

Reduced columns.

Mobile

Vertical wallet list.

Wallet hierarchy remains unchanged.

---

# Accessibility

Wallet balances remain readable.

Icons support labels.

Touch targets remain comfortable.

Wallets should remain keyboard accessible on Desktop.

---

# Privacy

Wallet balances may support:

Hidden Balance

Masked Balance

Privacy Mode

Wallet structure should remain functional even when values are hidden.

---

# Performance Considerations

Prioritize loading:

Asset Summary

↓

Wallet List

↓

Wallet Details

Supporting information should never delay balance visibility.

---

# Success Criteria

Users can immediately understand:

Where money is stored.

How much is available.

Which wallets require management.

How assets and liabilities are distributed.

---

# Things To Avoid

Do not:

Mix assets and liabilities together.

Display fake balances.

Use decorative charts.

Overcomplicate wallet management.

Duplicate Dashboard functionality.

Turn Wallets into an analytics page.

---

# Definition of Done

The Wallets screen is complete when:

✓ Asset organization is obvious.

✓ Wallet hierarchy is clear.

✓ Balances remain the primary information.

✓ Management actions are easy to discover.

✓ Navigation remains predictable.

✓ Responsive behaviour follows the Pocket Mint Design System.

This document serves as the functional specification for the Wallets screen.