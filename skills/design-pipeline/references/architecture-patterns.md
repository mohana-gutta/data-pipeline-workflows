# Architecture Patterns

## 1. Batch ELT (simple)
```
[SaaS/DB] -> [Loader: Fivetran/Airbyte/custom] -> [Raw/Bronze] -> [dbt staging -> marts] -> [BI]
                                                      ^ orchestrated by Airflow / managed scheduler
```
- Use when: freshness >= hourly, moderate volume, SQL-centric team.
- Trade-offs: cheap and simple; limited latency; full-refresh loads get costly as data grows.
- Levers: incremental models, partition pruning, clustering.

## 2. CDC + Medallion (intermediate)
```
[OLTP DB] -> [CDC: Debezium/DMS/Datastream] -> [Kafka/Kinesis/PubSub/EventHub]
          -> [Object store: Bronze (raw changes)] -> [Spark/SQL MERGE -> Silver (current state)]
          -> [dbt/Spark -> Gold (marts)] -> [Warehouse/BI]
```
- Use when: near-real-time needs (minutes), deletes/updates must propagate, large tables.
- Trade-offs: more moving parts; need compaction, schema evolution, and ordering/dedup handling.
- Table format: Delta/Iceberg/Hudi for MERGE and time travel.

## 3. Unified streaming + batch (advanced)
```
[Events] -> [Stream bus] -> [Flink/Spark Structured Streaming] -> [Lakehouse tables (append + upsert)]
                                                               -> [Serving store / feature store]
Batch backfills reuse the same transform code against the same tables.
```
- Use when: sub-minute SLAs, event-time semantics, shared logic for backfill and live.
- Trade-offs: highest ops burden and cost; need watermarking, state management, exactly-once sinks.

## 4. Reverse ETL
```
[Gold marts] -> [Census/Hightouch/custom] -> [CRM/Ads/Product DB]
```
- Use when: operational systems need warehouse-derived data. Watch rate limits and idempotent upserts.

## Cross-cloud mapping
| Concern | AWS | GCP | Azure |
|---|---|---|---|
| Object store | S3 | GCS | ADLS Gen2 |
| Stream bus | Kinesis/MSK | Pub/Sub | Event Hubs |
| Spark | EMR/Glue | Dataproc | Synapse/Databricks |
| Warehouse | Redshift | BigQuery | Synapse SQL |
| Orchestration | MWAA | Composer | Data Factory/MWAA-equivalent |
