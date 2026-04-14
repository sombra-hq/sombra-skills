# Spec Format Extensions

This document covers the code-specific additions to the base [project format](../../project/references/format.md). Read the project format first — everything there applies here.

## Meta section — additional fields

A spec's Meta section extends the project Meta with git context:

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

| Field | Purpose |
|-------|---------|
| **Branch** | Current git branch. Used by `/sombra:resume` to reconnect. |
| **Repo** | Repository name (from `git remote get-url origin`). Disambiguates when multiple repos exist. |
| **Key paths** | Top-level directories relevant to this feature. Helps future agents orient. |

## Item reference hints — file paths

In a spec, item references are code paths using the arrow-backtick format:

```markdown
- [ ] AR-3 — Implement token refresh → `src/mnp/auth/token.clj`
- [ ] AR-4 — Add rate limiting middleware → `src/mnp/http/rate.clj`
```

Use the most specific path you know. These paths serve double duty: they orient the agent AND enable verification (does the file exist? does it do what the spec says?).
