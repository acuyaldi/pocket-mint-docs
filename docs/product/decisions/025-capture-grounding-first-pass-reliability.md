# PD-025 — Capture Grounding & First-Pass Reliability (Phase 35)

**Status:** Accepted
**Date:** 2026-10-15
**Author:** Agent (Phase 35 implementation)

## Context

A read-only inspection at the start of this phase found three narrow, unrelated reliability gaps in Assistant Core's natural-language transaction capture path — not a redesign of capture, confirmation, or clarification:

1. **No reporting-timezone date anchor.** The deterministic system instruction sent to Gemini (`buildAssistantSystemInstruction`) carried capability rules and a reply-language directive, but no current-date grounding. Relative Indonesian phrases — "kemarin" (yesterday), "tadi" (earlier today), "hari ini" (today) — had no fixed calendar reference to resolve against inside the model's own interpretation. The backend's own omitted-date default (today, reporting timezone) already existed and was already correct; the gap was upstream, in what the model could infer before that default ever applies.
2. **`k` Rupiah notation was explicitly rejected.** `rupiah-amount-recovery.ts`'s deterministic single-amount-token fallback already supported `rb`/`ribu` (thousands) and `jt`/`juta` (millions) suffixes, with an existing test (`'bayar internet 350k'`) pinning that `k` was declined. `k` (`25k`, `450k`) is common, especially in quick casual typing, and is the same thousands multiplier as `rb`/`ribu` — the grammar already had the right shape, it just didn't recognize the letter.
3. **No capture-quality lifecycle signal.** The existing `assistant.message.*`/`assistant.draft.*`/`assistant.clarification.*` events (PD-018) are operational — request/response timing and outcome for the HTTP boundary. None of them distinguish *why* a turn did or didn't produce a draft on the first pass: whether a draft came directly from one message or only after clarification, whether a clarification was needed at all (and why), whether the user abandoned a clarification, or whether a confirm carried an edit. That is a product-quality question PD-018's events were never designed to answer.

None of these three gaps involve the confirmation boundary, idempotency, draft-correction semantics, or transaction lifecycle — draft creation, clarification, and confirmation already work correctly; this phase only improves how often a message reaches a correct first-pass draft and makes that improvement (and its absence) measurable.

## Decision

**Selected: three narrow, independent, additive fixes — a date-anchor line in the existing system instruction, one more accepted suffix in the existing deterministic amount-recovery grammar, and a new `assistant.capture.*` event family alongside (not replacing) the existing ops taxonomy.**

### 1. Reporting-timezone date anchor

