# Changelog

## v1.3.0 — 2026-05-03

### The JSON Layer

- `team-config.json` — single source of truth for team names, models, install type, version, file paths, and session startup protocol. All agents read from it. Replaces scattered prose references and the separate VERSION file.
- `project-standards.json` — user-defined coding standards, linting, test requirements, and forbidden patterns. Bob reads it before every recipe. Richard reviews against it.
- `handoff/build-recipe.json` — Bob generates this before touching any files. Arch approves it. Richard uses it as source of truth during review.
- `handoff/sprint-plan.json` — Arch maps the full sprint upfront. All steps, status, current step. Bob and Richard see where each step fits.
- `handoff/known-gaps.json` — structured gap tracking with lifecycle (open/in-progress/addressed). Replaces KG-N prose in BUILD-LOG. Arch reviews at every session start.
- `handoff-schema.json` — authoritative schema for every handoff file. Single source of truth for what each file requires, who owns it, who reads it.

### Version check now JSON-to-JSON
Arch reads `version` from `team-config.json`, fetches GitHub releases API (returns JSON), compares `tag_name`. No separate VERSION file — removed.

### Review status upgraded
`Must Fix / Should Fix / Ready for Builder` replaced with `APPROVED / APPROVED WITH CONDITIONS / REJECTED`. There is no "Should Fix." If it needs fixing it is a Condition. Updated in both templates and all handoff files.

### Bob's workflow: recipe before code
Bob generates `handoff/build-recipe.json` before writing any file. Arch approves the recipe. Bob adds linting gate and self-review before handing to Richard.

### Generic template brought to parity with project-folder
- Version check added to `templates/generic/ARCHITECT.md`
- Model selection added as question 4 in `templates/generic/new-setup.md`
- `@.claude/skills/token-optimization.md` import added to CLAUDE.md creation
- `.claude/skills/token-optimization.md` now ships with generic template

### Project standards as onboarding question
Question 5 added to both `new-setup.md` templates. Arch writes `project-standards.json` from the answer during first session.

### Structured JSON blocks in handoff files
All handoff `.md` files now include a machine-readable JSON block at the bottom. Human prose stays. Agents can parse status, step number, and file lists without scanning prose.

### JSON config files ship with both templates
`team-config.json`, `project-standards.json`, `handoff-schema.json`, and the three handoff JSON files now live in both `templates/project-folder/` and `templates/generic/`. The existing `cp -r templates/project-folder/. /project/` install command copies them automatically — no script change required. Setup script echo updated to list what is installed.

## v1.2.3 — 2026-05-03

- Auto-update check: Arch now checks the GitHub releases API at session start and notifies the Project Owner if a newer version is available
- Added `VERSION` file to `templates/project-folder/` — tracks installed version for comparison

## v1.2.2 — 2026-05-03

- Fix: token-optimization.md now ships with every install — added to `templates/project-folder/.claude/skills/` so the `@` auto-load reference works out of the box
- Fix: CLAUDE.md creation instructions in `new-setup.md` now include `@.claude/skills/token-optimization.md` for both new and existing project context files
- Fix: install `cp` command changed from `*` to `.` in setup script, README, and INSTALL.md — hidden directories (`.claude/`) are now copied correctly
- Docs: added "one session, three roles" callout to README Quick Start and INSTALL.md — clarifies that Bob and Richard are subagents within a single Claude Code session, not separate windows
- Docs: `new-setup.md` now shows the "one session" model explanation to the Project Owner during first-time setup

## v1.2.0 — 2026-04-20

- RTK install block: dropped curl command, now links to github.com/rtk-ai/rtk README (fixes private fork URL)
- Windows support: added Windows section to INSTALL.md — Git Bash/WSL for setup script, manual copy fallback, RTK not supported on Windows
- RTK install note in new-setup.md clarifies macOS/Linux only; Windows users can skip
- handoff/ paths: all agent files now reference handoff/ARCHITECT-BRIEF.md, handoff/REVIEW-REQUEST.md, etc. — prevents files landing in project root
- BUILD-LOG discipline: added Anti-Drift rule requiring BUILD-LOG updates immediately on any decision, not only at deploy
- Model assignment: new-setup.md adds 4th setup question for per-agent model selection; ARCHITECT.md briefing sections now document the `model` parameter with available Claude model IDs

## v1.1.0 — 2026-04-03

- Added `new-setup.md` — guided first-session onboarding handled by Arch: team naming, project context file, RTK install
- Architect now spins up Builder and Reviewer via Agent tool (foreground only) — documented in ARCHITECT.md and INSTALL.md
- Foreground-only requirement documented — background agents stall on Edit approval
- Reviewer protocol upgraded: APPROVED / APPROVED WITH CONDITIONS / REJECTED replaces Must Fix / Should Fix
- Reviewer now runs `git diff` first — diff is primary source of truth, not REVIEW-REQUEST
- Builder self-review and linting gate added before handing off to Reviewer
- Setup script rewritten as guided walkthrough — detects global vs per-project, prints tailored instructions
- `handoff/` folder added to both templates so `cp` command includes it
- `CLAUDE.md` removed from templates — setup script is the only source of truth for that step
- `team.yml.example` removed — renaming handled by Arch during setup
- Stale docs removed: `customizing-your-team.md`, `project-setup.md`
- README Quick Start restructured — two clearly labeled install paths
- Path mismatch in `CLAUDE.md` session router resolved

## v1.0.0 — 2026-03-31

Initial public release.

- Three-agent team: Architect, Builder, Reviewer
- Generic agents in `agents/` with customizable personas and [CUSTOMIZE] placeholders
- Named persona template in `templates/project-folder/` (Arch, Bob, Richard)
- Generic clean-slate template in `templates/generic/`
- Structured handoff files: ARCHITECT-BRIEF, REVIEW-REQUEST, REVIEW-FEEDBACK, BUILD-LOG, SESSION-CHECKPOINT
- Token optimization rules baked into every session router
- RTK integration guidance in docs/token-optimization.md
- Setup script with CLAUDE.md instructions printed on install
- Full documentation suite
