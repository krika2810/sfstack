---
name: principle-subtract-before-you-add
description: "Apply when sequencing an addition, refactor, or rewrite. Remove dead code, redundant automation, and speculative guards first, then build on the simpler base."
disable-model-invocation: true
---

# Subtract Before You Add

When evolving a system, remove first, then build. Adding to a complex system compounds the complexity; removing first leaves less code, reveals the essential structure, and usually makes the next design obvious. Default to subtraction, and make simplification a continual investment: leave every design slightly simpler and more capable behind the same or smaller surface than you found it.

## When it applies

Sequencing any addition, refactor, or rewrite.

## The pattern

1. **Sequence removal before construction.** The first commit deletes; the later commits build on the simpler base.
2. **Cut before you polish.** Get to the minimum before investing in quality on top of it.
3. **Design for observed usage, not speculative cases.** No validators, guards, or configuration for requirements nobody has.
4. **Delete stubs and dead references** rather than leaving them "for later". A reference with no novel content is load, not documentation.

## Salesforce application notes

- **Dead automation is the highest-value subtraction.** Inactive flows, retired Process Builder processes still present in the org, workflow rules nobody remembers: they are all reader load and reactivation risk. Removing them (with a **sf-blast-radius** check first) shrinks the surface every future change must consider.
- **Fields are not free.** Unused fields tax every layout, every report builder, every integration mapping. Deleting a field is a data decision, so check usage and archive the data first, but do delete.
- **Speculative Apex rots fastest.** The generic framework built for the second trigger that never came is subtraction material when the third year confirms it. Design for the automation that exists.
- **Subtraction applies to the org too, not just the repo:** stale scratch orgs (`sf org list --all`, then delete), expired named credentials, unused permission sets.
- **Ask the platform first.** Before adding custom anything, check whether a standard feature does it. The best subtraction is the code you never write (see **principle-laziness-protocol**).

## Gotchas and failure modes

- **Deleting the load-bearing fence.** Some ugly code guards a real constraint nobody wrote down. Understand why it exists (**sf-why**) before removing it, and say what replaced its function.
- **Blast-radius skipped under time pressure.** Deletion is the change most likely to break something far away. The **sf-blast-radius** pass is not optional because the deadline is close.
- **Subtraction as scope creep.** A feature PR that also deletes half the module is unreviewable. Sequence the subtraction as its own commit or PR before the addition.

## Proof it applied

The diff deletes before it adds, the deletions survived a blast-radius check, and the final surface is smaller or simpler than the starting one, with the suite green throughout.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
