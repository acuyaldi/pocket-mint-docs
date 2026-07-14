# Pocket Mint — Implementation Reconciliation Report

> Audit date: 2026-07-14  
> Audit method: Static inspection of `pocket-mint-docs`, `pocket-mint-fe`, and `pocket-mint-be`  
> Requirements read first: `docs/product/product-rfc.md` and `docs/product/design-system.md`  
> Source-code changes: None

## Scope and interpretation

The Product RFC and Design System both identify themselves as drafts that require implementation reconciliation (`pocket-mint-docs/docs/product/product-rfc.md:3-6`, `pocket-mint-docs/docs/product/design-system.md:1-12`). This report therefore distinguishes intended behavior from proven implementation. Repository-local architecture and skill documents are treated as evidence of current engineering intent, but they do not silently override the Product RFC.

This was a static audit. Supabase project settings, deployed environment variables, database contents, production scheduler infrastructure, and rendered behavior against live services were not available. Those items are marked `UNVERIFIED` where they affect a conclusion.

Verification run on 2026-07-14: frontend `npm test` passed 26/26 tests. Backend `npm test` passed 314 tests, skipped 4, and failed 3 JWT success-path tests (`test/auth.test.ts`, `test/authContext.test.ts`, and `test/userSync.test.ts`) because the test requests received `401 invalid_token` instead of their expected success responses. The cause of those failures was not established in this no-fix audit.

## Classification summary

| ID | Area | Classification | Priority |
|---|---|---|---|
| PM-WAL-001 | Wallet type classification | IMPLEMENTED | P3 |
| PM-WAL-002 | Debt calculations and limits | PARTIALLY_IMPLEMENTED | P0 |
| PM-FIN-001 | Net worth definition | STALE_DOCUMENTATION | P0 |
| PM-INS-001 | Installment Model A | PARTIALLY_IMPLEMENTED | P0 |
| PM-INS-002 | Installment scheduler and lifecycle | MISSING | P0 |
| PM-INS-003 | Forward and reverse installment calculator | PARTIALLY_IMPLEMENTED | P1 |
| PM-FIN-002 | Decimal usage | PARTIALLY_IMPLEMENTED | P0 |
| PM-TRF-001 | Transfer atomicity | IMPLEMENTED | P1 |
| PM-TRF-002 | Separate Transfer model | STALE_DOCUMENTATION | P2 |
| PM-OWN-001 | Ownership scoping | PARTIALLY_IMPLEMENTED | P0 |
| PM-AUTH-001 | Owner-first authentication | CONFLICT | P0 |
| PM-FE-001 | Frontend currency formatting | PARTIALLY_IMPLEMENTED | P1 |
| PM-DS-001 | Light versus dark direction | IMPLEMENTED | P3 |
| PM-DS-002 | Design-token adoption | PARTIALLY_IMPLEMENTED | P1 |
| PM-DS-003 | Design-system internal consistency | CONFLICT | P1 |
| PM-UI-001 | Wallet card behavior | PARTIALLY_IMPLEMENTED | P1 |
| PM-UI-002 | Dashboard layout | PARTIALLY_IMPLEMENTED | P1 |
| PM-UI-003 | Explainable financial presentation | CONFLICT | P0 |
| PM-SCR-001 | Implemented and missing screens | PARTIALLY_IMPLEMENTED | P1 |
| PM-SCR-002 | Authoritative screen inventory | UNVERIFIED | P1 |
| PM-INS-004 | Installment screen actions | MISSING | P1 |
| PM-TST-001 | RFC minimum test coverage | PARTIALLY_IMPLEMENTED | P1 |

## Detailed findings

### PM-WAL-001 — Wallet type classification

- **Classification:** IMPLEMENTED
- **Area:** Wallet domain model; frontend and backend classification
- **Expected behavior:** Cash, Bank, and E-Wallet are assets; Credit Card and PayLater are debts; debt classification is derived in application logic.
- **Current implementation:** Prisma exposes exactly the five RFC wallet types. Backend classification maps the three asset types and two debt types explicitly. Frontend uses a shared `DEBT_WALLET_TYPES` list and derived `isDebtWallet` predicate.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:137-152`; `pocket-mint-be/prisma/schema.prisma:44-50`; `pocket-mint-be/src/utils/financial.ts:3-15`; `pocket-mint-fe/src/types/wallet.ts:1-14`.
- **Impact:** Asset/debt branching is consistent across the main runtime paths.
- **Recommended decision:** Retain the five-type model and document that `LOAN_PAYLATER` represents PayLater and other loan-like debt subtypes currently collapsed by the frontend.
- **Suggested priority:** P3

### PM-WAL-002 — Debt calculations and credit-limit enforcement

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Outstanding, available credit, utilization/debt ratio, and debt limits
- **Expected behavior:** Outstanding is the absolute debt balance; available credit is limit minus outstanding; debt ratio is outstanding divided by credit limit. Backend rules have priority for financial validation.
- **Current implementation:** Backend serialization and frontend wallet cards implement the formulas. The frontend clamps remaining credit and renders utilization. However, the transaction service never compares a new debt expense or installment total with the wallet credit limit; that guard exists only in the frontend. Thresholds are also inconsistent: wallet cards use warning at 30% and danger at 80%, while the wallets summary switches to danger above 50%.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:164-184`, `pocket-mint-docs/docs/product/product-rfc.md:284-294`; `pocket-mint-be/src/controllers/account.controller.ts:118-133`; `pocket-mint-be/src/services/transaction.service.ts:97-139`, `pocket-mint-be/src/services/transaction.service.ts:141-177`; `pocket-mint-fe/components/WalletCard.tsx:50-72`; `pocket-mint-fe/app/(app)/wallets/page.tsx:41-49`, `pocket-mint-fe/app/(app)/wallets/page.tsx:193-210`; `pocket-mint-fe/app/(app)/transactions/components/AddTransactionModal.tsx:225-233`.
- **Impact:** An API caller can exceed a debt wallet's credit limit even when the UI would reject the same operation. Users also receive inconsistent risk colors for the same ratio.
- **Recommended decision:** Confirm one utilization threshold policy, then make the backend authoritative for credit-limit enforcement and return the canonical computed values to the frontend.
- **Suggested priority:** P0

