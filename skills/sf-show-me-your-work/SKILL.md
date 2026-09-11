---
name: sf-show-me-your-work
description: "Keep a reviewable decision trail for long-running or unattended Salesforce work: a TSV log with one row per decision - what, why, evidence (deployment IDs, test-run IDs, query results). Use for /sf-show-me-your-work or any run the human reviews after the fact."
---

# sf-show-me-your-work

Log decisions as you make them to `DECISIONS.tsv` (or the project's named trail file). One row per decision:

`timestamp | decision | why | evidence | result`

- **decision**: what you chose, in one line.
- **why**: the reason, with the alternative rejected.
- **evidence**: the Salesforce-native proof - deployment ID, test-run ID, debug log path, query output, screenshot path. On this platform evidence has IDs; use them.
- **result**: what happened when it ran. Fill it after running; a decision without a result is a TODO.

Rules: local by default, committed when the human asks for a durable record. The trail is for review after the fact - write rows a skeptic can re-run. At the end of the run, summarize: decisions made, evidence pointers, anything that failed and how it was handled.
