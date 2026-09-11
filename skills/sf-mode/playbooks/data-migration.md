# Playbook: data-migration

Moving data at scale, Well-Architected's data-volume patterns applied.

1. **Profile the data**: volumes, distribution, skew risks (10k+ children per parent, 10k+ records per owner, 10k+ lookups to one record - all named anti-patterns).
2. **Choose the lane**: Bulk API 2.0 for scale; batch Apex for transformations; never record-by-record REST for bulk.
3. **Design the load**: sort by ParentId, off-peak windows, minimum necessary fields, automations that must (and must not) fire - disabling automations is itself a metadata change, tracked in source.
4. **Rehearse in a Full sandbox or scratch org** with production-like volume; Scale Test if the org has it. Measure and record timings.
5. **Verify by counts and samples**: source vs target counts per object, sampled field-level checks, and the business's own acceptance queries.
6. **Cutover plan with rollback**: what gets deleted or archived, when, and how it is undone.
