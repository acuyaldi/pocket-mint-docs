# Pocket Mint — Product RFC

> Version: 3.0  
> Status: Draft — Requires Implementation Reconciliation  
> Last Updated: 2026-07-14  
> Scope: Product behavior, financial rules, and cross-repository contracts  
>
> Implementation references:
> - Backend: `pocket-mint-be`
> - Frontend: `pocket-mint-fe`
>
> Related documents:
> - [`design-system.md`](./design-system.md)
> - `screen-spec.md` — planned
> - `component-spec.md` — planned

# Purpose

This document defines the product behavior, business rules, financial logic,
and domain model of Pocket Mint.

It is the primary source of truth for product requirements.

This document intentionally does **not** define visual design,
colors, typography, layouts, or reusable UI components.

See also:

- design-system.md
- screen-spec.md
- component-spec.md

---

# Product Overview

Pocket Mint is a privacy-first, self-hosted personal finance workspace
designed for individuals who want complete visibility and control over
their financial life.

Unlike traditional expense trackers, Pocket Mint focuses on financial
clarity through multi-wallet consolidation, debt tracking, installment
management, and automation.

Core capabilities:

- Multi-wallet management
- Net worth calculation
- Credit & PayLater tracking
- Installment management
- Financial reporting
- Automation-ready architecture

---

# Product Principles

Pocket Mint follows these principles:

1. Financial clarity over feature quantity.
2. Privacy first.
3. Every financial number must be explainable.
4. Fast data entry is more valuable than excessive customization.
5. Financial calculations must always be deterministic.
6. Automation assists the user, never replaces user control.
7. Business rules belong in the backend.

---

# Goals

Pocket Mint aims to:

- provide accurate financial visibility
- simplify debt management
- minimize manual bookkeeping effort
- support long-term financial planning
- remain fully self-hostable
- remain extensible for automation

---

# Non Goals

Pocket Mint is NOT:

- an investment platform
- a budgeting-envelope application (zero-based allocation of unspent funds across categories, with rollover between them)
- accounting software
- an online banking platform
- a collaborative finance application
- a multi-currency accounting system

> **Clarification (PD-009, Approved):** Budgeting tracks a fixed monthly spend limit per Category against recorded transactions — it does not allocate, move, or roll over funds between categories, and is therefore distinct from the excluded "budgeting-envelope application" pattern above.

---

# Target Users

Pocket Mint is designed for:

- privacy-conscious users
- self-hosting enthusiasts
- users managing multiple wallets
- users using PayLater or credit cards
- users wanting financial automation

---

# Domain Model

Primary entities:

- User
- Wallet
- Category
- Transaction
- Installment
- Transfer

Detailed schemas are defined by Prisma.

Backend Prisma schema remains the implementation source of truth.

---

# Financial Rules

Pocket Mint follows these financial principles.

## Money Precision

- All money values use Decimal.
- Floating point arithmetic is prohibited.
- UI formatting must never change stored values.

---

## Wallet Types

Wallet types are divided into:

Assets

- Cash
- Bank
- E-Wallet

Debts

- Credit Card
- PayLater

Debt classification is derived by application logic.

---

## Net Worth

Pocket Mint Net Worth is calculated from financial records tracked by
Pocket Mint at the same reporting cutoff.

Net Worth

= Total Assets − Total Outstanding Debt

Total Assets is the combined balance of all asset wallets tracked by
Pocket Mint.

Total Outstanding Debt is the combined outstanding amount of all debt
wallets tracked by Pocket Mint.

Rules:

- An installment obligation already represented in a debt wallet's
  outstanding balance must not be deducted again.
- An assets-only value must be labeled Total Assets and must not be
  presented as Net Worth.
- Net Worth may be negative.
- Net Worth represents only assets and debts tracked by Pocket Mint.

---

## Outstanding

Outstanding

= absolute value of debt wallet balance

---

## Available Credit

Available Credit

= Credit Limit − Outstanding

---

## Debt Ratio

Debt Ratio

= Outstanding / Credit Limit

---

## Installment Model

Pocket Mint uses the Front-loaded Balance Deduction model.

Rules:

- Wallet balance is deducted only once.
- Outstanding immediately reflects the total debt.
- Monthly reports only record monthly installments.
- Future installment records never deduct wallet balance again.

---

## Installment Calculator

Supported modes:

Forward Calculation

Input:

- principal
- interest rate
- duration

Reverse Calculation

Input:

- principal
- monthly payment
- duration

---

