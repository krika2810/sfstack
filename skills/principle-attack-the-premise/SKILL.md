---
name: principle-attack-the-premise
description: "Apply when two or more fixes that share one premise have failed the same gate. Take a census of which actors hold the imbalance before the next fix, then question the premise instead of writing another fix that assumes it."
disable-model-invocation: true
---

# Attack the Premise

When two or more fixes that shared one assumption have failed the same way, stop fixing. The fixes were not the problem. The shared assumption was. Each failure under the same premise is evidence about the premise, and the evidence is telling you to question it.

## When it applies

Debugging or hardening work where the same failure keeps coming back after fixes that each looked reasonable. The tell is a history of "fixed it" commits touching the same symptom.

## The pattern

1. **Write the premise down.** One sentence that every failed fix assumed. "The duplicate syncs come from the queueable being enqueued twice." Until this sentence exists, you are guessing.
2. **Take a census before the next fix.** Count the imbalance per actor: which records, which users, which transactions hold the problem. Write the census as a rerunnable script or query so it can be checked again after the next change (see **principle-build-the-lever**).
3. **Read the skew.** If the same few actors hold most of the imbalance on every run, something assigns them that role. Find what assigns it. That assignment is the next "why" (see **principle-fix-root-causes**).
4. **Remove the asymmetry instead of compensating for it.** Rotate the role, randomize the assignment, or move it, so no actor holds it on every run. A compensation (a retry, a cleanup job, a periodic rebalance) leaves the assignment in place and adds work forever.

## Stop conditions

- Do not start the next fix before the premise is written down and the census exists.
- If the census is even across actors, the premise is not the cause. Look elsewhere, and keep the census as evidence.

## Salesforce application notes

- The classic Salesforce case: "duplicate records keep appearing despite three trigger fixes." The census query (`SELECT External_Id__c, COUNT(Id) FROM Contact GROUP BY External_Id__c HAVING COUNT(Id) > 1`) shows the skew lives in one integration user. The premise "the trigger deduplicates" was wrong; the assignment is "that integration bypasses the trigger's entry conditions." Fix the assignment, not the trigger a fourth time.
- Another common one: repeated governor-limit fixes on one object. The census (`LIMIT_USAGE_FOR_NS` lines across transactions) shows one managed package holds most of the CPU. The premise "our code is too slow" gives way to "this object carries an automation load our code never consented to."
- The census tool on Salesforce is usually a SOQL aggregate query or a debug-log analysis script, both cheap and rerunnable.

## Gotchas and failure modes

- **The census becomes the goal.** Measuring the imbalance forever instead of fixing its assignment. The census exists to point at the fix, not to be admired.
- **A census that misses actors.** If the query cannot see some actors (an integration that writes through a different path, a managed package's hidden automation), the skew reading is fiction. Check the observation method before trusting the skew.
- **Even census, wrong conclusion.** An even spread does not prove the premise right; it proves this premise is not the cause. Record it and move to the next hypothesis.

## Proof it applied

The premise sentence is written down, the census exists as a rerunnable artifact, and the fix addresses the assignment of the imbalance rather than compensating for it. If the census cleared the premise, that is recorded too.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