### PM-FIN-001 — Net worth definition

- **Classification:** STALE_DOCUMENTATION
- **Area:** Net worth calculation
- **Expected behavior:** The Product RFC currently defines net worth as assets minus debts.
- **Current implementation:** Backend, frontend dashboard, wallets screen, repository-local financial guidance, and tests deliberately define net worth as asset total only. Debt is reported separately and only reduces assets when repaid.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:156-160`; `pocket-mint-be/src/utils/financial.ts:26-47`; `pocket-mint-be/src/services/dashboard-query.service.ts:30-46`; `pocket-mint-be/test/dashboardQueryService.test.ts:56-74`; `pocket-mint-fe/app/(app)/dashboard/page.tsx:55-60`; `pocket-mint-fe/app/(app)/wallets/page.tsx:36-49`; `pocket-mint-fe/skills/financial-logic.skill.md:18-23`.
- **Impact:** The primary product definition contradicts every active calculation path. Updating code to match the RFC would materially change displayed finances; updating the RFC would formalize a nonstandard definition that users may misinterpret.
- **Recommended decision:** Product ownership must explicitly choose either conventional net worth (`assets - debts`) or the implemented “asset position” metric. If retaining the implementation, rename the metric or add an explicit rationale and a separately displayed conventional net-worth figure.
- **Suggested priority:** P0

### PM-INS-001 — Installment Model A

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Front-loaded installment balance deduction and reporting
- **Expected behavior:** Deduct the wallet once for the full debt, immediately reflect total outstanding, record monthly installments in monthly reports, and never deduct the wallet again for future installment records.
- **Current implementation:** Creation is atomic, creates one Installment plus one Transaction, stores the monthly amount on the transaction, deducts `grandTotal` once, and marks `balanceDeducted`. Reversal restores the full `grandTotal`. However, only the initial transaction row is created. There is no code that creates future monthly reporting rows, so later monthly reports cannot record each monthly installment.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:188-197`; `pocket-mint-be/src/services/transaction.service.ts:116-177`; `pocket-mint-be/src/domain/transactionBalance.ts:64-74`, `pocket-mint-be/src/domain/transactionBalance.ts:99-106`; `pocket-mint-be/src/services/transaction-query.service.ts:108-143`; `pocket-mint-be/test/transactionService.test.ts:102-110`, `pocket-mint-be/test/transactionService.test.ts:153-163`.
- **Impact:** Outstanding balance behavior matches Model A at creation, but cash-flow reporting after the creation month is incomplete and installment progress cannot advance.
- **Recommended decision:** Keep front-loaded balance locking, but specify and implement the monthly ledger/reporting representation before describing Model A as complete.
- **Suggested priority:** P0

### PM-INS-002 — Installment scheduler and lifecycle

