---
name: mission-control
description: >
  Coordination-first workflow for complex agent tasks. Use when an agent should
  verify context quality before planning, persist a working plan in repo docs
  for crash recovery, and break execution into explicit todo actions so multiple
  agents or sessions do not overlap or step on each other. Mission Control
  checks context strength, records the plan durably, assigns and tracks tasks,
  and keeps execution aligned until completion. Do not use for trivial,
  single-step changes that do not need durable planning or coordination.
license: MIT
compatibility: >
  Portable Agent Skill. Verified layouts for Claude Code (.claude/skills/),
  Cursor (.cursor/skills/), GitHub Copilot (.github/skills/), and Factory
  Droid (.factory/skills/). Requires normal file access to read/write repo docs
  and access to a todo/task tracker if the hosting agent provides one; if not,
  the agent should emulate the task list in its normal planning workflow.
metadata:
  version: "1.0.0"
  workflow: mission-control-4-step
---

# Mission Control

Mission Control is a coordination skill for non-trivial work that may span
multiple planning steps, multiple agents, or multiple sessions. It prevents an
agent from charging ahead with a weak understanding of the task, and it leaves
behind enough durable state that work can resume cleanly after interruption.

## Positioning

**Mission Control: validate context, persist plans, and coordinate execution.**

## When to use this skill

Use for tasks where one or more of the following is true:

- The request depends on subtle or incomplete context and the agent should
  assess whether it has enough signal before planning.
- The plan should survive session loss, crashes, or handoff to another agent.
- The work benefits from explicit task decomposition and ownership tracking.
- Multiple agents, branches of investigation, or follow-up sessions could work
  on the same task and need coordination boundaries.
- The task is long-running enough that a durable record of decisions and next
  actions is more valuable than an in-memory-only plan.

Skip this skill for tiny, obvious, single-session tasks where a normal direct
implementation flow is faster and safer.

## Workflow overview

```
Step 1: Check context strength  -->  Step 2: Persist mission plan
   (requirements, prior docs,         (write durable plan to repo docs,
    repo state, open questions)        include scope, assumptions, todo list)
                  |                                   |
                  v                                   v
       If context is weak, stop               Step 3: Coordinate execution
       and request missing context              (track todo actions, ownership,
                                                status, dependencies)
                                                            |
                                                            v
                                              Step 4: Close the mission loop
                                              (update docs, mark outcomes,
                                               capture follow-ups/risks)
```

## Step 1 — Check context strength before planning

Before making a plan, inspect the task context and explicitly judge whether it
is strong enough to support useful execution.

Evaluate at least these dimensions:

- **Goal clarity**: Is the requested outcome concrete and testable?
- **Scope clarity**: Is it clear what is in scope and what is out of scope?
- **Codebase grounding**: Has the agent found the relevant files, patterns,
  systems, or prior decisions?
- **Constraint coverage**: Are there known constraints such as security,
  compatibility, performance, deadlines, or style requirements?
- **Dependency awareness**: Does the task rely on other work, decisions, or
  external approvals?

Classify the context as:

- **Strong**: Enough signal to create a reliable plan.
- **Partial**: Enough to sketch a tentative plan, but with explicit open
  questions or risks.
- **Weak**: Not enough information to plan responsibly.

If context is **weak**, stop and request clarification before planning.
If context is **partial**, continue only if the plan clearly records the open
questions, assumptions, and decision points.

## Step 2 — Persist the mission plan in repo docs

Once context is strong enough, write the plan to a durable document in the
repository so another session can recover the work if needed.

The persisted plan should include:

- The task objective in plain language.
- The current context-strength assessment.
- Key findings from repository review or prior documentation.
- Assumptions being made.
- Open questions that could change the approach.
- A todo/action list with statuses.
- Coordination notes such as ownership boundaries, parallel work streams, or
  blocked items.
- Exit criteria for considering the mission complete.

Prefer appending or updating an existing shared planning document if the repo
already has one for this purpose. Otherwise use the repository's standard task
or timeline documentation location.

The point is not to create perfect documentation; it is to create durable,
recoverable operational state.

## Step 3 — Coordinate execution with explicit todo actions

Break the mission into concrete actions and track them as todo items.

Each todo should ideally capture:

- A short action-oriented title.
- Current state (todo, in progress, blocked, done).
- Dependencies or prerequisites.
- Owner, if multiple agents or humans are involved.
- Notes needed to avoid duplication or conflicts.

Coordination rules:

- Only one active owner should work a given action at a time.
- Mark blocked items explicitly rather than leaving them ambiguous.
- Use dependencies to sequence work and avoid stepping on related tasks.
- Update the durable plan whenever the action list materially changes.
- If a subtask branches into a separate investigation, record that boundary.

When the hosting environment provides a task/todo system, keep it aligned with
this plan. If not, maintain the task list in the durable document.

## Step 4 — Close the mission loop

At meaningful milestones and at task completion:

- Update the durable plan document with what changed.
- Mark completed, cancelled, or deferred todo items.
- Record decisions that future sessions need to know.
- Capture remaining risks, blockers, or follow-up work.
- Make it obvious whether the mission is complete, paused, or awaiting input.

A later session should be able to open the documentation and answer:

- What was the task?
- Was the context actually sufficient?
- What plan was chosen?
- What has already been done?
- What remains, and who should do it?
- What should be avoided to prevent overlapping work?

## Recommended durable plan template

```markdown
## Mission: <short title>

- **Objective**: ...
- **Context strength**: Strong / Partial / Weak
- **Relevant context**: ...
- **Assumptions**: ...
- **Open questions**: ...
- **Exit criteria**: ...

### Todo actions
- [ ] <action> — Owner: <agent/human or unassigned> — Status: todo
- [ ] <action> — Owner: <agent/human or unassigned> — Status: blocked
- [x] <action> — Owner: <agent/human or unassigned> — Status: done

### Coordination notes
- ...

### Progress log
- <date/time>: <decision, update, blocker, or handoff note>
```

## Notes for agents operating this skill

- State the context-strength assessment explicitly before presenting a plan.
- Treat durable documentation as part of the task, not optional bookkeeping.
- Keep todo items specific enough that ownership is unambiguous.
- Prefer updating a single source of truth rather than scattering plans across
  multiple files.
- If interrupted, leave the repo docs and task state in a form another agent
  can resume without guessing.
