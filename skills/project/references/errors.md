# Error Recovery

Common failures when managing living projects and how to handle them.

## replace_context_section failures

| Error | Cause | Fix |
|-------|-------|-----|
| Heading not found | Typo in heading text, or section doesn't exist yet | Check exact heading text. Use `create_if_missing: true` for new sections. |
| Duplicate heading | Two sections share the same `##` heading | Fall back to `write_collection_context` with the full document. Then fix the duplicate by renaming one section. |
| Content too large | Section body exceeds size limits | Split into two phases with distinct headings. |

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

## Recovery principle

When a tool call fails, **stop and diagnose** before retrying. Read the error message. Check the current state with `read_collection_context`. Fix the root cause, then retry. Don't blindly repeat the same call.
