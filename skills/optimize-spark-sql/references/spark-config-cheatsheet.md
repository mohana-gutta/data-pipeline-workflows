# Spark Config Cheatsheet

| Setting | Purpose | Guidance |
|---|---|---|
| `spark.sql.adaptive.enabled` | AQE | Keep true |
| `spark.sql.adaptive.coalescePartitions.enabled` | Merge small shuffle partitions | true |
| `spark.sql.adaptive.skewJoin.enabled` | Split skewed partitions | true |
| `spark.sql.shuffle.partitions` | Shuffle parallelism | Start ~2-4x total cores; AQE adjusts down |
| `spark.sql.autoBroadcastJoinThreshold` | Broadcast small tables | Raise (e.g. 64-256MB) if executors have memory |
| `spark.sql.files.maxPartitionBytes` | Input split size | 128MB default; raise for many small files |
| `spark.sql.parquet.filterPushdown` | Parquet predicate pushdown | true |
| `spark.memory.fraction` | Execution/storage memory | Tune only if spill observed |
| `spark.dynamicAllocation.enabled` | Autoscale executors | true for variable workloads (cost) |

## Skew salting pattern
```python
from pyspark.sql import functions as F

SALT_BUCKETS = 16  # tune to skew severity

# Add a random salt to the large/skewed side
large_salted = large_df.withColumn("salt", (F.rand() * SALT_BUCKETS).cast("int"))

# Explode the small side across all salt values
small_exploded = small_df.withColumn("salt", F.explode(F.array(*[F.lit(i) for i in range(SALT_BUCKETS)])))

joined = large_salted.join(small_exploded, on=["join_key", "salt"], how="inner").drop("salt")
```
