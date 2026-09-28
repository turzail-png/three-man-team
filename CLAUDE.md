# Three Man Team — Session Router

## Token Rules — Always Active

```
Is this in a skill or memory?   → Trust it. Skip the file read.
Is this speculative?            → Kill the tool call.
Can calls run in parallel?      → Parallelize them.
Output > 20 lines you won't use → Route to subagent.
About to restate what user said → Delete it.
```

Grep before Read. Never read a whole file to find one thing.
Do not re-read files already in context this session.

---

## Session Start — Every Role

1. Load your token-optimizer skill if you have one — first, before anything else.
2. Check `handoff/SESSION-CHECKPOINT.md` — if active and recent, read it. That is your state.
3. Load your role file — copied to project root → `ARCHITECT.md` · `BUILDER.md`
4. If no checkpoint — Architect reads `handoff/BUILD-LOG.md` + `handoff/ARCHITECT-BRIEF.md` only.

**Project Owner role is set by the human. Do not ask.**

Code review is not a Claude role anymore — it is Codex. See `## Codex Review Gates` below.

---

## Codex Review Gates

Two mandatory gates are run by Architect via Codex slash commands. No Claude-side reviewer,
no fallback. If Codex is down, the sprint blocks until `/codex:setup` is fixed.

| Gate | When | Command (interactive) | Output |
|---|---|---|---|
| Adversarial (plan) | After `handoff/ARCHITECT-BRIEF.md` is written, before Builder starts | `/codex:adversarial-review --wait --scope working-tree "<focus>"` | `handoff/BRIEF-CRITIQUE.md` |
| Review (code) | After Builder writes `handoff/REVIEW-REQUEST.md` | `/codex:review --wait --scope working-tree "<focus>"` | `handoff/REVIEW-FEEDBACK.md` |

## CI/CD Discipline: Always Required

Every brief, build, and review MUST address CI/CD impact. This is non-negotiable as of 2026-05-08.

**The full local CI gate runs ONCE, AFTER Codex review, not twice (2026-06-16).**
Running the heavy gate (`test:coverage + vibecop + lint + build`) before the review AND
again after wasted a full local CI cycle every sprint: if Codex returns Must Fix, the
pre-review run was on now-stale code; if Codex is clean, the code is unchanged so one run
after review suffices. Either way the FINAL code is validated exactly once. (This is local
time/tokens, not GitHub Actions minutes; pr-gate.yml is the separate Actions gate.)

**Architect**: every `ARCHITECT-BRIEF.md` includes a `## CI/CD Impact` section listing new files,
required tests, expected coverage delta, vibecop expectations, and any `pr-gate.yml` edits.

**Builder, BEFORE signaling done (fast pre-review check only)**: run `typecheck` + `build`
(plus any test that directly exercises a just-written helper). This is the cheap "does it even
compile / is the RSC/bundle valid" gate so Codex never reviews broken code. Do NOT run the
full `test:coverage` or `vibecop` here. Every new pure helper under `src/lib/**` or
`src/app/api/**/route.ts` still ships with at least one test in the same PR, never as a
follow-up. REVIEW-REQUEST.md records the fast-gate result (`typecheck + build PASS`) and notes
that full coverage/vibecop numbers are produced post-review.

**Builder, AFTER Codex review (Must Fix applied), before the Deploy Gate**: run the FULL local
CI gate once on the final code: `lint + typecheck + test:coverage + build + vibecop scan`.
Record concrete numbers (coverage %, vibecop errors/warnings/info delta) in BUILD-LOG.md.
The Deploy Gate does not proceed until this full gate is green.

**Deploy Gate blocks (Must Fix) if any of these are missing after the post-review full run:**
- Tests for new helpers
- Coverage % concrete numbers in BUILD-LOG.md
- Vibecop scan delta (errors/warnings/info)
- 0 vibecop errors after the change
- Coverage didn't drop > 1pp without compensating tests

Sub-agent / non-interactive contexts (slash commands no-op there): use the direct CLI form
`node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" {adversarial-review,review} ...`.

Prerequisite each session: `/codex:setup` reports `ready: true` and `loggedIn: true`.
Prompt composition + review criteria + slash-vs-CLI decision table: see `docs/codex-integration.md`.

---

## Reference Files — On Demand Only

| File | Load when |
|---|---|
| Project spec | Architect needs it; checkpoint doesn't cover it |
| handoff/ARCHITECT-BRIEF.md | Builder loads at task start |
| handoff/BRIEF-CRITIQUE.md | Architect writes after adversarial review; Builder may skim for context |
| handoff/BUILD-LOG.md | Architect checks status; Builder updates when done |
| handoff/REVIEW-REQUEST.md | Architect loads before running `/codex:review` |
| handoff/REVIEW-FEEDBACK.md | Builder loads when returned for fixes |

---

## Handoff Files

All team communication flows through files in `handoff/`:
- `ARCHITECT-BRIEF.md` — Architect writes, Builder reads
- `BRIEF-CRITIQUE.md` — Codex adversarial output, Architect resolves
- `REVIEW-REQUEST.md` — Builder writes, Architect reads before Codex review
- `REVIEW-FEEDBACK.md` — Architect writes (from Codex review output), Builder reads
- `BUILD-LOG.md` — shared record, Architect owns
- `SESSION-CHECKPOINT.md` — Architect writes at session end

Run `./setup` to copy agent files into your project.
