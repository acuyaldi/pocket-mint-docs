# Telegram Channel Deployment Runbook

Operational runbook for enabling, registering, rotating, and disabling the Telegram channel (Phase 25, [PD-014](../product/decisions/014-telegram-channel-foundation.md); durable processing added in Phase 26B, [PD-015](../product/decisions/015-durable-channel-processing.md)) on `pocket-mint-be`. Companion to the backend's own `docs/deployment-runbook.md`, which owns general Railway/environment conventions this document does not repeat.

No secret values appear in this document. Fill placeholders (`<...>`) from the platform's secret manager at deploy time.

## 1. Environment variables

Parsed in `src/telegram/config.ts` (`pocket-mint-be`), following the same hand-rolled-parsing convention as every other backend config module — no Zod, no separate secret store.

| Variable | Required when enabled | Sensitive | Notes |
| --- | --- | --- | --- |
| `TELEGRAM_ENABLED` | — | no | `true`/`false`. Default `false` — the webhook route always returns 401 and the Telegram client is never constructed while unset. |
| `TELEGRAM_BOT_TOKEN` | yes | **yes** | From [@BotFather](https://t.me/BotFather). Never logged; used only in the Telegram API request path. |
| `TELEGRAM_WEBHOOK_SECRET` | yes | **yes** | A random shared secret you choose (not Telegram-issued) — verified against Telegram's `X-Telegram-Bot-Api-Secret-Token` header on every webhook call. |
| `TELEGRAM_BOT_USERNAME` | no | no | Public value, shown to users in linking instructions only. |

Setting `TELEGRAM_ENABLED=true` without `TELEGRAM_BOT_TOKEN`/`TELEGRAM_WEBHOOK_SECRET` fails fast at process startup (the config module throws) — it cannot silently boot into a half-configured, unsafe state.

### 1.1 Channel worker environment variables (Phase 26B)

Parsed in `src/channels/workerConfig.ts`. These control the inbound/outbound processing loops described in §2a — they are generic to the channel-durability layer, not Telegram-specific, so a future channel reuses the same variables.

| Variable | Default | Notes |
| --- | --- | --- |
| `CHANNEL_WORKERS_ENABLED` | `true` | `true`/`false`. Forced `false` automatically under `NODE_ENV=test` regardless of this value — the test suite never starts a background poll loop. |
| `CHANNEL_INBOUND_POLL_MS` | `2000` | Idle-sleep interval for the inbound loop when the last claim returned nothing. |
| `CHANNEL_OUTBOUND_POLL_MS` | `2000` | Same, for the outbound loop. |
| `CHANNEL_BATCH_SIZE` | `10` | Max rows claimed per poll tick, per worker. |
| `CHANNEL_LEASE_MS` | `30000` | How long a claimed row is protected from being reclaimed by another worker instance. |
| `CHANNEL_MAX_ATTEMPTS_INBOUND` | `5` | Attempts before a retryable inbound failure becomes terminal. |
| `CHANNEL_MAX_ATTEMPTS_OUTBOUND` | `5` | Attempts before a retryable outbound (Telegram send) failure becomes terminal. |
| `CHANNEL_RETENTION_DAYS` | `7` | Successful/terminal rows older than this are opportunistically deleted; terminal rows get 4× this window before deletion (see §8). |

None of these are secrets. Both workers run inside the same Railway web process as the HTTP server (see §2a) — there is no separate worker process to configure.

## 2. Webhook registration (deliberate operator action)

The application never calls Telegram's `setWebhook` itself. After deploying with the channel enabled:

```bash
TELEGRAM_BOT_TOKEN=<token> TELEGRAM_WEBHOOK_SECRET=<secret> \
  node scripts/telegram-set-webhook.mjs https://<railway-domain>/api/v1/telegram/webhook
```

This prints Telegram's own JSON response (never the token/secret) and exits non-zero on failure. Run it once per environment after the backend serving that webhook path is live and reachable over HTTPS.

## 2a. Processing model & worker topology (Phase 26B)

As of Phase 26B ([PD-015](../product/decisions/015-durable-channel-processing.md)), the webhook no longer processes an update synchronously. It only verifies, parses, resolves the connection, and durably persists a `ChannelInboundJob` before acknowledging. Two background loops — started inside the same Railway web process as the HTTP server, not a separate deployable — do the rest:

- **Inbound worker:** claims pending/expired-lease jobs (`FOR UPDATE SKIP LOCKED`), resolves the connection fresh, runs commands or calls the Assistant, and creates one `ChannelOutboundDelivery`.
- **Outbound worker:** claims pending/expired-lease deliveries the same way and sends via the Telegram client, with bounded retry.

Both loops are safe to run across multiple Railway replicas — claiming is database-authoritative (leases + `SKIP LOCKED`), not "exactly one process" dependent. Disabling `CHANNEL_WORKERS_ENABLED` stops both loops but leaves the webhook accepting and persisting updates normally; they simply queue as `PENDING` until workers are re-enabled.

## 3. Environment separation

Follows the existing staging/production split (`dev` → Railway staging, `main` → Railway production, `master` retired):

- Use a **separate Telegram bot** (separate `TELEGRAM_BOT_TOKEN`) per environment. Never point a staging bot's webhook at the production URL or vice versa.
- Use a distinct `TELEGRAM_WEBHOOK_SECRET` per environment.
- Manual validation (see PD-014 and the Assistant Core spec's manual-validation checklist) must use a dedicated test bot and a fictional Pocket Mint account — never the production bot.

## 4. Webhook removal / rotation

**Disable the channel** (fastest, safest): set `TELEGRAM_ENABLED=false` and redeploy. The webhook route immediately returns 401 for every request regardless of what Telegram still has registered; Telegram will see failed deliveries and back off retrying on its own schedule.

**Remove the webhook registration** (stop Telegram from calling at all):

```bash
TELEGRAM_BOT_TOKEN=<token> node scripts/telegram-set-webhook.mjs --delete
```

**Rotate the webhook secret:** generate a new `TELEGRAM_WEBHOOK_SECRET`, deploy it, then re-run the `setWebhook` registration command with the new secret. There is a brief window between deploy and re-registration where Telegram may still send the old secret header — those requests will 401 and Telegram will retry, so no update is lost as long as re-registration happens promptly.

**Rotate the bot token:** revoke/regenerate via @BotFather, update `TELEGRAM_BOT_TOKEN` in the platform's secret manager, redeploy, then re-run `setWebhook` (the URL and secret token are unaffected by a bot-token rotation, but Telegram associates the webhook with the bot token, so re-registration is required).

## 5. Local development

Telegram requires an HTTPS-reachable webhook URL, so local development typically uses a disabled channel (`TELEGRAM_ENABLED` unset) plus direct unit/integration tests, or a tunneling tool (ngrok or similar) pointed at a local dev server with its own throwaway bot token if manual end-to-end testing is needed. Never register a tunnel URL against the production bot.

## 6. Troubleshooting via observability events

All `channel.*` structured log events (see [Telegram Security § Observability](../architecture/telegram-security.md#observability--what-is-never-logged)) carry a `requestId` correlating to the request/job; none carry message content or secrets, so they are safe to share when triaging:

| Symptom | Likely event(s) to check | Likely cause |
| --- | --- | --- |
| Telegram shows "webhook failing" | `channel.webhook.rejected` | `TELEGRAM_WEBHOOK_SECRET` mismatch between the deployed backend and the last `setWebhook` call — re-run registration. |
| A linked user's messages go unanswered | `channel.inbound.failed`, `channel.outbound.failed` | Assistant provider misconfigured (`ASSISTANT_PROVIDER` disabled), Telegram API outage (see §7), or workers disabled (`CHANNEL_WORKERS_ENABLED=false`) — check §8 for stuck-job inspection. |
| `/link <code>` always fails | (no `channel.link.created` follow-through) | Token expired (10-minute TTL), already used, or the user copy-pasted an extra character — ask them to generate a fresh code from `/profile`. |
| Duplicate-looking replies | `channel.inbound.duplicate` present | Expected — Telegram retried a slow delivery; `ChannelInboundJob`'s unique constraint already prevented a second Assistant turn. Not a bug. |
| Jobs never seem to process | `channel.worker.started` absent at boot | `CHANNEL_WORKERS_ENABLED=false`, or `TELEGRAM_ENABLED=false` (workers are only constructed when the channel is enabled). |
| A job is stuck `PROCESSING`/`SENDING` for a long time | `channel.inbound.lease_recovered` / `channel.outbound.lease_recovered` absent | The owning worker crashed mid-job; it self-heals once `CHANNEL_LEASE_MS` elapses and another poll reclaims it — see §8 if it doesn't. |
| A job repeatedly fails with `errorCategory: ambiguous_assistant_execution` | `channel.inbound.failed` | The rare crash window PD-015 documents: a prior attempt started the Assistant call but crashed before recording the result. This is terminal by design — see §8 for manual inspection; do not blindly retry it. |

## 7. Telegram API outage handling

If `channel.outbound.failed` events show a sustained `provider_unavailable`/`provider_rate_limit` category, the Assistant already ran and its result is safely persisted (conversation history, any draft, and the rendered reply on the `ChannelOutboundDelivery` row) — only the Telegram send failed. The outbound worker retries automatically with backoff up to `CHANNEL_MAX_ATTEMPTS_OUTBOUND`; no re-execution of the Assistant occurs (delivery and Assistant execution are separate steps, by design — see PD-014 and PD-015). Once the outage clears, pending retries resolve on their own; no manual action is normally required.

## 8. Operator procedures (Phase 26B)

All procedures below are read-only inspections or bounded, targeted updates — never ad hoc payload injection, and never a direct HTTP admin endpoint (none exists; these are database-safe operator queries run against the application database).

**Inspect aggregate job/delivery counts** (by status, to spot a growing backlog or a stuck lease):

```sql
SELECT status, count(*) FROM channel_inbound_jobs GROUP BY status;
SELECT status, count(*) FROM channel_outbound_deliveries GROUP BY status;
```

**Find stuck leases** (claimed but not completed well past their lease window — usually self-heals on the next poll, but useful to confirm workers are actually running):

```sql
SELECT id, status, lease_owner, lease_expires_at, attempt
FROM channel_inbound_jobs
WHERE status = 'PROCESSING' AND lease_expires_at < now();
```

**Inspect a terminally failed job** (never retry blindly — read `error_category` first):

```sql
SELECT id, status, attempt, error_category, completed_at
FROM channel_inbound_jobs
WHERE status = 'FAILED_TERMINAL'
ORDER BY completed_at DESC LIMIT 20;
```

`error_category = ambiguous_assistant_execution` specifically means the Assistant call may or may not have completed for that job — check `assistant_turns`/`assistant_financial_drafts` for a matching turn from around the same time before deciding whether the user needs to be asked to retry manually. Never flip this row back to `PENDING` without that check — retrying the underlying job would re-invoke the Assistant.

**Manually retry a terminal outbound delivery** (safe: this only re-sends Telegram, never re-runs the Assistant):

```sql
UPDATE channel_outbound_deliveries
SET status = 'PENDING', available_at = now(), attempt = 0, error_category = NULL
WHERE id = '<delivery-id>' AND status = 'FAILED_TERMINAL';
```

**Pause/resume workers:** set `CHANNEL_WORKERS_ENABLED=false` (or `true`) and redeploy. Pending jobs/deliveries are untouched and resume processing as soon as workers restart — nothing is lost while paused.

**Clean up retention data manually** (normally unnecessary — both workers already run this opportunistically on every poll tick that processes at least one row):

```sql
-- Never deletes PENDING/PROCESSING/SENDING/leased rows — see src/channels/retention.ts.
```

Prefer letting the worker's own inline cleanup run rather than deleting rows by hand; it already excludes anything active or leased.
