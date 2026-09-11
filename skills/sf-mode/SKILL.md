---
name: sf-mode
description: "Rigorous engineering mode for Salesforce work: governor-limit discipline, source-driven development, verify everything in a real org. Use for /sf-mode, 'sf mode', or any non-trivial Apex, Flow, LWC, integration, or data task on the Salesforce platform. Sticky: stays on across turns until told otherwise."
---

# sf-mode

sf-mode is the entry point for all serious Salesforce work in sfstack. It does three things. First, it reads your task and matches it to a playbook from the `playbooks/` folder. Second, it opens a todo list whose first items are that playbook's steps, copied in word for word, so nothing gets skipped quietly. Third, it keeps the working rules below switched on for every turn that follows, not just the first one.

sf-mode is sticky. Once it is on, it stays on across turns until the user says to stop. A casual question in the middle of a big task does not switch it off.

## When to use it

- Any non-trivial Apex, Flow, LWC, integration, or data task on Salesforce.
- Any task where rigor matters more than speed: production-bound work, shared orgs, anything with a blast radius.
- Skip it for casual questions and one-line lookups; it is a working mode, not a tax on conversation.

## The working rules

These six rules decide how every task runs. When a rule changes a decision, say which rule and what it changed in your reply.

1. **Verification is everything.** Never call work done because Apex compiled, a deploy succeeded, or the code looks right. Work is done when the behavior ran in a real org and you kept the evidence: a test-run ID, a deployment ID, a debug log excerpt, or a screenshot. Tests passing is how work stays done, not proof that it is done.
2. **Governor limits are uncatchable.** When Apex breaches a limit the transaction dies with `System.LimitException` and no try/catch can catch it. So limits are a design constraint, not an error to handle. Bulkify every data operation, write selective queries, move volume to async, and probe with the `Limits` class when you are unsure how close you are.
3. **Source-driven, never org-based.** All work happens in SFDX source and moves to orgs with the sf CLI: `sf project deploy start`, `sf project retrieve start`. Nothing gets edited by hand in a production org's Setup menu. Salesforce Well-Architected names org-based development an anti-pattern.
4. **Metadata is code.** Flows, validation rules, custom objects, and permission sets get the same rigor as Apex: version control, review, tests where they exist, and a blast-radius check before change. A Flow that updates records can blow a governor limit exactly like Apex can.
5. **Access is explicit.** CRUD, field-level security, and sharing are chosen deliberately and stated in the work: `with sharing` on the class, `Security.stripInaccessible()` or `WITH USER_MODE` on the data path. Silent privilege escalation is a defect even when it makes the demo pass.
6. **Never block on the human for reversible work.** Scratch orgs make almost everything reversible, so proceed and present evidence instead of asking permission. Reserve confirmation for the genuinely irreversible: production deployments, data deletion, and anything that touches customer-facing state.

## Matching the task to a playbook

Read the task, pick the playbook that fits, open its file from `playbooks/`, and copy its steps into the todo list verbatim. A step you deliberately skip stays in the list with a one-line `skip: <reason>`.

| playbook | use it for |
|---|---|
| investigation | A read-only question: how does this automation work, why was it built this way, are we sure about this behavior |
| bug-fix | A defect to reproduce in a scratch org, root-cause with the order of execution in mind, fix, and verify |
| feature | New or changed behavior, built from a named data shape |
| perf-issue | Measured slowness to trace and improve against a baseline: query plans, selectivity, async, caching |
| deploy | Shipping to a sandbox or production: validate first, then deploy, keep the evidence |
| data-migration | Loading or transforming large data volumes: Bulk API 2.0, batch strategy, skew rules |
| flow-change | Creating or changing Flows as the metadata they are: entry criteria, subflows, when to hand off to Apex |
| multi-phase-plan | Work that spans phases or stacked commits, where every task ends in a check |
| autonomous-run | A long task to drive to completion without stopping |
| opening-a-pr | The end of every other playbook: ordered commits and a reviewable briefing |

If no playbook fits, use the **sf-figure-it-out** skill to design a one-off playbook for the task.

## The principles index

Read the matching principle skill's SKILL.md in full when the work touches it. Name the principles that shaped a decision in your reply, and cite only principles you actually read this session.

Salesforce-native principles, written for this plugin:

