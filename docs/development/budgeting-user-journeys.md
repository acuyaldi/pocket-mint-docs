# Pocket Mint Budgeting — MVP User Journeys

> Status: Finalized design, **not implemented**. Governed by [PD-009](../product/decisions/009-budgeting-scope.md) (Approved), the [Budgeting API Contract](./budgeting-api-contract.md), and the [Phase A.5 Design Review](./budgeting-phase-a5-design-review.md). No AI, smart categorization, forecasting, or notification behavior is included — those remain out of scope until their own phases.

Each journey below assumes the finalized screens: **Budgets** (list, with Archived toggle), **Budget Detail**, **Create Budget** (modal), **Edit Budget** (modal). All backend calls reference [budgeting-api-contract.md](./budgeting-api-contract.md).

---

## 1. First-time user with no Budget

- **Entry point:** Budgets, from primary navigation.
- **Visible information:** Empty state — "No budgets yet" copy, a Create Budget call to action. No fabricated Budget Status or zero-value cards.
- **User action:** none required to reach this state; may tap Create Budget.
- **Backend operation:** `GET /budgets?status=active` returns `[]`.
- **Success behavior:** empty state renders; Archived toggle still present but inert until an archived Budget exists.
- **Failure behavior:** see Journey 14 (API/network failure).
- **Accessibility:** empty-state region announced via a heading, not solely an icon; Create Budget button is the first focusable element after the page heading.

## 2. Creating a Budget

- **Entry point:** Budgets → Create Budget button opens the Create Budget modal.
- **Visible information:** Category selector (expense categories with no existing Budget, active or archived), amount input, submit/cancel.
- **User action:** selects a Category, enters an amount, confirms.
- **Backend operation:** `POST /budgets { categoryId, amount }`.
- **Success behavior:** modal closes; `["budgets"]` and `["dashboard"]` React Query keys invalidated (`invalidateBudgetDependents`); the Budgets list now includes the new Budget at `spent = 0`, `HEALTHY`. Toast confirms creation (`t("budgetModals.create.success")`).
- **Failure behavior:** `422 INVALID_AMOUNT` → inline error under the amount field, entered value preserved. `404 CATEGORY_NOT_FOUND` → should not occur given client-side filtering, but if the category list is stale, show a generic retry error and refresh the category list. Modal never closes on error.
- **Accessibility:** `FormField` wires `aria-invalid`/`aria-describedby` to the amount input on error; focus moves to the first invalid field on submit failure.

## 3. Attempting to create a duplicate Budget

- **Entry point:** Create Budget modal, category list is stale (e.g. two browser tabs) or a race between two submits.
- **Visible information:** submit is attempted for a category that already has a Budget.
- **User action:** confirms creation.
- **Backend operation:** `POST /budgets` → `409 BUDGET_ALREADY_EXISTS`.
- **Success behavior:** n/a.
- **Failure behavior:** inline error: "This category already has a budget." Modal stays open; entered amount preserved; a "View existing budget" link is not required for MVP (user can cancel and find it via Budgets/Archived toggle).
- **Accessibility:** error announced via `role="alert"` (`FormErrorMessage`).

## 4. Viewing a healthy Budget

- **Entry point:** Budgets list or Budget Detail.
- **Visible information:** Category, Budget Limit, Budget Spent, Budget Remaining, status pill "Healthy" (green), progress bar width = `percentUsed`.
- **User action:** none (read).
- **Backend operation:** `GET /budgets?status=active` or `GET /budgets/:id`, `percentUsed < 75`.
- **Success behavior:** figures render as returned; no client-side recomputation.
- **Failure behavior:** n/a.
- **Accessibility:** progress bar has `role="progressbar"`, `aria-valuenow={percentUsed}`, `aria-valuemin=0`, `aria-valuemax=100`, `aria-label` including the status text (e.g. "Makan budget, 42% used, Healthy") — not color alone.

## 5. Viewing a Budget at exactly 75%

