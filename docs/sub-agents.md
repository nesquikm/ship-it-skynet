# Sub-agents & agent teams — deep dive

> Two rows in the matrix, one spectrum: delegated helpers at one end, coordinating peers at the other.

**Last verified:** 2026-04-19

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

Docs: <https://geminicli.com/docs/core/subagents/> and <https://geminicli.com/docs/core/remote-agents/>

User-definable sub-agents live in `.gemini/agents/*.md` (project) or `~/.gemini/agents/*.md` (user). Each agent is a markdown file with YAML frontmatter plus a body that becomes the sub-agent's system prompt. Supported frontmatter: `name` and `description` (required), `kind` (defaults to `local`), `tools` (explicit list or wildcards like `*`, `mcp_*`, `mcp_server_*`; omit to inherit the parent's toolset), `mcpServers` (inline MCP config), `model` (defaults to `inherit`), `temperature` (default `1`), `max_turns` (default `30`), `timeout_mins` (default `10`). Invocation happens three ways: automatic selection by the main agent, explicit force-routing via `@subagent_name`, or calling the sub-agent as a same-named tool.

Built-in agents: `codebase_investigator` (codebase analysis), `cli_help` (CLI documentation lookup), `generalist` (all-tools helper for complex multi-step tasks), and `browser_agent` (web automation, experimental and disabled by default). Discovery tiers are workspace > user > extension.

The Gemini sub-agent model is deliberately non-recursive: _"subagents cannot call other subagents. If a subagent is granted the `*` tool wildcard, it will still be unable to see or invoke other agents."_ There is no shared task list, no inter-agent messaging, and no `SubagentStart` / `SubagentStop` hook in the 11-event hook catalog. That lands Gemini at `✅` on the Sub-agents row and a clean `❌` on the Agent teams row.

**Remote agents** (`geminicli.com/docs/core/remote-agents/`) are a separate feature and don't change the Agent teams row. They use the same `.gemini/agents/*.md` location but specify an `agent_card_url` (or inline `agent_card_json`) pointing at an external service that speaks the Agent-to-Agent (A2A) protocol. Gemini CLI acts as a client delegating tasks unidirectionally; there is no peer-to-peer messaging between remote agents, no shared task list, and no lead/teammate relationship. Supported auth schemes include API keys, OAuth 2.0, Google credentials, and HTTP Basic.

## What to check when refreshing

- Has Claude Code's `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` flag been retired? If Agent Teams goes GA, the matrix cell moves from `🟡 experimental` to `✅` and this deep-dive should drop the flag requirement.
- Has Codex added anything resembling a shared task list, peer messaging, or a dedicated teams page? The `/codex/subagents` page is where it would land first.
- Has Gemini CLI added peer messaging, a shared task list, or recursive sub-agent spawning? Those would land at `geminicli.com/docs/core/subagents/` (changes to the recursion guard) or a new `agent-teams/` page. Any such shift would flip the Agent teams cell.
- Do any new events join Claude Code's hook catalog under the Team\* namespace? The existing set is `TeammateIdle`, `TaskCreated`, `TaskCompleted`; additions belong in the Claude Code section above.
- Are the command `/resume` + in-process-teammates limitations still open? Each limitation in the list above is worth re-checking on every refresh, since the feature is experimental by Anthropic's own label.
- Have Gemini CLI sub-agent hooks (e.g. `SubagentStart`/`SubagentStop`) landed? The hook catalog at `geminicli.com/docs/hooks/` is the source of truth for this.

## Why this deserved its own page

The matrix can say `✅` or `🟡` or `❌`, but it can't capture the _architectural_ difference between "the parent spawns a helper and waits for a string back" and "five independent sessions claim tasks from a shared queue and debate each other by name." That's what this deep-dive exists for. If you're evaluating the three CLIs for work that genuinely benefits from parallel exploration with cross-talk — research, competing-hypothesis debugging, cross-layer refactors where each layer has a different owner — only Claude Code currently lets you set it up, and even there it's still flag-gated.
