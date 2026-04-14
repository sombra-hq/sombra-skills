# Sombra Plugin

Living documentation that persists across conversations. Track projects, manage feature specs, and build technical references — all stored in [Sombra](https://sombra.so) so you can pick up exactly where you left off.

Works with Claude Code, Claude Desktop, Claude.ai, ChatGPT, OpenCode, and any MCP-compatible client.

## Install

### Claude Code

```
/plugin marketplace add sombra-hq/claude-code
/plugin install sombra@sombra-hq-claude-code
```

### Claude Desktop & Claude.ai

Go to **Settings** → **Connectors** → **Add Custom Connector**. Enter "Sombra" as the name and `https://sombra.so/mcp` as the URL. Click Connect and log in.

### ChatGPT

In ChatGPT, go to **Settings** → **Apps**, enable **Developer mode**, click **Create App**, and set the URL to `https://sombra.so/mcp` with auth type **OAuth**.

### OpenCode

Add to `opencode.json` in your project root (or `~/.config/opencode/opencode.json` for global):

```json
{
  "mcp": {
    "sombra": {
      "type": "remote",
      "url": "https://sombra.so/mcp"
    }
  }
}
```

OpenCode handles OAuth automatically — it will open your browser to log in on first use.

### Other MCP clients

Connect to `https://sombra.so/mcp` via Streamable HTTP with OAuth 2.1. See [sombra.so/mcp](https://sombra.so/mcp) for details.

On first use, you'll be prompted to connect your Sombra account via OAuth. Free accounts work — no credit card required.

## Skills

### `/sombra:project` — Living project tracker

Track any multi-phase effort across conversations: house purchase, renovation, content strategy, product launch, learning goals. Works in Claude Desktop, Claude.ai, ChatGPT, and Claude Code.

```
/sombra:project Kitchen Renovation
```

Creates a Sombra collection with structured phases, checkboxes, a decisions log, and open questions. Updates surgically across sessions — no rewriting the whole document when one item changes.

### `/sombra:spec` — Living feature spec

Everything in `project`, plus git branch context, code path references, and commit-based progress detection. For software features. Requires git access (Claude Code).

```
/sombra:spec Auth Rework
```

Tracks which branch, which files, which items are done — and can cross-reference your git log to suggest what to check off.

### `/sombra:resume` — Pick up where you left off

Loads an existing project or spec, shows what's changed, and highlights the next step. Adapts automatically — git-aware in a repo, plain status report elsewhere.

```
/sombra:resume
/sombra:resume Auth Rework
```

### `/sombra:research` — Build a technical reference

Deep research on APIs, libraries, and frameworks. Saves sources to Sombra, distils them into a dense context document that any agent can load to write correct code without stale training data.

```
/sombra:research Pedestal HTTP routing
```

## How it works

Each skill uses Sombra collections as the persistence layer:

- **Collections** organise related material
- **Context documents** are the distilled, structured view (the spec, the project plan, the API reference)
- **Artifacts** hold raw material — notes, saved web pages, decision records

Updates are surgical (`replace_context_section`) rather than full rewrites, so your documents stay clean as they evolve.

## Compatibility

| Skill | Works in |
|-------|----------|
| `project` | Claude Desktop, Claude.ai, ChatGPT, Claude Code, OpenCode, any MCP client |
| `spec` | Claude Code, OpenCode (requires git + file access) |
| `resume` | Everywhere (adapts — git-aware in repos, plain status elsewhere) |
| `research` | Anywhere with web search access |

## Requirements

- A [Sombra](https://sombra.so) account (free tier available)
- An MCP client that supports Sombra

For setup help or troubleshooting, see [sombra.so/mcp](https://sombra.so/mcp).
