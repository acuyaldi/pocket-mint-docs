# Pocket Mint Backend Architecture Audit

> Audit date: 2026-07-14  
> Audit scope: `pocket-mint-be`  
> Documentation baseline: all files under `pocket-mint-docs/docs/`  
> Method: read-only static inspection plus non-mutating TypeScript, test, Prisma-schema, diff, and status checks  
> Source changes: none  
> Audit classifications: `COMPLIANT`, `PARTIAL`, `NON_COMPLIANT`, `BLOCKED_BY_PD`, `OUT_OF_SCOPE`, and `UNVERIFIED` where evidence is unavailable  
> Re-audit 2026-07-15 — scope limited to Batch 1 (P1-05) findings: BE-AUTHZ-002 and BE-TX-002 reclassified to `COMPLIANT`; BE-DB-002 remains `PARTIAL`. See the Re-audit Log. All other findings retain their 2026-07-14 classifications.

---

## Executive Summary

The backend has a credible technical foundation: JWT-only authentication, authenticated route gates, User-scoped reads, an Express-free service layer for the principal financial modules, a single adapter-backed Prisma client, Decimal domain helpers, and atomic transaction/transfer orchestration. Those controls are real and are covered by a substantial automated test suite.

The backend does not yet match the two approved Product Decisions closely enough to be implementation-ready. PD-001 is contradicted by the active Net Worth calculation and tests, which return Total Assets as Net Worth. PD-002 is contradicted by `/users/sync`, which provisions any caller holding a valid Supabase JWT without first-Owner initialization, Owner state, invitation, or allowlist enforcement. The current schema has no installation Owner or admission representation.

Financial correctness is also weakened at the request boundary. Wallet and Transaction command services coerce authoritative money through JavaScript `Number` before Decimal persistence. Transaction update does not verify Category ownership (resolved 2026-07-15 by P1-05 — see Re-audit Log). Consequential financial mutations have no request-level idempotency key or persisted deduplication evidence. Dashboard and wallet summary contracts omit the Reporting Cutoff and provenance required to explain PD-001 metrics.

Installment lifecycle, fee treatment, Debt Ratio policy, dedicated Reporting, Transfer representation retirement, Automation, AI, and design-dependent behavior remain governed by Draft Product Decisions or later roadmap phases. These are classified as blocked or out of scope rather than treated as approved implementation requirements.

### Verification Snapshot

| Check | Result |
|---|---|
| `npx tsc --noEmit` | Passed; no TypeScript errors. |
| `npx vitest run` | Failed: 314 passed, 4 skipped, 3 failed. The three failures are JWT success-path tests receiving `401 invalid_token`. |
| `npx prisma validate` | Passed; schema valid. |
| `git diff --check` | Passed for the backend working tree. |
| `git status --short` | Existing unrelated untracked `.claude/settings.local.json`; not modified. |
| Build and generated-output comparison | Not run because `npm run build` writes `dist/`, which would violate the read-only audit constraint. |
| Migration status, drift, and replay | `UNVERIFIED`; no database command was run and no target database was identified or authorized. |

### Classification Summary

| Classification | Findings |
|---|---:|
| `COMPLIANT` | 16 |
| `PARTIAL` | 17 |
| `NON_COMPLIANT` | 6 |
| `BLOCKED_BY_PD` | 9 |
| `UNVERIFIED` | 2 |
| `OUT_OF_SCOPE` | 2 |
| **Total** | **52** |

Counts reflect the 2026-07-15 re-audit (see Re-audit Log). Initial 2026-07-14 counts were `COMPLIANT` 14 and `NON_COMPLIANT` 8; BE-AUTHZ-002 and BE-TX-002 moved to `COMPLIANT`.

### Re-audit Log

**2026-07-15 — Batch 1 — Ownership Integrity Hotfix (P1-05).** Findings re-audited: BE-AUTHZ-002, BE-TX-002, BE-DB-002.

- BE-AUTHZ-002 and BE-TX-002 reclassified `NON_COMPLIANT` → `COMPLIANT`. Transaction update now validates Category ownership (`{ id: categoryId, userId }`) before entering `$transaction`, with regression coverage asserting the stable `NOT_FOUND` error code, that `$transaction` is never entered on rejection, and that no financial writes occur.
- BE-DB-002 remains `PARTIAL`. The application-level gap it cited is closed, but same-User integrity is still not structural; the reinforcement assessment (backlog P1-07) is open.
- Verification at re-audit: focused transaction-service tests 24 passed / 0 failed; full `npx vitest run` 317 passed, 3 failed (pre-existing JWT success-path baseline, backlog P2-04), 4 skipped; `npx tsc --noEmit` clean; `npx prisma validate` passing; `dist/services/transaction.service.js` verified byte-identical to a fresh compile of `src/`.
- Evidence observed in the backend working tree pending commit.

---

## 1. Repository Structure

### BE-STR-001 — Backend repository README describes the wrong application

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Repository structure and onboarding
- **Expected:** Repository entry documentation should identify the Express 5, TypeScript, Prisma backend and its supported commands, boundaries, and deployment model.
- **Current:** `README.md` is the untouched Next.js `create-next-app` template and directs readers to an `app/page.tsx` frontend that does not exist in this repository.
- **Evidence:** `pocket-mint-be/README.md:1-31`; `pocket-mint-be/package.json:2-24`; `pocket-mint-be/AGENTS.md:1-4`.
- **Impact:** New maintainers and automation receive false repository identity, startup, and deployment guidance before reaching the accurate local architecture documents.
- **Recommendation:** Replace the README with backend-specific repository purpose, architecture entry points, commands, environment boundaries, and links to authoritative Pocket Mint documentation.
- **Priority:** P1

### BE-STR-002 — Runtime source is separated into recognizable layers

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Repository structure
- **Expected:** HTTP routing, authentication middleware, controllers, Business Services, domain calculations, persistence construction, and tests should have distinct ownership.
- **Current:** Runtime code is separated across `routes`, `middleware`, `controllers`, `services`, `domain`, `http`, `lib`, and `utils`; tests mirror the principal boundaries.
- **Evidence:** `pocket-mint-be/src/routes/index.ts:1-17`; `pocket-mint-be/src/app.ts:17-44`; `pocket-mint-be/src/services/transaction.service.ts:1-35`; `pocket-mint-be/src/domain/transactionBalance.ts:1-13`; `pocket-mint-be/test/httpBoundaryGuards.test.ts:62-112`.
- **Impact:** Ownership is discoverable and most backend changes can be isolated to the correct layer.
- **Recommendation:** Preserve these boundaries and remove stale documentation rather than introducing another repository abstraction.
- **Priority:** P3

---

## 2. Architecture

### BE-ARCH-001 — Core financial request path follows the documented layering

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Architecture
- **Expected:** Requests should flow through authenticated Backend API boundaries into Business Services and then Prisma, with domain calculations outside controllers.
- **Current:** Wallet, Transaction, Installment query, and Dashboard routes use authentication middleware, thin controllers, Express-free services, domain helpers, and the shared Prisma client.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:125-160`; `pocket-mint-be/src/routes/transaction.routes.ts:8-20`; `pocket-mint-be/src/controllers/transaction.controller.ts:161-224`; `pocket-mint-be/src/services/transaction.service.ts:54-60`; `pocket-mint-be/src/lib/prisma.ts:26-43`.
- **Impact:** Most financial behavior has one traceable orchestration path and is testable without Express.
- **Recommendation:** Retain the route → auth → controller → service → domain → Prisma flow as the mandatory default.
- **Priority:** P3

### BE-ARCH-002 — User provisioning bypasses the Business Service layer

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Architecture and Business Service ownership
- **Expected:** Controllers should perform HTTP mapping; application behavior and persistence should be owned by Business Services.
- **Current:** `UserController.sync` performs Prisma lookup and creation directly in the controller. No User or admission service owns provisioning policy.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:125-148`; `pocket-mint-be/src/controllers/user.controller.ts:2-4`; `pocket-mint-be/src/controllers/user.controller.ts:53-68`; `pocket-mint-be/docs/architecture-http-boundary.md:104-114`.
- **Impact:** The highest-risk access-policy mutation lacks a service boundary where Owner initialization, admission, concurrency, and audit rules can be enforced consistently.
- **Recommendation:** After the admission contract is documented, make User provisioning an application service operation and keep the controller as an allowlisting HTTP adapter.
- **Priority:** P0

### BE-ARCH-003 — Wallet response serialization performs business calculations

