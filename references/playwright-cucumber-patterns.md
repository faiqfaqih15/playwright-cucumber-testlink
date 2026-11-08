# Playwright patterns under Cucumber.js

Apply these runner-independent patterns only after inspecting the target
repository. Cucumber.js remains the runner; do not introduce a second test
runner or its fixtures and hooks.

## Locators and synchronization

Prefer user-facing locators in this order: `getByRole`, `getByLabel`, visible
text, then the repository's configured test IDs. Use CSS or XPath only when no
stable user-facing contract exists, and keep the reason local to the locator.

Rely on Playwright actionability and auto-waiting. Use web-first assertions for
observable state such as visibility, text, URL, count, or enabled state. Wait
for a specific condition or response when a business action has an asynchronous
precondition; never add arbitrary sleeps.

## Layer responsibilities

Page and component methods perform interactions and expose meaningful state.
Business-level `Then` steps own assertions. A reusable component may enforce an
invariant when every caller requires the same guarantee, but page methods must
not hide scenario-specific expectations.

Keep step definitions as translation glue. They obtain the page, API adapter,
or domain layer from `AutomationWorld`, pass Gherkin values through, and make
the business assertion readable.

## API automation

Create an `APIRequestContext` for the scenario and store it on the World. Do not
depend on a test-runner request fixture. Dispose the context in the scenario
cleanup hook, and keep credentials in environment variables rather than source,
logs, traces, or artifact names.

## Failure diagnosis

Start with the failed scenario's trace. Rank evidence before editing:

1. Inspect the failed action and its before/after DOM snapshots.
2. Check failed network requests and browser-console errors.
3. Compare a passing and failing trace at their first divergence.
4. Check isolation leaks, ambiguous locators, animation, and missing observable
   preconditions.
5. Change the narrowest responsible layer, then burn in the focused scenario.

Do not use unconditional retries to make a failure disappear. A retry may help
expose intermittency, but the accepted fix must address the evidence-backed root
cause. Never replace diagnosis with forced actions, weakened assertions, or
arbitrary sleeps.
