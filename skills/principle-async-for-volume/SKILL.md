---
name: principle-async-for-volume
description: "Apply to anything beyond small data volumes. Move volume out of the synchronous path: queueable for chainable jobs, batch Apex for large data volumes, Bulk API 2.0 for loads, platform events for decoupling. The sync path is for work the user is waiting on."
---

# Async for volume

The synchronous transaction is the most expensive real estate on the platform: 10 seconds of CPU, 100 SOQL queries, and a user staring at a spinner. Anything that does not need to finish before the user sees the screen does not belong there. Volume work - thousands of records, callouts to slow systems, fan-out notifications - belongs in asynchronous execution, where the budgets are bigger (60 seconds CPU, 200 queries) and nobody is waiting.

## When it applies

- Any operation whose record count can grow past the hundreds.
- Any callout to an external system that is not part of the user's click.
- Any fan-out: notifications, integrations, recalculations.

## The toolbox, and when each tool is right

1. **Queueable Apex** for chainable, record-scoped jobs: process these 200 orders, then chain the next chunk. Carries complex state, supports `System.enqueueJob`, chains one level deep from async context.
2. **Batch Apex** for large data volumes: up to 50 million records via a query locator, processed in chunks with per-chunk transactions. The default answer for "touch every record that matches".
3. **Platform events** for decoupling: the transaction publishes, subscribers process after commit in their own transactions. The right shape when the producer must not know or care who consumes.
4. **Bulk API 2.0** for data loads and extracts from outside: CSV in, CSV out, built for millions of records.
5. **Scheduled Apex** for time-based work. Note: scheduled jobs run under synchronous limits.

## The rules

1. **The sync path earns its place.** Ask of every synchronous operation: is the user waiting on this exact answer? If not, it is async.
2. **Design the failure story.** Async jobs fail out of sight. Queueable and batch jobs need error logging (a custom object, a platform event for failures, `TransactionFinalizers` for queueable). A failed job nobody sees is data corruption on a delay.
3. **Make reruns safe.** Async work gets retried, by the platform and by humans. Jobs must be idempotent: reprocessing the same record converges instead of duplicating. See **principle-make-operations-idempotent**.
4. **Mind the org-wide budget.** Async executions count against a 24-hour org limit (250,000 or 200 per license, whichever is greater). A chatty design - one queueable per record at volume - can exhaust it. Reference: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm
5. **Callouts have their own placement rules.** No callouts after DML in the same transaction ("uncommitted work pending"). The usual fix is exactly this principle: commit the DML, do the callout async.

## Salesforce example

```apex
// sync trigger hands volume work to a queueable, bulkified from the start
public class OrderSyncService {
    public static void enqueueSync(List<Id> orderIds) {
        if (orderIds.isEmpty()) { return; }
        System.enqueueJob(new OrderSyncJob(orderIds));
    }
}
```

## Gotchas and failure modes

- **Async is not a limit dodge.** A batch class with SOQL in a loop fails in every chunk, just later and quieter. Bulkification (**principle-bulkify-by-default**) still applies everywhere.
- **Stale reads in async paths.** Post-commit jobs re-query; the record may have changed since the trigger fired. Query fresh state inside the job.
- **Testing async.** Async only runs in tests inside `Test.startTest()`/`Test.stopTest()` (see **principle-test-real-transactions**). A test without them asserts nothing about the job.

## Proof you are following it

The sync transaction's log shows CPU and query headroom at 200-record volume, the async path has a visible failure record, and a rerun of the same job produces no duplicates.

## Source

Salesforce Well-Architected, Reliable - Performance: https://architect.salesforce.com/well-architected/overview; Execution Governors and Limits: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm
