# Backend Engineering Backlog

> Source: [Backend Architecture Audit](./backend-audit.md) (2026-07-14) transformed against the [Implementation Roadmap](./implementation-roadmap.md).
> Scope: `pocket-mint-be`, with documentation prerequisites in `pocket-mint-docs`.
> This backlog converts audit findings into engineering work. It does not restate the audit, does not authorize Draft Product Decision behavior, and does not estimate time. Finding IDs in each description trace back to the audit. Tasks gated by a Draft Product Decision list that approval in **Depends On** and must not start before it.
>
> `COMPLIANT` findings (BE-STR-002, BE-ARCH-001, BE-AUTH-001, BE-AUTHZ-001, BE-SVC-001, BE-WAL-001, BE-TX-001, BE-FIN-002, BE-DB-001, BE-MIG-001, BE-ATOM-001, BE-VAL-003, BE-LOG-001, BE-ERR-001) require no work and produce no tasks; their preservation rules are acceptance constraints on the tasks below.

---

## Phase 0 — Documentation and Repository Preparation

### Epic: Documentation Authority Baseline

Establishes the approved-documentation baseline every implementation task depends on. Merges BE-STR-001, BE-MIG-003, and the documentation halves of BE-WAL-002 / BE-FIN-001, BE-FIN-003, BE-AUTH-002, BE-MONEY-001/002, BE-IDEM-001, BE-WAL-004 / BE-DB-004, BE-ERR-002.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P0-01 | P0 | Synchronize `pocket-mint-be/.claude/skills/financial-logic.skill.md`, backend architecture documents, and affected test descriptions with approved PD-001; remove all assets-only Net Worth claims (BE-WAL-002, BE-FIN-001) | — | No repository-local guidance or test description asserts Net Worth = Total Assets; all guidance states Net Worth = Total Assets − Total Outstanding Debt at one Reporting Cutoff | High | pocket-mint-be, pocket-mint-docs | TODO |
| P0-02 | P0 | Document the missing backend contracts: Financial Summary cutoff/provenance, controlled admission, exact money transport, mutation idempotency, correction/deletion policy, deterministic error envelopes (audit backlog item 4) | P0-01 | Each contract is documented in authority order and reviewable before its dependent implementation task starts; Draft-PD-governed portions are explicitly deferred | High | pocket-mint-docs, pocket-mint-be | TODO |
| P0-03 | P1 | Replace the stale Next.js `README.md` with backend repository purpose, architecture entry points, commands, environment boundaries, and links to authoritative documentation (BE-STR-001) | — | README identifies the Express 5 / TypeScript / Prisma backend; no `create-next-app` or `app/page.tsx` guidance remains | Low | pocket-mint-be | TODO |
| P0-04 | P1 | Reconcile `DIRECT_URL` documentation with the actual Prisma CLI connection configuration in `prisma.config.ts` (BE-MIG-003) | — | Deployment runbook, skill docs, and `prisma.config.ts` agree on one migration connection contract; operators cannot follow two conflicting URL instructions | Medium | pocket-mint-be | TODO |
| P0-05 | P1 | Drive PD-003 through PD-008 to individual approval and authority-order synchronization in their actual implementation order; do not batch implementation (audit backlog item 3) | — | Each PD is individually Approved and synchronized before its governed phase's gated tasks start; approval status recorded in the individual decision files | Medium | pocket-mint-docs | TODO |

---

## Phase 1 — Financial Core

### Epic: Financial Correctness

