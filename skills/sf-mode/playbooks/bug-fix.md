# Playbook: Bug Fix

A defect to reproduce, root-cause, and fix with runtime evidence. You own the task: plan, review, verify. Be scientific. Every shipped line traces to evidence from a real org. A change that "might help" is a hypothesis, not a fix, and it does not ship.

## Steps

1. **Reproduce it yourself.** Spin up a scratch org, deploy the current source, and make the bug happen with your own hands:
   ```bash
   sf org create scratch --definition-file config/project-scratch-def.json --alias bugrepro --duration-days 7
   sf project deploy start --source-dir force-app --target-org bugrepro
   ```
   Then drive the failing path: an Apex script (`sf apex run --file repro.apex --target-org bugrepro`), a data load, or a UI drive through `sf org open`. Do not hand the reproduction to the user. If it will not reproduce directly, force it: synthesize the trigger data, tighten the conditions, or add `System.debug` instrumentation until it fires.
2. **Binary-search the cause.** Write down the candidate hypotheses, then rule them out until one survives. Seed the list with **sf-how** over the affected automation and **sf-why** for regression history (`git log`, the setup audit trail). On Salesforce the usual suspects, in order: the order of execution (something else writes the field after your code), a bulk path your single-record fix never saw, sharing or FLS hiding records from the running user, and a governor limit killing a side path silently in a managed package context. Each pass, take the split that cuts the most remaining problem space, get runtime evidence, eliminate.
3. **Confirm the mechanism, not just the location.** Read the debug log at FINEST level for the relevant categories and point at the exact line where state went wrong:
   ```bash
   sf apex tail log --target-org bugrepro   # while you reproduce
   ```
   A stack trace tells you where it died. The mechanism is why it died, and you need the why before step 4.
4. **Write the failing test first.** Follow the **sf-tdd** skill: an Apex test that fails for the right reason before the fix and passes after. Run it and watch it fail:
   ```bash
   sf apex run test --tests OrderTriggerTest.testRepro --target-org bugrepro --result-format human --wait 10
   ```
   Skip this only when a test would be impractical, and say why in the reply.
5. **Write the smallest fix.** The smallest change the evidence justifies. No drive-by refactoring, no defensive code for cases the evidence does not show.
6. **Verify on the same surface.** The original reproduction now passes in the same scratch org. Then run the neighboring tests: the whole class, then the whole suite if the change is in shared code:
   ```bash
   sf apex run test --test-level RunLocalTests --target-org bugrepro --result-format human --wait 20
   ```
   "Inconclusive" is not a pass. Rerun until the evidence is clean or you can say exactly what blocked it.
7. **Run Opening a PR** (`opening-a-pr.md`). The failing test lands before the fix in the commit order, so history shows the bug and then the cure.

## Gotchas and failure modes

- **Fixing the symptom in the trigger layer.** If a before-save flow and a trigger both touch the field, fixing one and leaving the other is a relapse waiting for production. Trace the whole order of execution before choosing where the fix lives.
- **Single-record tunnel vision.** The bug report is one record. The fix runs on 200. Test the bulk path even when the report was a single record; the **principle-bulkify-by-default** skill applies.
- **Static state across a retried transaction.** Partial-success DML (`Database.update(records, false)`) can fire triggers more than once in one transaction, and static variables are not reset between those fires. If the bug involves duplicate side effects, this is the first place to look. Reference: Triggers and Order of Execution, https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_order_of_execution.htm
- **Environment-specific reproduction.** Data-dependent bugs (skew, sharing, volume) often refuse to fire in a fresh scratch org. Load representative data with `sf data import tree` or a Bulk API load before concluding "cannot reproduce".
- **Fixing in production.** Never. The fix ships through source, validate, deploy, per the **deploy** playbook.

## Proof it worked

- The failing test run, before the fix, with its test-run ID (starts with `707`) and the failure message.
- The passing test run, after the fix, with its test-run ID.
- The reproduction steps now passing, verbatim, from the same scratch org.
- For limit-adjacent bugs, the `LIMIT_USAGE_FOR_NS` lines from the debug log showing headroom after the fix.

## Reply

What was broken, the root cause (the mechanism, not the location), the fix, and the evidence: failing-then-passing test output pasted verbatim, with org alias and test-run IDs named.
