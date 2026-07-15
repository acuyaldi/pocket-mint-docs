# Dashboard Composition

Version: 1.0

Status: Draft

Owner: Pocket Mint

Category: Product Composition Specification

Last Updated: 2026

---

# Dependencies

This document depends on:

- dashboard-specification.md
- design-principles.md
- layout-spacing.md
- app-shell.md

---

# 1. Purpose

This document defines how the Dashboard should be visually composed.

It specifies:

- reading flow
- visual hierarchy
- visual weight
- content balance
- progressive disclosure
- composition rules

This document intentionally does not define:

- spacing values
- typography scale
- colors
- business logic

Those belong to their respective specifications.

---

# 2. Composition Philosophy

The Dashboard is not an information dump.

The Dashboard is a carefully composed financial workspace.

Composition should reduce thinking.

Users should understand their financial condition before consciously scanning the interface.

Every composition decision should support confidence.

Never curiosity.

---

# 3. Reading Flow

The Dashboard should naturally guide the user's eyes.

Reading order:

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

The reading flow should feel effortless.

Users should never wonder where to look first.

---

# 4. Visual Hierarchy

Hierarchy is determined by visual emphasis.

Highest

Financial Position

↓

High

Needs Attention

↓

Medium

Quick Actions

↓

Medium

Wallet Overview

↓

Low

Current Period Summary

↓

Lowest

Recent Activity

No lower section should visually compete with a higher section.

---

# 5. Visual Weight

Every section has a visual weight.

Financial Position

★★★★★

Needs Attention

★★★★☆

Quick Actions

★★★☆☆

Wallet Overview

★★★☆☆

Current Period Summary

★★☆☆☆

Recent Activity

★★☆☆☆

Visual weight should decrease gradually.

Never abruptly.

---

# 6. Eye Movement

Desktop should encourage a vertical scanning rhythm.

Users should never zig-zag across unrelated content.

Preferred flow:

Hero

↓

Attention

↓

Actions

↓

Wallet

↓

Summary

↓

Recent

Avoid compositions that require excessive horizontal scanning.

---

# 7. Progressive Disclosure

The Dashboard provides summaries.

Detail belongs elsewhere.

Example:

Wallet Overview

↓

Wallet Page

Recent Activity

↓

Transactions

Needs Attention

↓

Installments

Users should move deeper only when necessary.

---

# 8. Information Density

The Dashboard should feel balanced.

Not sparse.

Not crowded.

Whitespace exists to improve understanding.

Not decoration.

---

# 9. Visual Balance

The page should feel visually stable.

Large sections should be balanced by lighter sections.

Avoid placing multiple visually dominant sections next to each other.

The Financial Position should remain the primary anchor.

---

# 10. Empty Space Strategy

Whitespace is intentional.

Increase whitespace around major layout regions.

Do not increase whitespace inside closely related information.

Preferred:

More space

between sections.

Less space

between related labels and values.

---

# 11. Focal Point

The Financial Position card is always the focal point.

It should attract attention immediately after the page loads.

No other component should compete for this role.

---

# 12. Hero Composition

The Hero should communicate:

Current Position

↓

Supporting Metrics

↓

Reporting Context

This creates a natural top-down reading sequence.

Avoid fragmented layouts.

Avoid multiple competing values.

---

# 13. Section Relationships

Each section supports the previous one.

Financial Position

↓

Needs Attention

"What requires action?"

↓

Quick Actions

"What can I do?"

↓

Wallet Overview

"Where is my money?"

↓

Current Period Summary

"What happened this period?"

↓

Recent Activity

"What happened most recently?"

Each section answers the next logical question.

---

# 14. Desktop Composition

The desktop Dashboard should prioritize clarity over density.

Large displays should not be filled simply because additional space is available.

Instead, use whitespace to improve structure.

The Dashboard should feel balanced across the available width.

Avoid creating isolated content islands.

---

## Desktop Layout Philosophy

Desktop is designed for scanning.

Users should understand their financial position within one visual pass.

Scrolling is acceptable but should not be immediately required to understand the current financial state.

---

## Recommended Composition

Financial Position

↓

Needs Attention + Quick Actions

↓

Wallet Overview

↓

Current Period Summary + Recent Activity

The top half of the page should communicate the user's current financial situation.

The lower half provides supporting information.

---

## Desktop Balance

Financial Position should visually dominate.

Needs Attention should immediately support it.

