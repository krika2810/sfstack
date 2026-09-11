---
name: principle-boundary-discipline
description: "Apply when wiring validation, error handling, or framework adapters. Concentrate guards at system boundaries (API versions, callouts, LWC wire adapters, Trigger context); trust internal types and keep business logic in pure, testable classes."
disable-model-invocation: true
---

# Boundary Discipline

Validation, type narrowing, and error handling belong at the edges of the system, where outside data comes in. Inside the system, code trusts its types and gets on with the work. Business logic lives in pure, testable units; the framework wiring around them is thin and mechanical. Scattered validation is noise that gives a false sense of safety, and logic trapped inside framework glue cannot be tested without the framework.

## When it applies

Wiring validation, error handling, or adapters: REST resources, callout clients, LWC controllers, platform-event subscribers, trigger entry points, config parsing.

## The pattern

- **At the boundary:** validate everything. Parse raw input into typed domain values. Return clean errors. Handle the hostile cases, because the boundary is where hostile data lives.
- **Inside the system:** no re-validation. Typed data flows in, errors propagate out, business logic is pure and trusts the types.
- **Across the boundary:** expose domain concepts, not the boundary's private representation. Do not leak transport types (HTTP wrappers, JSON maps, wire shapes) into the business layer.

## Salesforce application notes

The boundaries on Salesforce are specific:

- **The trigger context is a boundary.** Validate and normalize `Trigger.new` at the handler's entry (null checks on relationship fields the trigger context does not populate, filtering to relevant records), then pass clean domain lists into services. Services never read `Trigger.new` directly.
- **`@AuraEnabled` controllers are a boundary.** Everything from the browser is untrusted: re-check access (`WITH USER_MODE`, `stripInaccessible`), validate IDs and parameters, and never trust client-computed values. The controller adapts; the service behind it stays pure.
- **Callout clients are a boundary.** Parse JSON responses into typed wrapper classes at the client, throw a domain exception on bad payloads, and let the rest of the codebase see only the typed result. Deserialization scattered through business logic is the anti-pattern.
- **`RESTResource` methods** get untyped `RestRequest` input. Parse and validate in the resource method; hand typed values inward.
- **Named credentials, custom metadata, and custom labels** are config boundaries: read them in one config class that fails loudly on missing values, not in fifteen places that each assume they exist.

## The tests

- "Is this data crossing a system boundary right now?" If not, the validation is redundant; delete it.
- "Can this be a pure method the shell just calls?" If yes, extract it and test it without the framework.

## Proof it applied

Every entry point validates and parses before logic runs, no business class imports trigger context or HTTP types, and the business logic has unit tests that need no framework setup.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
