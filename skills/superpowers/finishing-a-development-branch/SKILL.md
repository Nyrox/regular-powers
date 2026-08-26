---
name: finishing-a-development-branch
description: "QA + integration phase of the superpowers workflow. Use when implementation is complete and tests pass. Runs an operator QA cycle (manual review, fixup commits for requested changes) and merges only on explicit operator approval, autosquashing fixups."
---

# Finishing a Development Branch

## Overview

**Core principle:** Implementation done ≠ task done. Verify tests → **operator
QA cycle** → integrate only on explicit approval → autosquash fixups → clean up.

**Announce at start:** "Using finishing-a-development-branch — entering the QA
cycle before anything gets merged."

Do NOT present a merge menu or merge anything until the operator has manually
reviewed the work.

## Step 1: Verify Tests

Run the project's full test suite (`npm test` / `cargo test` / `pytest` /
`go test ./...`). If multiple test suites exist, make sure to run all - including e2e tests.

- **If tests fail:** report the failures and stop. Fix (as fixup commits per the
  QA cycle below) before continuing.
- **If tests pass:** continue to Step 2.

## Step 2: Detect Environment

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
# Capture now — later steps change directory before cleanup needs this.
WORKTREE_PATH=$(git rev-parse --show-toplevel)
BRANCH=$(git branch --show-current)
```

| State | Integration menu | Cleanup |
|-------|------------------|---------|
| `GIT_DIR == GIT_COMMON` (normal repo) | Standard 3 options | No worktree to clean up |
| `GIT_DIR != GIT_COMMON`, named branch | Standard 3 options | Provenance-based (Step 6) |
| `GIT_DIR != GIT_COMMON`, detached HEAD | Reduced 2 options (no merge) | Externally managed — leave in place |

## Step 3: Hand Off for Operator QA

Present the completed work for manual review. This is a gate — do not proceed to
integration until the operator responds.

Give the operator what they need to review:

- The branch name and a one-line summary of what was built.
- The commit list: `git log --oneline <base>..HEAD`.
- How to run / exercise it (commands, entry points, the test command).
- The **"Rulings I made"** list carried over from subagent-driven-development —
  the decisions you took on their behalf, each with what it costs if wrong.

Then ask, explicitly:

> "Implementation and automated review are complete and tests pass. Please
> manually review the changes. Tell me about any bugs or changes you want, or
> say 'merge' when you're satisfied. I won't merge until you approve."

Wait for the operator.

## Step 4: The QA Cycle

Loop until the operator explicitly approves.

**If the operator reports bugs or requests changes:**

1. Treat the feedback with technical rigor — use superpowers:receiving-code-review
   (verify against the codebase, ask if unclear, push back with reasoning when a
   request is wrong). Never performative agreement.
2. Fix each item. Use superpowers:test-driven-development for behavior changes and
   superpowers:systematic-debugging for reported bugs.
3. **Commit fixes as fixup commits when possible.** Find the original commit the
   fix belongs to and attach the change to it:
   ```bash
   git log --oneline <base>..HEAD          # find the target commit
   git add <files>
   git commit --fixup=<target-sha>          # e.g. --fixup=a1b2c3d
   ```
   A fixup commit keeps the eventual history clean: it will be folded into its
   target when you autosquash at merge time.
   - If a change maps cleanly to one existing commit → `git commit --fixup=<sha>`.
   - If it maps to several, split it and make one fixup per target.
   - If it belongs to no existing commit (a genuinely new file or a cross-cutting
     addition) → make a normal commit and note it; autosquash will leave it as-is.
4. **Verify before claiming done** — use superpowers:verification-before-completion.
   Re-run the covering tests and the full suite; show the evidence.
5. Report what you changed (and which commits) and return to Step 3's ask — the
   operator re-reviews. Loop.

**If the operator says merge / approves:** go to Step 5.

Do not skip the loop by declaring the work done yourself. Only the operator's
explicit approval ends the QA cycle.

## Step 5: Integrate (only after explicit approval)

### 5a. Confirm the base branch

The base is whatever this work forked from — usually named in the plan or the
branch's upstream. If not already known, ask:
"This branch split from <best guess> — correct?" Merging into the wrong base is
expensive to undo.

### 5b. Autosquash fixup commits

Fold every `fixup!` commit into its target, non-interactively (no editor):

```bash
BASE=<base-branch-or-merge-base>
GIT_SEQUENCE_EDITOR=true GIT_EDITOR=true git rebase -i --autosquash "$BASE"
```

`--autosquash` reorders each `fixup!` under its target and marks it to be
squashed; `GIT_SEQUENCE_EDITOR=true` accepts that todo list unchanged, so the
rebase runs unattended. If the rebase stops on a conflict, resolve it, `git add`
the resolution, and `git rebase --continue`.

After the rebase, verify the history and re-run the suite on the squashed result:

```bash
git log --oneline "$BASE"..HEAD    # confirm no stray fixup! commits remain
<test command>                     # must still be green
```

If tests fail on the squashed result, stop and investigate — nothing has been
pushed, so it is recoverable.

### 5c. Present the integration menu

**Normal repo and named-branch worktree — exactly these 3 options:**

```
QA approved and fixups squashed. How should I integrate?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)

