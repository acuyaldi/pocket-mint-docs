# Assistant Reliability & Lifecycle — Release Baseline (Phases 27–35)

## Purpose

This document consolidates Phases 27–35 of the Pocket Mint Assistant work — currently merged into `pocket-mint-be`/`pocket-mint-fe` `dev` and `pocket-mint-docs` `main` — into a single release-facing changelog, ahead of Phase 36. It groups changes by user/product outcome rather than by commit, so a reviewer can evaluate what actually shipped without re-reading nine phases of implementation notes.

**Scope.** Pocket Mint Assistant is a natural-language transaction *capture* interface: it helps a user create a transaction draft through chat (Web or Telegram) and confirm it. It is not a financial-advice or recommendation product, and this baseline does not change that.

**Evidence basis.** Every item below is sourced from the per-phase entries in [Implementation Roadmap — Phases 27–35](implementation-roadmap.md#engineering-phases), their linked Product Decisions ([PD-017](../product/decisions/017-assistant-request-level-idempotency.md)–[PD-025](../product/decisions/025-capture-grounding-first-pass-reliability.md)), [Assistant Core Architecture §§15.7–15.13](../architecture/assistant-core-architecture.md), and the corresponding merged PRs in `pocket-mint-be` (#97, #99–#107) and `pocket-mint-fe` (#84, #86–#89), all merged with green CI. No item below claims behavior beyond what those sources document.

**Not a public announcement.** `pocket-mint-fe`'s own public changelog (`src/lib/changelog.ts`, rendered at `/changelog`) has no Assistant entry as of its latest published release. Per that changelog's own convention (`docs/releases/README.md` §1a in `pocket-mint-fe`), completing internal release documentation does not by itself create a public changelog entry — that remains a separate, explicit product decision. This document is the internal/engineering release baseline; it does not alter or substitute for that decision.

---

## 1. Reliable transaction capture

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

---

## 2. Conversation history and lifecycle

**Verified and documented** (no behavior change — Phase 33 inspected, tested, and wrote down semantics for already-shipped archive/restore/delete)

- Archive, restore, and delete of an Assistant conversation, including their interaction with a pending financial draft, a pending clarification, a channel-linked conversation, and the two channel operational tables, is now covered by integration tests and documented in `pocket-mint-be`'s API docs. (Phase 33, PD-023)
- Ownership is privacy-safe by construction: archiving, restoring, or deleting a conversation that does not exist, or belongs to another user, is indistinguishable from "not found." (Phase 33, PD-023)
- Deleting a conversation never deletes the underlying confirmed `Transaction` record it produced — only the conversation/draft/idempotency rows are cascaded. (Phase 33, PD-023)

**Known limitations surfaced by this review**

- `AssistantConversationStatus.EXPIRED` is a reserved but currently unreachable status value.
- Two channel-operational columns (`ChannelInboundJob.assistantTurnId`, `ChannelAssistantOperation.turnId`) have no foreign-key relation and are left stale (not nulled) if their conversation is deleted. This is a documented data-quality gap, not a correctness or access-control issue.
- A delete racing an in-flight `RUNNING` turn is not guarded against — an accepted gap, consistent with the no-lock-across-in-flight-work precedent set in Phase 27.

---

## 3. Web and Telegram continuity

**Added**

- Source attribution: a conversation shared across Web and Telegram now reports which channel each turn originated from (`channel` per turn, `sourceChannels` on the conversation summary), surfaced in the Web UI as a subtle badge. (Phase 30, PD-020)
- Delivery state for Telegram-originated Assistant replies: `NOT_APPLICABLE` (Web), `PENDING`, `PROCESSING`, `DELIVERED`, `FAILED`, surfaced in the Web UI next to the Assistant's reply. (Phase 31, PD-021)
- An explicit `UNKNOWN` delivery-state value for Telegram turns old enough that their delivery record has aged out of retention — replacing a previously *omitted* field with an explicit one, so old conversation history has well-defined API semantics instead of looking broken. (Phase 32, PD-022)

**Explicitly guaranteed (not a limitation — a stated boundary)**

- No Telegram `externalChatId`, `externalUserId`, `externalSenderId`, provider message ID, destination chat ID, reply markup, callback token, or raw provider payload is exposed through any DTO, in Phase 30, 31, or 32. (PD-020, PD-021, PD-022)

**Known limitations**

- Attribution cannot be recovered for turns whose originating channel-inbound record has already been retention-purged, or for pre-Phase-30 callback-originated turns — a bounded, one-time historical-data gap, not an ongoing one. (Phase 30, PD-020)
- `deliveryStatus` is a pure read-time computation, never persisted, and is not available on conversation list summaries (a per-turn concept only).

---

## 4. Operational safety

**Added**

- Operator visibility: a read-only report (script-based, run on demand or in CI) counting and sampling ambiguous Assistant/callback executions, terminal inbound/outbound channel failures, and stale `RUNNING` Assistant turns — the operational states Phases 26B/27 deliberately fail closed into rather than ever risk a duplicate financial mutation. (Phase 28, PD-018)
- Bounded operator remediation for those same states: three explicit, single-row, dry-run-by-default subcommands (`mark-reviewed`, `requeue-outbound`, `reconcile-turn`), each gated by a safety predicate and a conditional update that cannot touch a draft, transaction, or idempotency record. (Phase 29, PD-019)
- Privacy-safe, bounded capture-lifecycle events — `assistant.capture.draft_created` (direct vs. clarification-resolved), `clarification_required`, `clarification_cancelled`, `draft_confirmed` (whether an override was used), `draft_cancelled`, `draft_expired` — extending the existing closed, redacted ops-event taxonomy. No raw message content, amounts, IDs, or provider payloads are logged. (Phase 35, PD-025)

**Explicitly guaranteed (not a limitation — a stated boundary)**

- No automatic replay or re-execution of any ambiguous Assistant or Telegram-callback execution exists anywhere in Phases 27–35. Every remediation action requires an explicit operator-supplied row identifier; there is no "fix all" mode and no background scheduler performing remediation on its own. (Phase 28, Phase 29)
- None of Phases 28–32 change any financial-mutation, idempotency-claim, or confirmation semantics.

---

## 5. Known limitations / next work

These are current, intentional boundaries of the Assistant as of Phase 35 — not defects:

- **Natural-language transfer capture is not yet supported.** The Assistant can capture income/expense transactions; transfers between wallets are out of scope for this baseline. (Phase 35, PD-025, "Scope — Explicitly not done")
- **No financial-advice or recommendation behavior.** The Phase 35 date-anchor addition is explicitly scoped to relative-date grounding only — never budgeting, advice, or other reasoning — and nothing in Phases 27–35 adds any advisory capability.
- **Confirmation remains mandatory before a transaction is created.** No phase in this baseline introduces a path that commits a transaction without an explicit user (or Telegram-interactive) confirmation step.
- **No automatic retry of ambiguous financial actions.** An ambiguous or stuck execution is surfaced to an operator (Phase 28) and may be remediated only by an explicit, single-row operator action (Phase 29) — never retried automatically.

---

## Not included in this baseline

`dev` on both `pocket-mint-be` and `pocket-mint-fe` currently contains additional merged work that predates or is unrelated to Phases 27–35 and is **not** represented above. It should not be read into this changelog, and needs its own release handling before or alongside promotion:

- **Analytics v2** (`feature/analytics-v2`) — backend and frontend.
- **Smart Categorization Foundation** and **Merchant Mapping ("Phase 19")** (`feature/smart-categorization-foundation`, `feature/merchant-mapping`) — backend and frontend.
- `pocket-mint-fe`: Storybook foundation and its follow-up fixes, the frontend test/a11y baseline fix, two general navigation/analytics performance fixes (CLS, LCP) unrelated to the Assistant, and routine lockfile-sync commits.
- `pocket-mint-be`: `perf(categories): skip default-seed upsert when categories already exist`, and two analytics test-determinism fixes (`test-analytics-relative-dates`) — none of these change Assistant behavior.

See the promotion-readiness report (delivered alongside this document) for the exact commit lists.
