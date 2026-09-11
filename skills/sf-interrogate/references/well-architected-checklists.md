# Well-Architected checklists for sf-interrogate

Distilled from Salesforce Well-Architected (architect.salesforce.com) pattern/anti-pattern tables. Cite the source when a finding comes from here.

## Reliable - Performance (throughput)

Patterns: bulkified data operations; DML/database methods on collections; selective SOQL (positive operators, named fields only); SOSL for wildcard search; async processing favored where latency allows; Platform Cache for repeated reads.
Anti-patterns: DML/SOQL in loops; single-record operations; non-selective SOQL (LIKE, NOT/NOT IN as primary logic, = NULL as primary filter); LIMIT 1; ALL ROWS; synchronous-first design at volume.
Source: architect.salesforce.com/docs/architect/well-architected/guide/reliable

## Reliable - Scalability (data volume)

Patterns: no parent with 10k+ children; no user owning 10k+ records of one object; no 10k+ lookups to the same record; bulk loads sorted by ParentId; bulk loads off-peak; minimum needed data loaded; data lifecycle defined per object.
Anti-patterns: parent-child skew, ownership skew, lookup skew; unsorted concurrent bulk loads; peak-hour loads.
Source: architect.salesforce.com/docs/architect/well-architected/guide/reliable

## Adaptable - Resilient (ALM)

Patterns: source-driven development with the Salesforce CLI; environments matched to work type; tests automated on source control changes; scale tests in Full sandboxes for B2C-scale or high-volume systems.
Anti-patterns: org-based development; test automation absent; scale testing skipped for high-volume systems or done in undersized sandboxes.
Source: architect.salesforce.com/docs/architect/well-architected/guide/resilient

## Easy - Automated (data handling in automations)

Flow patterns: no hard-coded values (record types, users, IDs); entry criteria via decision elements; subflows for reused logic; logic handed to Apex at large data volumes.
Flow anti-patterns: hard-coded values; flows deactivated manually before data loads; flows erroring on governor limits; logic copy-pasted across flows.
Apex patterns: SOQL wrapped in try-catch; no SOQL in loops; selective queries; no hard-coded values.
Note: the order of execution is recursive and overlapping - where a limit error surfaces is often not the root cause. Bulkification alone does not make an automation large-data-volume safe; use batch Apex or Bulk API 2.0 past per-transaction limits.
Source: architect.salesforce.com/docs/architect/well-architected/guide/automated

## Trusted - Secure

Patterns: least-privilege permission sets; CRUD/FLS enforced in code paths; sharing model designed, not defaulted; session and API access scoped.
Source: architect.salesforce.com/docs/architect/well-architected/guide/trusted-overview
