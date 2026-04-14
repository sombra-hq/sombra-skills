---
name: spec
description: Create and manage a living spec for a software feature, tracked in Sombra across conversations. Extends the project skill with git branch context, code paths, and commit-based progress detection. Use when the user is building a feature, planning a migration, or says "spec this out", "track this feature", "create a living spec".
when_to_use: "create spec, living spec, track feature, spec this out, plan feature, update spec, feature tracker"
argument-hint: "[feature name or collection name]"
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
  - Bash(git branch --show-current)
  - Bash(git log *)
  - Bash(git remote get-url origin)
  - Read
  - Grep
  - Glob
---

# Living Spec — Sombra

A living spec is a [living project](../project/SKILL.md) specialised for software development. Follow the project skill for the core workflow — this skill adds code-specific behaviour.

Load [format.md](references/format.md) for the code-specific format extensions. Load the project's [format.md](../project/references/format.md) for the base format, and [errors.md](../project/references/errors.md) when a tool call fails.

## Skill boundaries

- **This skill** is for software features with git branches and code paths.
- For **non-code projects** (renovation, content strategy, etc.), use [project](../project/SKILL.md) instead.
- If the user wants to **resume** an existing spec, switch to [resume](../resume/SKILL.md).
- If the user wants to **research an API or library**, switch to [research](../research/SKILL.md).

## What spec adds to project

### 1. Git context in Meta

When initializing, populate the Meta section with repo information:

```markdown
## Meta
- **Prefix**: AR
- **Project**: Auth Rework
- **Branch**: dan/auth-rework
- **Repo**: sombra/core
- **Key paths**: src/mnp/auth/, src/sfe/views/auth/
- **Created**: 2026-04-14
- **Updated**: 2026-04-14
- **Next**: 5
```

Run `git branch --show-current` and `git remote get-url origin` to get these values.

### 2. File path hints on items

Every item should reference the relevant code path:

```markdown
- [ ] AR-3 — Implement token refresh → `src/mnp/auth/token.clj`
- [ ] AR-4 — Rate limiting middleware → `src/mnp/http/rate.clj`
```

Use the most specific path you know. A file is better than a directory, but a directory is better than nothing.

### 3. Commit-based progress detection

At natural breakpoints — after finishing a task, before a commit, when the user pauses — check the git log for evidence of completed items:

```
git log --oneline -10
```

Compare commit messages and changed files against open spec items. If something looks done, propose checking it off:

> "Based on the last 3 commits, AR-1 and AR-2 look complete. Want me to check those off?"

### 4. Code verification

When resuming or auditing a spec, you can verify items by checking whether the referenced files exist and contain the expected functionality:

- Use `Glob` to check if referenced paths exist
- Use `Grep` to search for key functions or patterns
- Use `Read` to verify implementation details

This helps catch specs that have drifted from reality.

## Everything else

For initializing, updating, decisions, open questions, and evolving the spec — follow [project](../project/SKILL.md). The workflow is identical; spec just adds the code-specific layers above.

## Arguments

Feature name or existing collection name: $ARGUMENTS