- **Entry point:** Budgets list or Budget Detail.
- **Visible information:** status pill "Approaching" (amber), `percentUsed = 75` displayed as "75%".
- **User action:** none.
- **Backend operation:** same as above; `status: "APPROACHING"` returned directly by the backend — the frontend does not compute the 75% boundary itself.
- **Success behavior:** boundary is inclusive at exactly 75% (per the calculation spec's boundary table) — this is asserted by the existing backend unit tests (`test/budget.test.ts`), not re-derived by the frontend.
- **Failure behavior:** n/a.
- **Accessibility:** same as Journey 4, with "Approaching" in the accessible name.

## 6. Viewing a Budget at exactly 100%

- **Entry point:** Budgets list or Budget Detail.
- **Visible information:** status pill "Reached" (amber/orange, visually distinct from both Approaching and Exceeded), `percentUsed = 100`, Budget Remaining = 0 shown as "Rp 0 remaining", not "exceeded" language.
- **User action:** none.
- **Backend operation:** `status: "REACHED"`.
- **Success behavior:** progress bar fills to exactly 100% width, no overflow styling.
- **Failure behavior:** n/a.
- **Accessibility:** accessible name includes "Reached" distinctly from "Exceeded" so a screen-reader user does not conflate the two.

## 7. Viewing an exceeded Budget

- **Entry point:** Budgets list or Budget Detail.
- **Visible information:** status pill "Exceeded" (red), `percentUsed > 100` (e.g. "125%"), Budget Remaining shown as a negative amount with "Over by Rp 250.000" copy rather than "-Rp 250.000" alone.
- **User action:** none.
- **Backend operation:** `status: "EXCEEDED"`, `remaining < 0`.
- **Success behavior:** progress bar visually caps its filled width at 100% (never renders wider than the track) while the numeric label still shows the true percentage (e.g. "125%") next to the capped bar — the cap is a rendering choice, not a data truncation.
- **Failure behavior:** n/a.
- **Accessibility:** `aria-valuenow` still reports the true `percentUsed` (e.g. `125`), not the visually capped `100`, even though `aria-valuemax` stays `100` — the accessible name text calls out the true percentage explicitly (e.g. "Makan budget, 125% used, exceeded by Rp 250.000") since a capped `aria-valuenow` would misreport progress-bar semantics.

## 8. Editing the Budget amount

- **Entry point:** Budget Detail → Edit Budget.
- **Visible information:** Edit Budget modal, Category shown read-only, amount pre-filled.
- **User action:** changes the amount, confirms.
- **Backend operation:** `PATCH /budgets/:id { amount }`.
- **Success behavior:** modal closes, Budget Detail (and Budgets list) reflect the new amount and recalculated `status`/`percentUsed` for the current period immediately (pessimistic update — the UI waits for the `200` before showing the new value, no optimistic write). Toast confirms.
- **Failure behavior:** `422 INVALID_AMOUNT` → inline error, entered value preserved, modal stays open.
- **Accessibility:** same form-error pattern as Journey 2.

## 9. Archiving a Budget

- **Entry point:** Budget Detail or a Budgets list row action.
- **Visible information:** confirmation is **not** a destructive `alertdialog` (archiving is reversible, unlike delete) — a lighter confirmation or none at all is acceptable; recommend a simple confirm step (not a full modal) since restore is one tap away, avoiding over-warning a reversible action.
- **User action:** confirms archive.
- **Backend operation:** `POST /budgets/:id/archive`.
- **Success behavior:** Budget disappears from the active list, `isArchived: true` / `status: "ARCHIVED"` returned; toast confirms; `["budgets"]`/`["dashboard"]` invalidated.
- **Failure behavior:** `409 ALREADY_ARCHIVED` (stale UI, e.g. archived in another tab) → toast error, refetch the list. `404 NOT_FOUND` → toast error, refetch the list (Budget may have been removed by another session — no delete exists in v1, so this is only reachable via a data race, not a normal path).
- **Accessibility:** action button has an explicit `aria-label` (e.g. "Archive Makan budget"), not an icon-only ambiguous control.

## 10. Restoring an archived Budget

- **Entry point:** Budgets → Archived filter → a row action, or Budget Detail for an archived Budget.
- **Visible information:** archived Budget row/detail with a "Restore" action instead of "Archive".
- **User action:** confirms restore.
- **Backend operation:** `POST /budgets/:id/restore`.
- **Success behavior:** Budget reappears in the active list with its current-period usage recalculated live (not reset to zero) — if expenses were recorded against that category while archived is structurally impossible in v1 since a category with an archived Budget still has no *active* Budget blocking transactions, so this can legitimately show non-zero spend on restore. Toast confirms.
- **Failure behavior:** `409 ALREADY_ACTIVE`, `404 NOT_FOUND` — same handling as Journey 9.
- **Accessibility:** same as Journey 9.

## 11. Transaction changes causing Budget usage to recalculate

- **Entry point:** user adds/edits/deletes an Expense transaction in a categorized-and-budgeted category, from the Transactions feature (not Budgeting).
- **Visible information:** no immediate Budgeting UI change unless the user is currently viewing Budgets/Budget Detail.
- **User action:** the transaction mutation itself (owned by the Transaction feature).
- **Backend operation:** none on the Budget resource — Budget usage is never cached, so nothing needs to be pushed or invalidated server-side. The Transaction feature's existing mutation hooks must add `["budgets"]` to their `invalidateTransactionDependents` query-key list (a cross-feature touch point — see the Phase A.5 review's frontend contract) so that if the user is on Budgets/Budget Detail in the same session, React Query refetches on next focus/mount rather than showing stale usage.
- **Success behavior:** next `GET /budgets`/`GET /budgets/:id` reflects the new transaction set automatically (live calculation, no invalidation logic beyond the query-key list above).
- **Failure behavior:** n/a — this is a read-side consequence, not its own mutation.
- **Accessibility:** n/a.

