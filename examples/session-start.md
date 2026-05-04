# Starting a Three Man Team Session

## Architect Session (most common)

```
You are [Architect name] on this project.
Read CLAUDE.md, then ARCHITECT.md.
Report project status in one paragraph, then wait for me.
```

## Builder Session (Architect spins this up as a sub-agent)

```
You are [Builder name] on this project.
Load token-optimizer skill first.
Then read BUILDER.md, then ARCHITECT-BRIEF.md.
Your task is Step [N]. Confirm the brief is complete before writing any code.
```

## Codex Review (Architect runs these — not a session, slash commands)

After ARCHITECT-BRIEF.md is written:

```
/codex:adversarial-review --wait --scope working-tree "challenge Step [N] brief: design trade-offs, hidden assumptions, missed edge cases, scope risks"
```

Architect translates the output into BRIEF-CRITIQUE.md and resolves findings before
spinning up Builder.

After Builder writes REVIEW-REQUEST.md:

```
/codex:review --wait --scope working-tree
```

Architect translates the structured output into REVIEW-FEEDBACK.md (Must Fix / Should
Fix / Escalate / Cleared) and decides `Ready for Builder: YES/NO`.

## Resuming After a Break

If SESSION-CHECKPOINT.md exists and is recent, use the resume prompt inside it.
Otherwise:

```
You are [Architect name] on this project.
Read CLAUDE.md, then ARCHITECT.md, then BUILD-LOG.md.
Tell me where the project stands and what is next.
```

## Tips

- Always start with Architect, not Builder.
- Let Architect report status before giving any instructions.
- If you know what you want to build, say so after Architect reports — not before.
- Keep the Architect session focused on planning and diagnosis. Build sessions are separate.
- Run `/codex:setup` once per session before the first review gate.
