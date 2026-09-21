# PD-023 — Conversation History Retention & Deletion Semantics (Phase 33)

**Status:** Accepted
**Date:** 2026-09-21
**Author:** Agent (Phase 33 implementation)

## Context

By Phase 32 the Assistant conversation lifecycle already spans archive, restore, delete, a derived `title`, channel source attribution (Phase 30), and delivery status including the historical `UNKNOWN` value (Phase 31/32). Inspection of `pocket-mint-be` (branch `feature/phase-32-channel-delivery-retention-backend`, the current tip of the Assistant phase stack) found that `archiveOwnedConversation`, `restoreOwnedConversation`, and `deleteOwnedConversation` were already implemented in `conversation.service.ts`, already wired through `assistant.controller.ts` and `assistantRoutes.ts`, and already had baseline route- and service-level test coverage — including a code comment on `deleteOwnedConversation` that had already independently corrected the exact false claim this phase's brief warned about (that `Transaction.conversationId` exists and is `SetNull`'d on delete; it does not exist at all — `Transaction` has no relation to `AssistantConversation`).

What was missing was not correctness but explicitness: the interaction between delete/archive and a still-pending financial draft, a still-pending clarification, a channel-linked conversation, and the two channel operational tables (`ChannelInboundJob`, `ChannelAssistantOperation`) had never been exercised by a test or spelled out in `docs/api/assistant-conversations.md`. Two additional facts, true today but never previously documented, surfaced during this verification pass:

1. `AssistantConversationStatus.EXPIRED` is declared in the schema and defensively checked by both `archive` and `restore`, but no code path in the current backend ever assigns it — it is unreachable today, not a currently-active lifecycle state.
2. `ChannelInboundJob.assistantTurnId` and `ChannelAssistantOperation.turnId` are plain string columns with **no** Prisma `@relation` to `AssistantTurn` — confirmed against `prisma/schema.prisma` and against an existing test (`test/channels/durableProcessing.integration.test.ts`) that already sets `turnId: 'turn-already-done'`, a value referencing no real row, without any FK error. Deleting a conversation (which cascades away its turns) therefore leaves these two columns pointing at now-deleted turn ids, with no cascade, restrict, or `SetNull` cleaning them up.

This phase's job is to make all of the above explicit, tested, and documented — not to change behavior that inspection found to already be correct and safe.

## Decision

**Selected: document and test the lifecycle semantics as they already exist. No backend, frontend, or schema behavior changes.**

