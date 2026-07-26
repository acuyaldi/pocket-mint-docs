# PD-013 — Assistant Production Observability Foundation (Phase 24)

**Status:** Accepted
**Date:** 2026-07-26
**Author:** Agent (Phase 24 implementation)

## Context

The Assistant v1 foundation — conversation flow, clarification, financial drafts, idempotent replay, recovery-state reconciliation — is functionally complete and validated (Phases 21–22.5). Before any external channel is added, operating it in production requires answering basic questions that are currently only answerable by reading code or guessing: did a request succeed, which lifecycle stage failed, was a request retried or replayed, was a clarification or draft created, did recovery reconciliation resolve an ambiguous outcome, and is a given failure a provider, validation, policy, database, or transport problem.

This is an observability decision, not a logging-hygiene bug fix: the existing `src/utils/logger.ts` (structured JSON, `redact()` blocklist) and `src/http/correlation.ts` (per-request correlation ID) were already sound and low-risk. The open question was how to extend them into a consistent lifecycle-event/error-taxonomy schema without either (a) inventing a second logging system, or (b) letting a well-intentioned lifecycle event become the next place sensitive content leaks into logs.

## Decision

- **One canonical event schema, added to the existing logger.** `logEvent(level, fields: AssistantLogEvent)` in `src/utils/logger.ts` — `AssistantLogEvent` is a closed TypeScript interface (no index signature) listing only bounded operational fields (`event`, `requestId`, `operation`, `stage`, `outcome`, `durationMs`, `httpStatus`, `errorCategory`, `retryable`, `idempotentReplay`, `idempotencyOutcome`, `provider`, `tool`, booleans like `draftCreated`). A caller cannot pass message text, draft payloads, or tokens through it — the compiler rejects unknown fields. The existing `redact()` key-fragment blocklist remains a runtime backstop under every log call, including plain `logger.*` calls elsewhere.
- **One bounded error taxonomy, derived from error type/code, never message text.** `categorizeError(err)` in `src/utils/errorCategory.ts` maps `AssistantError`/`AssistantProviderError` codes (and generic HTTP-status/Prisma-code errors) to a fixed `ErrorCategory` union (`authentication` … `internal`). `AssistantError.draftConflict(status)` now carries `status` in a structured `detail` field specifically so the mapper can distinguish an `EXPIRED` conflict from any other conflict status without parsing the human-readable message.
- **Events fire at the existing HTTP controller boundary**, plus the provider and tool boundaries where timing was already isolated (`provider-runtime.ts`, `executor.ts`) — not deep inside the clarification/draft transaction-safety code (`clarification.service.ts`'s atomic claim, `financial-draft.service.ts`'s advisory lock and `P2002` handling). Those files carry explicit "do not weaken" invariants (see `assistant-core.skill.md` §§6–9); instrumenting one layer up, from data the controller already has (response status, `durationMs`, a small literal `idempotencyOutcome` field added to the two successful draft-confirm return branches), gets the same operational visibility without touching that logic.
- **Idempotency outcome is a first-class field**, not inferred from response shape: `'new' | 'replay' | 'conflict' | 'expired'`, computed from real backend branches (`financial-draft.service.ts confirm()`), stripped from the client-facing response before it's sent, and never accompanied by the idempotency key itself.
- **Metrics are log-based, not a new library or endpoint.** No metrics backend (Prometheus, Datadog, etc.) was already connected in `pocket-mint-be`, and Railway has no scrape target configured. Counters/histograms are derived by aggregating the structured `event`/`durationMs`/`errorCategory` fields from the existing log stream. No `prom-client`, no `/metrics` endpoint, no vendor was added.
- **No new identifiers.** Only the existing per-request correlation ID (`requestId`) is logged. No conversation ID, turn ID, or user ID was added to any event — the spec's own "no safe pseudonymous-identifier invention" constraint meant the honest choice was to scope this phase to request-level correlation only, not to design a new ID scheme as a side effect of a logging change.
- **Frontend unchanged.** `pocket-mint-fe` has no existing request-ID plumbing, error boundary, or error-reporting library to extend (verified before deciding this); building all three from scratch to display a support-reference ID was judged out of scope for a backend observability phase and deferred until a real support/debugging gap is observed in practice.

## Alternatives considered

