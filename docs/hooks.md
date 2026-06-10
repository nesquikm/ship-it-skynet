# Hooks — deep dive

> Lifecycle events, and the "can I bet a CI pipeline on this?" question.

**Last verified:** 2026-06-10

A hook is a user-defined script the harness runs at a lifecycle event. Critically, hooks run _outside_ the model — the model cannot bypass them by choosing not to run them. That's what separates a hook from a skill or a custom tool.

## Why the CI pipeline question matters

If a hook can be silently skipped, it's not a gate — it's a suggestion. A pre-commit hook that the model can decide not to run is useless for enforcing gate checks. A pre-commit hook that the harness unconditionally runs before every commit is a real guarantee.

**Short answer for the three tier-1 tools:**

- **Claude Code:** yes, bet the pipeline. Full lifecycle with blocking `PreToolUse`.
- **Antigravity CLI:** unknown. Hooks are announced as preserved from Gemini CLI, but no event catalog or blocking semantics have been published anywhere fetchable. The predecessor Gemini CLI was a clear "yes, bet the pipeline" (full lifecycle, blocking `BeforeTool`, rewritable tool input) — an answer that remains valid for enterprise users still on Gemini CLI.
- **Codex CLI:** partially. `PreToolUse` and `PermissionRequest` intercept Bash, `apply_patch` (with `Edit` / `Write` matcher aliases), and MCP tool calls, with structured `allow` / `deny` decisions — and `PreToolUse` can now rewrite tool input via `updatedInput`, closing what used to be a Gemini-only capability. Coverage still doesn't extend to `WebSearch` or the streaming `unified_exec` shell mechanism, and the docs explicitly call hooks "a guardrail rather than a complete enforcement boundary because Codex can often perform equivalent work through another supported tool path." Not yet a deterministic CI-pipeline gate.

## Claude Code

Docs: <https://code.claude.com/docs/en/hooks>

Hooks are configured in `.claude/settings.json` (project), `.claude/settings.local.json` (gitignored local), `~/.claude/settings.json` (user), managed policy settings, or scoped to a skill or sub-agent via frontmatter.

Full event list (30 as of 2026-06-10): `SessionStart`, `Setup`, `UserPromptSubmit`, `UserPromptExpansion`, `PreToolUse`, `PermissionRequest`, `PermissionDenied`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`, `Notification`, `MessageDisplay`, `SubagentStart`, `SubagentStop`, `TaskCreated`, `TaskCompleted`, `Stop`, `StopFailure`, `TeammateIdle`, `InstructionsLoaded`, `ConfigChange`, `CwdChanged`, `FileChanged`, `WorktreeCreate`, `WorktreeRemove`, `PreCompact`, `PostCompact`, `Elicitation`, `ElicitationResult`, `SessionEnd`. Recent additions: `Setup` (fires only with `--init-only`, or `--init`/`--maintenance` in `-p` mode), `UserPromptExpansion` (when a typed command expands into a prompt — can block the expansion), `MessageDisplay` (display-only, while assistant text streams), and `PostToolBatch` (after a parallel tool batch resolves, before the next model call).

Blocking events: `PreToolUse` (deny/allow/ask/defer via `hookSpecificOutput.permissionDecision`), `PermissionRequest`, `UserPromptSubmit`, `UserPromptExpansion`, `Stop`, `SubagentStop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `ConfigChange`, `PostToolBatch` (stops the agentic loop before the next model call), `PreCompact`, `Elicitation`, `ElicitationResult`, `WorktreeCreate`.

Hook handler types: `command`, `http`, `prompt`, `agent`. Command hooks receive JSON on stdin, emit JSON on stdout, and use exit code 2 to block. Matcher field supports exact strings, `|`-separated lists, or JavaScript regex (e.g. `"mcp__.*"` to match every MCP tool).

## Codex CLI

Docs: <https://developers.openai.com/codex/hooks> (primary) and <https://developers.openai.com/codex/config-advanced> (legacy `notify` hook only)

**Full lifecycle events:** Codex gained a full lifecycle hook surface in early 2026, expanding from the previous notify-only design, and has kept growing. Current events (10 as of 2026-06-10): `SessionStart` and `SubagentStart` at thread scope; `PreToolUse`, `PermissionRequest`, `PostToolUse`, `UserPromptSubmit`, `SubagentStop`, `PreCompact`, `PostCompact`, and `Stop` at turn scope. Configured in `hooks.json` at `~/.codex/hooks.json` (user) or `<repo>/.codex/hooks.json` (project), or inline `[hooks]` tables in `config.toml`; plugins can also bundle hooks via their manifest. Hooks are now **enabled by default** — disable with `[features] hooks = false` (`codex_hooks` survives as a deprecated alias). The older end-of-turn `notify` hook still exists, configured separately in `config.toml` under `[notify]`.

`PreToolUse` supports blocking via `permissionDecision: "deny"` or via exit code 2, matching the Claude Code convention. It can also **rewrite tool input**: return `permissionDecision: "allow"` with `updatedInput` — a replacement `command` string for Bash and `apply_patch`, or a replacement arguments object for MCP tools (`updatedInput` with any other decision is reported as an error). Hooks can additionally inject `additionalContext` on `PreToolUse` and `SubagentStart`. `PermissionRequest` is a separate event that runs when Codex is about to surface an approval prompt: hooks return a structured `decision.behavior: "allow"` or `"deny"`, with `deny` winning when multiple matchers fire. Matchers on `PreToolUse`, `PermissionRequest`, and `PostToolUse` accept `Bash`, `apply_patch` (with `Edit` / `Write` aliases), and MCP tool names like `mcp__server__tool` or regexes like `mcp__filesystem__.*`.

