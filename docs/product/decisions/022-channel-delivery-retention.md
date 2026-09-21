# PD-022 — Channel Delivery Retention & Historical UX Semantics (Phase 32)

**Status:** Accepted
**Date:** 2026-09-21
**Author:** Agent (Phase 32 implementation)

## Context

Phase 31 ([PD-021](021-channel-delivery-receipts.md)) added `turn.deliveryStatus`, computed at read time from the still-live `ChannelOutboundDelivery` row for a TELEGRAM turn. That row is retention-managed (`src/channels/retention.ts`, default 7 days for a `SENT` delivery, 28 days for `FAILED_TERMINAL` — see the deployment runbook §1.1/§8.1), so once a Telegram-originated turn is old enough, its delivery row no longer exists. Phase 31 deliberately chose to **omit** the field in that case rather than guess a value — safe, but it left one real ambiguity un-closed: on the wire, "this TELEGRAM turn's delivery row was purged" and "this backend predates Phase 31 and has no concept of delivery status at all" produce the exact same JSON shape — the key is simply not there. Nothing was broken (the frontend already renders nothing for an absent field, so old history never *looked* broken), but the API contract itself did not let a client tell those two situations apart, and the omission was undocumented beyond one inline code comment. This phase closes that gap.

## Decision

**Selected: replace the omission with an explicit `'UNKNOWN'` value in the existing `AssistantDeliveryStatus` union. On a Phase-32-or-later backend, `turn.deliveryStatus` is always present — never omitted — for every turn.**

