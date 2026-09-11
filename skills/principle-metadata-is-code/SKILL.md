---
name: principle-metadata-is-code
description: "Apply when touching any Salesforce customization. Flows, validation rules, objects, layouts, and permission sets are source: version-controlled, reviewed, tested, and deployed through the pipeline. Never org-based development; audit-trail drift is a defect."
---

# Metadata is code

Well-Architected is blunt: do not use org-based development. Every customization - Apex, Flow, validation rule, field, layout, permission set - lives in version control in SFDX format and reaches orgs through `sf project deploy`, never through Setup clicks.

What that demands of agents:

- **Retrieve before you assume**: orgs drift from source (a hotfix in Setup, a consultan's sandbox experiment). `sf project retrieve start` and diff before building on an assumption about the org.
- **Declarative gets the same rigor**: a record-triggered Flow is code that runs in the transaction - reviewed, tested (flow tests + Apex tests of outcomes), blast-radiused before change.
- **Audit-trail entries that have no source counterpart are defects**: flag them, retrieve them into source or revert them, and name them in the PR.
- **Destructive changes are code review too**: every line of a destructive manifest reviewed against `/sf-blast-radius` output.
