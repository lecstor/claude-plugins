# basic

Personal dev toolkit for Claude Code — subagents for context isolation and skills for domain-specific coding guidance. Keeps the main agent's context window lean by delegating verbose work to focused subagents that return summaries instead of raw output.

## What's inside

| Subagent | Model | Purpose |
|---|---|---|
| **coder** | sonnet | Execute fully-specified, mechanical code edits. Designed for parallel dispatch. Escalates back when anything is ambiguous. |
| **docs-lookup** | haiku | Look up library/framework docs via Context7 MCP (if installed) or `WebFetch`. |
| **test-runner** | haiku | Run `pnpm test` / `pnpm e2e` (or equivalents). Read-only — returns structured pass/fail report, never edits code. |
| **review** | opus | Review the current diff for quality, bugs, performance, security. Returns severity-tagged findings. |
| **browser** | sonnet | Drive Chrome via the **chrome-devtools-mcp** connector for UI verification. Saves screenshots to disk as JPEG q40 — never inline. |

A `CLAUDE.md` ships with the plugin containing delegation rules that bias the primary agent toward using these subagents.

## Skills

| Skill | Description |
|-------|-------------|
| **course-authoring** | Create courses using the Course App platform's composable DSL |
| **e2e-testing** | Write Playwright end-to-end tests following semantic locator patterns |
| **npm-oidc-trusted-publishing** | Migrate npm packages from NPM_TOKEN to OIDC trusted publishing |
| **react-best-practices** | Idiomatic React patterns, hooks, and performance optimizations |
| **react-testable-code** | React components designed for easy testing with Playwright and Testing Library |
| **sentry-cli** | Interact with Sentry from the command line |
| **stylex** | Write StyleX styles correctly — longhand properties, no shorthands |
| **sync-cf-secrets** | Sync Cloudflare Workers secrets from a password manager |
| **test-runner** | Execute test suites and report results without modifying code |
| **testing-library** | Write React component tests using Testing Library |
| **work-phase** | Drive a project's task list unattended, dispatching subagents sequentially |

Skills are individually toggleable — enable only what's relevant to your project.

## Prerequisites

### Agents

- **docs-lookup**: works without anything special, but is more useful with the [Context7 MCP](https://github.com/upstash/context7) connected.
- **browser**: requires the [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) connector.

The other agents have no external dependencies beyond a Node.js project using a standard package manager.

### Skills

Some skills require external tools installed and available in your `$PATH`. Skills with unmet requirements are automatically hidden — no errors, they just won't appear.

| Skill | Requires | Install |
|-------|----------|---------|
| sentry-cli | `sentry` | `brew install getsentry/tools/sentry-cli` |
| sync-cf-secrets | `sync-cf-secrets` | `npm install -g sync-cf-secrets` |

## Installation

Use the **Claude Code CLI** — the desktop app doesn't currently expose `/plugin` and adding marketplaces through its UI is awkward. Start `claude` in a terminal, then:

```
/plugin marketplace add lecstor/claude-plugins
/plugin install basic@lecstor-claude-plugins
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
    "basic@lecstor-claude-plugins": true
  }
}
```

Restart Claude Code after editing.
</details>

## Customization

- **Personal CLAUDE.md rules** still live in your own `~/.claude/CLAUDE.md` and stack on top of the plugin's CLAUDE.md.
- **Project overrides**: drop a `.claude/agents/<name>.md` with the same name into a project to override an agent locally.

## Notes

- All agents use Claude Code's stable model aliases (`sonnet`, `haiku`) so they track latest versions automatically.
- Subagent invocations consume from your interactive Claude Code quota, not the Agent SDK credit bucket.
