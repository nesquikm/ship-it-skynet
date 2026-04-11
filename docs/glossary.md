# Glossary

Half the confusion between these tools is that "agent" means five different things. This page maps vendor terminology to a neutral vocabulary.

**Format:** neutral term → what each vendor calls it → short definition.

**Last updated:** 2026-04-11

## Terms

### Sub-agent _(neutral)_

A child process the main agent can spawn with its own isolated context window and tool access.

| Vendor      | Term              | Notes                                                                                           |
| ----------- | ----------------- | ----------------------------------------------------------------------------------------------- |
| Claude Code | sub-agent         | First-class. Built-in `Explore`, `Plan`, `general-purpose`; user-defined via `.claude/agents/`  |
| Codex CLI   | subagent          | First-class feature for "parallelized task workflows" — see [Codex features][cx-features]       |
| Gemini CLI  | research subagent | Built-in only (`codebase_investigator`, `cli_help`); not user-definable. Invoked via tool calls |

### Skill _(neutral)_

A reusable, named procedure (usually markdown-with-frontmatter) that the agent can invoke on demand. Distinct from a slash command in that skills can be triggered by model reasoning, not just by user input.

All three vendors implement the [Agent Skills open standard](https://agentskills.io) — `SKILL.md` files are largely portable across CLIs.

| Vendor      | Term  | Path                                                                       |
| ----------- | ----- | -------------------------------------------------------------------------- |
| Claude Code | skill | `.claude/skills/<name>/SKILL.md` (project), `~/.claude/skills/` (user)     |
| Codex CLI   | skill | `.agents/skills/<name>/SKILL.md` (project), `$HOME/.agents/skills/` (user) |
| Gemini CLI  | skill | `.gemini/skills/` or `.agents/skills/` alias (workspace/user/extension)    |

### Hook _(neutral)_

A user-defined script the harness runs at a lifecycle event (pre-tool, post-tool, on-submit, etc.). Runs outside the model and cannot be bypassed by the model.

| Vendor      | Term | Notes                                                                                                                                                                                            |
| ----------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Claude Code | hook | Full lifecycle — `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, and ~20 others. Configured in `.claude/settings.json`                                                          |
| Codex CLI   | hook | Five events — `SessionStart`, `PreToolUse` (Bash-only, bypassable), `PostToolUse`, `UserPromptSubmit`, `Stop` — in `hooks.json`, plus legacy `notify`. See [hooks deep-dive](hooks.md#codex-cli) |
| Gemini CLI  | hook | Full lifecycle — `BeforeTool`, `AfterTool`, `BeforeAgent`, `AfterAgent`, etc. Configured in `.gemini/settings.json`. Fingerprinted for safety                                                    |

### MCP server _(neutral)_

A Model Context Protocol server — an external process the CLI connects to for tools, resources, or prompts. Standardized protocol, bring-your-own-server.

| Vendor      | Term       | Configuration                                                             |
| ----------- | ---------- | ------------------------------------------------------------------------- |
| Claude Code | MCP server | `.mcp.json` (project), `~/.claude/settings.json` (user), or CLI flag      |
| Codex CLI   | MCP server | `~/.codex/config.toml` under `[mcp_servers.*]` with per-tool overrides    |
| Gemini CLI  | MCP server | `.gemini/settings.json` and extensions; tools named `mcp_<server>_<tool>` |

### Persistent memory file _(neutral)_

A markdown file the CLI loads automatically into the model's context at session start, used for project conventions, style guides, and standing instructions.

| Vendor      | File        | Hierarchy                                                                                            |
| ----------- | ----------- | ---------------------------------------------------------------------------------------------------- |
| Claude Code | `CLAUDE.md` | Project root + parents, `~/.claude/CLAUDE.md` for user-level, agent `memory` for per-agent           |
| Codex CLI   | `AGENTS.md` | Git root down, `~/.codex/AGENTS.md` (or `AGENTS.override.md`) for global                             |
| Gemini CLI  | `GEMINI.md` | Workspace + parents, `~/.gemini/GEMINI.md` for global, plus JIT discovery when tools access new dirs |

### Plan mode _(neutral)_

A read-only mode where the agent explores and proposes a plan without making changes. Exits when the user approves the plan (or manually).

| Vendor      | Term      | How to enter                                                    |
| ----------- | --------- | --------------------------------------------------------------- |
| Claude Code | plan mode | `permissionMode: "plan"` or in-session toggle                   |
| Codex CLI   | Plan mode | `/plan [prompt]` or `plan_mode_reasoning_effort` in config.toml |
| Gemini CLI  | Plan mode | `/plan [goal]`, `Shift+Tab`, or `gemini --approval-mode=plan`   |

---

**Adding a term?** Pick a neutral name, define it in one sentence, then fill the vendor table. If one vendor doesn't have the concept at all, write `❌ no equivalent` instead of `⏳ pending`.

[cx-features]: https://developers.openai.com/codex/cli/features
