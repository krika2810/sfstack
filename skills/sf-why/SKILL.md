---
name: sf-why
description: "Investigate why a Salesforce solution was built this way: git history, PR comments, tickets, docs, setup audit trail. Use for /sf-why, design rationale questions, postmortems, or data-backed threshold questions."
---

# sf-why (Salesforce)

Code and metadata tell you what happens, rarely why. Query the historical evidence in parallel:

- **Git history and PR review comments** for the source in the repo.
- **Issue tracker / docs** for the requirement behind the change.
- **Setup audit trail** in the org for changes made outside source control (and note that their existence violates metadata-is-code - worth flagging).
- **Chat and incident records** for the outage, escalation, or deadline that produced the decision.
- **Data evidence**: when the question is a threshold or a volume assumption ("why batch size 50?"), measure the current data distribution and say whether the original reason still holds.

Distinguish three kinds of answer: deliberate decision (evidence of a tradeoff), accident of history (no evidence, just age), and platform constraint that no longer exists (Salesforce retired the limit or shipped the feature since). The third kind is a refactoring invitation.
