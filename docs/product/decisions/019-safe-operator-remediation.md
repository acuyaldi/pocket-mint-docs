# PD-019 — Safe Operator Remediation (Phase 29)

**Status:** Accepted
**Date:** 2026-09-17
**Author:** Agent (Phase 29 implementation)

## Context

Phase 28 ([PD-018](018-assistant-operations-visibility.md)) closed the discoverability gap: an operator can now run `src/scripts/opsVisibility.ts` and get a count of ambiguous assistant/callback executions, terminal inbound/outbound failures, and stale `RUNNING` Assistant turns, instead of needing to already know the runbook's SQL. It deliberately stopped there — PD-018's own "Rejected alternatives" section declined to extend `ChannelInboundJob`/`ChannelAssistantOperation` with a "reviewed" flag because "no operator workflow exists yet that would consume such a flag," and said explicitly to "revisit if/when a real triage workflow is built on top of this phase's read-only visibility."

That triage workflow is still manual today: an operator sees a count from `opsVisibility.ts`, then hand-runs the runbook §8.1 SQL to act — there is no record that a given ambiguous row was ever looked at, and the one safe SQL remediation the runbook documents (requeuing a terminal outbound delivery) has no tooling around it beyond "paste this UPDATE." This phase adds the smallest layer of bounded, explicit, operator-invoked remediation on top of Phase 28's visibility — without touching financial mutation semantics and without ever automatically replaying an ambiguous Assistant execution.

## Decision

**Selected: a new read-only-by-default remediation script, `src/scripts/opsRemediate.ts`, following the same pattern as `reconcile.ts` and `opsVisibility.ts` — not a new HTTP endpoint, not a bulk/"fix all" tool.** Every subcommand takes exactly one explicit row id, defaults to a dry-run print of what it *would* change, and requires an explicit `--apply` flag to actually write. Discovery for this phase re-confirmed PD-018's finding: no admin/operator authorization concept exists in either repository, so this stays a script run at the same trust level as `reconcile.ts`, `db-verify.mjs`, and `opsVisibility.ts`.

- **Minimal, additive schema extension**: `ChannelInboundJob` gains three nullable columns — `reviewedAt`, `reviewedBy`, `reviewNote`. No existing worker code reads or writes these fields, so this is a pure additive migration with zero behavior change to inbound job processing. This is the schema extension PD-018 deferred, now justified by an actual consuming workflow.
- **New pure module, `src/domain/opsRemediation.ts`.** Mirrors `opsVisibility.ts`'s pattern: pure, DB-free functions.
  - Safety gates — `canMarkReviewed(job)`, `canRequeueOutbound(delivery)`, `canReconcileStaleTurn(turn, now)` — each a pure predicate an operator action must pass before any database write is attempted.
  - Update-payload builders — `buildMarkReviewedUpdate(operator, note)`, `buildRequeueOutboundUpdate(now)`, `buildReconcileTurnUpdate(now)` — each returns the *exact*, fixed set of fields the corresponding action is allowed to write. This is the enforcement mechanism, not just documentation: `buildMarkReviewedUpdate`'s output can never contain `status`/`attempt`/`availableAt`/`errorCategory` (the fields that would cause the inbound job to be reprocessed), and `buildRequeueOutboundUpdate`'s output can never contain `renderedText`/`replyMarkup`/`inboundJobId` (nothing about *what* gets sent changes, only *that* a resend is attempted).
- **Three bounded remediation actions, one script, three subcommands:**
  1. `mark-reviewed --job <id> --operator <name> --note <text> [--apply]` — for a `ChannelInboundJob` at `FAILED_TERMINAL` with an ambiguous `errorCategory`. Writes only `reviewedAt`/`reviewedBy`/`reviewNote`. Never changes `status`, so the row can never be reclaimed/reprocessed by this action — the Assistant is never re-invoked, by construction, not just by convention.
  2. `requeue-outbound --delivery <id> [--apply]` — for a `ChannelOutboundDelivery` at `FAILED_TERMINAL`. Codifies the runbook §8.1 SQL (`status → PENDING`, `attempt → 0`, `available_at → now()`, `error_category → NULL`) as a script instead of hand-typed SQL. Only re-sends an already-rendered Telegram message; never touches `ChannelInboundJob`, never calls into Assistant Core, never creates a `Transaction`.
  3. `reconcile-turn --turn <id> [--apply]` — for an `AssistantTurn` at `RUNNING` past `STALE_RUNNING_TURN_MS`. Writes only `status: 'FAILED'`, a dedicated `safeErrorCode` (`operator_reconciled_stale_turn`), and `finishedAt` — the exact field subset `conversation.service.ts`'s own `finalizeRejected` touches on a normal failure path. Never creates or updates an `AssistantFinancialDraft`, never touches `AssistantIdempotencyRecord`, never creates a `Transaction`. This does not resolve the underlying ambiguity about whether the client's original request actually completed — it only stops the turn from reading as "in progress" forever. The operator must still cross-check `assistant_financial_drafts`/`transactions` per runbook §8.1 before telling a user anything about their request.
