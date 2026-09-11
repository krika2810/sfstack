---
name: sf-blast-radius
description: "Find what a Salesforce change could break somewhere else before it ships - via the metadata dependency graph and real test runs, not assertion. Use for /sf-blast-radius, before renaming/deleting fields, classes, or flows, or when a 'small' change touches shared metadata."
---

# sf-blast-radius

On Salesforce the blast radius of a change is mostly invisible in the diff: a field feeds a Flow, a report, an integration mapping, and three list views, none of which are in your package dir's call graph. Find the real dependents, then prove the one fact it is safe because of.

## Steps

1. **Dependency graph query.** Use the Tooling API through the CLI against a reference org:
   `sf data query --tooling-api --query "SELECT MetadataComponentName, MetadataComponentType, RefMetadataComponentName, RefMetadataComponentType FROM MetadataComponentDependency WHERE RefMetadataComponentName = '<name>'"` - and the reverse direction. Note: the dependency graph is not exhaustive (dynamic references, reports, list views, email templates, and external integrations can be invisible to it) - say which categories your query cannot see.
2. **Source grep.** Search the package dirs for the API name in flows, validation rules, formulas, flexipages, layouts, permission sets, named credentials, and Apex (including dynamic SOQL strings - the grep the dependency graph cannot do).
3. **Runtime dependents.** Reports, dashboards, list views, connected apps, and integration users that read the element. Check what you can via the CLI; list what needs human confirmation.
4. **Prove the safety fact.** State the one fact the change is safe because of ("nothing references X except Y, and Y is updated in this PR"), then prove it: deploy the change to a scratch org with the dependents, run the test suite and the verification drives touching the dependents. A green run is the proof; the dependency query is only the hypothesis.
5. **Report**: dependents found, categories not visible to the query, the safety fact, and the evidence (deployment ID, test-run ID).
