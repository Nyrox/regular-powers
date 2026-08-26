---
name: brainstorming
description: "Design phase of the superpowers workflow. Use when the operator asks to brainstorm/design a feature with the workflow (or via /skill:brainstorming). Turns an idea into a clear design with no open questions, then hands off to writing-plans."
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
design phase and produces a design document, then hands off to writing-plans:

1. **Explore project context** — read the relevant files, docs, and recent
   commits before asking anything.
2. **Ask clarifying questions** — one at a time. Prefer multiple-choice when it
   helps, open-ended when it does not.
3. **Propose approaches** — present the realistic options with trade-offs, lead
   with your recommendation and why. YAGNI ruthlessly.
4. **Present the design in sections** — get approval after each section.
5. **Write the design document** — to the resolved specs location.
6. **Spec self-review** — fix placeholders/contradictions/ambiguity inline.
7. **Operator reviews the written spec** — wait for approval.
8. **Hand off to writing-plans.**

## The Target: No Open Questions

Do not stop at some fixed number of questions. Keep asking until **there are no
open questions left** and the design is unambiguous. Before presenting the
design, verify:

- Every requirement has a single, clear interpretation.
- Purpose, constraints, and success criteria are all pinned down.
- There is nothing you would have to guess about during implementation.

If a question would change the design, it is still open — ask it. Only present
the design once you could hand it to a stranger and they would not need to ask
you anything. If new ambiguity surfaces while presenting, go back and resolve
it before moving on.

## The Process

**Understanding the idea:**
- Check the current project state first (files, docs, recent commits).
- If the request spans multiple independent subsystems, flag it and help the
  operator decompose into sub-projects; brainstorm the first one through this
  flow. Each sub-project gets its own design → plan → implementation cycle.
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
- Smaller, well-bounded units are easier to implement and review. A file that
  wants to grow large is usually doing too much.

**Working in existing codebases:**
- Explore the current structure and follow existing patterns.
- Include targeted improvements to code you're touching when it serves the goal;
  don't propose unrelated refactoring.

## After the Design

**Resolve where the design doc goes** (see the entry skill's storage rule):

```bash
../scripts/sp-workspace   # prints "commit <dir>" or "scratch <dir>"
```

- **Commit mode:** write the design to `<dir>/specs/YYYY-MM-DD-<topic>-design.md`.
  Do NOT commit it now — the design and the plan are committed together in a
  single commit at the end of writing-plans.
- **Scratch mode:** write the design to `<dir>/specs/YYYY-MM-DD-<topic>-design.md`
  in the scratch directory. Never commit it. Tell the operator the path and that
  it is session scratch.

**Spec self-review** (fix inline, no re-review needed):
1. Placeholder scan — any TBD/TODO/vague requirement? Fix it.
2. Internal consistency — any sections contradict each other?
3. Scope check — focused enough for one implementation plan?
4. Ambiguity check — any requirement open to two readings? Pin one down.

**Operator review gate:**
> "Design written to `<path>`. Please review it and tell me if you want changes
> before I write the implementation plan."

Wait for the operator's response. If they request changes, make them and re-run
the self-review. Only proceed once they approve.

**Transition:** the ONLY next step is the writing-plans skill. Do not invoke any
other implementation skill from here.
