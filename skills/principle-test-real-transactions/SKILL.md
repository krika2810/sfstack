---
name: principle-test-real-transactions
description: "Apply when writing, changing, or reviewing Apex tests. Test.startTest/stopTest boundaries for async, data factories over fixtures, no SeeAllData, assert outcomes not implementation, and remember: 75% coverage is a deployment floor, and passing tests alone are not verification."
---

# Test real transactions

- **Async only runs inside the boundaries**: future, queueable, batch, scheduled, and platform-event-triggered work executes at `Test.stopTest()`. Setup before `startTest`, trigger inside, asserts after. A test that never stops never runs the async code it claims to cover.
- **Every test builds its own data** through a shared factory. `SeeAllData=true` makes tests depend on org contents: they pass in one org, fail in the next, and prove nothing in either.
- **Setup DML discipline**: User/Group/other setup objects and business records cannot mix DML in one transaction - use `System.runAs` with a freshly created user.
- **Assert outcomes**: records created with the right fields, counts, platform events published, emails queued (`Limits.getEmailInvocations()`). Asserting a method ran is coverage theater.
- **One record and 200 records are different tests.** Both, for anything bulk.
- **75% is the deployment floor, not the goal.** Well-Architected expects automated tests wired to source control; coverage of lines is not coverage of behavior.
- **Tests alone are not verification** (pstack's rule, and it fits Salesforce perfectly): the task is verified when the behavior ran in a real org with evidence. Tests are how it stays verified.