## Budgeting

Governed by [PD-009](./decisions/009-budgeting-scope.md) (Approved). Phase A (domain foundation — `Budget` model and calculation service) is implemented; API, frontend, dashboard integration, and notifications are not yet implemented.

A Budget is a recurring monthly spending limit for one expense Category, calculated from posted Expense transactions only. Full formulas live in the [Budgeting Calculation Specification](../development/budgeting-calculation-spec.md); scope decisions live in [PD-009](./decisions/009-budgeting-scope.md).

Rules:

- Budget Spent is always calculated live from the Ledger; it is never stored or cached.
- Only posted Expense transactions in the matching Category count. Transfers, Income, and unconfirmed recurring transactions never count.
- Budget Status is Healthy (<75%), Approaching (≥75%, <100%), Reached (exactly 100%), Exceeded (>100%), or Archived — evaluated identically by backend and frontend.
- Budgeting does not allocate, move, or roll over funds between categories; it is distinct from an envelope-budgeting system.
- One persistent Budget per user and expense Category; archiving does not free the Category for a second Budget — an archived Budget is restored or edited instead.

---

# Business Rules

General rules:

- Every resource belongs to exactly one user.
- Every operation is user scoped.
- Every financial mutation must be atomic.
- Every ownership validation happens in backend.
- All multi-table operations use database transactions.

---

# Authentication

Pocket Mint uses First Owner with Controlled Access.

Rules:

- An uninitialized installation permits a narrowly scoped first-Owner initialization journey. The first account to complete initialization becomes the installation's single active Owner.
- Public self-registration is prohibited after initialization.
- Additional Users require explicit Owner approval through an invitation or allowlist before admission.
- Every User owns isolated financial resources, and every financial operation is scoped to the requesting User.
- Owner authority is installation-level. It governs admission and installation settings but does not permit access to another User's financial resources.
- Non-financial landing content, first-Owner initialization while uninitialized, authentication, approved authentication callbacks, invitation or allowlist admission completion, account recovery, and minimal health capabilities may remain public.
- Public capabilities must not expose private financial information or disclose another User's account, invitation, or resource existence.
- Every user-financial capability requires an authenticated identity and Backend-enforced User ownership.
- Account recovery restores the requesting User's own access. Owner replacement transfers installation authority only and does not transfer or expose financial resources.
- Automation and service access require explicit authorization and narrowly defined User scope through the same Backend authorization boundary as interactive access.

---

# API

REST API

Base path

/api/v1

The backend API specification is documented separately.

This RFC defines behavior only.

---

# User Experience Principles

Pocket Mint should feel:

- calm
- trustworthy
- organized
- responsive
- data-first

User experience priorities:

1. Important financial information first.
2. Minimize interaction cost.
3. Reduce cognitive load.
4. Keep layouts predictable.
5. Never hide financial calculations.

Visual implementation is documented in design-system.md.

---

# Validation Rules

Backend validation always has priority.

Examples:

- amount > 0
- duration >= 1
- interest between 0–100%
- ownership validation
- transfer cannot use same wallet

---

# Error Handling

The backend must:

- never silently ignore failures
- preserve financial consistency
- rollback partial operations
- provide deterministic responses

---

# Testing Requirements

Minimum coverage should include:

- installment calculations
- net worth calculation
- transfer consistency
- ownership validation
- transaction rollback

---

# Roadmap

Current milestone

Stage 4

Automation

Planned:

- Secure Webhooks
- n8n Integration
- WhatsApp Parsing
- AI Transaction Extraction

Budgeting, Analytics v2, and Smart Categorization Foundation (Phase 18, [PD-010](./decisions/010-smart-categorization-foundation.md)) have since shipped. Merchant Mapping (Phase 19, [PD-011](./decisions/011-merchant-mapping.md)) is implemented, extending the same categorization pipeline as a higher-priority stage before keyword matching. Rule Engine (Phase 20, [PD-012](./decisions/012-rule-engine.md)) is implemented, extending the same pipeline as the new highest-priority stage, ahead of Merchant Mapping.

Roadmap after Rule Engine, in order: Automation, React Native, optional PWA, Dark Mode backlog.

---

# Known Issues

Current technical debt is documented separately.

Examples:

- duplicated constants
- scheduler implementation
- multi-currency discussion

---

# Related Documents

Product

- vision.md
- design-system.md
- screen-spec.md
- component-spec.md

Backend

- architecture.md
- database.md
- api.md

Development

- roadmap.md
