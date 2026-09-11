---
name: sf-figure-it-out
description: "Design an auditable one-off playbook when no bundled playbook fits the task. Use for /sf-figure-it-out, odd-shaped Salesforce work, or any task where sf-mode finds no match."
---

# sf-figure-it-out

Most Salesforce tasks fit a bundled playbook. Some do not: a one-time data repair with audit requirements, a managed-package upgrade rehearsal, an org split. sf-figure-it-out designs a bespoke playbook for the task in front of you, with the same rigor the bundled ones carry.

## When to use it

- sf-mode matched no playbook to the task.
- The task is large or cross-cutting enough that a narrower playbook would force it into the wrong shape.

## The steps

1. **State the outcome in one sentence.** What "done" means, verifiably. If you cannot write this sentence, the task is not ready to plan; run the **investigation** playbook first.
2. **Steal before inventing.** Read the bundled playbooks and lift the steps that apply. A data repair borrows from data-migration (idempotency, rehearsal, counts) and deploy (evidence IDs). The bespoke playbook is usually three bundled playbooks cut and reassembled.
3. **Write the steps with a proof per step.** Every step names what it produces and how that output is verified: a query result, a test-run ID, a deployment ID, a log excerpt. A step without a proof is two steps.
4. **Write the stop conditions.** Which discoveries halt the run and escalate to the user: destructive choices, production effects, judgment calls the evidence cannot settle.
5. **Run it like a bundled playbook.** Copy the steps into the todo list verbatim, keep a **sf-show-me-your-work** trail, and follow it. Deviations get logged with reasons.
6. **Offer the playbook back.** If the shape is likely to recur, sf-reflect it into a real playbook file.

## Gotchas and failure modes

- **Designing around the boring parts.** The odd task still needs the boring rigor: scratch-org rehearsal, evidence IDs, ordered commits. A bespoke playbook is not an exemption.
- **Proof-free steps.** The whole point is auditability. Step 3's proof-per-step is the skill.
- **Scope drift mid-run.** A bespoke plan is a contract. Changing it mid-run is allowed, but log the change and the reason in the trail.

## Proof it worked

The written playbook exists with a proof per step and stop conditions, the run followed it, and the trail shows each proof captured.

## Reply

The outcome sentence, the playbook, and after the run, the evidence per step.
