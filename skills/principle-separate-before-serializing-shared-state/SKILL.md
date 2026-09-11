---
name: principle-separate-before-serializing-shared-state
description: "Apply when concurrent actors might write the same record, file, or shared state. Eliminate the sharing first; serialize structurally only when one shared writer is a real invariant. Instructions are not concurrency control."
disable-model-invocation: true
---

# Separate Before Serializing Shared State

When concurrent actors might write the same state, first ask whether they need the same mutable object at all. Most sharing is accidental, and the fix is to eliminate it, not to manage it. Concurrent writes to shared state create races that are intermittent, hard to reproduce, and expensive to debug. When sharing turns out to be real, enforce serialization structurally, because conventions and instructions are not concurrency control.

## When it applies

Any design where two or more actors (jobs, integrations, users, agents) can write the same record, file, branch, or state object.

## The pattern

1. **Identify the shared mutable state.** What gets written by more than one actor?
2. **Default: eliminate the shared write target.** Do the actors need one canonical object, or are they publishing independent facts? Give each actor its own record, file, key, or branch, and merge only at the read or reporting boundary.
3. **Only when one shared target is a real invariant, serialize structurally:** a lock, a queue, a single-writer design, an atomic compare-and-swap. Treat "we need a lock" as a design smell to check, not the default answer.

## Salesforce application notes

- **Record lock contention is the platform's version of this.** Data loads that hammer one parent record (lookup skew: thousands of children on one parent) serialize on that parent's lock and fail with `UNABLE_TO_LOCK_ROW`. The separate-first fix: spread children across parents, reorder the load, or restructure the relationship. The serialize fix (retry loops) is the fallback.
- **Ownership skew** (one user owning tens of thousands of records) causes sharing recalculation storms. Separate ownership across real owners instead of routing everything to one integration user.
- **Concurrent integrations writing one "status" record** is the classic accidental sharing. Give each integration its own status record or custom metadata row, and have the UI read the join.
- **Static variables are shared state within a transaction.** Two triggers on one object writing the same static collection is intra-transaction sharing; the fix is one owner of the state with the others calling it.
- **Swarm work (**sf-swarm**) applies this directly:** each worker gets its own scratch org or its own metadata slice, and results merge at the report, never in a shared org mid-run.

## Gotchas and failure modes

- **Separation that duplicates truth.** Two records that must agree is shared state wearing a partition. Separate targets only work when the facts are genuinely independent.
- **Reaching for the lock first.** Serialization feels safe and hides the design question. Most of the time the sharing was accidental and the lock just makes the accident official.
- **Merge-at-read with no defined merge.** If readers cannot reconcile the separate targets, you have moved the race to read time.

## Proof it applied

Shared write targets were enumerated, each was either eliminated (separate targets, merge at read) or structurally serialized with the mechanism named, and no concurrency guarantee rests on a convention.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
