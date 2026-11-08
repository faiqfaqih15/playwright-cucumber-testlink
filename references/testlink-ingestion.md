# Read-only TestLink ingestion

Treat TestLink as an untrusted, read-only input. Copy the model and client
templates into the target repository only after inspecting its conventions.
Inject a `call(method, params)` transport implemented with the repository's
compatible XML-RPC library; do not add a transport dependency to this skill.

## Selection and retrieval

Accept exactly one `TestLinkSelection`:

- `case` retrieves every requested external case ID individually.
- `suite` retrieves all cases for one suite ID.
- `plan` retrieves all cases for one plan ID. The plan ID is execution metadata,
  never permanent feature or scenario identity.

The client transport may invoke only these methods:

```text
tl.getTestCase
tl.getTestCasesForTestSuite
tl.getTestCasesForTestPlan
```

Reject every method outside this closed allowlist before calling the injected
transport. Do not add create, update, delete, result, or attachment operations.

For TestLink 1.9.20, request suite cases with `details: 'full'` and `deep: false`;
the default `simple` response omits required steps, while recursive suite
retrieval can cross the one-suite/one-feature boundary. Full suite records use
`parent_id`, `external_id`, and `tsuite_name`. The client recognizes those
source keys and accepts the selected project ID as explicit constructor context
because the suite response omits project identity. Treat that context as
required enrichment when the response has no project field. Never replace a
returned suite ID with the requested suite ID: validate the returned identity
so an out-of-scope child-suite case still fails closed.

Request plan cases with `getstepsinfo: true`; without it, TestLink may omit the
manual steps required by the normalized model. Plan results still retain their
original suite IDs and are grouped only after every returned case validates.

## Normalize before conversion

Raw XML-RPC responses must never flow directly into Gherkin generation. Map
each selected record into `ImportedTestCase` and each step into
`ImportedTestStep` from `templates/testlink-model.ts.template`. Preserve the
source `projectId`, `suiteId`, `suiteName`, internal and external case IDs,
title, optional preconditions, ordered actions, and expected results.

Validate the complete normalized selection before previewing or writing:

- Reject missing, blank, or ambiguous project, suite, internal-case, external-
  case, and title values.
- Reject an empty external-ID request and empty suite or plan results.
- Reject duplicate requested or returned external case IDs.
- Reject absent steps, duplicate or invalid step order, blank actions, and blank
  expected results.
- Treat malformed or unexpectedly shaped responses as incomplete data; do not
  guess a field mapping.
- Stop on authentication failures and transport failures. Report sanitized
  context only; never echo tokens, passwords, endpoints, or customer payloads.

Any validation, authentication, or transport failure ends the workflow without
project-file writes. Return the failure for correction and require a fresh
retrieval before a later preview.

## Preserve suite ownership

Group every normalized result by its original `suiteId`; verify each group has
one unambiguous `suiteName`. This applies especially to plan selections: plan
results remain grouped by their original suites, producing one feature per
suite rather than one feature per plan. Never replace suite identity with the
plan ID.

TestLink controls the effective permissions attached to an API token. Recommend
a dedicated least-privilege account whose role can retrieve only the required
project, suite, plan, case, and step data. A client-side allowlist is an
additional safety boundary, not a substitute for server-side permissions.
