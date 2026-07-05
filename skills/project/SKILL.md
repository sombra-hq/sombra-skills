---
name: project
description: Create and manage a living project tracker in Sombra that persists across conversations. Use for any multi-phase effort — house purchase, renovation, content strategy, product launch, event planning, learning goals, or anything with phases and evolving plans. Works in Claude Desktop, Claude.ai, ChatGPT, Claude Code, and any MCP client.
when_to_use: "create project, track project, living doc, plan project, manage project, project tracker, track progress"
argument-hint: "[project name or collection name]"
allowed-tools:

  - mcp__claude_ai_Sombra__browse_collections
  - mcp__claude_ai_Sombra__create_collection
  - mcp__claude_ai_Sombra__read_collection
  - mcp__claude_ai_Sombra__read_collection_context
  - mcp__claude_ai_Sombra__write_collection_context
  - mcp__claude_ai_Sombra__replace_context_section
  - mcp__claude_ai_Sombra__append_to_context
  - mcp__claude_ai_Sombra__create_artifact
  - mcp__claude_ai_Sombra__update_artifact
  - mcp__claude_ai_Sombra__append_to_artifact
  - mcp__claude_ai_Sombra__fetch_artifact
  - mcp__claude_ai_Sombra__search_artifacts
  - Read
---

# Living Project — Sombra

You manage **living projects**: structured trackers that evolve across multiple conversations. A living project is a Sombra collection whose **context document** is the canonical plan.

The context document is an **index, not a journal**. It's read in full every session over MCP, where oversized documents get truncated — so it stays lean (checklist, pointers, recent decisions), and everything with bulk lives in **linked artifacts** in the same collection: item detail, reports, the full decision history, archived phases.

Load [format.md](references/format.md) for the exact format, heading rules, item codes, and pre-flight checklist. Load [errors.md](references/errors.md) when a tool call fails.

## Skill boundaries

- **This skill** is for creating, updating, and managing any project.
- For **software features** with git branches and code paths, use [spec](../spec/SKILL.md) instead — it extends this skill with code-specific context.
- If the user wants to **pick up** an existing project (e.g., "where was I"), switch to [pickup](../pickup/SKILL.md).
- If the user wants to **research a topic**, switch to [research](../research/SKILL.md).

## Getting started

Jump straight into the workflow below. Your first Sombra tool call (typically `browse_collections` or `create_collection`) will confirm the connection is live.

If a Sombra tool call fails with an auth or connection error, guide the user:

> Sombra doesn't seem to be connected. Check `/mcp` to see if "Sombra" appears in your server list.
>
> If it's not there, head to **sombra.so/mcp** for setup instructions.
> If it shows "connected" but tools aren't working, try selecting it in `/mcp` to re-authenticate.

## Initializing a new project

1. **Discuss scope.** Ask the user what they're working on. Understand the goal, constraints, timeline, and rough phases before writing anything.
2. **Check Sombra.** Search for existing collections that might cover this project. Don't create duplicates.
3. **Create the collection.** Name it clearly after the project (e.g., "House Purchase — 14 Elm Street", "Q2 Content Strategy", "Kitchen Renovation").
4. **Generate the prefix.** 2-3 uppercase letters from the project name (e.g., "House Purchase" → `HP`). Items get monotonically increasing numbers: `HP-1`, `HP-2`, `HP-3`. Track the next number in the Meta section. See [format.md](references/format.md) for the full format.
5. **Write the context document.** Follow the structure in [format.md](references/format.md) exactly. Every section must use a unique `##` heading.

## Linked artifacts carry the detail

Tracker lines are one line each — status, description, reference hint. Anything that doesn't fit on one line goes into an artifact in the same collection. This is the default pattern, not an exception. See "Linked artifacts" in [format.md](references/format.md) for naming conventions.

### Item detail artifacts

