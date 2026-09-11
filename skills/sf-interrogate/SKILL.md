---
name: sf-interrogate
description: "Multi-lens adversarial review of Salesforce work before it ships: bulk safety, CRUD/FLS and sharing, SOQL selectivity, transaction and async design, test quality, and the Well-Architected pattern checklists. Use for /sf-interrogate, before any non-trivial deploy, or when a design needs an enemy."
---

# sf-interrogate

sf-interrogate attacks the work before production does. It reviews the diff through a fixed set of lenses, each tuned to a way Salesforce changes actually fail, and reports findings by severity. It is adversarial on purpose: the goal is to find the problem now, in the review, not in the 2 AM incident.

## When to use it

- Before any non-trivial deploy to a shared org.
- A design that survived sf-architect and needs an enemy before implementation.
- Code review where the stakes are high: payment paths, data deletion, security surfaces.

## The lenses

Run every lens against the diff. Each lens lists its findings with the file and line, a severity, and the reason.

### Lens 1: Bulk safety

- SOQL or DML inside any loop, including Flow loops.
- Single-record assumptions in triggers or invocable Apex: is `Trigger.new` treated as a list everywhere?
- Queries without a LIMIT inside batch or queueable paths that can grow.
- Recursion guards: static booleans that break on 200-record batches instead of static sets of processed IDs.
Reference: **principle-bulkify-by-default**.

### Lens 2: CRUD/FLS and sharing

- Every data read and write has a stated access decision: `with sharing` on the class, `WITH USER_MODE` on the query, or `Security.stripInaccessible()` on the DML.
- On API version 67.0 and later, Apex runs in user mode by default and undeclared classes run `with sharing`; older code may still assume system mode. Know which side the project sits on: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_security_sharing_chapter.htm
- `without sharing` and `WITH SYSTEM_MODE` need a written reason in the code or the PR.
- Queries that leak records across the sharing boundary into a UI or an integration response.
Reference: **principle-enforce-access-explicitly**.

### Lens 3: SOQL selectivity

- Filters on non-indexed fields against large objects.
- Negative operators (`!=`, `NOT IN`), leading-wildcard `LIKE`, and null comparisons, all of which defeat standard indexes.
- Formula fields in WHERE clauses, which cannot be indexed unless they are deterministic and custom-indexed.
Reference: **principle-selective-queries**.

### Lens 4: Transaction and async design

- Work in the synchronous path that exceeds what a 10-second CPU budget allows at real volume.
- Callouts after DML in the same transaction (the `System.CalloutException: You have uncommitted work pending` failure).
- Async chosen correctly: queueable for chainable small jobs, batch for large data volumes, platform events for decoupling.
- Anything approaching a limit that is uncatchable: prevention, not try/catch.
References: **principle-respect-the-shared-runtime**, **principle-async-for-volume**, and Execution Governors and Limits: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm

### Lens 5: Test quality

- Asserts on outcomes (field values, counts, side effects), not on method calls.
- Bulk tests (200 records) next to single-record tests for trigger and batch code.
- No `SeeAllData=true`; every test builds its own data.
- Coverage theater: tests written to hit 75% without asserting behavior.
Reference: **principle-test-real-transactions** and **principle-test-behavior-not-implementation**.

### Lens 6: Well-Architected fit

Run the diff against the pattern and anti-pattern checklists in `references/well-architected-checklists.md`, drawn from Salesforce Well-Architected (Trusted, Easy, Adaptable): https://architect.salesforce.com/well-architected/overview

## The steps

1. Assemble the diff: the changed source plus the Feature Map entries for every object it touches.
2. Run each lens in order. For every finding, write: severity, file and line, the failure it becomes in production.
3. Classify: blockers (fix before ship), warnings (fix or justify in the PR), notes (awareness only).
4. Report the findings table. The author fixes blockers and reruns the lenses that fired.

## Gotchas and failure modes

- **Lens shopping.** Running only the lenses the work is likely to pass defeats the point. All six, every time.
- **Severity inflation.** A naming nit is not a blocker. Blockers are findings that become incidents: limit breaches, security leaks, data corruption, deployment failures.
- **Rubber-stamping.** If three consecutive reviews find nothing, the lenses are not being run honestly. Re-read the diff with the checklist open.

## Proof it worked

The findings table exists, every lens was run, blockers are fixed or the deploy is held, and warnings carry either a fix or a written justification in the PR.

## Reply

Findings by lens and severity, what got fixed, and the residual risks a reviewer accepts by shipping.
