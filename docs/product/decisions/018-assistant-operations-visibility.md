# PD-018 — Assistant Operations Visibility (Phase 28)

**Status:** Accepted
**Date:** 2026-09-17
**Author:** Agent (Phase 28 implementation)

## Context

Phase 26B ([PD-015](015-durable-channel-processing.md)) accepted one deliberate, narrow failure mode as the price of never risking a duplicate financial mutation: a crash strictly between "operation row inserted" and "turn recorded" leaves a `ChannelInboundJob` `FAILED_TERMINAL` with `errorCategory: ambiguous_assistant_execution` (and its Phase 26A sibling, `ambiguous_callback_execution`) — a state the system deliberately refuses to resolve on its own. Phase 27 ([PD-017](017-assistant-request-level-idempotency.md)) accepted the same shape of gap for Web: a process crash between claiming an `Idempotency-Key` and resolving it leaves that row, and the `AssistantTurn` it belongs to, `RUNNING` forever.

Both decisions were correct to defer automatic resolution — guessing wrong risks a duplicate financial mutation, which this system has never accepted as a tradeoff. But "defer automatic resolution" was never meant to mean "defer *discoverability*." Today, an operator's only way to learn that any of these states exist is to already know to run one of the ad hoc SQL queries in the [Telegram Deployment Runbook §8](../development/telegram-deployment-runbook.md#8-operator-procedures-phase-26b), or to be reading production logs at the moment the `channel.inbound.failed` event fires. There is no aggregate view, no standing script, no alert, and nothing that answers "is this happening right now, and how much" without a database client already open. An audit of the post-Phase-27 system ranked this the highest-leverage remaining gap: the state that most needs manual inspection is exactly the state operators currently have no dedicated way to inspect.

## Decision

**Selected: a read-only diagnostic script, following the exact pattern `src/scripts/reconcile.ts` already established, not a new HTTP endpoint.** Discovery for this phase confirmed there is no admin/operator concept anywhere in `pocket-mint-be` or `pocket-mint-fe` — no admin role, no privileged-access middleware, no admin UI route, nothing beyond ordinary per-user JWT auth. Building one from scratch to gate a single read-only diagnostic endpoint would be new authorization infrastructure introduced solely to serve this phase, which is exactly the kind of unrequested surface the project's existing operator tooling (`scripts/db-verify.mjs`, `scripts/db-backup.mjs`, `src/scripts/reconcile.ts`) already avoids by being a script an operator with database/deploy access runs directly, at the same trust level as those existing tools.

- **New pure module, `src/domain/opsVisibility.ts`.** Mirrors `src/domain/reconciliation.ts`: pure, DB-free functions that take already-fetched rows and produce (a) a safe, explicitly-allowlisted projection of each row (`toSafeInboundJobRow`, `toSafeOutboundDeliveryRow`, `toSafeAssistantTurnRow`) and (b) small aggregation helpers (`summarizeByErrorCategory`, `isAmbiguousExecution`). The projection functions are the redaction control: each one reads only a fixed, named set of fields off its input and cannot forward an unlisted field, so passing a full Prisma row (which may carry `text`, `externalSenderId`, `renderedText`, `replyMarkup`, etc.) through the projector is what makes the safe shape testable independent of the Prisma `select` clause also being correct.
- **New script, `src/scripts/opsVisibility.ts`.** Read-only (`findMany`/`count` only, exactly like `reconcile.ts`'s own guarantee), reporting four sections: ambiguous assistant/callback executions, terminal inbound job failures, terminal outbound delivery failures, and stale `RUNNING` Assistant turns. Each section reports a total count plus up to 20 most-recent rows (the same `LIMIT 20` the runbook's existing SQL already uses) through the safe projector. Supports `--json` for machine-readable output, matching `reconcile.ts`'s existing flag convention. Exit code `0` clean, `2` when anything actionable was found, `1` on usage/DB error — identical convention to `reconcile.ts`, so it composes the same way with CI or a manual check.
- **Safe field set matches the runbook's own existing precedent**, not a new judgment call: `id`, `provider`, `status`, `attempt`, `errorCategory`, `createdAt`/`completedAt`/`startedAt`, plus the specific correlation identifiers each table already carries (`ChannelInboundJob.assistantTurnId`, `ChannelOutboundDelivery.inboundJobId`, `AssistantTurn.correlationId`/`conversationId`/`intent`). Never selected or projected: `text` (inbound message body), `renderedText`/`replyMarkup` (outbound content), `externalSenderId`/`externalChatId`/`callbackQueryId`/`callbackMessageId` (provider-supplied identifiers), or anything from `AssistantFinancialDraft`/transaction amounts — none of which the operator triage steps in the runbook ever needed.
- **Stale `RUNNING` turn threshold is a fixed constant** (`STALE_RUNNING_TURN_MS`, 5 minutes), not a new environment variable — marked `ponytail:` in the code as a naive fixed heuristic with an env-var upgrade path if 5 minutes proves too noisy or too lax in practice. Queried via the existing `AssistantTurn @@index([status, startedAt])` — no new index.
- **No new automation loop.** `pocket-mint-be` already runs two in-process poll loops (`src/channels/workers/inbound.worker.ts`, `outbound.worker.ts`), but their poll interval (`CHANNEL_INBOUND_POLL_MS`/`CHANNEL_OUTBOUND_POLL_MS`, default 2s) is tuned for message latency, not a periodic health check — piggybacking a table-wide diagnostic scan onto every 2-second tick would be wasteful, and a *separate*, slower ticker is a new scheduling primitive this phase's own non-goals rule out (see below). Since no metrics/alerting backend is connected (PD-013 already established this — "no scrape target exists in the current Railway deployment"), a log-only periodic check would have no consumer to page off of. The script plus the runbook's operator-run cadence is the honest scope for this phase.

