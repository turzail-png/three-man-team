# Starting a Three Man Team Session

## First-time setup

```
You are the Architect on this project. Please read new-setup.md.
```

Arch will introduce the team, handle your project context file, and give you the prompt to use every session going forward.

---

## Architect Session (every session after setup)

```
You are [Architect name] on this project.
Read [your project file], then ARCHITECT.md.
```

## Builder Session (Architect spins this up as a sub-agent)

```
You are [Builder name] on this project.
Load token-optimizer skill first.
Then read BUILDER.md, then handoff/ARCHITECT-BRIEF.md.
Your task is Step [N]. Confirm the brief is complete before writing any code.
```

## Codex Review (Architect runs these — not a session, slash commands)

After `handoff/ARCHITECT-BRIEF.md` is written:

```
/codex:adversarial-review --wait --scope working-tree "challenge Step [N] brief: full critique, all severities (critical, high, medium, low)"
```

Architect translates the output into `handoff/BRIEF-CRITIQUE.md` and resolves findings before spinning up Builder.

After Builder writes `handoff/REVIEW-REQUEST.md`:

```
/codex:review --wait --scope working-tree "full review, all severities"
```

Architect translates the structured output into `handoff/REVIEW-FEEDBACK.md` (Must Fix / Should Fix / Escalate / Cleared) and decides `Ready for Builder: YES/NO`.

In sub-agent contexts (slash commands no-op there), use the direct CLI form:

```bash
node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" {adversarial-review,review} --wait --scope working-tree "<focus>"
```

## Resuming After a Break

If `handoff/SESSION-CHECKPOINT.md` exists and is recent, use the resume prompt inside it.
Otherwise:

```
You are [Architect name] on this project.
Read [your project file], then ARCHITECT.md, then handoff/BUILD-LOG.md.
Tell me where the project stands and what is next.
```

## Tips

- First session always uses new-setup.md. Every session after uses ARCHITECT.md.
- Let Architect report status before giving any instructions.
- If you know what you want to build, say so after Architect reports — not before.
- Keep the Architect session focused on planning and diagnosis. Build sessions are separate.
- Run `/codex:setup` once per session before the first review gate.
