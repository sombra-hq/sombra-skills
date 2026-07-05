# Project Format Reference

This document defines the exact format for living project documents. Load this when creating or updating a project.

## The context document is an index, not a journal

The context document is read in full at the start of every session — usually over MCP, where large tool results get truncated. A truncated tracker is a broken tracker: items silently disappear and the agent works from a partial plan.

So the document has one job: **orientation, not storage**. It holds the checklist, pointers, and just enough context to know what's next. Everything with bulk — reports, analyses, meeting notes, full decision history, completed-phase detail — lives in **linked artifacts** in the same collection.

**Size budget: aim to keep the context document under ~100 lines. If an update would push it past ~150 lines, run the compaction pass (below) before writing.**

What goes where:

| Content | Where |
|---------|-------|
| Checklist items — one line each | Context document |
| Overview — 3–5 lines | Context document |
| Open questions — currently open only | Context document |
| 5 most recent decisions — one line each | Context document |
| Full decision history | `{PREFIX}: Decision Log` artifact |
| Item specs, requirements, acceptance criteria | `{PREFIX}-{N}: {title}` artifact |
| Reports, analyses, research, meeting notes | `{PREFIX}: {title}` artifact |
| Completed-phase item detail | `{PREFIX}: Archive` artifact |

## Structure

A living project is a Sombra collection whose **context document** follows this structure exactly. Every section uses `##` markdown headings — these are the addressable units for `replace_context_section`.

```markdown
## Meta
- **Prefix**: HP
- **Project**: House Purchase — 14 Elm Street
- **Created**: 2026-04-14
- **Updated**: 2026-04-14
- **Next**: 6

## Overview
Purchasing a 3-bed semi. Target completion: June 2026.
Budget: 350k. Mortgage agreed in principle with Halifax.

## HP: Viewings
Complete 2026-04-08 — 3 items (HP-9–HP-11) → archive: "HP: Archive"

## HP: Offer & Legal
- [x] HP-1 — Submit offer
- [x] HP-2 — Offer accepted — 338k
- [ ] HP-3 — Instruct solicitor → Brennan & Co, ref: BC/2026/1234
- [ ] HP-4 — Commission survey → book via HomeBuyer, call 0800 123 456

## HP: Mortgage
- [ ] HP-5 — Submit full application → Halifax, appt 2026-04-20

## Decisions
Full log: "HP: Decision Log"
- 2026-04-10 — Chose fixed rate over tracker. Stability preferred given rate environment.
- 2026-04-12 — Going with homebuyer's report, not full structural. House is 15 years old, no obvious concerns.

## Open Questions
- Should we negotiate on the boiler? Survey will tell us condition.
```

## Heading rules

- **Only `##` headings denote sections.** Never use `###`, bold text, or any other formatting as structural headings. The `replace_context_section` tool matches on `##` headings exclusively.
- **Every heading must be unique** within the document. If `replace_context_section` finds duplicate headings, it will fail. Use the `{PREFIX}: Phase Name` format to guarantee uniqueness.
- **Heading text is the section identifier.** The exact string including the `## ` prefix is what you pass to `replace_context_section`. For example: `heading: "## HP: Offer & Legal"`.

## Item code format

Codes follow the pattern `{PREFIX}-{NUMBER}`:

- **Prefix**: 2-3 uppercase letters derived from the project name. "House Purchase" → `HP`, "Kitchen Renovation" → `KR`, "Content Strategy" → `CS`.
- **Number**: Monotonically increasing across the entire project, regardless of which phase the item belongs to. The next number to assign is tracked in the Meta section under `Next`.
- **Codes are permanent.** Never renumber or reassign codes, even if items are moved between phases, removed, or archived. This ensures references across conversations remain stable.
- **Phase grouping comes from headings, not codes.** Items can move between phases without their code changing.

Examples: `HP-1`, `HP-2`, `HP-3`, `CS-12`

## Item reference hints

Every item should include a relevant reference where applicable — a contact, link, reference number, file path, location, or deadline:

```markdown
- [ ] HP-3 — Instruct solicitor → Brennan & Co, ref: BC/2026/1234
- [ ] HP-5 — Submit mortgage application → Halifax, appt 2026-04-20
- [ ] CS-2 — Draft positioning doc → https://docs.google.com/d/abc123
- [ ] KR-4 — Order worktops → Howdens, 3-week lead time
```

Use whatever reference helps the most when picking this item up in a future session. **Items stay on one line.** The moment an item needs a second line of context, that context belongs in a detail artifact.

## Linked artifacts

Artifacts in the same collection carry everything the tracker line can't. Naming conventions make them discoverable via `search_artifacts`:

