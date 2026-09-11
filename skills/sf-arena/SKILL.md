---
name: sf-arena
description: "Spawn N parallel candidate solutions to the same design or implementation task, each in its own scratch org, measure them with the verification skill, and graft the strongest parts of the losers into the winner. Use for /sf-arena, 'arena this', or comparing competing designs with evidence instead of debate."
---

# sf-arena

Never accept the first design. Never debate designs abstractly either - on Salesforce, candidates are cheap to make real because scratch orgs are disposable.

## Steps

1. **Frame the task precisely**: the same prompt, the same Phase A grounding, the same acceptance criteria for every candidate. 2-3 candidates is usually right.
2. **One scratch org per candidate.** Each runner launches its own org (verification skill's Launch), implements its candidate, and proves it with the same drives and measurements. Parallel by default. Check the DevHub daily scratch org cap first and stagger if needed - the cap is a hard limit, not a suggestion.
3. **Measure the same things for every candidate**: Apex test results and coverage, Limits telemetry from instrumented runs (SOQL count, heap, CPU at realistic volume - seed enough data to make limits honest), deployment time, and for UI work, screenshots and interaction timings.
4. **Pick a base, graft the winners' parts.** The verdict is a short evidence table, not an essay. Losers' orgs are deleted; their best ideas are not.
5. **Record the decision** in the show-me-your-work trail: what won, why, with the numbers.

## Rules

- Throwaway means throwaway: candidate code that lost does not linger "just in case".
- If every candidate fails the same acceptance criterion, the criterion or the framing is wrong - reframe before re-running.
- Seeded data volume must be stated and identical across candidates. A design that wins on 10 records and dies on 10,000 is a trap.
