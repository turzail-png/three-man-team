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
2. Check `SESSION-CHECKPOINT.md` — if active and recent, read it. That is your state.
3. Load your role file: `agents/ARCHITECT.md` · `agents/BUILDER.md`
4. If no checkpoint — Architect reads `BUILD-LOG.md` + `ARCHITECT-BRIEF.md` only.

**Project Owner role is set by the human. Do not ask.**

Review is not a Claude role anymore — it is Codex. See `## Codex Review Gates` below.

---

## Codex Review Gates

Two mandatory gates are run by Architect via Codex slash commands. No Claude-side reviewer,
no fallback. If Codex is down, the sprint blocks until `/codex:setup` is fixed.

| Gate | When | Command | Output |
|---|---|---|---|
| Adversarial (plan) | After `ARCHITECT-BRIEF.md` is written, before Builder starts | `/codex:adversarial-review --wait --scope working-tree "<focus>"` | `BRIEF-CRITIQUE.md` |
| Review (code) | After Builder writes `REVIEW-REQUEST.md` | `/codex:review --wait --scope working-tree` | `REVIEW-FEEDBACK.md` |

Prerequisite each session: `/codex:setup` reports `ready: true` and `loggedIn: true`.
Prompt composition + review criteria: see `docs/codex-integration.md`.

---

## Reference Files — On Demand Only

| File | Load when |
|---|---|
| Project spec | Architect needs it; checkpoint doesn't cover it |
| ARCHITECT-BRIEF.md | Builder loads at task start |
| BRIEF-CRITIQUE.md | Architect writes after adversarial review; Builder may skim |
| BUILD-LOG.md | Architect checks status; Builder updates when done |
| REVIEW-REQUEST.md | Architect loads before `/codex:review` |
| REVIEW-FEEDBACK.md | Builder loads when returned for fixes |

---

## Handoff Files

All team communication flows through files in `handoff/`:
- `ARCHITECT-BRIEF.md` — Architect writes, Builder reads
- `BRIEF-CRITIQUE.md` — Codex adversarial output, Architect resolves
- `REVIEW-REQUEST.md` — Builder writes, Architect reads before Codex review
- `REVIEW-FEEDBACK.md` — Architect writes (from Codex review output), Builder reads
- `BUILD-LOG.md` — shared record, Architect owns
- `SESSION-CHECKPOINT.md` — Architect writes at session end

Copy templates from `handoff/` into your project root to get started.