One epic for canonical money semantics end to end. Merges Money Precision (BE-MONEY-001, BE-MONEY-002, BE-VAL-001), Net Worth (BE-WAL-002, BE-FIN-001), and Financial Summary (BE-FIN-003) — same financial truth, one body of work.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P1-01 | P0 | Implement PD-001 Net Worth (Total Assets − Total Outstanding Debt at one Reporting Cutoff) in the shared calculation, Wallet query service, Dashboard query service, serializers, and tests; retain Total Assets and Total Outstanding Debt as separate components (BE-WAL-002, BE-FIN-001) | P0-01 | `calculateNetWorth` returns assets minus debt; `totalAssets` remains a separate labeled field; all Wallet and Dashboard endpoints report PD-001 semantics; tests assert the approved calculation, not the contradiction | High | pocket-mint-be | TODO |
| P1-02 | P0 | Parse authoritative monetary inputs (balance, Credit Limit, Interest, Administrative Fee, amounts) directly into Decimal at the request boundary; reject malformed, empty-string, and non-finite values with typed operational errors before persistence (BE-MONEY-001, BE-VAL-001) | P0-02 | No `Number` coercion precedes Decimal construction on Wallet or Transaction command paths; malformed bodies return deterministic validation errors, never 500 or silent zeroes; regression tests cover large-integer and long-decimal inputs | High | pocket-mint-be | TODO |
| P1-03 | P0 | Implement the canonical Financial Summary contract: exact components, Reporting Cutoff, calculation identity, and provenance for Dashboard and wallet mutation snapshots (BE-FIN-003) | P0-02, P1-01 | Summary responses identify their calculation, time boundary, and supporting facts; all components in one response share one Reporting Cutoff; contract is reusable by Reports, Automation, and AI phases | High | pocket-mint-be, pocket-mint-docs | TODO |
| P1-04 | P1 | Implement the backward-compatible exact-money response contract; document existing JSON-number fields as lossy display conveniences until migration completes (BE-MONEY-002) | P0-02 | An exact representation of authoritative money is available in responses without breaking existing consumers; the compatibility status of numeric fields is documented | Medium | pocket-mint-be | TODO |

### Epic: Ownership and Validation Integrity

Merges the cross-User Category gap reported three times (BE-AUTHZ-002, BE-TX-002, BE-DB-002) with cross-field validation (BE-VAL-002) — one authorization-invariant body of work.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P1-05 | P0 | Add Category ownership validation (`{ id: categoryId, userId }`) to Transaction update before the mutation transaction; add regression coverage proving cross-User Category assignment is rejected (BE-AUTHZ-002, BE-TX-002) | — | Update enforces the same ownership invariants as create for every changed relationship identifier; regression test demonstrates rejection of another User's Category ID | High | pocket-mint-be | DONE (2026-07-15; audit re-audit reclassified BE-AUTHZ-002 and BE-TX-002 `COMPLIANT`) |
| P1-06 | P0 | Complete approved cross-field validation: wallet type changes revalidate required Debt Wallet fields; Category type must match Income/Expense on create and update; related-resource consistency enforced on all mutation paths. PD-005-dependent Credit Limit rules stay blocked (BE-VAL-002) | P1-05 | Each specified invariant is enforced in Business Services with typed errors and covered by tests; deferred PD-005 rules are documented as blocked, not silently implemented | High | pocket-mint-be | TODO |
| P1-07 | P1 | Assess structural same-User relationship reinforcement (e.g., composite constraints) after application-level gaps are closed (BE-DB-002) | P1-05, P1-06 | Written assessment of structural options exists; any adopted constraint ships as a deliberate reviewed migration; no unnecessary schema complication | Medium | pocket-mint-be | TODO |

### Epic: Mutation Idempotency

Merges BE-IDEM-001 with the idempotency prerequisites that Phases 5 and 7 inherit.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P1-08 | P0 | Implement persistent request-level idempotency for consequential Wallet, Transaction, Transfer, and Installment mutations per the documented contract (BE-IDEM-001) | P0-02 | Mutation routes accept a stable idempotency key; a repeated accepted request produces exactly one financial effect and a deterministic response; durable deduplication evidence is persisted; retry-after-timeout tests prove no duplicate Income, Expense, Transfer, or Installment | High | pocket-mint-be | TODO |

### Epic: Historical Record Integrity

Merges wallet force-deletion (BE-WAL-004) and cascade/SetNull semantics (BE-DB-004) — one correction/deletion policy problem.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P1-09 | P0 | Resolve correction, archival, and deletion policy in documentation, then change runtime and schema behavior so deletion cannot silently destroy or orphan Ledger facts: `DELETE /wallets/:id?force=true` cascade of Transactions/Installments and destination-wallet `SetNull` producing destination-less Transfers (BE-WAL-004, BE-DB-004) | P0-02 | Policy is documented before runtime change; wallet deletion no longer erases Financial Events contrary to policy; destination-null Transfer states are prevented or explicitly represented; constraint changes ship as reviewed migrations | High | pocket-mint-docs, pocket-mint-be | TODO |

---

## Phase 2 — Authentication and Controlled Access

### Epic: Owner-First Admission (PD-002)

