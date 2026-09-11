# sfstack

**pstack, rebuilt for Salesforce.** Rigorous agent-engineering skills tuned to the realities of the Salesforce platform: governor limits, multi-tenancy, metadata-driven deployments, the 75% Apex test gate, scratch orgs, and the sf CLI.

sfstack is a conversion of [Lauren Tan](https://x.com/poteto)'s [pstack](https://github.com/cursor/plugins/tree/main/pstack) (MIT) - the skill set behind her ~2,000 PRs/month - re-designed from first principles for a managed, multi-tenant cloud. It is grounded in [Salesforce Well-Architected](https://architect.salesforce.com/well-architected/overview) (Trusted, Easy, Adaptable) and the official Salesforce developer documentation. Every Salesforce-specific rule in these skills cites the platform guidance it comes from.

Companion site with the full research, mapping, and interactive walkthroughs: **https://sfstack.vercel.app**

## Why a conversion instead of a copy

pstack's two hardest prerequisites already ship with the Salesforce platform:

1. **The control CLI exists.** pstack Part 1 teaches you to build a small CLI so agents can drive your app like a user. On Salesforce, the [sf CLI](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference.html) already launches (scratch orgs), drives (data, Apex), observes (debug logs, test results), and tears down. What is missing is the *verification skill* that wires it together per project - sfstack generates one.
2. **Disposable parallel environments exist.** pstack runs fleets of cloud agents; Salesforce has [scratch orgs](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_scratch_orgs.htm) - ephemeral, source-tracked orgs. Every parallel agent gets its own throwaway org. (Mind the DevHub daily creation caps - the platform governor-limits the harness too.)

What genuinely changes in a governed, multi-tenant runtime:

- **Governor limits are uncatchable.** Breach one and the transaction dies with `System.LimitException` - no try/catch saves you. Agents must *prevent* breaches by design: bulkify by default, write selective queries, favor async for volume. ([Execution Governors and Limits](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm))
- **Tests are a deployment gate, not a nicety.** Production deploys require 75% org-wide Apex coverage, and `sf project deploy validate` makes a check-only deploy + test run the pre-merge proof. ([Writing Tests](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_intro_writing_tests.htm), [project deploy validate](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_validate.html))
- **Metadata is the code.** Flows, validation rules, and objects are XML source. Well-Architected is explicit: don't use org-based development; adopt source-driven development with the Salesforce CLI. ([Resilient - ALM](https://architect.salesforce.com/docs/architect/well-architected/guide/resilient))
- **You can't run the app locally.** Verification happens against real orgs: deploy to a scratch org, run Apex tests, drive the Lightning UI through a browser session, capture debug logs as evidence.

## What's in the box

**52 skills** (21 core + 8 Salesforce-native principles + 23 upstream pstack principles vendored in) **+ 10 playbooks.** Self-contained: install sfstack alone and you get everything - no separate pstack install needed.

### Core skills

| skill | what it does |
|---|---|
| `/sf-mode` | Sticky entry mode. Routes your task to a playbook, applies the principles, keeps rigor on across turns. Start here. |
| `/sf-setup` | Verifies sf CLI, DevHub auth, default scratch org definition; writes project config. Run once per project. |
| `/sf-create-verification-skill` | Meta-skill: interviews your sfdx project and generates a project-local verification skill (launch / doctor / drive / evidence / cleanup) plus a metadata-aware Feature Map. The centerpiece. |
| `/sf-maintain-verification-skill` | Periodic pass keeping the verification skill and Feature Map honest as the org's source changes. |
| `/sf-architect` | Five-phase design loop: ground in the real schema and automations, sketch data model + sharing model first, arena competing designs, implement, scrap when the signals say the design is wrong. |
| `/sf-arena` | N competing designs, each in its own scratch org; evidence picks the winner. |
| `/sf-swarm` | N parallel workers across scratch orgs or metadata slices; one aggregated report. |
| `/sf-interrogate` | Multi-lens adversarial review: bulk safety, CRUD/FLS + sharing, SOQL selectivity, transaction/async design, test quality, and the Well-Architected pattern/anti-pattern checklists. |
| `/sf-blast-radius` | What else a change could break, via the metadata dependency graph (Tooling API) - with the safety fact proven by running tests, not asserted. |
| `/sf-tdd` | Failing Apex test first, then the fix. Real transaction semantics: `Test.startTest/stopTest`, data factories, no `SeeAllData`. |
| `/sf-deploy` | Validate-first shipping: check-only deploy with tests, then deploy, keep deployment/test-run IDs as evidence, post-deploy smoke check. |
| `/sf-how` | Traces runtime mechanics the Salesforce way: order of execution, automation map, schema, debug logs. |
| `/sf-why` | Investigates intent: git history, tickets, docs, setup audit trail. |
| `/sf-teach` | Runs how + why, weaves one plain explanation. |
| `/sf-recall` | Rebuilds your working context from history and the repo record. |
| `/sf-reflect` | Reviews a landed task and routes learnings into concrete skill edits. The self-improvement loop. |
| `/sf-show-me-your-work` | Reviewable decision trail (TSV): what, why, evidence - deployment IDs and test-run IDs included. |
| `/sf-figure-it-out` | Designs an auditable one-off playbook when none fits. |
| `/sf-technical-writing` | Diátaxis + Google developer style doc standard. |
| `/sf-unslop` | Cuts AI tells from writing. |
| `/sf-bro` | Restates the last message in plain language. |

### Salesforce-native principles

| principle | rule | grounded in |
|---|---|---|
| `principle-bulkify-by-default` | All data operations against collections; never single-record paths. Bulkification is necessary but not sufficient for large data volumes. | Well-Architected: Reliable - Performance |
| `principle-selective-queries` | Positive operators, no `LIKE` wildcards, only needed fields, no `LIMIT 1`, no `ALL ROWS`. | Well-Architected: Reliable - Performance |
| `principle-respect-the-shared-runtime` | Governor limits are uncatchable. Prevent by design; probe with the `Limits` class. | Apex Governor Limits |
| `principle-test-real-transactions` | `Test.startTest/stopTest`, data factories, no `SeeAllData`, assert outcomes. 75% is a floor, not a target; proof means it ran in a real org. | Apex Testing |
| `principle-metadata-is-code` | Flows, validation rules, objects live in version control. Source-driven, never org-based, development. | Well-Architected: Adaptable - Resilient |
| `principle-enforce-access-explicitly` | CRUD/FLS and sharing are deliberate: `with sharing`, `stripInaccessible`, no silent escalation. | Well-Architected: Trusted - Secure |
| `principle-async-for-volume` | Favor queueable/batch for volume; Bulk API 2.0 for LDV; mind latency tradeoffs. | Well-Architected: Reliable - Performance |
| `principle-no-hardcoded-ids` | No hard-coded record, user, or record-type IDs in Apex or Flow. | Well-Architected: Easy - Automated |

### Upstream pstack principles (vendored)

All 23 of Lauren Tan's engineering principles ship inside this repo under `skills/principle-*`, vendored from pstack @ f5bdd68 and expanded with Salesforce application notes (MIT, (c) 2026 Lauren Tan - see `NOTICE.pstack`). `/sf-mode` indexes them next to the Salesforce-native ones and reads them from the local files.

`attack-the-premise` · `boundary-discipline` · `build-the-lever` · `encode-lessons-in-structure` · `exhaust-the-design-space` · `experience-first` · `fix-root-causes` · `foundational-thinking` · `guard-the-context-window` · `laziness-protocol` · `make-operations-idempotent` · `migrate-callers-then-delete-legacy-apis` · `minimize-reader-load` · `model-the-domain` · `never-block-on-the-human` · `outcome-oriented-execution` · `prove-it-works` · `redesign-from-first-principles` · `separate-before-serializing-shared-state` · `sequence-verifiable-units` · `subtract-before-you-add` · `test-behavior-not-implementation` · `type-system-discipline`

### Playbooks (inside `/sf-mode`, loaded by task type)

`investigation` · `bug-fix` · `feature` · `perf-issue` · `deploy` · `data-migration` · `flow-change` · `multi-phase-plan` · `autonomous-run` · `opening-a-pr`

## Install

Works with any tool that supports the [Agent Skills spec](https://agentskills.io/).

```bash
# Claude Code, Cursor, Codex, OpenCode, ...
npx skills add krika2810/sfstack

# Claude Code plugin install (alternative)
/plugin marketplace add krika2810/sfstack
/plugin install sfstack@sfstack
```

Then, inside any sfdx project:

```
/sf-setup
/sf-create-verification-skill
/sf-mode <your first real task>
```

## Mapping to pstack

All 47 upstream pstack skills are accounted for: 20 core skills ported and adapted, 4 deliberately not ported (`typescript-best-practices` → replaced by the 8 Salesforce-native principles; `automate-me`, `make-bot-ui`, `no-comments` → tool-specific or work as-is upstream), and the 23 upstream engineering principles are vendored in this repo and expanded with Salesforce application notes (MIT, see `NOTICE.pstack`) - `/sf-mode` indexes them alongside the Salesforce-native ones and reads them from the local files. The 23 upstream playbooks are represented by 10 Salesforce playbooks covering the platform's actual task shapes. Full interactive mapping table: https://sfstack.vercel.app

## Prior art and credit

- **Lauren Tan (@poteto)** - [pstack](https://github.com/cursor/plugins/tree/main/pstack) and the two-part guide ([Part 1](https://x.com/poteto/status/2094457600259842065), [Part 2](https://x.com/poteto/status/2097732320606507506)). This project is a conversion of her ideas, not affiliated with her or Cursor. pstack ships MIT.
- **Salesforce** - [sf-skills](https://github.com/forcedotcom/sf-skills) (official curated agent skills: complementary - sfstack is the discipline/verification layer, sf-skills covers generation workflows), [Well-Architected](https://architect.salesforce.com/well-architected/overview), and the Salesforce developer documentation cited throughout.
