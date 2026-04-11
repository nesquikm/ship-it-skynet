---
name: refresh
description: Re-verify the ship-it-skynet coding-agent comparison matrix against primary sources. Fetches official docs for tier-1 CLIs (Claude Code, Codex CLI, Gemini CLI), diffs claims in README.md and docs/*.md against current reality, proposes updates for human approval, bumps "Last verified" dates, and runs the link-check gate. Optional arg scopes to one tool (e.g., "/refresh gemini-cli"). Without arg, refreshes all three tier-1 tools. Use monthly, or whenever you notice a vendor release.
---

# /refresh — re-verify the matrix

This skill keeps ship-it-skynet's comparison matrix honest. It fetches current official docs, diffs them against the claims in this repo, and proposes updates for human approval.

## Scope

If the user supplied an argument (e.g., `claude-code`, `codex-cli`, `gemini-cli`), scope the refresh to that single tool. Otherwise refresh all three tier-1 tools.

Tier 2 tools are **never** refreshed into the matrix — they belong in prose only. If the user tries to add one as a matrix column, push back and point them at `docs/*.md`.

## Canonical sources

These URLs are the single source of truth for "what counts as primary" in this project. **If a URL is wrong or outdated, update it in this file** — then commit the correction as part of the refresh run.

### Claude Code

Confirmed (explicitly linked in the project):

- Skills: <https://code.claude.com/docs/en/skills>
- Sub-agents: <https://code.claude.com/docs/en/sub-agents>

Probable (pattern extension — verify by fetching before trusting):

- Overview: `https://code.claude.com/docs/en/`
- Hooks: `https://code.claude.com/docs/en/hooks`
- MCP: `https://code.claude.com/docs/en/mcp`
- Settings / permissions: `https://code.claude.com/docs/en/settings`

> **Note:** Custom slash commands are now part of skills. Fetch the skills page as authoritative for both — there is no separate slash-commands page. Confirmed 2026-04-11: `code.claude.com/docs/en/slash-commands` now returns the skills page content, so fetching both pages is duplicate work.

Authoritative secondary:

- Changelog: `https://github.com/anthropics/claude-code` (look for CHANGELOG.md or releases)
- Latest release: `gh api repos/anthropics/claude-code/releases/latest` — `tag_name` is the authoritative version string, `published_at` is the authoritative publish date (same pattern as Codex and Gemini).

### Codex CLI (`@openai/codex`)

Primary source — developers site (preferred; check this first):

- Features: `https://developers.openai.com/codex/cli/features`
- Skills: `https://developers.openai.com/codex/skills`
- Slash commands: `https://developers.openai.com/codex/cli/slash-commands`
- Agent approvals & security: `https://developers.openai.com/codex/agent-approvals-security`
- AGENTS.md guide: `https://developers.openai.com/codex/guides/agents-md`
- Hooks: `https://developers.openai.com/codex/hooks` _(bare path — not under `/cli/`; `codex/cli/hooks` 404s)_

> **Note:** `github.com/openai/codex/docs/*.md` files are mostly one-line redirect stubs pointing at the developers site. Check the developers site first; use the GitHub repo only for README, CHANGELOG, and version signals.

GitHub repo (README, changelog, version):

- Repo: `https://github.com/openai/codex` — README and `/CHANGELOG.md`
- Latest release: `gh api repos/openai/codex/releases/latest` — `tag_name` is the authoritative version string, `published_at` is the authoritative publish date. The npm package page is intentionally not listed here: it returns 403 to `WebFetch`, so the GitHub Release API is the reliable source for version signals.

### Gemini CLI (`@google/gemini-cli`)

Canonical doc paths (fetch these directly — don't discover them by walking the repo tree at runtime):

- `docs/hooks/reference.md`
- `docs/hooks/index.md`
- `docs/cli/plan-mode.md`
- `docs/cli/skills.md`
- `docs/cli/custom-commands.md`
- `docs/cli/gemini-md.md`
- `docs/cli/sandbox.md`
- `docs/cli/git-worktrees.md`
- `docs/cli/settings.md`

GitHub repo (README, changelog, version):

- Repo: `https://github.com/google-gemini/gemini-cli`
- Latest release: `gh api repos/google-gemini/gemini-cli/releases/latest` — `tag_name` is the authoritative version string, `published_at` is the authoritative publish date. The npm package page is intentionally not listed here: it returns 403 to `WebFetch`, so the GitHub Release API is the reliable source for version signals.

> **Maintenance:** The first time this skill runs for a given tool, the agent should verify each URL resolves. Broken URLs → update this file, then continue. This list is expected to drift; that's why it's in the skill, not in the matrix.

## Fetching tips

These rules make the fetch phase reliable. They were surfaced on the 2026-04-11 refresh run and apply to every future run.

### Prefer `gh api` raw-content over `WebFetch` for GitHub files

For anything under `github.com/<owner>/<repo>/blob/...`, use:

```bash
gh api repos/{owner}/{repo}/contents/{path} -H "Accept: application/vnd.github.raw"
```

`WebFetch` against blob URLs redirects to rendered HTML, and the extraction model summarizes the page instead of returning raw markdown. The `gh api` raw-content header returns clean markdown every time. Use this for every doc file, README, and CHANGELOG hosted on GitHub.

### Expect long Claude Code docs pages to persist — grep the temp file

Three Claude Code docs pages (`sub-agents` ~51 KB, `mcp` ~54 KB, `settings` ~94 KB) reliably trip the `WebFetch` output budget and are auto-persisted to a temp file. **Narrow extraction prompts don't help** — confirmed empirically on 2026-04-11 — because `WebFetch` measures the upstream document's wire size _before_ the summarization model sees the prompt. Prompt verbosity cannot reduce upstream bytes. This was a false lead; treat persistence as the expected path for these pages.

Procedure:

1. Fetch each page with whatever prompt is natural for what you're looking for.
2. When `WebFetch` reports "Output too large (NNN KB)" and writes to a temp file, capture the temp file path from the message.
3. `grep` the temp file for the specific heading, field, table row, or keyword you need — e.g. `grep -A 5 "defaultMode" /tmp/webfetch-XXXX.md`. This is still fast: no extra network round-trip.
4. The `skills` and `hooks` pages are smaller and usually stay inline; don't over-engineer fetches for them.

The NFR-1 30-minute refresh budget is protected because fetch + grep is still fast in practice (full 3-tool dry run completed in 216 s on 2026-04-11, including all three persisted pages). The friction the old advice tried to eliminate — "pages got persisted to temp files" — turns out to be trivial once the procedure plans for it.

## Process

Run these steps for each target tool. Never batch writes — one proposed edit at a time, each approved before the next.

### 1. Fetch primary sources

Fetch the URLs from the canonical-sources section above, following the [Fetching tips](#fetching-tips):

- For GitHub-hosted files, use `gh api repos/{owner}/{repo}/contents/{path} -H "Accept: application/vnd.github.raw"`.
- For long Claude Code docs pages, pass a narrow extraction prompt to `WebFetch` to stay under the output budget.
- For everything else (e.g., `developers.openai.com/codex/*`), use `WebFetch` (or the rubber-duck MCP if it's configured).
- For version signals on Codex and Gemini CLI, call `gh api repos/{owner}/{repo}/releases/latest` and read `tag_name` / `published_at`.

If a fetch fails:

- Note the failure in the refresh report
- Try the next URL in the list — don't silently skip
- Never substitute a vibes-based claim for a failed fetch

### 2. Diff against current claims

For each row in `README.md`'s matrix that references this tool, verify the cell against the fetched docs. For each section in `docs/skills.md`, `docs/hooks.md`, `docs/mcp.md`, and `docs/glossary.md` that references this tool, verify the prose.

### 3. Categorize each change

| Change                           | Severity  | How to handle                                                    |
| -------------------------------- | --------- | ---------------------------------------------------------------- |
| ⏳ → ✅ / 🟡 / ❌ (first verify) | normal    | Expected on first run — fill the cell, add source link           |
| ✅ → ✅ with new URL             | minor     | URL rot — update the link, bump the date                         |
| ✅ → 🟡 (feature degraded)       | **major** | Flag as regression — document what moved and why in the report   |
| ✅ → ❌ (feature removed)        | **major** | Flag prominently — this is the kind of thing people need to know |
| new feature exists               | normal    | Propose a new matrix row — but ask before adding it              |
| new tool version / rename        | major     | Call out in the report header; may require updating other cells  |

### 4. Write a report

Produce a markdown summary before touching any files:

```text
## Refresh report — <tool> — <YYYY-MM-DD>

Sources fetched:
- <url 1> ✓
- <url 2> ✓
- <url 3> ✗ (reason)

Proposed changes:

| Row / Section | Old | New | Source |
| ------------- | --- | --- | ------ |
| ...           | ... | ... | ...    |

Regressions flagged: <count, or "none">
```

### 5. Propose edits one at a time

Use the `Edit` tool. For each row:

1. Show the proposed change (old_string → new_string)
2. Explain which source backs it
3. Wait for approval
4. Move to the next row

**Do not stage multiple edits in one batch** — the user's trust in this matrix comes from being able to approve or reject each cell independently.

### 6. Bump "Last verified" dates

On every row / page you touched, even if the cell didn't change. "Verified and still correct as of `<date>`" is useful signal. Use today's date in YYYY-MM-DD format.

### 7. Run the gate

Run `npm run gate`. If `check:links` catches broken links anywhere, fix them before finishing. Do not hand back a refresh run that leaves the gate red.

### 8. Post-run self-check

After the gate passes, ask the user to flag any new friction discovered during the run:

- Were any canonical URLs wrong, 404-ing, or returning unexpected content?
- Did any `WebFetch` response exceed the output budget and require temp-file persistence?
- Were any important doc paths missing from the canonical-sources list?
- Did any step in the process (fetch, diff, approval flow, report format) feel painful?

If the user flags anything, offer to open a follow-up task — either as a new milestone in `specs/plan.md` or as a TODO at the top of this skill file. This closes the feedback loop so future improvements happen systematically, not reactively. Rationale: the 2026-04-11 refresh run surfaced six friction points that sat undocumented until a retrospective caught them. A self-check at the end of each run prevents that pattern from recurring.

## Rules

- **NEVER edit silently.** Every proposed change goes through human approval.
- **NEVER invent sources.** If a claim can't be verified from the canonical URLs, leave the cell as-is and note it in the report.
- **NEVER delete historical claims without explanation.** A feature disappearing is a story, not a silent deletion.
- **NEVER mark unverified cells as ✅.** Use ⏳ until you have a primary-source link.
- **NEVER add Tier 2 tools as matrix columns.** They belong in prose only.
- **NEVER use GitHub footnote syntax (`[^name]`) in matrix cells.** VSCode's default markdown preview doesn't render footnotes, so the raw `[^name]` marker leaks into the rendered output. Always use reference-style links: `[✅][cx-plan]` in the cell, with `[cx-plan]: url "short description"` at the bottom of the file. Reference-style links render correctly in both GitHub and VSCode, and the link `title` attribute doubles as a hover tooltip for the source description.
- **ALWAYS bump the date** on every row you verified, changed or not.
- **ALWAYS run `npm run gate`** at the end. A green gate is part of the deliverable.

## Final output

When done, produce a single summary suitable for pasting into a commit message:

```text
refresh: <tools> — <YYYY-MM-DD>

Sources fetched: <n>
Changes proposed: <n>
Changes approved: <m>
Regressions: <count, or "none">
Link check: <pass/fail>

<one-paragraph changelog of what actually moved>
```

## When to run

- **Monthly** — tied to a `/schedule` cron if one is configured. See `specs/plan.md` for the cadence decision.
- **After a vendor release** — new major version, blog post, changelog entry.
- **Before publishing externally** — don't ship a stale matrix.
- **When opening a vendor PR** that needs maintainer verification of a `vendor-unverified` claim.
