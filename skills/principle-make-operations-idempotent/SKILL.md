---
name: principle-make-operations-idempotent
description: "Apply when designing commands, jobs, lifecycle steps, or processing loops that run amid crashes, restarts, and retries. Converge to the same end state regardless of partial prior runs."
disable-model-invocation: true
---

# Make Operations Idempotent

Design every state-changing operation so it converges to the same end state no matter how many times it runs or where a previous run died. Crashes, restarts, and retries are normal operating conditions, not edge cases. If partial state changes the next run's outcome, every restart becomes a debugging session.

## When it applies

Commands, async jobs, data loads, lifecycle steps, and processing loops: anything that mutates state and can be retried.

## The pattern

- **Convergent startup:** scan for existing state, clean stale artifacts, adopt what is already there instead of assuming a blank slate.
- **Content-based cleanup:** compare by content, not by creation order or timestamps.
- **Self-healing locks:** detect stale locks (a dead owner, an expired lease) instead of waiting forever.
- **Idempotent scheduling:** failed work respawns cleanly, and fresh input is regenerated each cycle.

## The test

1. What happens if this runs twice in a row?
2. What happens if the previous run crashed at every possible point?
3. Does re-execution converge to the same end state?

If any answer is "depends what was left behind", the operation needs a reconciliation step.

## Salesforce application notes

Idempotency is a survival trait on Salesforce because retries are built into the platform:

- **Data loads:** upsert on an external ID, never blind insert. A rerun after a partial Bulk API 2.0 job converges instead of duplicating. This is the **data-migration** playbook's core rule.
- **Platform-event and queueable consumers:** the same event can be delivered more than once. Consumers check state before acting ("has this order already been synced?") or write with upserts keyed on the event's identity.
- **Partial-success DML:** `Database.update(records, false)` can re-fire triggers inside the same transaction for retried subsets. Trigger logic that appends side effects without checking what already exists duplicates them.
- **Deploys:** `sf project deploy start` is naturally idempotent (it converges the org to the source), which is why "rerun the deploy" is a valid recovery and "hand-fix the org" is not.
- **Scheduled jobs and batch reruns:** a nightly job that reprocesses "everything modified since last success" must tolerate overlapping windows. Use a high-water mark stored durably, and make overlap harmless.

## Gotchas and failure modes

- **Idempotency that hides errors.** Reruns converge, but the first failure stays invisible. Convergence and error visibility are separate requirements; a job needs both.
- **Convergence that overwrites humans.** A job that "heals" by overwriting manual edits makes the platform fight its users. Converge to the intended state, and treat human changes as state to respect or flag, not stomp.
- **The reconciliation that never runs.** A reconciliation step nobody schedules is a reconciliation step that does not exist.

## Proof it applied

The twice-in-a-row test and the crash-at-each-point test were run (or walked through with evidence), and reruns converge: no duplicates, no missing work, no manual cleanup.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
