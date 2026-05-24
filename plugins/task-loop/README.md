# task-loop

Subagent + coordinator skill for working through a project's task list unattended — pick the next dependency-unblocked task, dispatch a worker, verify the result, then loop. Designed for overnight runs where every completed task lives as a local commit on the branch, so the morning review is a `git log` away.

## What's inside

| Component | Type | Purpose |
|---|---|---|
| **task-worker** | subagent (sonnet) | Execute a single task end-to-end: read the detail file, implement, validate, commit, update docs, archive. Returns a structured report. Never picks up the next task. |
| **work-phase** | skill *(planned)* | Coordinator that picks the next task, dispatches `task-worker`, self-verifies (clean tree, new commit, archive moved, validate green, done-when satisfied), then loops. Stops on any check failure rather than compounding errors. |

## Assumptions about the host project

The worker expects the project to follow a small set of conventions — most are documented in the project's `CLAUDE.md`:

- A `tasks.md` index at the repo root, with one detail file per task in a `tasks/` directory and a `tasks/_done.md` archive.
- Each task detail file has a **Plan** section and a **Done when** checklist.
- `docs/` holds the system-as-intended; tasks update those files when decisions change.
- A standard validate command (`pnpm typecheck && pnpm lint && pnpm test` or equivalent).
- Commits use an imperative subject and the project's `Co-Authored-By` trailer.

If your project doesn't match this shape, you'll want to either adjust the conventions or fork the worker's "Close out" section.

## Safety model

The whole point of this plugin is to run unattended without compounding errors into untracked drift. Three load-bearing constraints:

1. **The worker cannot push, deploy, merge, or schedule.** Every change is a local commit. Worst case is a messy branch, never a messy production.
2. **The coordinator self-verifies before looping.** Clean working tree + new commit + archive moved + validate green + done-when satisfied. Any failure → stop, leave the branch as-is for inspection.
3. **The worker never edits `tasks.md` to add new work.** Discoveries go in its report under "Follow-ups noted"; the coordinator decides whether they become tasks. This keeps the task list as the single source of truth.

## Installation

Use the **Claude Code CLI**:

```
/plugin marketplace add lecstor/claude-plugins
/plugin install task-loop@lecstor-claude-plugins
```

## Status

The `task-worker` subagent is ready. The `work-phase` coordinator skill is sketched but not yet shipped — the coordinator loop currently has to live in the dispatching session's prompt. Track progress in the repo.
