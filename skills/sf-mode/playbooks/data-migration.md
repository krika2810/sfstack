# Playbook: Data Migration

Loading or transforming large data volumes into or inside Salesforce. Volume changes every rule: the tool is Bulk API 2.0, the risks are skew and limits, and the plan is rehearsed before it is run.

## Steps

1. **Count and profile the data.** How many records, which objects, in what dependency order (parents before children). Profile for the two classic skews: ownership skew (more than about 10,000 records owned by one user) and lookup skew (more than about 10,000 child records pointing at one parent). Both cause lock contention and failed loads.
2. **Choose the tool by volume.**
   - Under roughly 10,000 records: `sf data import tree` or a simple insert is fine.
   - Beyond that: Bulk API 2.0, through the sf CLI:
     ```bash
     sf data upsert bulk --sobject Account --file accounts.csv --external-id External_Id__c --target-org target --wait 60
     sf data query --query "SELECT Id FROM Account WHERE ..." --bulk --target-org target --result-format csv --output-file extract.csv
     ```
   Reference: Bulk API 2.0, https://developer.salesforce.com/docs/atlas.en-us.api_asynch.meta/api_asynch/bulk_api_2_0.htm
3. **Rehearse in a scratch org or sandbox.** Run the full load against a copy first. Measure duration, failure rate, and which automations fired. Production loads re-fire every trigger, flow, and validation rule on every record; if the rehearsal blows the 24-hour asynchronous limit or takes days, redesign before the real run.
4. **Design for failure.** Bulk loads are idempotent by external ID: upsert, never insert, so a rerun after a partial failure converges instead of duplicating. Keep the CSV and the job IDs; every failed row must be explainable.
   ```bash
   sf data upsert bulk --sobject Contact --file contacts.csv --external-id Legacy_Id__c --target-org target --wait 60
   ```
5. **Manage the automation blast radius.** Decide deliberately whether triggers and flows should fire during the load. If they should not, use a bypass mechanism the codebase already has (a custom permission check in the trigger handler, for example), and remove the bypass immediately after.
6. **Run off-peak, in dependency order.** Parents first, then children, then junction objects. Large loads run off-peak because they compete with users for the same shared runtime.
7. **Verify by counts and samples.** Compare source counts to target counts per object, then sample records across the range and check field-level fidelity:
   ```bash
   sf data query --query "SELECT COUNT() FROM Account WHERE External_Id__c != null" --target-org target
   ```
8. **Hand back the report.** Per object: attempted, succeeded, failed with reasons, job IDs, and the verification counts.

## Gotchas and failure modes

- **Lookup skew locks.** Children pointing at one parent serialize on that parent's lock. Spread children across parents or load those objects last and slowly.
- **The 24-hour async window.** Batch and queueable work the load triggers counts against the org's daily asynchronous Apex executions (250,000 or 200 per license, whichever is greater). A migration that fires heavy automation can exhaust it. Reference: Execution Governors and Limits, https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm
- **Hard-deleting is one-way.** `sf data delete bulk` hard deletes bypass the recycle bin only with the hard-delete flag and the right permission. Treat deletes as irreversible and confirm with the user.
- **External IDs must be indexed.** Mark the external ID field as an external ID (which indexes it) or upserts do full-table work.
- **Validation rules from the new world break old data.** Legacy rows often fail current validation rules. Decide per rule: fix the data, or relax the rule for the load with a condition on the running user.

## Proof it worked

Per-object counts matching the source, zero unexplained failures, the Bulk API job IDs, and sampled records verified field by field.

## Reply

What was loaded, into which org, the numbers (attempted, succeeded, failed), the job IDs, and the follow-ups (bypasses removed, indexes added, failures to re-run).
