---
name: writing-plans
description: "Planning phase of the superpowers workflow. Use after a design is approved (or when the operator explicitly asks to write an implementation plan with the workflow). Breaks the design into bite-sized TDD tasks, then hands off to subagent-driven-development."
---

# Writing Plans

Write a comprehensive implementation plan assuming the engineer has zero context
for this codebase and questionable taste. Document everything: which files to
touch per task, the code, the tests, how to verify. Bite-sized tasks. DRY.
YAGNI. TDD. Frequent commits.

Assume a skilled developer who knows almost nothing about this toolset, problem
domain, or good test design.

**Announce at start:** "Using writing-plans to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via
`using-git-worktrees` at execution time.

## Where the Plan Goes

Resolve storage with the shared script (same result the design used):

```bash
../scripts/sp-workspace   # prints "commit <dir>" or "scratch <dir>"
```

- **Commit mode:** save the plan to `<dir>/plans/YYYY-MM-DD-<feature-name>.md`.
- **Scratch mode:** save the plan to `<dir>/plans/YYYY-MM-DD-<feature-name>.md`
  in the scratch directory. Never commit.

## Scope Check

If the design covers multiple independent subsystems, suggest breaking it into
separate plans — one per subsystem. Each plan should produce working, testable
software on its own.

## File Structure

Before defining tasks, map which files are created/modified and what each is
responsible for. This locks in decomposition.

- Each file has one clear responsibility with a well-defined interface.
- Prefer smaller, focused files. Files that change together live together.
- In existing codebases, follow established patterns; only split a file you're
  already modifying if it has grown unwieldy.

## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a fresh
reviewer's gate. Fold setup/config/scaffolding/docs into the task whose
deliverable needs them; split only where a reviewer could meaningfully reject
one task while approving its neighbor. Each task ends with an independently
testable deliverable.

## Bite-Sized Task Granularity

Each step is one action (2-5 minutes): write the failing test → run it and watch
it fail → implement minimal code → run tests and watch them pass → commit.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development
> to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Spec:** [path to the design doc this plan implements — executors read both]

## Global Constraints

[The design's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, exact values
copied verbatim. Every task implicitly includes this section.]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact names, parameter and return types]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan
failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — tasks may be read out of order)
- Steps describing what to do without showing how (code blocks required)
- References to types/functions/methods not defined in any task

## Self-Review

After writing the complete plan, check it against the design with fresh eyes:

1. **Spec coverage:** Can you point to a task for each design requirement? List gaps.
2. **Placeholder scan:** Search for the red flags above. Fix them.
3. **Type consistency:** Do types/signatures/names in later tasks match earlier ones?

Fix issues inline. If a design requirement has no task, add the task.

## Commit / Scratch Handoff

Once the plan is written and self-reviewed:

- **Commit mode:** stage the design doc AND the plan, and commit them together in
  a **single commit**, e.g.:
  ```bash
  git add docs/superpowers/specs/<...>-design.md docs/superpowers/plans/<...>.md
  git commit -m "docs: design and implementation plan for <feature>"
  ```
- **Scratch mode:** do nothing to git. Report the scratch paths to the operator
  and remind them these won't outlive the session.

## Execution Handoff

There is only one execution mode. Do NOT offer a choice.

> "Plan complete and saved to `<path>`. Proceeding with subagent-driven
> development."

**REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development — fresh
subagent per task with two-stage review.
