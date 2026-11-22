---
name: optimize-spark-sql
description: Optimizes slow or costly PySpark and SQL workloads. Use when the user asks to "optimize this Spark job", "why is my query slow", "reduce warehouse cost", shares a query plan, or mentions skew, shuffles, or spill.
---

# Optimize Spark / SQL

Give the best-hypothesis bottleneck first, then fixes. Ask clarifying questions only afterward.

## Priority order (apply in this order)

1. **Reduce data volume**: filter and project as early as possible, partition pruning, columnar formats (Parquet/Delta/Iceberg), incremental instead of full reads, pre-aggregate before joins, drop unused columns.
2. **Pushdown**: predicates, projections, and aggregates pushed to the source/scan; verify in `EXPLAIN` (PushedFilters, PartitionFilters). Avoid UDFs that block pushdown; prefer built-in functions.
3. **Parallelism**:
   - Shuffle partitions sized to ~100-200 MB each; enable AQE (coalesce, skew join)
   - Detect skew (stage task time spread, hot keys); fix with salting or skew hints
   - Broadcast small join sides; avoid cartesian/explosive joins
   - Reduce small files (compaction, `OPTIMIZE`/`coalesce` on write)
4. **Caching**: cache only reused, expensive DataFrames; choose storage level deliberately; unpersist after use; prefer materializing to a table over cache for long pipelines.

## Diagnose

- Read the plan (`df.explain("formatted")`, Spark UI SQL tab, warehouse query profile)
- Look for: large shuffles, spill to disk, skewed tasks, full table scans, repeated scans of the same source, wide joins before filters
- Quantify: bytes read, shuffle bytes, task time p50 vs max

## Output

- Suspected bottleneck (one line)
- Ranked fixes, each with expected impact
- Optimized code with comments and clear variable names
- How to verify (metric to compare before/after)

See `references/spark-config-cheatsheet.md` for settings.
