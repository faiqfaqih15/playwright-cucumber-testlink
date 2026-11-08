# Target-project architecture

Inspect the target repository before proposing folders or files. Reuse its
module format, TypeScript loader, configuration, helpers, page objects, and
domain language when they already satisfy the need. Keep one focused
responsibility per file and do not scaffold speculative layers.

Use this dependency direction:

```text
Feature -> thin step definition -> page/component/API/database layer
World -> scenario-owned state and adapters
Hooks -> worker browser, scenario context/page, trace, attachments, cleanup
```

## Ownership boundaries

- Cucumber.js owns execution and lifecycle. Playwright supplies browser and API
  automation.
- Launch one `Browser` in `BeforeAll` for each Cucumber worker and close it in
  `AfterAll`. Cucumber runs those hooks once per worker process, not once for an
  entire parallel run.
- Create one isolated `BrowserContext` and `Page` in `Before` for each scenario.
  Start tracing there; attach failure evidence and close scenario resources in
  `After`, even when evidence capture fails.
- Register exactly one World constructor. Extend `RPWorld` so ReportPortal and
  Cucumber attachments remain available while project state stays typed.
- Store scenario state and adapters on the World. Never use module-level mutable
  scenario state; module scope is reserved for worker lifecycle resources such
  as the browser.
- Keep feature files business-readable, step definitions thin, and interactions
  in page, component, API, or database layers. Prefer composition and existing
  repository conventions over new framework abstractions.

The templates are inert examples. Adapt imports and paths only after repository
inspection, and keep the lifecycle and ownership boundaries above intact.
