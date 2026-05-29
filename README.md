# lecstor-claude-plugins

Personal Claude Code plugin marketplace.

## Plugins

| Plugin | Description |
|---|---|
| [basic](plugins/basic) | Personal dev toolkit — subagents for context isolation and skills for domain-specific coding guidance. |
| [context-management](plugins/context-management) | Subagent set for context isolation — coder, docs-lookup, test-runner, review, browser. |

## Use this marketplace

Install via the **Claude Code CLI** — the desktop app doesn't expose `/plugin` and adding marketplaces through its UI is awkward. Start `claude` in a terminal, then:

```
/plugin marketplace add lecstor/claude-plugins
/plugin install basic@lecstor-claude-plugins
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
    "basic@lecstor-claude-plugins": true
  }
}
```

Restart Claude Code after editing.
</details>