- **principle-bulkify-by-default** - any data operation in Apex or Flow
- **principle-selective-queries** - writing or reviewing SOQL or SOSL
- **principle-respect-the-shared-runtime** - any Apex transaction design
- **principle-test-real-transactions** - writing or changing tests
- **principle-metadata-is-code** - touching Flows, validation rules, objects, or permissions
- **principle-enforce-access-explicitly** - any data read or write path
- **principle-async-for-volume** - anything beyond small data volumes
- **principle-no-hardcoded-ids** - reviewing Apex or Flow

Upstream engineering principles from Lauren Tan's pstack, vendored in this plugin under `skills/` and expanded with Salesforce application notes (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`). Read the matching `principle-*/SKILL.md` like any other skill in this plugin; no separate pstack install is needed:

- **principle-attack-the-premise** - two fixes sharing one premise have failed; question the premise
- **principle-boundary-discipline** - wiring validation, error handling, or API adapters
- **principle-build-the-lever** - any non-trivial work; build the tool that does or proves it
- **principle-encode-lessons-in-structure** - you wrote the same instruction twice
- **principle-exhaust-the-design-space** - a novel design with no precedent
- **principle-experience-first** - product or behavior tradeoffs
- **principle-fix-root-causes** - debugging anything
- **principle-foundational-thinking** - before writing logic: data shapes and sequencing
- **principle-guard-the-context-window** - long sessions, big logs, wide reads
- **principle-laziness-protocol** - refactoring or tempted to add abstraction
- **principle-make-operations-idempotent** - retries, reruns, deployments
- **principle-migrate-callers-then-delete-legacy-apis** - replacing an old code path
- **principle-minimize-reader-load** - writing or reviewing for readability
- **principle-model-the-domain** - shaping objects, fields, and state
- **principle-never-block-on-the-human** - reversible work waiting on approval
- **principle-outcome-oriented-execution** - phased rewrites and migrations
- **principle-prove-it-works** - before declaring anything done
- **principle-redesign-from-first-principles** - the current design fights you
- **principle-separate-before-serializing-shared-state** - concurrency and shared state
- **principle-sequence-verifiable-units** - ordering any multi-step plan
- **principle-subtract-before-you-add** - before adding anything
- **principle-test-behavior-not-implementation** - writing or changing tests
- **principle-type-system-discipline** - schema, types, and validation design

## Routing to the other skills

- Non-trivial change or "are we sure?" question -> the **sf-how** skill to trace how the org actually behaves.
- Question about intent or history -> the **sf-why** skill.
- New design or a change that crosses object or class boundaries -> the **sf-architect** skill before writing code.
- Contested design -> **sf-arena** (competing designs in separate scratch orgs) and then **sf-interrogate** (multi-lens adversarial review) before shipping.
- Parallel work across metadata slices -> the **sf-swarm** skill.
- Change whose impact is unclear -> the **sf-blast-radius** skill for the dependency graph.
- Any deploy -> the **sf-deploy** skill.
- Any test work -> the **sf-tdd** skill.
- Docs, READMEs, PR descriptions -> the **sf-technical-writing** skill; any prose surface -> the **sf-unslop** skill.
- Long or unattended work -> a decision trail via the **sf-show-me-your-work** skill.
- Lost context -> the **sf-recall** skill; finished task -> the **sf-reflect** skill.

## Autonomy

Reversible work proceeds without asking: scratch orgs, deploys to scratch orgs and sandboxes, data loads into scratch orgs, code changes on a branch. Always pause for: production deployments, destructive data operations in anything shared, and messages sent to people.

## Reply style

Write replies in plain English with no filler. Lead with what changed and the proof. Name the org alias, the deployment ID, and the test-run ID when they exist. If something is a guess, label it a guess in the same sentence. Never claim done without evidence attached.

## Gotchas and failure modes

- **Playbook theater.** Copying the steps and then freelancing anyway. The todo list is the contract; deviations get a `skip:` line with a reason or they do not happen.
- **Mode amnesia.** sf-mode is sticky, but a new session is not a turn. Re-enter sf-mode explicitly when resuming work in a fresh session (**sf-recall** rebuilds the context first).
- **Rigor inflation on trivial asks.** A one-field label change does not need the arena. Match the rigor to the blast radius; the playbooks scale down as well as up.

## Proof it worked

An sf-mode session ends the way its playbooks end: every claimed change carries evidence (a test-run ID, a deployment ID, a log excerpt, or a screenshot), every skipped playbook step has its one-line reason, and the reply names the orgs and IDs a reviewer can check. If the work finished with nothing a reviewer can rerun, sf-mode was not actually on.
