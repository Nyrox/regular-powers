---
name: using-superpowers
description: "Opt-in, subagent-driven software development workflow (design → subagent implementation → operator QA → merge). Use ONLY when the operator explicitly asks for it — e.g. 'use superpowers', 'run the workflow', 'brainstorm and build this properly', or /skill:superpowers. Never engage automatically for ordinary coding requests."
---

# superpowers

A structured development methodology, adapted from obra/superpowers. Unlike
upstream superpowers, **this workflow is never mandatory and never
auto-engages.** Use it only when the operator asks for it.

## When to Use

Engage this workflow ONLY when the operator requests it:

- "use superpowers", "run the workflow", "brainstorm this and build it properly"
- an explicit `/skill:superpowers` (or `/skill:brainstorming`, etc.) invocation
- a clear request to follow the design → subagent-implementation → QA process

If the operator just asks you to write or fix some code without invoking the
workflow, **do the normal thing.** Do not announce skills, do not force
brainstorming, do not write design documents. This is the core difference from
upstream superpowers: the ceremony is opt-in.

## The Flow

Once invoked, the workflow runs these phases in order. Each phase is its own
skill — read that skill's `SKILL.md` when you reach it.

1. **Design** — `../brainstorming/SKILL.md`. Refine the idea into a clear design.
   The target is **no open questions** — keep going until the design is
   unambiguous. Produces a design document ending in a numbered task list.
   The design says *what* to build, never *how* — no files, code, or steps.
2. **Implement** — `../subagent-driven-development/SKILL.md`. Dispatch a fresh
   implementer subagent per task, given only the design doc and a task number,
   with two-stage review. Implementers own all implementation decisions.
   **This is the only execution mode** — there is no inline/batch alternative.
3. **QA + Merge** — `../finishing-a-development-branch/SKILL.md`. Do NOT assume
   the task is done when implementation finishes. Enter an operator QA cycle;
   apply requested changes as fixup commits; merge (autosquashing fixups) only
   after the operator explicitly approves.

Supporting skills used within these phases: `test-driven-development`,
`using-git-worktrees`, `requesting-code-review`, `receiving-code-review`,
`systematic-debugging`, `verification-before-completion`.

## Document Storage (commit vs. scratch)

Design docs are stored one of two ways, decided per repo by whether a committed
`docs/superpowers/` directory exists. Resolve it once with the shared script
and reuse the result across phases:

```bash
../scripts/sp-workspace   # prints "commit <dir>" or "scratch <dir>"
```

- **Commit mode** (`docs/superpowers/` exists in the repo): write the design to
  `docs/superpowers/specs/` and commit it once the operator approves it.
- **Scratch mode** (no `docs/superpowers/`): write the design to the throwaway
  directory the script prints (under `$TMPDIR`). **Never commit it.** Tell the
  operator where it is and that it is session scratch — it is not expected to
  outlive the session.

The SDD progress ledger, implementer reports, and review packages always live
in git-ignored scratch (`.superpowers/sdd/`) regardless of mode — that is
separate from design storage.
