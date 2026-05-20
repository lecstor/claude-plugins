# lecstor-claude-plugins

Personal Claude Code plugin marketplace.

## Plugins

| Plugin | Description |
|---|---|
| [context-management](plugins/context-management) | Subagent set for context isolation (coder, docs-lookup, test-runner, review, browser) plus delegation rules. |

## Use this marketplace

From inside Claude Code (CLI or desktop):

```
/plugin marketplace add lecstor/claude-plugins
/plugin install context-management@lecstor-claude-plugins
```

Or run `/plugin` to browse interactively.

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
    "context-management@lecstor-claude-plugins": true
  }
}
```

Restart Claude Code after editing.
</details>
