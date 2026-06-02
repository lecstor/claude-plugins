# lecstor-claude-plugins

Personal Claude Code plugin marketplace.

## Plugins

| Plugin | Description |
|---|---|
| [development](plugins/development) | Personal collection of Claude Code skills — domain-specific coding guidance loaded on demand. |
| [context-management](plugins/context-management) | Subagent set for context isolation — coder, docs-lookup, test-runner, review, browser. |
| [automation](plugins/automation) | Unattended task execution — the work-phase coordinator drives a project's task list, dispatching task-worker subagents to complete, verify, and commit each task in a loop. |
| [tools](plugins/tools) | Skills for driving external developer tools and services — Sentry CLI, Cloudflare Workers secrets. |

## Use this marketplace

Install via the **Claude Code CLI** — the desktop app doesn't expose `/plugin` and adding marketplaces through its UI is awkward. Start `claude` in a terminal, then:

```
/plugin marketplace add lecstor/claude-plugins
/plugin install development@lecstor-claude-plugins
```

Or run `/plugin` to browse interactively. Plugins installed via the CLI are picked up by the desktop app automatically — they share the same `~/.claude/settings.json`.

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
    "development@lecstor-claude-plugins": true
  }
}
```

Restart Claude Code after editing.
</details>
