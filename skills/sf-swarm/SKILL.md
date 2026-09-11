---
name: sf-swarm
description: "Fan out N parallel workers across scratch orgs or metadata slices, drain them, return one aggregated report. Use for /sf-swarm, 'swarm this', big-sample verification, fuzzing with randomized data, or repo-wide checks."
---

# sf-swarm

pstack's swarm, with scratch orgs as the fleet.

## Patterns

- **Big-sample verification**: run the same verification drive in N scratch orgs to trust a result (flaky test hunts, performance measurements, order-of-execution race checks).
- **Fuzzing**: N workers load randomized or adversarial data (bulk loads, skewed ownership, 10k+ children, unicode, max-length fields) and report what breaks. Governor limits love adversarial data.
- **Slice coverage**: one worker per metadata slice (per object, per flow, per package dir) for repo-wide checks: every Apex class has meaningful tests, every flow has entry criteria, no hard-coded IDs anywhere.

## Rules

- Budget scratch orgs before launching: DevHub daily cap minus what is already alive is your fleet size. Queue the rest; do not fail mid-swarm.
- Every worker returns structured results (same shape), and the report aggregates - one table, outliers named, evidence links.
- Workers never share an org. Isolation is the point.
- The swarm proves at scale what one run suggests; it never replaces the single careful drive.
