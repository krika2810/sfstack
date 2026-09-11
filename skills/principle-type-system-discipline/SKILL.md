---
name: principle-type-system-discipline
description: "Apply when designing types, signatures, or schema in Apex or any typed layer. Make illegal states unrepresentable, distinguish semantically different primitives, parse external data at boundaries, and let the compiler or the platform catch what it can."
disable-model-invocation: true
---

# Type System Discipline

The type checker is a proof assistant: use it to eliminate impossible states and unhandled cases at compile time instead of discovering them at runtime. Prefer defining errors out of existence over proliferating handlers. A case the types let you ignore becomes the production incident the compiler could have stopped.

## When it applies

Designing types, reviewing a signature, shaping schema, or writing in any typed layer of the stack.

## The patterns

1. **Make illegal states unrepresentable.** Model variants as explicit types, not bags of optional fields where contradictory combinations compile. If a bug forces the question "can this combination actually happen?", the type is too loose.
2. **Types are constructions, not restrictions.** Build the type up from the values you want instead of carving them out of a looser type with runtime checks.
3. **Distinguish semantically different primitives.** Two strings that mean different things (an order number and an account number) should not be interchangeable.
4. **External data is untyped until parsed.** JSON, API payloads, CLI input, config: a parse function at every boundary turns unstructured input into the typed model (see **principle-boundary-discipline**).
5. **Do not lie to the type system.** Casts and assertions that bypass the compiler are latent runtime crashes. If the compiler cannot prove a fact, validate at the boundary or accept the hazard consciously.
6. **Exhaust the variants.** When you switch on a type or status, the structure should fail loudly when a new variant arrives unhandled.
7. **Derive from authoritative schemas.** When an API spec, a schema, or a metadata definition owns the shape, derive from it instead of hand-rolling a parallel type (see **principle-encode-lessons-in-structure**).

## Salesforce application notes

Apex is nominally typed and the platform adds its own type layer, so this principle has platform-specific teeth:

- **The schema is the strongest type system you have.** A picklist is an enum the platform enforces. A required field is a non-null guarantee. A validation rule is a platform-checked invariant. Putting a constraint in the schema beats asserting it in five Apex classes, because the schema covers every writer: UI, API, integrations, data loads.
- **Record types and custom metadata model variants.** An order that can be internal or external with different required fields is two record types, not one object with conditionally-required fields checked in triggers.
- **The bag-of-optional-fields anti-pattern is common in Apex wrappers:** an integration response class with twelve nullable fields where only certain combinations are valid. Split it into typed variants with a parse method that rejects illegal combinations at the boundary.
- **Switch on picklist values with an else that throws.** When a new picklist value arrives, the throw is the alarm; a silent fall-through is the incident.
- **SObject types are branded primitives already:** passing an `Id` is safer than passing a `String` that might be an Id, and `Id.valueOf` validation belongs at the boundary. Use the specific SObject type (`Order__c`) over generic `SObject` wherever the code actually knows the type.
- **`SObject` generics and `Map<String, Object>` are the "any" of Apex.** They have their place in framework code; everywhere else they are lies to the type system.

## The tests

- "Can I write a comment explaining when this combination of fields is valid?" Then the type is too loose; split it.
- "Do two arguments share a primitive type but mean different things?" Distinguish them.
- "If a new picklist value or variant arrives next month, does anything fail loudly?" If not, make it.
- "Is this type duplicating a shape the schema or an API spec already owns?" Derive instead.

## Gotchas and failure modes

- **Precision past the point of use.** Types exist to make wrong code fail, not to describe data perfectly. If nothing would break, the plain type is fine.
- **Framework code typed like app code.** The reusable layer genuinely needs generics; the app genuinely does not. Confusing the two produces either needless `SObject` soup or needless rigidity.
- **Schema constraints that fight legitimate writers.** A validation rule that blocks the integration user is a type system lie of its own: it says the state is illegal while the business says it is required. Constraints must cover every writer or they get bypassed.

## Proof it applied

Illegal states fail to compile or fail at the boundary parse, new variants raise alarms instead of silent fall-throughs, and no wrapper class admits combinations the domain forbids.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
