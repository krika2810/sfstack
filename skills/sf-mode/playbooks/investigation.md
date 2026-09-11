# Playbook: Investigation

A read-only question about a Salesforce org or its source: how does this automation work, why was it built this way, are we sure this behavior is what we think it is, should we do X or Y. The deliverable is an answer with citations, not a change. Do not edit source, do not deploy, do not run DML outside a scratch org.

## Steps

1. **Restate the question in one sentence.** Write it at the top of your working notes. Every step below exists to answer that sentence. If the question turns out to be three questions, list all three and answer them separately.
2. **Ground in the runtime.** Run the **sf-how** skill over the objects, automations, and classes the question touches. Naming a file is not grounding. You need the traced model: what runs, in what order, with what data. If the question is about why something exists, also run the **sf-why** skill on the same surface.
3. **Gather evidence from the real org.** Use the sf CLI against the relevant org, read-only commands only:
   ```bash
   sf data query --query "SELECT ApiName, Label, ProcessType, TriggerType FROM FlowDefinitionView WHERE Label = 'Order Routing'" --target-org myorg
   sf org display --target-org myorg
   sf apex get log --log-id 07Lxx0000000000 --target-org myorg
   ```
   Pull the actual metadata when the question is about configuration:
   ```bash
   sf project retrieve start --metadata Flow:Order_Routing --target-org myorg
   ```
4. **Cross-check against the source of truth.** If the org and the repo disagree, say so. The repo shows intent, the org shows reality, and drift between them is itself a finding.
5. **Answer with citations.** Every claim in the answer carries its source in the same sentence: a file path with line, a metadata component name, a log excerpt, or a query result. Label anything you could not verify as unverified instead of smoothing it over.

## Gotchas and failure modes

- **Answering from the repo alone.** Source can be undeployed or stale relative to the org. For behavior questions, the org is the truth; check it.
- **Answering from the org alone.** Setup menus show the current state, not the reason. Pair `sf-how` with `sf-why` before saying why something exists.
- **Silent mutations.** "Read-only" tools can still mutate: an Apex script that calls update, a Flow interview started by opening a record with an autolaunched flow on a platform event. Keep investigations to queries, retrieves, and log reads. If you must run behavior, do it in a scratch org and say so.
- **The order of execution is the usual culprit.** If the question is "why did this field get that value", the answer almost always lives in the sequence: before-save flows, before triggers, validation rules, after triggers, workflow field updates, after-save flows. Trace it, do not guess it. Reference: Triggers and Order of Execution, https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_order_of_execution.htm

## Proof it worked

The answer is done when every factual claim in it points at evidence you can reproduce: the exact `sf data query` you ran and its output, the log ID you read, or the metadata file you retrieved. A reviewer should be able to rerun any single command and see the same thing.

## Reply

Question restated, answer in plain English, evidence per claim, open questions that remain. If org and source disagree, that disagreement is the headline.
