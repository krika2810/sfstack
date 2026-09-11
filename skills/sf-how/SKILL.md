---
name: sf-how
description: "Trace how a Salesforce subsystem actually works at runtime: order of execution, automation map, schema, debug logs. Use for /sf-how, code walkthroughs before changing something, or 'where does this fire'."
---

# sf-how (Salesforce)

Answer how it works from evidence, not from reading one file. On Salesforce the behavior lives in the interactions: triggers, record-triggered Flows, validation rules, rollups, and platform events all fire in one transaction, in an order no single file shows.

## Method

1. **Map the automation surface** for the object(s) in question: triggers (source), Flows (source: `*.flow-meta.xml`), validation rules, required fields, rollups, duplicate/matching rules, and managed-package automations (visible in the org, not the repo - say so).
2. **Read the order of execution against the map**: before-save, validation, after-save, workflow-ish effects, async handoffs. Where exactly does the behavior in question happen, and what runs before and after it in the same transaction?
3. **Watch it run**: in a scratch org, set a trace flag, perform the operation (sf data or UI drive), pull the debug log (`sf apex tail log` / `sf apex get log`), and trace the actual sequence: Flow interview IDs, trigger entry/exit, SOQL counts. The log is ground truth; the source is the explanation.
4. **Scale check**: if the subsystem spans many objects or packages, spawn parallel explorer agents per slice and merge - one map, slices named.

Output: the sequence as it actually runs, with the source references and the log evidence. Diagrams when the order matters.
