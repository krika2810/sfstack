---
name: sf-create-verification-skill
description: "Interview an sfdx project and generate a project-local verification skill: how to launch an org, health-check the toolchain, drive the app like a user, capture evidence, and clean up, plus a metadata-aware Feature Map. Use for /sf-create-verification-skill or when a project has no repeatable way to prove changes work."
---

# sf-create-verification-skill

This is the centerpiece of sfstack and the answer to pstack's core move, translated to Salesforce. pstack Part 1 has you build a small CLI so an agent can drive your app like a user. On Salesforce that CLI already exists: the sf CLI launches orgs, deploys source, runs Apex and tests, reads logs, and tears down. What is missing is the project-specific wiring: which definition file, which objects, which page proves the app works, what evidence to capture. This skill interviews the project and writes that wiring down as a project-local verification skill that every future task uses.

## When to use it

- A project has no repeatable way to prove a change works beyond "the tests passed".
- After sf-setup, once the toolchain is confirmed.
- Whenever the project's shape changes enough that the existing verification skill lies (then run sf-maintain-verification-skill instead).

## The steps

1. **Interview the project.** Read `sfdx-project.json`, the scratch definition, and the source tree. Answer these from the files, not by asking:
   - What are the package directories and the default one?
   - What are the core custom objects and their relationships?
   - What is the app's front door: a Lightning app page, an LWC on a record page, an Experience Cloud site, a REST endpoint, or pure automation?
   - What seed data does a meaningful drive need?
2. **Map the automations.** List every trigger, record-triggered flow, validation rule, and workflow remnant on the core objects. This becomes the Feature Map (see `references/feature-map-example.md` for the shape): object by object, what runs, in which order-of-execution phase, from which source file.
3. **Write the verification skill.** Generate `.claude/skills/verify/SKILL.md` (or the project's skills location) with five sections:
   - **Launch**: the exact `sf org create scratch` command with this project's definition file and alias convention, plus `sf project deploy start --source-dir <default package dir>`.
   - **Doctor**: the health checks from sf-setup condensed to this project: auth valid, DevHub reachable, definition deploys.
   - **Drive**: the concrete steps to exercise the app like a user. `sf org open --target-org <alias> --path /lightning/o/<Object>/list` for UI apps, `sf apex run --file` scripts for automation, `curl` against a REST resource for integration surfaces. Include the seed data load (`sf data import tree --plan ...`).
   - **Evidence**: what to capture every time - test-run ID from `sf apex run test`, deployment ID from deploy output, a debug log excerpt from `sf apex tail log`, and a screenshot or query result from the drive step.
   - **Cleanup**: `sf org delete scratch --target-org <alias> --no-prompt` and `sf org list --all` hygiene.
4. **Write the Feature Map** alongside the skill, shaped like the reference example. It is metadata-aware: it names the flow versions and trigger classes that run on each object, so a future change can see its neighbors before it edits.
5. **Smoke-test the generated skill.** Run its own launch, drive, and evidence steps once, end to end. A verification skill that has never run is a hypothesis.

## Salesforce details worth knowing

- The drive step must be honest about what "like a user" means per project. For a record-triggered automation, the drive is creating or editing a record and observing the outcome. For an LWC app, it is loading the page and clicking the path. For an integration, it is calling the endpoint.
- `sf org open --path` accepts Lightning URLs, so the skill can deep-link straight to the surface that proves the feature.
- Test-run IDs start with `707`, deployment IDs with `0Af`. Naming the ID shape in the generated skill teaches future runs what real evidence looks like.

## Gotchas and failure modes

- **Generating from aspiration.** Write down what the project is, not what the README claims. If the README says LWC app and the source says Visualforce, the drive step drives Visualforce.
- **Skipping the smoke test.** Step 5 is not optional. A drive path that 404s or a seed plan that fails on lookup order poisons every future verification.
- **Letting it go stale.** The skill describes the org's automation surface. That surface changes with every flow and trigger. Schedule sf-maintain-verification-skill when the drift shows.

## Proof it worked

The generated skill ran its own launch, drive, and evidence steps once, successfully, and the evidence from that run (deployment ID, test-run ID, drive output) is attached to the run that created it.
