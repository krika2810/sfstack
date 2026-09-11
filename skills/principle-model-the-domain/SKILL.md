---
name: principle-model-the-domain
description: "Apply when writing stateful logic, or when code branches a lot or repeats a shape assumption across files. Encode the domain in a structure (a state machine, a typed model, a registry, the right collection) instead of scattered conditionals."
disable-model-invocation: true
---

# Model the Domain

Encode the real domain in a structure instead of scattering it across conditionals. Scattered booleans, repeated shape assumptions, and branching spread across files are accidental complexity. A structure that matches the domain makes invalid states unrepresentable and deletes branches. Choosing the structure at write time is cheap; recovering it later is a refactor that keeps getting deferred.

## When it applies

Writing stateful logic, or noticing that code branches a lot or repeats the same shape assumption in several files.

## Reach for structures like these

- **A state machine** instead of scattered booleans and lifecycle checks.
- **A typed model** instead of loose parameters or maps of strings passed between methods.
- **A map, registry, or lookup table** instead of branching spread across files.
- **A module organized around one body of domain knowledge** instead of one organized by execution order (load, validate, transform, save). Execution order is not ownership.
- **The right collection**: a queue, an index, a graph, a normalized set, where the access pattern calls for it.
- When nothing fits, work out what the code must never allow and how the data gets read, then find the structure that encodes exactly that.

Do not force an abstraction. If the current shape is clear, local, and unlikely to grow, boring code wins. Be skeptical of any abstraction that adds indirection without deleting branches, duplicated rules, or invalid states.

## Salesforce application notes

- **The data model is the first domain model.** Status fields with legal transitions, record types that distinguish genuinely different shapes, and junction objects that encode a many-to-many reality are domain modeling the platform enforces for you. A "type" field on an object that secretly splits it into three different things is a sign the model wants to be three record types or three objects.
- **The classic Apex case:** order status logic scattered as `if (o.Status__c == 'Draft' && o.Total__c > 0)` across four classes. The structure is a transition table (a map from current status to allowed next statuses, or a custom metadata type) that every class consults. New states then change one structure instead of four files.
- **Registries beat if-chains for routing.** Handler-per-record-type or strategy-per-integration keyed in a `Map<String, Handler>` replaces the else-if ladder that grows one branch per feature. The sign you skipped this principle is a new feature landing as one more branch in an existing if-else chain.
- **Do not rebuild the platform.** The domain model that already exists as custom objects and fields does not need a parallel class hierarchy mirroring it. Model what the objects cannot say: transitions, policies, calculations.

## Gotchas and failure modes

- **Forcing structure on clear boring code.** An abstraction over a shape that was already obvious is indirection without payoff. Boring wins when boring is clear.
- **Modeling what the schema already says.** A parallel class hierarchy mirroring the custom objects duplicates the domain instead of modeling it. Model the rules the objects cannot express.
- **A model that drifts.** The transition table nobody updates becomes a lie with a type signature. The structure owns the rule only while it stays current.

## Proof it applied

The domain rule lives in one structure, adding the next case means editing the structure rather than hunting branches, and invalid states are unrepresentable or fail loudly at the boundary.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