Merges the open `/users/sync` provisioning (BE-AUTH-002), controller-owned provisioning (BE-ARCH-002, BE-SVC-002), and the non-concurrency-safe bootstrap (BE-IDEM-002) — one admission-service body of work.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P2-01 | P0 | Design and document the backend admission model: installation initialization state, single active Owner, invitation or allowlist representation, revocation, recovery boundaries, and existing-installation migration (BE-AUTH-002, audit backlog item 14) | P0-02 | Documented schema, state machine, migration path, and recovery boundaries exist and are approved in authority order before implementation begins | High | pocket-mint-docs, pocket-mint-be | TODO |
| P2-02 | P0 | Move User provisioning out of `UserController` into a concurrency-safe admission Business Service with atomic first-Owner creation (BE-ARCH-002, BE-SVC-002, BE-IDEM-002) | P2-01 | Controller performs HTTP mapping only; concurrent first-sync races resolve to one deterministic Owner outcome without 500; unique-identifier conflicts map to the existing User or a deterministic operational response | High | pocket-mint-be | TODO |
| P2-03 | P0 | Enforce closed admission on `POST /users/sync`: after initialization, a valid Supabase JWT alone must not provision a local User without Owner-approved invitation or allowlist evidence (BE-AUTH-002) | P2-02 | Post-initialization callers without admission evidence are rejected; tests cover first-Owner bootstrap, admitted-User, and rejected-caller paths; public self-registration is impossible | High | pocket-mint-be | TODO |

### Epic: Authentication Verification Health

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P2-04 | P0 | Diagnose and resolve the 3 failing JWT success-path tests (`401 invalid_token`) as a read-only-first debugging task; determine whether the cause is `jose`, fixture configuration, environment certificate state, or runtime code (BE-AUTH-003) | — | Full `npx vitest run` is green; root cause is documented; the suite is trustworthy as the authentication release gate | High | pocket-mint-be | TODO |

### Epic: Request Trust Hardening

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P2-05 | P1 | Enforce the numeric-hop-only `TRUST_PROXY` contract; fail production startup on `TRUST_PROXY=true` (BE-AUTH-004) | — | `parseTrustProxy` rejects `true` in production with a startup failure; numeric hop counts are validated; behavior matches the documented contract and is tested | Medium | pocket-mint-be | TODO |

---

## Phase 3 — Wallet Engine

### Epic: Backend-Owned Debt Calculations

Merges controller-local debt arithmetic (BE-ARCH-003) with the service-ownership gap it exemplifies (BE-SVC-002).

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P3-01 | P0 | Move Outstanding Balance and Available Credit calculation out of `serializeWallet` into a service-owned exact Decimal debt-summary contract; the controller serializes service-produced values only (BE-ARCH-003) | P1-02, P1-03 | No floating-point arithmetic or `Math.abs` in controllers; debt-derived values come exact from Business Services and are reusable by Reports, Automation, and other clients; PD-005-dependent thresholds remain blocked | High | pocket-mint-be | TODO |
| P3-02 | P0 | **PD-005 gated.** Implement the approved Credit Limit enforcement, Debt Ratio thresholds, over-limit handling, and PayLater risk policy, including data review and migration behavior for existing Debt Wallets (BE-WAL-003) | PD-005 approval, P3-01 | The chosen constraint model is implemented exactly as approved; existing over-limit wallets follow the approved migration behavior; Debt Ratio is returned per the approved contract | High | pocket-mint-be | TODO |

---

## Phase 4 — Transaction and Transfer Engine

### Epic: Wallet Event Query Completeness

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P4-01 | P1 | Define the wallet-related Financial Event query contract and include incoming Transfers (`toWalletId` matches) in wallet-filtered Transaction listing, consistent with sparkline queries (BE-TX-003) | P0-02, PD-007 synchronization | A wallet's filtered event list explains its full balance, including events where the wallet is source or destination; contract is documented; tests cover both directions | Medium | pocket-mint-be | TODO |

### Epic: Canonical Transfer Representation (PD-007)

Merges the runtime/schema duplication (BE-TX-004) and its database counterpart (BE-DB-003).

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P4-02 | P1 | **PD-007 gated.** Inventory external consumers and legacy data of the unused `Transfer` model, then retire it or explicitly justify retention through a deliberate migration (BE-TX-004, BE-DB-003) | PD-007 approval | One documented canonical Transfer representation exists; the unused model is removed or its retention documented; the migration is reviewed and replayed on disposable PostgreSQL | Medium | pocket-mint-be | TODO |

---

## Phase 5 — Installment Engine

### Epic: Approved Installment Contract (PD-003 / PD-004 / PD-005)

