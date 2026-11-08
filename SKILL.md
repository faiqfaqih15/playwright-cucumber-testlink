---
name: playwright-cucumber-testlink
description: Use when building or maintaining TypeScript test automation with Playwright Test or Cucumber.js with Playwright, including TestLink or local ReportPortal work. Do not use for unrelated browser automation.
---

## Hard gates

- Select the runner from the requested test suite: use Playwright Test (@playwright/test) for Playwright-only tests; use Cucumber.js when the suite uses Cucumber, with Playwright supplying browser and API automation.
- Inspect package scripts, dependencies, configuration, and test files before selecting a mode. A dependency alone does not establish the active runner. In a mixed repository, preserve each suite's runner; do not convert suites or mix runner lifecycles unless explicitly requested. Ask only when the target suite remains ambiguous.
- TestLink access is read-only. Never call create, update, delete, result-write, or attachment-write methods.
- Inspect the target repository before proposing structure.
- Before every target-project file write, including scaffold and development workflows, show an exact file preview and wait for explicit approval.
- When TestLink is involved, include the TestLink-to-test mapping in that preview: Gherkin scenarios for Cucumber mode, native test cases for Playwright Test mode.
- Keep real tokens, passwords, customer data, and production endpoints out of files and output.

## Runner modes

### Playwright Test

- Use @playwright/test tests, fixtures, assertions, configuration, and reporting. Reuse the repository's native scripts and setup; do not add Cucumber, feature files, step definitions, World, or Cucumber hooks.
- Let Playwright Test own test isolation and fixture cleanup. Keep interactions in existing page/component/API helpers and assertions in the native tests.
- Read references/playwright-cucumber-patterns.md only for locator, synchronization, and failure-diagnosis guidance. Its Cucumber runner, step, World, and API lifecycle instructions apply only to Cucumber mode; use native page/request fixtures in Playwright Test mode.
- The existing architecture, Gherkin conversion, Cucumber troubleshooting sections, and Cucumber lifecycle/reporting templates are Cucumber-specific. Do not apply them to native tests. Check current official Playwright documentation when implementation details are needed.
- For TestLink import or synchronization, read references/testlink-ingestion.md for retrieval and normalization. Map each original suite to a native suite group and each case to a native test using the repository's conventions; preserve suite/case IDs as stable traceability metadata and plan IDs as execution context only. Preserve ordered actions, expected results, and preconditions. Validate unique mappings and preview added, changed, removed, expected-result, and precondition differences. Never remove tests automatically or overwrite existing TypeScript implementations during synchronization; report implementation changes for separate approval. Stop on validation failures and invalidate approval if source or target data changes.
- For local ReportPortal, use references/reportportal-local.md only for shared infrastructure approval, readiness, secrets, and best-effort reporting principles. Check the current official Playwright integration for the native reporter; do not use the Cucumber formatter, RPWorld, or Cucumber attachment templates. Preserve the native test result and local artifacts when reporting alone fails.

### Cucumber.js with Playwright

- Cucumber.js owns execution and lifecycle. Playwright supplies browser and API automation; do not introduce Playwright Test fixtures or hooks into Cucumber support code.
- Use the existing Cucumber references and templates through the workflow routes below.

## Workflow routing

- **Scaffold:** select the runner mode first. For Cucumber mode, read references/architecture.md and references/playwright-cucumber-patterns.md. For Playwright Test mode, follow the native mode above and existing repository conventions.
- **Import from TestLink:** read references/testlink-ingestion.md; in Cucumber mode also read references/gherkin-conversion.md. In Playwright Test mode, follow the native mapping above.
- **Synchronize:** follow the selected mode's TestLink mapping and preserve existing TypeScript implementations.
- **Develop:** in Cucumber mode, read references/architecture.md and only the needed sections of references/playwright-cucumber-patterns.md. In Playwright Test mode, follow the native mode above.
- **Review and debug:** read references/troubleshooting.md for applicable shared or Cucumber-specific failures, plus the smallest relevant domain reference. Diagnose native runner and fixture issues using the repository configuration and current official Playwright documentation.
- **Local ReportPortal:** follow the selected runner mode and references/reportportal-local.md within its applicable scope; Docker, WSL, software, and system changes require separate approval.
- **PostgreSQL:** invoke postgres-test-helper; do not duplicate its query or write-safety rules here.
