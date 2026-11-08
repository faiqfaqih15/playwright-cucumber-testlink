# Evidence-first troubleshooting

Use the same order for every failure: symptom -> evidence to collect -> ranked
likely causes -> smallest safe check -> corrective action -> verification. Do
not edit code, configuration, infrastructure, or timeout values until the
collected evidence supports a cause. Keep credentials, endpoints containing
credentials, customer data, and full TestLink payloads out of output.

## TestLink authentication failure

- **Symptom:** A read-only TestLink request reports an authentication or
  authorization failure and retrieval stops before preview or project writes.
- **Evidence to collect:** Record the sanitized error category, requested
  allowlisted method, selection type, and whether each required environment
  variable is present. Do not record values, headers, tokens, endpoints, or
  response payloads. The client diagnostic contains only the trusted selection
  kind and allowlisted method; it does not include IDs, parameters, endpoint,
  credentials, payloads, or the raw transport cause.
- **Ranked likely causes:** (1) missing or expired token, (2) account lacks
  read access to the selected project or case, (3) endpoint or project context
  is wrong, (4) TestLink or its XML-RPC API is unavailable.
- **Smallest safe check:** Validate required configuration locally without
  printing it. When live access is separately authorized, repeat one
  allowlisted read for the same selection; do not try a write method to test
  permissions.
- **Corrective action:** Correct only the evidence-backed credential source,
  least-privilege TestLink role, endpoint configuration, or service issue.
  Retrieve again and regenerate the preview because failed or stale retrieval
  data must not be reused.
- **Verification:** The fresh read returns a complete normalized selection,
  validation passes, the workflow stops at preview pending approval, and
  neither TestLink nor target-project files are changed.

## undefined Cucumber step

- **Symptom:** Cucumber reports an undefined Cucumber step and suggests a step
  definition snippet.
- **Evidence to collect:** Save the exact undefined expression, scenario name,
  loaded configuration, step-definition globs, TypeScript loader settings, and
  the dry-run output. Search existing step expressions for equivalent wording
  before considering a new definition.
- **Ranked likely causes:** (1) an existing step file is outside the configured
  glob, (2) the TypeScript loader did not load it, (3) wording or parameter
  syntax differs from an equivalent expression, (4) the behavior genuinely
  lacks a definition.
- **Smallest safe check:** Run a focused Cucumber dry run for the scenario using
  the repository's existing configuration. If the file is absent from loaded
  support code, inspect the effective glob and loader; if it is loaded, compare
  the expression literally with the feature step.
- **Corrective action:** Fix the narrowest evidenced issue: configuration or
  loader ownership first, reuse or narrowly adjust an equivalent expression
  second, and add a thin definition only when no equivalent behavior exists.
- **Verification:** The focused dry run reports zero undefined steps, then the
  focused scenario executes with its existing business assertion intact.

## ambiguous Cucumber step

- **Symptom:** Cucumber reports an ambiguous Cucumber step with two or more
  matching definitions.
- **Evidence to collect:** Capture the step text and every matching expression
  with its source location. Compare their parameter domains and callers; do not
  delete or rename a definition from the error message alone.
- **Ranked likely causes:** (1) duplicate equivalent expressions, (2) an
  overly broad regular expression or Cucumber expression, (3) overlapping
  parameter types, (4) duplicate support-code loading.
- **Smallest safe check:** Run a focused dry run and confirm whether the same
  source file is loaded twice or distinct expressions overlap for the reported
  text.
- **Corrective action:** Remove duplicate loading, reuse the established
  expression, or narrow only the evidenced overlapping expression while
  preserving its intended callers.
- **Verification:** The focused dry run resolves the step exactly once and all
  scenarios using either affected expression still pass.

## Playwright timeout

- **Symptom:** A Playwright action or web-first assertion reaches its timeout
  during a Cucumber scenario.
- **Evidence to collect:** Inspect locator resolution and actionability state,
  current and expected navigation state, failed or pending network requests,
  browser-console errors, and visible overlays or animations. Open the failed
  scenario's `trace.zip` and compare its first divergence with a passing trace
  when one exists.
- **Ranked likely causes:** (1) locator does not identify the intended element
  uniquely or in the expected state, (2) navigation or an observable
  precondition did not complete, (3) a network failure blocked the UI, (4) an
  overlay or animation prevented actionability, (5) isolation leaked between
  scenarios, (6) the operation has an evidenced legitimate duration longer
  than the current limit.
- **Smallest safe check:** Re-run only the failing scenario with tracing. Check
  locator state, navigation, network, overlays, and trace evidence in that
  order before considering any timeout increase.
- **Corrective action:** Fix the narrowest evidenced locator, observable wait,
  application/network condition, overlay handling, or isolation boundary. Only
  increase a timeout when the trace proves the operation is healthy and
  legitimately slower; never add an arbitrary sleep or forced action.
- **Verification:** The focused scenario passes repeatedly with the expected
  assertion, its trace shows the intended actionability transition, and no
  unconditional retry or arbitrary wait hides the failure.

## ReportPortal unavailable

- **Symptom:** Local ReportPortal is disabled, unreachable, or rejects an
  upload while Cucumber is running.
- **Evidence to collect:** Preserve the application-test exit code and local
  artifacts first. Record sanitized readiness and upload error categories,
  whether reporting and strict mode were enabled, and whether the required key
  is present; never print the key or secret-bearing configuration.
- **Ranked likely causes:** (1) reporting is intentionally disabled, (2) local
  ReportPortal is not ready, (3) the client key is absent or rejected, (4)
  endpoint or project configuration is wrong, (5) the formatter transport
  failed after the scenario result was recorded.
- **Smallest safe check:** Confirm the local Cucumber result and files under
  `artifacts/`, then validate configuration without logging it. Inspect service
  status only when that local infrastructure check is separately authorized;
  do not start containers or change system settings as diagnosis.
- **Corrective action:** In default mode, warn and continue with local
  formatters without changing the scenario result. Correct an evidenced local
  configuration or service issue only with the required approval. Treat
  delivery as command-failing only when strict mode is explicitly enabled.
- **Verification:** Local artifacts remain available and the application-test
  result is unchanged by a ReportPortal-only failure. If strict mode was
  explicitly enabled, verify it adds a reporting failure without rewriting
  scenario outcomes.
