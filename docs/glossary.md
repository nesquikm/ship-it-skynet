# Glossary

Half the confusion between these tools is that "agent" means five different things. This page maps vendor terminology to a neutral vocabulary.

**Format:** neutral term → what each vendor calls it → short definition.

**Last updated:** 2026-06-10

> **Migration note (2026-06-10):** the Gemini CLI rows below were replaced by Antigravity CLI (`agy`), its announced successor, ahead of Gemini CLI's 2026-06-18 consumer sunset. Antigravity visibly inherits Gemini conventions (`~/.gemini/` paths, extension variables), but only claims verifiable from the [official repo](https://github.com/google-antigravity/antigravity-cli) appear here — "pending docs" means exactly that. The pre-migration Gemini CLI rows are preserved in git history and remain accurate for enterprise users of the predecessor.

## Terms

### Sub-agent _(neutral)_

A child process the main agent can spawn with its own isolated context window and tool access.

| Vendor          | Term      | Notes                                                                                                                                                                                            |
| --------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Claude Code     | sub-agent | First-class. Built-in `Explore`, `Plan`, `general-purpose`; user-defined via `.claude/agents/`                                                                                                   |
| Codex CLI       | subagent  | First-class feature for "parallelized task workflows" — see [Codex features][cx-features]                                                                                                        |
| Antigravity CLI | subagent  | Specialized agents auto-discovered from installed plugins; per-subagent interaction timeouts; subagent conversations tracked separately from `/resume`. Definition format and paths pending docs |

### Skill _(neutral)_

A reusable, named procedure (usually markdown-with-frontmatter) that the agent can invoke on demand. Distinct from a slash command in that skills can be triggered by model reasoning, not just by user input.

Claude Code and Codex CLI implement the [Agent Skills open standard](https://agentskills.io) — `SKILL.md` files are largely portable between them. Antigravity CLI's predecessor implemented it too; whether `agy` keeps the `SKILL.md` layout is pending docs.

| Vendor          | Term  | Path                                                                                                                        |
| --------------- | ----- | --------------------------------------------------------------------------------------------------------------------------- |
| Claude Code     | skill | `.claude/skills/<name>/SKILL.md` (project), `~/.claude/skills/` (user)                                                      |
| Codex CLI       | skill | `.agents/skills/<name>/SKILL.md` (project), `$HOME/.agents/skills/` (user)                                                  |
| Antigravity CLI | skill | Custom/fallback skills load from config and plugin directories; skill-derived slash commands. Path conventions pending docs |

### Hook _(neutral)_

A user-defined script the harness runs at a lifecycle event (pre-tool, post-tool, on-submit, etc.). Runs outside the model and cannot be bypassed by the model.

| Vendor          | Term | Notes                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------- | ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code     | hook | Full lifecycle — `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, and ~20 others. Configured in `.claude/settings.json`                                                                                                                                                                                                                                                                          |
| Codex CLI       | hook | Ten events — `SessionStart`, `SubagentStart`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `UserPromptSubmit`, `SubagentStop`, `PreCompact`, `PostCompact`, `Stop` — in `hooks.json`, enabled by default, plus legacy `notify`. Matchers cover Bash, `apply_patch`, and MCP tool names (now with `updatedInput` rewriting) but not `unified_exec` or `WebSearch`. See [hooks deep-dive](hooks.md#codex-cli) |
| Antigravity CLI | hook | ⏳ Announced as preserved from Gemini CLI in the transition blog; no event catalog published yet. Predecessor Gemini CLI had a full fingerprinted lifecycle (`BeforeTool`, `AfterTool`, ...)                                                                                                                                                                                                                     |

### MCP server _(neutral)_

A Model Context Protocol server — an external process the CLI connects to for tools, resources, or prompts. Standardized protocol, bring-your-own-server.

| Vendor          | Term       | Configuration                                                                                                         |
| --------------- | ---------- | --------------------------------------------------------------------------------------------------------------------- |
| Claude Code     | MCP server | `.mcp.json` (project), `~/.claude/settings.json` (user), or CLI flag                                                  |
| Codex CLI       | MCP server | `~/.codex/config.toml` (or project `.codex/config.toml`) under `[mcp_servers.*]`; per-tool overrides; `codex mcp` CLI |
| Antigravity CLI | MCP server | `config/mcp_config.json`; stdio or `url` transports; per-server disable from the TUI; configurable launch timeout     |

### Persistent memory file _(neutral)_

A markdown file the CLI loads automatically into the model's context at session start, used for project conventions, style guides, and standing instructions.

| Vendor          | File        | Hierarchy                                                                                                                |
| --------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| Claude Code     | `CLAUDE.md` | Project root + parents, `~/.claude/CLAUDE.md` for user-level, agent `memory` for per-agent                               |
| Codex CLI       | `AGENTS.md` | Git root down, `~/.codex/AGENTS.md` (or `AGENTS.override.md`) for global                                                 |
| Antigravity CLI | `.md` rules | Rule `.md` files loaded at boot; exclusion rules and allowlists via `rules.json`. File naming and hierarchy pending docs |

### Plan mode _(neutral)_

A read-only mode where the agent explores and proposes a plan without making changes. Exits when the user approves the plan (or manually).

| Vendor          | Term            | How to enter                                                                                                 |
| --------------- | --------------- | ------------------------------------------------------------------------------------------------------------ |
| Claude Code     | plan mode       | `permissionMode: "plan"` or in-session toggle                                                                |
| Codex CLI       | Plan mode       | `/plan [prompt]` or `plan_mode_reasoning_effort` in config.toml                                              |
| Antigravity CLI | ⏳ pending docs | Not yet documented for `agy`; predecessor Gemini CLI had `/plan [goal]`, `Shift+Tab`, `--approval-mode=plan` |

---

**Adding a term?** Pick a neutral name, define it in one sentence, then fill the vendor table. If one vendor doesn't have the concept at all, write `❌ no equivalent` instead of `⏳ pending`.

[cx-features]: https://developers.openai.com/codex/cli/features
