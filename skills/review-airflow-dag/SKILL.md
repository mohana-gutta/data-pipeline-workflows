---
name: review-airflow-dag
description: Reviews Airflow DAGs for reliability, idempotency, and scalability. Use when the user shares a DAG, asks to "review this Airflow DAG", "why is my DAG flaky/slow", or wants orchestration best practices.
---

# Review an Airflow DAG

Lead with the biggest reliability risk, then the rest ranked by severity.

## Checklist

1. **Idempotency**: tasks safe to re-run for the same data interval; use `data_interval_start/end`, not `now()`; writes are overwrite/MERGE by partition, not blind append.
2. **Scheduling**: `catchup`, `max_active_runs`, `depends_on_past`, timezone, and dataset/asset-based scheduling where upstream dependencies exist.
3. **Task design**: right granularity (not one giant task, not hundreds of tiny ones); heavy compute offloaded to Spark/warehouse, not run in the worker; dynamic task mapping instead of loop-generated tasks when fan-out is data-dependent.
4. **Reliability**: retries with exponential backoff, `execution_timeout`, SLAs, `on_failure_callback` alerting, sensors in `reschedule` or deferrable mode, pools to protect downstream systems.
5. **Top-level code**: no heavy imports, DB calls, or API calls at parse time; keep DAG parse fast.
6. **Data passing**: XCom only for small metadata; large data via object store paths.
7. **Secrets/config**: connections and variables via secrets backend; no hardcoded credentials; Jinja templating used correctly.
8. **Testability**: DAG integrity test, task unit tests, and a backfill plan.

## Output

- Verdict and top risk
- Findings table: severity | task | issue | fix
- Refactored snippet (TaskFlow API where it helps) with clear variable names and comments
- Backfill/rerun instructions if relevant
