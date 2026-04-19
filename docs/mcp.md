# MCP (Model Context Protocol) — deep dive

> Who actually supports MCP, and how deep the support goes.

**Last verified:** 2026-04-19

MCP is a standardized protocol — servers expose tools, resources, and prompts; clients (CLIs) connect to them. "Supports MCP" is binary, but the _quality_ of support varies: which transports (stdio, HTTP, SSE), which features (tools only vs. resources + prompts), error handling, reconnect behavior.

All three tier-1 CLIs support MCP as a first-class extension mechanism. The differences are in configuration surface, per-tool permissioning, and how tool names flow into the rest of the system (permission rules, hook matchers).

## Claude Code

Docs: <https://code.claude.com/docs/en/mcp>

MCP servers are configured in `.mcp.json` (project-level, committed), via `~/.claude/settings.json` (user), via CLI flag, or scoped to a sub-agent with the `mcpServers:` frontmatter field. Tools from MCP servers become first-class tool calls the model can invoke. MCP tool names follow `mcp__<server>__<tool>` and can be referenced in permission rules (`mcp__memory__.*`) and hook matchers.

Sub-agents can receive their own MCP server stack: string references share the parent session's connection, inline definitions are scoped to the sub-agent and disconnect when it finishes. This lets you keep MCP tools out of the main conversation's tool-description budget.

## Codex CLI

Docs: <https://github.com/openai/codex/blob/main/docs/config.md> (see "Connecting to MCP servers")

MCP servers are configured in `~/.codex/config.toml` under `[mcp_servers.*]`. Codex stores **per-tool approval overrides** directly in the same table:

```toml
[mcp_servers.docs.tools.search]
approval_mode = "approve"
```

This is tighter per-tool scoping than Claude Code's permission rules — you approve or deny at the individual tool level, not just the server level. The tradeoff is you edit TOML instead of using a CLI, and there's no documented way to scope MCP servers to a specific sub-agent.

## Gemini CLI

Docs: <https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/reference.md> (MCP tool naming), plus `docs/tools/mcp-server.md` and the extensions system.

MCP is exposed alongside Gemini's built-in tool catalog. Tool names follow the pattern `mcp_<server_name>_<tool_name>` (underscore-separated, contrast with Claude Code's double-underscore). The naming is important because hook matchers are regexes — a `BeforeTool` hook with `matcher: "mcp_github_.*"` gates every GitHub MCP tool.

MCP support also composes with Plan Mode: read-only MCP tools like `github_read_issue` or `postgres_read_schema` are allowed in Plan Mode, write-capable ones aren't.

## What to check when refreshing

- Which transports are supported (stdio, HTTP, SSE, streamable-http)?
- Are resources and prompts supported, or just tools?
- Can the user scope permissions per-tool, or only per-server?
- What happens on reconnect / server crash?
- Can MCP servers be scoped to a sub-agent (vs. global to the session)?
- How do MCP tool names flow into permission rules and hook matchers?
