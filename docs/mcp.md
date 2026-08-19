# MCP (Model Context Protocol) — deep dive

> Who actually supports MCP, and how deep the support goes.

**Last verified:** 2026-08-19

MCP is a standardized protocol — servers expose tools, resources, and prompts; clients (CLIs) connect to them. "Supports MCP" is binary, but the _quality_ of support varies: which transports (stdio, HTTP, SSE), which features (tools only vs. resources + prompts), error handling, reconnect behavior.

All three tier-1 CLIs support MCP as a first-class extension mechanism. The differences are in configuration surface, per-tool permissioning, and how tool names flow into the rest of the system (permission rules, hook matchers).

## Claude Code

Docs: <https://code.claude.com/docs/en/mcp>

MCP servers are configured in `.mcp.json` (project-level, committed), in `~/.claude.json` (user and local scopes, written by `claude mcp add`), via CLI flag, or scoped to a sub-agent with the `mcpServers:` frontmatter field. Tools from MCP servers become first-class tool calls the model can invoke. MCP tool names follow `mcp__<server>__<tool>` and can be referenced in permission rules (`mcp__memory__.*`) and hook matchers.

Sub-agents can receive their own MCP server stack: string references share the parent session's connection, inline definitions are scoped to the sub-agent and disconnect when it finishes. This lets you keep MCP tools out of the main conversation's tool-description budget.

## Codex CLI

Docs: <https://developers.openai.com/codex/extend/mcp> (back on developers.openai.com after a mid-2026 detour through learn.chatgpt.com; both hosts mirror the same content)

MCP servers are configured in `~/.codex/config.toml` under `[mcp_servers.*]` — or, new since the 2026-05-15 refresh, in a project-scoped `.codex/config.toml` (trusted projects only) or via the `codex mcp` CLI (`codex mcp add <name> ... stdio <command>`). Supported transports are stdio and streamable HTTP. Codex stores **per-tool approval overrides** directly in the same table:

```toml
[mcp_servers.docs.tools.search]
approval_mode = "approve"
```

This is tighter per-tool scoping than Claude Code's permission rules — you approve or deny at the individual tool level (`tools.<tool>.approval_mode`, with `enabled_tools` / `disabled_tools` allow/deny lists and a `default_tools_approval_mode` of `auto` / `prompt` / `writes` / `approve` — the `writes` mode prompts for tools that aren't marked read-only), not just the server level. The two tradeoffs this page used to flag have both closed: `codex mcp` now manages servers from the CLI, and custom-agent TOML files accept an `mcp_servers` field, scoping servers to a specific sub-agent the way Claude Code's `mcpServers:` frontmatter does.

## Antigravity CLI (successor to Gemini CLI)

Docs: <https://antigravity.google/docs/cli/mcp> — a CLI-specific MCP page that did not exist at the 2026-07-25 refresh, when this section cited the shared platform page at `/docs/mcp`; the platform page still exists and agrees. Plus the [repo CHANGELOG](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md) for version attributions.

MCP support is documented: global servers in `~/.gemini/config/mcp_config.json`, workspace servers in `.agents/mcp_config.json`. Local servers use stdio via `command`; remote servers use `serverUrl` for SSE, Streamable HTTP, or websocket — the docs mark the legacy `url` / `httpUrl` fields as not supported (though the platform changelog of 2026-06-11 re-accepts `url` as an alias, so don't assert either absolutely). There's an `/mcp` interactive manager overlay, per-server `disabledTools` lists, `mcp(server/tool)` / `mcp(server/*)` / `mcp(*)` permission rules (unconfigured MCP tools default to Ask), and OAuth / ADC / custom-header auth. From the CHANGELOG era: configurable launch timeout with `-1` to disable (1.0.7), parallelized server initialization (1.0.4), per-server disable from the TUI (1.0.3), timeouts bounding hung servers (1.1.3), embedded-resource results surfaced instead of dropped (1.1.5), and relaxed OAuth issuer validation for nonconforming providers like Salesforce and Atlassian (1.1.7). Custom agents can scope MCP inheritance via the `inheritMcp` frontmatter field (CHANGELOG 1.1.6). Newer fixes worth knowing if you run MCP servers unattended: servers that drop a connection no longer force a full re-authentication, and a slow or hanging server no longer stalls the first interactive turn (servers load in the background for interactive sessions, while headless and one-shot runs keep blocking so their single scripted turn still sees the full toolset) (1.1.9); tools an MCP server marks as always-background stopped executing as blocking calls that stalled the turn, and a process leak on unexpected disconnect was closed (1.1.10); dropped progress callbacks were restored so long-running tool calls report progress again (1.1.11); OAuth client ID metadata documents are now supported, so a server implementing that part of the spec needs neither a hand-supplied client ID nor dynamic client registration, and one malformed entry in `mcp_config.json` is logged and skipped instead of taking down every other server (1.1.14). Not yet documented: resources/prompts support in the MCP doc itself.

### Predecessor: Gemini CLI (consumer sunset 2026-06-18; enterprise still served)

Docs: <https://geminicli.com/docs/tools/mcp-server/> (canonical MCP reference) and <https://geminicli.com/docs/hooks/reference/> (MCP tool naming inside hook matchers), plus the extensions system.

MCP is exposed alongside Gemini's built-in tool catalog. Tool names follow the pattern `mcp_<server_name>_<tool_name>` (underscore-separated, contrast with Claude Code's double-underscore). The naming is important because hook matchers are regexes — a `BeforeTool` hook with `matcher: "mcp_github_.*"` gates every GitHub MCP tool.

MCP support also composes with Plan Mode: read-only MCP tools like `github_read_issue` or `postgres_read_schema` are allowed in Plan Mode, write-capable ones aren't.

## What to check when refreshing

- Which transports are supported (stdio, HTTP, SSE, streamable-http)?
- Are resources and prompts supported, or just tools?
- Can the user scope permissions per-tool, or only per-server?
- What happens on reconnect / server crash?
- Can MCP servers be scoped to a sub-agent (vs. global to the session)?
- How do MCP tool names flow into permission rules and hook matchers?
