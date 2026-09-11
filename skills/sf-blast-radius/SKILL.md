---
name: sf-blast-radius
description: "Map what a metadata change could break before you make it: the dependency graph via the Tooling API's MetadataComponentDependency, then the behavioral blast radius proven by running the tests that cover the neighbors. Use for /sf-blast-radius, before deleting or reshaping shared metadata, or before destructive changes."
---

# sf-blast-radius

In Salesforce, everything references everything: flows reference fields, validation rules reference fields, Apex references objects, reports reference fields, email templates reference fields. Deleting or reshaping one component can break ten you forgot existed. sf-blast-radius maps those references before you touch anything, then proves the neighbors still work afterward.

## When to use it

- Before deleting or renaming a field, object, class, flow, or permission set.
- Before changing a field's type, a picklist's values, or a sharing setting.
- Before destructive-changes deploys.

## The steps

1. **Name the component exactly.** API name and type: `Order__c.Total__c`, `OrderTriggerHandler`, `Order_Routing` (flow).
2. **Query the dependency graph.** The Tooling API's `MetadataComponentDependency` table records who references whom:
   ```bash
   sf data query --query "SELECT MetadataComponentName, MetadataComponentType, RefMetadataComponentName, RefMetadataComponentType FROM MetadataComponentDependency WHERE RefMetadataComponentName = 'Total__c' AND RefMetadataComponentType = 'CustomField'" --target-org myorg --use-tooling-api
   ```
   This catches flows, validation rules, layouts, reports, and Apex classes that reference the component.
3. **Search source for what the graph misses.** Dependency data has gaps (some reference types are not recorded). Grep the repo for the API name as a backstop:
   ```bash
   grep -r "Total__c" force-app --include="*.cls" --include="*.flow-meta.xml" --include="*.object-meta.xml" -l
   ```
4. **Classify each dependent.** For every reference found: breaks (hard failure, deploy or runtime error), changes behavior (keeps working, differently), or unaffected. Breakers must be updated in the same change or the deletion does not ship.
5. **Make the change with its dependents.** The diff includes every breaker, updated, in the same commit or PR.
6. **Prove the neighbors still work.** Run the tests that cover the dependents, and drive the critical ones:
   ```bash
   sf apex run test --tests OrderTriggerTest,OrderRollupTest,OrderFlowTest --target-org verify --result-format human --wait 10
   ```
   The safety claim is proven by the run, not asserted by the graph. A dependency graph tells you what references the component; only execution tells you what still behaves.

## Gotchas and failure modes

- **Trusting the graph alone.** `MetadataComponentDependency` coverage varies by metadata type and API version. The source grep in step 3 is not optional.
- **Forgetting non-source references.** Reports, dashboards, list views, and email templates may live only in the org. Query the org, not just the repo.
- **Behavioral breakage with zero references.** Changing a picklist value breaks every integration and report filter that matches on the old value, none of which show up as dependencies. Think past the graph to data and integrations.
- **Hard deletes of fields destroy data.** A deleted custom field's data is gone after the platform's grace behavior. Treat field deletion as a data decision, not a metadata one.

## Proof it worked

The dependency list exists (graph query results plus grep results), every breaker was updated in the same change, and the test-run IDs show the dependents' tests green after the change.

## Reply

Component, the dependents found and their classification, what changed with it, and the evidence the neighbors survived.
