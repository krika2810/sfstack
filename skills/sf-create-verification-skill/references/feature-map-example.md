# Feature Map: example shape

One block per core object. The point: a future change can see everything that fires on an object before it edits any of it. Order follows the platform's order of execution.

## Order__c (custom object)

**Schema**: Master-detail to Account. Fields: Status__c (picklist), Total__c (currency, roll-up from Order_Line__c), External_Id__c (text, external ID, unique).

**Before-save (record-triggered flow, before)**
- `Order_Defaults` (flows/Order_Defaults.flow-meta.xml): sets Status__c to 'Draft' when null.

**Before triggers**
- `OrderTrigger` -> `OrderTriggerHandler.beforeUpdate` (classes/OrderTriggerHandler.cls): validates Total__c > 0 when Status__c leaves 'Draft'.

**Validation rules**
- `Order_Requires_Account_Contact` (objects/Order__c/validationRules): Primary_Contact__c required when Status__c = 'Submitted'.

**After triggers**
- `OrderTrigger` -> `OrderTriggerHandler.afterUpdate`: publishes `Order_Submitted__e` platform event on submit.

**After-save (record-triggered flow, after)**
- `Order_Notify` (flows/Order_Notify.flow-meta.xml): posts to Chatter on the Account when Status__c = 'Submitted'.

**Roll-up / cascade**
- `Total__c` roll-up summary recalculates when Order_Line__c changes (parent save procedure).

**Async**
- `OrderEventTrigger` on `Order_Submitted__e` enqueues `OrderSyncQueueable` (classes/OrderSyncQueueable.cls): callout to the ERP. Runs post-commit.

## Order_Line__c

**Schema**: Master-detail to Order__c (reparenting off). Lookup to Product2.

**Before triggers**: none. **Flows**: none. **Validation rules**: `Line_Quantity_Positive`.

**Notes**: quantity changes recalculate the Order roll-up, which re-runs the Order save procedure. Any change here is a change to Order behavior too.