- **A dedicated metrics library (`prom-client`) with a `/metrics` endpoint.** Rejected: no scrape target exists in the current Railway deployment, and adding an unused endpoint (or one requiring new auth/network-boundary review) is speculative infrastructure for a foundation phase. Revisit if/when a metrics backend is actually connected.
- **OpenTelemetry / a tracing vendor.** Rejected: `pocket-mint-be` is a single backend service with no downstream services to trace across; a full tracing SDK would add dependency and complexity with no distributed system to correlate.
- **Instrumenting inside `clarification.service.ts` and `financial-draft.service.ts` directly** (i.e., logging from within the atomic-claim/advisory-lock transaction bodies). Rejected in favor of controller-boundary instrumentation: those files' concurrency-safety mechanisms (conditional `updateMany` claims, `pg_advisory_xact_lock`, `P2002` re-check) are explicitly protected invariants, and the same operational signal (outcome, timing, idempotency result) is fully derivable one layer up without touching them.
- **A conversation/turn-scoped pseudonymous identifier for cross-request correlation.** Rejected for this phase: no existing safe-hashing or pseudonymization policy exists in the repository, and the phase's own constraints explicitly prohibit inventing one ad hoc. Request-level correlation via the existing `X-Correlation-Id` is sufficient for the operational questions this phase answers.
- **A new `.claude/skills/assistant-observability.skill.md` file.** Rejected: the addition (event names, forbidden fields, taxonomy location) is small and belongs with the domain it instruments; it was added as a new "§13 Observability" section in the existing `assistant-core.skill.md` instead, avoiding a duplicate-rules file.
- **A dedicated event per every lifecycle transition listed illustratively in the originating spec** (e.g. separate `assistant.tool.selected` and `assistant.clarification.created`/`.expired` events). Rejected where a transition was already visible via an outcome/boolean field on an adjacent event (e.g. `chainedClarification`, `draftCreated`) — adding a second event for the same transition was judged to add log volume without new operational value.

## Consequences

**Positive:**

- Every Assistant HTTP request is traceable end-to-end by `requestId` across received/completed/failed lifecycle events, with millisecond durations at the message, provider, and tool boundaries.
- Duplicate financial mutations are now directly observable (`idempotencyOutcome: 'replay'`) without exposing the idempotency key, closing a real operational blind spot ("did my confirm actually double-charge?").
- Failure investigation starts from a bounded `errorCategory` instead of grepping raw exception text, and is safe to page off of or dashboard from without a data-classification review.
- No new dependency, vendor, or attack surface was introduced; the change is additive logging/classification only.
- Zero changes to Assistant lifecycle behavior, authorization, or financial mutation paths — the full existing test suite passes unmodified, plus new/extended tests assert no sensitive content enters any new or modified log call.

**Costs:**

- Metrics remain log-derived rather than queryable in a dashboard until an actual metrics backend is connected — extracting a counter or histogram today means aggregating log lines, not querying a time-series store.
- The event taxonomy is deliberately narrower than the illustrative list in the originating request (no `assistant.clarification.created`/`.expired`, no `assistant.tool.selected`, no conversation/turn identifiers) — extending it is straightforward but not yet done.
- `AssistantError.draftConflict`'s new `detail` field and the two draft-confirm return branches' new `idempotencyOutcome` field are small, permanent additions to internal shapes that future changes to those paths must keep in sync with `categorizeError`/the controller's stripping logic.

## Related documents

- [Assistant Core Architecture §17 — Auditability and Correlation](../../architecture/assistant-core-architecture.md) and §23 (Security Considerations), extended by this phase.
- [System Architecture — Logging / Privacy / Observability and Reliability](../../architecture/system-architecture.md), extended with concrete references to this ADR and the runbook.
- [Implementation Roadmap — Phase 24](../../development/implementation-roadmap.md).
- Operational runbook: `pocket-mint-be/docs/observability-runbook.md`.
- Implementation skill reference: `pocket-mint-be/.claude/skills/assistant-core.skill.md` §13 (Observability).
- Backend PR: `feature/assistant-observability-foundation` against `dev` (`pocket-mint-be`).

## Explicitly out of scope (deferred to later phases or not planned)

- A metrics backend, `/metrics` endpoint, OpenTelemetry, or any observability vendor.
- Frontend request-ID surfacing, error boundary, or error-reporting integration.
- Conversation ID, turn ID, or user ID in any log record.
- A dedicated event for every lifecycle transition illustratively listed in the originating spec.
- Any change to Prisma schema, health/readiness behavior, or Assistant lifecycle/authorization logic.
- Automatic financial retries, streaming/SSE/WebSockets, or external-channel (Telegram/Discord/WhatsApp/n8n) monitoring.
