---
name: design-pipeline
description: Designs data pipeline architectures. Use when the user asks to "design a pipeline", "architect an ETL/ELT flow", "how should we ingest X", "batch vs streaming", or wants options for moving and modeling data at scale.
---

# Design a Data Pipeline

Audience: a data engineer. Skip fundamentals. Be direct and structured.

## Process

1. **Extract requirements** from the request. If missing, state assumptions instead of stalling: source systems, volume (rows/day, GB/day), freshness SLA, consumers, schema volatility, cost sensitivity, cloud (default: cloud-agnostic, map to AWS/GCP/Azure equivalents in one line each).
2. **Propose 3 architectures, simple -> advanced.** For each:
   - ASCII diagram (sources -> ingest -> storage layers -> transform -> serve)
   - Components (use SQL, dbt, Airflow, PySpark, cloud-native services where relevant)
   - Trade-offs: cost, latency, operational burden, failure modes
   - "Use when" criteria
3. **Recommend one**, with the deciding factor in one sentence.
4. **Only then** offer code (DAG skeleton, dbt model, PySpark job). Do not write code before the diagram.

## Always address

- Idempotency and backfill strategy
- Data volume reduction early (filter/project/partition at source, incremental loads, CDC vs full)
- Failure handling: retries, dead-letter, alerting, late-arriving data
- Cost levers: storage format (Parquet/Delta/Iceberg), partitioning, compute autoscaling, compression
- Observability: freshness, volume, schema, and null-rate checks

## Output format

Lead with a 2-3 line summary of the recommendation, then the three options, then trade-off table, then next steps. See `references/architecture-patterns.md` for reusable patterns (batch ELT, CDC + medallion, lambda-free streaming, reverse ETL).
