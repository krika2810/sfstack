---
name: principle-laziness-protocol
description: "Apply when refactoring, sizing a diff, or tempted to add abstractions, layers, or signal threading. Bias to deletion and the smallest change that solves the problem."
disable-model-invocation: true
---

# Laziness Protocol

Aim for the most result with the least code and complexity. When asked to improve something, look first for what to remove. The best change is usually smaller than the one you were about to make.

## When it applies

Refactoring, sizing a diff, or any moment you feel the pull toward another abstraction, layer, or parameter threaded through five files.

## The rules

1. **Prefer deletion.** Before adding, ask what can go. Dead code, redundant guards, superseded automation, unused fields.
2. **Keep the call hierarchy flat.** If answering a question about the code means tracing through more than about three files or layers, flatten it. A rich interface that hides substantial work is not a deep chain; a pass-through is.
3. **Consolidate decisions.** The same choice made in several places belongs behind one source of truth, passed onward as a simple value.
4. **Minimize the diff.** The smallest change that solves the problem. Fewer lines beat elegant boilerplate.
5. **Question the threading.** If the task asks you to pass a new signal through types, triggers, services, and flows, stop and look for the direct path first.
6. **Sweat the small leaks.** Tiny pass-throughs, representation leaks, and duplicated choices compound into permanent coordination cost. Remove them while they are small.

## Salesforce application notes

- **The laziest Apex is often a flow, and the laziest automation is often a platform feature.** Before writing a trigger for a same-record default, ask whether a before-save flow or a default value does it. Before building a custom rollup, check whether a roll-up summary field does it. Deletion includes deleting code you were about to write.
- **Trigger frameworks are the usual over-abstraction.** A handler-router-service-util chain for a 20-line trigger is three files of coordination for one file of work. Add layers when the second trigger arrives, not in anticipation of it (see **principle-subtract-before-you-add**).
- **The test:** if a Salesforce developer would find this exhausting to maintain, it is a bad solution. Apex ages in orgs maintained by admins and consultants, not just developers; the maintenance reader is less patient than you think.

## Gotchas and failure modes

- **Deletion that removes needed flexibility.** Some complexity is load-bearing. If you cannot say what a layer does, that is an argument to understand it, not to delete it blind.
- **The smallest change that ignores the real shape.** A two-line patch on a design that is wrong is not laziness, it is denial. The protocol governs solutions to the actual problem.
- **Flat for flat's sake.** Collapsing a boundary that had two real callers recreates the duplication the layer existed to kill.

## Proof it applied

The diff is the smallest one that solves the problem, deletions are visible, and no layer exists without a second user of the layer.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
