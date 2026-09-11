---
name: principle-redesign-from-first-principles
description: "Apply when integrating a new requirement into an existing design. Redesign as if the requirement had been a foundational assumption from day one, instead of bolting it on."
disable-model-invocation: true
---

# Redesign From First Principles

When a new requirement meets an existing design, do not bolt it on. Redesign as if the requirement had been there from the start. Bolt-ons compound: each one makes the next one uglier, until the design is nothing but additions. The redesign question keeps the shape honest.

## When it applies

Integrating a change into an existing design, especially when the change does not fit the current shape cleanly.

## The pattern

1. **Read all the affected files** and understand the current design as it is, including why it is that way (**sf-why**).
2. **Ask the question:** "If we were building this from scratch with the new requirement included, what would we build?"
3. **Propagate the answer through every reference:** types, schema, docs, examples, the Feature Map, tests.
4. **Think the whole redesign, deliver it incrementally.** The vision is whole; the commits are small (see **principle-outcome-oriented-execution**).

## Salesforce application notes

- **The order-of-execution version:** a new requirement ("also notify the warehouse on submit") bolted onto an object that already has a trigger, two flows, and a roll-up produces a third flow firing in an order nobody chose. The first-principles pass asks where notification belongs in the whole automation map, and the answer is often one consolidated after-save design instead of a third bolt-on.
- **The data model version:** a requirement that does not fit the objects ("we also need to track orders that are not tied to accounts") is usually the model telling you it was underspecified. Adding a nullable lookup is the bolt-on; redesigning the relationship is the first-principles move, and it is far cheaper before the data lands.
- **Distinction from attack-the-premise:** that principle questions a fact the design assumes when fixes keep failing. This one rebuilds the design around a new requirement. One is diagnosis, the other is integration.
- **sf-architect Phase E is this principle operationalized:** the signals that say the design is wrong, and the scrap-and-redesign loop.

## Gotchas and failure modes

- **Redesign as an excuse to rewrite.** The question is "what would we build with this requirement from day one", not "what would I enjoy rebuilding". If the current design absorbs the requirement cleanly, the answer is the current design.
- **Ignoring why the design exists.** A from-scratch answer that violates a constraint the current design encodes (see **sf-why**) is a regression with ambition.
- **Scope explosion.** The redesign is delivered incrementally. Using it to bundle ten other wishes into one change is how migrations die.

## Proof it applied

The from-scratch answer is written down, the delivered increments converge on it, and no bolt-on was chosen without the redesign being considered and consciously rejected with its reason.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
