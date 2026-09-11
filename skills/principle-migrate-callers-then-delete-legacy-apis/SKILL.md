---
name: principle-migrate-callers-then-delete-legacy-apis
description: "Apply when introducing a new internal API or automation path while old callers still exist. Migrate callers and delete the old path in the same wave instead of preserving compatibility layers."
disable-model-invocation: true
---

# Migrate Callers Then Delete Legacy APIs

When a new API or automation path is the right design, migrate every caller and delete the old path in the same wave. Do not preserve compatibility layers because internal callers still exist. Keeping both paths creates dual-path complexity, slows cleanup, and makes the codebase append-only: every new reader must learn two ways to do one thing, and every future change must be made twice.

## When it applies

- Internal refactoring where no external consumer depends on the old contract.
- The project can absorb a coordinated breaking change.
- The new path is part of a simplification.

## The rules

1. **Inventory the callers.** All of them, found by search and by the dependency graph, not by memory.
2. **Migrate them to the new contract.**
3. **Delete the old path in the same wave.** Same PR or the immediately following one, with a date.
4. **Time-box any adapter.** A temporary bridge is exceptional, named, and has a deletion date; it is not architecture.
5. **Update the tests to the new contract.** Delete tests that only protected pre-refactor implementation details.

## Salesforce application notes

- **The caller inventory is a dependency problem, so use the platform's tools:** `MetadataComponentDependency` via the Tooling API and a source grep, exactly the **sf-blast-radius** method. "Who calls this class" on Salesforce includes flows, other Apex, and integrations outside the repo; check all three.
- **Automation migrations are the classic case:** moving a Process Builder process or workflow rule into a flow or trigger. Migrate the behavior, deactivate the old one, delete it in the same release. A deactivated-but-present process is a compatibility layer that somebody will reactivate by accident.
- **Global and public modifiers are the external contract.** `global` Apex in a managed package or a class consumed by an integration is a published API; this principle covers internal paths only, and external contracts get versioning instead.
- **Deleting metadata needs the destructive-changes path** and a validation pass; plan the deletion like a deploy, not an afterthought (see the **deploy** playbook).

## Gotchas and failure modes

- **Caller inventory from memory.** "I think only the trigger uses it" is how deletions break production. The inventory is a dependency query plus a source grep, every time.
- **Deletion without the blast-radius check.** On Salesforce the callers include flows, reports, and integrations outside the repo. Run **sf-blast-radius** before the delete.
- **The undated adapter.** A bridge with no deletion date is the old API with a new name. Time-box it or do not build it.

## Proof it applied

The old path is gone (not commented, not deactivated-but-present), the caller inventory was demonstrated by the dependency query plus grep, and the suite is green on the new contract only.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
