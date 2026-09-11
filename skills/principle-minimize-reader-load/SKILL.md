---
name: principle-minimize-reader-load
description: "Apply when reviewing or shaping code that is hard to trace. Count layers between question and answer, and hidden state the reader must hold; collapse one-caller wrappers and shrink mutable scope."
disable-model-invocation: true
---

# Minimize Reader Load

Maintainability is the work a reader must do to understand the code. Track two axes: layers to trace (how many indirections sit between the question and the answer) and state to hold (how much hidden or mutable context the reader must carry in their head). Code is read far more than it is written, and reader load, not line count or architecture diagrams, is what makes it expensive.

## When it applies

Reviewing or shaping any code that is hard to trace, and before adding any layer or piece of shared state.

## The pattern

1. **Collapse layers that cost more than they save.** Wrappers with one caller, adapters with no second implementation, speculative indirection: inline them.
2. **Demand that each layer compresses.** A layer that repeats the same methods and arguments upward adds load without hiding anything. Collapse pass-throughs.
3. **Shrink state scope.** Prefer returns over mutations, locals over fields, fields over static state. Derive values instead of syncing copies of them.
4. **Name the invariant once, at the boundary,** not in every consumer, so the reader learns it a single time.
5. **Before adding a layer or state, ask:** does this reduce reader load elsewhere by at least as much as it adds?

## The test

Can a new reader answer "where does X come from?" and "what can change X?" in under 30 seconds? If not, cut layers or cut state.

## Salesforce application notes

- **Static variables are the hidden state of Apex.** A static flag set in a trigger and read three classes away is invisible coupling, and it also behaves oddly across partial-success retries and test runs. Prefer parameters and return values; reach for statics deliberately and document the transaction scope they live in.
- **The trigger-handler-service-util stack** can be four layers that each add nothing. A trigger that calls one handler method that does the work is often the honest shape until a second caller exists (see **principle-laziness-protocol**).
- **Flows are read too.** A flow with forty elements and no descriptions is reader load in declarative form. Name elements for what they do ("Set Status to Submitted", not "Update Records 2"), write the flow description, and extract subflows only when the subflow has a second caller or a real boundary.
- **The Feature Map (**sf-create-verification-skill**) exists to cut reader load across the whole org:** the answer to "what runs on this object" in one file instead of a scavenger hunt.

## Gotchas and failure modes

- **Collapsing a load-bearing layer.** Some indirection exists for testability or isolation. Inline it and the tests get harder, which is reader load moving, not shrinking.
- **Deleting the wrong axis.** Flattening layers while leaving fifty static variables untouched optimizes the axis you can see, not the one that hurts.
- **Optimizing for the author.** The author already knows where everything is. The reader load that matters is the next person's.

## Proof it applied

A reader new to the change can answer the two test questions quickly, and the diff shows layers and state shrinking or holding, not growing.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
