# Sub-agents & agent teams — deep dive

> Two rows in the matrix, one spectrum: delegated helpers at one end, coordinating peers at the other.

**Last verified:** 2026-04-11

A "sub-agent" in the neutral sense is a helper agent the parent can spawn to do focused work in its own context window, then fold the result back into the main conversation. All three tier-1 CLIs implement some version of this. The deeper question — and the reason this deep-dive exists — is whether the helpers can _talk to each other_, claim tasks from a shared queue, and be driven individually by the user. That's where the three tools diverge, and it's why the matrix carries two separate rows (Sub-agents and Agent teams) instead of one.

## Sub-agents vs. agent teams

| Axis             | Sub-agent model                                       | Agent-team model                                               |
| :--------------- | :---------------------------------------------------- | :------------------------------------------------------------- |
| Context          | Own context window; results returned to caller        | Own context window; fully independent                          |
| Communication    | Helpers only report to the parent                     | Peers message each other directly                              |
| Coordination     | Parent owns the work list                             | Shared task list with file-locked claiming                     |
| User interaction | Only through the parent                               | Can message any teammate directly                              |
| Best for         | Focused delegated tasks where only the result matters | Research, debate, parallel implementation requiring cross-talk |
| Token cost       | Lower (results summarized back to parent)             | Higher (each teammate is a full session)                       |

The sub-agent row is binary: does the tool let you spawn helpers? The agent-team row asks a stricter question: can the helpers operate as peers without the parent brokering every message?

## Claude Code

Docs: <https://code.claude.com/docs/en/sub-agents> and <https://code.claude.com/docs/en/agent-teams>

### Sub-agents

Sub-agents live in `.claude/agents/<name>.md` (project), `~/.claude/agents/<name>.md` (user), plugin-bundled, or scoped to a skill via `agent:` frontmatter. Each agent file carries YAML frontmatter (`name`, `description`, `tools`, `model`, `mcpServers`, `skills`, `permissionMode`, `isolation: worktree`) plus a system-prompt body. Built-in types include `Explore`, `Plan`, and `general-purpose`.

