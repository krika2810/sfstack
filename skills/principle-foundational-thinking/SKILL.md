---
name: principle-foundational-thinking
description: "Apply before writing logic: choose the data model and core structures first, sequence scaffold before features, ask what concurrent actors share. Get the foundations right so downstream code becomes obvious."
disable-model-invocation: true
---

# Foundational Thinking

Get the foundations right before writing logic. Structural decisions protect your future options; code-level decisions protect simplicity. Both are cheaper at the start than after the first feature lands.

## When it applies

Before writing logic: choosing data structures, sequencing scaffold versus features, deciding what concurrent actors share.

## The rules

1. **Data structures first.** Define the core shapes before the logic. Trace the dominant access patterns and choose structures that match them. When the data shape is right, the code that follows is obvious.
2. **DRY the structure, not every line.** Types and data models should converge; three similar statements still beat a premature abstraction. Prefer explicit over clever.
3. **Scaffold first.** Whatever every later phase needs goes first: CI, test infrastructure, the data factory, shared types, the seed data. Ask "does every subsequent phase benefit from this existing?" and if yes, build it now.
4. **Subtraction before scaffolding.** Remove the dead weight first (see **principle-subtract-before-you-add**), then lay foundations on the simpler base.
5. **The concurrency corollary.** Before actors share state, ask "what happens if another actor modifies this concurrently?" If the answer is not "nothing", isolate it (see **principle-separate-before-serializing-shared-state**).
6. **Small, single-purpose commits.** Each lands a coherent piece and leaves the suite green.

## Salesforce application notes

- **The data model is the foundation with the highest stakes.** Object, field, and relationship choices (lookup vs master-detail, junction objects, external IDs) are nearly permanent once data exists. They get decided here, deliberately, per **sf-architect**.
- **Scaffold on Salesforce means:** the DevHub and scratch definition that deploy cleanly (**sf-setup**), the `TestDataFactory`, the CI validation job (`sf project deploy validate` on every PR), and the project-local verification skill (**sf-create-verification-skill**). Every later phase uses all of them.
- **Sequence schema before logic.** The object and field definitions land in the first commit; the Apex that uses them lands on top. Reversing the order produces logic written against a shape that does not exist yet.
- **The sharing model is a foundation too.** OWD, role hierarchy, and sharing rules decided late become Apex patches forever. Decide them with the data model.

## Gotchas and failure modes

- **Scaffold as procrastination.** Infrastructure that no feature has asked for yet is speculation, not foundation. The test is "does every subsequent phase benefit", not "could this be useful".
- **Over-modeling upfront.** The data model needs to be right about what is known, not clairvoyant about what might come. Model observed requirements; leave room for the rest.
- **Foundations that never get used.** If the second feature does not touch the scaffold, the scaffold was wrong. Revisit it instead of defending it.

## Proof it applied

The schema, the scaffold, and the sharing model landed before the logic that depends on them, and each early commit stands on its own with the suite green.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
