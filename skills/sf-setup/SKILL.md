---
name: sf-setup
description: "One-time per-project setup for sfstack: verify sf CLI, DevHub auth, scratch org definition, and project shape. Use for /sf-setup or 'set up sfstack'."
---

# sf-setup

Run once per sfdx project, before `/sf-create-verification-skill`. Verify the environment, then write what you find into the project's agent instructions file (CLAUDE.md, .cursorrules, or equivalent) so later sessions start warm.

## Checks (run, don't ask)

1. **sf CLI present and current**: `sf version`. If missing or ancient, say so and stop - everything else depends on it.
2. **DevHub authorized**: `sf org list --json` - a DevHub must appear and be non-expired. If none, give the exact command (`sf org login web --set-default-dev-hub --alias devhub`) and wait.
3. **Project shape**: read `sfdx-project.json` - package directories, namespace, sourceApiVersion. Note whether the project uses unlocked packages.
4. **Scratch org definition**: find `config/project-scratch-def.json` (or equivalent). Record edition, features, and settings. If missing, offer to draft one matched to the project's metadata (features actually used).
5. **Test baseline**: run the existing Apex test suite in a scratch org once (`sf apex run test --code-coverage --result-format human`) and record the pass rate and org-wide coverage. This is the floor you compare every later run against.
6. **Existing harnesses**: look for CI config (`.github/workflows`, etc.), existing scripts in `scripts/`, Jest config for LWC (`jest.config.js`, `sfdx-lwc-jest`), and any UTAM or Playwright setup. The verification skill must build on what exists, not parallel to it.

## Write the project config

Append a short `## sfstack` section to the project's agent instructions file: DevHub alias, default scratch def path, package dirs, test baseline (date, pass rate, coverage), and the verification skill's name once created. Keep it under 15 lines - it loads every session.
