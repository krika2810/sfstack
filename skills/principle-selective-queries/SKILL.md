---
name: principle-selective-queries
description: "Apply when writing or reviewing SOQL or SOSL. Every query filters on indexed fields with positive operators, returns only the fields needed, and stays selective at production volume: no leading-wildcard LIKE, no negative operators, no formula fields in WHERE without a custom index."
---

# Selective queries

A query is selective when the platform can use an index to find its rows instead of scanning the table. On a small org every query is fast, so non-selective queries hide until production volume arrives, and then they show up as 10-second timeouts, `System.QueryException: Non-selective query`, and locked tables during data loads. Selectivity is a production-readiness property, not a performance nicety.

## When it applies

Writing or reviewing any SOQL or SOSL, in Apex, flows, integrations, or reports, against any object that can grow.

## The rules

1. **Filter on indexed fields.** Standard indexes: Id, Name, record type, owner, most lookups and master-details, created/last-modified dates. Custom indexes: external IDs, unique fields, and fields Salesforce support or a custom index request has indexed. If your filter field is none of these, the query plan is a scan.
2. **Positive operators only.** `=`, `IN`, `>`, `<`, `LIKE 'prefix%'` use indexes. `!=`, `NOT IN`, `LIKE '%term'`, and comparisons against null do not.
3. **Select only the fields you need.** `SELECT FIELDS(ALL)` and wide field lists cost heap (10 MB synchronous) and readability. Name the fields.
4. **Keep result sets small by construction.** Filter by date windows, status, or ownership so the working set stays bounded as the table grows. A query that returns "everything active" is a time bomb on an object that never gets cleaned up.
5. **No formula fields in WHERE without thought.** Non-deterministic formulas cannot be indexed. Filtering on them scans.
6. **Check the plan at volume.** Use the Query Plan tool (Developer Console) or the REST explain endpoint against an org with representative data. As a rough platform rule of thumb, a filter is selective when it matches under about 10% of the first million records (and less beyond that); the plan tool gives the real verdict per query.

## Salesforce examples

```apex
// wrong: leading wildcard defeats the index, unbounded field list
List<Account> hits = [SELECT FIELDS(ALL) FROM Account WHERE Name LIKE '%Acme%'];

// right: prefix search on an indexed field, named fields
List<Account> hits = [SELECT Id, Name, OwnerId FROM Account WHERE Name LIKE 'Acme%'];

// wrong at volume: negative operator
List<Order__c> open = [SELECT Id FROM Order__c WHERE Status__c != 'Closed'];

// right: positive enumeration of the values you actually want
List<Order__c> open = [SELECT Id FROM Order__c WHERE Status__c IN ('Draft', 'Submitted')];
```

## Gotchas and failure modes

- **Selectivity is data-dependent.** The same query is selective in the demo org and a full scan in production. Test against loaded volume (Bulk API 2.0 rehearsal) before shipping query shape.
- **Skinny tables are a support lever, not a design.** If a query fundamentally needs to scan, that is an async or reporting-design question, not something to paper over.
- **SOSL has its own rules.** `FIND {term}` with `RETURNING` limits is the selective path for text search; do not emulate it with `LIKE` wildcards in SOQL.

## Proof you are following it

The query plan for each hot query shows an index-driven plan at production-like volume, and the debug log shows query rows well under the 50,000-row retrieval limit for the transaction.

## Source

Salesforce Well-Architected, Reliable - Performance: https://architect.salesforce.com/well-architected/overview
