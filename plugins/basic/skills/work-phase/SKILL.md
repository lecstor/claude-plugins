---
name: work-phase
description: Drive the project's task list unattended — pick the next dependency-unblocked task, dispatch a task-worker subagent to complete it, self-verify the result, then loop. Use when the user wants to work through tasks.md sequentially without supervision (e.g. overnight runs). By default continues past isolated task failures (parks the failed work non-destructively, skips its dependents, keeps going) and trips a circuit breaker only on systemic failure; pass --strict for stop-on-first-failure.
---

# work-phase

You are the **coordinator**. You do not write code yourself. You dispatch `task-worker` subagents and verify their work.

The project's `CLAUDE.md` and `tasks.md` are the source of truth for the workflow. Read both before the first iteration so your dispatched prompts encode the project's actual conventions, not a generic version.

## Arguments

- `$1` (optional): max tasks to run. Default: until the active phase is exhausted, every remaining task is blocked, or the circuit breaker trips.
- `--strict`: revert to stop-on-first-failure — halt the whole loop the moment any verification check or worker report fails, and leave the branch exactly as the worker left it. Use when you'd rather inspect one clean failure than let the run continue.
- `--max-consecutive-failures N`: circuit-breaker threshold (default `2`). Ignored under `--strict`.
- `--require-approval`: pause after each task for user confirmation instead of self-verifying. Useful when babysitting the first run.

## Default failure policy: keep-going

Unless `--strict` is set, an isolated task failure does **not** stop the loop. You **park** the failed task's work (non-destructively, fully recoverable), leave its `tasks.md` entry in place, skip anything that depended on it, and continue with the next unblocked task. A circuit breaker (below) stops the run when failures look *systemic* rather than task-specific. The morning report lists everything completed and everything skipped, with the recovery ref for each parked failure.

## Before the loop — baseline gate

