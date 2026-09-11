# Playbook: autonomous-run

Drive a long task to completion without stopping - the Salesforce edition budgets its environments first.

1. **Environment budget up front**: scratch orgs needed for the whole run vs the DevHub daily cap; queue strategy if the run exceeds it. An autonomous run that exhausts its org budget at hour three is a failure mode, not bad luck.
2. **Checkpoint state in the repo**: progress file + show-me-your-work TSV, so an interruption resumes instead of restarting (session pickup).
3. **Verify every unit before moving on**: deploy, test, drive, evidence. An autonomous run that skips verification is just faster failure.
4. **Never pause on reversible work** (principle: never-block-on-the-human); DO pause on: production deployments, destructive changes, security elevations, and anything spending org-level daily limits near their cap.
5. **Final report**: what shipped, evidence bundle (deployment IDs, test-run IDs), what was deliberately left, recommended next step.
