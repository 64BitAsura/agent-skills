# agent-skills

Reusable [Agent Skills](https://agentskills.io/specification) (`SKILL.md`)
for coding agents.

## Skills

### `gated-agentic-development`

A gated, human-in-the-loop workflow for planning and implementing
non-trivial development tasks:

1. **Analyze** the task using deep reasoning, prior context from
   `project-timeline.md`, and current best-practices research.
2. **Present 2-5 implementation options** with confidence ratings,
   pros/cons, and a recommendation. If the top option is ≥ 80% confident
   and the task closely matches existing patterns in the repo, the agent
   may proceed straight to implementation; otherwise a decision gate is
   triggered.
3. **Human-in-the-loop**: when the gate isn't cleared, a human picks an
   option or asks the agent to analyze further.
4. **Implement**, adding unit, integration, and/or end-to-end tests as
   appropriate to the change.
5. **Document** the task, chosen approach, tests, design pitfalls, and
   residual risks in `project-timeline.md`, in language-agnostic terms.

The same skill content is provided for each supported agent, since all of
them implement the portable Agent Skills standard:

| Agent           | Path                                                   |
|-----------------|---------------------------------------------------------|
| Claude Code     | `.claude/skills/gated-agentic-development/SKILL.md`     |
| Cursor          | `.cursor/skills/gated-agentic-development/SKILL.md`     |
| Factory Droid   | `.factory/skills/gated-agentic-development/SKILL.md`    |
| GitHub Copilot  | `.github/skills/gated-agentic-development/SKILL.md`     |

See `project-timeline.md` for the running log of decisions this skill
produces.