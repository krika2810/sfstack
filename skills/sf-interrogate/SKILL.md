---
name: sf-interrogate
description: "Adversarial multi-lens review of a Salesforce change: bulk safety, CRUD/FLS and sharing, SOQL selectivity, transaction and async design, test quality, and the Well-Architected pattern/anti-pattern checklists. Use for /sf-interrogate, 'stress test this', 'review this PR', or before any production deployment."
---

# sf-interrogate

Have several reviewers, each with one lens, try to break the change. Every finding needs a concrete failing scenario, not a taste preference. The checklists live in `references/well-architected-checklists.md` - they are distilled from Salesforce Well-Architected pattern/anti-pattern tables; run the relevant ones, not all of them every time.

## Lenses

1. **Bulk safety** - SOQL or DML inside loops; single-record code paths on trigger inputs; collections that assume small `Trigger.new`; DML before callouts in the same transaction; recursive re-entry without a guard that is itself bulk-safe.
2. **Query quality** - non-selective filters (leading-wildcard `LIKE`, negative operators as primary logic, `= NULL` as primary filter); `LIMIT 1` masking cardinality assumptions; `ALL ROWS`; SELECT *-style field grabs instead of named fields; missing selective indexes on filters that will run at volume.
3. **Security** - CRUD/FLS enforcement (`Security.stripInaccessible`, `WITH USER_MODE`, or a stated reason); `with sharing` vs `without sharing` chosen deliberately and commented; dynamic SOQL/SOSL built from user input (injection); sensitive data in debug logs; guest-user exposure on Experience Cloud paths.
4. **Transaction design** - work that will exceed per-transaction limits at volume and is not async; queueable/batch chains without failure handling; `Test.startTest/stopTest` boundaries not matching the async design; mixed DML setup errors in tests pointing at real user/record DML ordering bugs.
5. **Test quality** - `SeeAllData=true` anywhere; tests without asserts; coverage theater (lines touched, behavior unproven); no bulk test (1 record passes, 200 fail); async code tested without `Test.stopTest()` forcing execution; test data that depends on org data or hard-coded IDs.
6. **Metadata and declarative** - Flow anti-patterns (no entry criteria, hard-coded values, logic repeated instead of subflows, Flow doing high-volume work Apex should do); validation rules that block data loads; destructive changes not reviewed line by line.
7. **Deployment risk** - what breaks if this deploys while users work; field type changes with existing data; removed fields still referenced; dependency-order failures the validate run would have caught.

## Process

- Run each applicable lens as a separate review pass (subagents if available). Lenses do not argue with each other; they report.
- Each finding: severity, the exact file/element, the concrete scenario that fails (with data volume or record shape), and the smallest fix that removes the scenario.
- Findings the author disputes get settled by running code in a scratch org, not by debate. `/sf-arena` if it is a design disagreement.
- Summarize as: must-fix before merge, should-fix soon, noted. Do not relabel taste as must-fix.
