---
name: principle-encode-lessons-in-structure
description: "Apply when you catch yourself writing the same instruction a second time, or notice a recurring correction. Encode the rule as a lint, a validation rule, a test, a CI check, or a script instead of more text."
disable-model-invocation: true
---

# Encode Lessons in Structure

When you catch yourself writing the same instruction a second time, stop writing it. Text instructions ask the reader to notice, remember, and comply. Structure enforces. Every correction, every surprise, every "don't do that again" is a learning signal: capture it, route it to the strongest mechanism that can hold it, and close the loop by applying it now.

## When it applies

- The same review comment appears on a second pull request.
- The same mistake ships twice.
- A runbook step exists only to say "remember to...".

## The pattern

1. **Ask: can this be a check?** A lint rule, a CI gate, a test, a validation rule, a metadata constraint, a script.
2. **If yes, encode it and delete the instruction.** The instruction was the symptom.
3. **If no (it needs judgment),** make the instruction more prominent and attach an example of the failure mode.

Pick the strongest mechanism the situation allows: a state the compiler or platform makes impossible beats a CI check, which beats a canonical helper, which beats a comment. People and agents copy what the surrounding code does; a weak guard becomes the next bad template.

## Salesforce application notes

The platform gives you unusually strong encoding mechanisms:

- **"Don't ship code without tests"** is already structural: the 75% deploy gate. Extend it with a CI step running `sf project deploy validate` on every PR so the gate fires before merge, not at release time.
- **"Don't query without a filter on this object"** becomes a Salesforce Code Analyzer rule run in CI: `sf code-analyzer run --rule-selector Recommended`.
- **"This status transition is illegal"** becomes a validation rule or an Apex guard the platform enforces, not a wiki page.
- **"Always set the external ID on load"** becomes a before-save flow or a required-field constraint.
- **Agent-side lessons** route to the skill files: a review gap becomes a line in **sf-interrogate**, a playbook gap becomes a playbook edit. That routing loop is the **sf-reflect** skill.
- **Recurring corrections in prompts or skills** get encoded into the skill text itself (this is how sfstack's own files should evolve).

## The feedback loop

- **Capture every correction.** One-off or pattern?
- **Route to the right layer.** One-off: note it. Recurring: skill line, lint rule, or platform constraint. Systemic: a principle.
- **Close the loop.** Apply the encoding now or file a concrete todo. "I'll keep that in mind" does not persist.

## Proof it applied

The recurring instruction is gone from prose and present as a mechanism: a rule, a test, a gate, or a constraint that fails loudly when violated.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
