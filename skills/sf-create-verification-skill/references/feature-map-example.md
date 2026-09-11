# Feature Map example - Preferences (from pstack Part 1, Salesforce-translated)

## Feature: Account Health Dashboard

Sub-features: health-badge (LWC on the Account record page), health-recalc (invocable Apex called by a record-triggered Flow), nightly-rollup (scheduled batch).

## How to get to it (user POV)

Open any Account record. The badge renders in the highlights panel. Recalc fires automatically on edit of AnnualRevenue or Industry. The nightly job is invisible to users.

## Driving it with sf + browser

- UI: open the org (`sf org open --url-only --json` -> browser session), navigate to a seeded Account (`/lightning/r/Account/<id>/view`), screenshot the badge.
- Data path: `sf data update record --sobject Account --record-id <id> --values "AnnualRevenue=5000000"`, then re-query `HealthScore__c` and assert the badge value changed.
- Logic path: `sf apex run --file scripts/recalc.apex` to invoke the entry point directly; check the debug log for the Flow interview ID.
- Batch path: `sf apex run` to execute the batch synchronously in a test context, or observe via `sf data query --query "SELECT Status FROM AsyncApexJob ..."`.

## Gotchas

- The recalc Flow is after-save; the badge does not update until the transaction commits - re-query, do not read the pre-update value.
- HealthScore__c is FLS-restricted to the Health App permset; the verification user needs `sf org assign permset --name Health_App`.
- The batch is scheduled, not triggered; in a scratch org either run it manually or shorten the schedule in seed metadata.
