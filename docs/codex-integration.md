# Codex Integration

Three Man Team uses two mandatory Codex review gates run by Architect. There is no
Claude-side Reviewer persona. If Codex is down, the sprint blocks until you fix it.

---

## Prerequisites

Before the first sprint each session:

```
/codex:setup
```

Expect `ready: true`, `loggedIn: true`. If not, run `codex login` and rerun setup.

The integration uses two Codex commands shipped with the codex Claude Code plugin:
- `codex:adversarial-review` — challenges the brief
- `codex:review` — reviews the build

Both write structured output that Architect parses into the team's handoff files.

### Slash command vs direct CLI — which to use

| Context | Use |
|---|---|
| Interactive user session (you typed at the prompt) | `/codex:adversarial-review` and `/codex:review` slash commands |
| Sub-agent / non-interactive Claude session | **Direct CLI** — slash commands have `disable-model-invocation: true` and will not fire from a spawned agent |
| Hook / automated script | Direct CLI |

**Direct CLI form** (works everywhere, including sub-agents):

```bash
node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" adversarial-review --wait --scope working-tree "<focus text>"
node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" review --wait --scope working-tree "<focus text>"
```

If `CLAUDE_PLUGIN_ROOT` is not set, hard-code the path. Glob to find the active version:

```
~/.claude/plugins/cache/openai-codex/codex/*/scripts/codex-companion.mjs
```

Output and structured-JSON contract are identical for slash command vs CLI. Architect translates the same way either way. **Past lesson**: a sub-session attempted `/codex:adversarial-review` and silently did nothing — direct CLI is the safe default when in doubt.

---

## Gate 1 — Adversarial Review (after the brief, before the build)

**When**: `handoff/ARCHITECT-BRIEF.md` is written. Bob has not started.

**Run** (full review — every severity, no filtering):

```
/codex:adversarial-review --wait --scope working-tree "Full critique of Step [N] brief. Surface ALL findings — critical, high, medium, AND low — covering design trade-offs, hidden assumptions, missed edge cases, scope risks, naming, doc gaps, perf hints, security implications. Do NOT filter to a brief summary. Past sprints missed real issues because reviews were too narrow."
```

**Why "full" instead of focused**: brief reviews look efficient but consistently miss
medium/low issues that matter (style drift, doc gaps, missed edges, hidden coupling).
Surface everything; Architect filters at resolution time, not at the prompt.

**Output**: Codex returns structured JSON (see `~/.claude/plugins/cache/openai-codex/codex/<version>/schemas/review-output.schema.json`).

**Translate to** `handoff/BRIEF-CRITIQUE.md`:
- Verdict (`approve` / `needs-attention`) verbatim
- **All findings, every severity**, ordered critical → high → medium → low. Do not drop medium/low.
- Architect Resolution: every finding gets accept / reject / escalate with a reason

**One-shot**. After Architect updates the brief based on accepted findings, the adversarial
review does not rerun. The update is trusted.

**Gate**: Bob does not start until `## Architect Resolution` is filled in.

---

## Gate 2 — Code Review (after Bob, before deploy)

**When**: Bob has written `handoff/REVIEW-REQUEST.md` and stopped.

**Run** (full review — every severity, no filtering):

```
/codex:review --wait --scope working-tree "Full review of the working-tree changes. Surface ALL findings — critical, high, medium, AND low. Include style, naming, doc gaps, dead code, perf hints, security implications, test coverage gaps, edge cases. Do NOT filter. Past sprints lost issues because reviews were too narrow."
```

In the prompt, paste the REVIEW-REQUEST.md `Files Changed` table and reference the
**Review Criteria** below — Codex needs to know what to look for.

**Why "full" instead of focused**: same reason as the adversarial gate. Past sprints
shipped with style/perf/doc issues that a narrow review skipped. Surface everything;
the Must Fix vs Should Fix split happens at translation time, not at the prompt.

**Output**: Same structured JSON contract.

**Translate to** `handoff/REVIEW-FEEDBACK.md`:

| Codex severity | REVIEW-FEEDBACK.md section |
|---|---|
| critical / high | `## Must Fix` (blocks step) |
| medium / low | `## Should Fix` (non-blocking, but logged) |
| product-decision | `## Escalate to Architect` (Architect escalates to Project Owner) |
| verdict: approve | `## Cleared` |

`Ready for Builder: YES` only if verdict is `approve` and there are no Must Fix items.
Otherwise `NO` — Bob fixes and resubmits.

**Never auto-apply Codex fixes**. Bob writes the code. Always.

**Gate**: no commit, no deploy until `Ready for Builder: YES`.

---

## Review Criteria (paste into the `/codex:review` prompt)

These are the same criteria a tight code review enforces. Codex needs them spelled out:

- **Spec compliance** — Did Bob build exactly what `handoff/ARCHITECT-BRIEF.md` asked? No more, no less?
- **Drift** — Did Bob add anything not in the brief?
- **Security** — Untrusted input handling, authorization checks, injection vectors, secret exposure.
- **Logic correctness** — Edge cases, error paths, failure modes.
- **Standards** — Project's established patterns (naming, layering, error handling).
- **Known gaps** — Did this step introduce or worsen anything in `handoff/BUILD-LOG.md` Known Gaps?

---

## Prompt composition (recipe blocks)

Per `codex:gpt-5-4-prompting`, Codex prompts use XML-tagged blocks. For both gates:

```xml
<task>
[concrete review job — paste the brief or REVIEW-REQUEST summary]
</task>

<completeness_contract>
Return EVERY finding you can substantiate, at every severity:
- critical, high, medium, AND low
- include style, naming, doc gaps, dead code, perf hints, test coverage gaps,
  edge cases, security implications, and minor inconsistencies
- do NOT pre-filter to "important issues" — Architect filters at translation time
- past sprints lost real bugs because reviews were too narrow
Aim for completeness over brevity. A long review with 12 low-severity items beats
a tight review that misses 3 medium ones.
</completeness_contract>

<grounding_rules>
- Cite file:line for every finding.
- Mark uncertainty explicitly. No unsupported claims.
- Do not propose fixes that exceed the diff scope.
</grounding_rules>

<structured_output_contract>
Return JSON matching review-output.schema.json:
verdict, summary, findings[severity, title, body, file, line_start, line_end, confidence, recommendation], next_steps.
findings[] MUST include all severities present in the change, not just critical/high.
</structured_output_contract>
```

For adversarial review, add:

```xml
<dig_deeper_nudge>
Challenge the design. Question trade-offs. Surface hidden assumptions.
What does this brief get wrong that would only show up in production?
Include nitpicks too — naming, doc gaps, test coverage holes — as low-severity findings.
</dig_deeper_nudge>
```

---

## Failure mode: Codex unreachable

No fallback. If `/codex:setup` fails or a slash command errors:

1. **Stop the sprint at the current gate.**
2. Run `codex login`. Rerun `/codex:setup`.
3. If `app-server` is broken, restart Claude Code session.
4. Do not deploy without review. Do not improvise a Claude-side review.

This is intentional. The whole point of the gate is the second opinion. Skipping it
defeats the architecture.

---

## Relationship to `/codex:setup --enable-review-gate`

Independent mechanism. The stop-time review gate (a Codex Stop hook) reviews the previous
turn's changes whenever Claude Code tries to end a session. That fires on session exit,
not at the team workflow gates.

You can run both at the same time — the stop-gate is a backstop; the team gates are the
primary review checkpoints. Or run only the team gates if you do not want stop-time prompts.