- **Classification:** MISSING
- **Area:** Scheduler behavior, term progression, settlement, and due dates
- **Expected behavior:** The RFC names scheduler implementation as known technical debt, while installment reporting requires future monthly records and the schema exposes `currentTerm`, status, and balance-deduction state.
- **Current implementation:** `currentTerm` is initialized to 1 and status to `ACTIVE`. Installment routes are read-only. No scheduler, job registration, lifecycle command, payment command, or term/status mutation exists in frontend or backend source. Due dates are inferred in the browser from `startDate + currentTerm`, but `currentTerm` never advances in the inspected implementation.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:338-346`; `pocket-mint-be/prisma/schema.prisma:169-200`; `pocket-mint-be/src/services/transaction.service.ts:141-175`; `pocket-mint-be/src/routes/installmentRoutes.ts:1-13`; `pocket-mint-be/src/services/installment-query.service.ts:3-14`; `pocket-mint-fe/app/(app)/cicilan/components/InstallmentCard.tsx:30-43`.
- **Impact:** Installments remain permanently on their initial term unless the database is changed outside the inspected application, making progress, overdue state, settlement, and later-month reporting unreliable.
- **Recommended decision:** Define scheduler ownership, idempotency key, reporting timezone, retry policy, catch-up behavior after downtime, and manual override semantics before implementation.
- **Suggested priority:** P0

### PM-INS-003 — Forward and reverse installment calculator

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Installment calculator modes
- **Expected behavior:** Support forward calculation from principal, interest rate, and duration, and reverse calculation from principal, monthly payment, and duration.
- **Current implementation:** Backend provides Decimal-safe forward calculation, and the add-transaction modal previews a forward estimate. No reverse-calculation domain function, API, form mode, or screen was found. The preview also excludes admin fees from the submitted contract even though it displays an admin-fee estimate.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:201-219`; `pocket-mint-be/src/domain/installment.ts:40-93`; `pocket-mint-fe/app/(app)/transactions/components/AddTransactionModal.tsx:269-275`, `pocket-mint-fe/app/(app)/transactions/components/AddTransactionModal.tsx:663-697`; `pocket-mint-fe/app/(app)/transactions/components/AddTransactionModal.tsx:235-247`.
- **Impact:** Users cannot solve for an implied interest rate/payment plan, and the preview can imply fees that are not persisted in the created installment.
- **Recommended decision:** Define reverse-calculation rounding and impossible-input behavior, then expose both modes from one backend contract. Decide whether admin fees are part of `grandTotal` before updating UI copy.
- **Suggested priority:** P1

### PM-FIN-002 — Decimal usage

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Monetary precision in schema, domain, services, and API boundary
- **Expected behavior:** All money values use Decimal; floating-point arithmetic is prohibited; formatting must not change stored values.
- **Current implementation:** Prisma money columns use Decimal, aggregation and installment domain math use `Prisma.Decimal`, and controllers convert to numbers at the response boundary. But wallet and transaction command services first coerce monetary inputs through JavaScript `Number`, then construct Decimal values from the rounded binary number. Wallet credit limit, interest, and fees are also written from `Number` values. Frontend previews use `Number` and `Math.round`.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:129-133`; `pocket-mint-be/prisma/schema.prisma:52-70`, `pocket-mint-be/prisma/schema.prisma:124-138`, `pocket-mint-be/prisma/schema.prisma:175-200`; `pocket-mint-be/src/domain/installment.ts:28-38`, `pocket-mint-be/src/domain/installment.ts:70-93`; `pocket-mint-be/src/services/transaction.service.ts:74-95`, `pocket-mint-be/src/services/transaction.service.ts:129-138`, `pocket-mint-be/src/services/transaction.service.ts:225-260`; `pocket-mint-be/src/services/wallet.service.ts:36-50`, `pocket-mint-be/src/services/wallet.service.ts:76-91`, `pocket-mint-be/src/services/wallet.service.ts:149-159`; `pocket-mint-fe/app/(app)/transactions/components/AddTransactionModal.tsx:199-200`, `pocket-mint-fe/app/(app)/transactions/components/AddTransactionModal.tsx:269-275`.
- **Impact:** Decimal storage does not fully prevent input precision loss. Values beyond JavaScript's safe integer/precision range or long decimals can be altered before persistence.
- **Recommended decision:** Parse monetary request strings directly into `Prisma.Decimal`, validate with Decimal methods, and document that numeric JSON responses are display conveniences rather than a lossless financial contract.
- **Suggested priority:** P0

### PM-TRF-001 — Transfer atomicity

- **Classification:** IMPLEMENTED
- **Area:** Transfer persistence, balance symmetry, rollback, and ownership
- **Expected behavior:** Transfers cannot target the same wallet; financial mutations are atomic; multi-table operations use database transactions; failures roll back partial writes.
- **Current implementation:** A transfer is one Transaction row containing source and destination wallet IDs. Both wallets are ownership-checked before mutation. The row and both signed balance deltas are applied inside one Prisma transaction. Update reverses the persisted effect before applying the new effect; delete reverses both sides. Tests cover symmetry, self-transfer rejection, atomic boundaries, and rollback.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:223-231`, `pocket-mint-docs/docs/product/product-rfc.md:284-305`; `pocket-mint-be/prisma/schema.prisma:124-155`; `pocket-mint-be/src/services/transaction.service.ts:81-107`, `pocket-mint-be/src/services/transaction.service.ts:180-210`, `pocket-mint-be/src/services/transaction.service.ts:283-320`, `pocket-mint-be/src/services/transaction.service.ts:350-379`; `pocket-mint-be/src/domain/transactionBalance.ts:76-89`, `pocket-mint-be/src/domain/transactionBalance.ts:119-135`; `pocket-mint-be/test/transactionService.test.ts:92-100`, `pocket-mint-be/test/transactionService.test.ts:138-163`.
- **Impact:** The active transfer path preserves balance symmetry and rollback semantics.
- **Recommended decision:** Retain the one-row Transaction representation and make it the documented transfer contract.
- **Suggested priority:** P1

