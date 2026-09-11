# Playbook: Deploy

Shipping source to a sandbox or production. The rule is validate first, deploy second, evidence always. A production deploy is the one irreversible act in this playbook set, so confirmation from the user gates the final step.

## Steps

1. **Confirm what is shipping.** `git status` and `git diff` against the target branch. The deploy set is explicit: the source directory or a manifest, never "whatever is in the org".
2. **Run the local suite in a scratch org first.** Production gates on 75% org-wide coverage and all tests passing, but the real bar is behavior:
   ```bash
   sf apex run test --test-level RunLocalTests --target-org ci --result-format human --wait 20 --code-coverage
   ```
   Green here is a precondition for step 3, not a substitute for it.
3. **Validate against the target org.** A check-only deploy runs the deploy and the tests against the real target without committing anything:
   ```bash
   sf project deploy validate --source-dir force-app --target-org prod --test-level RunLocalTests
   ```
   Record the validation job ID (starts with `0Af`). Reference: https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_validate.html
4. **Fix what validation finds, then revalidate.** Validation failures are almost always: a test that passes in scratch orgs but fails against production data or managed packages, a missing dependency (a field or permission set the manifest forgot), or an API version mismatch. Fix in source, never in the target org.
5. **Deploy with the validated job.** A successful validation can be quick-deployed without rerunning tests:
   ```bash
   sf project deploy quick --job-id 0Afxx0000000001 --target-org prod
   ```
   For production, get the user's explicit confirmation before this step. Record the deployment ID.
6. **Smoke check after deploy.** Run the behavior once in the target org: the critical user path, one record through the automation, a query that shows expected state. Capture the evidence (screenshot or query output).
7. **Report the IDs.** Deployment ID, validation job ID, test-run ID, and the smoke-check evidence.

## Gotchas and failure modes

- **Destructive changes are a separate deploy.** Removing fields, objects, or classes needs a destructive changes manifest (`sf project deploy start --destructive-manifest destructiveChanges.xml`) and its own validation pass. Deleting metadata that other metadata references fails; run **sf-blast-radius** first.
- **Flow version gotchas.** Deploying a Flow creates a new version; the old active version can stay active in the target. Confirm which version is active after the deploy (`sf data query --query "SELECT Id, VersionNumber, Status FROM Flow WHERE ..."`).
- **Test-level discipline.** `RunLocalTests` is the default bar. In production, managed-package tests can fail for reasons unrelated to you; know which failures are yours before rerunning with `RunAllTestsInOrg`.
- **Partial deploys hide drift.** Deploying one class without its test, or the test without the class, leaves the org in a state the repo never described. Deploy the coherent unit.
- **The deploy succeeded is not the feature works.** Metadata validity is step 5. Behavior is step 6. Both are required.

## Proof it worked

Deployment ID, validation job ID, test-run ID with pass counts and coverage, and smoke-check evidence from the target org. All four, named in the reply.

## Reply

What shipped, where, the four IDs, and anything deferred (known failing managed-package tests, flow versions to activate, follow-up destructive changes).