- **Classification:** `PARTIAL`
- **Severity:** High
- **Area:** Controller and Canonical Calculation boundary
- **Expected:** Controllers serialize values; Business Services own Available Credit, Outstanding Balance, Debt Ratio, and other Canonical Calculations.
- **Current:** `serializeWallet` converts Decimal values to numbers, then calculates remaining credit with addition and outstanding debt with `Math.abs` inside the controller. Debt Ratio is not returned.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:138-149`; `pocket-mint-be/src/controllers/account.controller.ts:112-133`; `pocket-mint-be/docs/architecture-http-boundary.md:104-114`.
- **Impact:** Authoritative debt results depend on controller-local floating-point logic and cannot be reused consistently by Automation, Reports, or other clients.
- **Recommendation:** Define an approved backend debt-summary contract and return exact service-produced values for serialization only. PD-005-dependent thresholds must remain blocked.
- **Priority:** P0

---

## 3. Authentication

### BE-AUTH-001 — Supabase Bearer JWT is the sole runtime authentication method

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Authentication
- **Expected:** Financial capabilities require verified identity, and self-asserted headers, body fields, or query fields must not authenticate a caller.
- **Current:** Middleware accepts only a Bearer token, verifies signature, audience, issuer when configured, and expiration through `jose`, then publishes the verified `sub` as `req.auth.userId`. Legacy identity paths are not used.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/002-owner-auth.md:143-147`; `pocket-mint-be/src/middleware/apiKeyAuth.ts:6-13`; `pocket-mint-be/src/middleware/apiKeyAuth.ts:47-52`; `pocket-mint-be/src/utils/supabaseJwt.ts:32-60`; `pocket-mint-be/test/auth.test.ts:101-149`.
- **Impact:** Request identity is fail-closed and cannot be selected by client-controlled identifiers.
- **Recommendation:** Preserve JWT-only authentication and the uniform external 401 contract.
- **Priority:** P3

### BE-AUTH-002 — Controlled admission and first-Owner initialization are not enforced

- **Classification:** `NON_COMPLIANT`
- **Severity:** Critical
- **Area:** Authentication and access policy
- **Expected:** The first initialized account becomes the single active Owner atomically; after initialization, additional Users require an Owner-approved invitation or allowlist and public self-registration is prohibited.
- **Current:** Any caller with a valid Supabase JWT may call `POST /users/sync`; if its verified `sub` has no local row, the controller creates one. The schema has no Owner, installation state, invitation, allowlist, or admission approval representation.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/002-owner-auth.md:129-145`; `pocket-mint-be/src/routes/user.routes.ts:8-11`; `pocket-mint-be/src/middleware/apiKeyAuth.ts:92-106`; `pocket-mint-be/src/controllers/user.controller.ts:53-70`; `pocket-mint-be/prisma/schema.prisma:20-39`.
- **Impact:** A deployment can admit arbitrary Supabase-authenticated accounts, cannot prove one Owner, and cannot enforce the approved closed-installation boundary.
- **Recommendation:** Treat Phase 2 as blocked until the admission representation, existing-installation migration, atomic bootstrap, invitation or allowlist contract, and recovery boundaries are documented; then implement them in the backend before frontend access flows.
- **Priority:** P0

### BE-AUTH-003 — JWT success-path verification is currently failing in tests

- **Classification:** `UNVERIFIED`
- **Severity:** High
- **Area:** Authentication verification
- **Expected:** Valid test JWTs should authenticate and populate the canonical request context; the full backend verification suite should be green before release.
- **Current:** Fresh audit execution produced 3 failing success-path tests with `401 invalid_token`; 314 tests passed and 4 were skipped. Static code shows the intended verifier, but this audit did not establish whether the failures come from `jose`, fixture configuration, environment certificate state, or runtime code.
- **Evidence:** `pocket-mint-be/test/auth.test.ts:38-45`; `pocket-mint-be/test/authContext.test.ts:39-47`; `pocket-mint-be/test/userSync.test.ts:49-60`; `pocket-mint-be/src/utils/supabaseJwt.ts:44-60`.
- **Impact:** The release signal is red on the authentication success path, so actual verified-user access cannot be claimed from test evidence.
- **Recommendation:** Diagnose the three failures as a separate read-only-first debugging task before relying on the suite as an authentication release gate.
- **Priority:** P0

### BE-AUTH-004 — `TRUST_PROXY=true` is accepted despite the numeric-hop contract

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Authentication-adjacent request trust and rate limiting
- **Expected:** Proxy trust should be a numeric hop count; unrestricted `true` must be rejected because it permits spoofed forwarded-address chains.
- **Current:** Configuration comments warn against `true`, but `parseTrustProxy` explicitly returns `true` when the environment value is `true`.
- **Evidence:** `pocket-mint-be/src/config/index.ts:31-42`; `pocket-mint-be/src/config/index.ts:118-118`; `pocket-mint-be/.claude/skills/authentication-security.skill.md:35-38`; `pocket-mint-be/docs/deployment-runbook.md:120-127`.
- **Impact:** A production misconfiguration can weaken the IP-based pre-authentication rate-limit key.
- **Recommendation:** Enforce the documented numeric-hop-only contract and fail production startup on `TRUST_PROXY=true`.
- **Priority:** P1

---

## 4. Authorization

### BE-AUTHZ-001 — Financial routes and primary reads are User-scoped

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Authorization
- **Expected:** Every financial query and mutation must pass through authenticated, User-scoped backend controls.
- **Current:** Wallet, Transaction, Installment, and Dashboard routes require `requireUser`; service queries include the authenticated `userId` in their ownership filters.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:266-270`; `pocket-mint-be/src/routes/walletRoutes.ts:8-22`; `pocket-mint-be/src/routes/transaction.routes.ts:8-20`; `pocket-mint-be/src/routes/installmentRoutes.ts:7-11`; `pocket-mint-be/src/services/wallet-query.service.ts:49-68`; `pocket-mint-be/src/services/installment-query.service.ts:61-69`.
- **Impact:** Direct cross-User enumeration is prevented on inspected financial endpoints.
- **Recommendation:** Preserve defense-in-depth controller guards and service-level ownership predicates.
- **Priority:** P3

### BE-AUTHZ-002 — Transaction update accepts another User's Category identifier

- **Classification:** `COMPLIANT` (re-audit 2026-07-15; was `NON_COMPLIANT` at the 2026-07-14 audit)
- **Severity:** Critical
- **Area:** Resource authorization
- **Expected:** Every related financial resource must belong to the requesting User; update paths must revalidate new relationship identifiers before mutation.
- **Current (2026-07-14):** Create validates Category ownership. Update validates source and destination wallets but writes `categoryId` without checking `{ id: categoryId, userId }`. The database foreign key checks existence only.
- **Evidence (2026-07-14):** `pocket-mint-docs/docs/ai/project-context.md:72-74`; `pocket-mint-be/src/services/transaction.service.ts:109-114`; `pocket-mint-be/src/services/transaction.service.ts:263-280`; `pocket-mint-be/src/services/transaction.service.ts:295-305`; `pocket-mint-be/prisma/schema.prisma:132-146`.
- **Impact:** A caller who learns another User's Category ID can attach it to their Transaction and receive cross-User Category metadata through relation serialization.
- **Recommendation:** In Phase 1, add ownership validation for every changed relationship before entering the mutation transaction and add a regression test proving cross-User Category assignment is rejected.
- **Priority:** P0
- **Re-audit (2026-07-15, P1-05):** Resolved. Update now validates `{ id: categoryId, userId }` via `db.category.findFirst` and rejects unowned Categories with the typed 404 `NOT_FOUND` error before entering `$transaction`; clearing the Category remains exempt. Compiled `dist/services/transaction.service.js` carries the identical check. Regression tests assert rejection with `NOT_FOUND`, that `$transaction` is never entered, that no financial writes occur, plus owned-Category acceptance and clearing without a lookup. Evidence: `pocket-mint-be/src/services/transaction.service.ts:283-290`; `pocket-mint-be/dist/services/transaction.service.js:249-256`; `pocket-mint-be/test/transactionService.test.ts:226-257`.

---

## 5. Business Services

### BE-SVC-001 — Principal financial modules use Express-free services and typed operational errors

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Business Services
- **Expected:** Business behavior should be Express-free, own validation and orchestration, return typed results, and throw deterministic operational errors.
- **Current:** Wallet, Transaction, Transaction query, Wallet query, Installment query, and Dashboard query services accept typed inputs, use injected Prisma surfaces, and expose typed operational errors where applicable.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:138-149`; `pocket-mint-be/src/services/wallet.service.ts:1-18`; `pocket-mint-be/src/services/transaction.service.ts:1-35`; `pocket-mint-be/src/services/transaction.errors.ts:11-24`; `pocket-mint-be/src/http/forwardError.ts:29-49`.
- **Impact:** Domain behavior is testable and less likely to drift across controllers.
- **Recommendation:** Extend this pattern to User admission and future approved reporting or automation commands.
- **Priority:** P3

### BE-SVC-002 — Service ownership is incomplete at access and response-calculation boundaries

- **Classification:** `PARTIAL`
- **Severity:** High
- **Area:** Business Services
- **Expected:** Admission, ownership, Canonical Calculations, lifecycle rules, and final acceptance or rejection should be owned by Business Services.
- **Current:** User provisioning writes from a controller, and wallet debt-derived values are calculated in a serializer. No service owns controlled admission or a complete Financial Summary contract.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:138-149`; `pocket-mint-be/src/controllers/user.controller.ts:53-68`; `pocket-mint-be/src/controllers/account.controller.ts:118-133`; `pocket-mint-be/src/services/dashboard-query.service.ts:40-45`.
- **Impact:** High-authority business rules are split across HTTP code or missing, making contract reuse and enforcement inconsistent.
- **Recommendation:** Fill these boundaries in roadmap order: Financial Core calculations first, then Controlled Access, then Wallet and Reporting contracts.
- **Priority:** P0