### PM-TRF-002 — Separate Transfer model

- **Classification:** STALE_DOCUMENTATION
- **Area:** Domain model and Prisma schema
- **Expected behavior:** The RFC lists Transfer as a primary entity and says Prisma is the domain-schema source of truth.
- **Current implementation:** Prisma still defines a separate `Transfer` model and relations, but active application code persists transfers only as Transaction rows with `toWalletId`. No service or controller reads or writes the separate model.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:108-121`; `pocket-mint-be/prisma/schema.prisma:31-36`, `pocket-mint-be/prisma/schema.prisma:75-82`, `pocket-mint-be/prisma/schema.prisma:214-235`; `pocket-mint-be/src/services/transaction.service.ts:180-210`.
- **Impact:** Two apparent transfer models make future migrations, API documentation, and reconciliation logic ambiguous.
- **Recommended decision:** Declare Transaction-with-destination canonical. Then either deprecate/remove the unused Transfer model through a deliberate migration or document a concrete future use that justifies retaining it.
- **Suggested priority:** P2

### PM-OWN-001 — Ownership scoping

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Backend resource ownership
- **Expected behavior:** Every resource belongs to exactly one user, every operation is user-scoped, and ownership validation occurs in the backend.
- **Current implementation:** Wallet, transaction, installment, dashboard, and transaction-query reads are scoped by authenticated `userId`. Transaction creation validates ownership of source wallet, destination wallet, and category. Update/delete first load the transaction by `{ id, userId }`. However, transaction update can assign any `categoryId` without checking that the category belongs to the caller. The database foreign key checks existence, not same-user ownership.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:223-231`; `pocket-mint-be/src/services/transaction.service.ts:97-114`, `pocket-mint-be/src/services/transaction.service.ts:241-280`, `pocket-mint-be/src/services/transaction.service.ts:295-306`, `pocket-mint-be/src/services/transaction.service.ts:339-342`; `pocket-mint-be/src/services/wallet.service.ts:121-125`, `pocket-mint-be/src/services/wallet.service.ts:183-205`; `pocket-mint-be/src/services/installment-query.service.ts:50-69`; `pocket-mint-be/src/services/dashboard-query.service.ts:40-45`; `pocket-mint-be/prisma/schema.prisma:96-112`, `pocket-mint-be/prisma/schema.prisma:124-147`.
- **Impact:** A caller who learns another user's category ID can attach that category to their transaction, violating the stated per-resource ownership invariant and exposing cross-user metadata through relation responses.
- **Recommended decision:** Treat this as an implementation defect: validate updated category ownership before entering the transaction and add a regression test. Consider composite ownership constraints only if they can be introduced without complicating the schema unnecessarily.
- **Suggested priority:** P0

### PM-AUTH-001 — Owner-first authentication

- **Classification:** CONFLICT
- **Area:** Registration, JWT flow, route protection, and owner model
- **Expected behavior:** No public registration, a single owner by default, schema support for multiple users, and authenticated ownership for every request.
- **Current implementation:** Backend correctly uses verified Supabase Bearer JWTs and derives identity from the verified `sub`; all financial API routes are gated. Frontend attaches the current access token and syncs the verified user. In conflict with “no public registration,” the login screen exposes email signup and Google OAuth, and the server action calls `supabase.auth.signUp`. Middleware protects only `/dashboard`; `/wallets`, `/transactions`, `/cicilan`, and `/profile` are not server-side redirected, although their API requests still require tokens. `/health`, landing, login, callback, and reset flows are intentionally public, so the RFC's “every request” wording is also too broad.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:235-244`; `pocket-mint-be/src/middleware/apiKeyAuth.ts:6-13`, `pocket-mint-be/src/middleware/apiKeyAuth.ts:55-80`, `pocket-mint-be/src/routes/walletRoutes.ts:8-22`, `pocket-mint-be/src/routes/transaction.routes.ts:8-20`, `pocket-mint-be/src/routes/installmentRoutes.ts:7-11`, `pocket-mint-be/src/routes/dashboardRoutes.ts:7-8`; `pocket-mint-fe/lib/api.ts:10-15`, `pocket-mint-fe/lib/api.ts:26-51`; `pocket-mint-fe/app/login/page.tsx:29`, `pocket-mint-fe/app/login/page.tsx:146-169`, `pocket-mint-fe/app/login/page.tsx:419-471`; `pocket-mint-fe/app/actions/auth.ts:49-80`, `pocket-mint-fe/app/actions/auth.ts:83-102`; `pocket-mint-fe/lib/supabase/middleware.ts:33-54`; `pocket-mint-be/src/app.ts:35-41`.
- **Impact:** Application behavior permits account creation unless external Supabase settings reject it. UI route protection is inconsistent and the RFC cannot be interpreted literally without breaking required public auth/health routes.
- **Recommended decision:** Decide whether Pocket Mint is invite/allowlist/first-owner-only or open multi-user. Enforce that policy in application code rather than relying solely on unverified provider configuration. Rewrite “every request” as “every user-financial API request,” with explicit public exceptions.
- **Suggested priority:** P0