## Rejected alternatives

- **A new admin HTTP endpoint (`GET /admin/ops-diagnostics` or similar).** Rejected: requires inventing admin authorization from nothing, for a repository that has never had that concept, solely to gate one read-only report — see Decision above. Revisit only if/when a real admin surface exists for another reason and this becomes one more read-only view on it.
- **A frontend admin page.** Rejected outright: no admin surface exists in `pocket-mint-fe` (verified before deciding this — no `admin` route, no privileged-access pattern), and building one is exactly the UI-not-yet-justified case this phase's own scope excludes.
- **A new periodic background ticker dedicated to this check.** Rejected: no existing scheduler/cron primitive exists to hang it on beyond the two channel-worker loops, whose poll cadence is wrong for this purpose (see Decision); introducing a third loop purely to log a count on a timer is new operational surface (`deployment-operations.skill.md`'s "no queues/infrastructure without evidence" posture, applied here to scheduling infrastructure) for a signal nothing currently consumes.
- **Automatic replay or remediation of ambiguous/terminal rows.** Rejected outright and explicitly out of scope — this phase is discoverability only; deciding whether an ambiguous Assistant execution actually completed still requires the same manual cross-check (`assistant_turns`/`assistant_financial_drafts`) the runbook already documents, and this phase does not change that.
- **Extending `AssistantIdempotencyRecord`/`ChannelAssistantOperation` schema to add a new "reviewed" or "resolved" flag.** Rejected: no operator workflow exists yet that would consume such a flag (no endpoint, no UI to set it), and adding persisted state for a workflow that doesn't exist yet is speculative. Revisit if/when a real triage workflow is built on top of this phase's read-only visibility.

## Consequences

**Positive:**

- An operator (or CI) can now answer "are there any ambiguous executions or stuck turns right now, and how many" with one command, instead of first knowing the exact SQL from the runbook.
- The safe-projection functions are unit-testable independent of whether the script's own Prisma `select` clause stays correct over time — a future edit that accidentally widens the `select` still can't leak a sensitive field through the projector without a test failing.
- No new dependency, endpoint, authorization system, or background loop was introduced.
- Reuses `AssistantTurn`'s existing `@@index([status, startedAt])` and the runbook's own already-agreed-safe field list — no new schema, no new index, no new judgment call about what is safe to show an operator.

**Costs:**

- Still fundamentally a pull-based tool: nothing pages anyone. An ambiguous execution or a stuck turn is only surfaced the next time someone runs the script (or the runbook's SQL) — this phase does not close that gap, by explicit non-goal.
- The stale-turn threshold is a single fixed constant, not tuned per environment or per intent; a legitimately slow provider call could be misclassified as "stale" until the constant is revisited.

## Related documents

- [PD-015 — Durable Channel Processing](015-durable-channel-processing.md)
- [PD-017 — Assistant Request-Level Idempotency & Turn Recovery](017-assistant-request-level-idempotency.md)
- [PD-013 — Assistant Production Observability Foundation](013-assistant-observability-foundation.md)
- [Telegram Deployment Runbook §8 — Operator Procedures](../development/telegram-deployment-runbook.md#8-operator-procedures-phase-26b)
- [Implementation Roadmap — Phase 28](../development/implementation-roadmap.md)

## Non-goals / explicitly out of scope

- Automatic replay or remediation of any ambiguous or terminal channel/Assistant row.
- Any change to transaction/draft confirmation semantics, or to `assistantOperationGuard`/idempotency-claim behavior.
- An admin HTTP endpoint or admin UI — no admin surface exists in either repository, and none is introduced by this phase.
- A new background scheduler/ticker — the script is operator- or CI-invoked, not self-scheduling.
- Alerting/paging integration — no metrics or alerting backend is connected to page off of.
