# Transactions Composition

Version: 1.0

Status: Draft

Owner: Pocket Mint

Category: Screen Composition

---

# Dependencies

Depends On

- transactions-specification.md
- design-philosophy.md
- design-principles.md
- information-hierarchy.md
- visual-language.md
- layout-spacing.md
- app-shell.md

---

# Purpose

This document defines the visual composition of the Transactions screen.

The composition should prioritize transaction readability before transaction management.

Users should immediately understand their financial history without feeling overwhelmed by data.

---

# Composition Philosophy

The Transactions screen is a chronological financial journal.

It should feel organized.

Structured.

Predictable.

Users should never feel like they are reading a spreadsheet.

History should remain readable.

Management tools should remain supportive.

---

# Reading Flow

Desktop

Search

↓

Filters

↓

Transaction List

↓

Transaction Detail (Optional)

↓

Contextual Actions

Mobile

Search

↓

Quick Filters

↓

Transaction List

↓

Transaction Detail

↓

Actions

Reading should always move from finding information toward understanding it.

---

# Visual Hierarchy

Highest

Transaction List

↓

High

Search

↓

High

Filters

↓

Medium

Transaction Detail

↓

Low

Supporting Information

The transaction list is always the visual anchor.

---

# Visual Weight

Transaction List

★★★★★

Search

★★★★☆

Filters

★★★☆☆

Transaction Detail

★★★☆☆

Supporting Information

★★☆☆☆

Management controls should never dominate transaction history.

---

# Focal Point

The transaction list is the primary focus.

Users came to review financial activity.

Everything else supports that task.

Search and filters should assist.

Not compete.

---

# Desktop Composition

Desktop prioritizes efficient review.

Recommended layout

Search + Filters

↓

Transaction List

↓

Optional Detail Panel

↓

Contextual Actions

Transaction Detail should remain secondary.

Avoid permanently displaying an empty detail panel.

---

# Tablet Composition

Tablet reduces simultaneous information.

Filters may collapse.

Transaction Detail may appear as a slide-over panel.

Maintain transaction readability.

---

# Mobile Composition

Mobile prioritizes fast scanning.

Search

↓

Quick Filters

↓

Transaction List

↓

Transaction Detail

↓

Actions

Scrolling should feel effortless.

Avoid large filter interfaces.

---

# Section Relationships

Search

supports

Filters

Filters

support

Transaction List

Transaction List

supports

Transaction Detail

Transaction Detail

supports

Management Actions

Every section exists to improve transaction understanding.

---

# Progressive Disclosure

Users should discover information gradually.

Transaction List

↓

Transaction Preview

↓

Transaction Detail

↓

Edit

↓

Management

Do not expose editing controls before users understand the transaction.

---

# Information Density

Transactions have the highest information density in Pocket Mint.

However, density should never reduce readability.

Rows should remain comfortable to scan.

Whitespace should separate transactions naturally.

---

# Content Balance

The Transaction List occupies most of the page.

Search and Filters should consume minimal visual attention.

Transaction Detail should remain supportive.

Avoid oversized filter panels.

---

# Empty Space Strategy

Whitespace separates groups.

Not rows.

Transaction rows should feel compact but breathable.

Avoid excessive spacing between list items.

Avoid large unused page regions.

---

# Component Relationships

Search

↓

Filters

↓

Transaction List

↓

Transaction Detail

↓

Contextual Actions

Every component contributes to reviewing financial history.

---

# Visual Rhythm

Find

↓

Filter

↓

Review

↓

Understand

↓

Manage

Users should never feel interrupted by the interface.

---

# Emphasis Rules

Visual emphasis should come from:

Hierarchy

Grouping

Typography

Composition

Never through:

Large icons

Decorative backgrounds

Heavy shadows

Bright colors

Financial values should always be the strongest element within each transaction row.

---

# Responsive Consistency

Desktop

Tablet

Mobile

should all communicate the same financial history.

Only layout changes.

The review workflow remains identical.

---

# Transaction Row Composition

Each row should contain:

Primary

Transaction Title

Amount

Secondary

Wallet

Category

Supporting

Date

Time

Status

Rows should remain immediately scannable.

---

# Transaction Detail Composition

The detail view should reveal:

Amount

↓

Wallet

↓

Category

↓

Date

↓

Notes

↓

Attachments

↓

Management Actions

Important financial information should always appear first.

---

# Search Composition

Search should remain lightweight.

It should never dominate the page.

Placeholder text should clearly communicate its purpose.

Avoid oversized search bars.

---

# Filter Composition

Filters should remain secondary.

Frequently used filters should appear first.

Advanced filters should remain hidden until requested.

Filtering should simplify.

Not complicate.

---

# Empty State Composition

When no transactions exist:

Display:

Friendly illustration (optional)

Short explanation

Primary CTA

Avoid large decorative empty states.

Encourage users to record their first transaction.

---

# Loading Composition

Preserve the transaction list layout.

Use skeleton rows.

Avoid full-page spinners.

Loading should feel progressive.

---

# AI Generation Notes

Google Stitch should prioritize:

- Readable transaction rows
- Consistent spacing
- Calm financial appearance
- Efficient scanning
- Professional data presentation

Google Stitch should avoid:

- Spreadsheet layouts
- Decorative charts
- Analytics-first interfaces
- Crypto exchange aesthetics
- ERP-style dense tables
- Oversized filter panels
- Visual clutter

The screen should inherit every Pocket Mint foundation document.

---

# Anti Patterns

Do not:

Turn transactions into analytics.

Use cards with inconsistent heights.

Mix unrelated controls into the transaction list.

Allow filters to dominate.

Create oversized empty states.

Display fake charts.

Use decorative KPI widgets.

Compete with Dashboard hierarchy.

---

# Acceptance Criteria

The composition is successful when:

✓ Transaction history is immediately recognizable.

✓ Search feels lightweight.

✓ Filters support rather than dominate.

✓ Rows remain easy to scan.

✓ Desktop feels efficient.

✓ Mobile feels comfortable.

✓ Financial values remain visually dominant.

✓ Users understand history before managing it.

---

# Definition of Done

The Transactions composition is complete when:

✓ Reading flow is obvious.

✓ Transaction List becomes the visual anchor.

✓ Search and Filters remain secondary.

✓ Detail views support rather than compete.

✓ Responsive behaviour preserves hierarchy.

✓ Composition follows the Pocket Mint Design System.

This document serves as the authoritative composition reference for the Transactions screen.