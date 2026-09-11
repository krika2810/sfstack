---
name: principle-guard-the-context-window
description: "Apply when context is filling up: large query results, long debug logs, wide metadata reads, fan-out planning. Route bulk to files and subagents; keep summaries and IDs in the main thread, not raw payloads."
disable-model-invocation: true
---

# Guard the Context Window

The context window is finite and does not refill within a session. Every token in it should be earning its place. When the context overflows, reasoning degrades, details get lost in compression, and the work stalls. Treat big payloads as something to route, not something to absorb.

## When it applies

Large outputs (query results, debug logs, deploy reports), long files, repeated reads of the same content, and any fan-out where planning could eat the budget before the work starts.

## The pattern

1. **Isolate large payloads.** Route verbose output to files and subagents. The main thread gets the summary and the IDs, not the raw data.
2. **Do not read what you will not use.** Read selectively, guided by relevance to the current step. A file not needed now is a file not read now.
3. **Keep high-frequency content inline.** Templates and references used on every invocation belong in the skill file itself, not in a separate file that costs a read each time.
4. **Size the phases.** Cap files per phase, set turn budgets, and checkpoint before the window forces you to.

## Salesforce application notes

Salesforce work generates unusually heavy payloads:

- **Debug logs are the main offender.** A full log at FINEST runs to megabytes. Save it (`sf apex get log --log-id ... --output-dir logs/`), then grep it for the lines that matter (`LIMIT_USAGE_FOR_NS`, `SOQL_EXECUTE_BEGIN`, `EXCEPTION`), and read only those windows. The log file path goes in the decision trail; the twelve relevant lines go in the thread.
- **Query results:** project only the columns you need, aggregate in SOQL (`COUNT()`, `GROUP BY`) instead of counting in your head, and page large extracts to CSV files (`sf data query --bulk --result-format csv --output-file ...`).
- **Metadata reads:** retrieve the specific components (`--metadata Flow:Order_Routing`), not the whole flow directory, when one component is the question.
- **Long autonomous runs:** write checkpoints to the decision trail (**sf-show-me-your-work**) so a fresh context can resume from the trail instead of re-reading the whole session (**sf-recall** is the resume path).

## Gotchas and failure modes

- **Routing out so much the thread loses the plot.** Summaries must carry the load-bearing facts (IDs, decisions, anomalies), not just "details in file".
- **File sprawl without a trail.** Payloads saved to files nobody can find are payload loss. Paths go in the decision trail.
- **Summaries that drop the anomaly.** The one failing row in 10,000 is the point of the read. Aggregation must name exceptions, not average them away.

## Proof it applied

The thread holds summaries, IDs, and file paths; the bulk lives in files; and a fresh session can reconstruct position from the trail alone.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
