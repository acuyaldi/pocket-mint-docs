# PD-015 — Durable Channel Processing (Phase 26B)

**Status:** Accepted
**Date:** 2026-07-26
**Author:** Agent (Phase 26B implementation)

## Context

PD-014 (Phase 25) shipped Telegram channel processing synchronously: the webhook handler awaits `telegram.service.handleUpdate(...)` — dedup claim, command/plain-text handling, the Assistant call, and outbound Telegram delivery — fully before responding. That decision was deliberate and explicitly time-boxed: PD-014 named "queue-backed asynchronous webhook processing" as the documented upgrade path once latency or reliability required it, and listed the concrete cost up front: *"Synchronous webhook processing ties Telegram's delivery latency to this backend's request-handling latency; if Assistant/provider latency grows, this becomes the first thing that needs a queue."*

Phase 26B is that upgrade. It is a processing-reliability change underneath the existing Scope A boundary — not a product feature. Telegram interactive clarification/confirmation (PD-014's "Scope B", still deferred) is explicitly out of scope here and remains untouched.

Discovery for this phase confirmed, again, that no queue, job table, cron, Redis, or worker process exists anywhere in `pocket-mint-be`; Railway runs a single web process. The only existing Postgres concurrency idioms in the codebase are `pg_advisory_xact_lock` (`financial-draft.service.ts`, guarding draft confirmation) and a conditional `UPDATE ... WHERE status = ?` claim (assistant turn processing). This phase introduces one new idiom — `FOR UPDATE SKIP LOCKED` batch claiming — rather than stretching either existing one to fit multi-row concurrent claiming, and deliberately reuses the advisory-lock/conditional-update *philosophy* (database-enforced, not application-enforced, mutual exclusion).

## Decision

**Selected: Option A — PostgreSQL inbox/outbox with application-owned workers, single process, two internal loops.** No new dependency, no dedicated worker process, no external queue.

- **`ChannelUpdateDedup` evolves in place into `ChannelInboundJob`**, rather than adding a parallel table. Its `@@unique([provider, externalUpdateId])` constraint *is* the dedup guarantee from Phase 25 and remains exactly that; the migration renames the table and adds job-lifecycle columns (`status`, `attempt`, `availableAt`, lease fields, `assistantTurnId`, `errorCategory`), backfilling existing rows to `SUCCEEDED` (a legacy dedup row represents an update that was already fully processed under the old synchronous path). Existing dedup history is preserved, not dropped.
- **Persisted job content is the bounded extracted text, never the raw Telegram `Update`.** `ChannelInboundJob.text` holds only what `envelope.ts` already extracted (≤4096 chars) — no metadata, entities, or sender profile. This is the smallest representation that lets a worker reprocess without re-parsing the provider payload, and it does not violate the "never persist raw provider payloads" policy from PD-013/PD-014: those decisions were about *webhook* persistence choices, and this is the deliberately narrower, already-normalized envelope shape those same decisions established.
- **New `ChannelOutboundDelivery` table**, one row per inbound job (`inboundJobId` is itself `@unique`). Telegram sends today are always exactly one reply per update; multi-part chunking is explicitly out of scope (see Non-goals) rather than speculative infrastructure for a case that has never occurred.
- **New `ChannelAssistantOperation` table — the operation-identity guard.** This is the most important addition. `assistant-core-architecture.md` §15 already documents that `/messages` has no request-level idempotency contract (only draft-confirmation does, and that's schema-scoped to a draft). Rather than changing `provider-runtime.ts` — a sensitive, heavily-tested core module — the guard lives entirely in the channel-worker layer: before calling `sendMessage`, the worker inserts a row keyed by `channel:<provider>:<jobId>`; a conflict means a previous attempt on this exact job already started. If that prior attempt recorded a `turnId` and rendered reply, the worker reuses them verbatim and never calls the Assistant again. If it didn't (the previous attempt crashed *between* starting the call and recording the result), the job fails terminally with `errorCategory: ambiguous_assistant_execution` for manual inspection — the system refuses to guess whether a financial mutation already happened rather than risking a duplicate one.
- **Webhook shrinks to bounded transport work only**: verify secret → parse/normalize → reject unsupported → resolve connection (read-only) → one transaction that creates the job (or discovers the P2002 duplicate) → acknowledge. No Assistant call, no Telegram send, no long-running work, in the request path, at all.
- **One inbound worker, one outbound worker**, both started as controlled async loops (`wait → claim batch → process → repeat`, no `setInterval`) inside the existing web process, gated by `CHANNEL_WORKERS_ENABLED` (default on, forced off under `NODE_ENV=test`). Claiming uses `FOR UPDATE SKIP LOCKED` in a short transaction that commits immediately — no transaction is ever held across a network or Assistant call.
- **Retries are classified through the existing `ErrorCategory`/`categorizeError` vocabulary** (`src/utils/errorCategory.ts`) — reused as-is for inbound Assistant failures, and the existing Telegram client's category subset (`provider_rate_limit` / `provider_unavailable` / `provider_invalid_response`) is reused as-is for outbound failures. No new error taxonomy was invented.
- **Retention is opportunistic and bounded**, matching the exact pattern PD-014 already established for `ChannelUpdateDedup` (`ponytail:`-marked inline cleanup on each poll tick, no cron) — extended to both new tables, and never touching `PENDING`/`PROCESSING`/leased rows.

## Rejected alternatives

- **pg-boss.** Would add a new dependency, its own schema, and its own operational model for a workload this phase's own discovery shows is still small (a single external channel, no other tenant of a queue). The custom-concurrency risk pg-boss would remove is exactly the risk `FOR UPDATE SKIP LOCKED` (a well-understood, already-partially-precedented Postgres primitive) already removes at zero dependency cost.
- **External queue / Redis.** No such infrastructure exists or is otherwise justified; this repo's deployment-operations skill already states "No Redis, Kubernetes, queues, or other infrastructure without evidence." Adding one solely for this phase would be exactly that kind of unjustified addition.
- **In-memory queue.** Fails the actual requirement (durability across process restart) by construction — rejected outright, not seriously evaluated further.
- **Railway cron-only processing.** A coarse polling interval driven by an external scheduler is a worse fit than an in-process loop for interactive messaging latency, and would still need the same database claim primitives underneath it — it would add operational surface without removing any.
- **A dedicated worker process.** Rejected for now: this repository has no `Procfile`/`railway.json` and no precedent for a second deployable process; introducing one is a deployment-topology change this phase's scope doesn't require. The claim/lease design does not assume a single process, so splitting into a dedicated worker later is additive, not a rewrite.
- **Modifying `provider-runtime.ts` to add a generic idempotency-key parameter.** Considered, but the smallest correct change was to keep the guard entirely in the channel layer (see Decision) rather than touch the core Assistant invocation path for a channel-specific need.

## Delivery guarantees

Precisely, and without overstating any of them:

- **Webhook acceptance:** durable before acknowledgment. A valid, supported update is persisted in the same transaction that would report a duplicate; the webhook never acknowledges a supported update it hasn't already written.
- **Update deduplication:** unchanged from Phase 25 in mechanism (database unique constraint), now carried by `ChannelInboundJob` instead of `ChannelUpdateDedup`.
- **Assistant execution:** at-most-once, with one narrow, explicitly-accepted exception — a crash strictly between "operation row inserted" and "turn recorded" leaves that job unable to safely resume; it is failed terminally for manual review rather than silently retried. Outside that window, execution is exactly-once per job via the operation-identity guard.
- **Financial mutation:** never duplicated. The existing draft/transaction idempotency mechanics are untouched, and the one ambiguous crash window above fails safe (no automatic retry) rather than risking a second mutation.
- **Outbound Telegram delivery: at-least-once / best-effort, not exactly-once.** Telegram's API gives no delivery-idempotency token. A crash between a successful `sendMessage` call and this backend recording `SENT` can, in principle, cause the same reply to be sent twice on the next claim. This is an accepted, documented limitation — it never causes a duplicate financial mutation, only a possible duplicate message.

## Consequences

**Positive:**

- Telegram webhook response latency is now independent of Assistant/provider latency — the exact cost PD-014 flagged is resolved.
- A process restart (deploy, crash, or scale event) can no longer lose an accepted update or a completed-but-undelivered Assistant reply.
- The claim/lease/retry primitives are provider-neutral (`ChannelInboundJob`/`ChannelOutboundDelivery` are keyed by `ChannelProvider`, currently only `TELEGRAM`) — a future channel reuses this durability boundary without duplicating it.
- No new operational dependency was introduced; the deployment topology (single Railway process) is unchanged.

**Costs:**

- Two additional background loops now run inside the web process; a stuck lease or a runaway retry loop is a new class of operational concern the deployment runbook must document (see the updated runbook).
- The operation-identity guard adds one new table and one new crash-window (the "ambiguous" case) that requires manual operator inspection rather than automatic resolution — an intentional trade against ever risking a duplicate financial mutation.
- Outbound delivery remains best-effort, not exactly-once; this was true before this phase too (Telegram itself gives no better guarantee), but it is now precisely documented instead of implicit.

## Related documents

- [PD-014 — Telegram Channel Foundation](014-telegram-channel-foundation.md)
- [PD-013 — Assistant Observability Foundation](013-assistant-observability-foundation.md)
- [Assistant Core Architecture § 15 — Idempotency](../../architecture/assistant-core-architecture.md)
- [Telegram Deployment Runbook](../../development/telegram-deployment-runbook.md)
- [Implementation Roadmap — Phase 26B](../../development/implementation-roadmap.md)

## Non-goals / explicitly out of scope

- Telegram-native interactive clarification/confirmation (inline buttons) — PD-014's "Scope B", still deferred pending proven product need, not started by this phase.
- Discord, WhatsApp, n8n, or any other channel.
- Redis, Kafka, RabbitMQ, SQS, pg-boss, or any other queue platform/dependency.
- A dedicated worker process or deployment-topology change.
- Multi-part/chunked outbound messages — every reply today is one bounded message; chunking is deferred until a real message actually needs to exceed Telegram's 4096-character cap in a way truncation can't handle.
- Exactly-once outbound Telegram delivery — not claimed, and not achievable without provider support this API doesn't offer.
- Any frontend change — no confirmed user-visible delivery-state requirement exists; the profile page's Telegram connection card is unchanged.
