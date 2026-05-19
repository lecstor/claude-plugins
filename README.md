# lecstor-claude-plugins

Personal Claude Code plugin marketplace.

## Plugins

| Plugin | Description |
|---|---|
| [context-management](plugins/context-management) | Subagent set for context isolation (coder, docs-lookup, test-runner, review, browser) plus delegation rules. |

## Use this marketplace

Add to `~/.claude/settings.json`:

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

Restart Claude Code.
