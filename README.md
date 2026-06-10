# ship-it-skynet

> A reference matrix comparing agentic AI coding CLIs. Continuous deployment, continuous dread.

**Last updated:** 2026-06-10 &nbsp;·&nbsp; **Matrix last verified:** 2026-06-10 (all three tier-1 columns)

This is not another "best AI tool 2026" listicle. It's a Rosetta stone: row-per-feature, column-per-tool lookup for developers who already use one of these CLIs and want to know what the other side has.

## Principles

- **Primary sources only.** Every non-trivial claim links to official docs, a changelog, or a commit. Nothing from Twitter.
- **Dated.** Each column has a "last verified" date. Run `/refresh` to re-verify.
- **No vibes benchmarks.** We don't rank "which writes better React." That's a blog post, not a reference.
- **Opinions live in prose, not cells.** The intro can say "Claude Code's hooks are the only implementation I'd bet a CI pipeline on." Cells just say yes / no / partial.
- **Maintainable by one person.** If updating the matrix takes more than 30 minutes, the format is wrong.

## Scope

**Tier 1** (this matrix): Claude Code, Codex CLI, Antigravity CLI (successor to Gemini CLI, whose consumer tiers sunset 2026-06-18).

**Tier 2** (considered for deep-dives, not the main matrix): Aider, Cline, Cursor CLI, Windsurf / Codeium, Continue.dev.

**Out of scope:** IDE plugins without terminal UX (Copilot), web UIs (ChatGPT, claude.ai), model-hosting services. This is about terminal-first agents.

See [`docs/glossary.md`](docs/glossary.md) for vendor-terminology mappings — half the confusion between these tools is that "agent" means five different things.

## The matrix

**Legend:** ✅ supported · 🟡 partial · ❌ not supported · ⏳ unverified (run `/refresh`). Hover a cell for a short source description; click to open the primary source.

> ⚠️ **Column migrated: Gemini CLI → Antigravity CLI (2026-06-10).** On 2026-05-19 (I/O), Google announced the [transition of Gemini CLI to Antigravity CLI][ag-blog]. Gemini CLI and the Gemini Code Assist IDE extensions stop serving free and paid **consumer** tiers on **2026-06-18**; enterprise Standard/Enterprise licenses keep access via paid API keys (and Gemini CLI is still shipping — v0.46.0 landed the morning of the migration). The Go-based Antigravity CLI (`agy`) is the successor and, per the announcement, preserves Skills, Hooks, Subagents, and Extensions (renamed "plugins"). Its docs site is still a JS-rendered SPA we can't verify cell-by-cell, but the [official GitHub repo][ag-repo] — linked from the announcement itself — provides a verifiable README, CHANGELOG, and Releases API, which is what the column below cites. Cells marked ⏳ have no shipped primary source yet and will fill in as Antigravity documentation lands. The fully-verified pre-migration Gemini CLI column lives in git history, and the deep-dive pages keep their Gemini CLI sections as predecessor reference for enterprise users.

