# Hooks — deep dive

> Lifecycle events, and the "can I bet a CI pipeline on this?" question.

**Last verified:** 2026-04-11

A hook is a user-defined script the harness runs at a lifecycle event. Critically, hooks run _outside_ the model — the model cannot bypass them by choosing not to run them. That's what separates a hook from a skill or a custom tool.

## Why the CI pipeline question matters

If a hook can be silently skipped, it's not a gate — it's a suggestion. A pre-commit hook that the model can decide not to run is useless for enforcing gate checks. A pre-commit hook that the harness unconditionally runs before every commit is a real guarantee.

**Short answer for the three tier-1 tools:**

- **Claude Code:** yes, bet the pipeline. Full lifecycle with blocking `PreToolUse`.
- **Gemini CLI:** yes, bet the pipeline. Full lifecycle with blocking `BeforeTool` and rewritable tool input.
- **Codex CLI:** partially. A `PreToolUse` hook can block simple Bash commands via `hooks.json`, but coverage is incomplete — it only intercepts shell, not MCP / `Write` / `WebSearch`, and the model can bypass the Bash filter by writing a script to disk and executing that instead. Not yet a deterministic CI-pipeline gate.

## Claude Code

Docs: <https://code.claude.com/docs/en/hooks>

Hooks are configured in `.claude/settings.json` (project), `.claude/settings.local.json` (gitignored local), `~/.claude/settings.json` (user), managed policy settings, or scoped to a skill or sub-agent via frontmatter.

Full event list: `SessionStart`, `SessionEnd`, `InstructionsLoaded`, `UserPromptSubmit`, `Stop`, `StopFailure`, `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`, `SubagentStart`, `SubagentStop`, `TaskCreated`, `TaskCompleted`, `CwdChanged`, `FileChanged`, `ConfigChange`, `WorktreeCreate`, `WorktreeRemove`, `PreCompact`, `PostCompact`, `Notification`, `TeammateIdle`, `Elicitation`, `ElicitationResult`.

Blocking events: `PreToolUse` (deny/allow/ask/defer via `hookSpecificOutput.permissionDecision`), `PermissionRequest`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `ConfigChange`, `Elicitation`, `ElicitationResult`, `WorktreeCreate`.

Hook handler types: `command`, `http`, `prompt`, `agent`. Command hooks receive JSON on stdin, emit JSON on stdout, and use exit code 2 to block. Matcher field supports exact strings, `|`-separated lists, or JavaScript regex (e.g. `"mcp__.*"` to match every MCP tool).

## Codex CLI

Docs: <https://developers.openai.com/codex/hooks> (primary) and <https://github.com/openai/codex/blob/main/docs/config.md> (legacy `notify` hook only)

**Five hook events:** Codex gained a full lifecycle hook surface in early 2026, expanding from the previous notify-only design. Events: `SessionStart`, `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`. Configured in `hooks.json` at `~/.codex/hooks.json` (user) or `<repo>/.codex/hooks.json` (project). The older end-of-turn `notify` hook still exists, configured separately in `config.toml` under `[notify]`.

`PreToolUse` supports blocking via `permissionDecision: "deny"` or via exit code 2, matching the Claude Code convention.

**What's still missing:**

- **Coverage gap.** `PreToolUse` only intercepts simple Bash calls. The model can bypass the Bash filter trivially by writing a shell script to disk and executing that instead, or by using MCP tools, `Write`, or `WebSearch` — none of which fire the hook.
- **No tool-input rewriting.** `PreToolUse` is all-or-nothing (allow or deny); it cannot mutate the model's arguments the way Gemini CLI's `BeforeTool` can.
- **No sub-agent lifecycle events.** Sub-agent start/stop is not hookable.

This means Codex CLI _partially_ passes the "can I bet a CI pipeline on this?" test — you can enforce a shell-command denylist, but not a comprehensive trust boundary. For deterministic enforcement across every tool, you still need to run gates outside Codex (git pre-commit hooks, CI), the same as before. The capability has narrowed the gap but not closed it.

## Gemini CLI

Docs: <https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/index.md> and <https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/reference.md>

Configured in `settings.json` (project: `.gemini/settings.json`, user: `~/.gemini/settings.json`, system: `/etc/gemini-cli/settings.json`, plus extensions). Hook handlers currently support only `type: "command"`.

Full event list: `SessionStart`, `SessionEnd`, `BeforeAgent`, `AfterAgent`, `BeforeModel`, `AfterModel`, `BeforeToolSelection`, `BeforeTool`, `AfterTool`, `PreCompress`, `Notification`.

Blocking events and what they can do:

- **`BeforeTool`** — `decision: "deny"` (or exit 2) blocks execution and feeds `reason` to the agent as a tool error. Can also rewrite arguments via `hookSpecificOutput.tool_input`, which **merges with and overrides** the model's arguments before the tool runs.
- **`AfterTool`** — `decision: "deny"` hides the real tool output and replaces it with the hook's `reason`. Can also trigger a tail-tool-call for programmatic tool routing.
- **`BeforeAgent`** — block the turn entirely (`decision: "deny"` discards the user's message; `continue: false` saves it to history).
- **`AfterAgent`** — reject the model's final response and force an automatic retry (`decision: "deny"` with a `reason` that becomes the retry prompt).
- **`continue: false`** on any hook kills the agent loop immediately.

Matcher field is regex for tool events (e.g. `"write_.*"`), exact string for lifecycle events. MCP tool names follow `mcp_<server>_<tool>`. Hooks are **fingerprinted**: if `.gemini/settings.json` changes a project hook via `git pull`, the new hook is treated as untrusted and you get a confirmation prompt before it runs — a nice defense against malicious PRs.

Environment variables in hook scripts: `GEMINI_PROJECT_DIR`, `GEMINI_SESSION_ID`, `GEMINI_CWD`. Gemini also provides `CLAUDE_PROJECT_DIR` as a compatibility alias — revealing that they expect hook authors to reuse Claude Code hook scripts.

## Regression watch

The Claude Code `↔` Gemini CLI parity is new as of the 2026-04-11 refresh. Codex CLI has narrowed the gap — a `PreToolUse` hook surface landed in early 2026 — but remains the outlier: enforcement is Bash-only and model-bypassable, so you still can't wire a comprehensive pre-tool gate.
