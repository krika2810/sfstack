---
name: sf-setup
description: "Verify the Salesforce toolchain and write project config: sf CLI installed and current, DevHub authorized, default scratch org definition sane, formatter and test tooling present. Use for /sf-setup, first task in an sfdx project, or when CLI auth or org creation breaks."
---

# sf-setup

sf-setup checks that the Salesforce toolchain actually works before any real task starts, and writes down what it found so later skills can rely on it. Run it once per project, and again whenever org creation or authentication breaks.

## When to use it

- The first task in any sfdx project.
- `sf org create scratch` fails, auth expires, or the CLI behaves oddly.
- A new machine, a new CI runner, or a new teammate joins the project.

## The steps

1. **Check the CLI exists and is current.**
   ```bash
   sf version
   sf update
   ```
   The modern CLI is `sf` (v2). If only `sfdx` (v7) exists, install the current one; the commands in these skills use `sf` syntax.
2. **Check the DevHub.** Scratch orgs come from a DevHub. List authorized orgs and confirm one is flagged as the DevHub:
   ```bash
   sf org list --all
   ```
   If none exists, authorize one interactively (`sf org login web --set-default-dev-hub`) or with a JWT flow in CI. Never paste credentials or tokens into chat or files; use the CLI's own auth flows.
3. **Check the scratch org definition.** Read `config/project-scratch-def.json` (or whatever `sfdx-project.json` points at). Confirm it names an edition, and the features and settings the project actually needs. A definition copied from a template that enables nothing will produce orgs where half the project's metadata fails to deploy.
4. **Prove org creation works.** Create a throwaway scratch org and delete it:
   ```bash
   sf org create scratch --definition-file config/project-scratch-def.json --alias setupcheck --duration-days 1
   sf project deploy start --source-dir force-app --target-org setupcheck
   sf org delete scratch --target-org setupcheck --no-prompt
   ```
   This is the real test: it exercises the DevHub, the definition, and the deploy path in one shot.
5. **Check the supporting tooling.** If the project has LWC, confirm `npm install` has run and `npm run test:unit` executes (that is sfdx-lwc-jest under the hood). If the project uses Salesforce Code Analyzer, confirm `sf code-analyzer run --rule-selector Recommended` works (v5 plugin; the retired `sf scanner run` command is the old v4 line).
6. **Write the project config note.** Record, in the repo or the project's local skill file: the DevHub alias, the scratch definition path, the deploy command that works, and any org quirks discovered (enabled features, required permission set licenses). Later skills read this instead of rediscovering it.

## Gotchas and failure modes

- **Expired auth.** `sf org list` shows stale entries. Re-auth with `sf org login web --alias <name>` and re-run the failing command before debugging anything else.
- **DevHub caps.** DevHubs limit daily scratch org creations and active scratch orgs. If creation fails with a limit error, list and delete stale orgs (`sf org list --all`, `sf org delete scratch`) before blaming the definition.
- **Edition mismatch.** A project that needs Person Accounts or Shield features cannot deploy to a default Developer-edition scratch org. The definition must ask for what the source needs.
- **sfdx-project.json sanity.** The `packageDirectories` must include the path being deployed, and `sourceApiVersion` pins the API version new metadata gets. On API version 67.0 and later, Apex defaults change (user mode, implicit with sharing); know which side of that line the project sits: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_security_sharing_chapter.htm

## Proof it worked

The step-4 sequence ran clean: a scratch org was created, the project's source deployed to it without errors, and the org was deleted. The config note exists with the DevHub alias and the working deploy command written down.
