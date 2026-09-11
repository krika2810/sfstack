---
name: principle-no-hardcoded-ids
description: "Apply when reviewing Apex, Flow, formula, or integration config. No hard-coded record IDs, user IDs, profile IDs, record-type IDs, or org-specific URLs anywhere. IDs differ per org; references go through custom metadata types, custom labels, Custom Settings, or queries by name."
---

# No hard-coded IDs

A Salesforce ID (`0015g00000XYZ12AAZ`) identifies one record in one org. Hard-code it in Apex or a flow and the code works in exactly the org where you copied the ID, and breaks - or worse, silently points at the wrong record - everywhere else: every sandbox, every scratch org, production after a refresh. It is the most portable-looking way to write unportable code.

## When it applies

Reviewing or writing any Apex, Flow, formula field, validation rule, email template, or integration configuration that references a specific record, user, profile, record type, queue, or org URL.

## The rules

1. **Record types by developer name, queried once.** `Schema.SObjectType.Order__c.getRecordTypeInfosByDeveloperName().get('Internal').getRecordTypeId()`, cached in a static, never the `012...` string.
2. **Configuration records by custom metadata types.** "Which queue receives escalations" is a custom metadata record, deployable with the rest of the source, referable by name in Apex and flows without a query.
3. **Users and profiles by permission, not identity.** Code asks "does the running user have the `Order_Admin` custom permission", never "is the running user rva.raghav". People change jobs; permissions follow roles.
4. **URLs from the platform, not strings.** `Url.getOrgDomainUrl()` and Lightning navigation, never `https://acme.lightning.force.com/...` baked into code or templates.
5. **Named credentials for endpoints.** Integration URLs live in named credentials, so environments differ by configuration, not by code.

## Salesforce examples

```apex
// wrong: dies in every other org, or silently wrong after a refresh
Id internalRt = '0125g000000Ab12AAC';

// right: resolved per org, cached per transaction
private static Id internalRt;
public static Id internalRecordTypeId() {
    if (internalRt == null) {
        internalRt = Schema.SObjectType.Order__c
            .getRecordTypeInfosByDeveloperName()
            .get('Internal').getRecordTypeId();
    }
    return internalRt;
}
```

## Gotchas and failure modes

- **The 15-character trap.** 15-character IDs are case-sensitive; 18-character are case-insensitive. Code that compares IDs as strings across systems hits this eventually. Compare as `Id` types in Apex.
- **IDs in flows and formulas are harder to see.** Grep for the tell: a 15 or 18-character alphanumeric literal starting with a known key prefix (`001`, `005`, `012`, `00e`, `a0X`-style custom prefixes).
- **Sandboxes after refresh.** Even "temporary" hard-coding breaks on every refresh. There is no temporary form of this defect.

## Proof you are following it

A grep of the diff for ID-shaped literals finds none, every environment-specific reference resolves through metadata or query, and the same source deploys and behaves correctly in a fresh scratch org with no edits.

## Source

Salesforce Well-Architected, Easy - Automated (configuration over hard-coding): https://architect.salesforce.com/well-architected/overview
