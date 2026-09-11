---
name: sf-show-me-your-work
description: "Keep a reviewable decision trail (TSV) for long or unattended work: every decision, command, and piece of evidence, with test-run IDs and deployment IDs included. Use for /sf-show-me-your-work, autonomous runs, or any task the user will review after the fact."
---

# sf-show-me-your-work

When the user steps away, the work has to speak for itself later. sf-show-me-your-work keeps a decision trail: a plain TSV file where every row is one thing that happened, why, and the evidence. The user reviews the trail, not your memory.

## When to use it

- Autonomous runs and any work the user will review after the fact.
- Large or risky tasks where the reasoning needs to be auditable.
- Any time "trust me" would be the alternative.

## The format

A TSV file (for example `decision-trail.tsv` in the repo's scratch area), one row per entry, columns:

```
ts	actor	action	why	evidence
2026-09-11T09:14:02	agent	created scratch org arena-a	isolate candidate A	dev hub alias devhub, org 00Dxx0000000123
2026-09-11T09:31:44	agent	deployed candidate A	baseline for contest	deployment 0Afxx0000000456
2026-09-11T09:44:10	agent	chose queueable over batch	batch is overkill below 1M records	test run 707xx0000000789, CPU 4.2s of 10s
```

Rules:

- **Every decision gets a row with its reason.** Commands without decisions matter less than choices with reasons.
- **Evidence columns carry real IDs**: deployment IDs (`0Af...`), test-run IDs (`707...`), log file paths, screenshot paths. Not "tests passed".
- **Write as you go.** A trail reconstructed at the end is a story, not a record.
- **Keep it plain.** TSV, no formatting, readable in any editor.

## The steps

1. Open the file at the start of the work and log the goal in the first row.
2. Log each decision, command outcome, and piece of evidence as it happens.
3. At checkpoints (per the autonomous-run playbook), log position: what is proven, what is next.
4. At the end, log the outcome row and hand the file to the review (attach it to the PR or link it in the reply).

## Gotchas and failure modes

- **Logging activity instead of decisions.** Twenty rows of "ran command" and zero rows of "chose X over Y because" is a shell history, not a decision trail.
- **Evidence-free rows.** A claim row without an ID is an assertion. The evidence column is what separates the trail from a diary.
- **Giant logs inlined.** Point at log files by path; the trail stays scannable.

## Proof it worked

The trail exists, a reviewer can follow the work top to bottom without asking a question, and every claim row carries an ID or a path they can check.

## Reply

The trail's location, the headline decisions, and the outcome row.
