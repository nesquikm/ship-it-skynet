# ship-it-skynet

A reference matrix comparing agentic AI coding CLIs. Markdown-first docs project — no application code, no build step, no test runner. The "features" are doc pages. The "bugs" are stale cells and broken links.

## Tech Stack

- **Content:** Markdown, rendered by GitHub
- **Lint:** [markdownlint-cli2](https://github.com/DavidAnson/markdownlint-cli2)
- **Format:** [Prettier](https://prettier.io/) (`proseWrap: preserve`)
- **Link check:** [markdown-link-check](https://github.com/tcort/markdown-link-check)
- **Package manager:** npm
- **Node:** >= 20 (using whatever ships in the current LTS)

No TypeScript, no bundler, no test runner. If a future need justifies adding one, document the reason in `specs/technical-spec.md`.

## Architecture

```text
ship-it-skynet/
├── README.md                    # The master matrix (M1 deliverable)
├── docs/
│   ├── glossary.md              # Vendor-terminology map
│   ├── skills.md                # Deep-dive: reusable procedures
│   ├── hooks.md                 # Deep-dive: lifecycle events
│   └── mcp.md                   # Deep-dive: Model Context Protocol
├── CLAUDE.md                    # This file
├── package.json                 # Scripts + dev dependencies
├── .markdownlint-cli2.jsonc     # Markdown lint config
├── .markdown-link-check.json    # Link checker config
├── .prettierrc                  # Prettier config
├── .prettierignore
├── .gitignore
├── .claude/
│   ├── settings.json            # Tool permissions
│   └── skills/
│       └── refresh/
│           └── SKILL.md         # The /refresh skill (the one thing that rots)
└── specs/                       # Local working notes — gitignored
    ├── requirements.md
    ├── technical-spec.md
    ├── testing-spec.md
    └── plan.md
```

## Key Commands

```bash
# Lint markdown
npm run lint

# Format markdown (writes)
npm run format

# Format check (no writes — what the gate runs)
npm run format:check

# Check links in README.md, CLAUDE.md, docs/*.md
npm run check:links

# Gate — run all three. Must pass before committing.
npm run gate
```

**Gating rule:** Every change must pass before merging:

```bash
npm run gate
```

This expands to `npm run lint && npm run format:check && npm run check:links`. The `/gate-check` skill from dev-process-toolkit runs exactly this.

## Workflows

**Refresh the matrix (the main recurring task):** Invoke the `/refresh` skill. With no arg it re-verifies all tier-1 tools; with an arg like `/refresh gemini-cli` it scopes to one tool. `/refresh` always proposes a diff for human approval — it never edits silently.

**Refresh cadence:** Target a full `/refresh` run every ~3 months. Any row whose "Last verified" date is older than 90 days is stale — that's the signal to re-run. Also refresh sooner when a vendor ships a major release, or when a canonical URL in `.claude/skills/refresh/SKILL.md` starts returning 404s. No automation by design: see `specs/plan.md` M3 for the rationale. The maintainer is expected to read this section when starting a session.

**Add a new row or column:** Edit `README.md` directly. For a new tool column, also add a row to each table in `docs/glossary.md`. For a new feature row, update `docs/{skills,hooks,mcp}.md` if the feature deserves a deep-dive.

**Bugfix (broken link, typo):** Edit → `/gate-check` → commit.

## Key Patterns

- **Primary sources only.** Every non-trivial claim in `README.md` or `docs/*.md` must link to an official doc, changelog, or commit. If the source exists but is behind a login, note that inline.
- **Dated claims.** Every column and every deep-dive page has a `Last verified:` line. Bump it when you re-verify, even if nothing changed.
- **Checkmarks in cells, nuance in prose.** The matrix says ✅ / 🟡 / ❌ / ⏳. Anything requiring explanation goes in `docs/*.md`.
- **Reference-style links, not GitHub footnotes.** Primary-source citations in `README.md`'s matrix use reference-style links (`[✅][cx-plan]` in the cell + `[cx-plan]: url "description"` at the bottom of the file). Do **not** use GitHub footnote syntax (`[^name]`) — VSCode's markdown preview doesn't render footnotes, so the raw marker leaks into the rendered output. The link `title` attribute doubles as a hover tooltip.
- **Tier 1 is privileged.** Claude Code, Codex CLI, Gemini CLI are the three columns in `README.md`. Tier 2 tools only appear in deep-dive prose, never as matrix columns — otherwise the matrix becomes unmaintainable.
- **Vendor PRs are tagged.** PRs from vendors are accepted but labeled `vendor-unverified` until a maintainer re-runs `/refresh` on the affected row.

## Testing Conventions

There are no unit tests — this is a docs project. The gate commands are the test suite:

- **Lint:** `markdownlint-cli2` enforces consistent markdown syntax
- **Format:** `prettier --check` enforces consistent whitespace and table shape
- **Link check:** `markdown-link-check` verifies every link resolves

If you find yourself wanting unit tests, you're probably building tooling that shouldn't live in this repo.

## DO NOT

- **Do not add a "best AI tool" ranking.** The whole project exists to be a reference, not a review. No stars, no verdicts, no "winner" callouts.
- **Do not make claims without a primary source.** No "I heard on Twitter" evidence. If you can't link it, it doesn't go in the matrix.
- **Do not let `/refresh` silently edit files.** It must always produce a diff for human approval — that's its core safety property.
- **Do not commit `specs/`.** It's intentionally gitignored as local working notes. If you need to share spec content, quote it into an issue or PR description instead.
- **Do not let the gate commands take more than ~30 seconds locally.** If link-checking slows down, scope the check to changed files rather than adding retries or skips.
- **Do not add Tier 2 tools as matrix columns.** They belong in prose only. The matrix caps at three columns by design.
- **Do not commit without running `npm run gate`.** This is the only project gate.
