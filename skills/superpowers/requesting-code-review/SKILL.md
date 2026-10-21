---
name: requesting-code-review
description: "Dispatch a code-reviewer subagent within the superpowers workflow. Use when invoked by subagent-driven-development (per-task and final reviews) or when the operator explicitly asks for a code review."
---

# Requesting Code Review

Dispatch a code reviewer subagent to catch issues before they cascade. The
reviewer gets precisely crafted context for evaluation — never your session's
history.

**Core principle:** Review early, review often.

## When to Request Review

**Mandatory (within the workflow):**
- After each task in subagent-driven development
- After completing a major feature
- Before the operator QA cycle (final whole-branch review)

**Optional but valuable:**
- When stuck (fresh perspective)
- Before refactoring (baseline check)
- After fixing a complex bug

## How to Request

**1. Get git SHAs:**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. Dispatch code reviewer subagent:**

Dispatch a subagent, filling the template at [code-reviewer.md](code-reviewer.md).

**Placeholders:**
- `{DESCRIPTION}` - Brief summary of what you built
- `{REQUIREMENTS}` - What it should do (design doc path, task, or requirements)
- `{BASE_SHA}` - Starting commit
- `{HEAD_SHA}` - Ending commit

**3. Act on feedback:**
- Fix Critical issues immediately
- Fix Important issues before proceeding
- Note Minor issues for later
- Push back if reviewer is wrong (with reasoning)

Use superpowers:receiving-code-review skill when implementing review feedback.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'll just review the diff myself instead of dispatching a reviewer" | You're the coordinator — reviewing inline burns the context you need to keep driving. Dispatch a reviewer: the diff and evaluation live in its context, only findings come back. |
| "The reviewer needs my whole session history" | Hand it precisely crafted context, never your session's history. |

## Red Flags

**Never:**
- Skip review because "it's simple"
- Ignore Critical issues
- Proceed with unfixed Important issues
- Argue with valid technical feedback

**If reviewer wrong:**
- Push back with technical reasoning
- Show code/tests that prove it works
- Request clarification

See template at: [code-reviewer.md](code-reviewer.md)
