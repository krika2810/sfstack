---
name: principle-test-behavior-not-implementation
description: "Apply when you write, change, or keep a test. Call the code the way its users do and assert the observable outcome against a literal expected value. If the test would still pass when the code does nothing, rewrite the assertion or delete the test."
disable-model-invocation: true
---

# Test Behavior, Not Implementation

A test calls the code the way its users do and asserts the result they observe against a literal expected value. A test that asserts which methods the code called, or restates a value the code contains, does neither. The check before keeping any test: would it still pass if the code under test did nothing? If yes, it observes no behavior and cannot fail for a defect. Rewrite the assertion or delete the test.

## When it applies

Writing, changing, or reviewing any test.

## The shapes that prove nothing

- **Weak or no assertion.** No assert at all (the "coverage special": call the method, assert nothing), or asserts that cannot fail: `System.assertNotEquals(null, x)` on a value the test itself just created.
- **Mock-only assertions.** Asserting a collaborator was called, not what the call achieved. On Salesforce the equivalents: asserting a service method ran, or that no exception was thrown, while the records sit unexamined.
- **Self-referential.** The expected value comes from the code under test: `System.assertEquals(OrderService.compute(o), OrderService.compute(o))`.
- **Constant pins.** The assertion restates a config value or label the code contains, so it fails when someone edits the constant and catches nothing real.
- **Fixture asserts fixture.** The assertion checks data the test built, and the subject never runs inside the test body.

## The fix

Call the subject inside the test body with one concrete input and assert the literal output or the observable effect. In Apex that means: perform the DML or call the method between `Test.startTest()` and `Test.stopTest()`, then query the records and assert field values, counts, and side effects:

```apex
Test.startTest();
o.Status__c = 'Submitted';
update o;
Test.stopTest();
System.assertEquals(1, [SELECT COUNT() FROM Order_Sync_Log__c WHERE Order__c = :o.Id],
    'submitting queues exactly one sync');
```

For an absence, assert the presence on the other input in the same test. For a constant, test the mechanism that reads it. For a mock, assert the payload it received or the state after the call.

## Salesforce application notes

- **The 75% deploy gate invites this sin.** Coverage counted without assertion is how orgs end up with 90% coverage and broken features. The gate wants lines executed; you want behavior proven. Serve both by never writing an assert-free test (see **principle-test-real-transactions**).
- **Asserts on platform side effects count as behavior:** emails queued (`Limits.getEmailInvocations()`), platform events published (observable via their subscribers' effects), tasks created, fields changed by the automation under test.
- **"Would it pass if the trigger did nothing?" is the Salesforce form of the check.** For a trigger test, that question is literal: comment out the logic and see. A test that stays green is deleted or rewritten.
- **Keep relational tests.** A test that asserts a real relation (the child exists for the parent, the sync log points at the order) is behavior testing, and it survives refactors that implementation-coupled tests do not.

## Gotchas and failure modes

- **Deleting tests that guarded real contracts.** The check is "would it pass if the code did nothing", not "is this test ugly". An implementation-coupled test that does catch defects gets rewritten, not deleted.
- **Literal asserts so brittle they pin the implementation anyway.** Asserting exact debug strings or full record serialization re-couples the test to internals. Assert the values a user observes.
- **Behavior tests nobody runs.** A suite too slow to run on every change gets skipped, and a skipped suite proves nothing. Keep behavior tests fast and isolate the slow ones.

## Proof it applied

Every kept test fails when the behavior is broken (verified at least once per test by the failing-first run in **sf-tdd**), and no test asserts only that code ran.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
