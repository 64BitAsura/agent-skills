# Project Timeline

This file is the running decision & documentation log for the
**gated-agentic-development** skill. Agents read it at the start of a task
(Step 1) for prior context and append an entry to it after finishing a task
(Step 5). Humans can also read it to understand *why* something was built the
way it was, not just *what* was built.

Keep entries in reverse-chronological order (newest first). Do not delete or
rewrite historical entries — if a decision is later reversed, add a new entry
that supersedes the old one and link back to it.

---

## How to add an entry (Step 5 template)

Copy this block, fill it in, and add it to the top of the "Entries" section
below.

```markdown
### <YYYY-MM-DD> — <short task title>

- **Task**: One or two sentences describing what was requested.
- **Options considered**: List of the option names/summaries evaluated in
  Step 2, with the confidence rating given to each.
- **Chosen approach**: Which option was implemented and why (recommendation
  accepted as-is, human override, or only-one-viable-option).
- **Human-in-the-loop**: Yes/No — if yes, summarize what the human decided
  and any follow-up questions that were resolved.
- **Implementation summary** (language/framework agnostic): What changed,
  described in terms of behavior/capability, not specific syntax.
- **Tests added**: Unit / integration / e2e — what was covered and what was
  intentionally left out and why.
- **Design pitfalls / shortcomings**: Known limitations of the approach that
  was shipped, even if they were accepted as trade-offs.
- **Residual risks**: What could still go wrong (perf, security, scaling,
  data migration, backwards compatibility, etc.) and how they are mitigated
  or monitored.
- **Follow-ups**: Any deferred work or open questions for future tasks.
```

---

## Entries

### 2026-08-04 — Add Be Simple skill

- **Task**: Add a communication skill that instructs agents to explain things plainly, match the developer's writing style and language proficiency, and never pad responses with filler or verbosity.
- **Options considered**: Embed the rules in each existing skill's "Notes for agents" section (40% confidence — scatters the rules, easy to miss); add a standalone portable skill that stacks with other skills (90% confidence — clean separation, reusable).
- **Chosen approach**: Standalone skill so it can be applied independently or layered on top of any workflow skill.
- **Human-in-the-loop**: No — confidence cleared the gate and the task matches the existing skill-creation pattern.
- **Implementation summary**: Added the `be-simple` SKILL.md to all four agent directories with ten core rules covering brevity, filler avoidance, style-matching, and language-proficiency adaptation. Updated the repository README with the new skill table.
- **Tests added**: None — documentation-only change with no executable behavior.
- **Design pitfalls / shortcomings**: "Short" is subjective; different developers may want different thresholds. The skill gives a rough ~15-line guideline but can't enforce it mechanically.
- **Residual risks**: Agents may over-compress and drop useful detail. The skill mitigates this with rule 5 ("keep all facts and meaning").
- **Follow-ups**: Consider adding concrete before/after examples if users want a reference.

### 2026-08-04 — Add Mission Control skill

- **Task**: Add a new portable skill named Mission Control to help agents check context strength before planning, persist plans in repository docs for crash recovery, and coordinate execution with explicit todo actions.
- **Options considered**: Reuse the existing gated planning skill with minor wording changes (55% confidence); add a separate coordination-focused skill alongside the existing planning skill (92% confidence); create agent-specific variants with different wording (40% confidence).
- **Chosen approach**: Added a separate portable skill so the repository keeps the existing gated planning workflow intact while introducing a distinct coordination-oriented workflow with a clearer name and purpose.
- **Human-in-the-loop**: Yes — the human selected the Mission Control name before implementation.
- **Implementation summary** (language/framework agnostic): Added the new skill in every supported agent location, documented its positioning and four-step workflow, and updated the repository overview so users can discover it easily.
- **Tests added**: No automated tests were added because the change is documentation-only and introduces no executable behavior.
- **Design pitfalls / shortcomings**: The skill intentionally stays tool-agnostic, so exact task tracking behavior still depends on the hosting agent environment.
- **Residual risks**: Agents may interpret durable plan storage differently if a repository lacks a clearly preferred planning document, but the skill mitigates this by emphasizing a single source of truth and reusable template.
- **Follow-ups**: Consider adding example mission documents or a dedicated shared planning file if future users want a stricter convention.

_No entries yet. The first agent to complete a gated task using this skill
should add its entry above this line._
