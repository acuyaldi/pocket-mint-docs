# Pocket Mint — Layout & Spacing

> Defines the spatial hierarchy used throughout Pocket Mint.

This document focuses on **layout relationships**, not individual component design.

---

# 1. Philosophy

Pocket Mint is a calm, information-first financial workspace.

Whitespace is used to improve readability, not to make the interface look empty.

The goal is:

- clear hierarchy
- comfortable scanning
- balanced density
- consistent rhythm

Scrolling is acceptable.

Empty layouts are not.

Dense layouts are not.

---

# 2. Layout Priority

When increasing spacing, always follow this order.

1. Page Safe Area

2. Layout Gutter

3. Section Separation

4. Card Padding

5. Component Padding

6. Component Gap

Never increase lower-priority spacing before higher-priority spacing.

Example

✔ Increase the distance between the sidebar and the dashboard.

✖ Increase the spacing between navigation items.

---

# 3. Layout Regions

Pocket Mint is divided into large layout regions.

Browser

↓

Application Frame

↓

Sidebar

↓

Main Content

↓

Sections

↓

Cards

↓

Components

Each level should be visually separated before increasing spacing inside the next level.

---

# 4. Page Safe Area

Desktop

Top

40px

Left

40px

Right

40px

Bottom

40px

Tablet

32px

Mobile

20px horizontal

24px below the app bar

32px bottom spacing

The application should never feel attached to the screen edges.

---

# 5. Layout Gutter

Spacing between major layout regions.

Desktop

Sidebar → Content

32px

Grid Gap

24px

Section Gap

32px

Mobile

Section Gap

24–32px

Increase layout separation before increasing component spacing.

---

# 6. Sidebar Density

The sidebar should feel compact.

Navigation belongs to one visual group.

Increase spacing around the sidebar.

Do not increase spacing inside the navigation.

Preferred:

Logo

↓

24px

↓

Navigation

Dashboard

Dompet

Transaksi

Cicilan

Akun

Avoid:

Logo

↓

48px

↓

Dashboard

↓

24px

↓

Dompet

↓

24px

↓

Transaksi

Navigation should resemble:

- Linear
- Notion
- GitHub
- Figma

Compact.

Readable.

Organized.

---

# 7. Section Rhythm

Major dashboard sections should breathe.

Financial Position

↓

32px

Needs Attention

↓

32px

Quick Actions

↓

32px

Wallet Overview

↓

32px

Current Period Summary

↓

32px

Recent Activity

Maintain a consistent rhythm throughout the page.

---

# 8. Card Density

Increase padding.

Not gaps.

Preferred:

Card Edge

↓

24px padding

↓

Title

↓

8px

↓

Value

↓

16px

↓

Content

Avoid increasing the distance between related information.

Related information should remain visually grouped.

---

# 9. Component Density

Components should remain compact.

Examples:

Navigation

Icon ↔ Label

12px

Button

Icon ↔ Label

8px

Wallet Card

Title ↔ Balance

12px

Transaction

Title ↔ Metadata

4px

Component spacing should support grouping.

Never create empty-looking components.

---

# 10. Responsive Rules

Desktop

Use whitespace for structure.

Tablet

Preserve desktop hierarchy while adapting composition.

Mobile

Prioritize readability over density.

Scrolling is preferred over reducing padding.

Do not compress cards to fit more content above the fold.

---

# 11. Dashboard Rules

Dashboard hierarchy is fixed.

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

Spacing may change.

Hierarchy may not.

---

# 12. Things To Avoid

Do not:

- attach content to screen edges
- create oversized empty areas
- increase navigation spacing excessively
- compress cards
- reduce touch targets
- reduce card padding
- mix unrelated sections
- use inconsistent spacing
- increase every spacing value equally

Layout should become more spacious.

Components should remain efficient.

---

# 13. Acceptance Criteria

A layout is considered successful when:

✓ Sidebar feels compact but not crowded.

✓ Main content has clear breathing room.

✓ Dashboard sections are visually separated.

✓ Cards feel comfortable.

✓ Navigation feels grouped.

✓ Mobile feels spacious without looking empty.

✓ The interface resembles a professional productivity application.

The user should immediately perceive the layout as:

calm

organized

balanced

professional

without noticing the spacing itself.