# Playbook: multi-phase-plan

A tactical execution plan after a design is agreed - every task structured around proof.

1. **Break the design into small, self-contained PRs**, each deployable and verifiable on its own (sequence-verifiable-units).
2. **Every task names its proof**: the test run, verification drive, deployment validation, or measurement that shows it is done. Tests alone are not sufficient verification - include the live run.
3. **Order by dependency, deploy by scratch org**: each phase validated in isolation before the next builds on it.
4. **Validate the plan structure before executing** (script it if the plan is long - build-the-lever).
5. **Execute item by item under `/sf-show-me-your-work`**; the trail is how the human reviews later.
6. **Delete the plan when done.** A stale plan is confusion with a date on it.
