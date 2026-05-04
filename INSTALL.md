# Installing Three Man Team

## Global Install — All Your Projects

Three Man Team installs into `~/.claude/skills/three-man-team` and is available in every project.

```bash
git clone https://github.com/russelleNVy/three-man-team.git ~/.claude/skills/three-man-team
cd ~/.claude/skills/three-man-team && ./setup
```

Then add to your global `~/.claude/CLAUDE.md`:
```
## Three Man Team
Available agents: /architect, /builder
Review: run by Codex (/codex:adversarial-review and /codex:review). Install the codex plugin and run /codex:setup once per session.
Token rules always active — see three-man-team/CLAUDE.md
```

## Per-Project Install

Three Man Team installs into `.claude/skills/three-man-team` inside your repo and only applies to that project.

```bash
git clone https://github.com/russelleNVy/three-man-team.git .claude/skills/three-man-team
cd .claude/skills/three-man-team && ./setup
```

Add to your project's `CLAUDE.md`:
```
## Three Man Team
Available agents: /architect, /builder
Review: run by Codex (/codex:adversarial-review and /codex:review). Install the codex plugin and run /codex:setup once per session.
```

## VS Code / Cursor / Codex

Three Man Team uses the SKILL.md standard and works with any agent that supports context files.
Copy the `agents/` directory and `CLAUDE.md` into your project root.

## Requirements

- Claude Code CLI (for slash command support)
- The codex Claude Code plugin (for `/codex:adversarial-review` + `/codex:review` review gates)
- Git
- Any agent supporting CLAUDE.md / SKILL.md context (for other tools)
