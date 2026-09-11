# Well-Architected checklists for sf-interrogate

Drawn from Salesforce Well-Architected (https://architect.salesforce.com/well-architected/overview), organized by its three pillars. Use as Lens 6 of sf-interrogate: check the diff against each applicable item and cite the pattern or anti-pattern by name.

## Trusted (Secure, Reliable, Compliant)

- Every class declares its sharing mode explicitly; `without sharing` carries a written reason.
- CRUD/FLS is enforced on every data path: `WITH USER_MODE`, `Security.stripInaccessible()`, or explicit describe checks.
- No secrets, tokens, or credentials in source, custom settings committed to the repo, or debug logs left at FINEST in production.
- Validation and automation fail closed: a fault path in a flow, a caught exception in Apex, both with a deliberate record of the failure.
- Input from users and integrations is validated at the boundary before it reaches business logic.

## Easy (Automated, Engaging, Intentional)

- The work is source-driven: every changed component exists in the repo, not only in an org.
- Manual steps introduced by the change are named and justified; automatable steps are automated.
- No hard-coded IDs, usernames, emails, or URLs; configuration lives in custom metadata types or custom labels.
- The change is understandable by the next owner: named for the domain, with the Feature Map updated if the automation map changed.

## Adaptable (Resilient, Composable, Scalable)

- The change survives volume: bulkified, selective queries, async where the volume demands it.
- The change survives failure: idempotent where retries exist, callouts isolated from DML ordering traps, async jobs recoverable.
- Dependencies are explicit: new references between metadata components are intentional and recorded (sf-blast-radius territory).
- The change removes more than it adds where possible; dead code and superseded automation are deleted, not commented out.