Which option?
```

**Detached HEAD — exactly these 2 options:**

```
QA approved and fixups squashed. You're on a detached HEAD (externally managed).

1. Push as new branch and create a Pull Request
2. Keep as-is (I'll handle it later)

Which option?
```

### Option 1: Merge Locally

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git checkout <base-branch>
git pull
git merge <feature-branch>
<test command>          # verify tests on the merged result
```

If tests fail on the merged result: stop, leave branch/worktree in place,
investigate. Once green: clean up the worktree (Step 6), then
`git branch -d <feature-branch>`.

### Option 2: Push and Create PR

```bash
git push -u origin <feature-branch>
# From a detached HEAD: git push origin HEAD:refs/heads/<new-branch>
```

Create the PR against <base-branch> with the forge's tooling, following the
repo's PR template, and report the URL. Keep the worktree for PR iteration.

### Option 3: Keep As-Is

Report: "Keeping branch <name>. Worktree preserved at <path>."

## Step 6: Cleanup Workspace

**Runs for Option 1 and confirmed discards only.** Options 2 and 3 preserve the
worktree. Run from the main repo root (callers already `cd` there), using the
Step 2 values.

- **If `GIT_DIR == GIT_COMMON`:** normal repo, nothing to clean up.
- **If `WORKTREE_PATH` is under `.worktrees/` or `worktrees/`:** superpowers
  created it — we own cleanup:
  ```bash
  git worktree remove "$WORKTREE_PATH"
  git worktree prune
  ```
- **If removal is refused** (`contains modified or untracked files`): the
  worktree holds files that exist nowhere else. Never `--force` on your own.
  Show `git -C "$WORKTREE_PATH" status --porcelain -uall` and ask whether to
  commit / move / delete them, then remove.
- **Otherwise:** the host owns this workspace — leave it in place.

Also remember: in scratch mode the design doc and plan live in `$TMPDIR` and are
never committed — leave them; they are session scratch.

## If the operator asks to discard the work

Only in response to an explicit request to throw the work away. Confirm first
(list branch, commits, worktree path) and require the operator to type `discard`.
On that exact confirmation, clean up the worktree (Step 6) and
`git branch -D <feature-branch>`.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Tests and review passed, so it's done — I'll merge" | The operator QA cycle decides done. Present the work and wait for approval. |
| "The operator seems happy, that's approval" | Only an explicit 'merge'/approval ends the QA cycle. |
| "I'll just commit the fix normally" | Prefer a fixup targeting the original commit; it keeps history clean after autosquash. |
| "Autosquash needs an interactive editor I can't open" | `GIT_SEQUENCE_EDITOR=true git rebase -i --autosquash <base>` runs unattended. |
| "Tests passed earlier this session" | Run the suite on the squashed/merged tree you're about to integrate. |
| "The base branch is obviously main" | Confirm the fork point. Wrong base is expensive to undo. |
| "Removal refused — --force finishes the cleanup" | Those files exist only there. Show the operator and ask. |
