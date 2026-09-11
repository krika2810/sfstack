---
name: principle-never-block-on-the-human
description: "Apply when tempted to ask 'should I do X?' on reversible work. Proceed, present the result, let the human course-correct after the fact; reserve confirmation for irreversible actions."
disable-model-invocation: true
---

# Never Block on the Human

The human supervises asynchronously. Do the work, present the result, and let them course-correct. Every permission pause stalls the pipeline and makes the human the bottleneck, and since most work is reversible and reviewable, a wrong decision usually costs less than the wait.

## When it applies

Any time you are tempted to ask "should I do X?" about work that can be undone.

## The pattern

1. **Proceed, then present.** Do the thing, show the result, explain the reasoning. "Should I refactor this trigger?" becomes the refactored trigger with the tests green and the reasoning attached.
2. **Reserve questions for genuine ambiguity.** Ask when intent cannot be inferred from context, not when the answer is merely uncertain. Uncertainty gets a decision and a note; ambiguity gets a question.
3. **Make the system self-healing.** Notice a problem, log it, fix it in the next round.
4. **Design for review after the fact.** The decision trail (**sf-show-me-your-work**) is what makes async supervision safe.

## The boundaries

- **Irreversible actions still require confirmation.** Force-push to shared branches, production deploys, data deletion, messages to people.
- **Product direction comes from the human.** Execution does not block; direction does.

## Salesforce application notes

Salesforce is unusually kind to this principle because scratch orgs make almost everything reversible:

- **A wrong experiment costs an org delete.** Trying the design in a scratch org (`sf org create scratch` ... `sf org delete scratch`) is cheaper than a meeting about whether to try it.
- **Reversible:** code on a branch, deploys to scratch orgs and most sandboxes, data loads into scratch orgs, metadata experiments, test runs. Proceed.
- **Not reversible:** production deploys (gated by the **deploy** playbook), destructive changes in shared orgs, hard deletes, anything touching live integrations or sent to users. Confirm.
- **The autonomy rules in sf-mode encode this boundary:** reversible work proceeds without asking, and the reply presents evidence instead of requesting permission. The exception list is short and explicit.

## Gotchas and failure modes

- **Proceeding through genuine ambiguity.** Reversible is a property of the work, not of the intent. If you cannot infer what the user wants, no amount of undo makes guessing right.
- **Reversibility as an excuse for no judgment.** Proceeding still means choosing well and presenting evidence, not trying things at random because they are cheap.
- **Presenting without proof.** "Done, review when you are back" without the evidence pack is a blocked review wearing a progress costume.

## Proof it applied

Work proceeded on the reversible list and the result was presented with evidence; anything on the irreversible list was confirmed first, with the confirmation recorded.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
