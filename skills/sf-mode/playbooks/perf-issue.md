# Playbook: perf-issue

A measured slowness, improved against a baseline - throughput mindset, not just CPU.

1. **Baseline first**: measure the current behavior in a real org at realistic volume - page timings, debug log timestamps, Limits telemetry. No baseline, no proof of improvement.
2. **Diagnose the actual bottleneck**: non-selective queries, N+1 patterns, serial DML, missing indexes, chatty managed packages, sync work that should be async. Query plan for suspect SOQL.
3. **One targeted fix** matching the diagnosis: selective filters, bulk paths, Platform Cache for repeated reads, async for volume (principle-async-for-volume), data-volume hygiene if skew is the cause.
4. **Same measurement again**: before/after numbers in the report. `/sf-swarm` for a big enough sample when variance is high.
5. **Regression check**: perf fixes that changed query shapes or transaction boundaries get the full test suite plus a verification drive.
