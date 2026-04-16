---
name: update
description: Force-sync a Sombra project or spec tracker with current state. Reads the tracker, compares against git history, conversation context, and user-provided changes, then marks items complete, adds new items, logs decisions, and creates supporting artifacts as needed.
when_to_use: "update sombra, sync tracker, update project, sync project, mark items done, log decision, tracker update, update spec"
argument-hint: "[what changed — e.g., 'batch 4 done', 'priorities shifted, X is now P0']"
allowed-tools:

  - mcp__claude_ai_Sombra__browse_collections
  - mcp__claude_ai_Sombra__read_collection
  - mcp__claude_ai_Sombra__read_collection_context
  - mcp__claude_ai_Sombra__write_collection_context
  - mcp__claude_ai_Sombra__replace_context_section
  - mcp__claude_ai_Sombra__append_to_context
  - mcp__claude_ai_Sombra__create_artifact
  - mcp__claude_ai_Sombra__update_artifact
  - mcp__claude_ai_Sombra__fetch_artifact
  - mcp__claude_ai_Sombra__search_artifacts
  - Bash(git branch --show-current)
  - Bash(git log *)
  - Bash(git remote get-url origin)
  - Read
  - Grep
  - Glob
---

# Tracker Update — Sombra

Force-sync a living project or spec tracker with current reality. Use this when you have new information to push into the tracker — items completed, priorities changed, decisions made, new work identified.

For the tracker format and rules, load the project [format.md](../project/references/format.md). For code-specific extensions, load the spec [format.md](../spec/references/format.md). For error handling, load [errors.md](../project/references/errors.md).

## Skill boundaries

- **This skill** is for pushing updates into an existing tracker.
- If no tracker exists, suggest `/sombra:project` (general) or `/sombra:spec` (code).
- If the user wants to **resume** and orient before working, use [resume](../resume/SKILL.md).
- If the user wants to **create a new project**, use [project](../project/SKILL.md) or [spec](../spec/SKILL.md).

## Workflow

### 1. Find the tracker

- If `$ARGUMENTS` names a project → search for it directly
- If a project is already loaded in this conversation → use that
- If in a git repo → check if a collection matches the current branch or repo name
- Otherwise → `browse_collections` and ask the user which tracker to update

### 2. Read current state

`read_collection_context` to load the tracker. Parse Meta and understand what's open, what's done, what phases exist.

### 3. Gather signals

Combine all available information to determine what changed:

**From the user** (the `$ARGUMENTS` free-text):
- Items completed, priorities shifted, new items to add, decisions made
- Context from meetings, calls, or external events

**From git** (if in a repo and Meta has Branch/Repo fields):
- `git log --oneline -20` — recent commits that may map to open items
- Compare commit messages and changed files against the tracker's open items

**From the conversation**:
- Work completed earlier in this session
- Decisions made during discussion

### 4. Apply updates

Make all changes in one pass:

1. **Mark completed items** — `replace_context_section` for the affected phase(s) with updated checkboxes
2. **Add new items** — use the next code from Meta's `Next` counter, then increment it
3. **Log decisions** — `append_to_context` to the Decisions section with today's date
4. **Create supporting artifacts** for any new items that cross the complexity threshold (see [project SKILL.md](../project/SKILL.md), "Supporting artifacts for complex items")
5. **Update Meta** — set the `Updated` date, update `Next` if items were added

Run the pre-flight checklist from [format.md](../project/references/format.md) before writing.

### 5. Report

Tell the user exactly what changed — concise, specific, no fluff:

> Updated **Kitchen Renovation** tracker:
> - Marked KR-4 and KR-5 complete
> - Added KR-8: Electrician quote comparison → detail: "KR-8: Electrician Quotes"
> - Logged decision: going with underfloor heating over radiators
> - Next open: KR-6 (plumber), KR-7 (tiling)

## Arguments

What changed (optional free-text): $ARGUMENTS
