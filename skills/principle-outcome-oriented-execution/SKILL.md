---
name: principle-outcome-oriented-execution
description: "Apply during planned rewrites and migrations with explicit phase boundaries. Converge on the target architecture; do not preserve smooth intermediate states with throwaway compatibility code."
disable-model-invocation: true
---

# Outcome-Oriented Execution

In a planned rewrite or migration, optimize for the intended end state, not for smooth intermediate states. Keeping every intermediate step fully stable breeds temporary compatibility code, and temporary code has a way of becoming permanent. Converge on the target architecture and prove correctness at explicit verification boundaries instead.

## When it applies

Planned rewrites and migrations with explicit phase boundaries: a trigger framework replacement, a data model migration, a move from Process Builder to flows.

## The rules

1. **Prioritize end-state integrity over transitional stability.** The target architecture is the deliverable, not the journey's comfort.
2. **Intermediate breakage is acceptable when planned, scoped, and reversible.** Declare in advance where it is acceptable, and keep it inside those fences.
3. **Keep high-signal checks running for the areas being touched,** so breakage is detected at the boundary where it was introduced.
4. **Run full verification at completion.** The plan ends with the whole suite green and the behavior driven, not with the last edit.

## Salesforce application notes

- **The classic case is automation migration.** Moving five Process Builder processes into flows: the comfortable path keeps both running "during transition" and produces double-firing automations and records updated twice. The outcome-oriented path migrates, deactivates, and deletes per object, with the verification boundary being that object's behavior driven and tested after each move.
- **Scratch orgs make planned breakage cheap.** The intermediate broken state lives in a scratch org where it harms no one, and the verified end state is what gets validated and deployed.
- **Time-box every shim.** A compatibility trigger that delegates to the new service during the migration gets a deletion phase with a date in the plan (see **principle-migrate-callers-then-delete-legacy-apis**). Undated shims are permanent.
- **The multi-phase-plan playbook carries this principle:** every phase names its verification, and the final phase's proof is the outcome sentence verified in a real org.

## Gotchas and failure modes

- **Breakage that leaks outside the fence.** Planned instability is scoped and declared. The moment it surprises a shared org or another team, it is an incident, not a phase.
- **Dogma about the end state.** Requirements move mid-migration. Re-derive the target when they do; converging on a stale outcome is waste with good posture.
- **Skipping the final verification.** The principle trades transitional stability for end-state proof. Without the full final verification, you kept the cost and dropped the benefit.

## Proof it applied

The plan named its target end state and its verification boundaries, intermediate states were declared and fenced, no undated compatibility code survives, and the final verification evidence exists.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