---

## 6. Wallet Engine

### BE-WAL-001 — Wallet type classification and ownership-scoped reads match canonical terminology

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Wallet Engine
- **Expected:** Cash, Bank, and E-Wallet are Asset Wallets; Credit Card and PayLater are Debt Wallets; classification is type-based and reads are User-scoped.
- **Current:** The backend explicitly classifies all five types, scopes wallet reads to `userId`, and uses absolute Debt Wallet balances for Total Outstanding Debt.
- **Evidence:** `pocket-mint-docs/docs/product/glossary.md:100-134`; `pocket-mint-be/src/utils/financial.ts:3-15`; `pocket-mint-be/src/utils/financial.ts:26-36`; `pocket-mint-be/src/services/wallet-query.service.ts:49-68`.
- **Impact:** Asset and debt aggregation begins from a stable, explicit classification.
- **Recommendation:** Preserve explicit exhaustive type handling.
- **Priority:** P3

### BE-WAL-002 — Net Worth is implemented as Total Assets only

- **Classification:** `NON_COMPLIANT`
- **Severity:** Critical
- **Area:** Wallet Engine and approved financial semantics
- **Expected:** PD-001 defines Net Worth as Total Assets minus Total Outstanding Debt at the same Reporting Cutoff; an assets-only value must be labeled Total Assets.
- **Current:** `calculateNetWorth` sets `netWorth = totalAset`. Wallet and Dashboard services and tests explicitly preserve that behavior.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/001-net-worth.md:129-139`; `pocket-mint-be/src/utils/financial.ts:26-47`; `pocket-mint-be/src/services/dashboard-query.service.ts:32-45`; `pocket-mint-be/test/dashboardQueryService.test.ts:63-73`; `pocket-mint-be/.claude/skills/financial-logic.skill.md:17-27`.
- **Impact:** Every backend Financial Summary overstates the approved Net Worth whenever tracked debt exists, and repository-local guidance reinforces the contradiction.
- **Recommendation:** In Phase 0 synchronize local backend guidance, then in Phase 1 implement and test PD-001's calculation and keep Total Assets as a separate field.
- **Priority:** P0

### BE-WAL-003 — Canonical Credit Limit enforcement is blocked by PD-005

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** High
- **Area:** Wallet Engine
- **Expected:** No hard-limit, threshold, over-limit migration, or PayLater risk policy may be treated as settled until PD-005 is approved.
- **Current:** The backend requires a positive Credit Limit when creating a Debt Wallet but does not reject debt-changing operations that exceed it and does not provide Debt Ratio thresholds.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/005-debt-policy.md:6-8`; `pocket-mint-docs/docs/development/implementation-roadmap.md:213-218`; `pocket-mint-be/src/services/wallet.service.ts:65-73`; `pocket-mint-be/src/services/transaction.service.ts:97-139`.
- **Impact:** Implementing a hard or soft limit now would encode Draft policy; omitting it leaves current client behavior non-authoritative.
- **Recommendation:** Approve and synchronize PD-005 before implementing the chosen backend constraint and migration behavior.
- **Priority:** P0

### BE-WAL-004 — Forced wallet deletion destroys Ledger history

- **Classification:** `NON_COMPLIANT`
- **Severity:** High
- **Area:** Wallet Engine and Historical Records
- **Expected:** Financial facts and corrections must remain explainable; deletion behavior must not silently destroy the Historical Records required to reproduce prior financial state.
- **Current:** `DELETE /wallets/:id?force=true` permits deletion when ordinary Transactions exist. The Wallet relation cascades Transaction and Installment deletion from the database.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:145-157`; `pocket-mint-be/src/services/wallet.service.ts:174-212`; `pocket-mint-be/prisma/schema.prisma:76-82`; `pocket-mint-be/prisma/schema.prisma:143-147`; `pocket-mint-be/prisma/schema.prisma:204-207`.
- **Impact:** Historical Financial Events and Installments can disappear, making past balances and Reports irreproducible.
- **Recommendation:** Stop treating force-cascade deletion as architecture-compliant; resolve deletion and correction policy in documentation before changing the runtime behavior.
- **Priority:** P0

---

## 7. Transaction Engine

### BE-TX-001 — Transfer creation, update, and deletion use symmetric atomic effects

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Transaction Engine
- **Expected:** A Transfer must affect source and destination together, never count as Income or Expense, and reverse from persisted facts.
- **Current:** Active transfers are one Transaction with `toWalletId`; both wallet deltas execute inside one Prisma transaction. Update reverses the persisted old effect before applying the new effect, and delete reverses both sides.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:244-259`; `pocket-mint-be/src/services/transaction.service.ts:180-210`; `pocket-mint-be/src/services/transaction.service.ts:283-320`; `pocket-mint-be/src/services/transaction.service.ts:350-379`; `pocket-mint-be/src/domain/transactionBalance.ts:76-106`.
- **Impact:** The active transfer path preserves balance symmetry and rollback behavior.
- **Recommendation:** Preserve the domain helper as the single source of balance effects.
- **Priority:** P3

### BE-TX-002 — Transaction update has an authorization gap for Category changes

- **Classification:** `COMPLIANT` (re-audit 2026-07-15; was `NON_COMPLIANT` at the 2026-07-14 audit)
- **Severity:** Critical
- **Area:** Transaction Engine
- **Expected:** Create and update must enforce the same User ownership invariants for all related resources.
- **Current (2026-07-14):** Create validates Category ownership; update writes the requested Category without ownership validation.
- **Evidence (2026-07-14):** `pocket-mint-be/src/services/transaction.service.ts:109-114`; `pocket-mint-be/src/services/transaction.service.ts:241-280`; `pocket-mint-be/src/services/transaction.service.ts:295-305`; `pocket-mint-be/test/transactionService.test.ts:167-269`.
- **Impact:** Transaction update is a cross-User metadata access path and violates the backend's principal authorization invariant.
- **Recommendation:** Address with BE-AUTHZ-002 in Phase 1 and add focused regression coverage.
- **Priority:** P0
- **Re-audit (2026-07-15, P1-05):** Resolved with BE-AUTHZ-002. Update enforces the same Category ownership invariant as create, before `$transaction`; focused regression coverage added. Evidence: `pocket-mint-be/src/services/transaction.service.ts:283-290`; `pocket-mint-be/test/transactionService.test.ts:226-257`; focused suite 24 passed / 0 failed.

### BE-TX-003 — Wallet-filtered transaction reads omit incoming Transfers

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Transaction Engine queries
- **Expected:** Wallet-related Financial Events should include events where the wallet is the source or destination so Wallet Detail can explain its balance.
- **Current:** Listing with `walletId` filters only `Transaction.walletId`; unlike sparkline queries, it does not include `toWalletId` matches.
- **Evidence:** `pocket-mint-docs/docs/product/screen-spec.md:157-177`; `pocket-mint-be/src/services/transaction-query.service.ts:82-105`; `pocket-mint-be/src/services/wallet-query.service.ts:102-115`.
- **Impact:** An incoming Transfer can affect a wallet balance while being absent from that wallet's filtered Financial Event list.
- **Recommendation:** Define the wallet-event query contract explicitly and make source/destination inclusion consistent after PD-007 synchronization.
- **Priority:** P1

