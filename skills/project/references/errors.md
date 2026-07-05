# Error Recovery

Common failures when managing living projects and how to handle them.

## replace_context_section failures

| Error | Cause | Fix |
|-------|-------|-----|
| Heading not found | Typo in heading text, or section doesn't exist yet | Check exact heading text. Use `create_if_missing: true` for new sections. |
| Duplicate heading | Two sections share the same `##` heading | Fall back to `write_collection_context` with the full document. Then fix the duplicate by renaming one section. |
| Content too large | Section body exceeds size limits | Move detail into linked artifacts, then split into two phases with distinct headings if still needed. |

## Oversized or truncated context

| Symptom | Cause | Fix |
|---------|-------|-----|
| `read_collection_context` returns a document that ends mid-line or is missing trailing sections | Document exceeds the MCP client's tool-result limit | Run the compaction pass from [format.md](format.md): archive completed phases, trim Decisions to 5, extract prose to artifacts. Section writes work on the full server-side document even when reads are truncated. Re-read to confirm. |
| Document keeps growing back past budget | Detail being written inline instead of into artifacts | Route reports, decision history, and item detail to linked artifacts per format.md. Items stay one line. |

## Authentication failures

| Error | Cause | Fix |
|-------|-------|-----|
| Not authenticated | OAuth token expired or never completed | Guide user: reconnect Sombra in their MCP client settings, or visit sombra.so/mcp for setup instructions. |
| Connection refused | MCP server not configured | Guide user to sombra.so/mcp for setup instructions specific to their client (Claude Desktop, Claude Code, ChatGPT, etc.). |
| Token expired after 21 days | Inactivity timeout | Simply reconnect — no need to remove and re-add. |

## Search failures

| Error | Cause | Fix |
|-------|-------|-----|
| No results for project name | Project exists but search term doesn't match collection name or Meta content | Try `browse_collections` and scan by name. Ask the user what they called it. |
| Multiple matches | Several collections could be the right project | Read each collection's context, check the Meta section, and ask the user which one is current. |

## Recovering from a bad write

If a section write went wrong — replaced the wrong section, clobbered content, or the document looks mangled after an update — the server keeps revision history:

1. `changes_since` (kind: `context`, with the collection ID and a timestamp before the suspect write) shows which sections each revision touched and the line counts changed. Filter by `section` to trace one heading's history.
2. `read_snapshot` (same ID, `as_of` a timestamp before the bad write, optionally filtered to the section) returns the content as it was.
3. Restore with `replace_context_section` using the snapshot content.

These are **recovery tools, not routine ones**. The working assumption everywhere else is that the context document is fresh and canonical — don't audit history on pickup or before ordinary updates. Reach for them only when something looks wrong or the user asks what happened.

## Recovery principle

When a tool call fails, **stop and diagnose** before retrying. Read the error message. Check the current state with `read_collection_context`. Fix the root cause, then retry. Don't blindly repeat the same call.
