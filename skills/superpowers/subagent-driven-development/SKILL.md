---
name: subagent-driven-development
description: "Implementation phase of the opt-in workflow — the only execution mode. Use when executing an approved design doc's task list. Dispatches a fresh implementer subagent per task (design doc + task number) with two-stage review, then hands off to the operator QA cycle in finishing-a-development-branch."
---

# Subagent-Driven Development

Execute the design's task list by dispatching a fresh implementer subagent per
task, a task review (spec compliance + code quality) after each, and a broad
whole-branch review at the end. This is the **only** execution mode — there is
no inline or batch alternative.

**Why subagents:** You delegate tasks to specialized agents with isolated
context. They never inherit your session's history — each gets the design doc,
its task number, and nothing else it doesn't need.

**You coordinate; implementers decide how.** A dispatch names a design doc and
a task number. How the task is built — which files, which functions, which
tests — is the implementer's call, made against the real codebase. Do not
design implementations in dispatch prompts, and do not pre-digest the design
into briefs.

**Core principle:** Fresh subagent per task + task review (spec + quality) +
broad final review = high quality, fast iteration.

**Narration:** between tool calls, narrate at most one short line.

**Continuous execution:** Do not pause to check in between tasks. Execute all
tasks without stopping. The only reasons to stop are the four named below, or
all tasks complete. (The operator QA cycle happens later, in finishing — not
between tasks.)

**Rulings, not stalls.** A running task list does not wait on a human.
Conflicts, ambiguities, design gaps, a cap you would have asked to exceed —
decide them. The design doc is the binding authority, and your judgment settles
what it doesn't answer. Record every decision in the ledger as
`Ruling: <what you decided> — <why> — <what it costs if wrong>`, and keep going.

Four things stop you, and only these: an irreversible or destructive operation;
a security-sensitive action; a side effect outside this worktree that norms say
you ask about first (a merge, a push to a shared branch, a publish); and a
design so broken that every path forward is a guess. For those, stop and ask.

## Setup

Ensure the work happens in an isolated workspace: use
superpowers:using-git-worktrees to create one or verify the existing one. Never
start implementation on a main/master branch without the operator's explicit
consent.

Conversation memory does not survive compaction. Track progress in a ledger
file, not only in todos.

- Each design owns a workspace: run this skill's `scripts/sdd-workspace
  DESIGN_FILE` — it prints the design's git-ignored directory
  (`<repo-root>/.superpowers/sdd/<design-basename>/`), home to every artifact
  for THIS design: ledger, reports, review packages. Another design's directory
  is never yours.
- Check for this design's ledger at `<workspace>/progress.md`. If its first line
  names your design file, tasks with a `Task <N>: complete` line are DONE — do
  not re-dispatch them; resume at the first task without one. A task whose last
  line is a fix round is mid-loop: resume at the next round. A ledger naming a
  different design is another design's — leave it and start your own.
- Create the ledger with its identity as the first line:
  `# SDD ledger — design: <design file path>`.
- The ledger is your recovery map: the commits it names exist in git even when
  your context no longer remembers creating them. After compaction, trust the
  ledger and `git log` over recollection.
- `git clean -fdx` will destroy the workspace (git-ignored scratch); recover
  from `git log` if that happens.

Read the design once, note its Global Constraints and its Tasks section, and
create a todo per task. A design with no numbered task list cannot be executed —
stop and send it back to brainstorming.

Before dispatching Task 1, scan the task list once against the design:

- tasks that contradict each other, the design, or the Global Constraints
- a task that depends on a later task
- anything a task requires that the review rubric treats as a defect

Write what you checked to the ledger. Rule on everything you find before
execution begins, record each ruling, then dispatch Task 1. If the scan is
clean, proceed without comment.

## Model Selection

Use the least powerful model that can handle each role.

- **Implementation:** implementers work from design-level tasks and make their
  own implementation decisions, so a mid-tier model is the floor. Use the most
  capable tier for tasks with heavy multi-unit integration or subtle
  correctness requirements.
- **Reviews:** scale to the diff's size, complexity, and risk; mid-tier floor.
  Scoped re-reviews of small fix diffs take a cheap-to-mid tier.
- **Final whole-branch review:** most capable.
- **Fix-loop escalation (rounds 4-5):** at least one tier above the implementer
  that got stuck.

**Always specify the model explicitly when dispatching a subagent.** An omitted
model inherits your session's model — often the most expensive.

