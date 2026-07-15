# Pocket Mint — Stitch Master Prompt

You are designing Pocket Mint, a privacy-first personal financial workspace.

Pocket Mint helps users understand:

- what they own
- what they owe
- what requires attention
- what happened recently

The product is already a working MVP. This project is a coherent UI/UX redesign,
not a concept for a new product.

Follow the attached DESIGN.md and screen brief as the source of truth.

## Design Direction

Create a calm, professional, light financial workspace.

Prioritize:

1. financial clarity
2. information hierarchy
3. readable financial numbers
4. predictable navigation
5. responsive behavior
6. honest application states
7. accessible interactions

Use restrained visual styling.

Prefer:

- typography over decoration
- spacing over excessive borders
- clear grouping over many nested cards
- semantic states over decorative colors
- reusable patterns over one-off components

Avoid:

- glassmorphism-heavy surfaces
- excessive gradients
- crypto-dashboard styling
- banking-card metaphors
- gamification
- decorative graphs
- fake analytics
- synthetic historical data
- excessive uppercase microcopy
- oversized welcome banners
- generic admin-dashboard layouts

## Financial Integrity

Never fabricate:

- financial trends
- growth percentages
- historical charts
- payment status
- installment progress
- financial advice

Net Worth is the primary financial metric.

Net Worth equals:

Total Assets minus Total Outstanding Debt.

Always display Total Assets and Total Outstanding Debt as supporting values.

Negative Net Worth is valid and must remain visible.

A current-state screen must not look like historical analytics.

## Responsive Grid

Desktop:

- 12-column grid
- 1280px maximum content width
- 40px outer margin where space permits

Tablet:

- 8-column grid
- tablet-specific navigation and composition
- do not simply compress desktop layouts

Mobile:

- 4-column grid
- 16px page margin
- no horizontal dashboard scrolling
- minimum comfortable touch targets
- primary financial numbers remain readable

## Typography

Use Inter.

Use tabular figures for:

- currency
- percentages
- balances
- debt values
- installment values

Financial values should be easy to compare vertically.

## Semantic Color Direction

Mint:

- brand
- primary interaction
- positive financial values

Slate:

- text
- navigation
- neutral information
- Transfer

Coral red:

- Expenses
- debt
- errors
- destructive actions

Amber:

- warning
- upcoming obligations
- attention states

Do not communicate meaning using color alone.

## Deliverables

Generate:

- production-quality desktop screen
- tablet adaptation
- mobile adaptation
- important loading, empty, partial-error, and unavailable states

Keep the design feasible for implementation in an existing Next.js frontend.

Do not invent unsupported features.