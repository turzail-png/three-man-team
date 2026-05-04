<p align="center">
  <img src="assets/banner.png" alt="Three Man Team" width="100%">
</p>

---

## The Problem With AI Coding Tools

AI coding tools are powerful but undisciplined. They read entire codebases when they
need one function. They add features nobody asked for. They drift mid-task. They burn
tokens on every session doing work that didn't need to happen.

The solution isn't a better prompt. It's a process.

Three Man Team gives you two Claude agents with distinct jobs, a Codex review gate at every handoff, and rules that prevent the most expensive failure modes. The Architect plans and deploys. The Builder builds exactly what the brief says. Codex challenges the brief and reviews the build — independent second opinion, every step.

---

## Why Two Claude Agents Plus Codex

DeepMind's multi-agent research shows teams of 3-5 with structured artifact handoffs
outperform both solo agents and larger groups. Three is not arbitrary — it is the
minimum for meaningful review and the maximum before coordination overhead eats the gain.

The original Three Man Team used a Claude Reviewer for the third role. This version
swaps that role for Codex (`/codex:adversarial-review` on the brief, `/codex:review` on
the build). The independent model is the point — a second Claude reviewing Claude's work
shares too many blind spots.

The roles map to how real software ships:
- Someone who understands the whole system and owns the deploy (Architect — Claude)
- Someone who builds fast and clean (Builder — Claude)
- An independent second opinion that catches what the builder missed (Codex)

---

## Quick Start

### Install globally (30 seconds)

Open your terminal and run:

```bash
git clone https://github.com/russelleNVy/three-man-team.git ~/.claude/skills/three-man-team
cd ~/.claude/skills/three-man-team && ./setup
```

Then tell Claude Code:
> Install three-man-team: add a three-man-team section to CLAUDE.md listing the available agents:
> /architect, /builder. Set token-optimizer rules as always active. Review is run by Codex
> (`/codex:adversarial-review` and `/codex:review`); install the codex plugin and run `/codex:setup` first.

### Install per project

```bash
git clone https://github.com/russelleNVy/three-man-team.git .claude/skills/three-man-team
cd .claude/skills/three-man-team && ./setup
```

### Copy the files into your project

```bash
# Pick a template — named personas (recommended) or blank slate
cp -r ~/.claude/skills/three-man-team/templates/project-folder/* /path/to/your/project/

# Copy the handoff files
cp ~/.claude/skills/three-man-team/handoff/* /path/to/your/project/
```

### Start your first session

Copy this into your Claude Code session:
```
You are the Architect on this project. Read CLAUDE.md, then ARCHITECT.md.
Report project status in one paragraph, then wait for me.
```

---

## The Workflow

<p align="center">
  <img src="assets/workflow.png" alt="Three Man Team Workflow" width="100%">
</p>

Every unit of work follows the same path. Architect plans and writes the brief. Codex runs `/codex:adversarial-review` against the brief — Architect resolves findings before Builder starts. Builder reads the brief, shows a plan, builds, and writes REVIEW-REQUEST.md. Architect runs `/codex:review` and translates the output into REVIEW-FEEDBACK.md. Architect deploys with the Project Owner's go-ahead. Nothing skips a step.

See [docs/codex-integration.md](docs/codex-integration.md) for the gate details.

---

## The Team

<p align="center">
  <img src="assets/role-cards-cropped.png" alt="Arch and Bob" width="100%">
</p>

Two Claude agents. One Codex review at every gate. Built to work together.

Architect and Builder are the defaults. Rename them to anything — see `config/team.yml.example` and `docs/customizing-your-team.md`. Review is run by Codex and is not a renamable persona.

---

## Token Optimization

Every session starts with five rules baked into CLAUDE.md:

```
Is this in a skill or memory?   → Trust it. Skip the file read.
Is this speculative?            → Kill the tool call.
Can calls run in parallel?      → Parallelize them.
Output > 20 lines you won't use → Route to subagent.
About to restate what user said → Delete it.
```

Install your `token-optimizer` skill globally at `~/.claude/skills/token-optimizer` or
reference it from your project's `.claude/skills/` directory.

For bash output compression on top of these rules, see [RTK](https://github.com/rtk-ai/rtk) —
a separate tool that compresses `find`, `ls`, `grep` output before it reaches Claude's context.
Not required, but recommended for heavy Claude Code CLI users. The combination of RTK (bash layer)
+ token-optimizer (behavior layer) is where real savings compound.

See `docs/token-optimization.md` for the full discipline.

---

## Templates

- `templates/project-folder/` — **Start here.** Named personas (Arch, Bob), fully written and ready to use. Customize the Who You Are sections and rename to fit your team. Review is handled by Codex, not a third persona.
- `templates/generic/` — Blank slate with `[CUSTOMIZE]` placeholders. Use this if you want to build your own personas from scratch or install globally across all projects.

See `docs/customizing-your-team.md` for the full walkthrough.

---

## License

MIT. Free forever.

---

## Built By

Russell Aaron — 20+ years building and supporting software the right way. He built this team
in production shipping a real SaaS platform. It works because it was used before it was
published, fine tuned, and will continue to get better over time as AI models and tools evolve.
