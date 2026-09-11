---
name: principle-test-real-transactions
description: "Apply when writing or changing Apex tests. Real transaction semantics: Test.startTest/stopTest around the behavior trigger, data factories instead of SeeAllData, bulk and single-record paths both, asserts on outcomes. 75% coverage is a deployment floor, not a target; proof means the behavior ran in a real org."
---

# Test real transactions

An Apex test is only worth what it proves, and on Salesforce it can only prove something if it respects the platform's transaction model. Async work does not run until you say so. Test data does not exist until you make it. A trigger fires on 200 records, not one. Tests that ignore these facts pass while the feature is broken.

## When it applies

Writing, changing, or reviewing any Apex test, and any decision about what "tested" means for a change.

## The rules

1. **startTest/stopTest frame the behavior.** Setup before `Test.startTest()`, the trigger of the behavior (the DML, the method call) between start and stop, asserts after `Test.stopTest()`. The stop call forces enqueued async work (future, queueable, batch, platform-event triggers) to execute, which is the only way a test can see its side effects. It also resets the governor counters, so the behavior gets a clean budget.
2. **Every test builds its own data.** A shared `TestDataFactory` creates the records each test needs. `@isTest(SeeAllData=true)` couples tests to org data and fails differently in every environment; it is a defect, not a shortcut.
3. **Bulk and single are different tests.** The 1-record test proves the logic. The 200-record test proves the bulkification. Trigger, invocable, and batch code gets both. Assert flat resource usage with the `Limits` class in the bulk test.
4. **Assert outcomes.** Query the records and assert field values, counts, and side effects: platform events published, emails queued (`Limits.getEmailInvocations()`), child records created. Asserting that a method was called proves a method was called.
5. **Mixed DML goes through System.runAs.** Setup objects (User, Group) and regular objects cannot mix in one transaction; create the test user inside `System.runAs`.
6. **75% is a floor, not a target.** Production deploys require 75% org-wide coverage with all tests passing, and every trigger needs some coverage. That is the gate, not the goal. The goal is every behavior a user can hit: positive, negative, single, bulk. Reference: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_intro_writing_tests.htm
7. **Tests are how work stays done, not proof it is done.** A green suite plus a behavior that was never driven is an unverified change. The **sf-tdd** skill and the verification drive are the other half.

## Salesforce example

```apex
@isTest
static void bulkSubmitPublishesOneEventPerOrder() {
    List<Order__c> orders = TestDataFactory.createOrders(200, 'Draft');
    Test.startTest();
    for (Order__c o : orders) { o.Status__c = 'Submitted'; }
    update orders;
    Test.stopTest();
    System.assertEquals(200, [SELECT COUNT() FROM Order_Sync_Log__c],
        'one sync queued per submitted order');
    System.assert(Limits.getQueries() < 10, 'bulk submit stays bulkified');
}
```

## Gotchas and failure modes

- **Asserting before stopTest.** The async side effect does not exist yet. Assert after.
- **Tests that pass only in the org they were born in.** Record types, managed packages, and parallel test side effects differ per org. Run with the deploy's test level before claiming green.
- **Coverage theater.** Tests written to reach 75% without asserting behavior buy a deploy and sell out the next change. Well-Architected expects use-case coverage wired to source control: https://architect.salesforce.com/well-architected/overview

## Proof you are following it

The test run shows behavior asserts passing for single and bulk paths in a real org, with the test-run ID (`707...`) captured, and no test class uses `SeeAllData=true`.

## Source

Testing Apex, Apex Developer Guide: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_intro_writing_tests.htm