### BE-TX-004 — Sole Transfer representation and schema retirement are blocked by PD-007

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** Medium
- **Area:** Transaction Engine
- **Expected:** The active Transaction representation must not be declared the sole product representation or trigger schema retirement until PD-007 is approved.
- **Current:** Runtime services use Transaction with `toWalletId`, while the Prisma schema still includes an unused separate `Transfer` model.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/007-transfer.md:6-8`; `pocket-mint-be/src/services/transaction.service.ts:180-210`; `pocket-mint-be/prisma/schema.prisma:214-235`.
- **Impact:** Two apparent domain representations remain available to future code and migration authors.
- **Recommendation:** Approve PD-007, search for external consumers and legacy data, then document and migrate one representation deliberately.
- **Priority:** P1

---

## 8. Installment Engine

### BE-INS-001 — Installment lifecycle implementation is blocked by PD-003

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** Critical
- **Area:** Installment Engine
- **Expected:** Automatic progression, catch-up, Installment Payments, settlement, payoff, cancellation, and Reporting Timezone behavior require approved PD-003 policy.
- **Current:** Creation sets `currentTerm = 1` and `status = ACTIVE`; the Installment HTTP surface is read-only and no code advances terms or records later payments.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/003-installment-lifecycle.md:6-8`; `pocket-mint-docs/docs/development/implementation-roadmap.md:261-285`; `pocket-mint-be/src/services/transaction.service.ts:141-175`; `pocket-mint-be/src/routes/installmentRoutes.ts:7-11`; `pocket-mint-be/src/services/installment-query.service.ts:50-70`.
- **Impact:** Existing Installments remain at their initial term and cannot provide authoritative lifecycle or later-period reporting.
- **Recommendation:** Do not add scheduler or lifecycle commands until PD-003 is approved and synchronized with API, persistence, migration, idempotency, and audit contracts.
- **Priority:** P0

### BE-INS-002 — Fee semantics are present in schema and rate responses but blocked by PD-004

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** High
- **Area:** Installment Engine
- **Expected:** Administrative Fee type, basis, persistence, and inclusion in Total Obligation require approved PD-004 policy.
- **Current:** Wallet and Installment schema fields support flat and percentage fees, and a static rates endpoint advertises provider fees. Creation ignores those fee fields and calculates Total Obligation from principal plus Interest only.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/004-installment-fees.md:6-8`; `pocket-mint-be/prisma/schema.prisma:66-70`; `pocket-mint-be/prisma/schema.prisma:184-193`; `pocket-mint-be/src/controllers/installment.controller.ts:9-18`; `pocket-mint-be/src/domain/installment.ts:83-93`.
- **Impact:** The backend exposes and persists shapes for Draft fee behavior while saved obligations do not match the advertised fee model.
- **Recommendation:** Treat current fee fields and static rates as provisional; approve PD-004 before defining calculation, response, data-review, and migration behavior.
- **Priority:** P0

### BE-INS-003 — Complete calculation and lifecycle contract is blocked by PD-003 through PD-005

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** High
- **Area:** Installment Engine
- **Expected:** Forward and reverse calculation, impossible-input behavior, fee-inclusive Total Obligation, Credit Limit enforcement, schedule creation, and lifecycle state must be approved together before Phase 5 completion.
- **Current:** The backend implements only forward calculation for fixed 3, 6, or 12-month tenors using a flat monthly Interest rate. No reverse calculator, durable payment schedule, final-term field, or lifecycle command exists.
- **Evidence:** `pocket-mint-docs/docs/development/implementation-roadmap.md:275-285`; `pocket-mint-be/src/services/transaction.service.ts:37-39`; `pocket-mint-be/src/services/transaction.service.ts:117-139`; `pocket-mint-be/src/domain/installment.ts:40-93`; `pocket-mint-be/prisma/schema.prisma:175-211`.
- **Impact:** Existing behavior is a partial provisional model and cannot be presented as an approved Installment Engine.
- **Recommendation:** Complete Phase 0 Product Decision work before expanding or normalizing the Installment implementation.
- **Priority:** P0

---

## 9. Financial Calculations

### BE-FIN-001 — Approved Net Worth calculation is contradicted by code and tests

- **Classification:** `NON_COMPLIANT`
- **Severity:** Critical
- **Area:** Financial Calculations
- **Expected:** Net Worth equals Total Assets minus Total Outstanding Debt, using one Reporting Cutoff.
- **Current:** The shared calculation returns Total Assets as Net Worth; tests assert the contradiction as intended behavior.
- **Evidence:** `pocket-mint-docs/docs/ai/project-context.md:81-86`; `pocket-mint-be/src/utils/financial.ts:26-47`; `pocket-mint-be/test/reportingEffect.test.ts:45-58`; `pocket-mint-be/test/dashboardQueryService.test.ts:69-73`.
- **Impact:** Incorrect approved semantics are consistently propagated across Wallet and Dashboard endpoints.
- **Recommendation:** Synchronize repository-local instructions first, then update the canonical helper, service contracts, serializers, and tests as one PD-001 implementation change.
- **Priority:** P0

### BE-FIN-002 — Balance-effect and Installment arithmetic are deterministic in their domain modules

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Financial Calculations
- **Expected:** Financial calculations should be deterministic, Decimal-based, and shared rather than reimplemented per mutation path.
- **Current:** Transfer, Income, Expense, reversal, reconciliation, and forward Installment arithmetic use shared Decimal helpers with explicit two-decimal half-up rounding.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:29-41`; `pocket-mint-be/src/domain/transactionBalance.ts:53-106`; `pocket-mint-be/src/domain/installment.ts:30-38`; `pocket-mint-be/src/domain/installment.ts:70-93`; `pocket-mint-be/test/installment.test.ts:7-63`.
- **Impact:** Once inputs reach these helpers correctly, calculations are reproducible and reversal rules are centralized.
- **Recommendation:** Preserve these helpers and fix precision loss before their input boundary.
- **Priority:** P3

### BE-FIN-003 — Financial Summary contracts omit Reporting Cutoff and provenance

- **Classification:** `PARTIAL`
- **Severity:** High
- **Area:** Financial Calculations and explainability
- **Expected:** Net Worth components must share one Reporting Cutoff, and Derived Metrics must identify their calculation, time boundary, and supporting facts.
- **Current:** Dashboard summary returns only three numeric fields. Wallet mutation snapshots return three totals without cutoff or provenance. Calculations use current stored balances but do not communicate when those balances were evaluated.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/001-net-worth.md:135-139`; `pocket-mint-docs/docs/architecture/system-architecture.md:33-37`; `pocket-mint-be/src/controllers/dashboard.controller.ts:14-34`; `pocket-mint-be/src/controllers/account.controller.ts:99-109`.
- **Impact:** Consumers cannot prove that components share a cutoff or explain a headline value from the response contract.
- **Recommendation:** In Phase 1 define a canonical Financial Summary result containing exact components, Reporting Cutoff, and provenance suitable for Dashboard, Reports, Automation, and AI explanation.
- **Priority:** P0

---

## 10. Reporting

### BE-REP-001 — Monthly cash-flow reporting has useful but incomplete canonical foundations

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Reporting
- **Expected:** Reporting should use User-scoped Ledger facts, exact aggregates, half-open Reporting Period boundaries, and explicit handling of Transfers and missing history.
- **Current:** Monthly Income and Expense are aggregated with Decimal database sums, Transfers are excluded, and half-open timezone-aware ranges are used. The response contains a month label but not Reporting Timezone, provenance, completeness, or supporting-record references.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:194-207`; `pocket-mint-be/src/services/transaction-query.service.ts:108-145`; `pocket-mint-be/src/domain/reportingTime.ts:79-91`; `pocket-mint-be/src/controllers/transaction.controller.ts:93-100`.
- **Impact:** Current summaries are numerically useful but do not satisfy the complete explainable Report contract.
- **Recommendation:** Preserve exact aggregation while deferring the full report contract to Phase 6 after PD-003 and PD-006 approval.
- **Priority:** P1

### BE-REP-002 — Dedicated Reporting contracts are blocked by PD-006

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** High
- **Area:** Reporting
- **Expected:** Dedicated Reports scope, Dashboard boundary, historical views, partial failure, and navigation must not be implemented as settled behavior until PD-006 is approved.
- **Current:** There is no report route or report service. Dashboard provides a current stored-balance summary, and Transaction provides only a monthly cash-flow summary.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/006-reporting.md:6-8`; `pocket-mint-docs/docs/development/implementation-roadmap.md:295-320`; `pocket-mint-be/src/routes/index.ts:10-15`; `pocket-mint-be/src/services/dashboard-query.types.ts:11-15`.
- **Impact:** A complete backend Reporting surface cannot be claimed or safely designed from approved policy.
- **Recommendation:** Approve PD-006 and PD-003, then define report questions, periods, provenance, partial failures, and missing-history behavior before API implementation.
- **Priority:** P0

### BE-REP-003 — Reporting Timezone authority is blocked by PD-003

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** Medium
- **Area:** Reporting Timezone
- **Expected:** The persisted authoritative Reporting Timezone and its change behavior remain governed by PD-003.
- **Current:** One process-wide environment value, defaulting to `Asia/Jakarta`, controls every User's business-date parsing and reporting period.
- **Evidence:** `pocket-mint-docs/docs/product/glossary.md:584-606`; `pocket-mint-docs/docs/product/decisions/003-installment-lifecycle.md:6-8`; `pocket-mint-be/src/config/index.ts:56-59`; `pocket-mint-be/src/services/transaction.service.ts:88-92`; `pocket-mint-be/src/services/transaction-query.service.ts:38-60`.
- **Impact:** Multi-User deployments cannot express per-approved-authority calendar boundaries, but persisting a choice now would encode Draft policy.
- **Recommendation:** Keep the current setting explicitly provisional and do not migrate timezone authority until PD-003 approval.
- **Priority:** P1

---

## 11. Database

### BE-DB-001 — Core persistence uses PostgreSQL Decimal fields, relations, and a single Prisma client

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Database
- **Expected:** Financial facts should use exact storage, structural integrity, indexes, and one controlled application path.
- **Current:** Money columns use `Decimal`; principal User, Wallet, Category, Transaction, and Installment relations have foreign keys and indexes; runtime code uses one adapter-backed Prisma singleton.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:153-162`; `pocket-mint-be/prisma/schema.prisma:52-85`; `pocket-mint-be/prisma/schema.prisma:124-155`; `pocket-mint-be/src/lib/prismaFactory.ts:44-70`; `pocket-mint-be/src/lib/prisma.ts:26-43`.
- **Impact:** Persistence has a sound base for exact values and controlled connection lifecycle.
- **Recommendation:** Preserve the singleton and reinforce cross-User and idempotency invariants deliberately.
- **Priority:** P3

