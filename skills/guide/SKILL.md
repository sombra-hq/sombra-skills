---
name: guide
description: >
  General-purpose Sombra usage — saving web pages, browsing collections, distilling
  context, and working with your research library. Use when the user mentions research,
  documentation, collections, context, saving, or looking something up, and no specific
  workflow skill (project, spec, resume, research) applies.
when_to_use: "save this, look up, check my collections, browse, distil, save url, what's in sombra, search library"
disable-model-invocation: false
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
  - mcp__claude_ai_Sombra__save_url
  - mcp__claude_ai_Sombra__move_artifact
  - mcp__claude_ai_Sombra__bulk_move_artifacts
  - mcp__claude_ai_Sombra__remove_from_collection
  - mcp__claude_ai_Sombra__delete_artifact
  - mcp__claude_ai_Sombra__delete_collection
  - mcp__claude_ai_Sombra__restore_artifact
  - mcp__claude_ai_Sombra__list_inbox
  - mcp__claude_ai_Sombra__list_archived
  - mcp__claude_ai_Sombra__update_collection
  - Read
---

# Sombra — Reader Mode for AI

Sombra gives you a persistent research library. Web pages saved as clean markdown, organised into collections, distilled into dense context. Everything persists between sessions.

## Skill boundaries

This skill covers **general Sombra usage** — saving, browsing, searching, distilling. For specific workflows, use the dedicated skills:

- **Track a project** (house purchase, renovation, content strategy) → [project](../project/SKILL.md)
- **Track a software feature** with git context → [spec](../spec/SKILL.md)
- **Resume where you left off** → [resume](../resume/SKILL.md)
- **Deep API/library research** → [research](../research/SKILL.md)

## Core workflow

### 1. Read context before acting

```
browse_collections → find relevant collection → read_collection_context
```

The distilled context is the signal. It's dense, preserves code verbatim, and replaces forty pages of raw docs with the five hundred tokens that matter. Always prefer `read_collection_context` over `read_collection` unless the user specifically asks for full content.

### 2. Save as you go

When you find useful pages or work something out during a task:

```
save_url → move_artifact to relevant collection
```

Or create a note with your findings:

```
create_artifact with a summary of what you learned
```

### 3. Distil when a collection grows

After adding several items to a collection, offer to distil:

```
read_collection → write_collection_context
```

When distilling:
- Preserve all code examples, CLI commands, config snippets, and function signatures **verbatim**
- Compress everything else to dense prose
- Focus on what an agent needs to write correct code: breaking changes, key API differences, auth flows, gotchas

Use the `distill_collection` or `distill_technical_collection` prompts for guided distillation.

## When to use Sombra proactively

- **Before coding against external APIs or libraries** — read the collection context first. It has verified, current docs. Don't rely on training data.
- **When the user mentions a collection by name** — browse or read that collection.
- **When the user asks you to save something** — save URLs or create notes.
- **When the user says "research" or "look up"** — search the library first before going to the web.
- **When starting a new task** — check if there's a relevant collection. `browse_collections` is cheap.

## Tools reference

### Save & Create
- `save_url` — save a web page as clean markdown
- `create_artifact` — create a note with markdown content

### Read & Browse
- `browse_collections` — list all collections (cheap, use often)
- `read_collection` — full content from a collection
- `read_collection_context` — distilled summary (prefer this)
- `fetch_artifact` — single artifact by ID
- `list_inbox` — recent unsorted artifacts
- `search_artifacts` — search across everything
- `list_archived` — find deleted items

### Update & Organise
- `update_artifact` — edit metadata or content (notes only)
- `move_artifact` — move to a collection
- `bulk_move_artifacts` — move multiple at once
- `remove_from_collection` — back to unsorted
- `create_collection` — new collection
- `update_collection` — edit name, description, icon
- `write_collection_context` — write/update distilled context
- `replace_context_section` — surgically update one section of context
- `append_to_context` — add to the end of context

### Delete & Restore
- `delete_artifact` — archive (recoverable)
- `delete_collection` — archive (recoverable)
- `restore_artifact` — recover deleted item

## Rules

- **Context before code.** Always read collection context before generating code against external APIs.
- **Don't hallucinate sources.** If it's not in the library, say so. Don't invent content.
- **Preserve code in distillations.** Code examples, CLI commands, and config must be preserved verbatim when writing collection context. Compress prose, never compress code.
- **One collection per topic.** When creating collections, scope tightly. "Pedestal rate limiting" not "Backend stuff".
- **Save the signal, not the noise.** When saving URLs, prefer authoritative sources: official docs, well-researched blog posts, primary sources. Skip aggregators and SEO content.
