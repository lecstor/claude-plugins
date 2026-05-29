---
name: test-runner
description: Execute test suites and report results. Runs unit tests (yarn test) and E2E tests (yarn e2e) without modifying code. Use when asked to run tests, check if tests pass, or get a test report.
---

Run the project's test suites and report results. This is a **read-only, execution-only** skill — do not write, modify, or fix code.

## What to run

| Request | Command | Fallback |
|---|---|---|
| Unit tests (default) | `yarn test` | `yarn test:ci` if tests hang |
| E2E tests (staging) | `yarn e2e` | — |
| E2E tests (local) | `yarn e2e:local` | — |
| All tests | Both unit and E2E | — |

Run whichever commands are requested. If no specific command is requested, run `yarn test`.

## E2E local server setup

E2E tests running locally require a **production build**, not the dev server. HMR causes test flakiness with Playwright.

1. **Start the local server:** `yarn dev-prod` (builds then runs `vite preview` on port 8688)
2. **Set env:** ensure `E2E_TESTING=true` is set in `.dev.vars`
3. **Run tests:** `yarn e2e:local`

Do NOT use `yarn dev` for E2E — the Vite HMR websocket interferes with Playwright's page lifecycle and causes flaky failures.

## Reporting format

Always return results in this structure:

**Commands Run:** list each command executed

**Overall Verdict:** PASS or FAIL

**Unit Tests:**
- Total / Passed / Failed / Skipped counts

**E2E Tests** (if run):
- Total / Passed / Failed / Skipped counts

**Failing Tests** (if any):
- Test name / describe block
- Error message
- Relevant error output (stack trace, assertion diff)

**Raw Output:** include full command output as evidence

## Rules

- Do NOT fix failing tests — report them only
- Do NOT edit or write any files
- Do NOT diagnose failures beyond reporting what failed and the error output
- If a command hangs, try the CI variant (`yarn test:ci`)
- Every failure must be surfaced with full output evidence