### BE-DB-002 — Same-User relationship integrity is not structural

- **Classification:** `PARTIAL`
- **Severity:** High
- **Area:** Database integrity
- **Expected:** A Transaction and all related Wallets, Categories, and Installments should belong to the same User; backend authorization must prevent cross-User relationships.
- **Current (2026-07-14):** Foreign keys enforce referenced-row existence, not equality of `userId`. Application checks cover most create paths but the Category update gap proves that structurally valid cross-User relations can be written.
- **Evidence (2026-07-14):** `pocket-mint-be/prisma/schema.prisma:96-112`; `pocket-mint-be/prisma/schema.prisma:124-147`; `pocket-mint-be/src/services/transaction.service.ts:295-305`.
- **Impact:** A missed service check can persist cross-User metadata links without a database constraint rejecting them.
- **Recommendation:** First close application authorization gaps; then assess composite constraints or other structural reinforcement without complicating the schema unnecessarily.
- **Priority:** P0
- **Re-audit (2026-07-15, P1-05):** Remains `PARTIAL`. The Category update authorization gap cited above is closed at the application layer (see BE-AUTHZ-002); the structural half is unchanged — foreign keys still check row existence only, and the composite-constraint assessment (backlog P1-07) has not run. No reclassification.

### BE-DB-003 — Duplicate Transfer models remain blocked by PD-007

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** Medium
- **Area:** Database domain model
- **Expected:** Schema retirement must follow an approved canonical Transfer representation and a deliberate migration.
- **Current:** Active transfers use `Transaction.toWalletId`; the separate `Transfer` table and relations remain in the schema and baseline migration.
- **Evidence:** `pocket-mint-docs/docs/product/decisions/007-transfer.md:6-8`; `pocket-mint-be/prisma/schema.prisma:36-36`; `pocket-mint-be/prisma/schema.prisma:217-235`; `pocket-mint-be/prisma/migrations/20260710000000_baseline/migration.sql:150-161`.
- **Impact:** The database presents two meanings for one business concept and invites future divergent write paths.
- **Recommendation:** Resolve only after PD-007 approval, consumer inventory, and legacy-data review.
- **Priority:** P1

### BE-DB-004 — Cascade and SetNull rules can erase or make financial history ambiguous

- **Classification:** `PARTIAL`
- **Severity:** High
- **Area:** Database deletion semantics
- **Expected:** Historical Records and financial effects should remain explainable after corrections or resource lifecycle changes.
- **Current:** Deleting a Wallet cascades source Transactions and Installments; deleting a destination Wallet sets `toWalletId` to null, producing a destination-less Transfer whose full historical effect is no longer self-describing.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:145-157`; `pocket-mint-be/prisma/schema.prisma:143-147`; `pocket-mint-be/prisma/schema.prisma:204-207`; `pocket-mint-be/prisma/migrations/20260711223000_add_transaction_to_wallet/migration.sql:24-25`.
- **Impact:** Deletion can destroy facts or convert a valid Transfer into an unresolved legacy state.
- **Recommendation:** Define correction, archival, and deletion policy before altering constraints; treat current destructive paths as a known integrity risk.
- **Priority:** P0

---

## 12. Migration

### BE-MIG-001 — A coherent local migration chain exists and the schema validates

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Migration repository state
- **Expected:** Schema changes should be represented by deliberate migrations and the current schema should validate.
- **Current:** The repository contains a reconstructed baseline, password-column removal, and additive Transfer-destination migration. Fresh `prisma validate` passed.
- **Evidence:** `pocket-mint-be/prisma/migrations/20260710000000_baseline/migration.sql:1-38`; `pocket-mint-be/prisma/migrations/20260711172700_remove_local_user_password/migration.sql:1-20`; `pocket-mint-be/prisma/migrations/20260711223000_add_transaction_to_wallet/migration.sql:1-25`; `pocket-mint-be/prisma/schema.prisma:1-15`.
- **Impact:** Local schema evolution is reviewable and can support a controlled replay process.
- **Recommendation:** Preserve immutable migration history and continue verifying new migrations against disposable PostgreSQL.
- **Priority:** P3

### BE-MIG-002 — Shared-database migration state and drift are unverified

- **Classification:** `UNVERIFIED`
- **Severity:** High
- **Area:** Migration deployment state
- **Expected:** Before rollout, the target database identity, applied migrations, schema drift, backup or PITR, and environment isolation must be verified read-only.
- **Current:** No database target was authorized for this audit. Existing repository documentation records a reconstructed-history reconciliation, but it is not fresh evidence of staging or production state.
- **Evidence:** `pocket-mint-be/docs/prisma-migration-reconciliation.md:1-21`; `pocket-mint-be/docs/deployment-runbook.md:145-181`; `pocket-mint-be/.claude/skills/prisma-database.skill.md:40-70`.
- **Impact:** Deployability, pending migration state, and drift cannot be claimed.
- **Recommendation:** Run `prisma migrate status` and a read-only diff only after the exact non-production target is identified and authorized; replay the chain on disposable PostgreSQL separately.
- **Priority:** P0

### BE-MIG-003 — Prisma CLI configuration does not use the documented direct URL

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Migration connection configuration
- **Expected:** Runtime uses `DATABASE_URL`; migration commands use an explicitly confirmed direct or session-mode `DIRECT_URL` where pooling requires it.
- **Current:** `prisma.config.ts` supplies `process.env.DATABASE_URL` to Prisma CLI. `DIRECT_URL` is documented in the runbook but is not consumed by the config.
- **Evidence:** `pocket-mint-be/prisma.config.ts:1-9`; `pocket-mint-be/.claude/skills/prisma-database.skill.md:24-31`; `pocket-mint-be/docs/deployment-runbook.md:16-35`.
- **Impact:** Operators may run migration tooling through a transaction pooler or against a different URL than the runbook implies.
- **Recommendation:** Reconcile the documented and actual CLI connection contract before the next migration rollout.
- **Priority:** P1

---

## 13. Money Precision

### BE-MONEY-001 — Authoritative command inputs pass through JavaScript `Number`

- **Classification:** `NON_COMPLIANT`
- **Severity:** Critical
- **Area:** Money Precision
- **Expected:** Authoritative money must use exact decimal parsing from the request boundary onward; binary floating point must not precede validation, calculation, or persistence.
- **Current:** Wallet creation and update convert balance, Credit Limit, Interest, and Administrative Fee with `Number`. Transaction create and update validate with `Number`, store `numAmount`, and construct Decimal from the binary number.
- **Evidence:** `pocket-mint-docs/docs/ai/project-context.md:77-78`; `pocket-mint-be/src/services/wallet.service.ts:36-50`; `pocket-mint-be/src/services/wallet.service.ts:68-90`; `pocket-mint-be/src/services/wallet.service.ts:149-159`; `pocket-mint-be/src/services/transaction.service.ts:77-95`; `pocket-mint-be/src/services/transaction.service.ts:129-137`; `pocket-mint-be/src/services/transaction.service.ts:228-259`.
- **Impact:** Large integers or long decimal inputs may be rounded before Decimal storage, and malformed non-finite values can escape clean validation into unexpected errors.
- **Recommendation:** In Phase 1 parse accepted monetary strings directly into Decimal, validate with Decimal methods, and preserve exact types through Business Services.
- **Priority:** P0

### BE-MONEY-002 — API responses convert authoritative money to JSON numbers

- **Classification:** `PARTIAL`
- **Severity:** High
- **Area:** Money Precision and contracts
- **Expected:** Authoritative financial communication should preserve exact values; presentation formatting must not change recorded or calculated money.
- **Current:** Controllers convert Decimal values with `parseFloat` or `Number` and return JSON numbers. Local guidance treats this as compatibility behavior, but no exact parallel representation is supplied.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:303-307`; `pocket-mint-be/src/controllers/account.controller.ts:99-104`; `pocket-mint-be/src/controllers/transaction.controller.ts:77-100`; `pocket-mint-be/src/controllers/installment.controller.ts:27-44`; `pocket-mint-be/src/controllers/dashboard.controller.ts:14-19`.
- **Impact:** Consumers cannot rely on lossless numeric transport for values outside JavaScript's exact range or for future precision requirements.
- **Recommendation:** Define a backward-compatible exact-money API contract before changing existing response shapes; document current numeric fields as lossy display conveniences until then.
- **Priority:** P1