## The Task Loop

**Batch small same-shape work.** When the design lists several tasks that are
each a small independent change of the same kind, dispatch them to a single
implementer as one batch (list every task number) and review the diff as one
unit. Reserve one-dispatch-per-task for work needing its own judgment, tests,
or review surface.

Everything you paste into a dispatch prompt — and everything a subagent prints
back — stays resident in your context. Hand artifacts over as files.

**Waiting on dispatched subagents:** never poll with short timeouts, and never
sit in one silent open-ended wait. While you have local work, keep working;
child results arrive on their own. When genuinely idle, wait in bounded
stretches and reconcile live children between stretches.

### 1. Dispatch the implementer

Record BASE (`git rev-parse HEAD`) before dispatching — the review package and
fix-round diffs need it.

- **Dispatch contents:** (1) the design doc path; (2) the task number(s);
  (3) any ledgered rulings that change or clarify this task, verbatim;
  (4) the report-file path and report contract. Nothing else. The implementer
  reads the whole design for context and finds how earlier tasks landed by
  reading the code and `git log` — do not summarize prior tasks for it.
- **Report file:** `<workspace>/task-<N>-report.md` (a batch joins the numbers:
  `task-3+4+5-report.md`). The implementer writes the full report there and
  returns only status, commits, a one-line test summary, and concerns.
- The dispatch carries the no-subagents contract (in the implementer template):
  the implementer never dispatches subagents — not helpers, never a reviewer.
- If an earlier task parked a finding in this task's area, carry a pointer to
  that ledger entry.
- Record the implementer's agent identity — fix-loop rounds 1-3 resume it.
- Never dispatch multiple implementation subagents in parallel (conflicts).

Template: [implementer-prompt.md](implementer-prompt.md)

### 2. Handle the report

**DONE:** Generate the review package (`scripts/review-package DESIGN_FILE BASE
HEAD` — it prints the file path it wrote; BASE is the commit you recorded before
dispatching, never `HEAD~1`), then dispatch the task reviewer with the printed
path.

**DONE_WITH_CONCERNS:** Read the concerns. If about correctness or scope,
address before review. If observations, note and proceed.

**NEEDS_CONTEXT:** If the design answers it, point at the section. If it is a
design-level gap, rule on it, ledger the ruling, and re-dispatch with the
ruling. If it is an implementation question, tell the implementer it owns that
decision.

**BLOCKED:** Assess the blocker — context problem (add context, re-dispatch),
needs more reasoning (more capable model), too large (split the task — ledger
the split as a ruling), design wrong (rule, ledger, re-dispatch with the
ruling). **Never** ignore an escalation or force the same model to retry
without changes.

If the implementer asks questions, answer clearly and completely; don't rush it.

### 3. Review the task

Per-task reviews are task-scoped gates. Never skip them, and never accept a
report missing either verdict — spec compliance AND task quality are both
required. Implementer self-review never replaces the task review.

- Hand the reviewer its diff as a file: run `scripts/review-package DESIGN_FILE
  BASE HEAD` and pass the printed path. Use the BASE you recorded, never
  `HEAD~1`. Never dispatch a task reviewer without a diff file.
- **Reviewer inputs:** the design doc path, the task number(s), any rulings you
  gave the implementer for this task, the report file, and the review package.
- Do not add open-ended directives without a concrete reason. Do not ask a
  reviewer to re-run tests the implementer already ran. Do not pre-judge
  findings ("do not flag", "at most Minor", "the design chose") — that is
  pre-judging; let the reviewer raise it and adjudicate in the loop.

The reviewer may report "⚠️ Cannot verify from diff" items. These don't block
the review, but resolve each yourself before marking the task complete. A
confirmed gap enters the fix loop.

Template: [task-reviewer-prompt.md](task-reviewer-prompt.md)

### 4. The fix loop

Triggers on spec ❌, any Critical/Important finding, or a ⚠️ item you confirmed.

Two routes leave it immediately:
- Minor findings: record in the ledger (`Task <N>: minor (deferred): <one-liner>`)
  and point the final review at that list. Minors never enter the loop.
- A design-mandated finding (or one conflicting with the design text): rule on
  it, ledger the ruling, then act.

Everything else enters. A fix round is one fix dispatch plus one scoped
re-review. Five rounds max per task:

