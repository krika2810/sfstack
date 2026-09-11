---
name: principle-respect-the-shared-runtime
description: "Apply to any Apex transaction design. Governor limits are uncatchable: a breach kills the transaction with System.LimitException and no try/catch recovers it. Design so limits are never approached; probe headroom with the Limits class."
---

# Respect the shared runtime

Salesforce is multi-tenant: your code shares a runtime with every other customer on the pod, so the platform enforces per-transaction governor limits. The critical fact is not the numbers; it is the enforcement. A breached limit throws `System.LimitException`, which cannot be caught. Your transaction dies mid-work, with whatever partial state that implies. You do not handle limit breaches. You design so they never happen.

## When it applies

Every Apex transaction design: triggers, batch, queueable, invocable actions, web services, and every flow, because flows share the same transaction budget as the Apex around them.

## The numbers that shape design

Synchronous / asynchronous per-transaction (current documented values):

- SOQL queries: 100 / 200
- Records retrieved by SOQL: 50,000
- DML statements: 150
- Records processed by DML: 10,000
- Heap: 10 MB / 25 MB
- CPU time: 10 seconds / 60 seconds
- Callouts: 100 per transaction, 120 seconds cumulative timeout
- Trigger batch size: 200 records

Org-wide: 250,000 async Apex executions per 24 hours (or 200 per license, whichever is greater). Full table, which changes over time: Execution Governors and Limits, https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm

## The rules

1. **Design to the limit, not around the exception.** Before writing the transaction, count its worst-case queries, DML rows, and CPU at real batch size. If the count is near the limit, the design is wrong, not unlucky.
2. **Probe headroom when unsure.** The `Limits` class reports usage and ceilings at runtime:
   ```apex
   if (Limits.getQueries() > Limits.getLimitQueries() - 10) {
       // stop enqueueing more work in this transaction
   }
   ```
   Use it for adaptive batching and for asserts in bulk tests.
3. **Move volume out of the synchronous path.** The synchronous 10-second CPU budget is for work the user is waiting on. Everything else is queueable, batch, or platform-event driven (see **principle-async-for-volume**).
4. **Isolate the failure domains.** Callouts after DML fail with "uncommitted work pending". Async work enqueued at the wrong moment outlives its data. Transaction boundaries are design decisions; place them deliberately.
5. **Remember flows share the budget.** A record-triggered flow's queries count against the same 100-query limit as the trigger on the same object. The automation map (see **sf-how**) is where you see the total cost of a save.

## Gotchas and failure modes

- **Uncatchable means unrecoverable.** A `try/catch` around a SOQL loop does nothing for the limit; it only catches the catchable exceptions. Prevention is the only strategy.
- **Partial state on death.** When a transaction dies at a limit, everything rolls back, including work that succeeded. Side effects that cannot roll back (callouts already made, emails already sent) may have already escaped.
- **Managed packages share your org.** Certified packages get their own per-namespace limits for most counters, but CPU time and a few others are shared across all namespaces. A chatty package eats your CPU budget.

## Proof you are following it

Bulk tests assert headroom (`Limits.getQueries()` flat at 200 records), the debug log's `LIMIT_USAGE_FOR_NS` block shows the transaction far from every ceiling, and no design relies on catching `System.LimitException`.

## Source

Execution Governors and Limits, Apex Developer Guide: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm; Salesforce Well-Architected: https://architect.salesforce.com/well-architected/overview
