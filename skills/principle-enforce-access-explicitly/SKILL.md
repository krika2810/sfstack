---
name: principle-enforce-access-explicitly
description: "Apply to any data read/write path. CRUD/FLS and record sharing are chosen deliberately and stated: with sharing by default, Security.stripInaccessible or WITH USER_MODE where user-mode is intended, no silent escalation, guest-user paths reviewed line by line."
---

# Enforce access explicitly

Trusted is the first pillar of Well-Architected; on Salesforce, access control has two independent halves and agents must handle both:

1. **Object and field access (CRUD/FLS)**: Apex runs in system mode by default - it sees everything regardless of the running user's permissions. Where user-context access is intended, say so and enforce it: `Security.stripInaccessible()`, `WITH USER_MODE` in SOQL, or explicit `Schema.sObjectType...isAccessible()` checks. Where system mode is intended (system integration paths), say that instead - in a comment, at the boundary.
2. **Record access (sharing)**: `with sharing` is the default expectation; `without sharing` is a deliberate elevation that needs a stated reason and a narrowed scope. Apex-managed sharing, criteria-based rules, and OWD are design decisions made in `/sf-architect` Phase B, not discovered in review.

Corollaries: guest-user (Experience Cloud) paths get line-by-line review - unauthenticated system-mode code is the classic breach shape; dynamic SOQL from user input is injection until proven parameterized; sensitive fields do not go into debug logs.
