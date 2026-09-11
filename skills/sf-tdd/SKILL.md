---
name: sf-tdd
description: "Apex test-first with real transaction semantics: failing test first, then the fix; Test.startTest/stopTest for async; data factories; no SeeAllData; behavior asserts, not coverage theater. Use for /sf-tdd, regression tests, or when the bug has a cheap local test target."
---

# sf-tdd (Salesforce)

## Loop

1. **Write the failing test first.** Reproduce the bug or specify the behavior as an Apex test. Run it in a scratch org and watch it fail for the right reason - a test that fails for a setup error is not the test you wrote.
2. **Write the smallest fix.** No drive-by refactoring.
3. **Watch it pass, then run the suite.** The suite run is the blast-radius check.

## Salesforce test rules (non-negotiable)

- **Real transaction semantics**: async work (future, queueable, batch, scheduled, platform-event triggers) only executes inside `Test.startTest()` / `Test.stopTest()`. Put setup before `startTest`, the trigger inside, asserts after `stopTest`.
- **Data factories, not fixtures**: every test builds its own data via a shared `TestDataFactory`. `@isTest(SeeAllData=true)` is a defect, not a shortcut.
- **Bulk tests**: the 1-record test and the 200-record test are different tests. Trigger and batch code gets both.
- **Assert outcomes, not implementation**: query the records and assert field values, counts, and side effects (events published, emails queued via `Limits.getEmailInvocations()`). Asserting that a method was called is coverage theater.
- **Mixed DML**: setup objects (User, Group) and regular objects in one transaction fail; use `System.runAs` with a new user for setup DML.
- **75% is a deployment floor, not a target.** Well-Architected expects automated tests wired to source control; aim for behavior coverage of every path a user can hit.
- **LWC**: Jest unit tests via `sf lwc test run` (sfdx-lwc-jest) for component logic; the wire service and LMS have official test utilities. UI-level behavior gets a verification drive, not a Jest test pretending the DOM is Lightning.
- pstack's rule stands: **tests alone are not verification.** The task is done when the behavior ran in a real org and evidence exists - the tests are how it stays done.
