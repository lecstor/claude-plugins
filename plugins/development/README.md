# development

Personal collection of Claude Code skills — domain-specific coding guidance loaded on demand. Each skill teaches the agent the right patterns for a particular stack or task, so you get idiomatic output without spelling out conventions every time.

For context-isolation subagents (`coder`, `docs-lookup`, `test-runner`, `review`, `browser`), see the **context-management** plugin; for unattended task-list execution, see **automation**; for skills that drive external CLIs/services (Sentry, Cloudflare), see **tools**.

## Skills

| Skill | Description |
|-------|-------------|
| **e2e-testing** | Write Playwright end-to-end tests following semantic locator patterns |
| **npm-oidc-trusted-publishing** | Migrate npm packages from NPM_TOKEN to OIDC trusted publishing |
| **react-best-practices** | Idiomatic React patterns, hooks, and performance optimizations |
| **react-testable-code** | React components designed for easy testing with Playwright and Testing Library |
| **stylex** | Write StyleX styles correctly — longhand properties, no shorthands |
| **test-runner** | Execute test suites and report results without modifying code |
| **testing-library** | Write React component tests using Testing Library |

Skills are individually toggleable — enable only what's relevant to your project.

## Installation

Use the **Claude Code CLI** — the desktop app doesn't currently expose `/plugin` and adding marketplaces through its UI is awkward. Start `claude` in a terminal, then:

```
/plugin marketplace add lecstor/claude-plugins
/plugin install development@lecstor-claude-plugins
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
    "development@lecstor-claude-plugins": true
  }
}
```

Restart Claude Code after editing.
</details>
