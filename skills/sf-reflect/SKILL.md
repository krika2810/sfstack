---
name: sf-reflect
description: "Review a landed task and route its lessons into concrete skill edits: what surprised us, what rule would have caught it, which skill file should say so. Use for /sf-reflect, after a notable task lands, or after an incident or near-miss."
---

# sf-reflect

sf-reflect is the self-improvement loop. A task that taught something is not finished until the lesson is written into the structure that would have prevented the mistake: a skill file, a checklist, a generated rule. Lessons left in chat evaporate; lessons in structure compound. This is **principle-encode-lessons-in-structure** applied to sfstack itself.

## When to use it

- After a notable task lands: something surprised, something failed, something took three tries.
- After an incident or a near-miss (a governor limit nearly breached, a wrong-org deploy caught in time).
- Periodically, over a batch of landed work.

## The steps

1. **Replay the task from the evidence.** The decision trail, the PR, the test-run IDs. What was the goal, what happened, what surprised?
2. **Name the lessons as rules.** Each lesson becomes a checkable sentence: "SOQL inside a flow loop survived review because the flow lens only checks Apex." Not "be more careful", which is a wish, not a rule.
3. **Route each lesson to its structural home.**
   - A gap in a review lens -> edit `sf-interrogate/SKILL.md` or its Well-Architected checklist reference.
   - A missing step in a playbook -> edit the playbook in `skills/sf-mode/playbooks/`.
   - A recurring judgment call -> sharpen the matching principle skill.
   - A project-specific quirk -> the project-local verification skill or Feature Map, not the shared skills.
4. **Make the edit concrete.** The edit is a line a future run can apply, in the file a future run will read. Vague additions ("consider performance") are rejected; the edit must be checkable ("assert query counts stay flat at 200 records in the bulk test").
5. **Verify the edit would have caught it.** Replay the incident against the edited skill: would following the new line have prevented the surprise? If not, the edit is decoration. Rewrite it.

## Gotchas and failure modes

- **Lessons with no home.** If no skill file owns the lesson, that is itself the finding: the routing map has a gap. Say so and propose where a new check belongs.
- **Overfitting to one incident.** A rule written from a single surprise can be noise. Prefer rules that generalize ("assert flat query counts in bulk tests") over rules that name the incident ("remember the Order bug").
- **Editing without reading.** Skill edits go through the same care as code: read the file, place the line where it belongs, keep the file's structure.

## Proof it worked

Each lesson landed as a concrete edit in a named file, and the replay test (step 5) shows the edit would have caught the original surprise.

## Reply

The lessons, the edits made per file, and the replay verdict for each.