---

## 14. Transactions

### BE-ATOM-001 — Multi-record Transaction and Installment creation paths are atomic

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Database transactions and rollback
- **Expected:** Operations affecting multiple records or wallet effects must commit entirely or roll back entirely.
- **Current:** Regular Transaction creation, Transfer effects, update reversal and reapplication, deletion reversal, and Installment plus Transaction plus wallet changes execute inside Prisma `$transaction` callbacks.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:239-247`; `pocket-mint-be/src/services/transaction.service.ts:141-177`; `pocket-mint-be/src/services/transaction.service.ts:184-210`; `pocket-mint-be/src/services/transaction.service.ts:283-320`; `pocket-mint-be/src/services/transaction.service.ts:350-379`; `pocket-mint-be/test/transactionService.test.ts:92-163`.
- **Impact:** Partial financial mutations are prevented on the active transaction paths.
- **Recommendation:** Preserve transaction boundaries and add idempotency above them.
- **Priority:** P3

---

## 15. Idempotency

### BE-IDEM-001 — Financial mutations have no request-level idempotency contract

- **Classification:** `NON_COMPLIANT`
- **Severity:** Critical
- **Area:** Idempotency
- **Expected:** Consequential operations must be safe to retry, with backend recognition of repeated intent and durable idempotency evidence where required.
- **Current:** Wallet and Transaction mutation routes accept no idempotency key, schema tables contain no mutation deduplication record, and services create new rows on each repeated accepted request.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:43-47`; `pocket-mint-docs/docs/ai/coding-rules.md:112-112`; `pocket-mint-be/src/routes/walletRoutes.ts:14-22`; `pocket-mint-be/src/routes/transaction.routes.ts:17-20`; `pocket-mint-be/prisma/schema.prisma:20-235`.
- **Impact:** Client retries after timeouts can duplicate Income, Expense, Transfer, or Installment creation and apply financial effects more than once.
- **Recommendation:** Define a stable mutation idempotency contract and persisted evidence in Phase 1 before Automation or externally triggered writes are introduced.
- **Priority:** P0

### BE-IDEM-002 — User sync is sequentially idempotent but not concurrency-safe

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Idempotency
- **Expected:** Repeating or racing the same bootstrap intent should produce one deterministic accepted outcome.
- **Current:** Sync performs `findUnique` followed by `create`, so sequential repeats return the existing User. Concurrent first calls can race on the unique identifier and the controller does not map that conflict to the existing User or a deterministic operational response.
- **Evidence:** `pocket-mint-be/src/controllers/user.controller.ts:53-70`; `pocket-mint-be/prisma/schema.prisma:20-23`; `pocket-mint-be/test/userSync.test.ts:49-75`.
- **Impact:** Bootstrap races can produce an unexpected 500 and cannot satisfy PD-002's atomic first-Owner requirement.
- **Recommendation:** Incorporate concurrency-safe provisioning into the approved Owner initialization and admission service design.
- **Priority:** P0

---

## 16. Validation

### BE-VAL-001 — Body validation relies on TypeScript assumptions and coercion

- **Classification:** `PARTIAL`
- **Severity:** High
- **Area:** Request validation
- **Expected:** Network input must receive runtime shape and business validation before it reaches persistence or Canonical Calculations.
- **Current:** Controllers allowlist body fields, which is positive, but body DTO types do not validate runtime values. Services use coercive `Number` checks, accept some empty-string conversions, and do not consistently reject non-finite Decimal inputs with typed operational errors.
- **Evidence:** `pocket-mint-be/docs/architecture-http-boundary.md:22-29`; `pocket-mint-be/src/controllers/transaction.controller.ts:14-32`; `pocket-mint-be/src/services/wallet.service.ts:36-50`; `pocket-mint-be/src/services/transaction.service.ts:74-95`.
- **Impact:** Malformed bodies can produce unintended zeroes, precision loss, or unexpected 500 responses instead of deterministic validation errors.
- **Recommendation:** Add focused runtime validation at the HTTP or service boundary without duplicating business rules; exact Decimal parsing is the first required control.
- **Priority:** P0

### BE-VAL-002 — Cross-field financial validation is incomplete

- **Classification:** `PARTIAL`
- **Severity:** High
- **Area:** Business validation
- **Expected:** Wallet type, Category type, related resources, amounts, and state transitions should be mutually consistent and backend-authoritative.
- **Current:** Wallet type may be changed without revalidating required Debt Wallet fields. Transaction create verifies Category ownership but does not verify Category type matches Income or Expense. Transaction update omits Category ownership and type checks. Credit Limit enforcement is blocked by PD-005.
- **Re-audit (2026-07-15, P1-05):** Category ownership on update is now enforced (see BE-AUTHZ-002). Category type matching and the other cross-field invariants above remain missing; this finding's classification is unchanged and stays with backlog P1-06.
- **Evidence:** `pocket-mint-be/src/services/wallet.service.ts:114-164`; `pocket-mint-be/src/services/transaction.service.ts:109-114`; `pocket-mint-be/src/services/transaction.service.ts:222-320`; `pocket-mint-be/prisma/schema.prisma:91-112`.
- **Impact:** Persisted relationships can be valid structurally while contradicting their business meaning.
- **Recommendation:** Inventory and specify cross-field invariants, implement approved ones in Business Services, and leave PD-005-dependent rules blocked.
- **Priority:** P0

### BE-VAL-003 — Query shape and business-date validation are explicit

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Query and date validation
- **Expected:** Query objects and arrays must not leak into persistence filters; date-only and full-timestamp inputs must be unambiguous under the Reporting Timezone.
- **Current:** Query helpers reduce values to safe scalars, and business-date parsing accepts date-only local-midnight values or full timestamps with an explicit offset.
- **Evidence:** `pocket-mint-be/src/http/queryParsers.ts:18-55`; `pocket-mint-be/src/domain/reportingTime.ts:112-124`; `pocket-mint-be/test/queryParsers.test.ts:8-58`; `pocket-mint-be/test/reportingTime.test.ts:46-58`.
- **Impact:** Inspected query filters and Transaction dates avoid ambiguous object coercion and server-local date assumptions.
- **Recommendation:** Preserve these focused validators while PD-003 determines final Reporting Timezone authority.
- **Priority:** P3

---

## 17. Logging

### BE-LOG-001 — Structured logger recursively redacts sensitive metadata

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Logging and privacy
- **Expected:** Logs must not contain tokens, authorization headers, credentials, connection URLs, or unnecessary private financial content.
- **Current:** Logger output is structured JSON, recursively redacts sensitive key fragments, handles cycles and bounded depth, and is used for authentication and database failures without logging raw credentials.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:284-301`; `pocket-mint-be/src/utils/logger.ts:18-40`; `pocket-mint-be/src/utils/logger.ts:43-73`; `pocket-mint-be/src/utils/logger.ts:78-95`; `pocket-mint-be/test/logger.test.ts:4-76`.
- **Impact:** Accidental metadata inclusion has meaningful defense against secret disclosure.
- **Recommendation:** Preserve redaction and route all new operational logs through this logger.
- **Priority:** P3

### BE-LOG-002 — Request logging and correlation are inconsistent

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Logging and observability
- **Expected:** Operational logs should identify requests, outcomes, failures, and correlations without exposing private data.
- **Current:** Express uses unstructured `morgan('dev')`; startup uses direct `console.log`; a `requestId` is generated only after an error and is not attached to the incoming request or success logs.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:280-301`; `pocket-mint-be/src/app.ts:17-20`; `pocket-mint-be/src/server.ts:24-27`; `pocket-mint-be/src/middlewares/error.middleware.ts:45-64`.
- **Impact:** Cross-service or multi-log request tracing is unavailable for successful requests and inconsistent for failures.
- **Recommendation:** Define one privacy-safe request-correlation and access-log contract before expanding Automation or external integrations.
- **Priority:** P1

