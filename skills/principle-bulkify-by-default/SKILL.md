---
name: principle-bulkify-by-default
description: "Apply to any data operation in Apex or Flow. All operations work on collections; no single-record code paths; trigger and invocable inputs handled as 200-record batches from the first line. Bulkification is necessary but not sufficient for large data volumes."
---

# Bulkify by default

The rule in one sentence: every SOQL query, every DML statement, and every Flow Get or Update element operates on a collection, never on one record at a time. Salesforce hands triggers up to 200 records at once, and the per-transaction governor limits (100 SOQL queries, 150 DML statements synchronous) punish code that treats them one by one. Bulkification is not an optimization pass you add later; it is the shape of the first line you write.

## When it applies

Every Apex trigger, invocable method, batch class, queueable job, and Flow that touches data. There is no "this will only ever see one record" exemption; the platform batches, integrations batch, and data loads batch, whether your code was ready or not.

## The rules

1. **SOQL and DML live outside loops.** Collect IDs in the loop, query once after it, update once at the end:
   ```apex
   // wrong: one query per record
   for (Order__c o : Trigger.new) {
       Account a = [SELECT Name FROM Account WHERE Id = :o.Account__c];
   }
   // right: one query for the batch
   Set<Id> accountIds = new Set<Id>();
   for (Order__c o : Trigger.new) { accountIds.add(o.Account__c); }
   Map<Id, Account> accounts = new Map<Id, Account>(
       [SELECT Name FROM Account WHERE Id IN :accountIds]);
   ```
2. **Helper methods take lists.** `process(List<Order__c> orders)`, never `process(Order__c o)` called in a loop. Single-record signatures invite single-record queries inside them.
3. **Triggers assume 200 records.** Write the trigger handler for the full batch from line one. Test both 1 and 200 records (see **principle-test-real-transactions**).
4. **Flows bulkify with collection variables.** A Get Records or Update Records element inside a Loop element is SOQL/DML in a loop wearing a declarative hat. Loop to build a collection variable, then one Update Records after the loop.
5. **Bulkification is necessary, not sufficient.** A bulkified transaction still dies when the work exceeds per-transaction limits (100 SOQL / 150 DML / 10,000 DML rows). Past that point the design is batch Apex, queueable chains, or Bulk API 2.0. That hand-off is **principle-async-for-volume**, and it is a design decision, not an accident.

## Gotchas and failure modes

- **The recursion trap.** Bulk-safe code re-entered by the order of execution (your update fires another trigger or flow that updates the same records) burns its limit budget on the second pass. Recursion guards must themselves be bulk-safe: a static `Set<Id>` of processed records, not a static `Boolean` that skips records 201-400 of a load.
- **Partial-success retries.** `Database.update(records, false)` can fire triggers again inside the same transaction for retried subsets. Static state is not reset between those fires; design for it.
- **"It works in the demo".** Ten seed records never trip a looped query. The 200-record test and the volume rehearsal are how you find out before production does.

## Proof you are following it

The bulk test (200 records) passes with flat resource usage: `Limits.getQueries()` and `Limits.getDmlStatements()` stay constant whether the batch is 1 record or 200, and the debug log's `LIMIT_USAGE_FOR_NS` block shows headroom. Reference for the limits: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm

## Source

Salesforce Well-Architected, Reliable - Performance: https://architect.salesforce.com/well-architected/overview and Apex governor limits documentation above.
