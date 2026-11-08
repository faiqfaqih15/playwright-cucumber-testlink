# TestLink to Gherkin conversion and synchronization

Use normalized `ImportedTestCase` values only. Complete the workflow in this
order, without skipping or reordering gates:

Scan existing `@TestLinkSuite_*` and `@TestLinkCase_*` tags first. For existing
mappings, require a unique one-to-one identity mapping before preview. A source
identity matching zero target tags is a valid new-case addition; classify it as
`added`. Exactly one unique target mapping may be preserved or updated as
described by the semantic diff. More than one target tag for one source, or
multiple source identities matching one target, is many-to-one ambiguity.
Duplicate suite or case tags are ambiguous even when their surrounding names
match. On any such ambiguity, abort without writes; do not choose a likely
mapping or ask for approval of a partial diff. Use the executable classifier in
`templates/testlink-synchronization.ts.template` when implementing this gate.

```text
retrieve -> validate -> group -> inspect existing code -> normalize -> detect equivalent steps -> preview semantic diff -> approval -> write
```

Authentication errors, transport errors, empty selections, duplicate external
IDs, incomplete steps, ambiguous suite names, or any other validation failure
stop the sequence before target-project files are written.

## Stable mapping

Map one TestLink suite to one feature:

- Feature name: the suite name.
- Feature tag: `@TestLinkSuite_<suiteId>`.
- Scenario name: the case title.
- Scenario tag: `@TestLinkCase_<externalCaseId>`.
- Scenario steps: ordered actions and their expected results, expressed as
  business-readable Given/When/Then behavior.
- Preconditions: a Background only when every case in the suite shares the same
  precondition; otherwise scenario-local Given steps or descriptive context.

Use `templates/feature.template` as the representative mapping. A plan ID is
execution metadata, not permanent identity, so never put it in a feature or
scenario traceability tag. For plan imports, retain the original suite groups
and generate or synchronize one feature for each suite.

Normalize whitespace, safe HTML formatting, vocabulary, and equivalent
parameter forms without changing business meaning. Inspect existing feature,
step-definition, page, component, API, and helper code first. Reuse an existing
step expression when it has the same meaning; do not create a near-duplicate
phrase merely because the imported wording differs.

## Semantic diff preview

Before requesting approval, show the proposed file paths, suite/case identity,
and a semantic diff with these categories:

- **added:** a selected external case or business step has no current mapping.
- **changed:** the imported action or scenario meaning differs from the mapped
  feature behavior.
- **removed:** an existing tagged case or step is absent from the selected
  authoritative scope; show it in preview but do not remove it automatically.
- **expected-result:** TestLink's expected result differs from the observable
  Then behavior.
- **precondition:** setup differs in meaning or placement between TestLink and
  the current Background or Given steps.

Distinguish wording-only normalization and equivalent reusable steps from real
semantic changes. The preview must state that no files have been changed and
must identify any affected TypeScript implementations.

## Approval and write boundary

Wait for explicit approval of the displayed semantic diff and exact feature
file plan. After approval, update only the approved feature scenarios. Always
preserve existing TypeScript step/page/API implementations, even when the diff
shows they may no longer match; report those affected implementations for
manual or later separately approved changes. Never overwrite, delete, or
regenerate implementation files during TestLink synchronization.

If the source data or target files change after preview, invalidate approval,
retrieve and inspect again, then present a fresh diff. Report completed feature
changes and remaining implementation follow-up without reporting a TestLink
result or mutating TestLink.
