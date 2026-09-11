# Playbook: Multi-Phase Plan

Work that spans phases or stacked commits: a migration across many call sites, a redesign landed in pieces, a large feature with separable parts. The plan is the deliverable before the work is, and every phase ends in proof.

## Steps

1. **Write the outcome in one sentence.** The end state, not the activity. "All Case routing runs through the new queueable service with the legacy trigger deleted" is an outcome. "Improve routing" is not.
2. **Break the work into verifiable units.** Apply **principle-sequence-verifiable-units**: each phase is small, ends in a check a reviewer can rerun, and leaves the org and the suite green. A phase that cannot be verified is two phases.
3. **Order by risk, not by convenience.** The riskiest unknown goes first. If the plan depends on a query being selective at volume, prove that in phase one with a scratch org and a Bulk API load, not in phase five after everything is built.
4. **Write the plan with a proof per phase.** For each phase: what changes, what verifies it, what evidence is captured (test-run ID, deployment ID, log excerpt, screenshot). The todo list in sf-mode carries these steps verbatim.
5. **Ship each phase behind the last one's green.** No phase starts until the previous phase's evidence exists. Stacked commits keep each reviewable; the deploy playbook governs anything that reaches a shared org.
6. **Re-plan when evidence contradicts the plan.** A phase that fails its check is information. Stop, say what the evidence showed, and rewrite the remaining phases before continuing. Do not absorb failures silently and press on.

## Gotchas and failure modes

- **Phases that are too big.** "Rewrite the service layer" is not a phase. "Move the three Account-trigger callers to the service, suite green" is.
- **Compatibility states that linger.** Temporary shims (an old trigger delegating to a new class) must have a deletion phase with a date, or they become permanent. Apply **principle-outcome-oriented-execution**: converge on the target, do not preserve throwaway states.
- **Long-lived scratch orgs.** A multi-week plan outlives a 7-day scratch org. Create longer-lived orgs (`--duration-days 30`), keep the source authoritative so any org can be rebuilt, and track active orgs with `sf org list --all` so you do not burn the DevHub's daily and active scratch org limits.

## Proof it worked

The plan itself, with each phase checked off by its evidence. The final phase's proof is the outcome sentence verified in a real org.

## Reply

The outcome sentence, the phases with their checks, current position, and what the next phase proves.
