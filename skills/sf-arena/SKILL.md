---
name: sf-arena
description: "Compete N designs or implementations against each other, each in its own scratch org, and let measured evidence pick the winner. Use for /sf-arena, contested designs, or when two approaches both look right and arguing is cheaper than measuring."
---

# sf-arena

Some design questions cannot be settled by reading. Will the trigger handler pattern or the domain class pattern stay cleaner as this object grows? Does the queueable chain beat the batch job for this workload? sf-arena settles them the Salesforce way: every candidate gets its own scratch org, the same workload runs against each, and the numbers decide.

## When to use it

- Two or more structurally different designs for the same problem, both defensible.
- A performance claim that reading cannot settle ("this query will be fine at volume").
- Choosing between automation placements: flow vs trigger vs async for the same behavior.

## The steps

1. **Write the contest question in one sentence.** "Which design keeps Order sync under the 10-second CPU limit at 200-record batches?" Everything below serves that sentence.
2. **Define the workload and the metric before building anything.** The workload: which records, how many, through which path (UI save, Apex script, Bulk API load). The metric: CPU time from `LIMIT_USAGE_FOR_NS`, SOQL count, wall time, test suite duration, or diff size for maintainability contests. A contest without a pre-agreed metric is a debate with extra steps.
3. **Give each candidate its own scratch org.**
   ```bash
   sf org create scratch --definition-file config/project-scratch-def.json --alias arena-a --duration-days 3
   sf org create scratch --definition-file config/project-scratch-def.json --alias arena-b --duration-days 3
   ```
   Deploy each candidate to its org. Isolation is the point: no shared state, no cross-contamination, and cleanup is `sf org delete scratch`.
4. **Run the same workload against each org.** Same seed data, same script, same measurement:
   ```bash
   sf apex run --file workload.apex --target-org arena-a
   sf apex tail log --target-org arena-a   # capture LIMIT_USAGE_FOR_NS
   ```
5. **Compare and pick.** Lay the numbers side by side. Pick the winner on the metric; break ties toward the simpler design (**principle-subtract-before-you-add**). Write down why the loser lost in one sentence.
6. **Graft, do not restart.** The winner proceeds. If the loser had one genuinely better part, graft that part onto the winner deliberately instead of blending both designs.
7. **Clean up.** Delete the arena orgs. DevHub active-org caps are real, and arena orgs multiply fast.

## Gotchas and failure modes

- **Unequal workloads.** Different seed data or a different script per org invalidates the contest. Step 4 runs the identical artifact everywhere.
- **Measuring the wrong thing.** Developer convenience is a real metric for maintainability contests, but measure it (lines changed to add a case, cyclomatic shape of the diff) instead of asserting it.
- **Volume fakery.** A performance contest on 50 records proves nothing about 200-record trigger batches or million-row tables. Load representative volume with Bulk API 2.0 first.
- **Org caps.** Each DevHub caps daily creations and active scratch orgs. Keep arenas small (two or three candidates) and delete as you go.

## Proof it worked

The contest question, the pre-agreed metric, the per-org measurements with their org aliases and log excerpts, and the winning decision with the one-sentence reason the loser lost.

## Reply

Question, metric, numbers per candidate, winner, and the graft decision.
