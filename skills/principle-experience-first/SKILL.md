---
name: principle-experience-first
description: "Apply when product, UX, or feature-scope tradeoffs come up. Choose the user's outcome over implementation convenience; ship fewer polished behaviors over more rough ones; the next maintainer is a user too."
disable-model-invocation: true
---

# Experience First

When implementation convenience conflicts with the user's outcome, choose the user's outcome. The user is whoever consumes the work: the sales rep clicking the button, the colleague importing your class, the engineer who maintains this trigger next year. Weigh all of them, and explain impact from their perspective.

## When it applies

Product, UX, and feature-scope tradeoffs: what to build, how much of it, which rough edges to accept.

## The rules

- **Every feature and option must justify itself.** If you cannot say what the user notices, the feature does not earn its complexity.
- **Ship less, ship better.** Three behaviors that work every time beat ten that mostly work. On Salesforce, "mostly works" means intermittent flow error emails and users who stop trusting the automation.
- **Prototype before committing.** Design decisions are cheaper in a scratch org than in production. Click the path before you build the real one.
- **Get the details right.** Error messages a user can act on, field help text that answers the real question, validation rules that say what to fix, loading states, the empty state of a related list.
- **Tighten the core loop.** Every feature serves the central workflow or gets out of the way.

## Salesforce application notes

- **The admin is a user.** Configuration you add (custom metadata, flows, permission sets) is a UI the admin lives in. Name things for the domain, write descriptions, and keep the Setup surface navigable.
- **The integration consumer is a user.** An API that returns opaque errors ("An unexpected error occurred") instead of actionable ones costs the consumer days. Boundary error design (see **principle-boundary-discipline**) is experience design.
- **Failure is part of the experience.** A record-triggered flow with no fault path delivers its failure as a dead save and a confused rep. The fault path is a UX feature.
- **Foundations serve the experience, not the reverse.** **principle-foundational-thinking** governs the sequence of the work; this principle governs its target.

## Gotchas and failure modes

- **Gold-plating past what the user notices.** Delight is what the user experiences, not what makes the builder feel proud. If no user can tell the difference, the polish is waste.
- **Delight as scope justification.** "Experience" can rationalize any feature. The test stands: name what the user notices, or cut it.
- **Forgetting the maintainer.** An experience that dazzles users and exhausts the next engineer is half-built. The maintainer's experience is part of the ship.

## Proof it applied

You can state, for the shipped work, what the end user and the next maintainer each notice. If you cannot say what either would notice, the work or the explanation is off.

## Credit

Originally from Lauren Tan's pstack (MIT, (c) 2026 Lauren Tan, see `NOTICE.pstack`), vendored and expanded with Salesforce application notes.