Create one when an item has multiple coordinated steps, specific requirements or constraints, reference material (contacts, quotes, comparisons, measurements), or dependencies on other items. The test: **could someone pick this item up cold from just the checklist line?** If not, the missing detail belongs in an artifact.

1. **Create the artifact** in the same collection, named `{PREFIX}-{N}: {descriptive title}` (e.g., "HP-7: Repair Negotiation Strategy").
2. **Write the full detail** as the artifact body.
3. **Reference it from the tracker**:

   ```markdown
   - [ ] HP-7 — Negotiate post-survey repairs → detail: "HP-7: Repair Negotiation Strategy"
   ```

4. **Do this when the item is added**, not as a later cleanup step. The detail is freshest at that point.

When the item's requirements change, update the artifact. The tracker line stays stable; the artifact holds the evolving detail.

### Reports, analyses, and notes

When work produces substantial output — a comparison report, research findings, an analysis, meeting notes — create an artifact named `{PREFIX}: {title}` and link it from the relevant item or decision. **Never paste multi-paragraph content into the context document.**

### Decision history

The context document keeps only the 5 most recent decisions. Older entries move to the `{PREFIX}: Decision Log` artifact via `append_to_artifact` — the exact flow is in [format.md](references/format.md), "Decisions section".

## Updating an existing project

1. **Read first.** Always `read_collection_context` before making changes.
2. **Use surgical updates.** `replace_context_section` for modifying a specific section. Never rewrite the whole document when a section update will do.
3. **Check off completed items.** Replace the phase section with updated checkboxes.
4. **Log decisions.** When plans change, update the relevant phase AND record a decision (one line, dated) following the Decisions flow in [format.md](references/format.md) — inline list capped at 5, overflow to the Decision Log artifact.
5. **Add new phases.** Use `replace_context_section` with `create_if_missing: true`.
6. **Update the Meta section.** Set the `Updated` date and increment `Next` when adding items.
7. **Keep the document under budget.** When a phase finishes, archive it to a stub. If the document is drifting past ~150 lines, run the compaction pass from [format.md](references/format.md).

Before every update, run through the pre-flight checklist in [format.md](references/format.md).

## Keeping the tracker current

Update the tracker as work happens — don't wait to be asked.

### Triggers

Act on these signals immediately:

- **Completion reported** — the user says something is done, finished, sorted, or handled → mark the item(s) complete; if that finishes a phase, archive it
- **Plan changed** — the user describes a new direction, shifted priority, or dropped item → update the phase and log a decision
- **New detail surfaced** — the user shares information that affects an open item (a quote, a date, a contact, a constraint) → update the item's reference hint, or create a detail artifact if it won't fit on one line
- **Substantial output produced** — a report, analysis, or set of findings came out of the session → create a `{PREFIX}: {title}` artifact and link it; don't paste it into the context document
- **Batch of work finished** — after helping the user work through several items in a conversation → update all affected items at once

### How

1. `read_collection_context` to get current state
2. `replace_context_section` for affected phase(s) and the Decisions section; `create_artifact` / `append_to_artifact` for detail, reports, and decision-log overflow
3. Update Meta (`Updated` date, `Next` counter if items were added)
4. If the document is over budget, run the compaction pass from [format.md](references/format.md)
5. Tell the user what you changed: "Marked HP-3 and HP-4 complete. Logged the solicitor decision."

Update first, report after. The user can correct if something's wrong — but the default is action, not asking permission.

## Evolving the project

Plans change. When they do:
- Update the phase items to reflect reality
- Add new items with the next available code
- Never delete items — mark cancelled ones with ~~strikethrough~~ and note why in Decisions
- If a phase is completely reworked, create a new phase heading and note the supersession
- When a phase fully completes, archive it to a one-line stub (see [format.md](references/format.md), "Archiving completed phases")

Evolution adds lines; archiving and compaction reclaim them. A project that runs for months should still have a context document that reads in one screen.

## Arguments

Project name or existing collection name: $ARGUMENTS
