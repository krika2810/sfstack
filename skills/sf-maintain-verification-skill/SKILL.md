---
name: sf-maintain-verification-skill
description: "Keep the project-local verification skill and Feature Map honest as the org's source changes: new objects, new flows, retired automations, moved package directories. Use for /sf-maintain-verification-skill, after schema or automation changes land, or when the drive path starts failing."
---

# sf-maintain-verification-skill

A verification skill is only useful while it describes the project as it is. Every new trigger, renamed object, or moved package directory makes it a little more wrong, and a wrong verification skill is worse than none because people trust it. This skill is the periodic pass that keeps it true.

## When to use it

- After work that changed the schema, the automation map, or the project layout.
- When the generated skill's drive step fails for reasons that look like drift (404 on the page, missing object, failing seed plan).
- On a cadence for active projects: after every few merged PRs that touch metadata.

## The steps

1. **Re-interview the project.** Run the same reads sf-create-verification-skill did: `sfdx-project.json`, package directories, the objects and flows now in source. Compare against the Feature Map, line by line.
2. **Diff the automation map.** List new triggers and flows (present in source, absent from the map), retired ones (in the map, gone from source), and changed ones (same name, different behavior). Each is an edit to the Feature Map.
3. **Re-run the drive path.** Execute the verification skill's launch, drive, and evidence steps as written. Fix every step that fails: the `sf org open --path` URL, the seed data plan, the Apex script names.
4. **Update the files.** Apply the edits to the verification SKILL.md and the Feature Map. Keep the five-section shape (launch, doctor, drive, evidence, cleanup) so the file stays predictable.
5. **Prove the update.** The updated skill runs clean end to end. That run is the evidence for this maintenance pass.

## Gotchas and failure modes

- **Editing without re-running.** An updated drive step that was never executed is an unverified claim. Step 3 and step 5 are the same discipline as sf-create-verification-skill's smoke test.
- **Map entries with no source.** A flow that only exists in some org and not in source is drift of a different kind: flag it for the team instead of silently adding it to the map.
- **Quiet scope creep.** Maintenance updates the map to match reality. Redesigning the verification approach is a separate task.

## Proof it worked

The updated verification skill ran end to end clean, and the Feature Map matches the source tree: every automation in source appears in the map, every map entry exists in source.
