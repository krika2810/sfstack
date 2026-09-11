---
name: sf-recall
description: "Rebuild your recent working context on a Salesforce topic from chat history, the repo record, and org state, handed back as a tight current-state brief. Use for /sf-recall at session start or when resuming parked work."
---

# sf-recall

Reconstruct where things stand from: your own chat history on the topic, git log and open branches, plan files and show-me-your-work TSVs left in the repo, live org state (what is actually deployed where - `sf org list`, deployment history), and any open incidents or user reports.

Hand back a current-state brief: what was decided (with the why if recorded), what is in flight (branch, scratch org alias, validation status), what evidence exists, and the exact next step. One screen, no narrative. If the record contradicts the org's actual state, the org wins and the discrepancy is the first item to fix.
