# Playbook: Perf Issue

A measured slowness to trace and improve against a baseline. The deliverable is a measured before, a measured after, and the smallest change that explains the difference. No speculative optimization ships.

## Steps

1. **Get the baseline number.** Reproduce the slowness in a scratch org or sandbox with production-like data volume and capture the measurement: the debug log's `LIMIT_USAGE_FOR_NS` block (CPU time, SOQL count, heap), the transaction duration in the log, or the page load time from the browser. A complaint without a number is an investigation, not a perf issue. Run the **investigation** playbook first if there is no measurement.
2. **Localize the cost.** Read the debug log at `FINEST` for `PROFILING` (or use the Analysis perspective in Developer Console) and find where the time actually goes. On Salesforce the usual costs, in order of frequency:
   - Non-selective SOQL: a query scanning a large table because its filter is not indexed. Check the query plan in Developer Console (Query Plan tool) or via the REST explain endpoint.
   - SOQL or DML in a loop: N queries where one would do. Count `SOQL_EXECUTE_BEGIN` entries in the log.
   - Re-entrant automation: the order of execution running your code more times than you expect (workflow field updates, roll-up summaries, recursive saves).
   - Serial processing of volume that belongs in batch or queueable Apex.
3. **Form one hypothesis, change one thing.** Write the hypothesis as a sentence: "The 8 seconds is one non-selective query on `Order__c.Status__c`." Then make the single change that tests it: add a selective filter, bulkify the loop, move the work async, or add a Platform Cache layer for repeat reads.
4. **Measure after, same way.** Rerun the exact step-1 measurement. Report before and after as numbers. If the hypothesis failed, revert and return to step 2 with the next hypothesis.
5. **Verify no regression.** Run the local test suite. Performance fixes that change query shape or transaction boundaries can change behavior:
   ```bash
   sf apex run test --test-level RunLocalTests --target-org perforg --result-format human --wait 20
   ```
6. **Run Opening a PR** with the before/after numbers in the body.

## Gotchas and failure modes

- **Testing with tiny data.** Selectivity problems are invisible on 50 seed records. Load representative volume with Bulk API 2.0 before measuring: `sf data upsert bulk --sobject Order__c --file orders.csv --external-id ExtId__c --target-org perforg --wait 30`.
- **Caching that hides staleness.** Platform Cache is right for repeated reads of slowly-changing data, wrong for anything transactional. State the invalidation rule when you add a cache.
- **The 10-second CPU wall.** Synchronous transactions get 10 seconds of CPU (60 seconds async). If the work cannot fit, the fix is an async design, not faster code. Reference: Execution Governors and Limits, https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm
- **Query selectivity rules.** A filter is generally selective when it matches under about 10% of the first million records (and less beyond that), using indexed fields with positive operators. `!=`, `NOT IN`, `LIKE '%term'`, and null comparisons defeat standard indexes. See **principle-selective-queries**.

## Proof it worked

Before and after numbers side by side from the same measurement method, the log excerpts that show the localized cost, and a green test-run ID after the change.

## Reply

The baseline, the root cause, the change, the after number, and how much headroom remains against the relevant limit.
