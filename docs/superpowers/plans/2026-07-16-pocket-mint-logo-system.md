# Pocket Mint Final Logo System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Package the frozen Pocket Fold geometry as production SVG assets and document the complete Pocket Mint logo system without modifying application code.

**Architecture:** Store all final logo assets together under `docs/product/stictch/core/logo/`. Reuse the exact approved path in every asset, vary only the declared color or lockup context, and keep the wordmark as live Inter SemiBold text.

**Tech Stack:** SVG 1.1-compatible XML, Markdown, React/TypeScript implementation examples, Sharp for rasterized verification only.

## Global Constraints

- The approved path data, proportions, terminals, countershape, radii, padding, and `viewBox="0 0 24 24"` must not change.
- Production assets contain no raster data, gradients, shadows, masks, clipping paths, filters, opacity effects, embedded fonts, or decorative effects.
- Do not modify Pocket Mint application code.
- Use `currentColor` for the master icon and lockup mark.
- Keep the wordmark as live `Inter` text at weight `600`; do not convert it to outlines.

---

### Task 1: Package the icon and favicon assets

**Files:**
- Create: `docs/product/stictch/core/logo/pocket-fold.svg`
- Create: `docs/product/stictch/core/logo/pocket-fold-black.svg`
- Create: `docs/product/stictch/core/logo/pocket-fold-white.svg`
- Create: `docs/product/stictch/core/logo/favicon.svg`

**Interfaces:**
- Consumes: approved Pocket Fold path from `docs/product/stictch/core/pocket-fold-validation.svg`
- Produces: current-color, fixed black, fixed white, and fixed-Slate favicon SVG assets

- [ ] **Step 1:** Create four standalone SVG files using the exact approved `d` attribute and `viewBox="0 0 24 24"`.
- [ ] **Step 2:** Set fills to `currentColor`, `#000000`, `#ffffff`, and `#0F172A` respectively.
- [ ] **Step 3:** Parse each file as XML and confirm each contains exactly one path.
- [ ] **Step 4:** Render each icon at 16px, 20px, 24px, 32px, 48px, and 64px; confirm the countershape remains open.

### Task 2: Create the horizontal lockup

**Files:**
- Create: `docs/product/stictch/core/logo/pocket-mint-lockup.svg`

**Interfaces:**
- Consumes: approved Pocket Fold path and Inter SemiBold typography rule
- Produces: one horizontal, live-text logo lockup

- [ ] **Step 1:** Place the unchanged mark in a nested 32px SVG using its original 24-unit viewBox.
- [ ] **Step 2:** Place live text `Pocket Mint` at 24px, weight 600, with an 8px optical gap.
- [ ] **Step 3:** Render the lockup and inspect baseline, apparent weight, and clipping.

### Task 3: Document the logo system

**Files:**
- Create: `docs/product/stictch/core/logo/logo-guidelines.md`

**Interfaces:**
- Consumes: packaged assets and authoritative brand tokens
- Produces: production rules and React implementation guidance

- [ ] **Step 1:** Document asset inventory, lockup ratio, clear-space unit, minimum sizes, backgrounds, and incorrect usage.
- [ ] **Step 2:** Document the 20px, 24px, 32px, 48px, and 64px application scale.
- [ ] **Step 3:** Include React examples for icon-only and horizontal lockup components without modifying application code.

### Task 4: Verify the final package

**Files:**
- Verify: `docs/product/stictch/core/logo/*.svg`
- Verify: `docs/product/stictch/core/logo/logo-guidelines.md`

**Interfaces:**
- Consumes: all final logo-system files
- Produces: evidence that the frozen geometry and technical constraints are preserved

- [ ] **Step 1:** Compare each SVG path against the approved `d` attribute byte-for-byte.
- [ ] **Step 2:** scan for prohibited SVG elements and effects.
- [ ] **Step 3:** transpile the documented JSX examples and require zero diagnostics.
- [ ] **Step 4:** render all SVG assets and confirm successful parsing, declared dimensions, and open countershapes.