- **Concurrency safety**: every `--apply` write is a conditional `updateMany` that re-asserts the same precondition the safety gate checked (e.g. `status: 'FAILED_TERMINAL'` for requeue), exactly like the existing `PENDING → CONSUMED` clarification claim pattern (assistant-core skill §8). If the affected-row count isn't `1` — the state changed since the operator last looked — the script reports that and exits non-zero instead of blindly writing.
- **No new subcommand replays an ambiguous execution.** There is deliberately no "reprocess job" or "resume turn" action anywhere in this script — that would re-invoke the Assistant for a request whose completion status is unknown, which is exactly the risk PD-015/PD-016 accepted `ambiguous_assistant_execution`/`ambiguous_callback_execution` to avoid taking.
- **"Export a report" need is already met by Phase 28**: `opsVisibility.ts --json` redirected to a file already produces a focused, redacted operator report; this phase does not duplicate that.

## Rejected alternatives

- **Automatically replaying/reprocessing an ambiguous `ChannelInboundJob`.** Rejected outright — this is the one thing PD-015/PD-016 built the ambiguous-execution category specifically to prevent from happening blindly. Nothing in this phase flips an ambiguous job's `status` back to `PENDING`.
- **A bulk "mark all ambiguous rows reviewed" or "requeue all terminal deliveries" action.** Rejected: every subcommand takes exactly one explicit id. A bulk action removes the one thing that makes this safe — an operator looking at a specific row before acting on it — and turns a bounded remediation tool into an unattended replay mechanism by another name.
- **An HTTP admin endpoint.** Rejected for the same reason as PD-018: no admin authorization concept exists in either repository, and inventing one solely to gate three narrow, low-frequency operator actions is exactly the unrequested surface PD-018 already declined to build.
- **Defaulting to apply-on-run instead of dry-run-by-default.** Rejected: a destructive-by-default CLI is a footgun for a script an operator may run interactively while still reading its output; `--apply` is an explicit, deliberate second step.
- **Resetting `attempt` to something other than `0` on outbound requeue, or leaving it unchanged.** Rejected in favor of matching the runbook §8.1 SQL precedent exactly (`attempt = 0`) — inventing a different reset policy here would silently diverge from the manual procedure operators already know and trust.
- **Folding remediation into `opsVisibility.ts` as new flags.** Rejected: visibility must stay unconditionally read-only so it is always safe to run on a whim (including from CI); mixing a mutating action into that script would break that guarantee.

## Consequences

**Positive:**

- An ambiguous execution an operator has manually investigated can now be recorded as reviewed, with a note, instead of the same row surfacing in every future `opsVisibility.ts` run with no memory of prior investigation.
- The one safe SQL remediation the runbook already documented (outbound requeue) is now a tested, repeatable script instead of hand-typed SQL an operator must get exactly right under pressure.
- A stale `RUNNING` turn can be reconciled to a terminal state without guessing at or fabricating a financial outcome — it only stops the turn from reporting as active forever.
- The safety gates and payload builders are unit-tested independent of the script's own CLI/Prisma wiring, so a future edit that widened a write payload would fail a test before it could reach production.
- No new authorization system, no new endpoint, no bulk-action footgun.

**Costs:**

- `reviewedAt`/`reviewedBy`/`reviewNote` is free-text operator bookkeeping, not a workflow with states or assignment — a deliberately small addition, not a ticketing system.
- Reconciling a stale turn still leaves the original ambiguity about draft/transaction completion for the operator to resolve by hand (runbook §8.1) — this phase narrows what "stuck" means without automating the actual investigation.
- Three narrow subcommands, not a general remediation framework — a fourth future case (if one arises) gets its own reviewed subcommand and its own safety gate, not a generic "run any update" escape hatch.

## Related documents

- [PD-018 — Assistant Operations Visibility](018-assistant-operations-visibility.md)
- [PD-017 — Assistant Request-Level Idempotency & Turn Recovery](017-assistant-request-level-idempotency.md)
- [PD-016 — Telegram Interactive Workflows](016-telegram-interactive-workflows.md)
- [PD-015 — Durable Channel Processing](015-durable-channel-processing.md)
- [Telegram Deployment Runbook §8 — Operator Procedures](../development/telegram-deployment-runbook.md#8-operator-procedures-phase-26b--phase-28--phase-29)
- [Implementation Roadmap — Phase 29](../development/implementation-roadmap.md)

## Non-goals / explicitly out of scope

- Automatic replay or re-execution of any ambiguous Assistant/callback execution — no subcommand ever flips an ambiguous job back to `PENDING`.
- Any change to `assistantOperationGuard`, idempotency-claim behavior, or transaction/draft confirmation semantics.
- Repeating or simulating a financial mutation of any kind.
- An admin HTTP endpoint or admin UI — no admin surface exists in either repository, and none is introduced by this phase.
- A bulk/"fix all" remediation mode — every action requires one explicit identifier.
- A new background scheduler/ticker — remediation stays operator-invoked, never self-scheduling.
- Cross-worker ordering or queue redesign of any kind.
