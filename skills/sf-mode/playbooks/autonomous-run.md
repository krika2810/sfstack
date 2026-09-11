# Playbook: Autonomous Run

A long task to drive to completion without stopping: "run until done", "own this migration until it lands". You are unsupervised, so the decision trail and the checkpoints are what make the run reviewable afterward.

## Steps

1. **Open the decision trail.** Start a **sf-show-me-your-work** TSV at the beginning. Every decision, command, and piece of evidence lands in it as you go. This is what the user reviews when they return.
2. **Copy the governing playbook.** An autonomous run is not its own kind of work; it is a bug fix, feature, or migration run unattended. Match the task to its playbook and copy those steps into the todo list, then add the autonomy rules below.
3. **Budget the orgs.** Long runs burn scratch orgs. Create them with enough lifetime (`--duration-days 30`), name them after the run, and clean up as you go:
   ```bash
   sf org list --all
   sf org delete scratch --target-org run-old-alias --no-prompt
   ```
   DevHubs cap daily scratch org creation and active scratch orgs; a run that creates one org per experiment will hit the cap and stall.
4. **Checkpoint at every verifiable unit.** After each unit's proof exists, write one line to the trail: what was proven, the evidence ID, what is next. If the run dies or the context window fills, the trail is the resume point; apply **principle-guard-the-context-window** and keep bulky logs in files, not in the thread.
5. **Stop conditions are explicit.** Write them down at the start: a failing validation you cannot fix without a judgment call, a production-affecting decision, a destructive operation, anything on the irreversible list from sf-mode. When a stop condition fires, stop cleanly, write the state to the trail, and report.
6. **End with the evidence pack.** Test-run IDs, deployment IDs, the trail file, and a plain-English summary of what changed and what remains.

## Gotchas and failure modes

- **Drifting off the playbook.** Unattended runs drift. Re-read the governing playbook at each checkpoint and confirm the next action is on it.
- **Silent scope creep.** "While I am in here" changes belong in the trail as new decisions with their own evidence, or they do not happen.
- **Org sprawl.** Fifteen stale scratch orgs and a stalled DevHub. Delete as you go.

## Proof it worked

The complete decision trail, the evidence pack, and a summary a reviewer can audit top to bottom without rerunning anything.

## Reply

Outcome, evidence pack locations, stop conditions hit (if any), and what remains for a human decision.