### PM-FE-001 — Frontend currency formatting

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** IDR display and formatting reuse
- **Expected behavior:** UI formatting must not alter stored values; the current frontend financial contract specifies Indonesian Rupiah display with no decimal digits and a shared formatter.
- **Current implementation:** A shared `formatCurrency` uses `Intl.NumberFormat('id-ID')`, currency `IDR`, and zero minimum fraction digits, and it is used across primary financial displays. Separate formatter logic still exists for compact transaction strings and wallet input masks. Financial data is represented as JavaScript `number` in frontend types and calculations.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:129-133`; `pocket-mint-fe/lib/utils.ts:8-14`; `pocket-mint-fe/src/types/wallet.ts:16-25`; `pocket-mint-fe/app/(app)/transactions/components/constants.ts:70-76`; `pocket-mint-fe/app/(app)/wallets/components/CreateWalletModal.tsx:62-68`, `pocket-mint-fe/app/(app)/wallets/components/CreateWalletModal.tsx:115-120`; representative shared usage: `pocket-mint-fe/components/WalletCard.tsx:175-206`.
- **Impact:** Primary display is consistent, but duplicate formatting/parsing paths can drift in spacing, sign, rounding, or locale behavior.
- **Recommended decision:** Keep one display formatter and separately name input parsing/compact-format utilities so their distinct purposes are explicit. Do not claim frontend number handling is lossless.
- **Suggested priority:** P1

### PM-DS-001 — Light versus dark direction

- **Classification:** IMPLEMENTED
- **Area:** Theme direction
- **Expected behavior:** The design system describes an off-white canvas, white cards, deep slate text, and mint accents—an explicitly light visual direction in the frontend UI guidance.
- **Current implementation:** Global tokens and body backgrounds implement the light palette. A Tailwind dark custom variant is declared, but no dark token set or theme toggle was found, so the shipped direction remains light rather than an incomplete dual-theme system.
- **Evidence:** `pocket-mint-docs/docs/product/design-system.md:126-141`, `pocket-mint-docs/docs/product/design-system.md:162-168`; `pocket-mint-fe/app/globals.css:5-29`, `pocket-mint-fe/app/globals.css:43-58`; `pocket-mint-fe/app/(app)/layout.tsx:4-16`.
- **Impact:** The implementation has a coherent light baseline. The unused dark selector may mislead maintainers but does not change the current theme.
- **Recommended decision:** Document light-only as the current product decision. Remove or defer dark-mode promises from future documentation until there is an approved dark palette and contrast audit.
- **Suggested priority:** P3

### PM-DS-002 — Design-token adoption

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Color, radius, typography, and semantic token use
- **Expected behavior:** The documented palette, radii, typography, and spacing should be consistently represented by reusable tokens.
- **Current implementation:** The main semantic color variables closely match the design document. Tabular numerals are attached to the `.font-mono` utility. Adoption is incomplete: radius aliases differ from the design-system scale; `--font-heading` and `--font-mono` both resolve to Inter even though Hanken Grotesk and JetBrains Mono are loaded; multiple feature files embed raw hex/RGBA values; and many currency figures use `font-heading` rather than the financial/tabular utility.
- **Evidence:** `pocket-mint-docs/docs/product/design-system.md:15-123`, `pocket-mint-docs/docs/product/design-system.md:143-175`; `pocket-mint-fe/app/globals.css:8-40`, `pocket-mint-fe/app/globals.css:61-79`; `pocket-mint-fe/app/layout.tsx:2-25`, `pocket-mint-fe/app/layout.tsx:46-50`; raw-value examples: `pocket-mint-fe/app/(app)/cicilan/components/InstallmentCard.tsx:45-69`, `pocket-mint-fe/app/(app)/wallets/page.tsx:203-207`; financial-font examples: `pocket-mint-fe/components/WalletCard.tsx:175-206`, `pocket-mint-fe/app/(app)/dashboard/page.tsx:121-126`.
- **Impact:** Visual changes cannot be made reliably from one token layer, and numerical alignment depends on component-by-component class choices.
- **Recommended decision:** Reconcile the intended font families and radius names first, then audit raw values and financial-number classes against the chosen token contract.
- **Suggested priority:** P1

### PM-DS-003 — Design-system internal consistency

- **Classification:** CONFLICT
- **Area:** Design documentation
- **Expected behavior:** One authoritative visual contract should give components and implementation a single answer.
- **Current implementation:** The document's token table defines background `#f8f9ff`, while prose says `#f8fafc`. It names amber as the Accent, while the token named `accent` is light blue and amber is under `tertiary`. It says primary buttons use Secondary Slate, while current primary buttons use the mint `primary` token. The document says all typography, including `data-mono`, uses Inter; repository-local UI guidance instead assigns Hanken Grotesk and JetBrains Mono, while actual CSS resolves both headings and mono to Inter.
- **Evidence:** `pocket-mint-docs/docs/product/design-system.md:15-62`, `pocket-mint-docs/docs/product/design-system.md:102-106`, `pocket-mint-docs/docs/product/design-system.md:134-147`, `pocket-mint-docs/docs/product/design-system.md:200-203`; `pocket-mint-fe/skills/ui-system.skill.md:13-36`; `pocket-mint-fe/app/globals.css:37-40`; representative mint primary button: `pocket-mint-fe/app/(app)/dashboard/page.tsx:105-111`.
- **Impact:** A developer can comply with one section and violate another. Documentation cannot become authoritative until these semantic collisions are resolved.
- **Recommended decision:** Choose canonical semantic names and actual font families, then rewrite the token block and prose together. Treat third-party brand colors (for example, Google) as an explicit exception category.
- **Suggested priority:** P1

