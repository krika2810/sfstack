---
name: principle-respect-the-shared-runtime
description: "Apply to any Apex transaction design. Governor limits are uncatchable - System.LimitException cannot be try/caught - so prevent breaches by design, probe headroom with the Limits class, and treat CPU/heap as a budget shared with every managed package and Flow in the transaction."
---

# Respect the shared runtime

Your code does not run on your machine. Every transaction shares a governed runtime with the platform, managed packages, Flows, and other tenants' noise - and the limits are enforced by killing the transaction with an **uncatchable** `System.LimitException`. No try/catch, no finally, no recovery. Prevention is the only strategy.

The budget (synchronous): 100 SOQL queries, 50,000 rows retrieved, 150 DML statements / 10,000 rows, 100 callouts with 120s total timeout, 10 MB heap, 10 seconds CPU, 10-minute transaction ceiling. Async doubles some budgets (200 SOQL, 25 MB heap, 60s CPU) - one reason principle-async-for-volume exists.

Design rules that follow:

- **Budget before you build**: for any path that touches data at volume, estimate SOQL/DML/heap per record times batch size before writing the loop.
- **Probe at runtime**: `Limits.getQueries()` / `Limits.getLimitQueries()` (and the heap/CPU equivalents) in instrumented verification runs - publish the headroom in your evidence. "It passed" without numbers is not proof at volume.
- **CPU is shared**: platform code, managed packages, and Flows all spend the same 10 seconds. Tight Apex can still die to a chatty managed package - measure the whole transaction in the verification drive.
- **The org has limits too**: daily async executions, concurrent long-running transactions, API calls, and DevHub scratch org creation. Designs that assume unlimited async or unlimited scratch orgs fail at the org boundary, not the transaction boundary.
