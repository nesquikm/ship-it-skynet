# ship-it-skynet

> A reference matrix comparing agentic AI coding CLIs. Continuous deployment, continuous dread.

**Last updated:** 2026-04-11 &nbsp;·&nbsp; **Matrix last verified:** 2026-04-11 (all three tier-1 columns)

This is not another "best AI tool 2026" listicle. It's a Rosetta stone: row-per-feature, column-per-tool lookup for developers who already use one of these CLIs and want to know what the other side has.

## Principles

- **Primary sources only.** Every non-trivial claim links to official docs, a changelog, or a commit. Nothing from Twitter.
- **Dated.** Each column has a "last verified" date. Run `/refresh` to re-verify.
- **No vibes benchmarks.** We don't rank "which writes better React." That's a blog post, not a reference.
- **Opinions live in prose, not cells.** The intro can say "Claude Code's hooks are the only implementation I'd bet a CI pipeline on." Cells just say yes / no / partial.
- **Maintainable by one person.** If updating the matrix takes more than 30 minutes, the format is wrong.

## Scope

**Tier 1** (this matrix): Claude Code, Codex CLI, Gemini CLI.

**Tier 2** (considered for deep-dives, not the main matrix): Aider, Cline, Cursor CLI, Windsurf / Codeium, Continue.dev.

**Out of scope:** IDE plugins without terminal UX (Copilot), web UIs (ChatGPT, claude.ai), model-hosting services. This is about terminal-first agents.

See [`docs/glossary.md`](docs/glossary.md) for vendor-terminology mappings — half the confusion between these tools is that "agent" means five different things.

## The matrix

**Legend:** ✅ supported · 🟡 partial · ❌ not supported · ⏳ unverified (run `/refresh`). Hover a cell for a short source description; click to open the primary source.

| Feature                      | Claude Code                                                                          | Codex CLI                                         | Gemini CLI                                           |
| ---------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------- | ---------------------------------------------------- |
| Plan mode                    | [✅][cc-plan]                                                                        | [✅][cx-plan]                                     | [✅][gm-plan]                                        |
| Skills                       | [✅][cc-skills]                                                                      | [✅][cx-skills]                                   | [✅][gm-skills]                                      |
| Slash commands               | [✅][cc-commands]                                                                    | [✅][cx-slash]                                    | [✅][gm-slash]                                       |
| Sub-agents                   | [✅][cc-subagents]                                                                   | [✅][cx-subagents]                                | [🟡 built-in only][gm-subagents]                     |
| Agent teams                  | [🟡 experimental (flag-gated)][cc-teams]                                             | [❌ hierarchical only][cx-teams]                  | [❌][gm-teams]                                       |
| Hooks                        | [✅][cc-hooks]                                                                       | [🟡 Bash-only PreToolUse + notify][cx-hooks]      | [✅][gm-hooks]                                       |
| MCP (Model Context Protocol) | [✅][cc-mcp]                                                                         | [✅][cx-mcp]                                      | [✅][gm-mcp]                                         |
| Custom tools                 | [✅ via MCP][cc-mcp]                                                                 | [✅ via MCP][cx-mcp]                              | [✅ via MCP][gm-mcp]                                 |
| Permission modes             | [✅ default / acceptEdits / plan / auto / dontAsk / bypassPermissions][cc-approvals] | [✅ on-request / never / untrusted][cx-approvals] | [✅ Default / Auto-Edit / Plan / YOLO][gm-approvals] |
| Persistent memory            | [✅ `CLAUDE.md` + auto-memory files][cc-memory]                                      | [✅ `AGENTS.md` hierarchical][cx-memory]          | [✅ `GEMINI.md` hierarchical + JIT][gm-memory]       |
| Worktrees / isolated sandbox | [✅ worktrees + sandbox][cc-sandbox]                                                 | [✅ sandbox][cx-sandbox]                          | [✅ worktrees + sandbox][gm-sandbox]                 |