### PM-UI-001 — Wallet card behavior

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Wallet cards, debt display, actions, and design fidelity
- **Expected behavior:** Asset cards have a mint left accent, debt cards a red left accent, large balance, and limit/available detail. Cards should present financial values clearly with tabular figures.
- **Current implementation:** Cards derive asset/debt state, show the expected accent and debt calculations, render utilization, and expose edit/delete controls. PayLater cards use amber rather than the documented debt red. Amounts use `text-xl font-heading` rather than `display-lg`/tabular styling. The `variant` prop is declared but ignored, so compact and full callers receive the same card. Delete copy says linked history will not be removed, but the backend either blocks deletion or cascade-deletes transaction history when forced.
- **Evidence:** `pocket-mint-docs/docs/product/design-system.md:179-183`; `pocket-mint-fe/components/WalletCard.tsx:20-46`, `pocket-mint-fe/components/WalletCard.tsx:60-81`, `pocket-mint-fe/components/WalletCard.tsx:122-224`, `pocket-mint-fe/components/WalletCard.tsx:233-263`; `pocket-mint-be/src/services/wallet.service.ts:174-212`; `pocket-mint-be/prisma/schema.prisma:143-147`.
- **Impact:** Core information is present, but visual hierarchy, variant behavior, and destructive-action messaging are inconsistent with the intended contract and backend behavior.
- **Recommended decision:** Decide whether PayLater is visually “debt red” or “installment amber,” define actual compact/full differences, and align deletion copy with the backend's force/cascade rules.
- **Suggested priority:** P1

### PM-UI-002 — Dashboard layout

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Dashboard composition and responsive layout
- **Expected behavior:** Net worth is the dominant hero at the top; wallet cards and recent transactions occupy the main area; monthly P&L and debt/installment information occupy a right sidebar; desktop content is constrained to 1280px and mobile stacks.
- **Current implementation:** The application shell uses `max-w-7xl` (1280px), responsive padding, and mobile bottom navigation. Dashboard uses a 12-column grid with an 8/4 main/right split, top net-worth hero, wallets, recent transactions, monthly P&L, and active installment widget. The hero's specified large line chart is absent—there is only an empty background placeholder—and the right column shows active installments rather than the design guidance's debt-ratio widget.
- **Evidence:** `pocket-mint-docs/docs/product/design-system.md:154-160`, `pocket-mint-docs/docs/product/design-system.md:195-198`; `pocket-mint-fe/app/(app)/layout.tsx:4-16`; `pocket-mint-fe/app/(app)/dashboard/page.tsx:89-180`, `pocket-mint-fe/app/(app)/dashboard/page.tsx:182-188`, `pocket-mint-fe/app/(app)/dashboard/page.tsx:307-426`; `pocket-mint-fe/components/layout/bottom-nav.tsx:63-76`.
- **Impact:** The high-level information architecture is implemented, but key visual/reporting elements do not match the written design.
- **Recommended decision:** Keep the current responsive shell and 8/4 composition. Decide whether the right rail's second canonical widget is debt ratio or active installments, and only add the hero history chart when real historical data exists.
- **Suggested priority:** P1

### PM-UI-003 — Explainable financial presentation

