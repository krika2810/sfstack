---
name: principle-sequence-verifiable-units
description: "Apply to multi-step work and to how you stack commits and PRs. Break work into small units that each end in a check, verify each before the next, and order delivery so the sequence proves itself to a reviewer."
disable-model-invocation: true
---

# Sequence Work into Verifiable Units

Order work as a sequence of small units, each ending in a state you can check, and do not advance until the current one is green. A break caught at the unit that caused it is cheap to localize. A break caught after a batch is buried under everything built on top of it. And a sequence ordered so each step's proof builds on the last turns "trust me" into "watch it go red, then green" for anyone reviewing later.

## When it applies

Multi-step work of any kind: sweeps of similar edits, migrations, stacked commits and PRs, phased plans.

## The pattern

1. **Pick the smallest unit that ends in a check.** One edit plus its test. One commit that stands alone. One object migrated plus its behavior driven.
2. **Verify before advancing.** Red to green per unit, never deferred to a final batch. Rebase onto clean trunk first so every check measures against the real baseline.
3. **Order the units so the sequence argues for itself.** The canonical shapes: failing test before fix, subtraction before reshape, baseline capture before treatment, scaffold before feature. Each commit lands on its own and the stack reads as an argument.

## Salesforce application notes

- **The deploy gives you the unit check for free.** Each unit deploys to a scratch org and runs its tests: `sf project deploy start` plus `sf apex run test --tests <the unit's tests>`. Green deploy plus green unit tests is the gate between units.
- **Data work:** one object per unit, counts verified after each (source count equals target count), never "load everything and check at the end".
- **The multi-phase-plan playbook is this principle as a plan shape:** every phase names its verification, and no phase starts until the previous phase's evidence exists.
- **The PR stack reads in platform order:** schema commit, then logic, then tests green, then the failing-test-before-fix order for bug work. A reviewer replaying the stack watches the system go red and then green for the right reasons.

## Gotchas and failure modes

- **Units too small to be worth their check.** If verification overhead exceeds progress, batch up one level. The unit is the smallest change that ends in a meaningful check, not the smallest change possible.
- **Advancing on yellow.** "Mostly green, probably flaky" is red. A unit is done when its check is clean or its failure is understood and accepted in writing.
- **The risky unit scheduled last.** Ordering for a pretty narrative instead of for risk puts the likely failure under the most built-on base. Risk first (see the **multi-phase-plan** playbook).

## Proof it applied

Every unit has its own check with its own evidence (test-run ID, deployment ID, count comparison), and the commit or PR order lets a reviewer follow red to green per unit.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