`buildAssistantSystemInstruction(catalog, locale, now = new Date())` appends one additional rule line: `Today's reporting date is YYYY-MM-DD (<reporting timezone>). Use it only to resolve relative date phrases in the current request (e.g. "kemarin", "tadi", "hari ini") into an explicit YYYY-MM-DD date; never use it for financial advice, budgeting, or any reasoning beyond relative-date grounding.` The date is computed with the existing `formatReportingDate`/`reportingConfig.timezone` helpers — the same reporting-calendar utilities every other date-default in the codebase already uses — never a second timezone implementation. The `now` parameter defaults to the real clock and is only ever overridden in tests, so production behavior is simply "the real current reporting date, computed fresh per request" (the instruction is rebuilt per call; nothing is cached across requests). This is grounding only: it does not add a new clarification type, does not change what happens when the model still omits a date (the backend's existing default — today, reporting timezone — is untouched), and the instruction explicitly forbids the model from using the date for anything beyond resolving relative phrases.

### 2. `k` Rupiah suffix notation

`rupiah-amount-recovery.ts`'s suffix grammar (`AMOUNT_TOKEN_RE`, `SUFFIX_RE`) now accepts `k`/`K` alongside the existing `rb`/`ribu`/`jt`/`juta`, reusing the existing thousands multiplier branch (`k` behaves exactly like `rb`/`ribu` — ×1,000) — no new multiplier logic. `25k`, `450k`, `1.5k`, and `1,5k` are all accepted; `350km`, `0k`, `-350k`, and any message containing more than one plausible amount-shaped token are all still declined, unchanged. This is the same narrow, single-token, "exactly one unambiguous amount or nothing" fallback that already existed for `rb`/`ribu`/`jt`/`juta` — not a new fuzzy or free-form numeric parser, and not a new amount clarification type. It only ever fires when the provider's own structured plan omitted `amount` in the first place; a provider-supplied amount is never second-guessed or overridden by it.

### 3. `assistant.capture.*` capture-lifecycle events

Six new bounded, privacy-safe events, fired at the same HTTP-controller boundary as the existing `assistant.message.*`/`assistant.draft.*`/`assistant.clarification.*` events (PD-018) and using the same closed `AssistantLogEvent` field set (extended, not widened to free-form):

- `assistant.capture.draft_created` — `captureSource: 'direct' | 'clarification_resolved'`.
- `assistant.capture.clarification_required` — `clarificationKind` (`'wallet' | 'merchant' | 'category' | 'transaction_fields' | 'provider_text'`).
- `assistant.capture.clarification_cancelled` — `clarificationKind`, same enum.
- `assistant.capture.draft_confirmed` — `overridesUsed: boolean` (whether confirm carried one or more override fields; never which fields or their values). Only fires on an actual new confirm, not an idempotent replay.
- `assistant.capture.draft_cancelled` — fires only for a genuine fresh cancellation, not a repeated/idempotent cancel-of-already-cancelled.
- `assistant.capture.draft_expired` — fires only when the existing lazy expiry check inside `confirm()`/`cancel()` actually detects and terminalizes an expired draft; no new expiry worker or polling was added.

These are deliberately a separate, additive family from PD-018's ops events, not a replacement: PD-018 answers "did the request succeed and how long did it take"; `assistant.capture.*` answers "did this message produce a usable draft on the first pass, and if not, why." The same closed, allowlisted `AssistantLogEvent` type (PD-018) gates every field — raw message text, amounts, wallet/category/merchant identifiers, descriptions, and clarification option labels/tokens are structurally impossible to pass through it, the same guarantee PD-018 already established.

## Rejected alternatives

- **Pass the date as a separate bounded request field instead of a system-instruction line.** Rejected: the existing prompt already separates a deterministic system instruction from one labelled untrusted JSON data message (§15, Assistant Core Architecture); a date anchor is backend-authoritative instruction content, not user-supplied data, so it belongs in the instruction the same way the reply-language directive already does. No new request-schema surface was needed.
- **Add a model-facing "amount clarification" when the message contains an ambiguous `k`-like token.** Rejected: the existing deterministic recovery already declines (returns no recovery) on any ambiguity — multiple amount-shaped tokens, decimals it can't safely multiply, suffixes it doesn't recognize — and the existing missing-amount `provider_text` clarification path already covers that case. A second, amount-specific clarification type would duplicate an existing path for no new capability.
- **Broaden amount recovery into a general numeric/currency parser (e.g. spelled-out numbers, `setengah juta`, arbitrary locale formats).** Rejected: explicitly out of scope — the existing design is a narrow, single-token, safe-or-nothing fallback for one measured gap (`k` notation), not a general NLU numeric parser. Existing declined cases (`setengah juta`, multi-amount messages) remain declined.
- **Replace the existing `assistant.draft.*`/`assistant.clarification.*` ops events with the new capture events.** Rejected: the two families answer different questions (operational health vs. capture-quality) for different audiences, and replacing one with the other would both lose the existing ops signal and conflate two concerns PD-018 deliberately kept separate.
- **Log the clarification option labels or the overridden field values on `draft_confirmed`/`clarification_cancelled`.** Rejected: violates the existing redaction boundary (PD-018, `.claude/skills/assistant-core.skill.md` §13) for no proven operational need — a bounded kind/boolean signal already answers the product question ("was there friction, and what kind") without capturing content.

## Consequences

**Positive:**

- The model has an explicit, deterministic, reporting-timezone-correct anchor for relative-date phrases in the single highest-volume natural language pattern ("kemarin", "tadi", "hari ini"), reducing first-pass date misinterpretation without adding a clarification step or changing the backend's own safe default.
- A common, previously-rejected casual amount notation (`k`) now recovers deterministically under the same safety constraints as the suffixes that already worked, reducing first-pass clarification-required messages for one measured gap.
- Product and engineering can now measure, from logs alone and without any raw content exposure, how often capture succeeds directly vs. needs clarification, how often clarification is abandoned, and how often confirmation involves a correction — closing a real measurement gap PD-018's ops events never covered.
- Zero schema or migration changes; zero new endpoints; zero changes to confirmation, idempotency, draft-correction, or clarification semantics.

**Costs:**

- Six new event names and three new `AssistantLogEvent` fields (`captureSource`, `clarificationKind`, `overridesUsed`) to maintain alongside the existing taxonomy — a small, bounded addition to an already-closed type, not an open-ended one.
- The system instruction is one line longer per request; negligible token-cost impact given the existing instruction's size.

## Related documents

- [Assistant Core Architecture §15.13](../../architecture/assistant-core-architecture.md#1513-capture-grounding--first-pass-reliability-phase-35)
- [Implementation Roadmap — Phase 35](../../development/implementation-roadmap.md)
- `pocket-mint-be`'s `docs/api/assistant-conversations.md` — date-anchor and amount-recovery notes added by this phase
- `pocket-mint-be`'s `docs/observability-runbook.md` and `.claude/skills/assistant-core.skill.md` §13 — `assistant.capture.*` taxonomy added by this phase
- [PD-018 — Assistant Operations Visibility](018-assistant-operations-visibility.md) — the existing ops event taxonomy and redaction boundary this phase's events extend, not replace
- [PD-024 — Draft Correction Completion](024-draft-correction-completion.md) — most recent prior Assistant-lifecycle phase; confirm-time overrides are confirmed unchanged by this phase's `draft_confirmed` event, which only observes whether an override was present

## Non-goals / explicitly out of scope

- No transfer support, no fuzzy or substring wallet/merchant matching, no merchant auto-learning from confirmed transactions.
- No new clarification type for amount or date — existing `provider_text`/`entity_selection` clarification paths are unchanged.
- No clarification UI redesign or copy rewrite.
- No change to confirmation, idempotency, draft-correction, or transaction lifecycle semantics — draft confirmation remains mandatory, and no transaction is created until confirm.
- No frontend changes — this phase is backend-only; no documented event/API contract required one.
- No new provider, no queue redesign, no admin UI, no analytics dashboard.
- No raw capture data (message text, amounts, wallet/category/merchant identifiers, descriptions, provider payloads, Telegram IDs, tokens, transaction identifiers) in any new log line.
- No financial-advice, insight, recommendation, budgeting, proactive-assistant, or channel-expansion behavior — this phase is capture reliability and measurement only.

## Follow-up Tasks

- If a future phase wants capture-quality metrics broken down by locale or channel, that is a new, separate decision — this phase's events are deliberately channel/locale-agnostic, consistent with PD-018's existing bounded-field philosophy.
- If `k`-notation recovery is found to still miss a measurable share of messages (e.g. a decimal grammar broader than one–two digits), widening the existing grammar is a narrow follow-up, not a reason to introduce a general parser.
