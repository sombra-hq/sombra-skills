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
  - mcp__claude_ai_Sombra__fetch_artifact
  - mcp__claude_ai_Sombra__search_artifacts
  - Read
---

# Living Project — Sombra

You manage **living projects**: structured trackers that evolve across multiple conversations. A living project is a Sombra collection whose **context document** is the canonical plan.

Load [format.md](references/format.md) for the exact format, heading rules, item codes, and pre-flight checklist. Load [errors.md](references/errors.md) when a tool call fails.

## Skill boundaries

- **This skill** is for creating, updating, and managing any project.
- For **software features** with git branches and code paths, use [spec](../spec/SKILL.md) instead — it extends this skill with code-specific context.
- If the user wants to **resume** an existing project (e.g., "where was I"), switch to [resume](../resume/SKILL.md).
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

## Supporting artifacts for complex items

The tracker context must stay concise — checklist items with status, one-line descriptions, and reference hints. But some items carry detail that won't survive compression into a single line. When that happens, create a **supporting artifact** with the full spec and reference it from the tracker.

### When an item needs its own artifact

Create one when the item has any of:

- Multiple coordinated steps that must happen in sequence
- Specific requirements, criteria, or constraints that shape how it's done
- Reference material — contacts, quotes, comparisons, examples, or measurements
- Dependencies on other items or external events

The test: **could someone pick this item up cold from just the checklist line?** If not, the missing detail belongs in a supporting artifact.

### How

1. **Create the artifact** in the same collection. Name it `{PREFIX}-{N}: {descriptive title}` (e.g., "HP-7: Repair Negotiation Strategy").
2. **Write the full spec** as the artifact body — all the detail that won't fit in the tracker line.
3. **Reference it from the tracker** using the `→ spec artifact` pattern:

   ```markdown
   - [ ] HP-7 — Negotiate post-survey repairs → detail: "HP-7: Repair Negotiation Strategy"
   - [ ] HP-12 — Insurance comparison → detail: "HP-12: Insurance Requirements"
   ```

4. **Do this when the item is added**, not as a later cleanup step. The detail is freshest at that point.

When the item's requirements change, update the artifact. The tracker line stays stable; the artifact holds the evolving detail.

## Updating an existing project

1. **Read first.** Always `read_collection_context` before making changes.
2. **Use surgical updates.** `replace_context_section` for modifying a specific section. Never rewrite the whole document when a section update will do.
3. **Check off completed items.** Replace the phase section with updated checkboxes.
4. **Log decisions.** When plans change, update the relevant phase AND append to the Decisions section explaining what changed and why.
5. **Add new phases.** Use `replace_context_section` with `create_if_missing: true`.
6. **Update the Meta section.** Set the `Updated` date and increment `Next` when adding items.

Before every update, run through the pre-flight checklist in [format.md](references/format.md).

## Keeping the tracker current

Update the tracker as work happens — don't wait to be asked.

### Triggers

Act on these signals immediately:

- **Completion reported** — the user says something is done, finished, sorted, or handled → mark the item(s) complete
- **Plan changed** — the user describes a new direction, shifted priority, or dropped item → update the phase and log a decision
- **New detail surfaced** — the user shares information that affects an open item (a quote, a date, a contact, a constraint) → update the item's reference hint, or create a supporting artifact if the detail is substantial
- **Batch of work finished** — after helping the user work through several items in a conversation → update all affected items at once

### How

1. `read_collection_context` to get current state
2. `replace_context_section` for affected phase(s), `append_to_context` for decisions
3. Update Meta (`Updated` date, `Next` counter if items were added)
4. Tell the user what you changed: "Marked HP-3 and HP-4 complete. Logged the solicitor decision."

Update first, report after. The user can correct if something's wrong — but the default is action, not asking permission.

## Evolving the project

Plans change. When they do:
- Update the phase items to reflect reality
- Add new items with the next available code
- Never delete items — mark cancelled ones with ~~strikethrough~~ and note why in Decisions
- If a phase is completely reworked, create a new phase heading and note the supersession

## Arguments

Project name or existing collection name: $ARGUMENTS
