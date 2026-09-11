---
name: principle-bulkify-by-default
description: "Apply to any data operation in Apex or Flow. All operations work on collections; no single-record code paths; trigger and invocable inputs handled as 200-record batches from the first line. Bulkification is necessary but not sufficient for large data volumes."
---

# Bulkify by default

The rule: every SOQL query, DML statement, and Flow Get/Update element operates on collections. Trigger code assumes `Trigger.new` has 200 records, never one. Helper methods take `List`, not single records. Well-Architected lists DML/SOQL in loops and single-record operations as named anti-patterns.

Two corollaries agents miss:

1. **Bulkification is necessary, not sufficient.** A bulkified automation that needs more than per-transaction limits (100 SOQL / 150 DML / 10k rows) still dies at volume. Past that point the design is batch Apex, queueable chains, or Bulk API 2.0 - that is principle-async-for-volume, and the boundary is a design decision, not an accident.
2. **The recursion trap.** Bulk-safe code re-entered by the order of execution (your update fires another trigger or Flow that updates the same records) can burn its limit budget on the second pass. Recursion guards must themselves be bulk-safe (static sets, not static booleans that break batches).
