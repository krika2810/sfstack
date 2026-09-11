---
name: sf-architect
description: "Design Salesforce solutions through a disciplined five-phase loop: ground in the real schema and automations, sketch data model and sharing model first, arena competing designs, implement against the sketch, scrap when the signals say the design is wrong. Use for /sf-architect, 'architect this', or any design that crosses an object or transaction boundary."
---

# sf-architect

Architecture on Salesforce is different from general software: your "data structures" are objects and relationships, your "runtime" is a shared governed cloud, and your "module boundaries" are transaction boundaries and package boundaries. Five phases.

## Phase A: Ground the problem

Before designing anything, observe the real system:
- **Schema**: `sf sobject describe` the objects in scope; existing relationships, required fields, record types.
- **Automation map**: what already fires on these objects - triggers, record-triggered Flows, validation rules, rollups. The order of execution makes automations interact in ways no single file shows.
- **Dependencies**: run `/sf-blast-radius` on anything you plan to touch.
- **Data volumes**: query actual row counts and distribution. The 10,000-record skew rules (parent-child, ownership, lookup) from Well-Architected apply to your design, not just to migrations.
- **Limits context**: which of these paths run synchronous user transactions vs async? The answer changes the design.

Skip Phase A only for genuinely greenfield work with no surrounding system.

## Phase B: Sketch

Run `/sf-arena` with the design task and Phase A artifacts. Each candidate produces a design package: data model (objects, fields, relationships - this is the decision everything else hangs on), sharing model (OWD, roles, sharing rules, Apex-managed sharing), automation placement (Flow vs Apex vs both - base the choice on packageability and testability per Well-Architected, not on preference), transaction design (sync vs async, where the limits headroom comes from), and API surface (what other systems or LWCs call).

## Phase C: Agree (opt-in)

If the human wants a checkpoint, present the candidates with measured evidence, not prose arguments. If they push back on the shape, treat it as Phase A evidence: re-ground, re-run Phase B.

## Phase D: Implement against the sketch

Build in small, verifiable units. Each unit ends in a deploy to a scratch org and a proof. Deviations from the sketch are allowed - but write down why; two independent deviations of the same shape means the sketch is wrong, not the code.

## Phase E: Scrap when the architecture is wrong

Empirical scrap signals, Salesforce edition:
- Governor-limit workarounds appearing across unrelated code paths (collections chunked to dodge heap, queries split to dodge row limits).
- Repeated order-of-execution fights (recursion guards multiplying, workflow/Flow/Apex stepping on each other).
- Sharing bypasses becoming normal (`without sharing` spreading, FLS checks deleted to make tests pass).
- Types the platform cannot express forcing parallel bookkeeping fields.

One signal: investigate. Two or more of the same shape: return to Phase B and re-run the arena. Sunk cost is not an architecture.

## Outputs

A design package the implementer can execute without you, plus the evidence from every phase. `/sf-show-me-your-work` logs the decisions.
