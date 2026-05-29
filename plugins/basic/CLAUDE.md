# Subagent delegation

Default to delegating. Reading files, running tests, looking up docs, and exploring code all bloat the main context with output the user never sees. Push that work to subagents so only summaries return.

Use a subagent whenever a task involves:
- Reading more than 2 files to answer a question → **Explore** (code) or **docs-lookup** (third-party libraries)
- Running tests or any command with verbose output → **test-runner**
- Surveying unfamiliar code, mapping call graphs, finding usages → **Explore**
- Browser/UI verification → **browser**
- Reviewing a diff before commit → **review**
- Executing a fully-specified, mechanical code edit (especially in parallel batches) → **coder**

Stay inline when:
- The answer truly is one file or one line away
- The change requires judgment, design decisions, or live context only you have
- It's a single small edit where delegation overhead outweighs the saving

## Skill usage

Load skills with the Skill tool before implementing work that matches their domain — and pass the skill name to the subagent so it loads the same guidance:
- `e2e-testing` — Playwright tests
- `react-best-practices`, `react-testable-code` — React components
- `testing-library` — unit tests
- `stylex` — StyleX styling

## Delegating to `coder`

Provide a fully-specified task: which files, what content, relevant conventions, and constraints. The coder will escalate back if anything is ambiguous — make the decision and re-dispatch. When multiple edits are independent, dispatch them to `coder` in parallel.

## Test + review pipeline

After implementing code: delegate test execution to **test-runner**, then delegate the diff to **review**. Apply warranted review feedback, re-run tests if changes were made, then summarise.
