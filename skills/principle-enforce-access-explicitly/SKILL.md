---
name: principle-enforce-access-explicitly
description: "Apply to any data read or write path. CRUD, field-level security, and sharing are deliberate and stated: with sharing on the class, WITH USER_MODE or Security.stripInaccessible() on the data path, and a written reason for every without sharing or system-mode exception."
---

# Enforce access explicitly

Apex can see everything. That is the default power of system mode, and it is exactly the problem: code that silently reads or writes records the running user has no right to see is a security hole that demos perfectly. On Salesforce, access control has three layers - object CRUD, field-level security (FLS), and record-level sharing - and every data path your code builds makes a decision about all three, whether you write the decision down or not. This principle says: write it down, in the code, on purpose.

## When it applies

Every Apex class that queries or writes data, every API and integration surface, every LWC `@AuraEnabled` controller, and any flow that runs in system context.

## The rules

1. **Declare sharing on every class.** `public with sharing class OrderService`. On API version 67.0 and later, classes without a declaration run `with sharing` by default; on earlier versions an undeclared class can inherit `without sharing` from its caller, which is how sharing checks silently vanish. Declare it either way so the intent survives refactors and version bumps.
2. **Run data operations in user mode.** `WITH USER_MODE` on SOQL enforces CRUD and FLS in the query:
   ```apex
   List<Order__c> orders = [
       SELECT Id, Status__c, Total__c
       FROM Order__c
       WHERE Account__c = :accountId
       WITH USER_MODE
   ];
   ```
   On API version 67.0 and later, user mode is the default for Apex and `WITH SECURITY_ENFORCED` is disallowed; on earlier versions the default was system mode and the clause is what protects you. Know which side of that line the project's `sourceApiVersion` sits on. Reference: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_security_sharing_chapter.htm
3. **Strip what the user cannot touch before DML.** `Security.stripInaccessible()` removes fields and records the running user cannot create or update:
   ```apex
   SObjectAccessDecision decision = Security.stripInaccessible(
       AccessType.UPDATABLE, orders);
   update decision.getRecords();
   ```
4. **Every exception carries a written reason.** `without sharing`, `WITH SYSTEM_MODE`, and `inherited sharing` chosen to escape the default are sometimes right (system integrations, rollups across private data). When they appear, the reason is written in the code or the PR. An unexplained exception in review is a blocker, per **sf-interrogate**.
5. **Flows are not exempt.** A flow can run in user context or system context; the choice is on the flow's properties. "System context without sharing" is the powerful default to justify, not to drift into.

## Gotchas and failure modes

- **Leaks through the UI layer.** An LWC controller that queries in system mode and returns full records to the browser bypasses every FLS rule the admin set. The access decision belongs on the server path, and it travels with the data to the boundary.
- **Assuming with sharing covers CRUD/FLS.** It does not. Sharing governs which records; CRUD/FLS governs which objects and fields. You need both decisions.
- **Test as the real user.** Tests run as a system admin by default. `System.runAs` with a least-privilege test user is how you prove the access decision behaves for the people who will actually use it.

## Proof you are following it

Every class declares sharing, every query and DML names its mode or strips inaccessible, every exception has a written reason, and a `System.runAs` test proves the least-privilege user sees exactly what they should.

## Source

Apex Security and Sharing Model, Apex Developer Guide: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_security_sharing_chapter.htm; Salesforce Well-Architected, Trusted - Secure: https://architect.salesforce.com/well-architected/overview
