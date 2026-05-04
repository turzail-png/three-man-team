# Changelog

## v2.0.0 — 2026-05-04

**Breaking change**: Reviewer role replaced with Codex review gates.

- The Claude `Reviewer` persona is removed. `agents/REVIEWER.md` and the Reviewer
  templates are deleted. The `/reviewer` slash command no longer exists.
- Architect now runs two Codex slash commands:
  - `/codex:adversarial-review` against `ARCHITECT-BRIEF.md` before Builder starts.
    Output translated into the new `BRIEF-CRITIQUE.md` handoff file with an
    `## Architect Resolution` section that gates Builder spin-up.
  - `/codex:review` against Builder's working-tree changes. Architect translates
    the structured findings (severity-ordered) into the existing `REVIEW-FEEDBACK.md`
    format (Must Fix / Should Fix / Escalate / Cleared).
- New required handoff: `handoff/BRIEF-CRITIQUE.md`.
- `config/team.yml.example`: added `codex_review` block, removed `reviewer` block.
- New doc: `docs/codex-integration.md` — gate prompts, schema mapping, preserved
  review criteria (spec compliance, drift, security, logic, standards, known gaps).
- **Full-review prompts**: both gate prompts explicitly request `critical + high + medium + low`
  findings. Past sprints lost real issues because reviews were too narrow — the new
  `<completeness_contract>` block tells Codex to surface everything (style, naming,
  doc gaps, test coverage, perf hints) and let Architect filter at translation time.
  REVIEW-FEEDBACK and BRIEF-CRITIQUE templates explicitly require all severity levels.
- **Sub-agent fallback** documented for both gates. The slash commands ship with
  `disable-model-invocation: true` and silently no-op when called from a spawned
  agent or hook. Each gate now lists a direct CLI alternative
  (`node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" {adversarial-review,review} ...`)
  that works in any non-interactive context. `docs/codex-integration.md` adds a
  "Slash command vs direct CLI" decision table.
- No fallback: if Codex is down (`/codex:setup` not green), the sprint blocks at
  the review gate. The whole point of the gate is the independent second opinion.
- Prerequisite: install the codex Claude Code plugin and run `/codex:setup` once
  per session.

Migration: existing projects can keep their handoff files. Delete any local
`REVIEWER.md` copy. Add `BRIEF-CRITIQUE.md` from `handoff/`. Update the local
CLAUDE.md to remove `/reviewer` from the available agents line and add the Codex
review gate prerequisite.

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
