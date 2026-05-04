# A Complete Three Man Team Sprint

This is what a full step looks like from start to deploy.

## 1. Project Owner describes a problem to Architect

> "The user registration form doesn't validate email format on the server side.
> It only validates in JavaScript."

## 2. Architect diagnoses

Architect reads the relevant file, confirms the gap, and describes what the code
currently does. Then asks: do you want server-side validation only, or both?

Project Owner: "Both. JS for UX, server-side for security."

## 3. Architect writes the brief

Updates ARCHITECT-BRIEF.md:

```
## Step 12 — Server-side email validation on registration
- Add server-side email format validation to the registration handler
- Validate after sanitization, before DB write
- Return error message matching the existing error format
- JS validation already exists — do not modify it
- Flag: use the framework's built-in validator, not a custom regex
```

## 4. Architect runs adversarial review on the brief

```
/codex:adversarial-review --wait --scope working-tree "challenge Step 12 brief: design trade-offs, hidden assumptions, missed edge cases"
```

Codex returns findings: "What about empty-string after sanitization? What about
internationalized domain names? Validator returns generic error — does it match the
i18n contract?"

Architect writes BRIEF-CRITIQUE.md, accepts the i18n finding (updates ARCHITECT-BRIEF
to flag it), rejects the empty-string concern (sanitization layer handles it), and
escalates the IDN question to Project Owner. Resolution filled in.

## 5. Architect spins up Builder

> You are Bob on this project. Load token-optimizer skill first.
> Then read BUILDER.md, then ARCHITECT-BRIEF.md.
> Your task is Step 12.

## 6. Builder builds

Builder reads the brief, shows a one-line plan, gets Architect's nod, and makes the
change. Updates BUILD-LOG. Writes REVIEW-REQUEST.md.

## 7. Architect runs Codex review

```
/codex:review --wait --scope working-tree
```

Codex returns structured findings: validator used correctly, error format matches —
but the error string is not wrapped in i18n helper, contradicting Builder's own brief
note. Severity: high.

## 8. Architect writes REVIEW-FEEDBACK.md

- `## Must Fix`: error string not i18n-wrapped (file:line + recommendation).
- `## Cleared` empty.
- `Ready for Builder: NO`.

## 9. Builder fixes

Wraps the error string in the project's i18n helper. Resubmits.

## 10. Architect reruns Codex review

```
/codex:review --wait --scope working-tree
```

Verdict: approve. Architect writes `## Cleared` line, sets `Ready for Builder: YES`.

## 11. Architect deploys

Tells Project Owner: "Server-side email validation added. Codex flagged the error
string wasn't translatable — Bob fixed it. Clean."

Project Owner: "Ship it."

Architect commits, deploys, confirms, updates BUILD-LOG and SESSION-CHECKPOINT.
