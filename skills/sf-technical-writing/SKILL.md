---
name: sf-technical-writing
description: "Layered documentation standard for Salesforce work: Diataxis structure (tutorial / how-to / reference / explanation), Google developer style sentences, plain global English. Use for /technical-writing, READMEs, design docs, runbooks, PR descriptions."
---

# sf-technical-writing

Every doc is exactly one Diataxis mode - mixing modes is what makes docs painful:

- **Tutorial**: learning by doing (set up your first scratch org and deploy this project).
- **How-to guide**: solving one real problem (rotate the integration user's credentials; roll back a deployment).
- **Reference**: dry, complete facts (object/field dictionary, API surface, test matrix).
- **Explanation**: background and tradeoffs (why the integration is queueable-chained; why sharing is Apex-managed).

Sentences: Google developer style - active voice, present tense, one idea per sentence, imperative steps. Global English: no idioms, no phrasal ambiguity - the reader may be three timezones away and translating mentally. Salesforce terms are used precisely and defined on first use in any doc aimed beyond the core team. Run `/sf-unslop` over the result before calling it done.
