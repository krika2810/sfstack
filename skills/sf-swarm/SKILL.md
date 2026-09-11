---
name: sf-swarm
description: "Fan out N parallel workers across scratch orgs or metadata slices and aggregate one report. Use for /sf-swarm, coverage matrices (test every object against every operation), races against a deadline, or partitioning a large read-only exploration."
---

# sf-swarm

sf-swarm is parallelism with a plan. It splits one large job into independent slices, runs the slices at the same time (each in its own scratch org or on its own metadata slice), and merges the results into one report. It is for coverage and speed, not for making one hard decision: that is sf-arena.

## When to use it

- A coverage matrix: every core object x every operation (create, update, delete, bulk load) verified.
- A wide read-only sweep: review all 40 flows for a specific anti-pattern, scan all triggers for hard-coded IDs.
- A deadline race: N independent work items that must all land today.

## The steps

1. **Define the matrix or partition.** Write down the slices explicitly: the object list, the flow list, the work items. Every slice names its scope and its deliverable.
2. **Check independence.** Slices must not write the same records, the same metadata, or the same org. If two slices share state, separate the state first (**principle-separate-before-serializing-shared-state**) or merge the slices.
3. **Assign each slice its resources.** Mutating slices get their own scratch org:
   ```bash
   sf org create scratch --definition-file config/project-scratch-def.json --alias swarm-orders --duration-days 2
   ```
   Read-only slices share one org freely: queries and retrieves do not collide.
4. **Run the slices.** Each worker runs its slice with the same rigor as a solo task: deploy, drive, capture evidence. The worker's output is a structured result, not prose: slice name, outcome, evidence IDs, anomalies.
5. **Aggregate into one report.** Merge the results into a single table. Anomalies get named individually, never averaged away. The report answers: what passed, what failed, what needs a human.
6. **Clean up.** Delete the swarm orgs.

## Salesforce details worth knowing

- **DevHub caps bound the swarm size.** Daily scratch org creations and active scratch orgs are limited per DevHub. A 20-slice mutating swarm can exhaust the daily cap; batch the waves or reuse orgs across slices that run at different times.
- **Parallel test runs in one org serialize.** Apex tests in one org run through a shared queue; a swarm of test runners against one org does not scale. One org per runner when tests are the work.
- **API request limits are per org.** Read-only swarms hammering one org's REST API can hit the 24-hour API request allocation on small editions. Spread reads or use bulk queries for large extracts.

## Gotchas and failure modes

- **Slices that are not independent.** Two workers updating the same Account in one org produce lock errors and flaky results. Independence is checked in step 2, not discovered in step 4.
- **Aggregation that hides failures.** "18 of 20 passed" without naming the 2 is a report that creates a second pass. Name every anomaly.
- **Org sprawl after the run.** Swarms are the fastest way to hit the active-org cap. Step 6 is part of the work.

## Proof it worked

The aggregate report exists with every slice accounted for, each result carries its evidence (test-run IDs, deployment IDs, log excerpts), and `sf org list --all` shows the swarm orgs deleted.

## Reply

The matrix, the per-slice results table, the anomalies by name, and the follow-ups.
