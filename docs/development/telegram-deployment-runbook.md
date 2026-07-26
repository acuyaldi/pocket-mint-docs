# Telegram Channel Deployment Runbook

Operational runbook for enabling, registering, rotating, and disabling the Telegram channel (Phase 25, [PD-014](../product/decisions/014-telegram-channel-foundation.md)) on `pocket-mint-be`. Companion to the backend's own `docs/deployment-runbook.md`, which owns general Railway/environment conventions this document does not repeat.

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

## 2. Webhook registration (deliberate operator action)

The application never calls Telegram's `setWebhook` itself. After deploying with the channel enabled:

```bash
TELEGRAM_BOT_TOKEN=<token> TELEGRAM_WEBHOOK_SECRET=<secret> \
  node scripts/telegram-set-webhook.mjs https://<railway-domain>/api/v1/telegram/webhook
```

This prints Telegram's own JSON response (never the token/secret) and exits non-zero on failure. Run it once per environment after the backend serving that webhook path is live and reachable over HTTPS.

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

All `channel.*` structured log events (see [Telegram Security § Observability](../architecture/telegram-security.md#observability--what-is-never-logged)) carry a `requestId` correlating to the request; none carry message content or secrets, so they are safe to share when triaging:

| Symptom | Likely event(s) to check | Likely cause |
| --- | --- | --- |
| Telegram shows "webhook failing" | `channel.webhook.rejected` | `TELEGRAM_WEBHOOK_SECRET` mismatch between the deployed backend and the last `setWebhook` call — re-run registration. |
| A linked user's messages go unanswered | `channel.assistant.failed`, `channel.delivery.failed` | Assistant provider misconfigured (`ASSISTANT_PROVIDER` disabled) or Telegram API outage (see §7). |
| `/link <code>` always fails | (no `channel.link.created` follow-through) | Token expired (10-minute TTL), already used, or the user copy-pasted an extra character — ask them to generate a fresh code from `/profile`. |
| Duplicate-looking replies | `channel.update.duplicate` present | Expected — Telegram retried a slow delivery; the dedup table already prevented a second Assistant turn. Not a bug. |

## 7. Telegram API outage handling

If `channel.delivery.failed` events show a sustained `provider_unavailable`/`provider_rate_limit` category, the Assistant already ran and its result is safely persisted (conversation history, any draft) — only the Telegram reply failed to send. No re-execution occurs automatically (delivery and Assistant execution are separate steps, by design — see PD-014). The affected user can retype their message once the outage clears; no data is lost or duplicated by design, but there is no automatic outbound-retry queue in this phase (documented limitation, see PD-014 § Costs).
