---
name: review-dbt-model
description: Reviews dbt models and projects for correctness, performance, and cost. Use when the user shares dbt SQL/YAML, asks to "review this dbt model", "is this incremental logic right", or wants to tune dbt materializations.
---

# Review a dbt Model

State the verdict first (ship / fix first / redesign), then findings ranked by severity.

## Checklist

1. **Materialization fit**: view vs table vs incremental vs ephemeral vs snapshot. Flag incremental on small tables and table on huge, append-only ones.
2. **Incremental correctness**:
   - `unique_key` truly unique; handle late-arriving data with a lookback window
   - `is_incremental()` filter uses an indexed/partition column, not a full scan
   - `on_schema_change` set deliberately; strategy (merge / delete+insert / insert_overwrite) matches warehouse
   - Full-refresh produces identical results
3. **Cost/perf**: partition and cluster config, early filtering, avoid `SELECT *`, avoid fan-out joins, push filters into staging, avoid repeated heavy CTEs (materialize or reuse).
4. **Tests and contracts**: unique/not_null on keys, relationships, accepted_values, freshness on sources, model contracts and constraints on marts, unit tests for tricky logic.
5. **Structure**: staging -> intermediate -> marts layering, one source reference per staging model, no business logic in staging, `ref`/`source` only (no hardcoded tables).
6. **Docs/metadata**: descriptions on marts, exposures, owners, tags.

## Output

- Verdict line
- Findings table: severity | location | issue | fix
- Corrected SQL/YAML for the top issues, with comments explaining each change
- Optional: estimated cost impact (bytes scanned, run time)

If key info is missing (warehouse, row counts, current runtime), give the best hypothesis first, then ask.