| Artifact | Name pattern | Holds |
|----------|--------------|-------|
| Item detail | `{PREFIX}-{N}: {title}` | Full spec, requirements, criteria, reference material for one item |
| Decision Log | `{PREFIX}: Decision Log` | Append-only decision history, oldest first |
| Archive | `{PREFIX}: Archive` | Item lists of completed phases |
| Report / note | `{PREFIX}: {title}` | Standalone reports, analyses, research, meeting notes |

Reference artifacts from the tracker with the arrow pattern:

```markdown
- [ ] HP-7 — Negotiate post-survey repairs → detail: "HP-7: Repair Negotiation Strategy"
- [x] HP-8 — Review survey results → report: "HP: Survey Findings"
```

Create the detail artifact **when the item is added**, not as a later cleanup step — the detail is freshest at that point. When an item's requirements change, update the artifact; the tracker line stays stable.

Never paste a report, analysis, or any multi-paragraph content into the context document. Create an artifact and link it.

## Checkbox conventions (GHFM)

```markdown
- [ ] HP-1 — Open item
- [x] HP-2 — Completed item
- [ ] ~~HP-3 — Cancelled item~~ (see Decisions 2026-04-14)
```

- `- [ ]` — open / in progress
- `- [x]` — completed
- `~~strikethrough~~` — cancelled (never delete items; strike them and note why in Decisions)

## Decisions section

The context document keeps only the **5 most recent decisions**, one line each. The full history lives in the `{PREFIX}: Decision Log` artifact.

```markdown
## Decisions
Full log: "HP: Decision Log"
- 2026-04-10 — Chose fixed rate over tracker. Stability preferred.
- 2026-04-15 — Dropped HP-3, replaced with HP-6. Changed solicitor after poor response time.
```

Rules:

- Each entry is one line: date — what changed and why. If the rationale needs more than a line, put the full reasoning in its own artifact and link it from the entry.
- **Adding a decision**: append the new entry to the inline list. If the list now exceeds 5 entries, move the oldest entries to the Decision Log artifact — create it if missing, `append_to_artifact` the displaced entries in order (keeping the log chronological, oldest first), then rewrite the section with the 5 most recent.
- Once the Decision Log artifact exists, the pointer line `Full log: "{PREFIX}: Decision Log"` is the first line of the section.
- When an Open Question is resolved, record the resolution as a decision and remove the question.

## Open Questions section

Questions that need answers before work can proceed or that represent unresolved decisions:

```markdown
## Open Questions
- Should we negotiate on the boiler?
- Is the access road adopted or private?
```

Only **currently open** questions live here. When resolved, record the answer as a decision and delete the question from this section.

## Archiving completed phases

When every item in a phase is complete or cancelled, collapse the phase to a stub:

1. Create the `{PREFIX}: Archive` artifact if it doesn't exist.
2. `append_to_artifact` the phase heading and its full item list (checkboxes, hints, and all).
3. Replace the phase section body with a single summary line:

```markdown
## HP: Offer & Legal
Complete 2026-05-02 — 6 items (HP-1–HP-6) → archive: "HP: Archive"
```

Keep the `##` heading — the phase narrative and code ranges stay visible in the tracker, and the section remains addressable. Codes are permanent even in the archive.

## Compaction pass

Run this when the document exceeds ~150 lines, when a read appears truncated, or before adding a large batch of new items:

1. **Archive completed phases** (above) — usually the biggest win.
2. **Trim Decisions to the 5 most recent** — displaced entries go to the Decision Log artifact.
3. **Extract prose** — any multi-line blob attached to an item or section becomes an artifact, replaced by a `→ detail:` pointer.
4. **Tighten the Overview** to 3–5 lines.
5. Re-read the context and confirm it's complete and under budget.

Compaction never deletes information — it relocates it to artifacts in the same collection. Use `replace_context_section` per section; each replace operates on the full server-side document, so it's safe even when your last *read* came back truncated.

## Pre-flight checklist

Before any project update, verify:

- [ ] Heading uses `## ` format (not bold, not `###`)
- [ ] Heading text is unique in the document
- [ ] Item codes follow `{PREFIX}-{NUMBER}` pattern (monotonic)
- [ ] New items use the `Next` counter from Meta, then increment it
- [ ] Checkboxes use `- [ ]` / `- [x]` (GHFM)
- [ ] Cancelled items use ~~strikethrough~~, not deletion
- [ ] Every item fits on one line — overflow detail goes to a linked artifact
- [ ] Decisions include dates; inline list holds at most 5 entries
- [ ] No multi-paragraph content in the document — reports and notes are artifacts
- [ ] Document stays under budget (~150 lines) — run the compaction pass if not
- [ ] Updated date in Meta section reflects the change
