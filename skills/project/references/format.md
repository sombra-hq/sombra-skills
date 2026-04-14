# Project Format Reference

This document defines the exact format for living project documents. Load this when creating or updating a project.

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

## HP: Offer & Legal
- [x] HP-1 — Submit offer
- [x] HP-2 — Offer accepted — 338k
- [ ] HP-3 — Instruct solicitor → Brennan & Co, ref: BC/2026/1234
- [ ] HP-4 — Commission survey → book via HomeBuyer, call 0800 123 456

## HP: Mortgage
- [ ] HP-5 — Submit full application → Halifax, appt 2026-04-20

## Decisions
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
- **Codes are permanent.** Never renumber or reassign codes, even if items are moved between phases or removed. This ensures references across conversations remain stable.
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

Use whatever reference helps the most when picking this item up in a future session.

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

Append-only. Each entry has a date and a reason:

```markdown
## Decisions
- 2026-04-10 — Chose fixed rate over tracker. Stability preferred.
- 2026-04-15 — Dropped HP-3, replaced with HP-6. Changed solicitor after poor response time.
```

When an Open Question is resolved, move it here with the resolution.

## Open Questions section

Questions that need answers before work can proceed or that represent unresolved decisions:

```markdown
## Open Questions
- Should we negotiate on the boiler?
- Is the access road adopted or private?
```

When resolved, move to Decisions with the answer.

## Pre-flight checklist

Before any project update, verify:

- [ ] Heading uses `## ` format (not bold, not `###`)
- [ ] Heading text is unique in the document
- [ ] Item codes follow `{PREFIX}-{NUMBER}` pattern (monotonic)
- [ ] New items use the `Next` counter from Meta, then increment it
- [ ] Checkboxes use `- [ ]` / `- [x]` (GHFM)
- [ ] Cancelled items use ~~strikethrough~~, not deletion
- [ ] Decisions include dates
- [ ] Updated date in Meta section reflects the change