## 12. No expenses in the current month

- **Entry point:** Budget Detail for a Budget whose category has zero matching transactions this period.
- **Visible information:** Budget Spent = "Rp 0", Budget Remaining = full Budget Limit, status "Healthy", contributing-transaction list shows an explicit empty state ("No matching expenses recorded yet this period") rather than an empty table with no explanation.
- **User action:** none.
- **Backend operation:** `GET /budgets/:id` (`spent: 0`) and the filtered transaction list call (`[]`).
- **Success behavior:** as above.
- **Failure behavior:** n/a.
- **Accessibility:** empty-list region has a heading/text, not a bare empty `<ul>`.

## 13. Category no longer available or inconsistent

- **Entry point:** any Budget whose `Category` was deleted. Under the current schema, `Transaction.categoryId` is `onDelete: SetNull` but `Budget.categoryId` is `onDelete: Cascade` (`prisma/schema.prisma:352-368`) — **deleting a Category deletes its Budget row automatically**. There is no public category-deletion endpoint today (Readiness Audit §2), so this state is currently unreachable through the product's own UI, but the schema-level cascade means a Budget can never outlive its Category.
- **Visible information:** not applicable in v1 — a "budget with a missing category" state cannot occur given the cascade. This journey is documented to confirm the gap does not need UI handling, not because it needs one.
- **User action:** none.
- **Backend operation:** none new required.
- **Success behavior/failure behavior:** n/a.
- **Accessibility:** n/a.
- **Note for Phase B:** if a future `DELETE /categories/:id` endpoint is ever added, this cascade behavior (silently deleting the user's Budget along with the Category) should be surfaced to the user at deletion time — out of scope for this review since no such endpoint exists.

## 14. API/network failure

- **Entry point:** any Budgeting screen, request fails (network error, 5xx, or the shared `lib/api.ts` 401-retry path exhausts).
- **Visible information:** Budgets/Budget Detail show an inline error state ("Budget information is unavailable") with a Retry action, per the screen spec's Error State requirement — never a blank page, never stale data presented as current.
- **User action:** taps Retry, which re-triggers the React Query fetch.
- **Backend operation:** re-issues the same `GET`.
- **Success behavior:** normal render on retry success.
- **Failure behavior:** repeats the error state; no infinite auto-retry loop beyond React Query's default retry count.
- **Accessibility:** error region `role="alert"`, Retry button reachable by keyboard, focus not force-moved away from where the user was.

## 15. Mobile navigation and create/edit interaction

- **Entry point:** mobile bottom navigation → Budgets (new entry, matching the existing 6-item `NAV_ITEMS` pattern in both `app-sidebar.tsx` and `bottom-nav.tsx`).
- **Visible information:** same Budgets list, stacked single-column cards (matching Wallets/Saving Goals mobile layout — no separate mobile-only component).
- **User action:** Create Budget and Edit Budget open as the same `AppModal`-based modal used on desktop (Base UI dialog already handles small-viewport sizing; no separate mobile sheet component exists in the codebase to reuse instead).
- **Backend operation:** identical to desktop journeys above.
- **Success behavior:** identical to desktop.
- **Failure behavior:** identical to desktop.
- **Accessibility:** touch targets for Archive/Restore row actions meet the existing app's minimum tap-target sizing (matches wallet card's kebab-menu precedent); no interaction in this feature depends on hover.
