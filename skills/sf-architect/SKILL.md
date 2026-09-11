---
name: sf-architect
description: "Design before implementing, the Salesforce way: ground in the real schema and automation map, sketch the data model and sharing model first, compare competing designs, implement against the sketch, scrap when the signals say the design is wrong. Use for /sf-architect, 'design this', or any change that crosses an object or class boundary."
---

# sf-architect

Jumping straight to Apex locks in the wrong shape. sf-architect forces the design conversation to happen first, in the order Salesforce demands it: data model and sharing model before logic, automation placement before code. It runs as five phases, and each phase has a visible output.

## When to use it

- A new feature that touches more than one object or crosses a class boundary.
- A redesign of existing automation ("this trigger has become a monster").
- Any time you catch yourself writing the third special-case branch.

## The phases

### Phase A: Ground

Build a real model of what exists. Run **sf-how** over every object and automation the design touches. Produce the Feature Map view: what runs on each object, in which order-of-execution phase. If the design changes ownership or layering, also run **sf-why** so the reasons behind the current shape become constraints instead of surprises. Skip this phase only for genuinely greenfield work.

### Phase B: Sketch

Design it twice. Produce at least two structurally different candidates, not two point-fixes of one shape. For each candidate, write down:

1. **The data model.** Objects, fields, relationships (lookup vs master-detail), external IDs. On Salesforce this decision is nearly permanent: changing a master-detail to a lookup later is a migration.
2. **The sharing model.** Organization-wide defaults, role hierarchy effects, sharing rules, Apex managed sharing if needed. Sharing is architecture, not an afterthought; the wrong model cannot be patched with Apex later.
3. **The automation placement.** Which behavior lives in a before-save flow, an after-save flow, a trigger, or a queueable. State the order-of-execution consequences of each placement.
4. **The async boundary.** What runs synchronously (user is waiting) versus asynchronously (volume or callouts). Name the governor limits each candidate approaches at expected volume.

Run **sf-arena** when the candidates are close enough that evidence should decide: each design gets its own scratch org, and the comparison is measured, not argued.

### Phase C: Agree (opt-in)

Default: proceed to implementation with the chosen sketch, no human checkpoint. If the invoker asked for a checkpoint ("show me before you build"), surface the sketch and pause. Either way, human pushback on the shape is Phase A evidence: re-ground and re-sketch.

### Phase D: Implement against the sketch

The sketch is the contract. Deviations are signal, not friction to absorb: if the implementation needs a field the sketch did not anticipate, ask whether the sketch was wrong, the requirement was missed, or the code is overreaching. Ship the schema commit first, then logic, per **principle-foundational-thinking**.

### Phase E: Scrap when the design is wrong

A pattern of friction means the design is wrong, not the implementation. Salesforce-specific tells:

- Recursion guards appearing in multiple triggers on the same object.
- Automation that needs "run once per transaction" static flags to behave.
- SOQL that keeps needing one more relationship subquery because the data model split what belongs together.
- Sharing code that exists to undo what the OWD and role hierarchy keep re-doing.
- Every new requirement landing as a new branch in one giant handler.

When you scrap: re-run sf-how on what got built, redesign as if the new constraints were day-one assumptions (**principle-redesign-from-first-principles**), subtract before adding (**principle-subtract-before-you-add**), and return to Phase B.

## Salesforce details worth knowing

- **Master-detail vs lookup** decides cascade delete, reparenting, roll-up summaries, and sharing inheritance. Get it right in the sketch; it is the most expensive decision to reverse.
- **Before-save flows are the cheap fast lane** for same-record updates: they run before triggers and avoid a DML round trip. After-save flows and triggers are for work on other records. A design that puts same-record updates in an after trigger pays an extra save.
- **The order of execution is the design's runtime.** A sketch that does not say where each behavior runs in the sequence is not finished. Reference: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_order_of_execution.htm

## Gotchas and failure modes

- **Sketching around the platform.** Designs that fight the platform (custom locking, hand-rolled queues, shadow objects duplicating standard ones) lose to designs that use it. Prefer the platform feature over the custom one when it fits.
- **One-candidate design.** A single sketch is a rationalization, not a design. Phase B requires two structurally different candidates.
- **Sharing as a Phase D surprise.** If "who can see these records" is unanswered at implementation, the design was never done.

## Proof it worked

The design package exists: data model, sharing model, automation placement, async boundary, and (when arena ran) the measured comparison. Implementation deviations from the sketch are named with their reasons. The shipped result matches the sketch or the sketch was deliberately revised, and either way the record shows which.