- **Archive** is confirmed idempotent, preserves every field `GET /conversations/:conversationId` returns (messages, turns, `sourceChannels`, `deliveryStatus`) unchanged, and does not fail or roll back an already-`RUNNING` turn's eventual `finalize` call — only a *new* turn is blocked from the moment `archive` commits, because `finalize`/`finalizeWithoutTool` write directly by `turnId` and never re-check conversation status. This is existing behavior; Phase 33 adds the test (`archive preserves messages, tool history, and channel/delivery metadata`, `does not block finalizing an in-flight RUNNING turn after its conversation is archived mid-request`) and the doc sentence that were both previously missing.
- **Restore** is confirmed idempotent and confirmed to preserve `title`/`sourceChannels` — both are derived at read time from message/turn history, never stored on the conversation row, so archive/restore structurally cannot desynchronize them. Phase 33 extends the existing restore test to assert this instead of adding a new one, since the existing test already covered idempotency and continuation.
- **Delete** is confirmed to cascade a `PENDING_CONFIRMATION` financial draft and a `PENDING` clarification request cleanly (they are simply among the rows the schema already cascades, with no special-cased rejection), to `SetNull` a linked `ChannelConnection.conversationId` rather than blocking or cascading it, and to never reach a `Transaction` — even a `COMMITTED` draft's own row, or an idempotency record's `transactionId` — because `onDelete: Restrict` on those two FKs guards the *Transaction* against deletion while referenced, and deleting the *referencing* draft/idempotency row is an ordinary child-row cascade that never touches the parent. Four new tests cover these four cases directly against a real cascade, not a mock.
- **`EXPIRED` is documented as reserved, not reachable**, rather than left to be independently rediscovered (as the `Transaction.conversationId` claim was, per this phase's own motivating incident) by a future reader who sees the defensive checks in `archive`/`restore` and assumes a real transition exists somewhere.
- **The `ChannelInboundJob`/`ChannelAssistantOperation` gap is documented as intentional-by-omission, not fixed.** Both rows are independently retention-managed (`src/channels/retention.ts`) and never dereferenced back into Assistant history by application code — the dangling id is inert, not a live bug, and adding a `SetNull` relation here would be schema surface added for a value nothing reads. A new test pins this exact behavior (`leaves unenforced ChannelInboundJob.assistantTurnId and ChannelAssistantOperation.turnId pointing at a deleted turn`) so a future schema change that silently adds a relation (and therefore new cascade behavior) shows up as a failing test, not a silent behavior change.
- **The concurrent-delete-of-a-RUNNING-turn gap is documented as an accepted fail-closed limitation**, the same class already accepted for Phase 27's crashed-process idempotency case — not fixed with a lock, since no lock is held across in-flight provider calls anywhere else in this module either, and adding one here alone would be an inconsistent, narrow-scope safety net rather than a considered concurrency design.
- **Frontend:** `messages/en.json` / `messages/id.json` copy for archive/delete/restore (`assistant.history.*`) was inspected and found already accurate against the verified backend semantics — `archiveDescription` already states financial data is unaffected, `deleteDescription` already states deletion is permanent and unrestorable. No frontend copy or component change was made.

## Rejected alternatives

- **Reject delete when a pending draft or clarification exists, requiring the user to resolve it first.** Rejected: nothing in the existing product surface (the delete confirmation dialog, PD-020 through PD-022) ever proposed this, it would be a new user-facing restriction invented by this phase rather than a documentation/test phase, and the underlying cascade is already safe — a pending draft has committed nothing, so deleting it deletes only a preview, never a financial fact.
- **Add a real Prisma relation (with `SetNull`) from `ChannelInboundJob.assistantTurnId`/`ChannelAssistantOperation.turnId` to `AssistantTurn`.** Rejected: this is a schema/migration change with its own blast radius (an index, a constraint, a migration, and a behavior change to two independently-retention-managed tables that currently tolerate a nonexistent-turn id by design, per the existing `durableProcessing.integration.test.ts` fixture) for a value that, per code inspection, is never read back through that relation by any current consumer. Out of scope for a phase whose brief is explicitly "verify and document," not "redesign retention."
- **Implement `AssistantConversationStatus.EXPIRED` as a reachable state (e.g., an auto-expiry job for long-idle conversations).** Rejected as a retention-policy/scope change — explicitly named as a hard non-goal in this phase's brief ("No new retention policy redesign," "No automatic remediation"). Documenting it as reserved-but-unreachable is the correct scope; deciding whether it *should* become reachable is a separate, future product decision.
- **Add application-level locking so a delete cannot race an in-flight `RUNNING` turn.** Rejected: no other Assistant write path in this module holds a lock across an in-flight provider call (Phase 27's own idempotency claim explicitly avoids this for the same reason — a lock held across a multi-second external call is its own reliability risk), so adding one narrowly for delete-during-RUNNING would be inconsistent with the module's established pattern and is not something this phase's evidence showed to be an active incident, only a theoretical gap worth naming.

## Consequences

**Positive:**

- Closes the exact class of risk this phase was commissioned to prevent: a stale or false lifecycle/cascade claim surviving undetected in docs or comments. Two previously-undocumented true facts (`EXPIRED` unreachable; two channel tables un-FK'd to `AssistantTurn`) are now written down where a future implementer will find them before assuming otherwise.
- Seven new backend integration tests pin exact cascade/SetNull/no-op behavior for every row family Phase 33's brief named (drafts, clarifications, idempotency records, channel connection, channel inbound/outbound/operation rows, Transaction) against a real database, not a mock — so a future schema change that alters any of these FK behaviors will fail a test instead of silently changing production semantics.
- Zero behavior change to ship — no migration, no deploy risk, no frontend regression surface, because inspection found the implementation already correct.

**Costs:**

- None beyond the diff itself. The `ChannelInboundJob`/`ChannelAssistantOperation` gap remains a real, if inert, piece of accumulating dead-reference data with no automatic cleanup tied to conversation delete — accepted, not fixed, and now at least visible instead of silent.

## Related documents

- [PD-020 — Channel Source Attribution & Cross-Channel UX](020-channel-source-attribution.md)
- [PD-021 — Channel Delivery Receipts & User-Facing Send State](021-channel-delivery-receipts.md)
- [PD-022 — Channel Delivery Retention & Historical UX Semantics](022-channel-delivery-retention.md)
- [PD-017 — Assistant Request-Level Idempotency & Turn Recovery](017-assistant-request-level-idempotency.md)
- [Assistant Core Architecture §15.11](../../architecture/assistant-core-architecture.md#1511-conversation-history-retention--deletion-semantics-phase-33)
- [Implementation Roadmap — Phase 33](../../development/implementation-roadmap.md)
- `pocket-mint-be`'s `docs/api/assistant-conversations.md` — "Archive, restore, and delete" section, expanded by this phase

## Non-goals / explicitly out of scope

- No new retention policy — no automatic expiry, archival, or deletion job of any kind for conversations, drafts, or channel operational rows.
- No admin/operator UI or endpoint.
- No queue redesign, no automatic remediation.
- No new provider.
- No broad Assistant frontend redesign — copy was inspected, found accurate, and left unchanged.
- No deletion or rewriting of financial transaction history — this phase's own tests exist specifically to prove that path stays closed.
- No schema or migration change — `EXPIRED` stays declared-but-unused, and the two un-FK'd channel columns stay as they are.

## Follow-up Tasks

- If a genuine product need for `AssistantConversationStatus.EXPIRED` emerges (e.g. auto-archiving conversations idle past some threshold), it should be evaluated as its own decision with its own migration and UX review — this phase found no such need during inspection and deliberately does not propose one.
- If a future operator tool ever needs to join `ChannelInboundJob`/`ChannelAssistantOperation` back to `AssistantTurn` for diagnostics, it must handle a missing turn explicitly (the id may reference a deleted turn) rather than assume referential integrity that the schema does not enforce.