- **Rounds 1-3 — resume the original implementer.** Send open findings verbatim.
  If your harness can't message a live subagent, dispatch a fresh implementer
  with the design path, task number, report-file path, and findings.
- **Rounds 4-5 — fresh implementer on a more capable model**, with design path,
  task number, report file, open findings, and: "A prior implementer attempted
  this task N times; you own it now. Read the report file for what was tried."
- **Every round:** the implementer fixes, re-runs covering tests, appends its
  fix report to the same report file, returns the short contract. Confirm the
  fix report has covering tests + command + output before re-dispatching review.
- **The re-review is scoped:** run `scripts/review-package DESIGN_FILE FIX_BASE
  HEAD` (FIX_BASE = the head the previous review saw) and dispatch
  [re-review-prompt.md](re-review-prompt.md). New Critical/Important breakage in
  the fix diff joins the open findings. Out-of-scope observations go to the
  ledger as deferred minors.
- **After each round,** append:
  `Task <N>: fix round <R>/5 (<X> addressed, <Y> open — <one-liners>; commits <a7>..<b7>)`

Never fix findings yourself in the controller session.

**The breaker.** When round 5's re-review still leaves findings open, stop
dispatching and adjudicate each:
- Reviewer wrong or contestable → park: `Task <N>: parked — <finding> — Ruling: <why the code stands>`.
- Real but nothing downstream builds on it → park with a ruling saying it's real and deferred.
- Real and load-bearing → rule on the smallest change that unblocks dependent
  work, ledger `Task <N>: Ruling: <finding> — <decision and why>`, carry it into
  the next dispatch. Stop only when every path forward is a guess.

Adjudicate only at the cap. Every adjudication is a ledger entry — silent
discards are forbidden.

### 5. Complete the task

When the review is clean — or every finding is parked with a ruling at the cap —
append the completion line:
- `Task <N>: complete (commits <base7>..<head7>, review clean)`
- `Task <N>: complete (commits <base7>..<head7>, <K> parked)` after a tripped breaker

Then mark the todo complete and move on. Never advance while a review has open
Critical/Important issues that are neither fixed nor parked-with-ruling at the cap.

## Final Review

Run `scripts/review-package DESIGN_FILE MERGE_BASE HEAD` (MERGE_BASE = where the
branch started, e.g. `git merge-base main HEAD`) and include the printed path in
the final review dispatch. Dispatch on the most capable model, using
superpowers:requesting-code-review's
[code-reviewer.md](../requesting-code-review/code-reviewer.md) with the design
doc as the requirements. Point it at the ledger's deferred-minor and parked
lines to triage what must be fixed before QA.

If the final review returns findings, dispatch ONE fix subagent with the
complete findings list (not one fixer per finding), then run exactly one scoped
re-review of the fix wave. Adjudicate residuals as in the task loop's breaker.
There is no second fix wave — residual load-bearing findings surface to the
operator in the QA cycle.

## Finish → Operator QA

Before deleting anything, collect every ledger line containing `Ruling:` into
your report to the operator under "Rulings I made", in order, each with what it
costs if wrong. That list is the only place the decisions you took on the
operator's behalf reach them.

When the final whole-branch review is clean and its fixes are merged into the
branch, delete this design's SDD workspace (`rm -rf <workspace>`) — git history
is the record now. Sibling directories belong to other designs; leave them
alone.

**Do NOT assume the task is complete, and do NOT merge.** Hand off to
superpowers:finishing-a-development-branch, which runs the operator QA cycle and
merges only on explicit approval.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Close enough on spec compliance" | Reviewer found gaps = not done. Fix, or hit the cap and adjudicate. |
| "I'll fix it myself, dispatching is overhead" | Controller fixes pollute context and skip review. Resume the implementer. |
| "I'll spell out the files and code in the dispatch to save the implementer time" | Implementation is the implementer's call. Design doc + task number + rulings. |
| "One more round will converge" | Past the cap, rounds don't converge — the failure is structural. |
| "The reviewer will just find something new" | Scoped re-reviews verify fixes; new findings on untouched code go to the ledger. |
| "This finding is obviously wrong, I'll drop it" | You adjudicate only at the cap, and every ruling is a ledger entry. |
| "The fix was small, skip the re-review" | Unreviewed fixes are how regressions land. |
| "Ledger bookkeeping is overhead" | The ledger is what survives compaction. |
| "Implementation is done, so the task is done" | Implementation done ≠ task done. The operator QA cycle in finishing decides that. |
