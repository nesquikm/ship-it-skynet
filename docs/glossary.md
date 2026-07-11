# Glossary

Half the confusion between these tools is that "agent" means five different things. This page maps vendor terminology to a neutral vocabulary.

**Format:** neutral term → what each vendor calls it → short definition.

**Last updated:** 2026-07-11

> **Migration note (2026-06-10):** the Gemini CLI rows below were replaced by Antigravity CLI (`agy`), its announced successor, ahead of Gemini CLI's 2026-06-18 consumer sunset. Antigravity visibly inherits Gemini conventions (`~/.gemini/` paths, extension variables). The rows initially carried "pending docs" caveats because only the [official repo](https://github.com/google-antigravity/antigravity-cli) was citable; since 2026-07-11 the official docs at antigravity.google are fetchable (llms.txt index + raw markdown pages) and the rows below cite them. The pre-migration Gemini CLI rows are preserved in git history and remain accurate for enterprise users of the predecessor.

## Terms

### Sub-agent _(neutral)_

A child process the main agent can spawn with its own isolated context window and tool access.

| Vendor          | Term      | Notes                                                                                                                                                                                                                                                                                                               |
| --------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code     | sub-agent | First-class. Built-in `Explore`, `Plan`, `general-purpose`; user-defined via `.claude/agents/`                                                                                                                                                                                                                      |
| Codex CLI       | subagent  | First-class: Codex "runs parallel agents and combines their results"; custom agents as TOML files under `~/.codex/agents/` or `.codex/agents/` — see [Codex subagents][cx-features]                                                                                                                                 |
| Antigravity CLI | subagent  | Custom agents at `.agents/agents/<name>/agent.md` (workspace) or `~/.gemini/config/agents/<name>/agent.md` (global) per the [agents command doc](https://antigravity.google/docs/cli/commands/agents); specialized agents also auto-discovered from installed plugins; nested subagents supported (CHANGELOG 1.1.1) |

### Skill _(neutral)_

A reusable, named procedure (usually markdown-with-frontmatter) that the agent can invoke on demand. Distinct from a slash command in that skills can be triggered by model reasoning, not just by user input.

Claude Code and Codex CLI implement the [Agent Skills open standard](https://agentskills.io) — `SKILL.md` files are largely portable between them. Antigravity documents both a flat `.md` workspace-skill format (CLI doc) and the `SKILL.md` folder standard (platform doc) — see [skills deep-dive](skills.md).

| Vendor          | Term  | Path                                                                                                                                                                                  |
| --------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code     | skill | `.claude/skills/<name>/SKILL.md` (project), `~/.claude/skills/` (user)                                                                                                                |
| Codex CLI       | skill | `.agents/skills/<name>/SKILL.md` (project), `$HOME/.agents/skills/` (user)                                                                                                            |
| Antigravity CLI | skill | Workspace `.agents/skills/` (flat `.md` files auto-converted to slash commands), global `~/.gemini/antigravity-cli/skills/`; the platform doc separately specifies `SKILL.md` folders |

### Hook _(neutral)_

A user-defined script the harness runs at a lifecycle event (pre-tool, post-tool, on-submit, etc.). Runs outside the model and cannot be bypassed by the model.

| Vendor          | Term | Notes                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------- | ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code     | hook | Full lifecycle — `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, and 25+ others (see [hooks deep-dive](hooks.md)). Configured in `.claude/settings.json`                                                                                                                                                                                                                                        |
| Codex CLI       | hook | Ten events — `SessionStart`, `SubagentStart`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `UserPromptSubmit`, `SubagentStop`, `PreCompact`, `PostCompact`, `Stop` — in `hooks.json`, enabled by default, plus legacy `notify`. Matchers cover Bash, `apply_patch`, and MCP tool names (now with `updatedInput` rewriting) but not `unified_exec` or `WebSearch`. See [hooks deep-dive](hooks.md#codex-cli) |
| Antigravity CLI | hook | `hooks.json` in workspace `.agents/` or global `~/.gemini/config/` — `PreToolUse`, `PostToolUse`, `PreInvocation`, `PostInvocation`, `Stop`; blocking `PreToolUse` per [the hooks doc](https://antigravity.google/docs/hooks). No input rewriting or fingerprinting (unlike predecessor Gemini CLI). See [hooks deep-dive](hooks.md)                                                                             |

### MCP server _(neutral)_

A Model Context Protocol server — an external process the CLI connects to for tools, resources, or prompts. Standardized protocol, bring-your-own-server.

| Vendor          | Term       | Configuration                                                                                                                                                                                                           |
| --------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code     | MCP server | `.mcp.json` (project), `~/.claude.json` (user and local scopes), or CLI flag (`--mcp-config`)                                                                                                                           |
| Codex CLI       | MCP server | `~/.codex/config.toml` (or project `.codex/config.toml`) under `[mcp_servers.*]`; per-tool overrides; `codex mcp` CLI                                                                                                   |
| Antigravity CLI | MCP server | `~/.gemini/config/mcp_config.json` (global) + `.agents/mcp_config.json` (workspace); stdio via `command`, remote SSE / Streamable HTTP / websocket via `serverUrl`; `/mcp` manager; `mcp(server/tool)` permission rules |

### Persistent memory file _(neutral)_

A markdown file the CLI loads automatically into the model's context at session start, used for project conventions, style guides, and standing instructions.

| Vendor          | File        | Hierarchy                                                                                                                                   |
| --------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code     | `CLAUDE.md` | Project root + parents, `~/.claude/CLAUDE.md` for user-level, agent `memory` for per-agent                                                  |
| Codex CLI       | `AGENTS.md` | Git root down, `~/.codex/AGENTS.md` (or `AGENTS.override.md`) for global                                                                    |
| Antigravity CLI | `GEMINI.md` | Workspace `GEMINI.md`/`AGENTS.md` + global `~/.gemini/GEMINI.md`; rules in `.agents/rules`; exclusion rules and allowlists via `rules.json` |

### Plan mode _(neutral)_

A read-only mode where the agent explores and proposes a plan without making changes. Exits when the user approves the plan (or manually).

| Vendor          | Term      | How to enter                                                                                      |
| --------------- | --------- | ------------------------------------------------------------------------------------------------- |
| Claude Code     | plan mode | `permissionMode: "plan"` or in-session toggle                                                     |
| Codex CLI       | Plan mode | `/plan [prompt]` or `plan_mode_reasoning_effort` in config.toml                                   |
| Antigravity CLI | plan mode | `/plan` prompt prefix, `Shift+Tab` mode cycling, `--mode=plan` flag, `agentMode` in settings.json |

---

**Adding a term?** Pick a neutral name, define it in one sentence, then fill the vendor table. If one vendor doesn't have the concept at all, write `❌ no equivalent` instead of `⏳ pending`.

[cx-features]: https://learn.chatgpt.com/docs/agent-configuration/subagents