---

## 18. Error Handling

### BE-ERR-001 — Unexpected and operational errors have safe central handling

- **Classification:** `COMPLIANT`
- **Severity:** Informational
- **Area:** Error Handling
- **Expected:** Errors should be deterministic and safe to expose; unexpected internals must be redacted and correlated.
- **Current:** Typed operational errors preserve safe status, code, and message. Unexpected failures reach a central handler that returns a generic production 500 with a request ID and logs through the redacting logger.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:287-291`; `pocket-mint-be/src/http/forwardError.ts:29-49`; `pocket-mint-be/src/middlewares/error.middleware.ts:39-73`; `pocket-mint-be/test/errorHandler.test.ts:27-75`.
- **Impact:** Prisma and stack internals are not exposed by inspected error paths.
- **Recommendation:** Preserve central forwarding and typed domain errors.
- **Priority:** P3

### BE-ERR-002 — Error envelopes and defensive authentication messages are inconsistent

- **Classification:** `PARTIAL`
- **Severity:** Medium
- **Area:** Error Handling and API contracts
- **Expected:** Equivalent failures should use stable codes and response shapes without stale authentication guidance.
- **Current:** The rate-limit response omits a machine `code`; authentication 401 omits `statusCode`; some controller guards return 400 with obsolete “body or API key auth” wording while middleware returns 401. The Dashboard success shape is intentionally bare, but error differences are not equally documented as compatibility contracts.
- **Evidence:** `pocket-mint-be/src/middleware/rateLimit.ts:38-43`; `pocket-mint-be/src/middleware/apiKeyAuth.ts:29-34`; `pocket-mint-be/src/controllers/transaction.controller.ts:169-174`; `pocket-mint-be/src/utils/response.ts:16-43`; `pocket-mint-be/src/controllers/dashboard.controller.ts:22-38`.
- **Impact:** Clients must handle inconsistent machine fields and can receive misleading remediation text.
- **Recommendation:** Inventory existing error contracts and reconcile them deliberately without applying an unreviewed global-envelope rewrite.
- **Priority:** P1

---

## 19. Automation

### BE-AUTO-001 — Installment scheduling and catch-up are blocked by PD-003

- **Classification:** `BLOCKED_BY_PD`
- **Severity:** High
- **Area:** Automation
- **Expected:** Scheduled Installment progression, retry, catch-up, and state distinctions require approved lifecycle policy and durable idempotency evidence.
- **Current:** No scheduler, job registration, retry store, catch-up processor, or automated lifecycle command exists.
- **Evidence:** `pocket-mint-docs/docs/development/implementation-roadmap.md:329-354`; `pocket-mint-docs/docs/product/decisions/003-installment-lifecycle.md:6-8`; `pocket-mint-be/src/routes/installmentRoutes.ts:1-14`; `pocket-mint-be/src/services/installment-query.service.ts:1-18`.
- **Impact:** Installment Automation cannot progress, and implementing it now would encode Draft behavior without idempotency.
- **Recommendation:** Approve PD-003 and complete earlier Financial Core, Controlled Access, Installment, and Reporting dependencies before Automation work.
- **Priority:** P0

### BE-AUTO-002 — General Automation adapters are not in the current backend scope

- **Classification:** `OUT_OF_SCOPE`
- **Severity:** Informational
- **Area:** Automation
- **Expected:** Secure Webhooks, n8n Integration, WhatsApp Parsing, imports, and notifications are later roadmap work and must use the same Backend and Business Services.
- **Current:** No runtime webhook adapter, queue, scheduler, n8n adapter, message parser, notification service, or automation identity implementation exists. The reconciliation CLI is an operator diagnostic, not an Automation integration.
- **Evidence:** `pocket-mint-docs/docs/development/implementation-roadmap.md:329-354`; `pocket-mint-be/package.json:14-36`; `pocket-mint-be/src/scripts/reconcile.ts:1-16`.
- **Impact:** No current Automation compliance can be measured beyond confirming absence; the backend is not ready for automated writes because idempotency and admission remain incomplete.
- **Recommendation:** Keep general Automation out of scope until Phase 7 prerequisites are met and its product contracts are approved.
- **Priority:** P2

---

## 20. AI Integration

### BE-AI-001 — AI Integration is absent and remains a later roadmap phase

- **Classification:** `OUT_OF_SCOPE`
- **Severity:** Informational
- **Area:** AI Integration
- **Expected:** AI may only extract, classify, propose, or explain through minimized authorized backend boundaries; it may not authenticate, authorize, calculate, confirm, persist, or invent history.
- **Current:** No AI SDK, AI route, prompt, model client, AI persistence path, or AI-specific authorization exists in the backend dependencies or runtime source.
- **Evidence:** `pocket-mint-docs/docs/architecture/system-architecture.md:177-188`; `pocket-mint-docs/docs/development/implementation-roadmap.md:362-389`; `pocket-mint-be/package.json:14-36`; `pocket-mint-be/src/routes/index.ts:1-17`.
- **Impact:** There is no current AI attack or data path to assess, but prerequisite backend contracts are not ready for safe AI integration.
- **Recommendation:** Keep AI out of scope until Phase 8 and require proposals to traverse approved Automation, Backend API, validation, confirmation, and audit boundaries.
- **Priority:** P2

---

## Overall Compliance

### Overall Compliance: 63% (re-audit 2026-07-15; initial audit 58%)

Scoring is finding-based and intentionally transparent:

- `COMPLIANT` = 1 point.
- `PARTIAL` = 0.5 point.
- `NON_COMPLIANT` = 0 points.
- `BLOCKED_BY_PD`, `UNVERIFIED`, and `OUT_OF_SCOPE` are excluded from the denominator because they cannot be fairly scored as approved implemented behavior.

Calculation after the 2026-07-15 re-audit (BE-AUTHZ-002 and BE-TX-002 moved to `COMPLIANT`):

```text
(16 COMPLIANT + (17 PARTIAL × 0.5)) ÷
(16 COMPLIANT + 17 PARTIAL + 6 NON_COMPLIANT)

= 24.5 ÷ 39
= 62.82%
= 63% rounded
```

Initial 2026-07-14 calculation, preserved for the record:

```text
(14 COMPLIANT + (17 PARTIAL × 0.5)) ÷
(14 COMPLIANT + 17 PARTIAL + 8 NON_COMPLIANT)

