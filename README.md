# ship-it-skynet

> A reference matrix comparing agentic AI coding CLIs. Continuous deployment, continuous dread.

**Last updated:** 2026-05-21 &nbsp;·&nbsp; **Matrix last verified:** 2026-05-15 (all three tier-1 columns)

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

> ⚠️ **Gemini CLI is being sunset.** On 2026-05-19 (I/O), Google announced it is [transitioning Gemini CLI to Antigravity CLI][gm-antigravity]. Gemini CLI and the Gemini Code Assist IDE extensions stop serving free and paid **consumer** tiers on **2026-06-18**; enterprise Standard/Enterprise licenses retain access via paid API keys. The Go-based Antigravity CLI is positioned as the successor and preserves Skills, Hooks, Subagents, and Extensions (renamed "plugins"). We're keeping the Gemini CLI column live until the sunset and will migrate it to Antigravity CLI on a future `/refresh` once the [Antigravity docs][gm-antigravity] are verifiable to our primary-source bar (they're currently JS-rendered single-page apps that don't expose cell-level claims).

| Feature                      | Claude Code                                                                          | Codex CLI                                                 | Gemini CLI                                           |
| ---------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------- | ---------------------------------------------------- |
| Plan mode                    | [✅][cc-plan]                                                                        | [✅][cx-plan]                                             | [✅][gm-plan]                                        |
| Skills                       | [✅][cc-skills]                                                                      | [✅][cx-skills]                                           | [✅][gm-skills]                                      |
| Plugins / extensions         | [✅ plugins + custom marketplaces][cc-plugins]                                       | [✅ plugins + custom marketplaces][cx-plugins]            | [🟡 extensions + official gallery only][gm-plugins]  |
| Slash commands               | [✅][cc-commands]                                                                    | [✅][cx-slash]                                            | [✅][gm-slash]                                       |
| Model selection              | [✅ `/model` + `--model` + aliases][cc-model]                                        | [✅ `/model` + `--model`][cx-model]                       | [✅ `/model` + `--model`][gm-model]                  |
| Sub-agents                   | [✅][cc-subagents]                                                                   | [✅][cx-subagents]                                        | [✅][gm-subagents]                                   |
| Agent teams                  | [🟡 experimental (flag-gated)][cc-teams]                                             | [❌ hierarchical only][cx-teams]                          | [❌][gm-teams]                                       |
| Hooks                        | [✅][cc-hooks]                                                                       | [🟡 PreToolUse + PermissionRequest, bypassable][cx-hooks] | [✅][gm-hooks]                                       |
| MCP (Model Context Protocol) | [✅][cc-mcp]                                                                         | [✅][cx-mcp]                                              | [✅][gm-mcp]                                         |
| Custom tools                 | [✅ via MCP][cc-mcp]                                                                 | [✅ via MCP][cx-mcp]                                      | [✅ via MCP][gm-mcp]                                 |
| Web search / grounding       | [✅ built-in `WebSearch` + `WebFetch`][cc-web]                                       | [✅ built-in web search on by default][cx-web]            | [✅ `google_web_search` + `web_fetch`][gm-web]       |
| Permission modes             | [✅ default / acceptEdits / plan / auto / dontAsk / bypassPermissions][cc-approvals] | [✅ on-request / never / untrusted][cx-approvals]         | [✅ Default / Auto-Edit / Plan / YOLO][gm-approvals] |
| Persistent memory            | [✅ `CLAUDE.md` + auto-memory files][cc-memory]                                      | [✅ `AGENTS.md` hierarchical][cx-memory]                  | [✅ `GEMINI.md` hierarchical + JIT][gm-memory]       |
| Checkpoints / rewind         | [✅ auto-checkpoint + `/rewind`][cc-rewind]                                          | [✅ `Esc+Esc` backtrack + `codex fork`][cx-rewind]        | [✅ `/rewind` + `Esc+Esc`][gm-rewind]                |
| Worktrees / isolated sandbox | [✅ worktrees + sandbox][cc-sandbox]                                                 | [✅ sandbox][cx-sandbox]                                  | [✅ worktrees + sandbox][gm-sandbox]                 |
| Headless / non-interactive   | [✅ `claude -p` + `--bare`][cc-headless]                                             | [✅ `codex exec`][cx-headless]                            | [✅ `gemini -p`][gm-headless]                        |
| IDE integration              | [✅ VS Code, Cursor, JetBrains][cc-ide]                                              | [✅ VS Code + forks, JetBrains][cx-ide]                   | [✅ VS Code + JetBrains/Zed via ACP][gm-ide]         |
| Image / multimodal input     | [✅ `Ctrl+V` paste → `[Image #N]` chip][cc-image]                                    | [✅ `-i`/`--image` + paste][cx-image]                     | [🟡 multimodal via Gemini model][gm-image]           |

