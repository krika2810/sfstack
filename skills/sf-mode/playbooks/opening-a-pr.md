# Playbook: opening-a-pr

Open a review-ready PR. Invoked at the end of every other playbook.

1. **Small ordered commits**, conventional-commit style messages, each commit deployable.
2. **Briefing-style body**: what changed and why; the data-model and automation-placement decisions; the access decisions (CRUD/FLS/sharing); the evidence - test-run ID, deployment/validation ID, coverage delta, verification-drive outputs; anything deliberately not done.
3. **Reviewability pass** (`/sf-interrogate` on your own diff first): no SOQL in loops, no hard-coded IDs, no SeeAllData, no leftover debug noise.
4. **Feature Map updated** for any new or changed user-facing behavior.
5. **Link the trail**: show-me-your-work TSV or a summary of it when the work was long.
