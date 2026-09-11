---
name: sf-maintain-verification-skill
description: "Periodic pass that keeps a project's verification skill and Feature Map honest as the source changes: parallel source readers per feature, one live pass in a scratch org, at most one PR of proven corrections. Use for /sf-maintain-verification-skill or when the Feature Map has drifted from the deployed metadata."
---

# Maintain the verification skill

Feature Maps drift: tabs get renamed, Flows get deactivated, fields change type, a record page gets a new region. A stale map costs every future agent tokens and wrong turns. This pass re-grounds the map in the current source and proves every correction.

## Steps

1. **Source wave (parallel readers).** Fan out one reader per mapped feature over the current source: `sf sobject list` / `sf sobject describe` against a reference org, the metadata XML in the package dirs (flexipages, flows, tabs, layouts, permissionsets), and git log since the last maintain pass. Each reader reports: map says X, source says Y, confidence, proposed correction. Readers propose; they do not edit.
2. **Reconcile.** Collapse the reports into one candidate-correction list. Drop anything a reader asserted without source evidence. Group by feature file.
3. **Live pass.** Launch the verification skill once (its own Launch + Doctor). Drive each corrected feature exactly as the map instructs - the correction is only true if following the map works. Capture evidence for each drive.
4. **Write at most one PR.** One commit set, all proven corrections, evidence attached (deployment ID, screenshots, query results). Corrections that could not be proven live are listed in the PR body as open questions, not written into the map.
5. **Update the maintain stamp.** Record the date, the source SHA verified against, and the features driven in the map README.

## Rules

- Never let the map grow unchecked: merging two feature files beats three stale ones.
- A feature removed from the source is removed from the map in the same PR, with the source evidence of its removal.
- If Launch or Doctor fails, the project itself drifted - fix that first and report it; a map maintained against a broken base teaches wrong steps.