> All three tier-1 columns were re-verified against primary vendor docs on 2026-05-15. Two asymmetries worth flagging: Codex CLI's hook surface still calls out as a guardrail rather than an enforcement boundary — `PreToolUse` and `PermissionRequest` match Bash, `apply_patch` (Edit/Write aliases), and MCP tool names, but `unified_exec` streaming shell, `WebSearch`, and non-MCP tool paths still aren't intercepted, so it isn't a CI-pipeline gate (see [hooks deep-dive](docs/hooks.md)); and Claude Code remains the only tier-1 CLI with an **Agent Teams** layer (experimental, flag-gated behind `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) where teammates run as independent sessions, share a task list, and message each other directly — Codex CLI subagents are strictly hierarchical (parent → children, no peer messaging), Gemini CLI subagents explicitly cannot spawn or message each other, and neither has a shared-task-list equivalent, which is why Agent Teams earns its own matrix row.
>
> One row moved on 2026-05-15. **Plugins / extensions — Codex CLI** flipped from 🟡 to ✅: `codex plugin marketplace add` now accepts GitHub shorthand (`owner/repo[@ref]`), HTTP/HTTPS or SSH Git URLs, and local paths as first-class marketplace catalogs, alongside the existing project-scoped `.agents/plugins/marketplace.json`, user-scoped `~/.agents/plugins/marketplace.json`, and the curated official Plugin Directory in the Codex app — closing the gap with Claude Code's `/plugin marketplace add`. Gemini CLI remains at 🟡: `gemini extensions install <github-url-or-local-path>` is still one extension at a time against a single official gallery, with no custom-marketplace catalog concept. Remaining rows unchanged: all three ship a built-in web search tool (Codex on by default, Gemini via `google_web_search` with grounded citations, Claude Code via `WebSearch` + `WebFetch` documented in the Agent SDK reference), plus headless mode, IDE plugins, image paste, model switching, sub-agents, and the rewind/checkpoint UX are 3/3 parity.

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
[cx-hooks]: https://developers.openai.com/codex/hooks "Codex CLI — full lifecycle hooks (SessionStart/PreToolUse/PermissionRequest/PostToolUse/UserPromptSubmit/Stop); PreToolUse and PermissionRequest match Bash, apply_patch (Edit/Write aliases), and MCP tool names — but the docs explicitly call them a guardrail rather than an enforcement boundary (no coverage for unified_exec streaming shell, WebSearch, or non-MCP tool paths)"
[cx-mcp]: https://developers.openai.com/codex/mcp "Codex CLI — MCP servers in ~/.codex/config.toml under [mcp_servers.*] with per-tool approval overrides"
[cx-approvals]: https://developers.openai.com/codex/agent-approvals-security "Codex CLI — approval modes on-request / never / untrusted; sandbox modes read-only / workspace-write / danger-full-access"
[cc-approvals]: https://code.claude.com/docs/en/settings "Claude Code — default / acceptEdits / plan / auto / dontAsk / bypassPermissions (defaultMode setting)"
[cx-memory]: https://developers.openai.com/codex/guides/agents-md "Codex CLI — AGENTS.md loaded hierarchically from ~/.codex and git root"
[cx-sandbox]: https://developers.openai.com/codex/agent-approvals-security "Codex CLI — per-platform sandbox: Seatbelt / bubblewrap+seccomp / Landlock / Windows native"
[gm-antigravity]: https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/ "Google Developers Blog (2026-05-19) — Gemini CLI is transitioning to the Go-based Antigravity CLI; Gemini CLI + Gemini Code Assist IDE extensions stop serving free and paid consumer tiers on 2026-06-18, enterprise Standard/Enterprise retains paid-API-key access; Antigravity CLI preserves Skills, Hooks, Subagents, and Extensions (renamed plugins). Migration guide at antigravity.google/docs/gcli-migration"
[gm-plan]: https://geminicli.com/docs/cli/plan-mode/ "Gemini CLI — /plan command; Shift+Tab toggle; --approval-mode=plan"
[gm-skills]: https://geminicli.com/docs/cli/skills/ "Gemini CLI — .gemini/skills/ or .agents/skills/; activate_skill tool; Agent Skills open standard"
[gm-slash]: https://geminicli.com/docs/cli/custom-commands/ "Gemini CLI — TOML files in ~/.gemini/commands/, namespaced subdirs, {{args}} and !{...} shell injection, plus @{...} file injection"
[gm-subagents]: https://geminicli.com/docs/core/subagents/ "Gemini CLI — user-definable subagents in .gemini/agents/*.md (project) or ~/.gemini/agents/*.md (user); frontmatter: name, description, kind, tools, mcpServers, model, temperature, max_turns, timeout_mins; invoked automatically, via @name force-routing, or exposed as a same-named tool; built-ins codebase_investigator, cli_help, generalist, plus experimental browser_agent. Subagents cannot spawn or message each other"
[gm-teams]: https://geminicli.com/docs/core/subagents/ "Gemini CLI — no multi-agent team coordination; user-defined subagents explicitly cannot spawn or message each other (the * tool wildcard doesn't expose other agents), and there is no shared task list, mailbox, or lead/teammate concept; geminicli.com/docs/ has no agent-teams or multi-agent page as of 2026-05-15"
[gm-hooks]: https://geminicli.com/docs/hooks/reference/ "Gemini CLI — full lifecycle hooks (BeforeTool, AfterTool, BeforeAgent, AfterAgent, BeforeModel, AfterModel, BeforeToolSelection, PreCompress, ...); blocking and rewrite supported"
[gm-mcp]: https://geminicli.com/docs/tools/mcp-server/ "Gemini CLI — MCP servers configured in .gemini/settings.json under mcpServers; tools exposed as mcp_<server>_<tool> and match hook regex"
[gm-approvals]: https://geminicli.com/docs/cli/plan-mode/ "Gemini CLI — approval modes Default / Auto-Edit / Plan plus YOLO"
[gm-memory]: https://geminicli.com/docs/cli/gemini-md/ "Gemini CLI — GEMINI.md hierarchical loading plus JIT discovery"
[gm-sandbox]: https://geminicli.com/docs/cli/sandbox/ "Gemini CLI — worktrees + Seatbelt / Docker / Podman / Windows native / gVisor / LXC sandbox backends"
[cc-plugins]: https://code.claude.com/docs/en/plugins "Claude Code — plugins bundle skills, agents, hooks, MCP + LSP servers, commands, and settings via .claude-plugin/plugin.json; distributed through marketplaces. Official claude-plugins-official is pre-registered; /plugin marketplace add accepts arbitrary GitHub repos, git URLs, local paths, or hosted marketplace.json as custom catalogs; /plugin install <name>@<marketplace> with user/project/local/managed scopes"
[cx-plugins]: https://developers.openai.com/codex/plugins/build "Codex CLI — plugins bundle skills, apps (connectors), and MCP servers via .codex-plugin/plugin.json. codex plugin marketplace add accepts GitHub shorthand (owner/repo[@ref]), HTTP/HTTPS or SSH Git URLs, and local paths as first-class catalogs, alongside project-scoped .agents/plugins/marketplace.json, user-scoped ~/.agents/plugins/marketplace.json, and the curated official Plugin Directory in the Codex app"
[gm-plugins]: https://geminicli.com/docs/extensions/ "Gemini CLI — extensions bundle prompts, MCP servers, custom commands, themes, hooks, sub-agents, and agent skills via gemini-extension.json; gemini extensions install <github-url-or-local-path>; official gallery at geminicli.com/extensions/browse but no custom-marketplace catalog concept"
[cc-model]: https://code.claude.com/docs/en/model-config "Claude Code — /model switches mid-session; --model flag, ANTHROPIC_MODEL env, or settings.model; aliases sonnet/opus/haiku/opusplan plus [1m] variants"
[cx-model]: https://developers.openai.com/codex/cli/slash-commands "Codex CLI — /model command picks active model and reasoning effort mid-session; --model/-m flag at launch"
[gm-model]: https://geminicli.com/docs/cli/model/ "Gemini CLI — /model opens picker (Auto Gemini 3, Auto Gemini 2.5, Manual); --model flag at launch selects a specific Gemini model"
[cc-web]: https://code.claude.com/docs/en/agent-sdk/typescript "Claude Code — built-in WebSearch tool (query + allowed_domains/blocked_domains) alongside WebFetch, documented in the Agent SDK tool reference"
[cx-web]: https://developers.openai.com/codex/cli/features "Codex CLI — first-party web search enabled by default, serves from an OpenAI-maintained cache; --search flag switches to live results"
[gm-web]: https://geminicli.com/docs/tools/web-search/ "Gemini CLI — google_web_search tool returns grounded summaries with source URIs and citations via Google Search"
[cc-rewind]: https://code.claude.com/docs/en/checkpointing "Claude Code — every prompt auto-creates a checkpoint; /rewind or Esc+Esc restores code, conversation, or both"
[cx-rewind]: https://developers.openai.com/codex/cli/features "Codex CLI — Esc+Esc walks back through the transcript to edit/fork a past user message; codex fork branches saved sessions"
[gm-rewind]: https://geminicli.com/docs/cli/rewind/ "Gemini CLI — /rewind command or Esc+Esc opens the rewind menu and restores files, conversation, or both; works across chat compression points. Covers only AI-edit-tool changes, not manual edits or shell-command mutations. A separate opt-in /restore + shadow git checkpointing system still exists for pre-tool snapshots"
[cc-headless]: https://code.claude.com/docs/en/headless "Claude Code — claude -p runs non-interactively; --output-format json|stream-json and --bare for clean CI starts"
[cx-headless]: https://developers.openai.com/codex/cli/features "Codex CLI — codex exec runs non-interactively, pipes plan and results to stdout; --json for newline-delimited events"
[gm-headless]: https://geminicli.com/docs/cli/headless/ "Gemini CLI — gemini -p triggers headless mode; --output-format json|stream-json with JSONL event stream and standard exit codes"
[cc-ide]: https://code.claude.com/docs/en/vs-code "Claude Code — VS Code + Cursor extension with inline diffs, @-mentions, plan review; separate JetBrains plugin for IntelliJ, PyCharm, WebStorm"
[cx-ide]: https://developers.openai.com/codex/ide "Codex CLI — Codex IDE extension for VS Code, Cursor, Windsurf, VS Code Insiders; separate JetBrains integration for Rider, IntelliJ, PyCharm, WebStorm"
[gm-ide]: https://geminicli.com/docs/ide-integration/ "Gemini CLI — Companion extension for VS Code and forks gives workspace context and native diffing; JetBrains and Zed connect via Agent Client Protocol"
[cc-image]: https://code.claude.com/docs/en/interactive-mode "Claude Code — Ctrl+V / Cmd+V / Alt+V pastes clipboard images, inserting an [Image #N] chip you can reference positionally in the prompt"
[cx-image]: https://developers.openai.com/codex/cli/features "Codex CLI — attach screenshots with -i/--image flag or paste directly into the TUI composer; comma-separated multi-file support"
[gm-image]: https://github.com/google-gemini/gemini-cli/blob/main/README.md "Gemini CLI — multimodal input: generate apps from PDFs, images, or sketches; specific paste/attach UX not documented on a dedicated page"
