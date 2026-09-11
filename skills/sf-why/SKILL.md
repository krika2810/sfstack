---
name: sf-why
description: "Investigate intent: why a class, flow, field, or design decision exists. Sources are git history, ticket references, docs, the setup audit trail, and the code's own shape. Use for /sf-why, 'why was this built this way', or before redesigning something whose rationale you do not know."
---

# sf-why

sf-why answers "why does this exist" before you change it. sf-how tells you what runs; sf-why tells you what it is for. Changing automation without knowing its intent is how teams re-introduce bugs that were fixed two years ago.

## When to use it

- "Why was this built this way?"
- Before redesigning or deleting anything whose rationale you cannot state.
- When sf-how surfaced behavior that looks wrong but might be load-bearing.

## The steps

1. **Read the git history.**
   ```bash
   git log --follow --oneline -- force-app/main/default/classes/OrderTriggerHandler.cls
   git log -p --follow -- force-app/main/default/flows/Order_Routing.flow-meta.xml
   ```
   The commit messages and diffs show what changed and, with luck, why. `--follow` survives renames.
2. **Chase the references.** Ticket IDs in commit messages, PR descriptions, comments naming requirements. Read the actual ticket or PR, not a summary of it.
3. **Check the setup audit trail.** For org-level configuration that predates source control, the audit trail shows who changed what and when:
   ```bash
   sf data query --query "SELECT Id, Action, CreatedBy.Name, CreatedDate, Display FROM SetupAuditTrail ORDER BY CreatedDate DESC LIMIT 50" --target-org myorg
   ```
4. **Read the shape as evidence.** A recursion guard implies a recursion incident. A defensive null check implies a null that once shipped. An odd field type implies an integration constraint. Treat the code's scars as testimony.
5. **Ask the humans last.** When the record is exhausted, ask the people who were there, with the specific findings in hand: "The history shows the flow was split in March after ticket 4182; do you remember what broke?" A pointed question gets an answer; "why is this like this" gets a shrug.
6. **Write the intent down.** One paragraph: what it is for, what constraint produced the shape, what would break if it changed. This paragraph feeds sf-architect's Phase A and the Feature Map's notes.

## Gotchas and failure modes

- **Confusing absence of evidence with absence of reason.** "I cannot find why" means the record is missing, not that the thing is pointless. Unknown intent is a risk to price into the change, not a license to delete.
- **Trusting comments over history.** Comments describe what the author hoped; the diff shows what they did. History wins.
- **Intent that expired.** The reason may be gone (an integration retired, a regulation lifted). Saying "the reason no longer applies, here is the evidence" is a valid sf-why outcome and the green light for simplification.

## Proof it worked

The intent paragraph exists, every claim in it names its source (commit hash, ticket ID, audit trail entry), and unknowns are labeled unknown.

## Reply

The intent paragraph, the sources, and what the answer implies for the change being considered.