Merges BE-INS-001, BE-INS-002, and BE-INS-003 — one installment contract delivered against three PD gates.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P5-01 | P0 | **PD-003 gated.** Implement durable schedule identity, automatic term progression, Installment Payment records, catch-up, settlement, payoff, cancellation, and lifecycle idempotency; include migration for existing Installments stuck at `currentTerm = 1` (BE-INS-001, BE-INS-003) | PD-003 approval, P1-08, Phases 1–4 complete | Lifecycle behavior matches approved PD-003 exactly; no debt effect applies twice; repeated or delayed progression is idempotent; existing Installments migrate per approved policy | High | pocket-mint-be | TODO |
| P5-02 | P0 | **PD-004 gated.** Make fee preview, calculation, persistence, Total Obligation, Outstanding Balance, and reporting agree exactly once; review existing saved obligations that exclude advertised fees (BE-INS-002) | PD-004 approval, P1-02 | Saved Total Obligation matches the approved fee model; the static rates endpoint and persisted fee fields stop being provisional; existing-data review and migration are complete | High | pocket-mint-be | TODO |
| P5-03 | P0 | **PD-005 gated.** Apply the canonical Credit Limit rule to Installment creation (BE-INS-003, audit backlog item 25) | PD-005 approval, P3-02 | Installment creation enforces the approved limit behavior; tests cover at-limit and over-limit creation attempts | High | pocket-mint-be | TODO |
| P5-04 | P1 | Define reverse-calculation rounding and impossible-input behavior before adding the reverse calculator contract (BE-INS-003, audit backlog item 26) | PD-003 and PD-004 synchronization | Rounding rules and rejection semantics for impossible reverse inputs are documented; no reverse calculator ships before those definitions | Medium | pocket-mint-docs, pocket-mint-be | TODO |

---

## Phase 6 — Reporting

### Epic: Explainable Reporting Surface (PD-006 / PD-003)

Merges BE-REP-001, BE-REP-002, and BE-REP-003 — one report-contract body of work.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P6-01 | P0 | **PD-006 + PD-003 gated.** Define canonical report questions, Reporting Periods, provenance, partial-failure contracts, and missing-history behavior before any report API implementation (BE-REP-002) | PD-006 approval, PD-003 approval, P1-03 | The Reports contract (scope, Dashboard boundary, periods, provenance, partial failures, missing-history gaps) is documented and approved before routes or services exist | High | pocket-mint-docs, pocket-mint-be | TODO |
| P6-02 | P1 | Return Reporting Timezone, provenance, and completeness context with period aggregates; keep the process-wide `Asia/Jakarta` default explicitly provisional until PD-003 assigns timezone authority (BE-REP-001, BE-REP-003) | P6-01 | Monthly summary responses carry Reporting Timezone and completeness/provenance context; timezone authority is not persisted before PD-003 approval; exact Decimal aggregation is preserved | Medium | pocket-mint-be | TODO |
| P6-03 | P1 | Build Reports exclusively from Ledger facts and Historical Records; do not materialize a competing source of truth; show missing history as missing (BE-REP-002, audit backlog item 29) | P6-01 | Report queries derive from User-scoped Transaction facts; no synthesized history; gaps render as explicit gaps | High | pocket-mint-be | TODO |

---

## Phase 7 — Automation

### Epic: Automation Readiness (PD-003)

Merges BE-AUTO-001 and BE-AUTO-002 — Automation inherits every Phase 1–6 control before any adapter exists.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P7-01 | P0 | **PD-003 gated.** Implement scheduled Installment progression and catch-up Automation with durable idempotency evidence, origin and audit records, and retry safety (BE-AUTO-001) | PD-003 approval, P5-01, P1-08, P2-03 | Scheduled runs are idempotent under retry and delayed delivery; every automated write records origin and audit evidence; no duplicate term processing occurs | High | pocket-mint-be | TODO |
| P7-02 | P2 | Keep general Automation adapters (Secure Webhooks, n8n Integration, WhatsApp Parsing, imports, notifications) out of scope until Phase 7 prerequisites and their product contracts are approved; when added, all automated writes traverse the same authenticated Backend API and Business Services as interactive writes (BE-AUTO-002) | P7-01, approved product contracts | No adapter bypasses Business Services, ownership, validation, transactions, or idempotency; each adapter's contract is approved before implementation | Medium | pocket-mint-docs, pocket-mint-be | TODO |

---

## Phase 8 — AI

### Epic: AI Boundary Gate

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| P8-01 | P2 | Keep AI absent until a bounded proposal or explanation contract is approved; require minimized authorized read models, explicit origin and uncertainty, backend validation, and required User confirmation before any AI-proposed mutation (BE-AI-001) | Phases 1–7 complete, approved AI product contract | AI may extract, classify, propose, or explain only; it cannot authenticate, authorize, calculate canonical values, confirm, persist, or invent history; every proposal traverses Automation, Backend API, validation, confirmation, and audit boundaries | Medium | pocket-mint-docs, pocket-mint-be | TODO |

