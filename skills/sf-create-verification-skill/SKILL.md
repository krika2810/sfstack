---
name: sf-create-verification-skill
description: "Generate a project-local verification skill that drives a Salesforce org the way a user does - scratch org lifecycle, sf CLI control, UI driving, evidence capture. Use for /sf-create-verification-skill, 'make a verify skill for this project', or when the project has no scripted way to prove behavior."
disable-model-invocation: true
---

# Create a verification skill (Salesforce)

Every serious sfdx project needs a scripted way to prove behavior in a real org: spin up a scratch org, deploy, exercise the feature the way a user would, and capture evidence. This skill generates that as a project-local skill (`verify-<project>/`) tailored to the repo. You write the generator's output for the next agent, not for a human: it will be read cold, mid-task, by an agent that has never seen the project.

The good news a Salesforce project gets that a web app does not: the control CLI (sf) and the disposable environment (scratch orgs) already exist. Your job is wiring them together, per project.

## 1. Interview the repo, not the user

Answer these from `sfdx-project.json`, `config/`, `force-app/`, and existing scripts. Only ask the user what you cannot observe:

- **Surface:** what does a user actually touch? Lightning record pages, an LWC app, Experience Cloud, REST endpoints, scheduled batch jobs, platform events, Flows? A project can have several; pick the primary one and note the rest.
- **Launch:** how does a fresh environment come up? The exact sequence: `sf org create scratch --definition-file ... --alias ... --set-default`, then `sf project deploy start`, then any seed (`sf data import tree`, `sf apex run --file scripts/seed.apex`), then permission set assignment (`sf org assign permset`). Note duration - a launch that takes 20 minutes changes how agents should batch work.
- **Drive:** how can an agent interact programmatically? Data layer: `sf data query`, `sf data upsert`, `sf data create record`. Logic layer: `sf apex run` for anonymous Apex. UI layer: browser automation against a session URL from `sf org open --url-only --json` (frontdoor session) - Playwright if the repo has it, otherwise document manual-spot checks. API layer: `sf api request rest` for REST endpoints.
- **Observe:** what evidence can be captured? Test results (`sf apex run test --json`, test-run IDs), code coverage, debug logs (`sf apex tail log`, `sf apex get log` - note that logging requires an active trace flag; document how to set one), deployment IDs and reports, screenshots from UI drives, Limits-class telemetry printed by instrumented Apex.
- **Isolate:** scratch orgs are the isolation unit - one per run, always. Never double-drive a shared sandbox. Record the DevHub's daily scratch org cap (`sf limits api display` on the DevHub / org list) so agents budget creation.

If the project does not deploy cleanly to a fresh scratch org as-is, fix that first (or report it precisely) before generating; a skill written against a broken base teaches wrong steps.

## 2. Generate the skill

Write `verify-<project>/SKILL.md` with YAML frontmatter (`name: verify-<project>` and a `description` naming the project, primary surface, and when to reach for it) and these sections, each grounded in what the interview actually found - no placeholders:

- **Launch:** the exact command sequence that produces a ready org, and how to tell it is ready (deployment succeeded, seed script exit code, permset assigned). Include teardown (`sf org delete scratch`). If the project supports scratch org snapshots or a pool, document the fast path.
- **Doctor:** one read-only check answering "is this org worth driving?" - `sf org display --json` (org alive, not expired), right source deployed (deployment status), auth valid, and limits headroom (`sf limits api display`). An agent runs this first whenever anything looks off.
- **Drive:** the real commands for this project's features, grouped by job: navigation (open specific Lightning pages via frontdoor URLs), interaction (record creates/updates via sf data, anonymous Apex entry points, UI clicks via the browser harness), inspection (queries, `sf sobject describe`), streaming (debug log tailing). Prefer stable handles: API names, record IDs from seed data, ARIA labels in Lightning.
- **Evidence:** what to capture for a proof and where it goes. Proof standards: exercise the real user path, not test-only shortcuts; capture the action and the resulting state (query the record after the UI action); verify side effects (child records, platform events, emails via the email log); check governor-limit telemetry for anything that will run at volume. Deployment IDs and test-run IDs go in the evidence bundle - they are how a reviewer re-runs your proof.
- **Cleanup:** delete the scratch org (`sf org delete scratch --no-prompt`). Evidence survives teardown in a named directory; cleanup removes orgs and scratch state, never proof.
- **Helpers:** any script the skill ships (seed, trace-flag setup, screenshot driver) is executable and its invocation is shown in the skill body.

## 3. Seed the Feature Map

Create `verify-<project>/features/README.md` plus one file per major user-facing feature (aim for the top 3-5 to start, discovered from custom tabs, apps, LWC components, Flows, and key objects). Follow the shape in `references/feature-map-example/`. Each file answers, from the user's point of view: what the feature is, how to reach it in the Lightning UI, how to drive it with the commands from section 2, what observable end state proves it works, and the gotchas (record-type gating, permission requirements, order-of-execution surprises, async lag). The four H2s are `Sub-features`, `How to get to it (user POV)`, `Driving it with sf + browser`, and `Gotchas`.

## 4. Prove the generated skill before handing it over

Run its own instructions end to end once: launch, doctor, drive ONE mapped feature, capture evidence, clean up. After cleanup, confirm the evidence still exists at the named location. Fix what fails, and run the generated cleanup after every failed iteration too, so broken attempts do not strand scratch orgs (they count against the DevHub daily cap). A generated skill that was never executed is a draft, not a deliverable.

## 5. Offer the maintenance loop

Point the user at `/sf-maintain-verification-skill` for keeping the map honest as the project's source changes. Suggest a cadence only if they ask.