**What's still missing:**

- **Coverage gap.** Matcher coverage now extends beyond Bash to `apply_patch` and MCP tools, but the docs are explicit: "this doesn't intercept all shell calls yet, only the simple ones. The newer `unified_exec` mechanism allows richer streaming stdin/stdout handling of shell, but interception is incomplete. Similarly, this doesn't intercept `WebSearch` or other non-shell, non-MCP tool calls." A model that wants to bypass a Bash matcher can route through `unified_exec` or pick a tool path the matcher doesn't see.
- **Uneven decision-control fields.** `continue`, `stopReason`, and `suppressOutput` aren't supported on `PreToolUse` / `PermissionRequest` (returning them marks the hook errored), `suppressOutput` is "parsed today but not yet implemented" everywhere else, and `continue: false` on `SubagentStart` doesn't stop the subagent from starting. Tool-input rewriting via `updatedInput` works, but only on the same Bash / `apply_patch` / MCP paths the coverage gap already limits.
- **Sub-agent spawns can't be vetoed.** `SubagentStart` / `SubagentStop` landed — and `SubagentStop` can block (`decision: "block"` or exit 2 keeps the subagent working, matching Claude Code) — but `SubagentStart` is observe-and-annotate only: `continue: false` is parsed and explicitly "doesn't stop the subagent from starting."

This means Codex CLI _partially_ passes the "can I bet a CI pipeline on this?" test — you can enforce denylists and rewrite arguments across Bash, `apply_patch`, and MCP tool calls, plus reject approval requests at `PermissionRequest` time, but not a comprehensive trust boundary. For deterministic enforcement across every tool, you still need to run gates outside Codex (git pre-commit hooks, CI), the same as before. Each refresh narrows the gap; the `unified_exec` / `WebSearch` bypass keeps it open.

## Antigravity CLI (successor to Gemini CLI)

Sources: the [transition blog](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/) names Hooks as one of the four Gemini CLI features Antigravity CLI preserves. That is currently the **entire** published hooks story: the [repo CHANGELOG](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md) (through v1.0.7) never mentions hook events, and `antigravity.google/docs` is a JS-rendered SPA we can't cite. Event catalog, matchers, blocking semantics, input rewriting, fingerprinting — all unknown. Until Google ships docs, the matrix cell stays ⏳ and the CI-pipeline question has no answer for `agy`.

### Predecessor: Gemini CLI (consumer sunset 2026-06-18; enterprise still served)

Docs: <https://geminicli.com/docs/hooks/> and <https://geminicli.com/docs/hooks/reference/>

Configured in `settings.json` (project: `.gemini/settings.json`, user: `~/.gemini/settings.json`, system: `/etc/gemini-cli/settings.json`, plus extensions). Hook handlers currently support only `type: "command"`.

Full event list (verified 2026-06-10): `SessionStart`, `SessionEnd`, `BeforeAgent`, `AfterAgent`, `BeforeModel`, `AfterModel`, `BeforeToolSelection`, `BeforeTool`, `AfterTool`, `PreCompress`, `Notification`.

Blocking events and what they can do:

- **`BeforeTool`** — `decision: "deny"` (or exit 2) blocks execution and feeds `reason` to the agent as a tool error. Can also rewrite arguments via `hookSpecificOutput.tool_input`, which **merges with and overrides** the model's arguments before the tool runs.
- **`AfterTool`** — `decision: "deny"` hides the real tool output and replaces it with the hook's `reason`. Can also trigger a tail-tool-call for programmatic tool routing.
- **`BeforeAgent`** — block the turn entirely (`decision: "deny"` discards the user's message; `continue: false` saves it to history).
- **`AfterAgent`** — reject the model's final response and force an automatic retry (`decision: "deny"` with a `reason` that becomes the retry prompt).
- **`continue: false`** on any hook kills the agent loop immediately.

Matcher field is regex for tool events (e.g. `"write_.*"`), exact string for lifecycle events. MCP tool names follow `mcp_<server>_<tool>`. Hooks are **fingerprinted**: if `.gemini/settings.json` changes a project hook via `git pull`, the new hook is treated as untrusted and you get a confirmation prompt before it runs — a nice defense against malicious PRs.

Environment variables in hook scripts: `GEMINI_PROJECT_DIR`, `GEMINI_SESSION_ID`, `GEMINI_CWD`. Gemini also provides `CLAUDE_PROJECT_DIR` as a compatibility alias — revealing that they expect hook authors to reuse Claude Code hook scripts.

## Regression watch

The Claude Code `↔` Gemini CLI parity held from the 2026-04-11 refresh until the Gemini column retired on 2026-06-10; whether Antigravity CLI restores that parity is now the biggest open question on this page — hooks are announced but entirely undocumented. Codex CLI keeps converging — by 2026-06-10 it had sub-agent lifecycle events, compaction events, hooks on by default, and `updatedInput` rewriting, leaving only one structural difference — but that difference is the load-bearing one: enforcement is still model-bypassable (no `unified_exec`, no `WebSearch` interception), so you still can't wire a comprehensive pre-tool gate.
