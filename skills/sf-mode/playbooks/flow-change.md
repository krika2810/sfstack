# Playbook: Flow Change

Creating or changing a Flow. A Flow is metadata, so it gets the same rigor as Apex: source control, review, blast-radius checks, and driven verification. Record-triggered flows share the transaction with triggers and share their governor limits.

## Steps

1. **Pick the right automation for the job.** Before building, decide:
   - Before-save record-triggered flow: same-record field updates. Fast, cheap, runs before triggers.
   - After-save record-triggered flow: actions on other records, chatter posts, subflows.
   - Autolaunched flow: reusable logic called from Apex, other flows, or REST.
   - Apex: anything that needs bulk-safe collection processing, complex error handling, or unit tests with real assertions. Flows process records one interview at a time inside the batch and are easier to hit limits with at volume; the hand-off point to Apex is a design decision, not a last resort.
2. **Check what already runs on the object.** Query the automation map before adding to it:
   ```bash
   sf data query --query "SELECT ApiName, Label, ProcessType, TriggerType FROM FlowDefinitionView WHERE Label LIKE '%Order%'" --target-org myorg
   sf project retrieve start --metadata Flow --target-org myorg
   ```
   A second after-save flow on an object with a trigger changes everyone's order of execution.
3. **Set entry conditions so the flow runs rarely.** A record-triggered flow that fires on every edit of every record taxes every save on that object. Entry conditions narrow it: `$Record.StageName = 'Closed Won'` and only when changed, not "any edit".
4. **Build in a scratch org or sandbox, in source.** Retrieve the flow to source after building, or write the XML directly if you know the shape. The flow ships from the repo, not from the Setup menu.
   ```bash
   sf project retrieve start --metadata Flow:Order_Routing --target-org dev
   ```
5. **Keep DML out of loops.** Flow Get Records and Update Records elements inside a Loop element are the Flow version of SOQL-in-a-loop. Use a Collection variable: loop to build the list, one Update Records after the loop.
6. **Add fault paths.** Every Create, Update, Get, and Delete element gets a fault connector to something deliberate: a custom error log record, a notification, or a rethrow with context. A fault that goes nowhere becomes an unhandled flow error email to the admin and a confused user.
7. **Verify by driving it.** Deploy to a scratch org, create the triggering record through the UI or Apex, and inspect the result. Then read the debug log for the interview: `FLOW_START_INTERVIEW`, element outcomes, and governor usage. Screenshot the record or capture the query result as evidence.
8. **Activate deliberately.** A deployed flow is inactive by default. Activate the intended version, and confirm which version is active afterward:
   ```bash
   sf data query --query "SELECT Id, VersionNumber, Status FROM Flow WHERE DefinitionId = '301xx...'" --target-org myorg
   ```

## Gotchas and failure modes

- **Flows run in the same transaction as triggers.** A flow does not get its own governor budget. SOQL inside a flow counts against the same 100-query limit as the Apex around it.
- **Scheduled paths and async paths are separate transactions.** They run after commit and can hit stale expectations about record state; re-query inside the path.
- **$Record vs $Record__Prior.** "Only when updated to meet the condition" needs the prior-value comparison; a plain condition fires on every qualifying edit forever.
- **Flow tests exist.** Declarative flow tests (`FlowTest` metadata) can assert interview outcomes and run with `sf apex run test` deployments; use them for flows with real branching logic.
- **Paused interviews pile up.** Screen flows and autolaunched flows with pause elements create interview records that survive deploys; check for orphans when redesigning.

## Proof it worked

The driven behavior in a scratch org (record created through the real path, result verified by query or screenshot), the debug log excerpt of the interview, and the query showing the intended active version in the target org.

## Reply

What the flow does, why a flow (and not Apex) is the right home for it, the entry conditions, the evidence, and the active version.