- **Classification:** CONFLICT
- **Area:** Dashboard, wallets, and installment analytics
- **Expected behavior:** Every financial number must be explainable, calculations deterministic, and the UI must never hide financial calculations.
- **Current implementation:** Several displayed analytics are synthetic or hard-coded: dashboard net-worth growth is fixed at `+4.2%`; wallet net-worth history and growth are generated by multiplying the current value by constants; installment trend repeats the current total for six points; remaining principal/interest is displayed as an arbitrary 70/30 split. The dashboard also computes net worth locally rather than consuming the backend summary contract.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:56-66`, `pocket-mint-docs/docs/product/product-rfc.md:272-280`; `pocket-mint-fe/app/(app)/dashboard/page.tsx:55-60`, `pocket-mint-fe/app/(app)/dashboard/page.tsx:121-130`; `pocket-mint-fe/app/(app)/wallets/components/WalletSummaryCard.tsx:23-34`, `pocket-mint-fe/app/(app)/wallets/components/WalletSummaryCard.tsx:49-58`; `pocket-mint-fe/app/(app)/cicilan/page.tsx:15-28`; `pocket-mint-fe/app/(app)/cicilan/components/OutstandingLiabilityCard.tsx:28-36`.
- **Impact:** Users may interpret fabricated trends and component splits as ledger-derived facts, undermining the product's trust and determinism principles.
- **Recommended decision:** Remove or label placeholders until backend-derived history exists. Financial facts should come from a canonical backend calculation with provenance/tooltips where the derivation is not obvious.
- **Suggested priority:** P0

### PM-SCR-001 — Implemented and missing screens

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Frontend route coverage
- **Expected behavior:** RFC core capabilities require multi-wallet management, net-worth visibility, credit/PayLater tracking, installment management, financial reporting, and automation-ready architecture.
- **Current implementation:** Implemented routes include landing, login/signup/reset, dashboard, wallets, transactions, installments (`/cicilan`), and profile. The primary navigation exposes Dashboard, Wallets, Transactions, and Installments. There is no dedicated reports/analytics route despite financial reporting being a core capability. Automation items are explicitly roadmap work, so their missing screens are not treated as current implementation defects. Reverse calculator is also absent as described in PM-INS-003.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:35-52`, `pocket-mint-docs/docs/product/product-rfc.md:321-334`; route entry points: `pocket-mint-fe/app/page.tsx:1`, `pocket-mint-fe/app/login/page.tsx:1`, `pocket-mint-fe/app/auth/reset-password/page.tsx:1`, `pocket-mint-fe/app/(app)/dashboard/page.tsx:1`, `pocket-mint-fe/app/(app)/wallets/page.tsx:1`, `pocket-mint-fe/app/(app)/transactions/page.tsx:1`, `pocket-mint-fe/app/(app)/cicilan/page.tsx:1`, `pocket-mint-fe/app/(app)/profile/page.tsx:1`; navigation: `pocket-mint-fe/components/layout/app-sidebar.tsx:29-34`, `pocket-mint-fe/components/layout/bottom-nav.tsx:20-27`.
- **Impact:** Day-to-day ledger workflows have screens, but the stated reporting capability is limited to dashboard/monthly summary widgets rather than a complete reporting surface.
- **Recommended decision:** Define whether dashboard widgets satisfy the current “financial reporting” goal. If not, specify a reports screen and its backend aggregates before implementing UI.
- **Suggested priority:** P1

### PM-SCR-002 — Authoritative screen inventory

- **Classification:** UNVERIFIED
- **Area:** Product screen requirements
- **Expected behavior:** A complete screen audit requires an authoritative screen specification.
- **Current implementation:** The Product RFC references `screen-spec.md` and `component-spec.md` as planned, but neither exists in `pocket-mint-docs/docs`. Therefore no undocumented screen can be classified as missing beyond screens directly implied by explicit RFC requirements.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:12-15`, `pocket-mint-docs/docs/product/product-rfc.md:27-31`, `pocket-mint-docs/docs/product/product-rfc.md:350-357`; current product docs are `pocket-mint-docs/docs/product/product-rfc.md` and `pocket-mint-docs/docs/product/design-system.md`.
- **Impact:** Claims of complete frontend coverage would be speculative.
- **Recommended decision:** Create the screen inventory only after the product decisions in this report are resolved. Include route, purpose, required states, permissions, data dependencies, and status.
- **Suggested priority:** P1

### PM-INS-004 — Installment screen actions

- **Classification:** MISSING
- **Area:** Installment UI interaction
- **Expected behavior:** Automation assists but does not replace user control; installment management is a core capability.
- **Current implementation:** Installment cards render “View Details,” “Pay Off Now,” and “Cancel Installment” buttons, but none has an event handler. Backend installment routes are read-only, so there is no API contract for those actions. The non-active branch offers “Cancel Installment” even for already settled/cancelled records.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:45-52`, `pocket-mint-docs/docs/product/product-rfc.md:56-66`; `pocket-mint-fe/app/(app)/cicilan/components/InstallmentCard.tsx:135-161`; `pocket-mint-be/src/routes/installmentRoutes.ts:1-13`.
- **Impact:** The UI promises controls that do nothing, and lifecycle semantics are undefined.
- **Recommended decision:** Hide inert controls until lifecycle commands are specified. Define payoff, cancellation, reversal, balance, reporting, and audit effects before exposing actions.
- **Suggested priority:** P1

### PM-TST-001 — RFC minimum test coverage