The parent invokes a sub-agent via the `Agent` tool. The sub-agent runs in its own context window, can receive its own MCP stack (inline definitions are scoped; string references share the parent's connection), and reports results back. Sub-agent lifecycle is hookable: `SubagentStart`, `SubagentStop`. Sub-agents can even run in isolated git worktrees by setting `isolation: worktree` in frontmatter — parallel work without stepping on each other's files.

### Agent teams (experimental)

Agent teams are a layer on top of sub-agents. A team has a **lead** (the session that created the team), **teammates** (independent Claude Code sessions, each in its own context window), a **shared task list**, and a **mailbox** for direct peer messaging. Config lives in `~/.claude/teams/<team-name>/config.json`; tasks live in `~/.claude/tasks/<team-name>/`.

Enable with the experimental flag:

```json settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Requires Claude Code v2.1.32 or later. Key differences from plain sub-agents:

- **Peer messaging.** Any teammate can message any other by name. `message` is point-to-point; `broadcast` hits the whole team. The lead isn't a relay — messages are delivered automatically.
- **Shared task list.** Tasks have `pending` / `in progress` / `completed` states plus dependency tracking. Teammates self-claim with file locking to prevent races.
- **Direct user interaction.** The user can cycle between teammates (`Shift+Down` in in-process mode) or interact via split panes (tmux / iTerm2 `it2` CLI). You can message, interrupt, or plan-approve any teammate individually.
- **Plan approval.** Teammates can be required to plan before implementing; the lead approves or rejects with feedback before the teammate exits plan mode.
- **Team-specific hooks.** `TeammateIdle` (exit 2 to keep the teammate working), `TaskCreated` (exit 2 to block creation), `TaskCompleted` (exit 2 to block completion) — all can return feedback strings.
- **Sub-agent definitions as teammates.** Spawn by referencing a sub-agent type by name; the definition's `tools` allowlist and `model` are honored, and the body is _appended_ to the teammate's system prompt rather than replacing it. Note: `skills` and `mcpServers` from the definition are **not** applied when run as a teammate — teammates load those from project/user settings.

Current limitations worth knowing before you bet a workflow on this:

- `/resume` and `/rewind` do not restore in-process teammates; the lead may attempt to message ghosts after resume.
- Task status can lag; a stuck task may need a manual nudge.
- One team per session. Nested teams are not allowed. Leadership is fixed at spawn — you can't promote a teammate.
- Split-pane mode requires tmux or iTerm2; VS Code's integrated terminal, Windows Terminal, and Ghostty are in-process only.

## Codex CLI

Docs: <https://developers.openai.com/codex/subagents>

Codex CLI has sub-agents but no agent-teams equivalent. Sub-agents are defined as standalone TOML files under `~/.codex/agents/` (user) or `.codex/agents/` (project). Global settings live in `config.toml` under `[agents]`.

The model is strictly hierarchical. Quoting the Codex docs: _"Codex handles orchestration across agents, including spawning new subagents, routing follow-up instructions, waiting for results, and closing agent threads."_ The parent spawns, routes, waits, and closes — there is no peer-to-peer messaging and no shared task list. The user does get direct visibility via the `/agent` slash command (switch between active agent threads, inspect the ongoing thread, open a sub-agent to answer a request before the parent does), which is more than Claude Code's sub-agent UX offers, but the sub-agent still reports _only_ to its parent.

Two features narrow the gap slightly but don't cross it:

- **`spawn_agents_on_csv`** — a batch mode that assigns one CSV row per worker. This is experimental and still parent-orchestrated; there's no shared queue and no messaging between the workers.
- **Consolidated returns** — when many agents run, Codex waits until all requested results are available and returns a consolidated response. That's a coordination convenience, not peer coordination.

No `TeammateIdle` equivalent, no shared mailbox, no "Agent Teams" page in the docs. In the matrix this is a clean `❌` on the Agent teams row despite a clean `✅` on the Sub-agents row.

## Gemini CLI

Docs: <https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/plan-mode.md>

Gemini CLI is the outlier on both rows. User-definable sub-agents don't exist; the only sub-agents are two built-in research helpers — `codebase_investigator` and `cli_help` — exposed through plan mode. You can't add your own, you can't customize their tool allowlists, and there is nothing that resembles a team, a mailbox, or a shared task list. `docs/cli/` has no `subagents.md`, `agents.md`, `teams.md`, or equivalent file as of 2026-04-11.

That lands Gemini at `🟡 built-in only` on the Sub-agents row and a clean `❌` on the Agent teams row. If Google ships user-defined sub-agents, or adds peer coordination of any kind, both cells move and this section gets rewritten.

## What to check when refreshing

- Has Claude Code's `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` flag been retired? If Agent Teams goes GA, the matrix cell moves from `🟡 experimental` to `✅` and this deep-dive should drop the flag requirement.
- Has Codex added anything resembling a shared task list, peer messaging, or a dedicated teams page? The `/codex/subagents` page is where it would land first.
- Does Gemini CLI have a new `docs/cli/subagents.md` or `agents.md`? That would be the signal that user-defined sub-agents have shipped.
- Do any new events join Claude Code's hook catalog under the Team\* namespace? The existing set is `TeammateIdle`, `TaskCreated`, `TaskCompleted`; additions belong in the Claude Code section above.
- Are the command `/resume` + in-process-teammates limitations still open? Each limitation in the list above is worth re-checking on every refresh, since the feature is experimental by Anthropic's own label.

## Why this deserved its own page

The matrix can say `✅` or `🟡` or `❌`, but it can't capture the _architectural_ difference between "the parent spawns a helper and waits for a string back" and "five independent sessions claim tasks from a shared queue and debate each other by name." That's what this deep-dive exists for. If you're evaluating the three CLIs for work that genuinely benefits from parallel exploration with cross-talk — research, competing-hypothesis debugging, cross-layer refactors where each layer has a different owner — only Claude Code currently lets you set it up, and even there it's still flag-gated.
