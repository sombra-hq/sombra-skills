---
name: verify-distill
description: >
  Run a verification pass over a Sombra collection's distilled context — identify
  which substantive claims are anchored to sources, which are partial matches, and
  which are unsourced — then walk the user through cleaning up the unsourced ones.
  Use when the user says "verify this collection", "check the distill", "are these
  claims sourced?", "audit the context", or asks to make sure a distilled context
  is grounded in real artifacts.
when_to_use: "verify distill, verify collection, check the distill, audit context, are these claims sourced, ground the distill, uncited claims, unsourced claims"
argument-hint: "[collection name or ID]"
allowed-tools:

  - mcp__claude_ai_Sombra__browse_collections
  - mcp__claude_ai_Sombra__verify_collection_context
  - mcp__claude_ai_Sombra__read_collection_context
  - mcp__claude_ai_Sombra__replace_context_section
  - mcp__claude_ai_Sombra__write_collection_context
  - mcp__claude_ai_Sombra__search_artifacts
  - mcp__claude_ai_Sombra__save_url
  - mcp__claude_ai_Sombra__move_artifact
  - mcp__claude_ai_Sombra__fetch_artifact
  - Read
---

# Verify Distill — Sombra

Run a grounding check over a collection's distilled context. The Sombra server scans every substantive sentence in the distill and classifies each one against the collection's saved artifacts:

- **ANCHORED** — strong source match (matcher score ≥ 0.7)
- **LOOSE** — partial match (0.3 ≤ score < 0.7); curation candidate
- **UNCITED** — no source above 0.3; the claim has no support in the collection

This skill surfaces the report and walks the user through cleaning up the UNCITED claims. The scanner is deterministic — the same input produces the same verdicts every time.

## Skill boundaries

- **This skill** is for verifying and cleaning a distilled context.
- For **writing or re-distilling** a context, use [research](../research/SKILL.md) or [guide](../guide/SKILL.md).
- For **tracking** a project or feature, use [project](../project/SKILL.md) or [spec](../spec/SKILL.md).
- **Don't accept or dismiss ANCHORED/LOOSE proposals from here.** That flow lives in the Sombra SPA — dashed indigo (anchored) and dotted amber (loose) inline chrome on the distill view. Point the user there at the end.

## Getting started

Jump straight into the workflow. If a Sombra tool call fails with an auth or connection error, guide the user to check `/mcp` for their connection status, or visit **sombra.so/mcp** for setup instructions.

## Workflow

### 1. Resolve the collection

- If `$ARGUMENTS` looks like a UUID, use it directly.
- If `$ARGUMENTS` names a collection, `browse_collections` and match by name.
- If `$ARGUMENTS` is empty or ambiguous, `browse_collections` and ask the user which one.

Confirm the collection before scanning — verifying the wrong distill wastes everyone's time.

### 2. Run the scan

Call `verify_collection_context` with the resolved collection ID. The response is structured:

```json
{
  "project_name": "string",
  "context_len": 24808,
  "summary": { "total": 174, "substantive": 121,
               "anchored": 37, "loose": 80, "uncited": 4 },
  "sentences": [
    { "text": "...", "verdict": "ANCHORED", "top_match": { ... } },
    { "text": "...", "verdict": "UNCITED" },
    { "text": "## Heading", "substantive": false }
  ]
}
```

`top_match` (when present) has the artifact id, title, and excerpt — useful when the user wants to know which source is closest.

### 3. Report the summary

Show counts and a calibration line. Keep it tight:

```
Verified: {Collection name} ({context_len} chars)

  {substantive} substantive claims
  {anchored} anchored      ({%})
  {loose} loose            ({%})  — review in the SPA
  {uncited} uncited        ({%})  — actionable here
```

The counts come straight from `summary.*`. `substantive` is the denominator for the percentages — non-substantive sentences (headings, code, list scaffolding) are excluded from the verdict math.

Add one calibration sentence if the shape is unusual:

- **Many UNCITED + small collection** → "This looks more like a working note than a research distill — the verifier is most useful on contexts that summarise external sources."
- **Mostly ANCHORED, no LOOSE/UNCITED** → "Distill is well-grounded. Nothing to clean up here."
- **All LOOSE, few ANCHORED** → "Most claims partially match sources but few anchor cleanly — wording may have drifted from source language during distillation."

If `summary.uncited == 0`, **stop here**. Don't modify the context. The report is the result.

### 4. Walk the UNCITED claims

For each UNCITED sentence, show it and ask the user what to do. Use this exact menu:

```
UNCITED {i}/{n}

  "{sentence.text}"

  How should this be handled?
   a. Strip from the distill — no source, not worth keeping
   b. Find a source — search the workspace or save a URL
   c. Common knowledge — leave it
   d. Edit the wording — reword closer to source language
   e. Skip
```

If the sentence has a near-miss (e.g. a LOOSE-scoring artifact came up in the matcher), surface the closest artifact title and score from `top_match` so the user can decide between (b) and (d):

```
  Closest source in this collection: "{top_match.title}" (score {top_match.score})
```

A score in the 0.3–0.7 band typically means the claim is *about* the right topic but phrased far from the source. That's the case for (d) — rewording — more often than (b).

#### a. Strip from the distill

1. `read_collection_context` to get the current text.
2. Identify the smallest containing section (the `##` heading the sentence falls under).
3. Remove the sentence, preserving surrounding prose — be precise, don't paraphrase neighbouring text.
4. Show the user the exact text being removed and the surrounding context, then confirm before writing.
5. `replace_context_section` for that one section. Use `write_collection_context` only when the change crosses section boundaries.

#### b. Find a source

1. `search_artifacts` with keywords from the sentence — start in the user's workspace.
2. If nothing relevant exists, ask for a URL; `save_url` and `move_artifact` into the collection.
3. After the source lands, mention that the next context-write will trigger a fresh scan automatically — the sentence should anchor on the next pass.
4. Don't strip the sentence in this branch; the goal was to ground it, not remove it.

#### c. Common knowledge

Skip silently. No tool calls.

#### d. Edit the wording

1. Show the original sentence and the closest source's excerpt (from `top_match` if present, or fetch the artifact).
2. Propose a rewrite that uses source-aligned language.
3. Confirm with the user before writing.
4. `replace_context_section` for the affected section.

#### e. Skip

Move on. No tool calls.

### Loop hygiene

- If the user picks **c** (common knowledge) or **e** (skip) three times in a row, stop offering and jump to step 5.
- Batch edits in your head, but write to the context one section at a time — never accumulate edits across multiple sections into a single `write_collection_context`.
- Every distill-modifying write needs explicit user confirmation. Quote the exact text being removed or changed.

### 5. Summarise

Tell the user what changed. Be specific:

```
Cleaned up {n} uncited claims in **{Collection name}**:
- Stripped: "{exact removed sentence}"
- Stripped: "{exact removed sentence}"
- Reworded: "{original}" → "{rewrite}"
  (closer to "{source artifact title}")
- Left as-is: "{sentence}" (common knowledge)

Next context-write will trigger a fresh scan.

For the {loose} loose claims, review them in the Sombra distill view — they show
as dotted amber inline. Anchored proposals ({anchored}) are dashed indigo. You
can Accept or Dismiss either without leaving the app.
```

Use the user's actual sentences and artifact titles — don't paraphrase. The summary is the user's audit trail for what changed.

If nothing was modified, the summary is just the count and the in-app pointer.

## Notes

- **Don't replicate the in-app citation flow.** ANCHORED and LOOSE proposals are designed to be reviewed in the SPA. The skill's value is the report + the UNCITED cleanup that the in-app flow can't easily drive.
- **The scan is deterministic.** If the user re-runs immediately without changes, they'll get the same numbers. Re-running after any context-write is fine — the server re-scans on every write anyway.
- **Treat the matcher as the ground truth for verdict, not your judgement.** If the matcher says UNCITED but you think the claim is obviously supported, search the collection — the source may not be saved, or the sentence may be paraphrased far from the source language.

## Arguments

Collection name or ID (optional — will list collections if omitted): $ARGUMENTS
