# Playbook: feature

New or changed behavior, built from a named data shape.

1. **Name the data shape first**: objects, fields, relationships, record types. On Salesforce the data model is the decision everything else hangs on. `/sf-architect` if it crosses object or transaction boundaries.
2. **Design the access**: CRUD/FLS and sharing for every new read/write path (principle-enforce-access-explicitly).
3. **Choose automation placement deliberately**: Flow vs Apex, decided on packageability and testability (Well-Architected), stated in the PR.
4. **Build in verifiable units**: each unit deploys to a scratch org and proves itself before the next starts.
5. **Tests with real transaction semantics** (`/sf-tdd`), bulk paths included.
6. **Verification drive** (`verify-<project>` skill): the feature exercised the way a user reaches it, evidence captured.
7. Update the Feature Map with the new feature - the map is the project's memory.
