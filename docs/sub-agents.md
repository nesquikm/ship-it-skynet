# Sub-agents & agent teams — deep dive

> Three rows in the matrix, one spectrum: delegated helpers at one end, coordinating peers in the middle, independently started sessions at the other.

**Last verified:** 2026-08-10

A "sub-agent" in the neutral sense is a helper agent the parent can spawn to do focused work in its own context window, then fold the result back into the main conversation. All three tier-1 CLIs implement some version of this. The deeper question — and the reason this deep-dive exists — is whether the helpers can _talk to each other_, claim tasks from a shared queue, and be driven individually by the user. That's where the three tools diverge, and it's why the matrix carries three separate rows (Sub-agents, Agent teams, and Cross-session messaging) instead of one.

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

Nested sub-agents changed direction in v2.1.217: by default a subagent can no longer spawn subagents of its own — the harness withholds the `Agent` tool from every subagent except a fork (where the tool stays listed but errors). To allow nesting, set the `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` env var to the number of layers you want below the main conversation; a subagent at the configured depth can't delegate further. To keep a specific subagent from spawning while nesting is on, omit `Agent` from its `tools` allowlist or add it to `disallowedTools`. (From v2.1.172 through v2.1.216 nesting was on by default, capped at a fixed five layers — the claim this paragraph made until the 2026-07-25 refresh.) Two sibling limits are documented alongside depth: at most 200 subagents per session by default (`CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION`, v2.1.212+) and a concurrent-subagent cap.

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

