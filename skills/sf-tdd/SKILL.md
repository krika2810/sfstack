---
name: sf-tdd
description: "Apex test-first with real transaction semantics: failing test first, then the smallest fix; Test.startTest/stopTest around the trigger of the behavior; data factories, never SeeAllData; bulk and single-record paths both; asserts on outcomes, not coverage theater. Use for /sf-tdd, regression tests, or any bug with a cheap test target."
---

# sf-tdd

sf-tdd makes the broken or promised behavior executable before the fix exists. On Salesforce this is not a style preference: tests are also the deployment gate (75% org-wide coverage and all tests passing for production deploys), so the discipline pays twice. Reference: Testing Apex, https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_intro_writing_tests.htm

## When to use it

- A bug fix where the broken behavior can be expressed as an Apex test.
- A feature with a clear behavioral promise.
- Skip it when the test would need expensive harness setup for a tiny change, and say why in the reply; use the closest executable check instead.

## The loop

1. **Write the failing test first.** Express the intended behavior as an `@isTest` method. Run it in a scratch org and watch it fail for the right reason:
   ```bash
   sf apex run test --tests OrderTriggerTest.testSubmittedOrderPublishesEvent --target-org dev --result-format human --wait 10
   ```
   A test that fails on a setup error is not the test you wrote; fix the setup until the failure is the behavior.
2. **Write the smallest fix.** The change that makes the assertion true. No drive-by refactoring.
3. **Watch it pass, then run the neighbors.** The new test, then its class, then the local suite if the code is shared:
   ```bash
   sf apex run test --test-level RunLocalTests --target-org dev --result-format human --wait 20 --code-coverage
   ```
   The suite run is your blast-radius check.

## Salesforce test rules (non-negotiable)

1. **Real transaction semantics.** Async work (future methods, queueable jobs, batch runs, scheduled Apex, platform-event triggers) only executes in a test between `Test.startTest()` and `Test.stopTest()`. Put data setup before `startTest`, the trigger of the behavior inside, and the asserts after `stopTest`:
   ```apex
   @isTest
   static void submittedOrderPublishesEvent() {
       Order__c o = TestDataFactory.createOrder('Draft');
       Test.startTest();
       o.Status__c = 'Submitted';
       update o;
       Test.stopTest();
       List<Order_Sync_Log__c> logs = [SELECT Id FROM Order_Sync_Log__c WHERE Order__c = :o.Id];
       System.assertEquals(1, logs.size(), 'submitting an order queues exactly one sync');
   }
   ```
   `stopTest` also fires the enqueued async work synchronously, which is why the assert after it can see the side effects.
2. **Data factories, not fixtures.** Every test builds its own records through a shared `TestDataFactory` class. `@isTest(SeeAllData=true)` is a defect: it makes tests depend on org data and fail unpredictably across environments.
3. **Bulk tests beside single tests.** Trigger and batch code gets both a 1-record test and a 200-record test. The single-record test proves the logic; the 200-record test proves the bulkification. `Limits.getQueries()` inside the test can assert the query budget stays flat as volume grows.
4. **Assert outcomes, not mechanics.** Query the records and assert field values, counts, and side effects (platform events published, emails queued via `Limits.getEmailInvocations()`, tasks created). A test that only asserts a method ran is coverage theater; it passes while the feature is broken. See **principle-test-behavior-not-implementation**.
5. **Mixed DML needs `System.runAs`.** Setup objects (User, Group, GroupMember) and regular objects cannot be inserted in the same transaction. Create the test user with `System.runAs` around the setup DML.
6. **75% is a floor, not a target.** The deploy gate cares about the percentage; you care about behavior. Cover every path a user can hit, positive and negative, single and bulk. Test classes themselves do not count toward coverage.
7. **LWC gets Jest for logic, a drive for behavior.** Component logic is unit-tested with sfdx-lwc-jest (`npm run test:unit`, or `sf force lightning lwc test run`), using the official mocks for `@salesforce/apex`, the wire service, and LMS. UI-level behavior gets a real drive through the org, not a Jest test pretending jsdom is Lightning.

## Gotchas and failure modes

- **Tests that pass in scratch orgs and fail in production validation.** Usually caused by org data differences (record types, managed packages, other tests' side effects in parallel runs). Run tests with the same test level the deploy will use before claiming green.
- **Flaky time-dependent tests.** Logic keyed on `System.now()` needs injectable clocks or `Test` seams, or it fails at midnight and month boundaries.
- **Asserting before `stopTest`.** Async side effects do not exist yet. The assert goes after.
- **The deployment floor trap.** Writing asserts-free tests to hit 75% buys a deploy and sells out the next change. Well-Architected expects tests wired to source control and covering use cases, not percentages: https://architect.salesforce.com/well-architected/overview

## Proof it worked

- The failing test run before the fix, with the failure message showing the right reason.
- The passing run after, with the test-run ID (starts with `707`).
- The suite run with pass counts and per-class coverage, as the blast-radius evidence.

## Reply

The failing-then-passing evidence pasted verbatim, the test-run IDs, and anything you deliberately did not test with the reason.