= 22.5 ÷ 39
= 57.69%
= 58% rounded
```

This percentage measures observed conformance coverage, not production readiness. Of the five Critical noncompliant areas identified on 2026-07-14, Category update authorization was resolved on 2026-07-15 (P1-05). The remaining four—PD-001 Net Worth, PD-002 controlled admission, authoritative money precision, and financial mutation idempotency—carry more release risk than an unweighted percentage expresses.

### Category Assessment

| Category | Assessment | Principal reason |
|---|---|---|
| Repository structure | Partial | Clear layering, incorrect README. |
| Architecture | Partial | Core flow is sound; User provisioning and wallet calculations cross boundaries. |
| Authentication | Partial | JWT-only is strong; PD-002 admission is absent and success tests fail. |
| Authorization | Compliant (re-audit 2026-07-15) | Routes are scoped; the Category update cross-User gap was closed by P1-05. |
| Business Services | Partial | Main financial services exist; admission and summary ownership are incomplete. |
| Wallet Engine | Non-compliant | PD-001 calculation is wrong and deletion destroys history. |
| Transaction Engine | Partial | Atomic transfer is strong; the update authorization gap was closed by P1-05 (re-audit 2026-07-15); the wallet-filter gap remains. |
| Installment Engine | Blocked | PD-003 through PD-005 remain Draft. |
| Financial Calculations | Partial | Domain helpers are deterministic; Net Worth and explainability contracts are not. |
| Reporting | Partial / blocked | Monthly aggregate exists; full Reports and timezone authority are Draft-dependent. |
| Database | Partial | Exact storage and singleton are strong; deletion, ownership, idempotency, and duplicate Transfer model remain. |
| Migration | Partial / unverified | Local chain validates; deployed state is unknown and URL contract conflicts. |
| Money Precision | Non-compliant | Command inputs pass through binary floating point. |
| Transactions | Compliant | Inspected multi-record financial operations are atomic. |
| Idempotency | Non-compliant | No financial request deduplication contract or evidence. |
| Validation | Partial | Query/date handling is strong; body and cross-field validation are incomplete. |
| Logging | Partial | Redaction is strong; request correlation is incomplete. |
| Error Handling | Partial | Central safety is strong; envelopes and messages are inconsistent. |
| Automation | Out of scope / blocked | No general adapters; Installment Automation awaits PD-003 and earlier phases. |
| AI Integration | Out of scope | No AI runtime exists; Phase 8 prerequisites are incomplete. |

---

## Top 20 Risks

Risks are ordered by the Implementation Roadmap dependency sequence, then by severity within a phase.

| Rank | Roadmap phase | Risk | Related findings |
|---:|---|---|---|
| 1 | Phase 0 | Backend-local guidance and tests preserve assets-only Net Worth despite approved PD-001. | BE-WAL-002, BE-FIN-001 |
| 2 | Phase 0 | Draft Installment, fee, debt-limit, reporting, and Transfer choices are already partly represented in runtime/schema and may be mistaken for approval. | BE-WAL-003, BE-TX-004, BE-INS-001, BE-INS-002, BE-INS-003 |
| 3 | Phase 1 | Authoritative money is converted through JavaScript `Number` before Decimal persistence. | BE-MONEY-001, BE-VAL-001 |
| 4 | Phase 1 | Financial mutations can be duplicated after retries because no idempotency contract exists. | BE-IDEM-001 |
| 5 | Phase 1 | Transaction update can persist a Category belonging to another User. **Resolved 2026-07-15 (P1-05):** application-level ownership validation and regression coverage added; the structural half (BE-DB-002) remains open under P1-07. | BE-AUTHZ-002, BE-TX-002, BE-DB-002 |
| 6 | Phase 1 | Dashboard and wallet summaries omit Reporting Cutoff and provenance. | BE-FIN-003 |
| 7 | Phase 1 | Wallet and Transaction deletion can remove or orphan Financial Events and invalidate Historical Records. | BE-WAL-004, BE-DB-004 |
| 8 | Phase 1 | API JSON numbers do not provide a lossless authoritative money representation. | BE-MONEY-002 |
| 9 | Phase 1 | Cross-field validation permits structurally valid but semantically inconsistent financial data. | BE-VAL-002 |
| 10 | Phase 2 | Any valid Supabase JWT can provision a local User without Owner approval. | BE-AUTH-002 |
| 11 | Phase 2 | No Owner, installation initialization, invitation, allowlist, or admission state exists in persistence. | BE-AUTH-002 |
| 12 | Phase 2 | JWT success-path tests currently fail and the cause is unverified. | BE-AUTH-003 |
| 13 | Phase 2 | Concurrent first sync requests are not atomic or deterministically idempotent. | BE-IDEM-002 |
| 14 | Phase 2 | `TRUST_PROXY=true` can weaken the pre-authentication rate-limit identity. | BE-AUTH-004 |
| 15 | Phase 3 | Debt-derived values are calculated with floating point in a controller and Debt Ratio is absent. | BE-ARCH-003 |
| 16 | Phase 4 | Wallet-filtered Financial Events omit incoming Transfers. | BE-TX-003 |
| 17 | Phase 5 | Existing Installments never progress beyond their initial term and later payments are not recorded. | BE-INS-001 |
| 18 | Phase 5 | Advertised Administrative Fees are not included in saved Installment obligations. | BE-INS-002 |
| 19 | Phase 6 | Complete Report contracts and authoritative Reporting Timezone behavior are unavailable. | BE-REP-002, BE-REP-003 |
| 20 | Phase 7 | Automation would inherit missing idempotency, origin, audit, and admission controls. | BE-AUTO-001, BE-AUTO-002, BE-IDEM-001 |

---

## Suggested Engineering Backlog

The backlog is architecture-ordered, not time-estimated. Draft-dependent work remains a documentation gate and is not implementation authorization.

### Phase 0 — Documentation and Repository Preparation

1. Synchronize `pocket-mint-be/.claude/skills/financial-logic.skill.md`, backend architecture documents, and affected test descriptions with approved PD-001; remove assets-only Net Worth claims.
2. Replace the stale Next.js README with backend repository guidance and authoritative documentation links.
3. Approve and synchronize PD-003 through PD-008 only in their actual implementation order; do not batch their implementation.
4. Define the missing backend contracts for Financial Summary cutoff/provenance, controlled admission, exact money transport, mutation idempotency, correction/deletion, and deterministic errors.
5. Reconcile `DIRECT_URL` documentation with the actual Prisma CLI configuration.

### Phase 1 — Financial Core

6. Implement PD-001 Net Worth in the shared calculation, Wallet query service, Dashboard query service, serializers, and tests while retaining Total Assets and Total Outstanding Debt as separate components.
7. Parse authoritative monetary inputs directly into Decimal and reject malformed or non-finite values with typed errors before persistence.
8. Add Category ownership validation to Transaction update and regression coverage for cross-User assignment. **Done 2026-07-15 (P1-05; see Re-audit Log).**
9. Define and implement persistent idempotency for consequential Wallet, Transaction, Transfer, and Installment mutations.
10. Define a Financial Summary contract with exact components, Reporting Cutoff, calculation identity, and provenance.
11. Resolve correction, archival, and deletion semantics before changing cascade behavior; prevent deletion from silently destroying or orphaning Ledger facts.
12. Complete approved cross-field validation for wallet types, Category types, resource relationships, amounts, and state transitions.
13. Define a backward-compatible exact-money response contract; treat existing JSON numbers as compatibility fields until migration is complete.

### Phase 2 — Authentication and Controlled Access

14. Design the backend admission model for installation initialization, one active Owner, invitation or allowlist state, revocation, recovery, and existing-installation migration.
15. Move provisioning from `UserController` into a concurrency-safe admission service and enforce atomic first-Owner creation.
16. Prevent `/users/sync` from admitting a post-initialization User without approved admission evidence.
17. Diagnose and resolve the three JWT success-path test failures before treating authentication verification as green.
18. Reject `TRUST_PROXY=true` and require a validated numeric hop count.

### Phase 3 — Wallet Engine

19. Move exact Outstanding Balance and Available Credit calculations out of the controller and into backend-owned result contracts.
20. After PD-005 approval, implement the chosen Credit Limit and Debt Ratio policy with data review and migration behavior for existing Debt Wallets.

### Phase 4 — Transaction and Transfer Engine

21. Define wallet-related Financial Event query semantics so incoming and outgoing Transfers both explain Wallet Balance changes.
22. After PD-007 approval, inventory consumers and legacy data, then retire or justify the unused Transfer model through a deliberate migration.

### Phase 5 — Installment Engine

23. After PD-003 approval, define and implement durable schedule identity, term progression, Installment Payments, catch-up, settlement, payoff, cancellation, and lifecycle idempotency.
24. After PD-004 approval, make fee preview, calculation, persistence, Total Obligation, Outstanding Balance, and reporting agree exactly once.
25. After PD-005 approval, apply the canonical Credit Limit rule to Installment creation.
26. Define reverse calculation rounding and impossible-input behavior before adding the reverse calculator contract.

### Phase 6 — Reporting

27. After PD-006 and PD-003 approval, define canonical report questions, periods, provenance, partial failures, and missing-history gaps.
28. Return Reporting Timezone and completeness context with period aggregates.
29. Build Reports from Ledger and Historical Records only; do not materialize a competing source of truth.

### Phase 7 — Automation

30. Add Automation only after stable authenticated commands, idempotency, origin, audit, retry, catch-up, and User confirmation contracts exist.
31. Keep all automated writes on the same Backend API and Business Service paths as interactive writes.

### Phase 8 — AI

32. Keep AI absent until a bounded proposal or explanation contract is approved.
33. Require minimized context, explicit origin and uncertainty, backend validation, and required User confirmation before any AI-proposed mutation.

### Cross-Cutting Verification Backlog

34. Add a non-mutating CI or audit check that detects repository-local claims contradicting Approved Product Decisions.
35. Add privacy-safe request correlation across access logs, errors, and future automated jobs.
36. Reconcile rate-limit and authentication error envelopes with stable machine codes without breaking documented compatibility.
37. Verify the migration chain on disposable PostgreSQL and separately verify authorized staging drift before rollout.
38. Run the full repository verification gate after each approved implementation phase: typecheck, build and committed `dist` comparison, full tests, Prisma validate, migration replay where applicable, diff check, and status inspection.

---

## Audit Constraints and Unverified Areas

- No source, schema, migration, generated output, configuration, or implementation file was modified.
- The only created artifact is this audit report in `pocket-mint-docs`.
- No external Supabase configuration, project admission settings, JWT key state, Railway topology, database contents, migration table, drift, backups, deployed revision, or runtime logs were available.
- No build was run because the repository's build writes committed `dist/` and generated Prisma files.
- No migration, database status, reconciliation, or database integration command was run because no exact disposable or shared database target was identified and authorized.
- Static absence of Automation or AI code does not prove that no external process calls the backend; external integrations are `UNVERIFIED` outside this repository audit.
- Existing untracked `.claude/settings.local.json` in `pocket-mint-be` and existing untracked documentation work in `pocket-mint-docs` were preserved without modification.
