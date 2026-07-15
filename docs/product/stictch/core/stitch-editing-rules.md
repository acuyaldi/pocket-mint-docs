# Pocket Mint — Google Stitch Editing Rules

> This document defines how Google Stitch should revise Pocket Mint screens throughout the redesign process.

This document is not about UI design.

It defines the expected editing workflow.

The goal is to ensure every iteration is focused, incremental, and consistent.

---

# 1. Primary Principle

Google Stitch is expected to behave as a design collaborator, not as an idea generator.

Unless explicitly instructed otherwise:

- modify existing work
- preserve existing work
- refine existing work

Do not redesign.

---

# 2. Default Behavior

Every prompt should be interpreted as:

> Improve the currently selected screen.

NOT as:

> Generate another design proposal.

---

# 3. Screen Ownership

Each screen has a single source of truth.

Examples:

- Application Shell
- Dashboard
- Wallets
- Transactions
- Installments
- Landing Page

Do not create duplicate versions of these screens unless explicitly requested.

Avoid screens named:

- Dashboard V2
- Dashboard Final
- Dashboard Alternative
- Dashboard Spacing
- Dashboard Revised

Continue refining the existing screen instead.

---

# 4. Revision Philosophy

Pocket Mint uses iterative refinement.

Each revision should improve one aspect of the design.

Examples:

✔ Sidebar spacing

✔ Card padding

✔ Typography rhythm

✔ Visual hierarchy

✔ Responsive behavior

Avoid combining many unrelated revisions into one iteration.

---

# 5. Scope Control

When revising a screen:

Only modify what is explicitly requested.

Preserve everything else.

For example:

If asked to improve spacing:

Do not change:

- colors
- typography
- hierarchy
- navigation
- icons
- components
- layout structure

Only adjust spacing.

---

# 6. Do Not Redesign

Unless explicitly instructed:

Do NOT:

- redesign layouts
- invent new sections
- add analytics
- introduce charts
- create alternative navigation
- replace components
- move information hierarchy
- simplify content
- rename screens

Pocket Mint already has an approved product direction.

---

# 7. Preserve Existing Structure

Always preserve:

- page hierarchy
- navigation
- component structure
- interaction model
- information architecture

Only improve the requested visual aspect.

---

# 8. One Revision = One Goal

Each iteration should have exactly one design objective.

Examples:

Iteration 1

Improve sidebar spacing.

Iteration 2

Improve card padding.

Iteration 3

Improve mobile safe areas.

Iteration 4

Improve typography hierarchy.

Never attempt multiple redesigns in one revision.

---

# 9. Incremental Editing

Think of each revision as editing an existing Figma file.

Not creating a new concept.

Do not rebuild the screen.

Refine it.

---

# 10. Existing Components First

Before introducing a new component:

Ask:

Can the existing component be improved instead?

Prefer refinement over replacement.

---

# 11. Respect Existing Decisions

Never override previously approved decisions.

These include:

- navigation labels
- page hierarchy
- dashboard philosophy
- design language
- semantic colors
- typography system
- spacing system

If a previous document defines the behavior, follow it.

---

# 12. Layout Changes

If the request concerns layout:

Modify only:

- page padding
- margins
- spacing
- gutters
- alignment
- card padding

Do not rearrange sections unless explicitly requested.

---

# 13. Visual Changes

If the request concerns visual styling:

Modify only:

- contrast
- elevation
- shadows
- borders
- corner radius

Do not alter layout.

---

# 14. Responsive Changes

If the request concerns responsiveness:

Modify only the requested breakpoint.

Desktop revisions should not unintentionally alter mobile.

Mobile revisions should not unintentionally alter desktop.

---

# 15. Design Consistency

Every revision must preserve consistency with:

- Design System
- Design Principles
- Layout & Spacing Specification
- Product RFC
- Screen Specifications

Never contradict those documents.

---

# 16. No Additional Screens

Do not generate:

- additional dashboard pages
- empty placeholder pages
- duplicate screens
- exploration boards
- comparison layouts

The output should update the current screen only.

---

# 17. Output Expectations

For every revision:

✔ Update the existing screen.

✔ Keep the screen name.

✔ Preserve component identity.

✔ Preserve hierarchy.

✔ Preserve visual language.

Only modify the requested elements.

---

# 18. Preferred Editing Workflow

Pocket Mint follows this workflow:

Application Shell

↓

Freeze

↓

Dashboard

↓

Freeze

↓

Wallets

↓

Freeze

↓

Transactions

↓

Freeze

↓

Installments

↓

Freeze

↓

Landing Page

↓

Implementation

Frozen screens should only receive bug fixes or explicit revision requests.

---

# 19. Editing Priority

When multiple improvements are possible, prioritize:

1. Information hierarchy

2. Layout

3. Spacing

4. Typography

5. Component consistency

6. Responsive behavior

7. Visual polish

Never reverse this order.

---

# 20. Pocket Mint Design Philosophy

Pocket Mint is a Private Financial Workspace.

Every revision should move the interface closer to:

- calm
- spacious
- professional
- trustworthy
- information-first
- privacy-first

Never toward:

- marketing landing pages
- crypto dashboards
- gaming interfaces
- decorative analytics
- visual clutter

---

# 21. Success Criteria

A revision is considered successful when:

- only the requested area has changed
- no duplicate screens are created
- the existing screen has been improved
- the approved hierarchy remains unchanged
- visual consistency is preserved
- the revision feels incremental rather than exploratory

The best revision is one that users immediately recognize as the same screen—only better.