| Feature                      | Claude Code                                                                          | Codex CLI                                          | Antigravity CLI                                 |
| ---------------------------- | ------------------------------------------------------------------------------------ | -------------------------------------------------- | ----------------------------------------------- |
| Plan mode                    | [✅][cc-plan]                                                                        | [✅][cx-plan]                                      | ⏳                                              |
| Skills                       | [✅][cc-skills]                                                                      | [✅][cx-skills]                                    | [✅][ag-skills]                                 |
| Plugins / extensions         | [✅ plugins + custom marketplaces][cc-plugins]                                       | [✅ plugins + custom marketplaces][cx-plugins]     | [✅ plugins via GitHub installs][ag-plugins]    |
| Slash commands               | [✅][cc-commands]                                                                    | [✅][cx-slash]                                     | [✅][ag-slash]                                  |
| Model selection              | [✅ `/model` + `--model` + aliases][cc-model]                                        | [✅ `/model` + `--model`][cx-model]                | [✅ `/model` + `--model` + `models`][ag-model]  |
| Sub-agents                   | [✅][cc-subagents]                                                                   | [✅][cx-subagents]                                 | [✅][ag-subagents]                              |
| Agent teams                  | [🟡 experimental (flag-gated)][cc-teams]                                             | [❌ hierarchical only][cx-teams]                   | [⏳ orchestration announced][ag-teams]          |
| Hooks                        | [✅][cc-hooks]                                                                       | [🟡 broad lifecycle, still bypassable][cx-hooks]   | [⏳ announced][ag-hooks]                        |
| MCP (Model Context Protocol) | [✅][cc-mcp]                                                                         | [✅][cx-mcp]                                       | [✅][ag-mcp]                                    |
| Custom tools                 | [✅ via MCP][cc-mcp]                                                                 | [✅ via MCP][cx-mcp]                               | [✅ via MCP][ag-mcp]                            |
| Web search / grounding       | [✅ built-in `WebSearch` + `WebFetch`][cc-web]                                       | [✅ built-in web search on by default][cx-web]     | ⏳                                              |
| Permission modes             | [✅ default / acceptEdits / plan / auto / dontAsk / bypassPermissions][cc-approvals] | [✅ on-request / never / untrusted][cx-approvals]  | [✅ rules + `proceed-in-sandbox`][ag-approvals] |
| Persistent memory            | [✅ `CLAUDE.md` + auto-memory files][cc-memory]                                      | [✅ `AGENTS.md` hierarchical][cx-memory]           | [✅ `.md` rules + `rules.json`][ag-memory]      |
| Checkpoints / rewind         | [✅ auto-checkpoint + `/rewind`][cc-rewind]                                          | [✅ `Esc+Esc` backtrack + `codex fork`][cx-rewind] | ⏳                                              |
| Worktrees / isolated sandbox | [✅ worktrees + sandbox][cc-sandbox]                                                 | [✅ sandbox][cx-sandbox]                           | [✅ sandbox][ag-sandbox]                        |
| Headless / non-interactive   | [✅ `claude -p` + `--bare`][cc-headless]                                             | [✅ `codex exec`][cx-headless]                     | [✅ `-p` / `--print`][ag-headless]              |
| IDE integration              | [✅ VS Code, Cursor, JetBrains][cc-ide]                                              | [✅ VS Code + forks, JetBrains][cx-ide]            | [🟡 Antigravity 2.0 session export][ag-ide]     |
| Image / multimodal input     | [✅ `Ctrl+V` paste → `[Image #N]` chip][cc-image]                                    | [✅ `-i`/`--image` + paste][cx-image]              | [✅ clipboard paste][ag-image]                  |

