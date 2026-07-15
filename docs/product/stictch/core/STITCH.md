# Pocket Mint — Stitch Generation Contract

`DESIGN.md` version 2.0 is authoritative. This file is a concise generation contract and does not replace it. If any instruction conflicts, follow `DESIGN.md`.

## 1. Product Identity

Generate Pocket Mint as a calm, professional, trustworthy, organized, private, information-first financial workspace. It is not an admin dashboard, bank, crypto or trading product, gamified tracker, or marketing-heavy fintech site.

## 2. Visual Constraints

Use an off-white canvas, white card surfaces, restrained Slate interface elements, minimal elevation, 12px default radius, 16px large-container radius, 24px card padding, 32px major section spacing, and an 8px rhythm. Premium quality comes from clarity and restraint.

## 3. Mandatory Navigation Labels

The application language is Bahasa Indonesia. Use exactly: `Dashboard`, `Dompet`, `Transaksi`, `Cicilan`, `Akun`. Never translate or rename them.

## 4. Layout Rules

Desktop is the master composition: persistent sidebar, page header, then content in a 12-column grid within a 1280px maximum width and 40px safe area. Tablet uses compact navigation, an 8-column grid, and 32px safe area. Mobile uses a top app bar, a single reading column within a 4-column grid, 20px horizontal margins, and bottom navigation. Preserve the specified screen reading order.

## 5. Typography Rules

Use Inter only. Hierarchy is financial values, section titles, body, then metadata. Keep headings controlled and application-scaled; never use oversized marketing typography or a competing display/monospace font.

## 6. Financial-Number Rules

Apply tabular figures to every financial value. Use Indonesian locale formatting and preserve complete amounts. Long values may resize or wrap as one unit but may not be misleadingly abbreviated or truncated. Negative debt or asset balances use a minus sign and explicit Coral context. Ordinary expenses remain dark neutral. Zero remains neutral.

## 7. Semantic Color Rules

Mint means assets, income, and positive financial meaning. Slate means navigation, neutral emphasis, and solid primary actions. Coral means debt, destructive actions, overdue obligations, negative balances, or critical states. Amber means warning, upcoming, pending, or attention. Never use color as the only cue and never assign arbitrary colors.

## 8. Shared Component Rules

Use one visual definition for Hero Card, Wallet Card, Summary Card, Quick Action, Activity Row, Installment Card, buttons, inputs, and section headers at every breakpoint. The Hero Card contains only Net Worth, Total Assets, Total Outstanding Debt, and Reporting Cutoff. Wallet balance is dominant. Buttons are compact: solid Slate primary, outlined secondary, Coral danger. Cards are white with minimal shadow. Wallet accents must be functional, never mandatory decorative side stripes.

## 9. Responsive Rules

Change only width, columns, wrapping, and stacking. Do not change component styling, content, typography hierarchy, radius, elevation, color meaning, spacing scale, icon style, or interactions. Keep the page free of horizontal overflow. A labelled table region may scroll horizontally on narrow screens while retaining semantic headers and row styling.

## 10. Accessibility Rules

Maintain sufficient contrast, logical keyboard order, native control semantics, visible focus, 44px minimum touch targets, and non-color state cues. Modals require an accessible name, focus trap, inert background, Escape close when dismissible, and trigger-focus restoration. Mobile modals keep the desktop visual definition while adapting width, wrapping, action stacking, and internal scrolling.

## 11. Prohibited Patterns

No glassmorphism, gradients, decorative blobs, bento marketing layouts, fake charts, fake sparklines, fabricated insights, ornamental graphics, excessive shadows, oversized hero typography, decorative card side stripes, arbitrary colors, English navigation, generic dashboard composition, or multiple visual directions.

## 12. Screen-Generation Rules

Preserve the authoritative information architecture and content order. Define truthful hover, focus-visible, pressed, selected, disabled, loading, skeleton, empty, error, destructive-confirmation, overdue, upcoming-payment, and no-search-result states. Never represent loading or failure as zero or empty data.

## 13. One Direction Only

Generate one production-ready Pocket Mint direction. Do not provide stylistic alternatives, concepts, experiments, or breakpoint-specific component redesigns.

## 14. No Invention

Do not invent product features, actions, charts, sections, financial data, trends, comparisons, insights, or placeholder branding. Use only the fields and behavior defined by `DESIGN.md` and the supplied screen specification.