Wallet Overview should occupy more visual presence than Current Period Summary.

Recent Activity should visually conclude the page.

No section should appear isolated.

---

# 15. Tablet Composition

Tablet layouts should preserve desktop hierarchy while simplifying composition.

Reduce simultaneous columns.

Increase readability.

Avoid forcing desktop density.

---

## Tablet Reading Flow

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

Tablet favors sequential reading over parallel comparison.

---

## Tablet Visual Weight

Financial Position remains dominant.

Wallet Overview becomes more prominent due to reduced width.

Current Period Summary and Recent Activity should remain secondary.

---

# 16. Mobile Composition

Mobile prioritizes focus.

Only one primary section should compete for attention at any moment.

The Dashboard becomes a narrative.

Each section answers one question before introducing the next.

---

## Mobile Reading Flow

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

Never change this order.

---

## Mobile Philosophy

Scrolling is expected.

Compression is discouraged.

Users should feel guided through the page.

Not overwhelmed.

---

# 17. Section Proportions

Visual emphasis should follow importance.

Approximate proportion:

Financial Position

35%

Needs Attention

15%

Quick Actions

10%

Wallet Overview

20%

Current Period Summary

10%

Recent Activity

10%

These percentages describe visual importance rather than physical height.

---

# 18. Visual Rhythm

Every section should feel like part of the same page.

Visual rhythm should alternate between:

Primary

↓

Supporting

↓

Action

↓

Supporting

↓

Summary

↓

History

Avoid repeating multiple visually heavy sections consecutively.

The page should breathe naturally.

---

# 19. Content Balance

Avoid situations where one side of the page feels empty while the other feels overloaded.

Balance should be achieved through:

- proportion
- grouping
- emphasis

Never through decorative elements.

Whitespace should support balance rather than compensate for poor composition.

---

# 20. Component Relationships

Components should reinforce each other.

Financial Position

supports

Needs Attention

Needs Attention

supports

Quick Actions

Wallet Overview

supports

Current Period Summary

Recent Activity

supports

Transactions

Every section should feel connected to the next.

---

# 21. Composition Consistency

Desktop

Tablet

Mobile

should all communicate the same story.

Only the arrangement changes.

Never the hierarchy.

Never the intent.

---

# 22. Scanability

Users should understand the Dashboard through quick scanning.

The page should communicate:

Current Position

↓

Urgency

↓

Available Actions

↓

Asset Distribution

↓

Period Summary

↓

Recent History

Scanning should feel natural without requiring deliberate searching.

---

# 23. Cognitive Load

Reduce decision making.

The Dashboard should answer questions.

Not create new ones.

Avoid placing multiple competing calls-to-action beside each other.

Avoid multiple primary focal points.

One page.

One story.

One primary message.

---

# 24. Visual Emphasis Rules

Emphasis should be created through:

- composition
- hierarchy
- size
- grouping

Avoid relying on:

- saturated colors
- oversized icons
- decorative illustrations
- excessive elevation

Information should remain the focus.

---

# 25. Progressive Focus

As users move down the page:

Information importance decreases.

Interaction complexity increases.

Detail increases.

Visual emphasis decreases.

This creates a natural progression from awareness to exploration.

---

# 26. Anti-Patterns

Avoid:

- two equally dominant hero sections
- oversized Quick Actions
- oversized Wallet cards
- decorative empty space
- analytics-first layouts
- marketing banners
- unrelated widgets
- promotional content
- visual clutter
- inconsistent section proportions

The Dashboard should always feel intentional.

---

# 27. Acceptance Criteria

The Dashboard composition is considered successful when:

✓ The Financial Position is immediately recognized as the primary focal point.

✓ Users naturally scan the page from top to bottom.

✓ Every section feels connected to the next.

✓ Supporting information never competes with primary information.

✓ Desktop feels balanced rather than empty.

✓ Mobile feels guided rather than compressed.

✓ Users understand their financial situation before interacting with the interface.

✓ The composition reinforces Pocket Mint's identity as a Private Financial Workspace.

---

# 28. Definition of Done

The Dashboard Composition is complete when:

- reading flow is unambiguous
- hierarchy is visually consistent
- visual weight matches business importance
- desktop, tablet, and mobile share the same narrative
- composition supports the Dashboard Specification
- composition respects the global Design Principles
- composition follows the Layout & Spacing Specification

This document is the authoritative composition reference for every future Dashboard iteration generated in Google Stitch or implemented in the frontend.