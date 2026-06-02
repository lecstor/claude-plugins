# tools

Skills for driving external developer tools and services from Claude Code. Each skill teaches the agent how to use a particular CLI or service correctly, so you can manage it conversationally instead of memorising commands.

## Skills

| Skill | Description |
|-------|-------------|
| **sentry-cli** | Interact with Sentry from the command line — view issues, events, projects, organizations, and make API calls |
| **sync-cf-secrets** | Sync Cloudflare Workers secrets from a password manager (1Password, Bitwarden) |

Skills are individually toggleable — enable only what's relevant to your project.

## Prerequisites

These skills require external tools installed and available in your `$PATH`. Skills with unmet requirements are automatically hidden — no errors, they just won't appear.

| Skill | Requires | Install |
|-------|----------|---------|
| sentry-cli | `sentry` | `brew install getsentry/tools/sentry-cli` |
| sync-cf-secrets | `sync-cf-secrets` | `npm install -g sync-cf-secrets` |

## Installation

Use the **Claude Code CLI** — the desktop app doesn't currently expose `/plugin` and adding marketplaces through its UI is awkward. Start `claude` in a terminal, then:

```
/plugin marketplace add lecstor/claude-plugins
/plugin install tools@lecstor-claude-plugins
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
    "tools@lecstor-claude-plugins": true
  }
}
```

Restart Claude Code after editing.
</details>
