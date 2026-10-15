# Pocket Mint — Release Baseline: `dev` → `main` Promotion

## Purpose

This document promotes the existing `dev` integration line to `main` for both `pocket-mint-be` and `pocket-mint-fe`. It replaces the earlier Assistant-only baseline (Phases 27–35) with a full release baseline covering every meaningful change in `origin/main..origin/dev` on both repositories, grouped by user/product outcome rather than by commit. The Phase 27–35 Assistant detail is retained as a traceable appendix (§8) rather than duplicated here.

**Scope.** Pocket Mint Assistant is a natural-language transaction *capture* interface: it helps a user create a transaction draft through chat (Web or Telegram) and confirm it. It is not a financial-advice or recommendation product. Category suggestions and merchant mapping (§3) are advisory aids to manual entry, not automated categorization. Nothing in this promotion changes either boundary.

**This is a promotion, not new work.** Everything described below is already merged into `dev` on both repositories (and, for docs, already on `pocket-mint-docs` `main`). This document and the accompanying PRs promote that existing integration line forward — no new feature work is introduced by the promotion itself.

**Evidence basis.** Sourced from `origin/main..origin/dev` commit/PR history on both repositories (backend: 131 commits / PRs #61–#107; frontend: 88 commits / PRs #53–#89), the linked Product Decisions (PD-010–PD-025), [Assistant Core Architecture §§15.7–15.13](../architecture/assistant-core-architecture.md), [ADR-010 — Analytics v2 Architecture](../product/decisions/010-analytics-v2-architecture.md), [PD-010 — Smart Categorization Foundation](../product/decisions/010-smart-categorization-foundation.md), and [PD-011 — Merchant Mapping](../product/decisions/011-merchant-mapping.md). No item below claims behavior beyond what those sources document.

**Not a public announcement.** `pocket-mint-fe`'s public changelog (`src/lib/changelog.ts`, rendered at `/changelog`) has no diff between `origin/main` and `origin/dev` — no entry for any of this promotion's work exists yet. Per that changelog's own convention (`docs/releases/README.md` §1a in `pocket-mint-fe`), completing this internal release baseline does not by itself create a public changelog entry. §7 below lays out a proposed entry and version recommendation; it has not been applied.

---

## 1. Assistant transaction capture

Covers Assistant foundation, conversation flow, natural-language capture, draft confirmation, clarification, recovery, draft correction, and reliability/lifecycle/privacy-safe operational hardening (backend PRs #65–#107 excluding #84/#95/#105; frontend PRs #60–#89). Full phase-by-phase detail is in the appendix (§8); this section is the condensed summary.

**Added**

- A full Assistant conversation UI is reachable in the product: a nav entry (desktop sidebar and mobile bottom dock) links to `/assistant`, a chat-style composer, guided and option-based clarification cards, draft review/edit/confirm/cancel, and conversation history with archive/restore/delete. All calls go through the existing API client to real backend endpoints — this is not a UI mock.
- Optional request-level idempotency for the Assistant message and execute endpoints: a retried or double-submitted request replays the original outcome instead of creating a second turn or draft.
- Recovery from interrupted or still-running Assistant work, surfaced in the Web UI as a recovery banner.
- Casual Rupiah shorthand (`25k`, `450k`, `1.5k`) and relative Indonesian date phrases ("kemarin", "tadi", "hari ini") are recognized by deterministic fallbacks, anchored to a fixed reporting-timezone calendar date.
- Draft correction (overriding amount/wallet/category/description/date at confirm time) is fully covered end-to-end, including the Web edit UI pre-selecting the correct category and date.

**Changed**

- Draft preview data now includes raw `categoryId` and an unambiguous `transactionDate` alongside display strings, so the Web edit UI can correctly preselect values when a user corrects a draft before confirming.

**Web and Telegram continuity**

- A conversation shared across Web and Telegram reports which channel each turn originated from, shown in the Web UI as a subtle badge.
- Delivery state for Telegram-originated Assistant replies (pending/processing/delivered/failed/unknown-for-aged-out-records) is surfaced in the Web UI next to the Assistant's reply.
- Telegram account linking is now available from Settings (Akun): link, view connection status, and revoke a linked Telegram account, backed by real link-token/connection/revoke endpoints. This is the account-linking counterpart that makes Telegram delivery continuity possible.
- No Telegram chat ID, user ID, provider message ID, destination chat ID, reply markup, callback token, or raw provider payload is exposed through any DTO.

**Operational safety**

- Read-only operator visibility reporting for ambiguous Assistant/callback executions, terminal channel failures, and stale in-progress turns.
- Bounded, explicit, single-row operator remediation actions, each dry-run-by-default and safety-gated — no "fix all" mode, no automatic replay or re-execution anywhere in this range.
- Privacy-safe, bounded capture-lifecycle operational events, extending the existing closed, redacted event taxonomy. No raw message content, amounts, IDs, or provider payloads are logged.

**Unchanged by design**

- Draft confirmation remains mandatory; no transaction is created until the user confirms.

---

## 2. Analytics and reporting

Backend PR #61 (`feature/analytics-v2`); frontend PR #55 (`feature/analytics-v2`) plus PR #76 (responsiveness and a response-unwrap fix).

**Added**

- A backend-authoritative `/api/v1/analytics` module (six endpoints: overview, trends, category breakdown, wallet breakdown, budget-performance, transaction drill-down), replacing the previous client-side full-transaction-dump computation on the existing `/analytics` page.
- Overview: income/expense/net cash flow compared against the previous equal-length period, with an explicit "no comparison available" state instead of a misleading percentage when the previous period has no data.
- Trend, category, and wallet breakdowns, each zero-filled/complete (no gaps, no silently-dropped zero-activity wallets or categories).
- Budget performance within Analytics reuses the existing Budgeting domain's calculation verbatim — the same numbers as the Budgets page, not a second formula.
- Transaction drill-down from any metric to its constituent transactions, paginated.

**Changed**

- The existing `/analytics` page's layout and loading behavior were improved (period selector, summary cards, cash-flow chart, category/wallet breakdowns, budget performance, drill-down table), with an accessible data-table fallback alongside every chart.

**Fixed**

- An Analytics API response-unwrapping defect found during the responsiveness pass.

**Internal only — not user-facing**

- Backend integration-test fixture fixes for current-month date assumptions (PR #95) are test-determinism work with no production-code change and are not part of this feature.

---

## 3. Smart categorization and merchant mapping

Backend PRs #62, #63; frontend PRs #56, #57 ("Phase 19").

**Added**

- Category suggestions while adding a transaction: typing a description surfaces up to 5 ranked suggestions (confidence HIGH/MEDIUM/LOW) from a deterministic Indonesian-keyword match. This is advisory only — the user always selects the category explicitly; nothing is auto-assigned.
- Merchant mapping: from Akun → Merchant Mapping, or via an opt-in "remember this merchant" checkbox when adding a transaction (unchecked by default), a user can map a merchant name to one of their own categories. A user's own mapping always takes priority over the keyword suggestion for that merchant.
- Mappings are strictly per-user — there is no shared or global merchant database, and no mapping is ever created automatically.

**Explicitly not included (by design, not a gap)**

- No automatic categorization, no learning from past corrections, no confidence adjustment over time.
- No rule-engine-based categorization — that capability does not exist in this promotion.

---

## 4. Transaction, wallet, budget, bill, savings, and navigation/UI changes

Outside of Assistant, Analytics, and categorization/merchant-mapping (§§1–3), this promotion makes no standalone changes to transaction, wallet, budget, bill, or savings-goal behavior — those domains are unchanged in `origin/main..origin/dev` beyond the read-only Analytics reuse described in §2 and a non-behavioral change enabling the Assistant to create a transaction inside its own existing draft-confirm transaction.

**Changed**

- The Savings Goals navigation entry is currently hidden behind a feature flag in the frontend; the underlying feature itself is unchanged.
- Minor navigation feedback (visual click feedback on nav links) was added alongside a Dashboard/Tagihan loading-performance fix (see §5).

---

## 5. Performance, reliability, and developer/platform improvements

**Backend**

- Category listing no longer runs its default-category upsert on every request — only when a user genuinely has none yet, removing redundant writes from the common path.
- CI now fails fast if the integration test database is misconfigured or diverges from the application database, instead of silently skipping or running against the wrong database.
- Cross-platform build fixes so committed TypeScript declaration maps stay consistent between a macOS development machine and Linux CI.

**Frontend**

- A Storybook foundation (component catalog, accessibility and interaction testing) was added, with every feature component built afterward (Assistant, Analytics, Merchant Mapping) shipping a story. Follow-up fixes corrected lockfile sync and story-discovery gaps in CI.
- A Dashboard/Tagihan navigation loading fix (LCP) and an Analytics layout-shift fix (CLS) reduce visible loading jank on those pages.
- Form-field accessibility improvements: clearer label association and error-text contrast.
- Frontend test baseline restored; no behavior change.

Routine dependency lockfile-sync commits on both repositories are omitted from this list as non-notable.

---

## 6. Known limitations

Current, intentional boundaries as of this promotion — not defects:

- **Natural-language transfer capture is not yet supported.** The Assistant can capture income/expense transactions; transfers between wallets are out of scope.
- **No financial-advice or recommendation behavior**, anywhere in the Assistant or in category suggestions.
- **Confirmation remains mandatory** before any transaction is created — no phase in this promotion introduces a path that commits a transaction without explicit user (or Telegram-interactive) confirmation.
- **No automatic retry of ambiguous financial actions.** An ambiguous or stuck execution is surfaced to an operator and may be remediated only by an explicit, single-row operator action — never automatically.
- **Category suggestions and merchant mapping are advisory and manual only.** Nothing auto-assigns a category, learns from corrections, or shares mapping data across users. A rule-engine layer (user-defined categorization rules) does not exist in this promotion.
- **Attribution/delivery-state history has bounded gaps.** Channel attribution cannot be recovered for turns whose originating record has already been retention-purged, or for pre-attribution-era callback turns — a bounded, one-time historical gap, not an ongoing one.

---

## 7. Public frontend changelog — proposed entry (not applied)

`src/lib/changelog.ts`'s `RELEASES` array has no entry for this promotion, and its `assertValidReleases`/`ReleaseStatus` validation does not support an "Unreleased" status — only a concrete `MAJOR.MINOR.PATCH` version is valid. No version is invented here; this is a proposal pending an explicit decision.

**Recommended bump: Minor (0.6.0).** Per `docs/releases/README.md` §2, a new backward-compatible feature is a minor bump, and major stays `0` until Public Stable. This promotion adds at least four independently-shippable, end-to-end-wired features (Assistant capture, Telegram account linking, category suggestions, merchant mapping) plus a substantial Analytics workspace overhaul — comfortably past the minor threshold, with no breaking change to justify more.

**Proposed entry content** (Indonesian, matching existing convention; not yet added to `changelog.ts`):

```
added:
  - "Asisten percakapan untuk mencatat transaksi via chat: klarifikasi terpandu, tinjau dan ubah draft sebelum konfirmasi, serta riwayat percakapan dengan arsip/pulihkan/hapus"
  - "Penautan akun Telegram sebagai kanal pengiriman balasan Asisten, dapat dikelola dari halaman Akun"
  - "Saran kategori otomatis saat menambah transaksi, berdasarkan kata kunci"
  - "Pemetaan merchant ke kategori per pengguna ('ingat merchant ini'), dikelola dari halaman Akun atau saat menambah transaksi"
improved:
  - "Analitik: ringkasan arus kas kini membandingkan dengan periode sebelumnya, dengan rincian kategori, dompet, kinerja anggaran, dan penelusuran transaksi"
  - "Performa pemuatan halaman Dashboard, Tagihan, dan Analitik"
fixed:
  - "Respons API Analitik yang sebelumnya salah dibaca oleh aplikasi"
  - "Asosiasi label dan kontras teks error pada beberapa formulir"
```

This is a draft for review — not a final editorial pass, and not committed to `changelog.ts`. Applying it requires an explicit version decision and a PR against `pocket-mint-fe`'s own release workflow (`docs/releases/README.md` §6), separate from this docs-only PR.

---

## Database migrations (backend)

12 new Prisma migrations in this range, all additive (new tables/columns/indexes for merchant mapping, Assistant conversation/draft/provider-execution/idempotency state, clarification engine, and channel/Telegram processing). No destructive operation (`DROP TABLE`, `DROP COLUMN`, data deletion) was found. One migration (`20260725000000_add_clarification_expiry`) adds a `NOT NULL` constraint to a column on a table introduced earlier in this same promotion range, so there is no pre-existing production data at risk. Standard `prisma migrate deploy` should apply cleanly; running `prisma migrate status`/`diff` against the actual target database before promotion is still recommended as a live-infra check this document cannot perform.

## Repository housekeeping note (backend)

The raw `origin/main..origin/dev` diffstat for `pocket-mint-be` (898 files, ~1.17M insertions) is misleading: ~533 of those files are the repo's committed `dist/` build output (an existing, intentional repo convention) and a doubled, committed Prisma-generated client — together explaining nearly all the volume. Separately, **90 files under `graphify-out/` (a knowledge-graph tool cache) are tracked on `dev` but not on `main`**, and appear to be an accidental commit rather than product code. The real source diff for this promotion is **248 files changed, +30,706/-59**. This is flagged for awareness in the promotion PR; no change has been made to `dev` to address it, per explicit decision to flag only.

---

## 8. Appendix — Phase 27–35 engineering detail (traceable context)

The sections below are the original Phase 27–35 baseline, retained verbatim for traceability. They are a subset of §1 above, not additional scope.

### 8.1 Reliable transaction capture

**Added**

- Optional request-level idempotency for the Assistant message and execute endpoints (`Idempotency-Key` header, same format as draft confirm): a retried or double-submitted request replays the original outcome instead of creating a second turn or draft. (Phase 27, PD-017)
- Recovery from interrupted or still-running Assistant work: the conversation recovery-state view now reports the conversation's in-progress (`RUNNING`) turn, if any, so a client can detect and wait on in-flight work instead of resubmitting blind. (Phase 27, PD-017)
- Common casual `k` Rupiah notation (`25k`, `450k`, `1.5k`) is now recognized by the same deterministic amount-recovery fallback that already handled `rb`/`ribu`/`jt`/`juta`. (Phase 35, PD-025)
- A fixed, reporting-timezone-anchored current-date reference inside the Assistant's own interpretation step, so relative Indonesian date phrases ("kemarin", "tadi", "hari ini") resolve against a concrete calendar date rather than an unanchored one. (Phase 35, PD-025)

**Changed**

- Draft preview data (initial draft, clarification-continuation, and recovery-state projections) now additionally includes the raw `categoryId` and an unambiguous `transactionDate`, alongside the existing display strings — so the Web edit UI can correctly preselect category and date when a user opens a draft to correct it before confirming. (Phase 34, PD-024)

**Fixed**

- Draft correction (confirm-time `amount`/`walletId`/`categoryId`/`description`/`date` overrides) is now covered by integration tests for per-field overrides, combinations, invalid wallet/category rejection, concurrent confirms, and overrides against expired/cancelled/already-committed drafts — closing a gap where this capability existed but had no documented contract or automated coverage. (Phase 34, PD-024)

**Unchanged by design**

- Draft confirmation remains mandatory; no transaction is created until the user confirms. Overrides merge only after idempotency-key handling, not around it. (Phase 27, Phase 34)

### 8.2 Conversation history and lifecycle

**Verified and documented** (no behavior change — Phase 33 inspected, tested, and wrote down semantics for already-shipped archive/restore/delete)

- Archive, restore, and delete of an Assistant conversation, including their interaction with a pending financial draft, a pending clarification, a channel-linked conversation, and the two channel operational tables, is now covered by integration tests and documented in `pocket-mint-be`'s API docs. (Phase 33, PD-023)
- Ownership is privacy-safe by construction: archiving, restoring, or deleting a conversation that does not exist, or belongs to another user, is indistinguishable from "not found." (Phase 33, PD-023)
- Deleting a conversation never deletes the underlying confirmed `Transaction` record it produced — only the conversation/draft/idempotency rows are cascaded. (Phase 33, PD-023)

**Known limitations surfaced by this review**

- `AssistantConversationStatus.EXPIRED` is a reserved but currently unreachable status value.
- Two channel-operational columns (`ChannelInboundJob.assistantTurnId`, `ChannelAssistantOperation.turnId`) have no foreign-key relation and are left stale (not nulled) if their conversation is deleted. This is a documented data-quality gap, not a correctness or access-control issue.
- A delete racing an in-flight `RUNNING` turn is not guarded against — an accepted gap, consistent with the no-lock-across-in-flight-work precedent set in Phase 27.

### 8.3 Web and Telegram continuity

**Added**

- Source attribution: a conversation shared across Web and Telegram now reports which channel each turn originated from (`channel` per turn, `sourceChannels` on the conversation summary), surfaced in the Web UI as a subtle badge. (Phase 30, PD-020)
- Delivery state for Telegram-originated Assistant replies: `NOT_APPLICABLE` (Web), `PENDING`, `PROCESSING`, `DELIVERED`, `FAILED`, surfaced in the Web UI next to the Assistant's reply. (Phase 31, PD-021)
- An explicit `UNKNOWN` delivery-state value for Telegram turns old enough that their delivery record has aged out of retention — replacing a previously *omitted* field with an explicit one, so old conversation history has well-defined API semantics instead of looking broken. (Phase 32, PD-022)

**Explicitly guaranteed (not a limitation — a stated boundary)**

- No Telegram `externalChatId`, `externalUserId`, `externalSenderId`, provider message ID, destination chat ID, reply markup, callback token, or raw provider payload is exposed through any DTO, in Phase 30, 31, or 32. (PD-020, PD-021, PD-022)

**Known limitations**

- Attribution cannot be recovered for turns whose originating channel-inbound record has already been retention-purged, or for pre-Phase-30 callback-originated turns — a bounded, one-time historical-data gap, not an ongoing one. (Phase 30, PD-020)
- `deliveryStatus` is a pure read-time computation, never persisted, and is not available on conversation list summaries (a per-turn concept only).

### 8.4 Operational safety

**Added**

- Operator visibility: a read-only report (script-based, run on demand or in CI) counting and sampling ambiguous Assistant/callback executions, terminal inbound/outbound channel failures, and stale `RUNNING` Assistant turns — the operational states Phases 26B/27 deliberately fail closed into rather than ever risk a duplicate financial mutation. (Phase 28, PD-018)
- Bounded operator remediation for those same states: three explicit, single-row, dry-run-by-default subcommands (`mark-reviewed`, `requeue-outbound`, `reconcile-turn`), each gated by a safety predicate and a conditional update that cannot touch a draft, transaction, or idempotency record. (Phase 29, PD-019)
- Privacy-safe, bounded capture-lifecycle events — `assistant.capture.draft_created` (direct vs. clarification-resolved), `clarification_required`, `clarification_cancelled`, `draft_confirmed` (whether an override was used), `draft_cancelled`, `draft_expired` — extending the existing closed, redacted ops-event taxonomy. No raw message content, amounts, IDs, or provider payloads are logged. (Phase 35, PD-025)

**Explicitly guaranteed (not a limitation — a stated boundary)**

- No automatic replay or re-execution of any ambiguous Assistant or Telegram-callback execution exists anywhere in Phases 27–35. Every remediation action requires an explicit operator-supplied row identifier; there is no "fix all" mode and no background scheduler performing remediation on its own. (Phase 28, Phase 29)
- None of Phases 28–32 change any financial-mutation, idempotency-claim, or confirmation semantics.

### 8.5 Known limitations / next work (Phase 27–35 scope)

- **Natural-language transfer capture is not yet supported.** The Assistant can capture income/expense transactions; transfers between wallets are out of scope for this baseline. (Phase 35, PD-025, "Scope — Explicitly not done")
- **No financial-advice or recommendation behavior.** The Phase 35 date-anchor addition is explicitly scoped to relative-date grounding only — never budgeting, advice, or other reasoning — and nothing in Phases 27–35 adds any advisory capability.
- **Confirmation remains mandatory before a transaction is created.** No phase in this baseline introduces a path that commits a transaction without an explicit user (or Telegram-interactive) confirmation step.
- **No automatic retry of ambiguous financial actions.** An ambiguous or stuck execution is surfaced to an operator (Phase 28) and may be remediated only by an explicit, single-row operator action (Phase 29) — never retried automatically.
