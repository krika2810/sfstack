---
name: principle-prove-it-works
description: "Apply after completing a task, before declaring done. Verify against the real artifact in a real org (run the behavior, read the actual value, inspect the diff), not a proxy, a self-report, or 'it compiles'."
disable-model-invocation: true
---

# Prove It Works

Verify every task by checking the real thing directly. Never infer done-ness from proxies, self-reports, or "it compiles". Unverified work has unknown correctness, and indirect verification (a green deploy, a file timestamp, a delegate's summary) feels cheaper than direct observation right up until it is wrong. Acting on a wrong inference costs far more than checking the source.

## When it applies

After completing any task, before declaring done. Ask: "how do I prove this actually works?"

## The pattern

1. **Check the real thing, not a proxy.** Read the actual value in the org, not the code that should have produced it. Run the behavior, not the test that simulates part of it.
2. **Follow the full chain.** Does data flow from input to output? For integrations, test the whole communication path end to end, both directions.
3. **Trust artifacts, not self-reports.** When verifying delegated work, inspect the actual output (the diff, the org state, the runtime behavior), not the delegate's summary.
4. **When verification fails, suspect the observation method before the system.** A wrong org alias, a stale log, a query against the wrong object have all impersonated bugs.
5. **Script the check when you can.** A deterministic script that reruns the same comparison beats a one-time eyeball. Keep its output as the artifact a reviewer reruns instead of trusting your word (see **principle-build-the-lever**).

## Salesforce application notes

On Salesforce there is no local run, so "the real thing" always means a real org, and the evidence has standard shapes:

- **The behavior ran:** a record driven through the real path (UI, Apex script, API call) with the outcome verified by query or screenshot.
- **The IDs exist:** deployment IDs (`0Af...`), validation job IDs, test-run IDs (`707...`). These are the platform's receipts; a claim without them is a story.
- **The log shows it:** a debug log excerpt with the actual sequence and the `LIMIT_USAGE_FOR_NS` block, not a paraphrase of it.
- **The classic proxy traps:** "the deploy succeeded" (metadata is valid; says nothing about behavior), "the tests pass" (necessary, not sufficient; the tests never drove the UI path), "the code handles that case" (the code is a claim; the org is the fact).
- **The verification skill (**sf-create-verification-skill**) exists to make this cheap:** launch, drive, evidence, cleanup, already wired for the project.

## Gotchas and failure modes

- **Verifying the proxy anyway.** A green deploy as "done", a green suite as "the feature works". The proxy is cheaper every time and wrong often enough to matter. Check the real thing.
- **Observation errors read as bugs.** Wrong org alias, stale log, wrong sandbox: the system was fine and the lens was wrong. Suspect the method first.
- **Evidence captured but never attached.** An ID in a scrollback nobody can find is not evidence. Attach it to the reply, the PR, or the trail.

## Proof it applied

The evidence exists and is attached: the ID, the log excerpt, the screenshot, or the script and its output. "Done" without evidence is not a status; it is a hope.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
