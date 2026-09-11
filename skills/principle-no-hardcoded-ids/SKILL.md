---
name: principle-no-hardcoded-ids
description: "Apply when reviewing Apex or Flow. No hard-coded record IDs, user IDs, record-type IDs, or org-specific values anywhere: they break in every other org and silently point at the wrong records. Use custom metadata/labels, queries, and stable API names."
---

# No hardcoded IDs

A Salesforce record ID is minted per org. Hard-code one and the code works in exactly one org - the sandbox it was written in - then fails or, worse, silently resolves to a *different* record elsewhere. Well-Architected names hard-coded values (record types, users, IDs) an anti-pattern in both Flow and Apex.

The fixes by case:

- **Record types**: query by `DeveloperName` (stable across orgs), cache per transaction.
- **Configuration values** (a queue, a template, a threshold): custom metadata types or custom labels - deployable, org-specific, admin-editable.
- **Users/queues/groups**: query by name/developer name; never embed `005...` IDs.
- **URLs and instance names**: `Url.getOrgDomainUrl()` / `URL.getSalesforceBaseUrl()`, never `na123.salesforce.com` literals.
- **Test data**: the factory creates everything; tests reference what they created, never org records.
