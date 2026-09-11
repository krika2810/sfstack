---
name: principle-selective-queries
description: "Apply when writing or reviewing SOQL/SOSL. Positive comparison operators, no leading-wildcard LIKE, only the fields needed, no LIMIT 1, no ALL ROWS; selective filters at production data volumes, verified with the query plan when in doubt."
---

# Selective queries

Well-Architected's exact bar, applied to every query:

- **Positive logic**: `IN`, `INCLUDES`, `=` as primary operators. `NOT IN`, `!=`, and `= NULL` as primary filters force table scans at volume.
- **No `LIKE` with leading wildcards** in filters that must scale; use SOSL for wildcard/text search.
- **Named fields only**: query the fields the code reads. `SELECT` grabs of every field burn heap and hide dependencies.
- **No `LIMIT 1`** masking a cardinality assumption - if exactly one row should exist, enforce it or handle the many case.
- **No `ALL ROWS`** unless the recycle bin is genuinely in scope.
- **Selectivity is about production volume, not the scratch org.** A query that is instant on 50 seed rows can be unselective on 5 million. For filters that will run at volume, check the query plan (Query Plan tool / REST explain) and whether a selective index exists.
