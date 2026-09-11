---
name: sf-deploy
description: "Validate-first shipping for Salesforce: check-only deployment with tests, then deploy, evidence kept, post-deploy smoke check. Use for /sf-deploy, 'ship this', release prep, or any sandbox-to-production move."
---

# sf-deploy

Deployment on Salesforce is an async, transactional, gated operation - treat it as the risky part of the pipeline, not the formality at the end.

## Steps

1. **Preflight.** Clean git state; the exact source SHA named. `/sf-blast-radius` on anything deleted or retyped. Destructive changes get a line-by-line human review of the destructive manifest - no exceptions.
2. **Validate first**: `sf project deploy validate` (check-only deploy) against the target with the right test level (`--test-level RunLocalTests` for production; RunSpecifiedTests only with a stated reason). This runs the real deployment analysis and the tests without committing the change. A validate you did not run is a deployment you have not tested.
3. **Deploy the validated package**: quick-deploy the successful validation when the window allows, otherwise `sf project deploy start` with the same test level. Record the deployment ID.
4. **Evidence bundle**: deployment ID, test-run ID, coverage delta, validate output. These go in the PR or release notes - they are how anyone re-checks this deploy later.
5. **Post-deploy smoke**: one verification-skill drive of the primary user path in the target org (for production: a read-only or carefully chosen probe). Deployment success means metadata landed; smoke means the system works.
6. **Rollback plan before you need it**: the previous source SHA and the redeploy command, written down before step 3. Rollback on Salesforce is a redeploy - there is no undo button.

## Rules

- Never deploy to production outside the agreed window; never deploy on a Friday afternoon unless the human explicitly owns that call.
- If validate fails, the failure is the work: fix and re-validate. Do not "try the deploy anyway" - the same gate stops it, now with users watching.
- Org differences (data, managed package versions, enabled features) are the classic validate-green/deploy-red cause. Note known deltas between source and target in the evidence bundle.
