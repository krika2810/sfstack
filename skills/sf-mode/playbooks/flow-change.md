# Playbook: flow-change

Flows are metadata, and metadata is code - the same rigor as Apex, plus Flow's own traps.

1. **Retrieve and read the current Flow** from source; diff against the org if drift is suspected (principle-metadata-is-code).
2. **Check the Well-Architected Flow bar**: entry criteria present (no run-on-every-edit flows), no hard-coded values, subflows for reused logic, Get/Update elements bulk-safe, high-volume work handed to Apex.
3. **Blast-radius**: what records, objects, and integrations this Flow touches when it fires - including what else fires *because* it fires (order of execution).
4. **Test**: flow tests where they fit, plus Apex tests asserting the outcomes on records; bulk input (200 records) always.
5. **Verify live**: activate in a scratch org, drive the triggering operation, capture the debug log showing the interview path and outcomes.
6. **Deployment note**: flow activation state is metadata - say explicitly which version activates on deploy.