The docs describe agent-teams behavior as of v2.1.178: with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` set, spawning a teammate no longer needs a setup step and cleanup happens automatically when the session exits (the earlier `TeamCreate` / `TeamDelete` tools no longer exist). Key differences from plain sub-agents:

- **Peer messaging.** Any teammate can message any other by name over `SendMessage`; delivery is automatic and the lead isn't a relay. There is no broadcast primitive — to reach everyone, send one message per recipient.
- **Shared task list.** Tasks have `pending` / `in progress` / `completed` states plus dependency tracking. Teammates self-claim with file locking to prevent races.
- **Direct user interaction.** In in-process mode, use the up/down arrow keys in the agent panel to select a teammate, Enter to view and message it, Escape to interrupt it, `x` to stop it, and Ctrl+T to toggle the task list; or interact via split panes (tmux / iTerm2 `it2` CLI). You can message or interrupt any teammate individually; plan approval requests go to the lead, which approves or rejects autonomously based on criteria you give it.
- **Plan approval.** Teammates can be required to plan before implementing; the lead approves or rejects with feedback before the teammate exits plan mode.
- **Team-specific hooks.** `TeammateIdle` (exit 2 to keep the teammate working), `TaskCreated` (exit 2 to block creation), `TaskCompleted` (exit 2 to block completion) — all can return feedback strings.
- **Sub-agent definitions as teammates.** Spawn by referencing a sub-agent type by name; the definition's `tools` allowlist and `model` are honored, and the body is _appended_ to the teammate's system prompt rather than replacing it. Note: `skills` and `mcpServers` from the definition are **not** applied when run as a teammate — teammates load those from project/user settings.

Current limitations worth knowing before you bet a workflow on this:

- `/resume` and `/rewind` do not restore in-process teammates; the lead may attempt to message ghosts after resume.
- Task status can lag; a stuck task may need a manual nudge.
- One team per session. Nested teams are not allowed. Leadership is fixed at spawn — you can't promote a teammate.
- Split-pane mode requires tmux or iTerm2; VS Code's integrated terminal, Windows Terminal, and Ghostty are in-process only.

## Codex CLI

Docs: <https://developers.openai.com/codex/agent-configuration/subagents> (back on developers.openai.com after a mid-2026 detour through learn.chatgpt.com; both hosts mirror the same content)

Codex CLI has sub-agents but no agent-teams equivalent. Sub-agent workflows are enabled by default in current releases (`agents.enabled` defaults to `true`). Sub-agents are defined as standalone TOML files under `~/.codex/agents/` (user) or `.codex/agents/` (project); each file must define `name`, `description`, and `developer_instructions`, while optional fields (`model`, `model_reasoning_effort`, `sandbox_mode`, `mcp_servers`, `skills.config`) inherit from the parent session when omitted — note `mcp_servers`, which scopes MCP servers to a single agent. Global settings live in `config.toml` under `[agents]`: `agents.max_concurrent_threads_per_session` caps concurrently open agent threads (`max_threads` survives as a legacy alias), and `agents.default_subagent_model` / `agents.default_subagent_reasoning_effort` set spawn defaults. The `max_depth` recursion setting documented through 2026-07-11 no longer appears in the reorganized docs.

The model is strictly hierarchical. Quoting the Codex docs: _"ChatGPT or Codex handles orchestration across agents, including spawning new subagents, routing follow-up instructions, waiting for results, and closing agent threads."_ The parent spawns, routes, waits, and closes — there is no peer-to-peer messaging and no shared task list. The user does get direct visibility via the `/agent` slash command (alias `/subagents` — switch between active agent threads, inspect the ongoing thread), which is more than Claude Code's sub-agent UX offers, but the sub-agent still reports _only_ to its parent.

One feature narrows the gap slightly but doesn't cross it: **consolidated returns** — when many agents run, Codex waits until all requested results are available and returns a consolidated response. That's a coordination convenience, not peer coordination. (The experimental `spawn_agents_on_csv` batch mode this page previously listed dropped out of the docs in the 2026-07 reorg.)

No `TeammateIdle` equivalent, no shared mailbox, no "Agent Teams" page in the docs. In the matrix this is a clean `❌` on the Agent teams row despite a clean `✅` on the Sub-agents row.

## Antigravity CLI (successor to Gemini CLI)

Docs: <https://antigravity.google/docs/subagents> (the platform subagents doc, which covers the CLI), <https://antigravity.google/docs/cli/subagents> (the CLI's "Background Tasks & Subagents" page, new since 2026-07-11), <https://antigravity.google/docs/cli/commands/agents> (the `/agents` panel), and <https://antigravity.google/docs/cli/features> — all server-rendered HTML as of 2026-07-25 (the raw-markdown asset path was retired), plus the [repo CHANGELOG](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md).

Subagents ship and got a substantially richer doc between 2026-07-11 and 2026-07-25. The parent spawns concurrent subagents via the `invoke_subagent` tool, with three workspace options: `inherit` (parent's workspace), `branch` (**an isolated Git worktree** — parent agents retain full access to subagent worktrees), or `share`. Built-in subagents: `research` (codebase exploration), `browser` (sandboxed web browsers, via `/browser`), and `self` (a clone of the calling agent). Custom subagents are markdown files with YAML frontmatter — `name`, `description`, `tools`, `mainAgent`, `subagent`, `model` (`inherit` / `flash` / `pro`, 1.1.5), `commandExecutionPolicy`, `skills` — at `.agents/agents/<name>.md` or `<name>/agent.md` (workspace), `~/.gemini/config/agents/` (global), or plugin-bundled; the format landed in CHANGELOG 1.1.6, and transient subagents can be defined mid-session with `define_subagent`. Recursion is capped at **10 nesting levels** below the primary agent (contrast Claude Code's configurable opt-in depth). The `/agents` panel switches custom agents and monitors background subagents; `/tasks` tracks background processes, with `Alt+J` "teleport" and `Ctrl+K` fast-approve for pending subagent confirmations. Two 1.1.10 fixes matter if you actually run deep trees: stopping a subagent tree used to stop only the conversation it was invoked from while every descendant and its background tasks kept running (with the CLI reporting them killed), and a coordinator waiting on active subagents could deadlock, looping empty continue steps until it hit the invocation limit.

The Agent-teams question got a partial answer on 2026-07-11 and a slightly stronger one on 2026-07-25: the subagents doc documents inter-agent messaging by unique conversation ID — agents "can communicate with parent agents, subagents, or peer agents whose ID is known," messaging an idle agent re-awakens it, and agents can read each other's transcripts — plus a **Multi-Agent Teamwork** framework behind the `/teamwork-preview` slash command (still preview, still Ultra-plan-only). No shared task list is documented, and the CLI's own subagents page still describes only parent-delegated background workers, so Agent teams stays 🟡 rather than ✅.

### Predecessor: Gemini CLI (consumer sunset 2026-06-18; enterprise still served)

Docs: <https://geminicli.com/docs/core/subagents/> and <https://geminicli.com/docs/core/remote-agents/>

User-definable sub-agents live in `.gemini/agents/*.md` (project) or `~/.gemini/agents/*.md` (user). Each agent is a markdown file with YAML frontmatter plus a body that becomes the sub-agent's system prompt. Supported frontmatter: `name` and `description` (required), `kind` (defaults to `local`), `tools` (explicit list or wildcards like `*`, `mcp_*`, `mcp_server_*`; omit to inherit the parent's toolset), `mcpServers` (inline MCP config), `model` (defaults to `inherit`), `temperature` (default `1`), `max_turns` (default `30`), `timeout_mins` (default `10`). Invocation happens three ways: automatic selection by the main agent, explicit force-routing via `@subagent_name`, or calling the sub-agent as a same-named tool.

Built-in agents: `codebase_investigator` (codebase analysis), `cli_help` (CLI documentation lookup), `generalist` (all-tools helper for complex multi-step tasks), and `browser_agent` (web automation, experimental and disabled by default). Discovery tiers are workspace > user > extension.

The Gemini sub-agent model is deliberately non-recursive: _"subagents cannot call other subagents. If a subagent is granted the `*` tool wildcard, it will still be unable to see or invoke other agents."_ There is no shared task list, no inter-agent messaging, and no `SubagentStart` / `SubagentStop` hook in the 11-event hook catalog. That landed Gemini at `✅` on the Sub-agents row and a clean `❌` on the Agent teams row while it held the matrix column (through 2026-06-10).

**Remote agents** (`geminicli.com/docs/core/remote-agents/`) are a separate feature and don't change the Agent teams row. They use the same `.gemini/agents/*.md` location but specify an `agent_card_url` (or inline `agent_card_json`) pointing at an external service that speaks the Agent-to-Agent (A2A) protocol. Gemini CLI acts as a client delegating tasks unidirectionally; there is no peer-to-peer messaging between remote agents, no shared task list, and no lead/teammate relationship. Supported auth schemes include API keys, OAuth 2.0, Google credentials, and HTTP Basic.

## Cross-session messaging

The matrix gained a **Cross-session messaging** row on 2026-08-10. It is a third point on the same spectrum, and it is genuinely distinct from the two rows above: sub-agents and teammates are agents that something _spawned_, whereas cross-session peers are sessions **you** started, in separate terminals, that later discover each other.

**Claude Code** (<https://code.claude.com/docs/en/cross-session-messaging>) — ✅. Two tools do the work: `ListAgents` discovers reachable sessions, `SendMessage` delivers to one by name. Users see the same listing via `/list-agents` (alias `/peers`), and `/status` shows the session's own `Peer address`. A session answers to the name set with `--name` or `/rename`, otherwise one derived from the working directory's folder name (`myapp-3f`); when names collide, the address takes a short identifier. Same-machine delivery rides a per-session Unix inbox socket that Claude Code restricts to your OS user and exports to hooks and Bash as `CLAUDE_CODE_MESSAGING_SOCKET` — it never touches Anthropic servers. Cross-machine messaging goes through Anthropic servers over the target's Remote Control connection and is **reply-only**: a session elsewhere can be answered, never cold-called. Inbound is governed by `crossSessionInbound` (`accept` / `hold` / `refuse`); with nothing set, Claude Code sorts sessions into a bypass-permissions class and a prompting class and holds anything crossing the line for approval, with `isolatePeerMachines` forcing approval on cross-machine sends and `dialogExpiry` (default five minutes) dropping unanswered dialogs. Messages are plain text — a `/compact` in the body arrives as text and is never executed — and the held-message buffer caps at 100. Requires v2.1.224+, macOS or Linux including WSL 2 (not native Windows), and is unavailable on Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform, and Microsoft Foundry. Where those hold, it is on with nothing to enable — GA, not a flag-gated preview, which is what separates this row from the flag-gated Agent teams row.

**Antigravity CLI** (<https://antigravity.google/docs/subagents>) — 🟡. The routing primitive exists and is named: `send_message` (arguments `Recipient`, `Message`) sits in the hooks tool catalog under "Agent Collaboration," and the subagents doc says agents "can communicate with parent agents, subagents, or peer agents whose ID is known," re-awakening an idle recipient and exposing mutually readable transcripts. What is missing is the _session_ half. Addressing is by agent conversation ID within an agent tree; nothing documents discovering or naming independent `agy` sessions the way `/list-agents` does, and there is no inbox-socket or inbound-policy surface. The closest the CLI gets to noticing a sibling session is a 1.1.10 advisory banner when the same conversation is already open in another CLI instance on the machine — and its advice is to `/fork`, not to message. Partial capability, sourced: 🟡.

**Codex CLI** (<https://developers.openai.com/codex/agent-configuration/subagents>) — ❌. Nothing in the Codex docs tree describes a session-to-session or peer channel. The subagents doc affirmatively scopes agent communication to parent-driven orchestration — "spawning new subagents, routing follow-up instructions, waiting for results, and closing agent threads" — and the CLI reference's slash-command table has no peer-listing or messaging entry; `/agent` (alias `/subagents`) only switches between the current session's own agent threads. Same sourcing standard as the Agent teams ❌ in the row above.

The ordering across the three rows is worth noticing, because it is not the same ordering each time. On Agent teams, Claude Code leads and Antigravity is the closest challenger. On cross-session messaging, Claude Code leads again — but Antigravity's 🟡 comes from a _different_ strength: it has the messaging primitive and lacks the session addressing, which is close to the mirror image of a tool that could enumerate peers but not talk to them.

## What to check when refreshing

- Has Claude Code's `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` flag been retired? If Agent Teams goes GA, the matrix cell moves from `🟡 experimental` to `✅` and this deep-dive should drop the flag requirement.
- Has Codex added anything resembling a shared task list, peer messaging, or a dedicated teams page? The `/codex/subagents` page is where it would land first.
- Antigravity's Agent-teams cell moved ⏳ → 🟡 on 2026-07-11 (documented peer messaging + Ultra-gated `/teamwork-preview`; by 2026-07-25 the doc described messaging by conversation ID and shared transcripts). The named `send_message` tool is back in the docs as of 2026-08-10 — the hooks tool catalog lists it under "Agent Collaboration" with `Recipient` and `Message` arguments — so the 2026-07-25 note that the docs no longer named it no longer holds. The next trigger: a shared-task-list or teammate-session doc, or `/teamwork-preview` leaving preview — either could justify ✅.
- Do any new events join Claude Code's hook catalog under the Team\* namespace? The existing set is `TeammateIdle`, `TaskCreated`, `TaskCompleted`; additions belong in the Claude Code section above.
- Does Antigravity document addressing an independent CLI session — a peer listing, a session name flag, an inbox surface — rather than only agent conversation IDs inside one tree? That is the single change that would move Cross-session messaging from 🟡 to ✅.
- Has Codex added any session-to-session channel? The subagents page and the CLI reference's slash-command table are where it would surface first; both were clean on 2026-08-10.
- Does Claude Code's cross-session messaging reach native Windows, or any of the four excluded providers? Either would change the caveats, though not the ✅.
- Are the command `/resume` + in-process-teammates limitations still open? Each limitation in the list above is worth re-checking on every refresh, since the feature is experimental by Anthropic's own label.
- Antigravity published its hook catalog on 2026-07-11 (`PreToolUse`, `PostToolUse`, `PreInvocation`, `PostInvocation`, `Stop` — see the [hooks deep-dive](hooks.md)); it contains no subagent lifecycle events, so watch for a `SubagentStart`/`SubagentStop` equivalent. For the enterprise-only Gemini CLI predecessor, `geminicli.com/docs/hooks/` remains the source of truth (no `SubagentStart`/`SubagentStop` as of 2026-06-10).

## Why this deserved its own page

The matrix can say `✅` or `🟡` or `❌`, but it can't capture the _architectural_ difference between "the parent spawns a helper and waits for a string back" and "five independent sessions claim tasks from a shared queue and debate each other by name." That's what this deep-dive exists for. If you're evaluating the three CLIs for work that genuinely benefits from parallel exploration with cross-talk — research, competing-hypothesis debugging, cross-layer refactors where each layer has a different owner — Claude Code is the only one with the full setup (shared task list + peer messaging + user-addressable teammates), and even there it's still flag-gated; Antigravity's documented peer messaging and Ultra-gated `/teamwork-preview` are the closest challenger.
