# Product Decision Approval Workflow

## Purpose

Product Decision approval changes provisional product direction into authoritative policy. The Decision document and every affected lower-authority document must be synchronized so that product rules, canonical terminology, architecture, specifications, repository guidance, and future AI work describe the same approved behavior.

Synchronization prevents Draft language from remaining in dependent documents, prevents implementation evidence from becoming product authority, and keeps each approval reviewable as one coherent documentation change. Approval does not authorize implementation. Implementation planning and implementation changes remain separate work.

---

## Approval Flow

```text
Product Decision Review

↓

Approval

↓

Decision Document Update

↓

RFC Update

↓

Glossary Update

↓

Architecture Update

↓

Design System Update

↓

Screen Specification Update

↓

Component Specification Update

↓

AI Project Context Update

↓

Repository Audit

↓

Commit
```

The flow is sequential because each document must remain consistent with the higher-authority documents above it. A document is updated only when the approved decision affects its responsibility. An unaffected step is reviewed and recorded as not applicable rather than changed for completeness.

---

## Synchronization Rules

- Confirm approval from the individual Product Decision document. Do not infer approval from a recommendation, aggregate status, reconciliation finding, implementation, or completed checkbox.
- Update the Decision document first. Set its status and approval metadata, replace provisional Final Decision wording, append its History, and update documentation-impact checkboxes only for documents actually reconciled.
- Update only documents whose product rules, terminology, architecture, design direction, screen behavior, component behavior, or AI guidance are affected by the approved decision.
- Never update unrelated documents, Product Decisions, sections, terminology, formatting, or examples.
- Preserve the documentation authority order. Lower-authority documents translate approved policy; they do not expand, weaken, or reinterpret it.
- Use canonical Glossary terminology. Introduce or promote terms only when the approved Product Decision authorizes them.
- Remove Draft or Provisional wording governed solely by the newly approved decision. Preserve provisional wording governed by other Draft Product Decisions.
- Never change implementation as part of Product Decision documentation synchronization.
- Never invent routes, contracts, schemas, fields, states, components, calculations, or implementation mechanisms that the approved decision does not define.
- Never rewrite historical reconciliation reports. Preserve their original findings and append or supersede them transparently only when the task explicitly includes that historical update.
- Preserve unrelated working-tree changes. Stop if an intended edit would overwrite or ambiguously merge unrelated work.
- Report conflicts between the approved decision and higher-authority documentation before continuing. Do not silently resolve a conflict by changing product policy.

---

## Verification Checklist

- [ ] The individual Product Decision frontmatter status is `Approved`.
- [ ] The Product Decision updated date is current.
- [ ] Final Decision status, Decision, Approved By, Approved On, and Rationale are populated.
- [ ] An approval row is appended to the Product Decision History.
- [ ] Documentation-impact checkboxes are complete only for documents updated in this synchronization.
- [ ] The Product RFC is synchronized where affected.
- [ ] The Canonical Glossary is synchronized where affected.
- [ ] PD-governed Draft and Provisional markers are removed where approval makes them obsolete.
- [ ] System Architecture is synchronized where affected.
- [ ] Design System is synchronized where affected.
- [ ] Screen Specification is synchronized where affected.
- [ ] Component Specification is synchronized where affected.
- [ ] AI Project Context moves the decision from Provisional Areas to approved Canonical Product Rules where applicable.
- [ ] No unrelated Product Decision or document changed.
- [ ] No implementation file changed.
- [ ] Historical reconciliation findings were preserved.
- [ ] Canonical terminology is consistent across every updated document.
- [ ] Higher- and lower-authority documents do not contradict the approved decision.
- [ ] No updated document still describes the approved decision as Draft or Provisional.
- [ ] Relative links and referenced headings resolve.
- [ ] Markdown headings, lists, tables, frontmatter, and checkboxes remain valid.
- [ ] The final diff contains only the intended synchronization changes.
- [ ] Final working-tree status is inspected and unrelated changes are identified without being modified.
- [ ] Required document checks pass, or every unavailable or failed check is reported accurately.

---

## Commit Rules

```text
One Product Decision

↓

One synchronization

↓

One commit
```

- A synchronization commit contains one approved Product Decision and only its affected documentation.
- Do not combine multiple Product Decision approvals, implementation changes, migrations, generated files, local configuration, or unrelated cleanup.
- Commit only after the repository audit and verification checklist are complete.
- Name the commit for the approved outcome, not for the editing activity.
- If verification fails, do not commit until the failure is resolved or the synchronization is explicitly stopped and reported.
- If commit authorization is not part of the task, prepare the verified synchronization and report the commit that remains to be created.

---

## Future Automation

Future AI agents must execute this workflow as an authority-preserving documentation task:

1. Read AI Project Context, repository-local instructions, the individual Product Decision, and the minimum higher-authority documents required by its scope.
2. Confirm the individual Product Decision is explicitly approved and record its current status before changing dependent documentation.
3. Inspect working-tree status and preserve all unrelated changes.
4. Derive the affected-document set from the approved decision's scope and documentation impact. Do not assume every document in the approval flow requires modification.
5. Update the Decision document, then reconcile affected documents in authority order.
6. Keep policy synchronization separate from implementation, implementation planning, and historical reconciliation work unless those tasks are separately authorized.
7. Search every updated document for stale Draft or Provisional references, contradictory wording, non-canonical terminology, and copied policy that diverges from the Decision.
8. Validate relative links, referenced headings, Markdown structure, and the final diff.
9. Report files inspected, files modified, checks run, results, assumptions, existing working-tree changes, unresolved conflicts, and remaining risks.
10. Create one synchronization commit only after all required checks pass and the task authorizes committing.

Automation must stop and request direction when approval is ambiguous, higher-authority documents conflict, the approved scope is incomplete, another Draft Product Decision controls a required choice, or safe synchronization would overwrite unrelated work.
