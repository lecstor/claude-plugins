# automation

Unattended task execution for Claude Code. Drive a project's task list end-to-end without supervision — a coordinator picks the next unblocked task, dispatches a worker subagent to complete it, self-verifies the result against the task's "Done when" criteria, then loops.

Built for overnight runs and batch task execution where you want to come back to a triage-able morning report rather than babysit each step.

## What's inside

| Component | Type | Purpose |
|---|---|---|
| **work-phase** | skill | The coordinator. Reads `tasks.md`, picks the next dependency-unblocked task, dispatches a `task-worker`, verifies the commit/archive/validate, then loops. Parks isolated failures non-destructively and keeps going; trips a circuit breaker on systemic failure. Supports `--strict`, `--require-approval`, and a max-tasks count. |
| **task-worker** | agent (sonnet) | Executes a single task end-to-end — read the detail file, implement, drive validate to green, commit specific files with the project's trailer, update docs, archive the task. Returns a structured report; never invents follow-up work. |

The coordinator never writes code itself — it only dispatches and verifies. The worker never picks up a second task — it completes one and returns. This separation keeps the loop auditable: every task is one or more real commits, and failures are parked (recoverable via branch/stash) rather than silently repaired.

## How it works

1. You maintain a `tasks.md` (the backlog) and per-task detail files in `tasks/` with a **Plan** and **Done when** checklist. The project's `CLAUDE.md` defines conventions (stack, package manager, commit trailer, validate command).
2. Run `/work-phase` (optionally `/work-phase 5` to cap at 5 tasks).
3. The coordinator runs a baseline validate gate, then loops: pick → dispatch `task-worker` → self-verify → record or park → repeat.
4. You get an end-of-run report listing everything completed (with SHA ranges) and everything parked (with the failed check and a recovery ref).

See the `work-phase` skill for the full failure policy, circuit-breaker rules, and recovery commands.

## Prerequisites

`task-worker` delegates verbose sub-work (exploration, test runs, doc lookups, diff review, mechanical edits) to the context-isolation subagents — `Explore`, `docs-lookup`, `test-runner`, `review`, `coder`. Those ship in the **context-management** plugin. Install one of those alongside `automation` so the worker has its full toolkit; without them it falls back to doing that work inline.

The workflow assumes a Node.js project with a standard package manager and the `tasks.md` / `tasks/` convention described above.

## Installation

Use the **Claude Code CLI** — the desktop app doesn't currently expose `/plugin` and adding marketplaces through its UI is awkward. Start `claude` in a terminal, then:

```
/plugin marketplace add lecstor/claude-plugins
/plugin install automation@lecstor-claude-plugins
/plugin install context-management@lecstor-claude-plugins
```

Or browse interactively with `/plugin`. Once installed, the plugin works in both the CLI and the desktop app — only the install step needs the CLI.

<details>
<summary>Manual install via <code>settings.json</code></summary>

```json
{
  "extraKnownMarketplaces": {
    "lecstor-claude-plugins": {
      "source": {
        "source": "github",
        "repo": "lecstor/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "automation@lecstor-claude-plugins": true,
    "context-management@lecstor-claude-plugins": true
  }
}
```

Restart Claude Code after editing.
</details>

## Notes

- `task-worker` uses Claude Code's stable `sonnet` alias so it tracks the latest version automatically.
- The loop is **local-only**: it never pushes, deploys, or merges. Remote state is always the human's call.
- Subagent invocations consume from your interactive Claude Code quota, not the Agent SDK credit bucket.
