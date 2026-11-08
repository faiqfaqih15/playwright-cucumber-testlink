# Local ReportPortal workflow

Use ReportPortal as an optional local reporting service. The dashboard is
`http://localhost:8080`; the Cucumber agent sends results to
`http://localhost:8080/api/v2`. Do not give the dashboard URL to the agent.

## Approval and readiness gate

Manual start and stop is the default. Before proposing any infrastructure
change, inspect the target machine and show the results:

1. Confirm supported 64-bit Windows and WSL 2 availability and status.
2. Confirm Docker Desktop is installed and running, and that the Docker Compose
   plugin is at least version 2.2.
3. Confirm Docker has at least 2 CPUs and 6 GB RAM available and the host has at
   least 20 GB free disk space. Increase these values if the target workload
   needs it.
4. Confirm TCP port 8080 is free. If it is occupied, stop and ask whether to
   free it or approve a different local port; do not silently change ports.

Get separate approval before installing or updating software, installing WSL,
enabling Windows features, changing system settings, downloading container
images, starting containers, or changing ports. This skill does none of those
actions itself. Docker Desktop auto-start is optional guidance only; do not
enable it without approval.

At target-project implementation time, download the current official
ReportPortal Compose file from the ReportPortal repository. Do not vendor a
Compose snapshot in this skill: it would become stale. Review the downloaded
file, its images, exposed ports, volumes, and secrets before requesting approval
to download images or start it.

Set a strong bootstrap administrator password before the first start and change
all documented default login credentials immediately. Server bootstrap secrets
belong in the local deployment environment; the Cucumber client's `RP_API_KEY`
is a user API key from ReportPortal. Keep those secret classes separate. Do not
commit, print, or log either one.

ReportPortal's Compose deployment includes its own internal PostgreSQL. It is
ReportPortal infrastructure and is distinct from any PostgreSQL instance used
by the application under test. Never point application-test database cleanup,
migrations, or fixtures at ReportPortal's database.

## Target-project lifecycle

After approval, keep the reviewed Compose file in a target-project-only ignored
directory such as `.reportportal/`, then map package scripts to the reviewed
file:

```json
{
  "scripts": {
    "reportportal:start": "docker compose -p reportportal -f .reportportal/docker-compose.yml up -d",
    "reportportal:status": "docker compose -p reportportal -f .reportportal/docker-compose.yml ps",
    "reportportal:logs": "docker compose -p reportportal -f .reportportal/docker-compose.yml logs --tail=200",
    "reportportal:stop": "docker compose -p reportportal -f .reportportal/docker-compose.yml down"
  }
}
```

Operate it manually with `npm run reportportal:start`,
`npm run reportportal:status`, `npm run reportportal:logs`, and
`npm run reportportal:stop`. The stop command intentionally omits `--volumes`,
so it does not delete ReportPortal data. Starting may download images; the image
download and container start require their own prior approval.

## Cucumber formatter and traceability

Copy `templates/reportportal-config.ts.template` into the target project and
load it only when `validateReportPortalConfig()` returns true. The environment
template defaults `RP_REPORTING_ENABLED=false` and `RP_STRICT_REPORTING=false`.
Set `RP_REPORTING_ENABLED=true` to opt into local reporting. Set
`RP_STRICT_REPORTING=true` only when reporting delivery itself must be a gate;
strict mode validates that `RP_API_KEY` exists before the formatter starts.
Never log the configuration object because it contains the client key.

Keep `scenarioBasedStatistics: true`. The Cucumber agent reports each executed
Scenario, including each Scenario Outline example, as the statistical test
item; its Cucumber steps are nested log items. Preserve the permanent TestLink suite
and case identity from `@TestLinkSuite_<id>` and `@TestLinkCase_<external-id>` as
feature/scenario metadata. If execution was selected by a TestLink plan ID, add
`{ key: 'TestLinkPlanId', value: '<selected-plan-id>' }` to that ReportPortal
launch's `attributes`. A plan is run-selection context, so keep its ID out of
permanent Gherkin tags.

The existing Cucumber hooks attach a failure screenshot and Playwright trace
with `this.attach`; the ReportPortal Cucumber agent forwards Cucumber
attachments. Always configure local Cucumber output and failure files under
`artifacts/` independently of the ReportPortal formatter.

## Offline fallback

Reporting is best-effort by default. The target-project launch wrapper should:

1. Validate the local configuration and ReportPortal readiness before adding
   the formatter. If it is disabled, lacks a key, or is unreachable, warn
   without printing endpoints containing credentials, headers, bootstrap
   secrets, or `RP_API_KEY`; then run Cucumber with local formatters only.
2. Preserve Cucumber's application-test exit code and the screenshot, trace,
   JSON, HTML, and other local files under `artifacts/`.
3. Treat a confirmed ReportPortal-only transport or upload failure as a warning.
   It must not change the application test result. Never mask a Cucumber failure
   or classify an unknown runner error as reporting-only.

If reporting delivery must fail the command, opt in explicitly with
`RP_STRICT_REPORTING=true`. Strict reporting may add a reporting failure after
the application-test result has been recorded, but it must not rewrite scenario
outcomes. A ReportPortal outage must not change the application test result in
the default mode.

## Official sources to re-check at implementation time

- ReportPortal installation guide: <https://reportportal.io/installation/>
- ReportPortal CucumberJS agent: <https://github.com/reportportal/agent-js-cucumber>
- ReportPortal attachment guide: <https://reportportal.io/docs/developers-guides/AttachmentsGuide/>
- ReportPortal launch attributes: <https://reportportal.io/docs/work-with-reports/LaunchesTestItemsAttributes/>
