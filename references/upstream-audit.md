# Upstream Adaptation Audit

This skill selectively adapts the pinned sources listed in
[`THIRD_PARTY_NOTICES.md`](../../../THIRD_PARTY_NOTICES.md). It keeps
CucumberJS as the runner and avoids native Playwright Test runner conventions.

| Source | Decision | Reason | Destination |
| --- | --- | --- | --- |
| JMR World/hooks guidance | Rewrite | Replace browser per scenario with browser per worker and context per scenario. | Skill World and hook guidance |
| JMR Gherkin guidance | Keep with attribution | Compatible with Cucumber runner. | Feature and step-definition guidance |
| JMR reporting guidance | Rewrite | Replace generic HTML focus with ReportPortal CucumberJS. | Reporting guidance |
| TestDino locators/assertions | Adapt | Runner-independent Playwright guidance. | Locator and assertion guidance |
| TestDino API/network/debugging | Adapt | Replace request fixture with World-owned APIRequestContext. | API, network, and debugging guidance |
| TestDino fixtures/config/test organization | Reject | Assumes native Playwright Test runner, including `test.extend()`, `.spec.ts`, and Playwright Test hooks. | Not carried forward |
| TestDino CommonJS assumptions | Reject | Existing repository module format wins; new templates use the selected current Cucumber TypeScript loader rather than a CommonJS-only mandate. | Not carried forward |

The TestDino references `fixtures-and-hooks.md`, `configuration.md`, and
`test-organization.md` were consulted only to identify and reject the native
runner assumptions above. No upstream source is adopted wholesale.

## Consulted source inventory

The following file-level inventory makes the audit complete against the
provenance recorded in `THIRD_PARTY_NOTICES.md`:

### e2e-agent-skills

| Consulted path | Audit decision |
| --- | --- |
| `skills/playwright-cucumber-expert/SKILL.md` | Selectively adapted; see the guidance rows above |
| `skills/playwright-cucumber-expert/cucumber-setup.md` | Selectively adapted; see the Gherkin and setup guidance above |
| `skills/playwright-cucumber-expert/project-structure.md` | Selectively adapted; see the architecture guidance above |
| `skills/playwright-cucumber-expert/feature-files.md` | Keep with attribution; see the Gherkin guidance above |
| `skills/playwright-cucumber-expert/step-definitions.md` | Selectively adapted; see the step-definition guidance above |
| `skills/playwright-cucumber-expert/hooks-and-world.md` | Rewrite; see the World/hooks guidance above |
| `skills/playwright-cucumber-expert/tags-and-profiles.md` | Selectively adapted; see the tag guidance above |
| `skills/playwright-cucumber-expert/reporting.md` | Rewrite; see the reporting guidance above |
| `skills/playwright-cucumber-expert/anti-patterns.md` | Selectively adapted; incompatible runner patterns rejected |
| `skills/playwright-cucumber-expert/assets/` | Templates selectively adapted; no asset copied wholesale |
| `skills/playwright-cucumber-expert/scripts/scaffold-bdd.mjs` | Selectively adapted; no script copied wholesale |

### playwright-skill

| Consulted path | Audit decision |
| --- | --- |
| `core/SKILL.md` | Selectively adapted; runner boundary preserved |
| `locators.md` | Adapt |
| `locator-strategy.md` | Adapt |
| `assertions-and-waiting.md` | Adapt |
| `api-testing.md` | Adapt; request fixture replaced with World-owned APIRequestContext |
| `network-mocking.md` | Adapt |
| `when-to-mock.md` | Adapt |
| `test-data-management.md` | Adapt |
| `debugging.md` | Adapt |
| `trace-analysis.md` | Adapt |
| `flaky-tests.md` | Adapt |
| `common-pitfalls.md` | Adapt |
| `fixtures-and-hooks.md` | Reject native Playwright Test assumptions |
| `configuration.md` | Reject native Playwright Test assumptions |
| `test-organization.md` | Reject native Playwright Test assumptions |

The `assets/` entry is an inventory of the consulted template directory; its
individual files are covered by the notice and were not adopted wholesale.
