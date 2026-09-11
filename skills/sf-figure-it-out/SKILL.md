---
name: sf-figure-it-out
description: "Design an auditable one-off playbook when no bundled playbook fits: a large metadata migration, an ambitious multi-part change, or work a human reviews after stepping away. Use for /sf-figure-it-out."
---

# sf-figure-it-out

When no playbook fits, design one before starting:

1. **Decompose** into verifiable units. Each unit ends in a deploy + proof in a scratch org (sequence-verifiable-units). If a unit cannot be verified, split it again.
2. **Budget the environment**: scratch orgs needed, DevHub daily cap, data volumes to seed. An auditable plan that cannot get orgs is not a plan.
3. **Write the steps down** before executing: steps, verification per step, rollback per step, and the checkpoints where a human reviews evidence.
4. **Run it under `/sf-show-me-your-work`** so the trail exists when the human returns.
5. **Hand the playbook back** when done: if it worked, `/sf-reflect` turns it into a real playbook or skill edit so the next similar task is not a figure-it-out.
