# Research Reference Format

This document defines the structure of a research collection's distilled context. Load it when creating or updating a technical reference.

## Why structure matters

The distilled context is read over MCP, where oversized tool results get truncated, and it's updated with `replace_context_section`, which addresses `##` headings. A structured reference can therefore be built and maintained **one section at a time** — no full-document rewrites, no all-at-once reads.

Heading rules (same as the project format):

- **Only `##` headings denote sections.** Never use `###`, bold text, or other formatting as structure — sub-headings aren't addressable.
- **Every heading must be unique.** When a topic outgrows its section, split it into its own `##` section (`## API: Queries`, `## API: Mutations`) rather than nesting.
- **Heading text is the section identifier** passed to `replace_context_section`, including the `## ` prefix.

## Skeleton

Write this once when the reference is first created (`write_collection_context`), then update sections surgically as sources are distilled:

```markdown
## Meta
- **Topic**: Lacinia GraphQL
- **Version**: 1.2.2
- **Scope**: Schema definition + resolvers for a Pedestal service
- **Updated**: 2026-07-05

## Core Concepts
The mental model — key abstractions and how they relate.

## API Surface
Function signatures, key methods, constructor patterns. Verbatim.

## Configuration
Required config, environment variables, connection strings. Verbatim.

## Patterns
Idiomatic usage examples. Verbatim code.

## Gotchas
Breaking changes, common mistakes, version-specific behaviour.

## Anti-Patterns
Deprecated usage that still appears in training data. What NOT to do.

## Version Notes
Which version this context reflects, and the deltas that matter from adjacent versions.

## Sources
- "Lacinia — Fields and Resolvers" — resolver signatures + context flow (distilled 2026-07-05)
- "Lacinia 1.2 changelog" — breaking changes vs 1.1 (distilled 2026-07-05)
```

Drop sections that don't apply to the technology; add topic sections as the research demands (`## Streaming`, `## Auth Flows`, `## Migration: v2 → v3`) — always as unique `##` headings.

## The Sources ledger

`## Sources` is the **distillation ledger**: one line per saved artifact, in one of two states —

```markdown
## Sources
- "Pedestal — Interceptors reference" — interceptor contract, context keys (distilled 2026-07-05)
- "Pedestal routing table syntax" — verbose vs terse syntax, constraints (distilled 2026-07-05)
- "Pedestal SSE guide" (saved 2026-07-05)
```

- **`(saved {date})`** — in the collection, not yet distilled. Add this line **at save time**, when the artifact's title is in hand for free.
- **`(distilled {date})`** — its contribution is folded into the context. Rewrite the line with what it contributed when you distil it.

The ledger is the source of truth for what remains: **any `(saved)` line is undigested work.** No MCP tool lists a collection's artifacts without fetching their full content, so the ledger must be maintained as sources land — then a future session resumes from the context alone, no collection-wide read needed.

A source that contributed nothing new still gets flipped to distilled ("no new material — confirms official docs") so it isn't re-fetched next pass. If the user saved sources outside this workflow and the ledger looks incomplete, `search_artifacts` by topic keywords to find strays, or ask the user what they added.

## Code is verbatim

Function signatures, code examples, CLI commands, and config snippets are preserved **verbatim** from sources — never paraphrased, never reconstructed from memory. Compress prose aggressively; never compress code. An agent loading this context will type what it reads here.

## Size and scoping

A reference earns density, not length. Target: **keep the context under ~300 lines.** When it outgrows that, the collection is scoped too broadly — split it:

1. Create a new, tighter collection for the outgrown topic ("Next.js 15" → "Next.js 15 Server Components" + "Next.js 15 Routing").
2. Move the relevant source artifacts across (`move_artifact` / `bulk_move_artifacts`).
3. Move the relevant context sections into the new collection's context.
4. Leave a pointer in each Meta: `- **Related**: "Next.js 15 Routing"`.

Splitting beats trimming — deleting hard-won detail defeats the purpose of the reference. Scoping tightly up front ("Datomic Client API", not "Databases") avoids most splits.
