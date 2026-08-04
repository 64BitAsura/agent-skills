---
name: gated-agentic-development
description: >
  Gated, human-in-the-loop workflow for planning and implementing non-trivial
  development tasks (new features, bug fixes, refactors, migrations). Use
  whenever a coding task is ambiguous, architecturally significant, or has
  more than one reasonable implementation approach. Enforces a confidence
  gate: the agent must analyze the task, research current best practices,
  present 2-5 ranked implementation options with confidence ratings and
  pros/cons, and only proceed autonomously to implementation when an option
  clears an 80% confidence bar AND the task closely matches existing
  patterns in the codebase. Otherwise it stops and asks the human to decide.
  All decisions, tests, and residual risks are documented in
  project-timeline.md. Do not use for trivial, single-obvious-answer changes
  (typo fixes, formatting, version bumps, one-line config edits).
license: MIT
compatibility: >
  Portable Agent Skill. Verified layouts for Claude Code (.claude/skills/),
  Cursor (.cursor/skills/), GitHub Copilot (.github/skills/), and Factory
  Droid (.factory/skills/). No OS or network requirements beyond normal
  agent tool access; Step 1's "check the internet" research is best-effort
  and skips gracefully if network access is unavailable.
metadata:
  version: "1.0.0"
  workflow: gated-5-step
---

# Gated Agentic Development

A 5-step, gate-controlled workflow that stops an agent from silently
committing to a risky or ambiguous implementation. The agent must earn the
right to proceed autonomously by demonstrating high confidence in a
well-researched option that closely matches existing project patterns.
Otherwise, a human makes (or helps make) the call.

## When to use this skill

Use for any task where the "how" is not obvious, including:

- New features or non-trivial bug fixes.
- Refactors that touch shared/core code, public APIs, or data models.
- Anything with more than one reasonable technical approach (library choice,
  architecture pattern, data flow, storage strategy, etc.).
- Tasks explicitly flagged by the user as needing a plan/design review.

Skip this skill for trivial, single-obvious-answer changes (typo fixes,
formatting, dependency version bumps, copy edits) — just make the change.

## Workflow overview

```
Step 1: Analyze                Step 2: Present options + gate check
  (thinking model,        -->    (2-5 options, confidence %,     --+
   project-timeline.md,          pros/cons, recommendation)       |
   best-practices research)                                       |
                                                                    |
        Gate: best option >= 80% confidence                        |
              AND task closely matches existing code/patterns?     |
                                                                    |
        YES ----------------------------------------------+       |
         |                                                 |       |
         v                                                 v       |
   Step 4: Implement                              Step 3: Human-in-the-loop
   (recommended option,                             (present options +
    tests per nature of change)                      confidence + questions;
         |                                            wait for decision or
         v                                            more analysis request)
   Step 5: Document in                                       |
   project-timeline.md                                       |
   (summary, pitfalls, risks)  <-----------------------------+
                                (after human decides, go to Step 4)
```

## Step 1 — Analyze the task

1. Use extended/deep reasoning ("thinking mode") to fully understand the
   request: goal, constraints, affected code paths, non-functional
   requirements (performance, security, backwards compatibility).
2. Read `project-timeline.md` at the repository root for prior related
   decisions, known pitfalls, and residual risks that might apply to this
   task. If it does not exist yet, create it (see the template at the
   bottom of this file) so future tasks have somewhere to read from and
   write to.
3. Explore the existing codebase for established patterns, conventions, and
   prior art that solve similar problems — reuse and consistency beat
   novelty.
4. When useful, research current best practices on the internet (official
   docs, changelogs, security advisories, framework release notes). Prefer
   authoritative/primary sources and note what you found and where. If
   internet access is unavailable, note that research was codebase-only and
   proceed — do not block the workflow on this step.

## Step 2 — Present options and check the confidence gate

Present **2-5 distinct implementation options** (not just parameter
variations of one idea) using this format:

```markdown
### Option A: <name>
- **Confidence**: <0-100>%
- **Pros**: ...
- **Cons**: ...

### Option B: <name>
- **Confidence**: <0-100>%
- **Pros**: ...
- **Cons**: ...

**Recommendation**: Option <X>, because <reasoning>.
```

Confidence reflects how sure the agent is that the option will work
correctly, fit the codebase's conventions, and not need significant rework —
not how much effort it takes.

**Gate logic** (evaluate after presenting options):

- **If the top-recommended option's confidence is ≥ 80% AND the task is
  substantially similar to existing, already-solved code/patterns in this
  repo** → the gate is satisfied. Skip Step 3 and proceed directly to
  Step 4 with the recommended option, but still show the options/ratings to
  the human for visibility before continuing.
- **Otherwise** (no option reaches 80%, the task is novel/architecturally
  significant, or there is meaningful ambiguity/risk) → the gate is **not**
  satisfied. Go to Step 3 and do not implement yet.

## Step 3 — Human-in-the-loop (only when the gate is not satisfied)

1. Stop before writing implementation code.
2. Ask the human to either:
   - Pick one of the presented options, or
   - Request deeper analysis (e.g., a spike, prototype, more research, or
     clarification of requirements) before choosing, or
   - Answer specific open questions the agent lists (assumptions that need
     confirming, missing requirements, trade-offs only the human can
     weigh — e.g., cost, org policy, product priorities).
3. If more analysis is requested, return to Step 1/Step 2 with the new
   information and re-run the gate check.
4. Once the human decides, proceed to Step 4 with the chosen option.

## Step 4 — Implementation

1. Implement the chosen (recommended or human-selected) option with small,
   incremental, reviewable changes.
2. Add automated tests appropriate to the nature of the change:
   - **Unit tests** for new/changed logic in isolation.
   - **Integration tests** where components, modules, or services interact.
   - **End-to-end tests** where the change affects user-facing flows or
     cross-system behavior.
   Not every change needs all three — use judgment based on blast radius,
   but justify any category you skip in Step 5's documentation.
3. Run existing lint/build/test tooling for the affected area and fix
   failures introduced by the change.

## Step 5 — Document in project-timeline.md

Append an entry to `project-timeline.md` (template included in that file)
covering, in **language-agnostic** terms (describe behavior/capability, not
syntax):

- The task and the options considered, with their confidence ratings.
- The approach chosen and why (including whether a human overrode the
  recommendation).
- Whether Step 3 (human-in-the-loop) was triggered, and its outcome.
- A summary of the implementation and the tests added.
- Design pitfalls or shortcomings of the shipped approach (trade-offs
  knowingly accepted).
- Residual risks (what could still break, and how it's mitigated/monitored).
- Any deferred follow-up work.

This keeps `project-timeline.md` useful as living institutional memory for
both humans and future agent runs (which read it again in Step 1).

## Notes for agents operating this skill

- Never silently skip the gate check — always state explicitly whether it
  was satisfied and why.
- If the human overrides a low-confidence recommendation, record that
  explicitly in Step 5 rather than presenting it as a normal high-confidence
  pick.
- Keep option presentations concise; the goal is fast, informed human
  decisions, not exhaustive essays.
