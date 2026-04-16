---
name: resume
description: Resume work on a project or feature by loading its living tracker from Sombra. Shows progress and highlights what's next. Adapts to context — uses git awareness in code environments, plain status elsewhere. Use when starting a session to continue previous work, or "where was I", "resume", "what's next".
when_to_use: "resume, where was I, continue, load project, pick up where I left off, what's next, what's remaining"
argument-hint: "[project name, feature name, or branch — defaults to current branch if in a git repo]"
allowed-tools:

  - mcp__claude_ai_Sombra__browse_collections
  - mcp__claude_ai_Sombra__read_collection
  - mcp__claude_ai_Sombra__read_collection_context
  - mcp__claude_ai_Sombra__search_artifacts
  - mcp__claude_ai_Sombra__fetch_artifact
  - mcp__claude_ai_Sombra__replace_context_section
  - Bash(git branch --show-current)
  - Bash(git log *)
  - Bash(git diff *)
  - Bash(git remote get-url origin)
  - Read
  - Grep
  - Glob
---

# Resume — Sombra

You reconnect a session to its living project or spec stored in Sombra.

## Skill boundaries

- **This skill** is for loading an existing project/spec and orienting the session.
- If no project is found, suggest `/sombra:project` (general) or `/sombra:spec` (code).
- Once resumed, continue as [project](../project/SKILL.md) or [spec](../spec/SKILL.md) would — update the tracker as work progresses, log decisions, evolve the plan.
- If the user wants to **research an API or library**, switch to [research](../research/SKILL.md).

## Getting started

Jump straight into the workflow below. If a Sombra tool call fails with an auth or connection error, guide the user to check `/mcp` for their connection status, or visit **sombra.so/mcp** for setup instructions.

## Workflow

### 1. Identify the project

Determine what to search for:
- If `$ARGUMENTS` is provided, search for that
- If in a git repo (Bash tools available), try `git branch --show-current` as a search term
- Otherwise, ask the user what project they want to resume

Search Sombra:
- `search_artifacts` for the project/branch name (appears in `## Meta`)
- `browse_collections` and scan names for a match

If multiple matches, read each collection's context and check Meta. Ask the user if ambiguous.

If nothing found, suggest creating one with `/sombra:project` or `/sombra:spec`.

### 2. Load the project

Read the collection context with `read_collection_context`. This is the living plan.

Parse `## Meta` to understand what kind of project this is:
- **Has Branch/Repo fields** → this is a code spec. Use git-aware behaviour.
- **No git fields** → this is a general project. Skip git steps.

### 3. Assess what's changed

**If code spec (git available):**

```
git log --oneline -20
```

Compare commits against open items. Look for:
- Items that appear completed based on commit messages or changed files
- New files or paths not yet referenced in the spec
- Work that doesn't map to any spec item

**If general project:**

Skip this step. Present the current state and ask the user what's changed since last session.

### 4. Report to the user

Present a concise status:

**Code spec format:**
```
## Resuming: Auth Rework
Collection: Auth Rework | Branch: dan/auth-rework

### Progress
- [x] AR-1 — Strip old middleware
- [x] AR-2 — Implement PKCE flow
- [ ] **AR-3 — Token refresh** → `src/mnp/auth/token.clj`
- [ ] AR-4 — Rate limiting

### Since last session
- 3 commits: added PKCE handler, updated tests, removed old session code
- AR-2 looks complete based on commit "implement PKCE auth flow"

### Suggested next step
AR-3: Implement token refresh handling

### Open questions
- How do we handle token revocation across devices?
```

**General project format:**
```
## Resuming: House Purchase — 14 Elm Street
Collection: House Purchase — 14 Elm Street

### Progress
- [x] HP-1 — Submit offer
- [x] HP-2 — Offer accepted
- [ ] **HP-3 — Instruct solicitor** → Brennan & Co
- [ ] HP-4 — Commission survey

### What's next
HP-3: Instruct solicitor. Ref: BC/2026/1234

### Open questions
- Should we negotiate on the boiler?

Anything changed since we last talked about this?
```

Highlight the **next actionable item**. For code specs, if commits suggest items are done, ask the user to confirm before updating.

### 5. Stay connected

After resuming, the project is loaded in context. Continue as [project](../project/SKILL.md) or [spec](../spec/SKILL.md) would.

## Arguments

Project name, feature name, or branch (optional — defaults to current branch if in a git repo): $ARGUMENTS