> All three tier-1 columns were re-verified against primary vendor docs on 2026-04-11. Three things worth flagging: Codex CLI added a `PreToolUse` hook surface but enforcement is Bash-only and model-bypassable, so it still isn't a CI-pipeline gate (see [hooks deep-dive](docs/hooks.md)); Gemini CLI exposes sub-agents only as a pair of built-in research agents (`codebase_investigator`, `cli_help`) — users can't define their own; and Claude Code shipped an experimental **Agent Teams** mode on top of sub-agents, flag-gated behind `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, where teammates run as independent sessions, share a task list, and message each other directly. Codex CLI subagents are hierarchical only (parent → children, no peer messaging) and Gemini CLI has nothing comparable, so this is currently a Claude-Code-only capability — which is why it earns its own row instead of living as a footnote on Sub-agents.

## Deep dives

Some features don't fit in a checkmark. See prose versions:

- [Skills](docs/skills.md) — how each CLI lets you package reusable procedures
- [Sub-agents & agent teams](docs/sub-agents.md) — delegated helpers vs. coordinating peers, and why the matrix carries two rows
- [Hooks](docs/hooks.md) — lifecycle events, and the "can I bet a CI pipeline on this?" question
- [MCP](docs/mcp.md) — who actually supports Model Context Protocol, and how

## Contributing

Vendor PRs accepted — but flagged `claim added by vendor, not yet independently verified` until a maintainer re-runs `/refresh` on that row. See [CLAUDE.md](CLAUDE.md) for the project conventions.

## Maintenance

Updating the matrix should take less than 30 minutes per refresh cycle. If it doesn't, the format is wrong, not the cadence. Run `/refresh` to re-verify one tool or all of them at once.

<!-- Reference-link definitions for the matrix. Hover text is the source description; URL is the primary source. -->

[cc-skills]: https://code.claude.com/docs/en/skills "Claude Code — skills live in .claude/skills/<name>/SKILL.md; follows Agent Skills open standard"
[cc-subagents]: https://code.claude.com/docs/en/sub-agents "Claude Code — sub-agents in .claude/agents/; built-in Explore, Plan, general-purpose"
[cc-teams]: https://code.claude.com/docs/en/agent-teams "Claude Code — experimental Agent Teams (CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1, requires v2.1.32+); team lead + peer teammates, shared task list, direct inter-agent messaging, TeammateIdle/TaskCreated/TaskCompleted hooks"
[cc-plan]: https://code.claude.com/docs/en/permission-modes "Claude Code — plan mode is one of six permission modes; Shift+Tab toggle, /plan prefix, --permission-mode plan flag; read-only while planning"
[cc-commands]: https://code.claude.com/docs/en/commands "Claude Code — built-in slash commands plus user-defined commands under .claude/commands/ (now merged into skills)"
[cc-hooks]: https://code.claude.com/docs/en/hooks "Claude Code — full lifecycle hook surface configured in settings.json; command/http/prompt/agent handlers; blocking PreToolUse, PermissionRequest, and 20+ other events"
[cc-mcp]: https://code.claude.com/docs/en/mcp "Claude Code — MCP servers in .mcp.json (project) or settings.json (user); mcp__<server>__<tool> tool naming flows into permission rules and hook matchers; sub-agents can scope their own MCP stack"
[cc-memory]: https://code.claude.com/docs/en/memory "Claude Code — CLAUDE.md hierarchical loading (project/user/managed/local) + auto-memory at ~/.claude/projects/<project>/memory/; first 200 lines or 25KB of MEMORY.md loaded every session"
[cc-sandbox]: https://code.claude.com/docs/en/sandboxing "Claude Code — OS-level sandbox via Seatbelt (macOS) or bubblewrap (Linux/WSL2); filesystem + network isolation with proxy; separate --worktree flag for parallel git worktrees under .claude/worktrees/ (see /common-workflows)"
[cx-plan]: https://developers.openai.com/codex/cli/slash-commands "Codex CLI — /plan slash command; plan_mode_reasoning_effort config"
[cx-skills]: https://developers.openai.com/codex/skills "Codex CLI — .agents/skills/<name>/SKILL.md; Agent Skills open standard; progressive disclosure"
[cx-slash]: https://developers.openai.com/codex/cli/slash-commands "Codex CLI — 26 built-in slash commands; user-defined via skills"
[cx-subagents]: https://developers.openai.com/codex/cli/features "Codex CLI — subagents listed as first-class feature"
[cx-teams]: https://developers.openai.com/codex/subagents "Codex CLI — subagents are hierarchical only: parent orchestrates spawning, routing, waiting, and closing; no peer-to-peer messaging or shared task list; CSV batch mode is experimental"
[cx-hooks]: https://developers.openai.com/codex/hooks "Codex CLI — five events (SessionStart/PreToolUse/PostToolUse/UserPromptSubmit/Stop); PreToolUse is Bash-only and model-bypassable"
[cx-mcp]: https://github.com/openai/codex/blob/main/docs/config.md "Codex CLI — MCP servers in ~/.codex/config.toml with per-tool approval overrides"
[cx-approvals]: https://developers.openai.com/codex/agent-approvals-security "Codex CLI — approval modes on-request / never / untrusted; sandbox modes read-only / workspace-write / danger-full-access"
[cc-approvals]: https://code.claude.com/docs/en/settings "Claude Code — default / acceptEdits / plan / auto / dontAsk / bypassPermissions (defaultMode setting)"
[cx-memory]: https://developers.openai.com/codex/guides/agents-md "Codex CLI — AGENTS.md loaded hierarchically from ~/.codex and git root"
[cx-sandbox]: https://developers.openai.com/codex/agent-approvals-security "Codex CLI — per-platform sandbox: Seatbelt / bubblewrap+seccomp / Landlock / Windows native"
[gm-plan]: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/plan-mode.md "Gemini CLI — /plan command; Shift+Tab toggle; --approval-mode=plan"
[gm-skills]: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/skills.md "Gemini CLI — .gemini/skills/ or .agents/skills/; activate_skill tool; Agent Skills open standard"
[gm-slash]: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/custom-commands.md "Gemini CLI — TOML files in ~/.gemini/commands/, namespaced subdirs, {{args}} and !{...} shell injection"
[gm-subagents]: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/plan-mode.md "Gemini CLI — only built-in research subagents (codebase_investigator, cli_help); not user-definable"
[gm-teams]: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/plan-mode.md "Gemini CLI — no user-definable agents and no multi-agent team coordination; docs/cli/ has no agent-teams or multi-agent file as of 2026-04-11"
[gm-hooks]: https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/reference.md "Gemini CLI — full lifecycle hooks (BeforeTool, AfterTool, BeforeAgent, AfterAgent, ...); blocking and rewrite supported"
[gm-mcp]: https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/reference.md "Gemini CLI — MCP tools exposed as mcp_<server>_<tool>; match hook regex"
[gm-approvals]: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/plan-mode.md "Gemini CLI — approval modes Default / Auto-Edit / Plan plus YOLO"
[gm-memory]: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/gemini-md.md "Gemini CLI — GEMINI.md hierarchical loading plus JIT discovery"
[gm-sandbox]: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/sandbox.md "Gemini CLI — worktrees + Seatbelt / Docker / Windows / gVisor / LXC sandbox backends"
