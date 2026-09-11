---
name: principle-fix-root-causes
description: "Apply when debugging. Trace each symptom to its root cause and fix it there: reproduce first, ask why until you reach the mechanism, no guards that silence crashes, fix the pattern not the instance."
disable-model-invocation: true
---

# Fix Root Causes

When debugging, do not fix symptoms. Trace the problem to its root cause and fix it there. Symptom fixes accumulate: each workaround makes the system harder to reason about, and the real bug is still in it. Root-cause fixes are slower today and faster forever.

## When it applies

All debugging. If you are about to write a guard, a retry, or a workaround, this principle is already in play.

## The pattern

1. **Reproduce first.** A bug you cannot reproduce is a rumor. Make it happen on demand before changing anything.
2. **Ask why until you hit the mechanism.** Not "where did it die" but "why did it die". The stack trace is the start of the question, not the answer.
3. **No silencing guards.** A null check that hides a crash is a symptom fix. The null means something upstream broke; find what.
4. **If a workaround needs a paragraph of comment to justify, the code is wrong.** Fix the code, not the comment.
5. **Fix the pattern, not the instance.** Grep for siblings of the same defect and fix them all, or write down why this one is unique.
6. **When stuck, instrument. Do not guess.** Add logging, read the actual error, watch the actual transaction.

## Salesforce application notes

- **The debug log is the instrument.** `sf apex tail log` at the right levels shows the actual order of execution, the actual queries, the actual limits. Most "mystery" behavior is the order of execution doing exactly what it documents: a workflow field update re-firing triggers, a roll-up re-running the parent save, a before-save flow writing the field after your trigger read it. Reference: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_order_of_execution.htm
- **"Fails after refresh or restart": suspect state, not code.** Stale org config, an old flow version still active, a named credential pointing at the old environment, cached platform cache values. If clearing state restores behavior, the fix is state validation.
- **Guards that hide limit problems.** Catching an exception and continuing leaves the transaction half-done. The root cause is usually a design that approached the limit at all (see **principle-respect-the-shared-runtime**).
- **Recursion guards are a confession.** A static boolean that stops re-entry treats the symptom. The root cause is which automation re-enters and why; sometimes the guard is the right call, but only after the re-entry is understood.

## Gotchas and failure modes

- **Why-chains past the point of action.** Five whys is a method, not a quota. Stop at the cause you can fix; the history of the universe is not the root cause of your null pointer.
- **Instrumentation heavier than the bug.** If reproducing needs a day of setup, weigh a cheaper observation first: an existing log, an audit trail, a query.
- **Root fixed, siblings left.** The same defect usually has siblings. Grep for the pattern and fix or rule out each one.

## Proof it applied

The reproduction exists, the mechanism is stated in one sentence, the fix addresses the mechanism, and sibling instances of the same defect were found and fixed or explicitly ruled out.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
