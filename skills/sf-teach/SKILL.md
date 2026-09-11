---
name: sf-teach
description: "Explain Salesforce behavior by running sf-how and sf-why together and weaving one plain-English explanation. Use for /sf-teach, 'explain this', onboarding a teammate, or turning a traced investigation into lasting understanding."
---

# sf-teach

sf-teach turns an investigation into an explanation. It runs **sf-how** for the mechanics and **sf-why** for the intent, then weaves both into one plain-English account a person can learn from. The explanation is the deliverable, not a byproduct.

## When to use it

- "Explain how this works", "walk me through this object", "teach me this part of the org".
- Onboarding material for a teammate joining the project.
- After a hard bug, when the lesson deserves to outlive the fix.

## The steps

1. **Run sf-how** over the surface and keep the traced model: what runs, in what order, with what side effects.
2. **Run sf-why** on the same surface and keep the intent paragraph: what it is for, why it is shaped this way.
3. **Weave one explanation.** Structure it as a narrative in plain English:
   - Start with the user's action: "When a rep clicks Submit on an Order..."
   - Walk the runtime in order, naming each participant and what it contributes.
   - Explain the intent where the shape is surprising: "This check looks redundant; it exists because the ERP rejects orders without a primary contact, see ticket 4182."
   - End with the boundaries: what this automation deliberately does not do, and where its edges are (volume, sharing, error paths).
4. **Keep the evidence attached.** File paths, log excerpts, and ticket references stay in the explanation so the reader can verify anything surprising.
5. **Deliver in the medium asked for.** A chat explanation, a markdown doc in the repo (shaped by **sf-technical-writing**), or an entry in the Feature Map notes.

## Gotchas and failure modes

- **Explaining the code instead of the behavior.** A line-by-line readout of the handler is not an explanation. The narrative follows what happens and why, using the code as evidence.
- **Jargon stacking.** Use the platform's real vocabulary (record-triggered flow, roll-up summary, queueable) but explain each term the first time it appears if the audience might not know it.
- **No verification.** An explanation of behavior that was never traced is a guess wearing a teacher's hat. Steps 1 and 2 are mandatory.

## Proof it worked

The explanation exists, every behavioral claim in it traces to the sf-how evidence, every intent claim traces to the sf-why sources, and a reader could verify any single claim from the references given.

## Reply

The explanation itself, with sources attached.
