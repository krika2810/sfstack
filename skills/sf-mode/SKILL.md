---
name: sf-mode
description: "Rigorous engineering mode for Salesforce work: governor-limit discipline, source-driven development, verify-everything. Use for /sf-mode, 'sf mode', or any non-trivial Apex, Flow, LWC, integration, or data task on the Salesforce platform. Sticky: stays on across turns until told otherwise."
---

# sf-mode

The Salesforce conversion of pstack's `poteto-mode`. Read the task, match it to a playbook from `playbooks/`, open a todo list whose first items are the playbook's steps copied in verbatim, and route to the other skills as steps fire. Sticky: once entered, stay in sf-mode across turns until the user opts out.

## Prime directives

1. **Verification is all you need.** Never declare done on a compile or a hunch. A change is verified when it ran in a real org and the evidence is captured: test-run ID, deployment ID, debug log excerpt, or screenshot. Tests alone are not sufficient verification - run the behavior.
2. **Governor limits are uncatchable.** A breached limit kills the transaction with `System.LimitException`; no try/catch recovers it. Design so limits are never approached: bulkify by default, selective queries only, async for volume. Probe with the `Limits` class when unsure.
3. **Source-driven, never org-based.** All work happens in source (SFDX format), deployed to scratch orgs or sandboxes via the sf CLI. Never edit directly in a production org's Setup. Well-Architected names org-based development an anti-pattern.
4. **Metadata is code.** Flows, validation rules, objects, and permission sets get the same rigor as Apex: review, tests, version control, blast-radius checks before change.
5. **Access is explicit.** CRUD/FLS and sharing are chosen deliberately and stated in the work: `with sharing`, `Security.stripInaccessible()`, `WITH USER_MODE` where appropriate.
6. **Never block on the human for reversible work.** Scratch orgs make almost everything reversible. Proceed, present evidence, course-correct after. Reserve confirmation for production deployments and destructive changes.

## Principles index

Read the matching principle skill when the work touches it.

Salesforce-native (in this plugin):
- principle-bulkify-by-default - any data operation
- principle-selective-queries - writing or reviewing SOQL/SOSL
- principle-respect-the-shared-runtime - any Apex transaction design
- principle-test-real-transactions - writing or changing tests
- principle-metadata-is-code - touching flows, validation rules, objects, perms
- principle-enforce-access-explicitly - any data read/write path
- principle-async-for-volume - anything over small data volumes
- principle-no-hardcoded-ids - reviewing Apex or Flow

Upstream pstack principles, vendored in this plugin under `skills/` (verbatim from cursor/plugins @ f5bdd68, MIT, (c) Lauren Tan - see NOTICE.pstack). Read the matching `principle-*/SKILL.md` like any other principle skill in this plugin; no separate pstack install is needed:
- principle-attack-the-premise - before accepting the task as framed
- principle-boundary-discipline - keeping changes inside their layer
- principle-build-the-lever - when a tool or skill beats repeated effort
- principle-encode-lessons-in-structure - after a mistake is understood
- principle-exhaust-the-design-space - before settling on a design
- principle-experience-first - choosing behavior over internals
- principle-fix-root-causes - when a symptom fix tempts you
- principle-foundational-thinking - reasoning from platform first principles
- principle-guard-the-context-window - long sessions, big logs, wide reads
- principle-laziness-protocol - refactoring or tempted to add abstraction
- principle-make-operations-idempotent - retries, reruns, deployments
- principle-migrate-callers-then-delete-legacy-apis - removing old paths
- principle-minimize-reader-load - writing or renaming for readability
- principle-model-the-domain - shaping objects and fields
- principle-never-block-on-the-human - reversible work waiting on approval
- principle-outcome-oriented-execution - framing work by the outcome
- principle-prove-it-works - any claim of done
- principle-redesign-from-first-principles - when the current design fights you
- principle-separate-before-serializing-shared-state - concurrency and shared state
- principle-sequence-verifiable-units - ordering a plan
- principle-subtract-before-you-add - before adding anything
- principle-test-behavior-not-implementation - writing or changing tests
- principle-type-system-discipline - schema, types, and validation

## Playbooks

Load the matching playbook from `playbooks/` and run its steps:

| playbook | for |
|---|---|
| investigation | a read-only question: how does this automation work, why was it built this way |
| bug-fix | reproduce in a scratch org first, root-cause (order of execution aware), fix, verify |
| feature | new or changed behavior, from a named data shape |
| perf-issue | measured slowness: query plan, selectivity, async, Platform Cache, before/after evidence |
| deploy | validate-first shipping: check-only deploy + tests, then deploy, evidence kept |
| data-migration | Bulk API 2.0 loads, skew rules, off-peak, batch strategy |
| flow-change | flows are metadata: entry criteria, subflows, hand off to Apex at volume |
| multi-phase-plan | tactical plan where every task is structured around proof |
| autonomous-run | long unattended runs: scratch org budget, decision trail, checkpoints |
| opening-a-pr | small ordered commits, briefing body, validation evidence attached |

## Reply style

Unslopped, framed for the reader. Lead with what changed and the proof. Name orgs, deployment IDs, and test-run IDs when they exist.
