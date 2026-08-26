---
name: systematic-debugging
description: "Root-cause-first debugging within the superpowers workflow. Use when the operator explicitly asks to debug a problem systematically, or when invoked by another superpowers skill (e.g. during the QA cycle) to investigate a bug before proposing fixes. Not auto-applied to every error."
---

# Systematic Debugging

## Overview

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for any technical issue you've been asked to debug: test failures, bugs,
unexpected behavior, performance problems, build failures, integration issues.

**Use this ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work
- You don't fully understand the issue

**Don't skip when:** the issue seems simple, you're in a hurry, or someone wants
it fixed NOW. Systematic is faster than thrashing.

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully** — don't skip errors/warnings; read stack
   traces completely; note line numbers, file paths, error codes.
2. **Reproduce Consistently** — can you trigger it reliably? Exact steps? Every
   time? If not reproducible → gather more data, don't guess.
3. **Check Recent Changes** — git diff, recent commits, new dependencies, config
   changes, environmental differences.
4. **Gather Evidence in Multi-Component Systems** — for each component boundary,
   log what enters and exits, verify env/config propagation, check state at each
   layer. Run once to gather evidence showing WHERE it breaks, THEN analyze,
   THEN investigate that specific component.
5. **Trace Data Flow** — when the error is deep in the call stack, trace backward
   to where the bad value originates. See [root-cause-tracing.md](root-cause-tracing.md).

### Phase 2: Pattern Analysis

1. **Find Working Examples** — locate similar working code in the same codebase.
2. **Compare Against References** — if implementing a pattern, read the reference
   COMPLETELY; understand it fully before applying.
3. **Identify Differences** — list every difference between working and broken,
   however small. Don't assume "that can't matter".
4. **Understand Dependencies** — what components, settings, config, environment,
   and assumptions does it need?

### Phase 3: Hypothesis and Testing

1. **Form Single Hypothesis** — "I think X is the root cause because Y." Be specific.
2. **Test Minimally** — smallest possible change, one variable at a time.
3. **Verify Before Continuing** — worked? → Phase 4. Didn't? → new hypothesis,
   don't stack fixes.
4. **When You Don't Know** — say "I don't understand X". Don't pretend. Research more.

### Phase 4: Implementation

1. **Create Failing Test Case** — simplest reproduction; automated if possible.
   MUST have before fixing. Use superpowers:test-driven-development.
2. **Implement Single Fix** — address the root cause; one change; no "while I'm
   here" improvements or bundled refactoring.
3. **Verify Fix** — test passes now? No other tests broken? Issue actually
   resolved? Use superpowers:verification-before-completion before claiming success.
4. **If Fix Doesn't Work** — STOP. Count fixes tried. If < 3: return to Phase 1
   with new information. **If ≥ 3: STOP and question the architecture (step 5).**
5. **If 3+ Fixes Failed: Question Architecture** — the pattern (each fix reveals
   new coupling elsewhere; fixes need "massive refactoring"; each creates new
   symptoms) means a wrong architecture, not a failed hypothesis. Discuss with
   the operator before attempting more fixes.

## Red Flags - STOP and Follow Process

If you catch yourself thinking:
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- "Skip the test, I'll manually verify"
- "It's probably X, let me fix that"
- Proposing solutions before tracing data flow
- **"One more fix attempt" (when already tried 2+)**
- **Each fix reveals a new problem in a different place**

**ALL of these mean: STOP. Return to Phase 1.**

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| "I'll write test after confirming fix works" | Untested fixes don't stick. Test first proves it. |
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
| "Reference too long, I'll adapt the pattern" | Partial understanding guarantees bugs. Read it completely. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Question the pattern. |

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

## Supporting Techniques

Available in this directory:

- [root-cause-tracing.md](root-cause-tracing.md) — trace bugs backward through the call stack to the original trigger
- [defense-in-depth.md](defense-in-depth.md) — add validation at multiple layers after finding the root cause
- [condition-based-waiting.md](condition-based-waiting.md) — replace arbitrary timeouts with condition polling
