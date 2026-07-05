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

The context is built and maintained **incrementally**: a structured skeleton of `##` sections, updated one source at a time with `replace_context_section`. Load [format.md](references/format.md) for the skeleton, the Sources ledger, and heading rules.

## Skill boundaries

- **This skill** is for researching technologies, APIs, and libraries.
- If the user wants to **track feature progress**, switch to [spec](../spec/SKILL.md).
- If the user wants to **pick up work** on an existing feature, switch to [pickup](../pickup/SKILL.md).

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
3. Register it in the `## Sources` ledger as `(saved {date})` — batch the ledger update per collection round with one `replace_context_section`, not one write per source. If the context doesn't exist yet, hold the lines and write them as part of the skeleton in Phase 4.

The ledger line at save time is what makes distillation resumable: there's no MCP tool that lists a collection's artifacts without their full content, so the context itself tracks what's been collected.

Aim for **5-15 high-quality sources**. Every source should earn its place by contributing something the others don't.

### Phase 4: Distil — one source at a time

**Never read the whole collection in one call.** A full-collection read over MCP truncates silently on collections this size, and the distill ends up built from a partial read. Work incrementally instead:

1. **Load the skeleton.** `read_collection_context`. If the context is empty, write the skeleton from [format.md](references/format.md) with Meta filled in (`write_collection_context`, once). If sources were collected in this session, the skeleton includes their `(saved)` ledger lines already.
2. **Find undigested sources.** Every `(saved)` line in the `## Sources` ledger is undigested work — this is how a fresh session resumes a half-distilled collection.
3. **Distil each source in turn:**
   - `fetch_artifact` for one source
   - Extract what it contributes — concepts, signatures, config, patterns, gotchas — and fold it into the affected sections with `replace_context_section` (use `create_if_missing: true` for new topic sections)
   - Flip its ledger line to `(distilled {date})` with a note of what it contributed
4. **Repeat** until no `(saved)` lines remain, then set Meta's `Updated` date.

What each section captures — and the rule that code is **verbatim**, prose is compressed — is defined in [format.md](references/format.md).

**The bar:** The distilled context should be sufficient to write correct code against this API without reading the raw sources. If an agent loads this context and still produces broken code because of missing information, the distillation isn't done.

Because each source is a separate pass, distillation is interruptible: a future session picks up the ledger and continues where this one stopped.

### Phase 5: Validate

Re-read the finished context (`read_collection_context`) and check:

- Does it cover the user's specific use case?
- Are all code examples syntactically correct for the target version?
- Are there contradictions between sources? Flag them explicitly.
- Is anything uncertain? Say so. "The docs say X but this GitHub issue suggests Y as of v3.2" is more useful than false confidence.
- Is it under the ~300-line budget? If it's outgrown its scope, split the collection per [format.md](references/format.md), "Size and scoping".

For a grounding audit — which claims in the distill are actually anchored to saved sources — offer [verify-distill](../verify-distill/SKILL.md).

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

1. Save the changelog and migration guide to the collection
2. Distil them like any other source: fetch each, update the affected sections with `replace_context_section`, add ledger lines
3. Fix code examples that changed, in place, section by section
4. Update `## Version Notes` and Meta's `Version` / `Updated` fields

Updates are surgical — a version bump touches the sections it changes, nothing else. Re-distil from scratch only if the technology has changed so much the skeleton no longer fits.

This is the whole point of using Sombra over a one-off research session — the reference compounds instead of evaporating.

## Arguments

Technology, API, or library name: $ARGUMENTS
