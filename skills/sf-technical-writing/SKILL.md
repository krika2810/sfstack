---
name: sf-technical-writing
description: "The doc standard for Salesforce work: plain English, task-shaped (Diataxis), Salesforce vocabulary used exactly, every command and claim verifiable. Use for /sf-technical-writing, READMEs, docs, PR descriptions, and runbooks."
---

# sf-technical-writing

sf-technical-writing is the standard for anything written down: READMEs, runbooks, PR descriptions, design docs. The bar is that a reader can act on the document without asking a question, and verify every claim in it without trusting the author.

## When to use it

- READMEs, runbooks, design docs, PR descriptions, and commit messages.
- Any document a teammate (or a future you) will act on.

## The rules

1. **Plain English, short sentences.** One thought per sentence. If a sentence needs two commas and a semicolon, it is two sentences.
2. **Use the platform's own vocabulary exactly.** It is a scratch org, not a "sandbox environment thing". It is a record-triggered flow, not "an automation". It is `sf project deploy start`, not "push it up". Wrong vocabulary makes documents unsearchable and unverifiable.
3. **Know the four doc shapes and write one of them.** Tutorial (learning by doing), how-to (solving a specific problem), reference (exact facts), explanation (understanding why). A document that tries to be two shapes fails at both. This is the Diataxis distinction: https://diataxis.fr/
4. **Commands are copy-pasteable and complete.** Every command in the doc runs as written, with real aliases and paths:
   ```bash
   sf project deploy validate --source-dir force-app --target-org prod --test-level RunLocalTests
   ```
   Placeholders like `<your-org>` belong only where the reader genuinely must substitute, and the doc says what to substitute.
5. **Claims carry evidence or a label.** "The deploy takes about 4 minutes" is measured, estimated, or guessed, and the doc says which. Unlabeled claims rot into misinformation.
6. **Structure for scanning.** Headings that name the task, numbered steps in order, warnings next to the step they warn about. Nobody reads runbooks front to back during an incident.
7. **Write for the reader's moment.** A README is read by someone deciding whether to use the project. A runbook is read by someone with a problem at 2 AM. The 2 AM reader gets the fix in the first three lines.

## Salesforce-specific guidance

- **Name orgs and aliases precisely.** "Production" is a claim; `prod` (the alias authorized on this machine) is a fact a reader can run against.
- **Version-pin what moves.** Platform behavior changes across API versions (user mode defaults in 67.0, for example). When a doc depends on versioned behavior, name the version.
- **Runbooks include the rollback.** Every deploy or data runbook names the way back: the validated job to redeploy, the backup query, the destructive-change guard.
- **Screenshots earn their place.** A screenshot that shows state (the active flow version, the deployment status page) is evidence. A screenshot of a terminal that could have been pasted text is decoration.

## Gotchas and failure modes

- **Docs written for the author.** If only the author can run it, it is notes, not documentation. The test is a teammate running it cold.
- **Stale commands.** CLIs change (`sf scanner run` to `sf code-analyzer run`, `sfdx` to `sf`). Docs cite the current command and rot loudly when they stop working; that is a maintenance task, not a shameful one.
- **AI-tell prose.** Run **sf-unslop** on every document before shipping it.

## Proof it worked

A reader (or you, cold, in a fresh session) followed the document and reached the promised outcome without asking anything, and every command ran as written.

## Reply

The document, its shape (tutorial, how-to, reference, explanation), and the verification pass you ran on it.
