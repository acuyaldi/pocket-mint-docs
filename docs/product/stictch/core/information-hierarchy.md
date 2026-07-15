# Pocket Mint Information Hierarchy

Version: 1.0

Status: Foundation

Owner: Pocket Mint

Category: Information Architecture

---

# Purpose

This document defines the information hierarchy used across every Pocket Mint screen.

Information hierarchy determines:

- what users notice first
- what users notice later
- how information is grouped
- how information supports decision making

Hierarchy exists independently from layout, spacing, typography, or visual styling.

Every screen must follow this hierarchy before considering visual composition.

---

# Philosophy

Information hierarchy exists to reduce cognitive effort.

Users should never search for important financial information.

The interface should naturally guide attention toward the most important information first.

Hierarchy should remain stable across Desktop, Tablet, and Mobile.

Only composition changes.

Importance never changes.

---

# Information Pyramid

Pocket Mint organizes information into six levels.

Application

↓

Page

↓

Section

↓

Card

↓

Component

↓

Content

Each level supports the level above it.

---

# Level 1 — Application

Purpose

Defines the overall financial workspace.

Examples

Dashboard

Wallets

Transactions

Installments

Account

Landing

Users should immediately understand which part of the application they are currently viewing.

---

# Level 2 — Page

Each page answers one primary user goal.

Examples

Dashboard

Understand financial position.

Wallets

Manage financial assets.

Transactions

Review financial history.

Installments

Track financial obligations.

Landing

Introduce the product.

Pages should never attempt to solve multiple unrelated goals.

---

# Level 3 — Section

Sections divide the page into meaningful information groups.

Each section should answer one question.

Example

Dashboard

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

Avoid combining unrelated information inside the same section.

---

# Level 4 — Card

Cards organize related information.

A card should represent one logical idea.

Examples

Wallet Card

Installment Card

Financial Position Card

Transaction Card

Cards should never become miniature dashboards.

---

# Level 5 — Component

Components are reusable interface elements.

Examples

Button

Input

Badge

Alert

Progress Bar

Navigation Item

Statistic

Components should support cards.

Not compete with them.

---

# Level 6 — Content

Content is the actual information users consume.

Examples

Numbers

Labels

Dates

Descriptions

Currency

Status

Icons

Content is always the final layer.

Everything else exists to support content.

---

# Hierarchy Principles

Every screen should answer:

What is most important?

↓

What supports it?

↓

What should happen next?

Hierarchy should never feel random.

---

# Primary Information

Primary information is what users came to see.

Examples

Dashboard

Net Worth

Wallets

Current Balance

Transactions

Transaction List

Installments

Upcoming Payments

Landing

Product Value Proposition

Only one primary information block should exist per screen.

---

# Supporting Information

Supporting information explains the primary information.

Examples

Wallet Type

Reporting Cutoff

Transaction Category

Installment Due Date

Supporting information should never compete with primary information.

---

# Secondary Information

Secondary information provides context.

Examples

Notes

Descriptions

Institution Name

Reference Number

Secondary information should remain visually quiet.

---

# Navigation Information

Navigation is not content.

Navigation supports movement.

Navigation should remain visually consistent.

It should never compete with page information.

---

# Action Hierarchy

Actions follow the same hierarchy.

Primary Action

Most common user task.

Secondary Action

Useful but less frequent.

Contextual Action

Available only when needed.

Overflow Action

Rarely used actions.

Every screen should have one clear primary action.

---

# Financial Hierarchy

Financial information has its own priority.

Highest

Net Worth

↓

Assets

↓

Liabilities

↓

Income

↓

Expense

↓

History

Historical information should never dominate current financial position.

---

# Time Hierarchy

Current

↓

Upcoming

↓

Recent

↓

Historical

Pocket Mint prioritizes present awareness over historical exploration.

---

# Interaction Hierarchy

Read

↓

Understand

↓

Decide

↓

Act

↓

Explore

Users should never be forced to interact before understanding.

---

# Visual Reinforcement

Hierarchy should be reinforced through:

Composition

Grouping

Alignment

Scale

Whitespace

Not decorative styling.

---

# Progressive Disclosure

Users should receive information gradually.

Overview

↓

Summary

↓

Detail

↓

Management

↓

History

Avoid exposing deep information immediately.

---

# Cross Screen Hierarchy

Dashboard

↓

Wallets

↓

Transactions

↓

Installments

↓

Account

Users naturally move from awareness to management.

---

# Consistency

The same information should occupy similar hierarchy across all screens.

Example

Balance

Always primary.

Notes

Always secondary.

Metadata

Always supporting.

Consistency reduces learning effort.

---

# Anti Patterns

Do not:

- create multiple primary information blocks
- give secondary information equal emphasis
- overload a section with unrelated content
- create competing actions
- repeat identical information
- promote decorative elements above financial information

Information should always dominate decoration.

---

# Acceptance Criteria

Information hierarchy is successful when:

✓ Users understand what matters first.

✓ Every section has one purpose.

✓ One primary information block exists.

✓ Supporting information remains supporting.

✓ Actions follow clear priority.

✓ Navigation never competes with content.

✓ Financial awareness is always the primary outcome.

---

# Definition of Done

The hierarchy is complete when every future Pocket Mint screen can be mapped into:

Application

↓

Page

↓

Section

↓

Card

↓

Component

↓

Content

This document serves as the authoritative information architecture for the entire Pocket Mint product.