- **Classification:** PARTIALLY_IMPLEMENTED
- **Area:** Automated verification
- **Expected behavior:** Minimum coverage includes installment calculations, net worth, transfer consistency, ownership validation, and transaction rollback.
- **Current implementation:** Tests exist for Decimal installment arithmetic, the implemented net-worth rule, transfer symmetry, source/destination ownership, JWT identity, and rollback. Coverage is incomplete relative to reconciled behavior: no scheduler/lifecycle tests can exist because the feature is missing; no reverse-calculator tests exist; no test covers assigning another user's category during transaction update; and net-worth tests encode the implementation that conflicts with the current RFC. The audit-date frontend run passed 26/26 tests. The backend run passed 314, skipped 4, and failed 3 JWT success-path tests with unexpected `401 invalid_token` responses.
- **Evidence:** `pocket-mint-docs/docs/product/product-rfc.md:309-317`; `pocket-mint-be/test/installment.test.ts:7-68`; `pocket-mint-be/test/dashboardQueryService.test.ts:35-85`; `pocket-mint-be/test/transactionBalance.test.ts:19-127`; `pocket-mint-be/test/transactionService.test.ts:92-163`, `pocket-mint-be/test/transactionService.test.ts:167-269`; `pocket-mint-be/test/auth.test.ts:38-157`; update category path without ownership validation: `pocket-mint-be/src/services/transaction.service.ts:241-306`.
- **Impact:** Existing tests strongly protect implemented behavior, but some tests currently reinforce unresolved product semantics and do not cover the identified ownership gap. The backend suite is not currently a green release signal; the JWT failures remain undiagnosed.
- **Recommended decision:** Resolve semantics first, then update tests to the chosen net-worth definition and add regression coverage for category ownership, lifecycle idempotency, later-month reporting, and reverse calculation.
- **Suggested priority:** P1

## 1. Decisions required before updating documentation

1. Choose the canonical net-worth definition and user-facing name: conventional `assets - debts` or the implemented assets-only metric.
2. Choose the authentication policy: closed first-owner/invite/allowlist or open Supabase registration; define explicit public routes.
3. Define installment lifecycle behavior: scheduler owner, timezone, idempotency, missed-run catch-up, payment rows, `currentTerm`, settlement, cancellation, and payoff.
4. Decide whether installment `grandTotal` includes admin fees and whether FLAT and PERCENT fees are supported end to end.
5. Choose canonical debt-ratio thresholds and whether PayLater cards use debt red or installment amber.
6. Choose canonical typography and radius tokens, including whether financial figures use Inter tabular figures or JetBrains Mono.
7. Decide whether dashboard widgets satisfy “financial reporting” or a dedicated reports screen is required.
8. Confirm Transaction-with-`toWalletId` as the sole transfer representation and decide the fate of the unused Transfer model.

## 2. Documentation that should change

After the decisions above:

1. Update `docs/product/product-rfc.md` with the chosen net-worth definition, precise owner-first/public-route rules, canonical installment lifecycle/reporting behavior, debt-limit enforcement, and an unambiguous transfer representation.
2. Update `docs/product/design-system.md` to remove the `#f8f9ff`/`#f8fafc`, accent/tertiary, button-color, font-family, radius, debt-color, and dashboard-right-rail conflicts.
3. Create `docs/product/screen-spec.md` with an authoritative implemented/planned/missing route inventory and required loading, empty, error, permission, and destructive-action states.
4. Create `docs/product/component-spec.md` for wallet-card variants, financial-number typography, installment progress/actions, and canonical interaction states.
5. Reconcile repository-local frontend guidance (`pocket-mint-fe/skills/ui-system.skill.md` and `pocket-mint-fe/skills/financial-logic.skill.md`) with the approved product documents so local instructions do not preserve a competing contract.
6. Update backend API/domain documentation to state that transfers are Transaction rows with `toWalletId` and to document any approved installment lifecycle endpoints.

## 3. Implementation gaps

1. Backend credit-limit enforcement for debt expenses and installments.
2. Transaction-update category ownership validation.
3. Scheduler/lifecycle processing for term advancement, monthly reporting records, settlement, retries, and catch-up.
4. Reverse installment calculation and its API/UI contract.
5. End-to-end admin-fee persistence/calculation or removal of misleading fee UI.
6. Decimal-safe request parsing in backend command services.
7. Removal or clear labeling of fabricated financial trends, growth, and principal/interest splits.
8. Functional installment detail, payoff, and cancellation actions.
9. Reconciled financial reporting surface, if dashboard widgets are not accepted as sufficient.
10. Consistent semantic tokens, financial-number typography, wallet-card variants, and debt-ratio thresholds.
11. Consistent server-side route protection for all authenticated app routes.
12. Resolution of the unused Prisma Transfer model.

## 4. Recommended next task

Run a short product-decision task that produces an approved **financial semantics and ownership addendum** covering net worth, owner-first registration, debt-limit enforcement, transfer representation, installment fees, and the complete scheduler/lifecycle state machine. Do not update the RFC or implement the scheduler until that addendum is approved. Once approved, the first implementation task should close the P0 integrity gaps: backend Decimal-safe parsing, debt-limit enforcement, and transaction-update category ownership validation, with regression tests.