Run validate on the clean tree **before dispatching any task**: `pnpm typecheck && pnpm lint && pnpm test` (or the project's equivalent — check `CLAUDE.md`). If it fails, **stop immediately** and report — the branch is broken independent of any task, so every task would fail. (Under `--strict` this is the same gate.)

## Per-iteration loop

### 1. Pick the next task

- Read `tasks.md`. Find the next dependency-unblocked task in the active phase.
- **Treat any task in the skipped set (parked this run) as an unmet dependency**: do not pick a task whose detail file or `tasks.md` entry names a dependency that was parked. If the only remaining tasks are blocked by parked ones, the phase is effectively exhausted — report and stop.
- If no task is available (phase complete, or everything left is blocked), report it and stop.
- Read the task's detail file in `tasks/`.
- Record the current `HEAD` SHA as `PRE_TASK_SHA` — you'll cite it to verify the worker committed, and use it as the known-good base to reset to if the task fails.

### 2. Draft the dispatch prompt

The worker has no memory of this conversation. The prompt must be self-contained. Include:

- Task ID, title, and absolute path to the detail file
- A pointer to the project's `CLAUDE.md` (for conventions: stack, package manager, commit trailer)
- The list of `docs/` files the detail file references
- The full **Done when** checklist copied from the detail file
- Explicit closing actions: validate, commit specific files with the project's trailer, update referenced docs, archive the detail file to `tasks/_done.md`, remove the entry from `tasks.md`
- The reporting contract from the `task-worker` agent (structured report — no narrative)

Dispatch via the Agent tool with `subagent_type: "task-worker"`. Wait for it to return.

### 3. Self-verify

Skip this step if invoked with `--require-approval` — surface the worker's report to the user and wait. Otherwise run all of the following. A check fails if it is not satisfied; the worker also "fails" if its report contains any `✗`, "deferred", or "blocked" item.

- `git status --porcelain` is empty (no uncommitted or untracked changes)
- `git log PRE_TASK_SHA..HEAD --oneline` shows at least one new commit
- Each new commit includes the project's `Co-Authored-By` trailer
- The task's detail file no longer exists in `tasks/` and now appears in `tasks/_done.md`
- The task entry is removed (or struck) from `tasks.md`
- Validate passes: `pnpm typecheck && pnpm lint && pnpm test` (or the project's equivalent)
- Read the diff (`git diff PRE_TASK_SHA..HEAD`). For each **Done when** bullet, confirm the diff demonstrably satisfies it. If any bullet is ambiguous, treat it as a failure.
- If the detail file named specific docs to update, confirm those files appear in the diff.

**On all checks passing → step 4 (record success).**
**On any failure → under `--strict`, stop and report (leave the branch untouched). Otherwise → step 5 (park and continue).**

### 4. Record success and continue

Reset the consecutive-failure counter to 0. One concise line to the user:

```
✓ <task-id> — <sha-range> — <N> done-when items verified
```

Then loop back to step 1.

### 5. Park the failure and continue (keep-going only)

Return the working branch to `PRE_TASK_SHA` (the known-good base), preserving everything the worker did so it can be revisited:

- **Worker committed (new commits since `PRE_TASK_SHA`):** preserve them on a branch first, then move the working branch back.
  - `git branch work-phase/<task-id> HEAD` (commits now safe on that branch)
  - `git reset --hard PRE_TASK_SHA` (working branch returns to known-good)
- **Then, regardless, park any remaining uncommitted/untracked work:** if `git status --porcelain` is non-empty, `git stash push -u -m "work-phase/<task-id>"`.
- **Confirm clean base:** `git status --porcelain` is empty **and** `HEAD` == `PRE_TASK_SHA`. If you cannot achieve this (stash error, reset conflict, detached state), **trip the circuit breaker** — stop and report; do not continue on an uncertain base.

Then:

- Leave the task in `tasks.md` (do **not** archive it; it isn't done).
- Add it to the **skipped set** with: task id, which check failed (or the worker's `✗`/deferred line), and the recovery ref (`branch work-phase/<task-id>` and/or `stash work-phase/<task-id>`).
- Increment the consecutive-failure counter.
- One concise line to the user:

```
⚠ <task-id> — parked (<failed-check>) — recover: <branch/stash ref>
```

- **Check the circuit breaker** (below). If it trips, stop. Otherwise loop back to step 1.

## Circuit breaker (keep-going)

Stop the loop — even in keep-going mode — when failure looks systemic rather than isolated:

- **Baseline validate failed** before the loop started (handled above).
- **`--max-consecutive-failures` reached** (default 2 failures in a row). A success resets the counter, so isolated duds scattered through a productive run don't trip it; a broken environment trips it within N attempts.
- **A park/cleanup could not produce a clean known-good base** (step 5).

On a circuit-breaker stop, report the trip reason plus the full completed/skipped summary, and leave the branch at the last known-good `HEAD`.

## Stop conditions

Stop the loop and report under any of these:

- `--strict`: any self-verification check fails, or the worker reports failure/deferred/unsatisfied done-when (leave the branch untouched)
- Keep-going: the circuit breaker trips (see above)
- The max-tasks count is reached
- The active phase has no more unblocked tasks (counting parked tasks as blocking their dependents)

## End-of-run report

Always end with a summary the human can triage in the morning:

```
work-phase complete — <reason for stopping>

Completed (N):
  ✓ <task-id> — <sha-range>
  ...
Skipped / parked (M):
  ⚠ <task-id> — <failed-check> — recover: <branch/stash ref>
  ...
```

Recovering a parked task later: `git stash list` / `git branch --list 'work-phase/*'` show the parked work; `git stash apply stash^{/work-phase/<task-id>}` or `git cherry-pick`/`git merge work-phase/<task-id>` brings it back for another attempt.

## Hard rules

- Do **not** fix the worker's mistakes. A failed task is parked for human revisit (keep-going) or left untouched (`--strict`) — never silently repaired and re-committed.
- Do **not** skip verification checks to "keep the loop going." Keep-going continues *past* a recorded failure; it never *ignores* one.
- The **only** reset permitted is the step-5 park-and-reset back to `PRE_TASK_SHA`, and **only after** the worker's commits are preserved on a `work-phase/<task-id>` branch. Never reset without first branching; never reset to anything other than the recorded known-good base; never force-push; never amend or rewrite an existing commit's history.
- Do **not** push, deploy, or merge. Local state only — the human handles remote.
- Do **not** add new entries to `tasks.md` based on the worker's "Follow-ups noted" list without explicit user direction — surface them in the end-of-run report instead. The task list is the human's backlog, not the loop's scratchpad.

## When to use `--strict`

- First run on a new project (verify the workflow assumptions match)
- A phase whose tasks are tightly sequential / heavily interdependent (little to gain from continuing, more to untangle)
- After a worker or skill change, to confirm behaviour
- Anytime you'd rather see one clean failure than a morning of parked branches

(For step-by-step babysitting, prefer `--require-approval`, which pauses after every task regardless of outcome.)
