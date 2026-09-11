---
name: sf-recall
description: "Rebuild working context after losing it: what task was in flight, what orgs exist, what state the source is in, what evidence was already captured. Use for /sf-recall, at the start of a resumed session, or whenever the thread's memory of the work is gone."
---

# sf-recall

Sessions end, contexts compact, machines restart. sf-recall rebuilds the working picture from durable sources instead of asking the user to repeat themselves: the repo, the org list, the decision trail, and the task record.

## When to use it

- The start of a resumed session on an in-flight task.
- After context compaction, when the thread's memory is suspect.
- When returning to a project after days away.

## The steps

1. **Read the durable trail.** If a **sf-show-me-your-work** decision trail exists for the task, read it first. It is the authoritative record of what was decided and proven.
2. **Check the repo state.**
   ```bash
   git status
   git log --oneline -10
   git branch --show-current
   ```
   Uncommitted changes and recent commits tell you where the work stood.
3. **Check the orgs.**
   ```bash
   sf org list --all
   ```
   Scratch org aliases named after tasks tell you what environments exist and whether they are still alive. A dead scratch org means re-running the verification skill's launch step.
4. **Read the task record.** Todos, open PRs, and the verification skill's Feature Map for the project context.
5. **State the rebuilt picture.** One short brief: the task, the state of the source, the live orgs, the last proven checkpoint, and the next action. Confirm the next action with the user only if the trail is ambiguous about it.

## Gotchas and failure modes

- **Trusting memory over records.** If this skill is running, memory already failed once. The records win every disagreement.
- **Assuming orgs survived.** Scratch orgs expire on schedule. Verify with `sf org list --all` before planning around an org.
- **Resuming mid-flight without the checkpoint.** If the trail shows a proven checkpoint, resume from there, not from wherever feels convenient.

## Proof it worked

The rebuilt brief exists and names its sources (trail file, git state, org list output), and the next action is stated explicitly.

## Reply

The rebuilt brief and the proposed next action.
