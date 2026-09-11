---
name: sf-deploy
description: "Validate-first shipping to sandboxes and production: check-only deploy with tests, then deploy the validated job, keep deployment and test-run IDs as evidence, and smoke-check the behavior after. Use for /sf-deploy or any push to a shared org."
---

# sf-deploy

sf-deploy is the **deploy** playbook as a standing skill: the rules and commands for getting source into a shared org safely. The one-sentence version: nothing reaches production that has not passed a check-only deploy with tests against that production org, and every deploy keeps its IDs.

## When to use it

- Any deploy to a sandbox, staging, or production org.
- Destructive changes of any kind.
- Quick-deploying a validated change.

## The steps

1. **Freeze the deploy set.** The source directory or manifest, confirmed with `git status`. Nothing rides along untracked.
2. **Prove green in a scratch org.**
   ```bash
   sf apex run test --test-level RunLocalTests --target-org ci --result-format human --wait 20 --code-coverage
   ```
3. **Validate against the target.** The check-only deploy runs everything against the real org without committing:
   ```bash
   sf project deploy validate --source-dir force-app --target-org prod --test-level RunLocalTests
   ```
   Keep the validation job ID (`0Af...`). Reference: https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_validate.html
4. **Fix and revalidate** until validation is clean. Fixes happen in source, never in the target org.
5. **Deploy.** Quick-deploy the validated job, or run the full deploy for non-production targets:
   ```bash
   sf project deploy quick --job-id 0Afxx0000000001 --target-org prod
   sf project deploy start --source-dir force-app --target-org staging
   ```
   Production deploys require the user's explicit confirmation first. Record the deployment ID.
6. **Smoke check.** Drive the critical behavior once in the target org and capture the evidence.
7. **Report the IDs**: validation job ID, deployment ID, test-run ID, smoke-check evidence.

## Salesforce details worth knowing

- **Production gates.** Deploying to production requires at least 75% org-wide Apex coverage with all tests passing, and every trigger needs some coverage. Validation runs these gates for free, which is why validate-first works. Reference: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_intro_writing_tests.htm
- **Destructive changes** ship via a destructive manifest and deserve their own validation and a **sf-blast-radius** pass first:
  ```bash
  sf project deploy start --destructive-manifest destructiveChanges.xml --target-org staging
  ```
- **Flow versions.** A deployed flow lands as a new version and may need activation; verify which version is active afterward.
- **Async deploys.** Long deploys run async (`--async`); poll with `sf project deploy report --job-id 0Af... --target-org prod`.

## Gotchas and failure modes

- **Managed-package test failures.** `RunAllTestsInOrg` runs managed-package tests that fail for reasons unrelated to you. Know your failures from theirs before choosing the test level.
- **Partial deploys.** A class without its test, or metadata without its dependencies, leaves a state the repo never described. Deploy coherent units.
- **Validation expires.** A validated job can go stale as the target org changes. Revalidate if the org moved since step 3.
- **"Deploy succeeded" is not "feature works".** Metadata validity and behavior are different claims. Step 6 exists because of this.

## Proof it worked

The four IDs and the smoke-check evidence, named in the reply. A deploy without its evidence pack is treated as unverified work.

## Reply

What shipped, where, the IDs, smoke-check evidence, and anything deferred.
