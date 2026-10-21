---
name: brainstorming
description: "Design phase of the superpowers workflow. Use when the operator asks to brainstorm/design a feature with the workflow (or via /skill:brainstorming). Turns an idea into a clear design with no open questions and a numbered task list, then hands off to subagent-driven-development."
---

# Brainstorming Ideas Into Designs

Turn an idea into a fully formed design through collaborative dialogue. This is
the design phase of the superpowers workflow, so it runs only when the
operator asked for the workflow.

**Announce at start:** "Using brainstorming to design [thing]."

<HARD-GATE>
Do NOT write code, scaffold, or take any implementation action until you have
presented the design and the operator has approved it.
</HARD-GATE>

## Single Linear Flow

There are no spike/bounded/architectural branches. Every invocation runs one
design phase and produces a design document, then hands off to
subagent-driven-development:

1. **Explore project context** — read the relevant files, docs, and recent
   commits before asking anything.
2. **Ask clarifying questions** — one at a time. Prefer multiple-choice when it
   helps, open-ended when it does not.
3. **Propose approaches** — present the realistic options with trade-offs, lead
   with your recommendation and why. YAGNI ruthlessly.
4. **Present the design in sections** — get approval after each section.
5. **Write the design document** — to the resolved specs location, ending in a
   numbered task list.
6. **Spec self-review** — fix placeholders/contradictions/ambiguity inline.
7. **Operator reviews the written spec** — wait for approval.
8. **Hand off to subagent-driven-development.**

## The Target: No Open Questions

Do not stop at some fixed number of questions. Keep asking until **there are no
open questions left** and the design is unambiguous. Before presenting the
design, verify:

- Every requirement has a single, clear interpretation.
- Purpose, constraints, and success criteria are all pinned down.
- There is nothing an implementer would have to guess about *what* to build.

If a question would change the design, it is still open — ask it. Only present
the design once you could hand it to a stranger and they would not need to ask
you anything. If new ambiguity surfaces while presenting, go back and resolve
it before moving on.

## Design, Not Implementation

The design says **what** gets built and how the pieces relate. It does not say
**how** each piece is coded — that is the implementer's job, done against the
real codebase at execution time. Leave out exact file paths, function bodies,
test code, and step-by-step instructions. Do pin down anything that crosses a
unit boundary or that the operator cares about: externally visible behavior,
interfaces between units, data formats, naming the user sees, constraints.

## The Process

**Understanding the idea:**
- Check the current project state first (files, docs, recent commits).
- If the request spans multiple independent subsystems, flag it and help the
  operator decompose into sub-projects; brainstorm the first one through this
  flow. Each sub-project gets its own design → implementation cycle.
- Ask questions one at a time, focused on purpose, constraints, success criteria.

**Exploring approaches:**
- Propose the realistic approaches with trade-offs; lead with your recommendation.
- Remove unnecessary features from every option (YAGNI).

**Presenting the design:**
- Scale each section to its complexity (a few sentences to a few paragraphs).
- Ask after each section whether it looks right.
- Cover: architecture, components, data flow, error handling, testing.

**Design for isolation and clarity:**
- Break the system into small units, each with one clear purpose and a
  well-defined interface, understandable and testable independently.
- For each unit: what does it do, how is it used, what does it depend on?
- Smaller, well-bounded units are easier to implement and review. A unit that
  wants to grow large is usually doing too much.

**Working in existing codebases:**
- Explore the current structure and follow existing patterns.
- Include targeted improvements to code you're touching when it serves the goal;
  don't propose unrelated refactoring.

## The Task List

The design ends with a numbered task list. Subagent-driven-development
dispatches implementers with nothing but this design doc and a task number, so
each task must stand on its own against the rest of the document.

A task is the smallest unit that carries its own test cycle and is worth a fresh
reviewer's gate. Fold setup/config/scaffolding/docs into the task whose
deliverable needs them; split only where a reviewer could meaningfully reject
one task while approving its neighbor. Each task ends with an independently
testable deliverable. Order tasks so each depends only on earlier ones.

Describe tasks in design terms — the deliverable and how to tell it's done — not
as implementation steps:

```markdown
### Task N: [Name]

**Delivers:** [the behavior or capability this adds, in terms of the design's units]
**Done when:** [testable acceptance criteria, one per line]
**Depends on:** [earlier task numbers, or "none"]
```

## Design Document Shape

```markdown
# [Feature Name] Design

**Goal:** [One sentence describing what this builds]

## Context
[Why, and the relevant current state of the project]

## Design
[Architecture, units and their interfaces, data flow, error handling, testing
approach — the sections approved with the operator]

## Global Constraints
[Project-wide requirements — version floors, dependency limits, naming and copy
rules, platform requirements — one line each, exact values. Every task
implicitly includes this section.]

## Tasks
[The numbered task list]
```

## After the Design

**Resolve where the design doc goes** (see the entry skill's storage rule):

```bash
../scripts/sp-workspace   # prints "commit <dir>" or "scratch <dir>"
```

- **Commit mode:** write the design to `<dir>/specs/YYYY-MM-DD-<topic>-design.md`.
  Commit it once the operator approves it.
- **Scratch mode:** write the design to `<dir>/specs/YYYY-MM-DD-<topic>-design.md`
  in the scratch directory. Never commit it. Tell the operator the path and that
  it is session scratch.

**Spec self-review** (fix inline, no re-review needed):
1. Placeholder scan — any TBD/TODO/vague requirement? Fix it.
2. Internal consistency — any sections contradict each other?
3. Scope check — focused enough for one implementation cycle?
4. Ambiguity check — any requirement open to two readings? Pin one down.
5. Task coverage — can you point to a task for each design requirement? Does
   every task's "Done when" follow from the design? Add or fix tasks for gaps.
6. Implementation creep — any task prescribing files, code, or steps rather
   than a deliverable? Lift it back to design level.

**Operator review gate:**
> "Design written to `<path>`. Please review it — including the task list — and
> tell me if you want changes before implementation starts."

Wait for the operator's response. If they request changes, make them and re-run
the self-review. Only proceed once they approve.

**Commit (commit mode only):**
```bash
git add docs/superpowers/specs/<...>-design.md
git commit -m "docs: design for <feature>"
```

**Transition:** the ONLY next step is the subagent-driven-development skill,
given the design doc path. Do not invoke any other implementation skill from
here.

> "Design approved and saved to `<path>`. Proceeding with subagent-driven
> development."
