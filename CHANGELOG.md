# Changelog

## v2.0.0 — 2026-05-04 (codex-review-gates fork)

**Breaking change**: Reviewer role replaced with Codex review gates. Personal fork; not
upstream Russell.

- The Claude `Reviewer` (Richard) persona is removed. `templates/project-folder/REVIEWER.md`
  is deleted. There is no `/reviewer` slash command in this fork.
- Architect now runs two Codex slash commands:
  - `/codex:adversarial-review` against `handoff/ARCHITECT-BRIEF.md` before Builder starts.
    Output translated into the new `handoff/BRIEF-CRITIQUE.md` with an
    `## Architect Resolution` section that gates Builder spin-up.
  - `/codex:review` against Builder's working-tree changes after Builder is done.
    Output translated into the existing `handoff/REVIEW-FEEDBACK.md` (Must Fix /
    Should Fix / Escalate / Cleared).
- **Full-review prompts**: both gates explicitly request `critical + high + medium + low`
  findings via a `<completeness_contract>` block. Past sprints lost real issues because
  reviews were too narrow — Codex now surfaces everything, Architect filters at translation
  time. REVIEW-FEEDBACK and BRIEF-CRITIQUE templates explicitly require all severity levels.
- **Sub-agent fallback**: slash commands ship with `disable-model-invocation: true` and
  silently no-op from spawned agents or hooks. Each gate documents a direct CLI alternative
  via `codex-companion.mjs` that works in any non-interactive context.
- New doc: `docs/codex-integration.md` — gate prompts, schema mapping, decision table for
  slash command vs direct CLI, preserved review criteria (spec compliance, drift, security,
  logic, standards, known gaps).
- New handoff: `handoff/BRIEF-CRITIQUE.md` template.
- Setup script + new-setup.md updated: only ask about Arch and Bob (no Richard); ask about
  the codex plugin instead.
- **No fallback** if Codex is down: sprint blocks at the review gate. The whole point of
  the gate is the independent second opinion.
- Prerequisite: install the codex Claude Code plugin and run `/codex:setup` once per session.

This branch is a personal fork of `russelleNVy/three-man-team` v1.2.3; not pushed upstream.

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
