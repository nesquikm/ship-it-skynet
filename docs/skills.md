# Skills — deep dive

> How each CLI lets you package reusable procedures.

**Last verified:** 2026-06-10

A "skill" in the neutral sense is a reusable, named procedure the agent can invoke on demand — distinct from a slash command in that the model can decide to trigger it based on a description, not only the user.

**Convergence note:** Claude Code and Codex CLI implement the same [Agent Skills](https://agentskills.io) open standard — a `SKILL.md` file with YAML frontmatter (`name`, `description`) plus optional supporting files is portable between them with only minor directory-layout differences. Gemini CLI implemented it too; its successor Antigravity CLI is announced as preserving Agent Skills, but hasn't yet published whether the `SKILL.md` layout carries over unchanged. The ergonomics still diverge — read on.

## Claude Code

Docs: <https://code.claude.com/docs/en/skills>

Skills live at `.claude/skills/<name>/SKILL.md` (project), `~/.claude/skills/<name>/SKILL.md` (user), plugin-bundled, or managed (enterprise). The description is how the model decides when to invoke the skill, so it matters — short, specific, trigger-oriented. Custom commands under `.claude/commands/` have been merged into skills; both still work and produce `/name` slash commands.

Unique surface area (some of the richest frontmatter in the tier-1 set):

- **`disable-model-invocation: true`** — make a skill user-triggered only (for commit, deploy, etc.).
- **`context: fork` + `agent:`** — run the skill in a forked sub-agent context (see [sub-agents](https://code.claude.com/docs/en/sub-agents)).
- **`allowed-tools`** — pre-approve a tool allowlist while the skill is active.
- **`` !`command` `` injection** — shell output is interpolated into the prompt before the model sees it.
- **`hooks:` frontmatter** — scope lifecycle hooks to a skill's lifetime.
- **Additional fields** surfaced in the 2026-04-11 skills frontmatter reference: `user-invocable`, `argument-hint`, `paths`, `model`, `effort`, `shell`. See the [skills docs](https://code.claude.com/docs/en/skills) for exact semantics — they govern slash-command surfacing, path scoping, model/effort overrides, and shell selection for `!` injection.

## Codex CLI

Docs: <https://developers.openai.com/codex/skills>

Skills live in `.agents/skills/<name>/SKILL.md` at the repo root (or any parent), `$HOME/.agents/skills/` for user-level, `/etc/codex/skills/` for admin, plus OpenAI-bundled skills. Same frontmatter (`name`, `description`) plus an optional `agents/openai.yaml` for UI configuration.

Activation:

- **Explicit:** `/skills` command, or `$skill-name` mention in the composer.
- **Implicit:** Codex loads the full `SKILL.md` when your task matches the description. Only the metadata (name, description, file path) is loaded at session start — the body loads on activation. This is the **progressive disclosure** pattern.

## Antigravity CLI (successor to Gemini CLI)

Sources: [repo CHANGELOG](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md) and the [transition blog](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/) — `antigravity.google/docs` is a JS-rendered SPA we can't cite yet.

What's verifiable as of `agy` v1.0.7: custom and fallback skills load even in Standalone mode (1.0.2); skills bundled in installed plugins are auto-discovered and made executable (1.0.1); skills surface as **skill-derived slash commands** with autocomplete in the composer (1.0.4). The transition blog names Agent Skills as one of the four Gemini CLI features Antigravity preserves. Not yet documented: whether the `activate_skill` explicit-tool-call model, the `SKILL.md` open-standard layout, or a `gemini skills`-style package-manager surface carries over.

### Predecessor: Gemini CLI (consumer sunset 2026-06-18; enterprise still served)

Docs: <https://geminicli.com/docs/cli/skills/>

Discovery tiers (highest to lowest precedence): **Workspace > User > Extension**. Workspace skills live in `.gemini/skills/` or `.agents/skills/` (the latter alias is Gemini's nod to portability across agent tools). User skills in `~/.gemini/skills/`.

Key differences from Claude Code / Codex:

- **`activate_skill` tool.** Skill activation is an explicit tool call the model makes, not ambient frontmatter magic. You also see a confirmation prompt in the UI before a skill's directory is added to the agent's allowed file paths.
- **`gemini skills` CLI.** `gemini skills list`, `install`, `uninstall`, `link`, `enable`, `disable` — a first-class package-manager surface for skill distribution, including installing from a Git URL (`gemini skills install https://github.com/user/repo.git`) or a `.skill` zip file.
- **Interactive management via `/skills list|enable|disable|reload|link`.**

---

## Why this isn't a matrix cell

The existence of skills is binary (✅ / ❌), but the _ergonomics_ aren't. Whether skills are triggered by description-matching vs. explicit user invocation, whether they run in main context or isolation, whether they compose with sub-agents — none of that fits a checkmark. Claude Code has the richest frontmatter surface; Codex has the cleanest progressive-disclosure semantics; Gemini CLI had the best distribution story (`gemini skills install` from Git URLs or `.skill` zips), and whether Antigravity CLI inherits it is one of the open questions for the next refresh.
