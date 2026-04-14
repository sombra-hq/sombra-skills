---
name: research
description: >
  Deep technical research on APIs, libraries, and frameworks using Sombra
  as a persistent knowledge base. Use when the user needs to learn a new API,
  evaluate a library, build reference documentation for a technology, understand
  migration paths between versions, or says things like "research this API",
  "build me a reference for X", "what's the right way to use Y", or
  "I need to understand Z before I start coding".
when_to_use: "research API, build reference, evaluate library, learn framework, understand migration, API docs, technical reference"
argument-hint: "[technology, API, or library name]"
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
  - WebSearch
  - WebFetch
  - Read
  - Grep
  - Glob
---

# API & Library Research — Sombra

Build persistent, distilled technical references using Sombra as your knowledge layer. The goal is a living collection whose distilled context contains everything an agent needs to write correct code — without relying on training data that may be stale.

## Skill boundaries

- **This skill** is for researching technologies, APIs, and libraries.
- If the user wants to **track feature progress**, switch to [spec](../spec/SKILL.md).
- If the user wants to **resume work** on an existing feature, switch to [resume](../resume/SKILL.md).

## Getting started

Jump straight into the workflow below. If a Sombra tool call fails with an auth or connection error, guide the user to check `/mcp` for their connection status, or visit **sombra.so/mcp** for setup instructions.

## What you'll end up with

A Sombra collection containing 5-15 authoritative sources (official docs, examples, migration guides, known gotchas) distilled into a dense context document. That context is your ground truth — load it before writing code, and you'll avoid the class of bugs that come from outdated training data or hallucinated APIs.

The collection persists across sessions. As the API evolves, add new sources and re-distil. The reference stays current without starting over.

## Methodology

### Phase 1: Scope

Before searching anything, establish:

- **What exactly?** Pin down the specific library, API version, or framework. "React" is too broad. "React Server Components in Next.js 15" is useful.
- **What for?** The user's use case determines which parts of the API matter. Don't research everything — research what's needed.
- **What exists already?** Check Sombra first. Browse collections, search for the technology name. You may already have a reference from a previous session.

If a relevant collection exists, read its context. You might already have what's needed.

### Phase 2: Source hierarchy

Search and save in this priority order. Each level fills gaps the previous one left:

1. **Official documentation** — API reference, getting started guides, changelogs. Ground truth. Start here always.
2. **Official examples and repos** — GitHub repositories from the maintainers, example projects, cookbooks. Idiomatic usage that docs often don't show.
3. **Migration guides** — If version differences matter, these are gold. They tell you exactly what changed and what breaks.
4. **High-quality technical blogs** — Posts from core contributors or well-known practitioners. Not "Top 10 Node.js Frameworks 2025".
5. **Issue trackers and discussions** — Known gotchas, breaking changes, workarounds not yet in docs.

**Skip:** Tutorials aimed at absolute beginners (unless the user is one), aggregator sites, AI-generated documentation summaries, SEO content farms.

### Phase 3: Collect

For each useful source:

1. Save it to Sombra with `save_url`
2. Move it into the collection (create one if needed — scope tightly: "Datomic Client API" not "Databases")
3. Briefly note what the source covers and why it matters

Aim for **5-15 high-quality sources**. Every source should earn its place by contributing something the others don't.

### Phase 4: Distil

Read the full collection content, then write a distilled context that captures:

- **Core concepts** — The mental model. Key abstractions and how they relate.
- **API surface** — Function signatures, key methods, constructor patterns. **Preserve verbatim** — this is what agents need to write correct code.
- **Configuration** — Required config, environment variables, connection strings. Verbatim.
- **Code patterns** — Idiomatic usage examples. Verbatim.
- **Gotchas** — Breaking changes, common mistakes, version-specific behaviour. Highest-value items.
- **What NOT to do** — Deprecated patterns, anti-patterns from older versions that still appear in training data.

**The bar:** The distilled context should be sufficient to write correct code against this API without reading the raw sources. If an agent loads this context and still produces broken code because of missing information, the distillation isn't done.

### Phase 5: Validate

Before presenting the reference:

- Does the context cover the user's specific use case?
- Are all code examples syntactically correct for the target version?
- Are there contradictions between sources? Flag them explicitly.
- Is anything uncertain? Say so. "The docs say X but this GitHub issue suggests Y as of v3.2" is more useful than false confidence.

## Variant: Library evaluation

When comparing competing libraries ("should I use X or Y?"):

1. Create a separate collection per candidate
2. Research each independently using the methodology above
3. Create a comparison note covering:
   - **API ergonomics** for the user's specific use case
   - **Maintenance health** — commit frequency, issue response time, bus factor
   - **Ecosystem fit** — does it play well with what they already use?
   - **Performance characteristics** (if relevant)
   - **Licence implications**

Present the tradeoffs clearly. Don't make the decision — the user knows their constraints better than you do.

## Keeping it current

The collection is alive. When the API ships a new version:

1. Save the changelog and migration guide
2. Update code examples that have changed
3. Re-distil the context
4. Note what version the context reflects

This is the whole point of using Sombra over a one-off research session — the reference compounds instead of evaporating.

## Arguments

Technology, API, or library name: $ARGUMENTS