---

## Cross-Cutting — Verification and Operability

### Epic: Verification, Observability, and Deployment Confidence

Merges BE-MIG-002, BE-LOG-002, BE-ERR-002, and audit backlog items 34–38. Not phase-bound; scheduled alongside the phases they protect.

| Task ID | Priority | Description | Depends On | Acceptance Criteria | Estimated Risk | Repository | Status |
|---|---|---|---|---|---|---|---|
| XC-01 | P0 | Verify the migration chain by replay on disposable PostgreSQL, and separately verify the authorized target database: identity, `prisma migrate status`, read-only drift diff, and backup/PITR state before any rollout (BE-MIG-002) | P0-04, authorized database target | The chain replays cleanly on disposable PostgreSQL; target identity, applied migrations, drift, and backup state are recorded read-only before deployment | High | pocket-mint-be | TODO |
| XC-02 | P1 | Implement privacy-safe request correlation: attach a request ID to incoming requests, emit structured access logs for successes and failures through the redacting logger, replace `morgan('dev')` and startup `console.log` (BE-LOG-002) | — | Success and error logs for one request share one correlation ID; no tokens, credentials, or unnecessary private financial content appear in logs; the redacting logger is the single log path | Medium | pocket-mint-be | TODO |
| XC-03 | P1 | Reconcile rate-limit and authentication error envelopes with stable machine codes and consistent fields; remove stale "body or API key auth" remediation wording; document intentional compatibility shapes (BE-ERR-002) | P0-02 | Equivalent failures return stable `code`/`statusCode` fields; no misleading remediation text remains; documented compatibility contracts are preserved rather than globally rewritten | Medium | pocket-mint-be | TODO |
| XC-04 | P1 | Add a non-mutating CI or audit check that detects repository-local claims contradicting Approved Product Decisions (audit backlog item 34) | P0-01 | CI flags contradictions such as assets-only Net Worth guidance automatically; the check itself mutates nothing | Medium | pocket-mint-be, pocket-mint-docs | TODO |
| XC-05 | P1 | Run the full repository verification gate after each approved implementation phase: typecheck, build with committed `dist` comparison, full tests, `prisma validate`, migration replay where applicable, diff check, and status inspection (audit backlog item 38) | XC-01 | The gate runs at each phase boundary and results are recorded; a phase is not declared complete with a red gate | Medium | pocket-mint-be | TODO |

---

## Immediate Sprint — Top 10 Tasks

Exact implementation order. Rationale: restore the release-gate signal first, close the standalone Critical authorization gap, then land the documentation baseline that authorizes the Phase 1 financial-correctness sequence.

| Order | Task ID | Task | Why now |
|---:|---|---|---|
| 1 | P2-04 | Diagnose and fix the 3 failing JWT success-path tests | The verification suite is the release gate for everything below; it must be green and trustworthy first |
| 2 | P1-05 | Category ownership validation on Transaction update + regression test — **Done 2026-07-15** | Standalone Critical cross-User authorization gap with no dependencies |
| 3 | P0-01 | Synchronize backend-local guidance and tests with PD-001 | Documentation-first prerequisite for the Net Worth code change |
| 4 | P0-02 | Document the missing backend contracts (summary, admission, exact money, idempotency, deletion, errors) | Authorizes and specifies tasks 5–10 |
| 5 | P1-01 | Implement PD-001 Net Worth across calculation, services, serializers, tests | Highest-ranked audit risk; every Financial Summary is currently wrong when debt exists |
| 6 | P1-02 | Exact Decimal parsing of monetary inputs with typed rejection | Removes binary floating point from authoritative money before further financial work builds on it |
| 7 | P1-03 | Canonical Financial Summary contract with Reporting Cutoff and provenance | Makes task 5's output explainable and reusable; consumes tasks 4–6 |
| 8 | P1-08 | Persistent idempotency for consequential financial mutations | Prevents duplicate financial effects from retries; prerequisite for Installment lifecycle and Automation |
| 9 | P1-06 | Complete approved cross-field validation | Closes remaining semantic-consistency gaps on the now-exact mutation paths |
| 10 | P1-09 | Correction/deletion policy + stop force-cascade destroying Ledger history | Last Phase 1 P0; policy documentation from task 4 must land before the runtime change |

Next after this sprint: P2-01 (admission model design) opens Phase 2.
