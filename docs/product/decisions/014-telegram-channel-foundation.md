# PD-014 — Telegram Channel Foundation (Phase 25)

**Status:** Accepted
**Date:** 2026-07-26
**Author:** Agent (Phase 25 implementation)

> **Update (Phase 26B):** The synchronous webhook processing described below
> was the deliberate, time-boxed v1 tradeoff this decision names in its own
> "Alternatives considered" and "Costs" sections. It has been superseded by
> [PD-015 — Durable Channel Processing](015-durable-channel-processing.md),
> which replaces it with a durable Postgres-backed inbox/outbox pipeline. The
> context and reasoning below remain an accurate record of Phase 25 as
> originally shipped — read PD-015 for the current processing model.

## Context

The Assistant has only ever been reachable from the Pocket Mint web frontend. PD-012 (§ "Explicitly out of scope") and the Assistant Core Architecture doc (§15.2, §26) both listed Telegram/Discord/WhatsApp and external-channel identity mapping as deliberately unbuilt — the Channel Adapter box in the architecture's component model (§6/§7) has, until now, been a diagram entry with no implementation.

Phase 25 fills that gap with the first external channel: Telegram. The goal is not to reproduce the full web experience — it is to prove a secure, auditable, channel-agnostic boundary that:

- resolves a Telegram identity to a Pocket Mint user without ever trusting Telegram-supplied identity as authorization;
- reaches the Assistant through its existing, already-transport-neutral application boundary (`assistantProviderRuntime.sendMessage` / `assistantApplicationService.execute`) rather than duplicating conversation, policy, draft, or clarification logic;
- never weakens the Draft → Preview → Explicit Confirmation → Commit boundary (architecture doc §13) merely because Telegram's UI is a plain-text surface.

This matters now because it is the first genuinely public (unauthenticated-by-JWT) route this backend has ever exposed, and the first time an external, unauthenticated principal's message can reach the Assistant at all — every invariant below exists to keep that principal's authority bounded to exactly the Pocket Mint user it was linked to, and nothing else.

## Decision