> All three tier-1 columns were re-verified against primary vendor docs on 2026-06-10. Two asymmetries worth flagging: Codex CLI's hook surface keeps expanding — now ten lifecycle events enabled by default, including `SubagentStart`/`SubagentStop` and compaction events, plus `updatedInput` tool-input rewriting on `PreToolUse` — but the docs still call it a guardrail rather than an enforcement boundary: `unified_exec` streaming shell, `WebSearch`, and non-MCP tool paths still aren't intercepted, so it isn't a CI-pipeline gate (see [hooks deep-dive](docs/hooks.md)); and Claude Code remains the only tier-1 CLI with an **Agent Teams** layer (experimental, flag-gated behind `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) where teammates run as independent sessions, share a task list, and message each other directly — Codex CLI subagents are strictly hierarchical (parent → children, no peer messaging), and Antigravity CLI's announcement touts background multi-agent orchestration without publishing peer-messaging or shared-task-list semantics (its predecessor Gemini CLI was a verified ❌), which is why Agent Teams earns its own matrix row.
>
> The 2026-06-10 column swap: **Gemini CLI → Antigravity CLI** (`agy`, v1.0.7). Fourteen of nineteen rows are verifiable today from the official repo's README and CHANGELOG: skills (custom-skill discovery, skill-derived slash commands), plugins (GitHub-subpath installs), slash commands, `/model` + `--model` + a `models` subcommand, subagents (plugin-discovered specialized agents, per-subagent timeouts), MCP (`config/mcp_config.json`, stdio or `url` servers), permission rules (three merged config levels plus a `proceed-in-sandbox` mode), `.md` rules files with `rules.json` allowlists, sandbox enforcement, headless `-p`/`--print`, Antigravity 2.0 session export, and clipboard image paste. Five rows wait on real documentation: plan mode, agent teams, hooks (announced as preserved, zero event-level docs), web search, and checkpoints/rewind. Antigravity visibly inherits Gemini DNA (`~/.gemini/` config paths, `${extensionPath}` extension variables, Gemini models underneath) — but inheritance isn't evidence, so those five stay ⏳ until Google ships crawlable docs. Same run, same date: the Codex CLI hooks story moved underneath its 🟡 — hooks are now enabled by default, the event surface grew six → ten, and `PreToolUse` gained `updatedInput` rewriting — and Claude Code's hook catalog grew to 30 events.

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
[cc-sandbox]: https://code.claude.com/docs/en/sandboxing "Claude Code — OS-level sandbox via Seatbelt (macOS) or bubblewrap (Linux/WSL2); filesystem + network isolation with proxy; separate --worktree flag (or isolation: worktree in subagent frontmatter) for parallel git worktrees under .claude/worktrees/ (see /docs/en/worktrees)"
[cx-plan]: https://developers.openai.com/codex/cli/slash-commands "Codex CLI — /plan slash command; plan_mode_reasoning_effort config"
[cx-skills]: https://developers.openai.com/codex/skills "Codex CLI — .agents/skills/<name>/SKILL.md; Agent Skills open standard; progressive disclosure"
[cx-slash]: https://developers.openai.com/codex/cli/slash-commands "Codex CLI — built-in slash commands documented in the linked table; user-defined via skills"
[cx-subagents]: https://developers.openai.com/codex/cli/features "Codex CLI — subagents listed as first-class feature"
[cx-teams]: https://developers.openai.com/codex/subagents "Codex CLI — subagents are hierarchical only: parent orchestrates spawning, routing, waiting, and closing; no peer-to-peer messaging or shared task list; CSV batch mode is experimental"
[cx-hooks]: https://developers.openai.com/codex/hooks "Codex CLI — lifecycle hooks enabled by default (SessionStart/SubagentStart at thread scope; PreToolUse/PermissionRequest/PostToolUse/UserPromptSubmit/SubagentStop/PreCompact/PostCompact/Stop at turn scope); PreToolUse can deny or rewrite tool input via updatedInput for Bash, apply_patch (Edit/Write aliases), and MCP tools — but the docs still call hooks a guardrail rather than a complete enforcement boundary (no coverage for unified_exec streaming shell, WebSearch, or non-MCP tool paths)"
[cx-mcp]: https://developers.openai.com/codex/mcp "Codex CLI — MCP servers in ~/.codex/config.toml (or project-scoped .codex/config.toml in trusted projects) under [mcp_servers.*] with per-tool approval overrides; managed via the codex mcp CLI"
[cx-approvals]: https://developers.openai.com/codex/agent-approvals-security "Codex CLI — approval modes on-request / never / untrusted plus a granular policy object; sandbox modes read-only / workspace-write / danger-full-access"
[cc-approvals]: https://code.claude.com/docs/en/settings "Claude Code — default / acceptEdits / plan / auto / dontAsk / bypassPermissions (defaultMode setting)"
[cx-memory]: https://developers.openai.com/codex/guides/agents-md "Codex CLI — AGENTS.md loaded hierarchically from ~/.codex and git root"
[cx-sandbox]: https://developers.openai.com/codex/agent-approvals-security "Codex CLI — per-platform sandbox: Seatbelt / bubblewrap+seccomp / Landlock / Windows native"
[ag-blog]: https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/ "Google Developers Blog (2026-05-19, I/O) — Gemini CLI transitions to the Go-based Antigravity CLI; consumer tiers end 2026-06-18, enterprise Standard/Enterprise keeps paid-API-key access; Antigravity CLI preserves Skills, Hooks, Subagents, and Extensions (renamed plugins)"
[ag-repo]: https://github.com/google-antigravity/antigravity-cli "Antigravity CLI — official repo, linked from the transition blog post; README, CHANGELOG, and the GitHub Releases API are the verifiable primary sources while antigravity.google/docs remains a JS-rendered SPA (no .md endpoints, no llms.txt, 26-URL sitemap with no CLI doc pages — checked 2026-06-10)"
[ag-skills]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — custom/fallback skill discovery including Standalone mode (1.0.2) and skill-derived slash commands executing from autocomplete (1.0.4); Agent Skills named as preserved in the transition blog"
[ag-slash]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — built-in slash commands (/help, /model, /settings, /resume, /diff, /permissions, /statusline, /usage, /quota, /credits, ...) with fuzzy matching and alias resolution (1.0.6/1.0.7); user-defined commands via skill-derived slash commands (1.0.4)"
[ag-subagents]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — per-subagent interaction timeouts (1.0.2), subagent conversations filtered from /resume (1.0.6), specialized agents auto-discovered from installed plugins (1.0.1); Subagents named as preserved in the transition blog"
[ag-teams]: https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/ "Antigravity CLI — the transition blog touts asynchronous workflows ('orchestrates multiple agents for complex tasks in the background'), but peer messaging, shared task lists, and user-addressable teammates have no published docs — unverified"
[ag-hooks]: https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/ "Antigravity CLI — Hooks named as preserved from Gemini CLI in the transition blog; no event catalog, matcher semantics, or blocking model published anywhere fetchable — unverified"
[ag-mcp]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — MCP servers in config/mcp_config.json with stdio or url transports (1.0.5), configurable launch timeout (1.0.7), parallelized initialization (1.0.4), per-server disable from the TUI (1.0.3)"
[ag-approvals]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — /permissions edits rules across config levels (1.0.5); permissions merge project, shared-with-Antigravity user, and CLI settings.json (1.0.7); proceed-in-sandbox mode auto-approves sandboxed commands (1.0.1)"
[ag-memory]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — discovery engine loads .md rule files at boot, with exclusion rules and allowlists in rules.json (1.0.4); file-naming conventions and hierarchy not yet documented"
[ag-sandbox]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — --sandbox flag enforced in interactive and headless modes (1.0.6), Windows sandbox network proxy fixes (1.0.7), proceed-in-sandbox permission mode (1.0.1); sandbox backends undocumented, no worktree feature documented"
[cc-plugins]: https://code.claude.com/docs/en/plugins "Claude Code — plugins bundle skills, agents, hooks, MCP + LSP servers, commands, and settings via .claude-plugin/plugin.json; distributed through marketplaces. Official claude-plugins-official is pre-registered; /plugin marketplace add accepts arbitrary GitHub repos, git URLs, local paths, or hosted marketplace.json as custom catalogs; /plugin install <name>@<marketplace> with user/project/local/managed scopes"
[cx-plugins]: https://developers.openai.com/codex/plugins/build "Codex CLI — plugins bundle skills, apps (connectors), and MCP servers via .codex-plugin/plugin.json. codex plugin marketplace add accepts GitHub shorthand (owner/repo[@ref]), HTTP/HTTPS or SSH Git URLs, and local paths as first-class catalogs, alongside project-scoped .agents/plugins/marketplace.json, user-scoped ~/.agents/plugins/marketplace.json, and the curated official Plugin Directory in the Codex app"
[ag-plugins]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — plugin subcommand installs into the shared ~/.gemini/config/ directory (1.0.2), plugin discovery makes bundled skills and agents executable (1.0.1), installs from GitHub subpaths with branch resolution (1.0.7); no custom-marketplace catalog documented yet"
[cc-model]: https://code.claude.com/docs/en/model-config "Claude Code — /model switches mid-session; --model flag, ANTHROPIC_MODEL env, or settings.model; aliases sonnet/opus/haiku/opusplan plus [1m] variants"
[cx-model]: https://developers.openai.com/codex/cli/slash-commands "Codex CLI — /model command picks active model and reasoning effort mid-session; --model/-m flag at launch"
[ag-model]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — --model launch flag and a models listing subcommand (1.0.5); /model command in the TUI; Gemini models underneath (tool-call limits raised for Gemini models in 1.0.7)"
[cc-web]: https://code.claude.com/docs/en/agent-sdk/typescript "Claude Code — built-in WebSearch tool (query + allowed_domains/blocked_domains) alongside WebFetch, documented in the Agent SDK tool reference"
[cx-web]: https://developers.openai.com/codex/cli/features "Codex CLI — first-party web search enabled by default, serves from an OpenAI-maintained cache; --search flag switches to live results"
[cc-rewind]: https://code.claude.com/docs/en/checkpointing "Claude Code — every prompt auto-creates a checkpoint; /rewind or Esc+Esc restores code, conversation, or both"
[cx-rewind]: https://developers.openai.com/codex/cli/features "Codex CLI — Esc+Esc walks back through the transcript to edit/fork a past user message; /fork clones the current conversation into a new thread; codex fork branches saved sessions"
[cc-headless]: https://code.claude.com/docs/en/headless "Claude Code — claude -p runs non-interactively; --output-format json|stream-json and --bare for clean CI starts"
[cx-headless]: https://developers.openai.com/codex/cli/features "Codex CLI — codex exec runs non-interactively, pipes plan and results to stdout; --json for newline-delimited events"
[ag-headless]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — headless print mode via -p / --print with --sandbox flag propagation (1.0.6); headless metadata cached under ~/.gemini/antigravity-cli/cache (1.0.5); output-format options not yet documented"
[cc-ide]: https://code.claude.com/docs/en/vs-code "Claude Code — VS Code + Cursor extension with inline diffs, @-mentions, plan review; separate JetBrains plugin for IntelliJ, PyCharm, WebStorm"
[cx-ide]: https://developers.openai.com/codex/ide "Codex CLI — Codex IDE extension for VS Code, Cursor, Windsurf, VS Code Insiders; separate JetBrains integration for Rider, IntelliJ, PyCharm, WebStorm"
[ag-ide]: https://github.com/google-antigravity/antigravity-cli "Antigravity CLI — shared agent engine and bidirectional settings sync with the Antigravity 2.0 desktop app, plus session export from terminal to the GUI (repo README); no third-party IDE (VS Code / JetBrains) integration documented"
[cc-image]: https://code.claude.com/docs/en/interactive-mode "Claude Code — Ctrl+V / Cmd+V / Alt+V pastes clipboard images, inserting an [Image #N] chip you can reference positionally in the prompt"
[cx-image]: https://developers.openai.com/codex/cli/features "Codex CLI — attach screenshots with -i/--image flag or paste directly into the TUI composer; comma-separated multi-file support"
[ag-image]: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md "Antigravity CLI — clipboard pipeline handles raw image data with native Wayland (wl-paste) and X11 (xclip) support, preferring copied files from file managers (1.0.7)"
