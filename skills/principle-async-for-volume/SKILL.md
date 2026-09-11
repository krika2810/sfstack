---
name: principle-async-for-volume
description: "Apply when a workload can exceed small data volumes or per-transaction limits. Favor asynchronous processing - queueable, batch, Bulk API 2.0 - sized to the volume, with latency tradeoffs stated and failure handling designed, not hoped for."
---

# Async for volume

Well-Architected's throughput guidance: favor asynchronous processing where latency allows. The per-transaction budget exists to stop runaway synchronous work; designs that need more than it provides belong in async lanes, sized to the data:

- **Queueable**: chained follow-up work, callouts after DML, medium volumes. State the chain-depth and failure behavior.
- **Batch Apex**: large data volumes processed in governed chunks (limits reset per batch). Choose scope size deliberately - it is the bulkification dial.
- **Bulk API 2.0**: data loads and extracts at scale. Loads sorted by ParentId, off-peak, minimum necessary data (Well-Architected's data-volume patterns).
- **Platform events**: decoupled, fire-and-forget integration - with the delivery semantics stated, because "eventual" is a design property, not a detail.

Tradeoffs to state, not skip: async adds latency (queue waits under load), complicates error handling (a failed batch needs an owner and a retry story), and spends org-level daily async limits. The synchronous user path stays synchronous when responsiveness is the requirement - the principle is *for volume*, not *always*.