- **Adapter, not a second Assistant.** `src/telegram/` owns transport concerns only (webhook parsing, command syntax, outbound delivery); `src/channels/` owns the provider-agnostic identity/linking boundary. Neither contains financial business logic, a second tool registry, or a parallel confirmation mechanism. Plain-text messages from a linked identity call `assistantProviderRuntime.sendMessage(userId, correlationId, { message, conversationId? })` — the exact same call `POST /assistant/messages` makes.
- **Canonical envelope.** `InboundChannelMessage { provider, externalUpdateId, externalChatId, externalSenderId, text, receivedAt }` is the only shape that crosses from `src/telegram/envelope.ts` into the rest of the system. The raw Telegram `Update` type never leaves `schema.ts`/`envelope.ts`, and is never persisted.
- **Identity linking, digest-only and one-time.** `ChannelLinkToken` mirrors PD-012's clarification-token pattern exactly: a random 32-byte value returned once to the authenticated web user, only its SHA-256 digest persisted, a 10-minute TTL, and an atomic conditional `updateMany` claim (`consumedAt: null` → set) so a replayed `/link <code>` is indistinguishable from an expired or invalid one.
- **Ownership-scoped connection, no implicit transfer.** `ChannelConnection` has `@@unique([provider, externalUserId])` (one Telegram identity → one Pocket Mint user) and `@@unique([userId, provider])` (one active Telegram connection per user), enforced in the database. Consuming a link token for an `externalUserId` already bound to a *different* user fails closed — the existing connection is never silently rewritten. A transfer requires an explicit `/unlink` (self-scoped to the caller's own identity) followed by a fresh link.
- **Private chats only.** `schema.ts` accepts only `update.message` with `chat.type === 'private'` and non-empty `text`; groups, supergroups, channels, `edited_message`, `callback_query`, `channel_post`, and non-text content are all normalized to "unsupported" and safely ignored (200 OK, no processing, no connection touched).
- **Authoritative update dedup.** `ChannelUpdateDedup` claims `(provider, externalUpdateId)` via a unique constraint before any processing; a retried Telegram delivery is a database-enforced no-op, not a best-effort in-memory guard. Retention is a bounded inline `deleteMany` on each webhook call (`ponytail:` no queue/cron exists in this backend — see below) rather than an unbounded archive.
- **Synchronous webhook processing.** The webhook handler awaits `telegram.service.handleUpdate(...)` fully before responding. No queue or background-job infrastructure exists in this backend (verified by discovery — none, anywhere). A "respond 200 then keep working" fire-and-forget promise was explicitly rejected because it can be lost on a process restart mid-request; the dedup table is the safety net if a slow response causes Telegram to retry.
- **Scope A confirmation and clarification: web handoff, not Telegram buttons.** A financial draft or a clarification-required outcome gets one fixed, bounded reply ("continue in the Pocket Mint web app") — no draft payload, no token, no deep link with an identifying value. Telegram never confirms, cancels, or selects a clarification option.
- **Outbound delivery is a separate, non-retried-into-reexecution step.** `src/telegram/client.ts` is a small typed `fetch` wrapper (no SDK) that classifies failures using the same `ErrorCategory` values the rest of the backend already uses (`provider_rate_limit` / `provider_unavailable` / `provider_invalid_response`) rather than inventing Telegram-specific categories. A delivery failure never re-invokes the Assistant.
- **Webhook registration is a deliberate operator action.** `scripts/telegram-set-webhook.mjs` is a standalone script; nothing in the application registers a webhook at boot.

## Alternatives considered

- **Scope B (Telegram inline-button confirmation/clarification).** Rejected for this phase: opaque bounded callback data, callback-replay-safety, and their own concurrency tests are real, non-trivial work with real risk if done casually — the same category of risk PD-012 already worked through for the web clarification flow. Preferred Scope A until product demand proves Scope B's cost is worth paying.
- **Bolt a `channel`/`source` column onto `AssistantConversation`.** Rejected: the Assistant's conversation model doesn't need to know it was reached via Telegram to behave correctly — `ChannelConnection.conversationId` already tracks "which conversation is this Telegram chat currently on," which is the only thing the channel layer needs. Keeps the Assistant models free of a transport concern.
- **Queue-backed asynchronous webhook processing.** Rejected for this phase: no queue/background-job infrastructure exists anywhere in this backend today; adding one speculatively, before synchronous processing is shown to actually violate Telegram's timeout, would be new infrastructure risk for an unproven problem. Documented as the upgrade path if latency ever requires it.
- **A full Telegram Bot SDK.** Rejected: this phase needs exactly `sendMessage`, `setWebhook`, `deleteWebhook` — a ~60-line typed `fetch` wrapper covers it with less surface area to audit than a general-purpose framework.
- **Telegram username or numeric user ID treated as sufficient identity.** Rejected outright — usernames are mutable and user IDs are guessable; only a digest-verified, one-time, expiring linking token issued to an already-authenticated Pocket Mint session can bind an external identity.
- **Group chat support in this phase.** Rejected: private-user financial authorization semantics do not translate safely to a shared chat; deferred pending a specific, separately-designed product need.

## Consequences

**Positive:**

- The Assistant's existing policy, entity-resolution, clarification, draft, and idempotency mechanics are exercised unchanged from a second transport — proving the "channel-independent foundation" claim from PD-012 § Consequences.
- Cross-user isolation, replay safety, and identity-transfer prevention are enforced at the database layer (unique constraints, atomic conditional claims), not only in application code.
- The first public route in this backend is deliberately narrow (secret-header verification, IP-scoped rate limiting, no CORS implications) rather than a general-purpose public API surface.
- Future Discord/WhatsApp adapters can reuse `src/channels/` (link tokens, connections, dedup pattern) and only need to add their own `src/<provider>/` transport layer.

**Costs:**

- Synchronous webhook processing ties Telegram's delivery latency to this backend's request-handling latency; if Assistant/provider latency grows, this becomes the first thing that needs a queue.
- `ChannelUpdateDedup` retention is a best-effort inline cleanup, not a scheduled job — acceptable at current volume, revisit if row count ever matters.
- Scope A means every draft/clarification interaction still requires switching to the web app — a real UX limitation until Scope B (or a smaller Telegram-side capability) is explicitly requested and designed.
- A new, first-of-its-kind frontend surface (`TelegramConnectionCard` on `/profile`) had to be added purely to give a user a place to mint a linking code — a small but real frontend cost for a backend-first phase.

## Related documents

- [Assistant Core Architecture § 6/7 — Channel Adapter component](../../architecture/assistant-core-architecture.md)
- [Assistant Core Architecture § 15.2 — Persistent Clarification Engine (channel-independent foundation claim)](../../architecture/assistant-core-architecture.md#152-persistent-clarification-engine)
- [PD-012 — Persistent Clarification Engine and Deterministic Continuation](012-persistent-clarification-engine.md)
- [Telegram Security](../../architecture/telegram-security.md)
- [Telegram Deployment Runbook](../../development/telegram-deployment-runbook.md)
- [Implementation Roadmap — Phase 25](../../development/implementation-roadmap.md)

## Explicitly out of scope (deferred to later phases or not planned)

- Discord, WhatsApp, n8n, or any other external channel/orchestration.
- Telegram group chats, supergroups, channels, inline queries, edited-message reconciliation, media, voice, location, contact sharing.
- Scope B (Telegram-native inline-button draft confirmation or clarification-option selection).
- A general notification platform, a provider-agnostic database abstraction beyond what Telegram proved necessary, and a generic frontend integrations marketplace.
- Automatic financial confirmation or any bypass of the existing confirmation policy.
