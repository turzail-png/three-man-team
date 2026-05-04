# Three Man Team — Methodology

Why this works, and the research behind it.

---

## Personas Over Labels (for Claude roles)

Telling an AI "you are a builder" produces generic building behavior. Giving the AI a
character — a backstory, a set of values, a voice, a specific reason they care about
the work — activates a richer cluster of behavior.

This is vocabulary routing: precise role framing activates relevant training patterns
more effectively than abstract job titles. Architect and Builder in Three Man Team are
not generic agents. They have histories, opinions, and standards. The specificity is
the point.

Code review is the exception. It is run by Codex, not a Claude persona — the value of
a review gate is the **independent model**, not a richer reviewing persona. A second
Claude reviewing Claude's work shares too many blind spots to be a real check.

---

## Three Roles, One Independent

DeepMind's multi-agent scaling research shows that structured teams of 3-5 agents with
defined artifact handoffs consistently outperform both solo agents and larger teams.
Solo agents drift. Large teams generate coordination overhead that eats the productivity
gain. Three is the sweet spot: enough for meaningful review, minimal enough for clean
handoffs.

Three Man Team keeps three roles by design: Architect (Claude), Builder (Claude), and
Codex review. The third is independent — different model, different blind spots —
because that is what makes the review gate meaningful. Resist adding a fourth role.

---

## Handoffs Through Files, Not Conversation

The agents and gates communicate through structured files:
- Architect writes `handoff/ARCHITECT-BRIEF.md`
- Codex (via `/codex:adversarial-review`) → Architect writes `handoff/BRIEF-CRITIQUE.md`
- Builder writes `handoff/REVIEW-REQUEST.md`
- Codex (via `/codex:review`) → Architect writes `handoff/REVIEW-FEEDBACK.md`

This is not just organization. It means each agent and gate starts with a clean context
window reading only what they need for their specific job. Builder never loads the full
spec. Codex never loads the schema. Token waste is structural, not behavioral — fix the
structure and the behavior follows.

---

## The Deploy Gate

Nothing ships without Architect's sign-off and the Project Owner's awareness. This is
not bureaucracy — it is accountability. The Project Owner knows what is going live.
The Architect has confirmed it passed Codex review. The Builder never touches the deploy
target directly. Codex never touches it either — it produces findings, nothing more.

This pattern eliminates the most expensive class of AI mistake: changes that were
technically correct but wrong for the project, shipping without anyone noticing.

---

## Token Discipline as Infrastructure

Token waste is not a Claude problem or a prompt problem. It is a context architecture
problem. The five rules in Three Man Team's CLAUDE.md are not guidelines — they are operating rules that fire before every tool call. The cost of re-reading a file you already have in context is paid every time. The cost of the rules is paid once, at session start.

Grep before Read. Never speculate. Parallelize when possible. Route large outputs to
subagents. Never restate what the user said.

---

## Scope Lock

One step at a time. The next step does not start until the current step is reviewed,
cleared, and deployed. Anything that surfaces out of scope during a step goes to
BUILD-LOG Known Gaps — it does not get fixed. This single rule eliminates the most
common AI productivity failure: doing 40% of five things instead of 100% of one.