- **Why a sixth enum value, not a second field.** The phase brief itself offered three shapes: keep the omission (documented only), add `'UNKNOWN'` to the existing enum, or add a parallel `deliveryStatusRetained: boolean`. The boolean was rejected first: it would duplicate information a single enum value already expresses completely, be one more field to document and keep in sync, and does not map cleanly onto the frontend's existing status-keyed rendering switch. Between "keep omitting" and "add `UNKNOWN`," the explicit value wins because it removes a real, currently-unresolvable ambiguity (pre-Phase-31 backend vs. purged-row Phase-31-or-later backend) that pure documentation cannot fix — documentation can only describe existing wire behavior, not make two indistinguishable payloads distinguishable.
  - `resolveDeliveryStatus(channel, mappedStatus)` (new, exported pure function in `conversation.service.ts`, same testability pattern as Phase 31's `DELIVERY_STATUS_MAP`) makes the three-way split explicit: `NOT_APPLICABLE` for `WEB`; the Phase 31 mapped status for `TELEGRAM` with a retained row; `'UNKNOWN'` for `TELEGRAM` with no retained row (`mappedStatus ?? 'UNKNOWN'`).
  - The only wire-format change from Phase 31 is that this one previously-omitted case now has a value. Every value Phase 31 already sent (`NOT_APPLICABLE`/`PENDING`/`PROCESSING`/`DELIVERED`/`FAILED`) keeps its exact meaning — additive, not a breaking change to any existing consumer that only switches on known values (and, notably, the Phase 31 frontend's own `deliveryStateOf` switch already has a `default: return null` arm, so it degrades safely on `'UNKNOWN'` with zero required frontend change).
- **No retention policy change.** `src/channels/retention.ts` and `CHANNEL_RETENTION_DAYS` are untouched — this phase is entirely a read-path/DTO clarification of what already happens once a row ages out, not a change to when that happens.
- **Frontend: render nothing for `UNKNOWN`, on purpose.** The phase brief allowed "restrained copy/icon only if useful." Nothing is useful here: `UNKNOWN` conveys no actionable information ("we no longer know" isn't reassuring or alarming, it's just absent data), and the one hard requirement — historical unknown must never look like failure — is trivially and unconditionally satisfied by rendering nothing at all, the same as `NOT_APPLICABLE` and the pre-Phase-31-absent case already do. `AssistantMessage`'s `deliveryStateOf` switch's existing `default` arm already produces exactly this behavior for an unrecognized value; Phase 32 only needs the frontend's `AssistantDeliveryStatus` union widened to name `'UNKNOWN'` for type accuracy, not a new render branch.
- **Documentation:** `docs/api/assistant-conversations.md` now states the field is always present on a current backend and spells out all six values; the Telegram deployment runbook cross-references that lowering `CHANNEL_RETENTION_DAYS` is a real, visible product trade-off (how long a real `DELIVERED`/`FAILED` outcome stays visible before it reads as `UNKNOWN`), not merely a storage-cost one.

## Rejected alternatives

- **`deliveryStatusRetained: boolean` alongside the existing status field.** Rejected — see above; redundant with a single enum value, more surface area, no cleaner for the frontend to consume.
- **Keep the omission, document it more thoroughly.** Rejected as insufficient: the ambiguity is structural (two different backend states produce identical JSON), not a documentation gap that better prose can close. Documenting "the field may be absent for two different reasons, indistinguishable from the response alone" is strictly worse than making them distinguishable for free.
- **Extend delivery retention specifically to keep `deliveryStatus` answerable for longer, or snapshot the terminal status onto `AssistantTurn` before the row is purged.** Both rejected as retention-policy or schema changes — explicitly out of scope for this phase (hard non-goal: no retention redesign, no duplicate status snapshot without inspection proving it necessary). Nothing in this phase's investigation found evidence that 7/28-day retention is actually too short for any real product need; if that need appears, it is a separate, explicit decision with its own trade-off analysis (see Follow-up).
- **Rendering a subtle "history unavailable" icon for `UNKNOWN`.** Considered, rejected: no user research or product ask motivates it, it adds a fourth visible delivery-state icon for a case with zero actionability, and the existing Phase 30/31 convention throughout this whole feature has been "render nothing for the uninteresting/default case" — introducing one exception here would be inconsistent with that convention for no benefit.

## Consequences

**Positive:**

- Closes a real, structural API ambiguity: a client (or a future maintainer reading raw JSON) can now tell "no Phase 31 support" (field absent) apart from "Phase 31/32 supported it, but this specific row aged out" (`'UNKNOWN'`), which pure documentation of Phase 31's behavior could not achieve.
- Zero required frontend behavior change — the existing default-safe rendering switch already handles the new value correctly, so Phase 31's shipped UI needed no code change to remain correct (only its type definition was widened, for accuracy).
- No schema change, no retention policy change, no new field — the smallest change that actually closes the ambiguity.
- `resolveDeliveryStatus` is now directly unit-testable in isolation (mirroring `DELIVERY_STATUS_MAP`), and the DB-integration suite gained an explicit purge-and-reread test proving the exact before/after transition (`DELIVERED → UNKNOWN` once the row is deleted), not just a "no row was ever created" case.

**Costs:**

- None beyond the diff itself: `AssistantDeliveryStatus` widening is additive to both backend and frontend types, and no existing test asserting omission needed anything other than updating its expected value to `'UNKNOWN'`.

## Related documents

- [PD-021 — Channel Delivery Receipts & User-Facing Send State](021-channel-delivery-receipts.md)
- [PD-020 — Channel Source Attribution & Cross-Channel UX](020-channel-source-attribution.md)
- [PD-019 — Safe Operator Remediation](019-safe-operator-remediation.md)
- [Assistant Core Architecture §15.10](../../architecture/assistant-core-architecture.md#1510-channel-delivery-retention--historical-ux-semantics-phase-32)
- [Implementation Roadmap — Phase 32](../../development/implementation-roadmap.md)
- [Telegram Deployment Runbook §8.1](../../development/telegram-deployment-runbook.md#81-raw-sql-equivalent-for-when-a-database-client-is-already-open)

## Non-goals / explicitly out of scope

- No retention policy redesign — `CHANNEL_RETENTION_DAYS`, `TERMINAL_RETENTION_MULTIPLIER`, and `src/channels/retention.ts`'s deletion logic are unchanged.
- No duplicate delivery-status snapshot stored anywhere (e.g. on `AssistantTurn` or `ChannelAssistantOperation`) — `deliveryStatus` remains a pure read-time computation.
- No Telegram `externalChatId`/`externalUserId`/`externalSenderId` exposure, no `providerMessageId`/`destinationChatId`/reply markup/raw provider payload exposure, through any DTO.
- No user-triggered resend/retry action — Phase 29's operator remediation remains the only remediation path for an actual `FAILED` delivery; `UNKNOWN` is not a failure and has no remediation action at all.
- No automatic replay or remediation of any kind.
- No queue redesign, no admin/operator UI or endpoint, no new channel provider, no broad Assistant UI redesign.
- No new visible icon/copy for `UNKNOWN` — it renders identically to the already-established "nothing" case.

## Follow-up Tasks

- If a genuine product need for a longer delivery-status-visible window emerges, it should be evaluated as its own decision weighing storage/PII-retention cost against the UX benefit — this phase found no such need during inspection and deliberately does not propose one.
- If a future consumer other than the Web Assistant UI needs to distinguish "purged" from "never created" specifically (this phase's `UNKNOWN` intentionally does not), that would need its own additive value or field — no such consumer exists today.
