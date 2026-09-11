---
name: principle-exhaust-the-design-space
description: "Apply when facing a novel interaction or architectural decision with no precedent. Build 2-3 structurally different candidates and compare side by side before committing. Design it twice."
disable-model-invocation: true
---

# Exhaust the Design Space

When the right answer is not obvious and nothing in the codebase sets a precedent, do not marry the first design that works. Build two or three structurally different candidates, compare them side by side, and only then commit. Building the wrong thing costs more than exploring three options. "Design it twice" is this rule by another name, and a second flavor of the first shape does not count as a second design.

## When it applies

- A novel UI interaction with no prior art in the codebase.
- An architectural choice with several viable approaches.
- A product decision where quality depends on feel, not logic.

## When it does not

- Mechanical implementation where the pattern is established.
- Bug fixes and refactors with a clear target state.
- Cases where constraints dictate one viable approach. Then say so and proceed.

## Salesforce application notes

On Salesforce, whole-shape alternatives are usually about placement and structure, not syntax:

- **Automation placement:** before-save flow vs before trigger vs async queueable for the same behavior. These are structurally different: different order-of-execution phases, different limits exposure, different testability. That is a design-space question, and **sf-arena** is the tool: each candidate in its own scratch org, the same workload, measured.
- **Data model shapes:** one object with a type field vs separate objects vs a junction object. Nearly permanent once data lands, so the comparison happens before.
- **Integration shapes:** platform events vs queueable-with-callout vs change data capture. Different failure modes, different coupling.
- **UX on Lightning:** an LWC on the record page vs a flow screen vs a quick action. Prototype cheaply and compare with the actual user path.
- The comparison is measured, not argued: CPU time, query counts, diff size, steps in the user path. Scratch orgs make the prototypes disposable, which is what makes honest comparison cheap.

## Proof it applied

At least two structurally different candidates existed, the comparison criteria were named before the build, and the record shows why the winner won.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
