---
name: principle-metadata-is-code
description: "Apply when touching flows, validation rules, objects, permissions, or any Salesforce metadata. Metadata is code: it lives in version control, moves through source-driven deployment, and gets review, tests, and blast-radius checks like Apex. Org-based development is an anti-pattern."
---

# Metadata is code

On Salesforce, a Flow is an XML file. A validation rule is an XML file. A custom object, a permission set, a layout: XML files. They define behavior exactly as much as Apex does, and they break production exactly as hard. So they get the same engineering rigor: version control, review, tested deployment, and a blast-radius check before change. The alternative, clicking changes straight into a production org's Setup menu, is what Salesforce Well-Architected calls org-based development, and it names it an anti-pattern.

## When it applies

Touching any metadata component: flows, validation rules, custom objects and fields, permission sets, layouts, Lightning pages, workflow remnants, reports and dashboards that are in source.

## The rules

1. **All changes start in source.** The repo is the truth. Orgs are projections of it. A change that exists only in an org does not exist.
2. **Move metadata with the CLI.** Retrieve to edit, deploy to apply:
   ```bash
   sf project retrieve start --metadata Flow:Order_Routing --target-org dev
   sf project deploy start --source-dir force-app --target-org staging
   ```
3. **Review metadata diffs like code diffs.** A flow XML diff shows entry conditions, element logic, and fault paths. Read it in review with the same hostility you bring to Apex.
4. **Test what can be tested.** Declarative flow tests (`FlowTest` metadata) assert interview outcomes and run in deployments. Validation rules get Apex tests that attempt the violating write. Untestable automation is a design smell; consider whether the behavior belongs in Apex.
5. **Blast-radius before change.** Fields and flows are referenced by reports, integrations, and other metadata. Run **sf-blast-radius** before deleting or reshaping shared components.
6. **Track what source cannot represent.** Some configuration lives only in orgs (some settings, data, certain metadata types). Keep the list explicit and small, and never let it silently grow.

## Gotchas and failure modes

- **Hotfixes that stay in the org.** The production hotfix clicked into Setup and never retrieved creates drift that the next deploy overwrites or, worse, half-overwrites. Hotfix in source, deploy normally, even under pressure.
- **Flow version sprawl.** Every deploy creates a new flow version. Old versions accumulate; obsolete ones should be deactivated and deleted deliberately.
- **Retired automation that still runs.** Workflow rules and Process Builder processes still execute. The metadata-as-code rule covers them too: retrieve, review, and migrate them to flows on a plan, not by accident.
- **Assuming declarative means safe.** A record-triggered flow shares the transaction and the governor budget with Apex. It can blow limits and corrupt data with zero lines of code involved.

## Proof you are following it

`git status` shows the change in source, the deploy moved it (deployment ID captured), no hand-edits exist in the target org, and the change passed the same review and verification bar as an Apex change.

## Source

Salesforce Well-Architected, Adaptable - Resilient (source-driven development, org-based development named as anti-pattern): https://architect.salesforce.com/docs/architect/well-architected/guide/resilient
