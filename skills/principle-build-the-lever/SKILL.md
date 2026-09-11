---
name: principle-build-the-lever
description: "Apply to any non-trivial work: edits, migrations, analyses, checks. Build the tool that does it or proves it (script, Apex anonymous harness, SOQL analysis, generator) instead of working by hand. The tool is the artifact a reviewer can rerun."
disable-model-invocation: true
---

# Build the Lever

When the work is not trivial, build the tool that does it or proves it instead of doing it by hand. Two payoffs. Throughput: a script does the work the same way every time and reruns for free. Confidence: the tool is one artifact a reviewer can read and rerun. Hand-done work can only be re-verified by redoing it. A deterministic script turns "trust me" into "run this".

## When it applies

Any non-trivial work: a sweep of similar edits, a data analysis, a repeated verification, a migration. Skip it only when the task is a couple of obvious edits you can check at a glance. Note the bar is triviality, not repetition: a one-off still earns a lever when the lever is what makes the work checkable.

## The pattern

1. **Do the first unit by hand** to learn the recipe.
2. **Build the tool**, then prove it by rerunning it on that first unit and diffing against the hand-done version.
3. **Make it safe to rerun.** Idempotent tools get run; fragile ones get feared (see **principle-make-operations-idempotent**).
4. **Prefer the deterministic lever over fan-out.** If a script can process every unit in one pass, run it yourself instead of delegating hand-work.
5. **Commit the lever** when the work outlives the session.

Applying this principle produces a file. If you cited it and there is no script, harness, query, or generator in the diff, you did not apply it.

## Salesforce application notes

The sf CLI and Apex make most levers cheap:

- **Verification levers:** an anonymous Apex script (`sf apex run --file checks/verify-orders.apex`) that asserts the end state and prints PASS or FAIL. This is the standard shape of "the check a reviewer can rerun."
- **Analysis levers:** SOQL aggregate queries saved to files, or `sf data query --bulk` extracts analyzed locally. A census (see **principle-attack-the-premise**) is a saved query, not a remembered one.
- **Edit levers:** `sed` or a small script over `force-app` for metadata sweeps (renaming a field across flows and layouts), verified by `git diff` and a scratch-org deploy.
- **Data levers:** a Bulk API 2.0 upsert keyed on an external ID, which is inherently rerunnable.
- **Agent levers:** when fanning work out, the recipe, the verification contract, and the do-not-touch fences go in one written artifact the workers read, kept outside their write scope.

Build the smallest script that does or proves the job, never a framework (see **principle-laziness-protocol**).

## Gotchas and failure modes

- **A lever bigger than the job.** A framework to rename six fields is not a lever, it is a hobby. Build the smallest script that does or proves the work.
- **An unproven lever applied to everything.** Run it on the first hand-done unit and diff before letting it loose on the rest. A wrong lever makes 200 consistent mistakes.
- **Editing the contract mid-run.** If the lever's recipe changes halfway through a sweep, the earlier units were processed by a different tool. Re-run or fence the change.

## Proof it applied

The lever exists as a file, it was proven against the hand-done first unit, and the report points at the lever and its output rather than at hand work.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
