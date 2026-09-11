---
name: sf-how
description: "Trace how Salesforce behavior actually happens at runtime: the order of execution, the automation map on an object, the schema, and the debug log as the ground truth. Use for /sf-how, 'how does this work', grounding before design, or any behavior question."
---

# sf-how

sf-how answers "how does this actually work" with a traced model, not a guess. On Salesforce, behavior is rarely in one place: a single save can run before-save flows, before triggers, validation rules, after triggers, workflow rules, after-save flows, roll-up recalculations, and async jobs, in that order. sf-how traces all of it.

## When to use it

- "How does this automation work?" or "what happens when this record saves?"
- Phase A of any design work (sf-architect grounds with sf-how).
- Before touching any object whose automation map you have not read this session.

## The steps

1. **Scope the surface.** Which objects, which operations (create, update, which fields), which entry points (UI, API, integration, batch).
2. **Map the automation.** Enumerate everything that fires on that surface, in order-of-execution phase:
   ```bash
   sf data query --query "SELECT ApiName, Label, ProcessType, TriggerType FROM FlowDefinitionView WHERE Label LIKE '%Order%'" --target-org myorg
   sf project retrieve start --metadata Flow,ApexTrigger,ValidationRule --target-org myorg
   ```
   List, per object: before-save flows, before triggers, validation rules, duplicate rules, after triggers, workflow remnants, after-save flows, roll-up summaries, and the async jobs or platform events any of them kick off. Reference for the sequence: Triggers and Order of Execution, https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_order_of_execution.htm
3. **Read the code path.** For each Apex participant, read the trigger, its handler, and the services it calls. Note every SOQL and DML and which transaction phase they run in.
4. **Watch it run.** Reproduce the behavior in a scratch org with the debug log at the right levels and read the trace:
   ```bash
   sf apex tail log --target-org dev   # while you make the behavior happen
   ```
   The log shows the actual order, the actual queries, and the `LIMIT_USAGE_FOR_NS` block. When the map (step 2) and the log disagree, the log is right; update the map.
5. **Write the traced model.** One plain-English narrative: "When an Order is submitted through the UI, these things run in this order, and these side effects land." Include the source file for each participant so a reader can verify.

## Salesforce details worth knowing

- **The order is not intuitive.** Before-save flows run before before-triggers. Workflow field updates re-run before and after update triggers one more time. Roll-up summaries re-run the parent's save procedure. If you have not read the order of execution recently, your mental model is stale.
- **Post-commit is a separate world.** Queueable jobs, future methods, and async flow paths run after the commit, in new transactions, possibly on other records' stale expectations. They belong in the model explicitly.
- **Recursion is structural.** An update in an after trigger that touches the same object re-enters the order of execution. The traced model says how many times each participant can run per save.

## Gotchas and failure modes

- **Answering from source alone.** An inactive flow, a deactivated validation rule, or a managed package's hidden automation all change runtime behavior without appearing in your source tree. Steps 2 and 4 catch this; do not skip them.
- **Stopping at the trigger.** The handler calls services, the services call other services. Trace to the actual DML, not to the first layer that looks tidy.
- **Confusing "can run" with "runs".** Entry conditions mean half the mapped automation fires only for specific edits. The model says which.

## Proof it worked

The traced model exists, every participant names its source, and the runtime trace (log excerpt with the key sequence) backs the narrative.

## Reply

The plain-English traced model, the log evidence, and the surprises: anything the runtime does that the source did not lead you to expect.
