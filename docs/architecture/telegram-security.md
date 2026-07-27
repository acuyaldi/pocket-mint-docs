# Telegram Channel Security

Companion to [PD-014 — Telegram Channel Foundation](../product/decisions/014-telegram-channel-foundation.md) and [Assistant Core Architecture § 15.4](./assistant-core-architecture.md#154-telegram-channel-phase-25). This document is the security-review-level detail; PD-014 owns the decision rationale.

## Webhook authenticity

- Telegram's shared `secret_token` (set via `setWebhook`) is sent on every request as `X-Telegram-Bot-Api-Secret-Token`. `src/controllers/telegram.controller.ts`'s `verifyTelegramWebhookSecret` compares it against `TELEGRAM_WEBHOOK_SECRET` using `crypto.timingSafeEqual` (constant-time, avoids a timing side-channel) before any body parsing or processing happens.
- The channel is fail-closed: if `TELEGRAM_ENABLED` is not `true`, every request to the webhook route returns 401, regardless of the header supplied.
- The route is the first genuinely public (non-JWT-gated) route in this backend. It is scoped as narrowly as possible: one path, one purpose, an IP-keyed rate limiter (`telegramWebhookLimiter`), and no CORS implications (server-to-server calls carry no `Origin` header, so the existing CORS allowlist is a non-issue here — verified, not assumed).

## Bot-token handling

- `TELEGRAM_BOT_TOKEN` is read once in `src/telegram/config.ts` (the same hand-rolled-parsing convention as every other backend secret, see `src/config/assistant-provider.ts`) and used only inside `src/telegram/client.ts`'s request URL (`https://api.telegram.org/bot<token>/...`) — Telegram's API puts the token in the path, not a header or body field, so there is nothing to accidentally log in a request body.
- The structured logger's redaction (`src/utils/logger.ts`) treats any key containing `token` as sensitive as a backstop, but the primary control is that the token is never passed as a logged value anywhere in `src/telegram/`.
- The webhook secret is handled identically: read once, compared, never logged.

## Account linking

- `ChannelLinkToken`: a random 32-byte value (`crypto.randomBytes(32).toString('base64url')`), returned to the authenticated web caller exactly once at creation. Only `sha256(token)` is persisted (`tokenDigest`). 10-minute TTL.
- Consumption (`consumeLinkToken`) is an atomic conditional claim: `updateMany({ where: { tokenDigest, provider, consumedAt: null, expiresAt: { gt: now } }, data: { consumedAt: now } })`. A `count !== 1` result — invalid, expired, or already-used — throws one indistinguishable error (`CHANNEL_LINK_INVALID`), so a `/link` attacker cannot learn which of the three occurred.
- Verified under concurrency: `Promise.all` of six concurrent consumption attempts against the same token resolves to exactly one success (integration test, `pocket-mint-be/test/channels/telegram-channel.integration.test.ts`).

## Ownership and identity-transfer prevention

- `ChannelConnection` has two database-level unique constraints: `(provider, externalUserId)` — one Telegram identity maps to exactly one Pocket Mint user — and `(userId, provider)` — one active Telegram connection per user. Neither is application-code-only.
- Consuming a link token for an `externalUserId` already bound to a *different* `userId` fails with `CHANNEL_LINK_IDENTITY_CONFLICT` inside the same transaction as the token claim, so the claim itself rolls back too (the token is not burned by a conflict it didn't cause) and the existing connection's `userId` is never silently rewritten.
- Every inbound Telegram command or message resolves `externalSenderId → ChannelConnection (status ACTIVE) → userId` before anything else happens. There is no code path that accepts a `userId` from message text, a command argument, or any Telegram-supplied field.
- `/unlink` and `/status` are scoped to the caller's own `externalSenderId` — a user can only ever act on their own connection, never another's, by construction (the lookup key is the identity the webhook itself received, not an argument).

## Update deduplication and replay

- `ChannelUpdateDedup` has a unique constraint on `(provider, externalUpdateId)`. `claimTelegramUpdate` (`src/telegram/dedup.ts`) attempts an insert and treats a `P2002` unique-violation as "already processed, no-op" — the same pattern used elsewhere in this backend for idempotency-record races (`AssistantFinancialDraftService.confirm`).
- Verified under concurrency: two simultaneous deliveries of the same `update_id` result in exactly one Assistant invocation and exactly one dedup row (integration test).
- Retention is a bounded inline `deleteMany` (rows older than 7 days) run alongside the claim, in the same transaction — no unbounded archive, no separate cron process (this backend has none).

## Private-chat restriction

- `src/telegram/schema.ts` accepts only `update.message` where `chat.type === 'private'` and `text` is a non-empty string. Group, supergroup, channel updates, `edited_message`, `callback_query`, `channel_post`, and non-text content are all normalized to "unsupported" — safely ignored (200 OK, no side effects), never partially processed.
- This is enforced structurally in the parser, not by a downstream permission check — an unsupported update never reaches identity resolution, dedup claiming, or the Assistant at all.

## Confirmation-boundary preservation

- A Telegram reply is classified from the same response shape (`success` / `clarification_required` / `rejected` / `error` / `unsupported`) the web frontend receives from `assistantProviderRuntime.sendMessage`.
- **As of PD-016 (Phase 26A)**, a `clarification_required` result with supported options, or a freshly created draft, is rendered as an inline keyboard of opaque callback tokens instead of unconditionally falling back to the fixed web-handoff message — see "Interactive callbacks" below. Every case Telegram still cannot safely handle (STEP_UP, free-form clarification) continues to receive the original fixed, bounded "continue on the web app" instruction, with no draft ID, clarification ID, option token, or other identifying value included in it.

## Interactive callbacks (Phase 26A)

Companion to [PD-016 — Telegram Interactive Clarification & Confirmation](../product/decisions/016-telegram-interactive-workflows.md), which owns the full decision rationale. Security-review-level detail only:

- **`callback_data` is always an opaque, randomly generated handle** (`ChannelCallbackToken`, `cbk_` + 24 random bytes, base64url) — never a domain identifier, never an action name, never a value. Only its `sha256` digest is persisted (`@@unique([provider, tokenDigest])`), mirroring `ClarificationOption.tokenDigest` exactly.
- **Ownership is re-derived from scratch on every callback**, never trusted from the Telegram payload: token digest lookup → token `PENDING` and unexpired → bound `ChannelConnection` is `ACTIVE` → `callback_query.from.id` matches that connection's `externalUserId` → the token's `conversationId` matches the connection's *current* `conversationId`. Only after all five checks pass is the token atomically claimed and the authoritative clarification/draft service invoked.
- **A revoked connection cannot act** — step 3 above fails closed the instant `ChannelConnection.status !== 'ACTIVE'`, with no separate keyboard-invalidation step required; the previously delivered keyboard is inert server-side from that moment on.
- **A stale keyboard cannot reactivate a terminal clarification or draft** — pressing an old button after the underlying resource has already reached a terminal state (via web or a different Telegram press) returns that actual terminal status; it never creates a new clarification/draft or flips a terminal row back to `PENDING`.
- **The one-time token claim is the race gate between two independent button presses** on the same token; the underlying domain services' own guards (clarification's conditional claim, the draft's advisory lock + idempotency key) are a second, independent layer, not the only one.
- **`answerCallbackQuery` is not the authoritative result** — it is a synchronous, neutral acknowledgment (`"Processing…"` or no text) that happens once, in the webhook, and never claims success before the durable worker actually resolves the button press.
- Nothing about this feature weakens the digest-only, one-time-consumption, or database-time-bounded-expiry guarantees the Persistent Clarification Engine and Pending Financial Draft lifecycle already provide — Telegram calls the same authoritative services the web client does, through `AssistantProviderRuntime` passthroughs, never a parallel path.

## Observability — what is never logged

Every `channel.*` event (`webhook.received`, `webhook.rejected`, `update.duplicate`, `link.created`, `connection.revoked`, `assistant.started/completed/failed`, `delivery.started/completed/failed`) carries only: `event`, `requestId` (correlation id), `provider: "telegram"`, `durationMs`, `outcome`/`errorCategory`. None of the following are ever logged, by construction (there is no code path that reads them into a log call): Telegram message text, usernames, display names, chat titles, the bot token, the webhook secret, linking tokens (raw or digest), or raw webhook payloads.

## Deployment separation

- `TELEGRAM_ENABLED` defaults to unset/false; the webhook route, the linking API's practical usefulness, and the Telegram client are all inert until it is explicitly turned on per environment (see the [deployment runbook](../development/telegram-deployment-runbook.md)).
- Webhook registration (`scripts/telegram-set-webhook.mjs`) is a manual, operator-run script — never triggered by application startup, so a staging bot token can never accidentally register a production webhook URL or vice versa.
