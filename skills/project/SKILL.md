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

## Updating an existing project

1. **Read first.** Always `read_collection_context` before making changes.
2. **Use surgical updates.** `replace_context_section` for modifying a specific section. Never rewrite the whole document when a section update will do.
3. **Check off completed items.** Replace the phase section with updated checkboxes.
4. **Log decisions.** When plans change, update the relevant phase AND append to the Decisions section explaining what changed and why.
5. **Add new phases.** Use `replace_context_section` with `create_if_missing: true`.
6. **Update the Meta section.** Set the `Updated` date and increment `Next` when adding items.

Before every update, run through the pre-flight checklist in [format.md](references/format.md).

## At natural breakpoints

When the user reports progress or you've helped them complete something, proactively offer to update the project:

> "Sounds like HP-3 and HP-4 are done. Want me to check those off and note any decisions?"

If the user confirms, update. If they add context ("actually the solicitor flagged an issue with the title"), capture that in Decisions too.

## Evolving the project

Plans change. When they do:
- Update the phase items to reflect reality
- Add new items with the next available code
- Never delete items — mark cancelled ones with ~~strikethrough~~ and note why in Decisions
- If a phase is completely reworked, create a new phase heading and note the supersession

## Arguments

Project name or existing collection name: $ARGUMENTS
