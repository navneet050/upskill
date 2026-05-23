# Databricks, Spark, Photon, Delta Lake & Iceberg — Deep Dive Master Notes

---

# 1. Introduction to Databricks

## What is Databricks?

Databricks is a cloud-based unified data and AI platform built originally by the creators of Apache Spark.

It combines:

* Data Engineering
* Data Warehousing
* Data Science
* Machine Learning
* AI / GenAI
* Streaming
* Governance

into a single platform.

Core philosophy:

> Bring all data, analytics, and AI workloads together on one platform.

---

# 2. Core Architecture of Databricks

## High-Level Architecture

```text
Data Sources
    ↓
Cloud Storage (S3 / ADLS / GCS)
    ↓
Delta Lake
    ↓
Databricks Compute Engine
    ↓
SQL / ML / AI / Streaming / BI
```

---

# 3. Separation of Compute and Storage

One of the foundational ideas in modern lakehouse architecture.

## Storage

Usually:

* AWS S3
* Azure ADLS
* Google GCS

## Compute

Usually:

* Spark Clusters
* Photon Engine
* SQL Warehouses

Benefits:

* Independent scaling
* Lower cost
* Flexibility
* Better cloud-native design

---

# 4. Delta Lake

## What is Delta Lake?

Delta Lake is an open table format and storage layer built on top of Parquet files.

It provides:

* ACID transactions
* Time travel
* Schema evolution
* Reliable updates/deletes
* Metadata management

---

## Internal Structure

```text
Table/
 ├── parquet files
 └── _delta_log/
```

The `_delta_log` stores:

* transaction logs
* schema metadata
* file additions/removals
* table versions

---

# 5. Apache Spark

## What is Spark?

Apache Spark is a distributed computing framework.

It handles:

* distributed execution
* scheduling
* DAG creation
* fault tolerance
* parallel processing

---

# 6. Spark Cluster Architecture

```text
        DRIVER NODE
             ↓
 ┌────────┬────────┬────────┐
 |Worker1 |Worker2 |Worker3 |
 └────────┴────────┴────────┘
```

## Driver Responsibilities

* Parses code
* Builds execution plan
* Schedules tasks
* Coordinates workers

## Worker Responsibilities

* Run executors
* Process partitions
* Execute distributed tasks

---

# 7. Spark Execution Flow

```text
Code
 ↓
Logical Plan
 ↓
Catalyst Optimizer
 ↓
Physical Plan
 ↓
DAG Scheduler
 ↓
Task Execution
```

---

# 8. Catalyst Optimizer

Catalyst is Spark’s query optimizer.

It performs:

* predicate pushdown
* join optimization
* column pruning
* constant folding
* query rewriting

---

# 9. DAG and Shuffle

## DAG

Directed Acyclic Graph.

Spark converts operations into stages.

## Narrow Transformations

Examples:

* filter
* map

No data movement required.

## Wide Transformations

Examples:

* join
* groupBy

Require shuffle.

---

# 10. Shuffle

Shuffle is one of the costliest Spark operations.

It involves:

```text
Worker → Network → Other Workers
```

Causes:

* network I/O
* disk I/O
* serialization overhead

Most Spark optimization revolves around reducing shuffle.

---

# 11. Photon

## What is Photon?

Photon is Databricks’ native high-performance execution engine.

Built in:

```text
C++
```

Photon accelerates:

* SQL workloads
* joins
* aggregations
* DataFrame execution
* Delta operations

---

# 12. Spark vs Photon

| Area        | Spark                         | Photon                  |
| ----------- | ----------------------------- | ----------------------- |
| Type        | Distributed compute framework | Native execution engine |
| Language    | JVM/Scala                     | C++                     |
| Handles     | Scheduling + execution        | Optimized execution     |
| Scope       | Full ecosystem                | Execution acceleration  |
| Open Source | Yes                           | No                      |

---

## Key Insight

Spark = orchestration + distributed framework
Photon = optimized execution layer underneath Spark

Photon does NOT replace Spark.

---

# 13. Vectorized Execution

Photon uses:

```text
Columnar + vectorized execution
```

Instead of processing:

```text
One row at a time
```

Photon processes:

```text
Batches of rows together
```

Benefits:

* better CPU utilization
* SIMD optimization
* lower memory overhead

---

# 14. Scala JAR Execution with Photon

## Can Scala JARs use Photon?

YES.

If the JAR runs on:

```text
Databricks Runtime with Photon
```

then Spark SQL/DataFrame operations may be accelerated by Photon.

---

## Important Caveat

Photon accelerates:

* DataFrame APIs
* SQL operators
* joins
* aggregations

Photon does NOT significantly accelerate:

* arbitrary Scala logic
* custom JVM code
* RDD-heavy operations
* many row-wise UDFs

---

## Best Practice

Prefer:

```scala
DataFrame APIs
```

instead of:

```scala
RDD-heavy logic
```

for maximum optimization.

---

# 15. Medallion Architecture

A very common Databricks pattern.

## Bronze Layer

Raw ingested data.

## Silver Layer

Validated and cleaned data.

## Gold Layer

Business-ready curated data.

---

# 16. Structured Streaming

Spark Structured Streaming supports:

* near real-time processing
* incremental ETL
* event pipelines
* fraud detection systems

Architecture:

```text
Kafka
  ↓
Spark Streaming
  ↓
Delta Lake
  ↓
Dashboards / Alerts
```

---

# 17. Unity Catalog

Databricks governance layer.

Provides:

* metadata management
* RBAC
* lineage
* auditing
* governance

Important for enterprises and banking.

---

# 18. Delta Lake vs Iceberg

## Both Are Open Table Formats

Both:

* sit on top of Parquet
* manage metadata
* provide transactions
* support time travel
* support schema evolution

---

# 19. What is an Open Table Format?

An open table format is:

> A public specification for managing analytical tables on object storage.

It defines:

* metadata
* snapshots
* transactions
* schema evolution
* file tracking
* query planning

---

# 20. Delta Lake

Optimized primarily for:

* Databricks ecosystem
* Spark ecosystem

Architecture:

```text
Parquet Files
      +
Transaction Log
```

---

# 21. Apache Iceberg

Iceberg is another open table format.

Optimized for:

* multi-engine interoperability
* large-scale metadata handling
* open ecosystem usage

Architecture:

```text
Metadata Files
    ↓
Manifest Lists
    ↓
Manifest Files
    ↓
Parquet Files
```

---

# 22. Key Difference Between Delta and Iceberg

## Delta Lake

Best integrated experience inside Databricks/Spark ecosystem.

## Iceberg

Best interoperability across many compute engines.

---

# 23. Engines Supporting Iceberg

Examples:

* Spark
* Trino
* Flink
* Athena
* Snowflake
* Hive
* Impala

---

# 24. Can Iceberg Work on Existing Cloudera Parquet Data?

YES.

You can:

* register existing parquet datasets
* convert them into Iceberg tables

After conversion you gain:

* ACID transactions
* schema evolution
* snapshots
* time travel
* reliable updates/deletes

---

# 25. Important Clarification

Iceberg does NOT replace Parquet.

Instead:

```text
Parquet = data files
Iceberg = table metadata brain
```

---

# 26. Spark Reading Plain Parquet vs Iceberg

## Plain Parquet

```python
spark.read.parquet("/path")
```

Spark directly scans filesystem/files.

---

## Iceberg

```python
spark.read.table("catalog.db.table")
```

Spark reads through:

```text
Catalog
  ↓
Iceberg Metadata
  ↓
Manifest Files
  ↓
Relevant Parquet Files
```

---

# 27. Advantages of Iceberg Read Path

Benefits:

* metadata-driven planning
* intelligent pruning
* snapshot isolation
* scalable metadata handling
* hidden partitioning
* efficient query planning

---

# 28. Hidden Partitioning in Iceberg

Traditional Hive partitioning:

```text
/date=2026-05-01/
```

Iceberg introduces:

```text
Hidden partitioning
```

Users don’t manually manage partition paths.

---

# 29. Time Travel in Iceberg

Iceberg stores snapshots.

Example conceptually:

```sql
SELECT * FROM table VERSION AS OF 123
```

Useful for:

* auditing
* debugging
* rollback
* compliance

---

# 30. Schema Evolution in Iceberg

Iceberg uses internal column IDs.

This allows safer:

* add column
* rename column
* reorder column
* drop column

without rewriting huge datasets.

---

# 31. Compaction in Iceberg

## Why Compaction Is Needed

Streaming systems create many small parquet files.

This causes:

* metadata explosion
* poor scan performance
* high query latency

---

# 32. What is Compaction?

Compaction means:

```text
Rewrite many small files
into fewer optimized larger files
```

---

# 33. Is Iceberg Compaction Automatic?

## Default Open-Source Iceberg

Usually:

```text
NO
```

You typically:

* schedule maintenance jobs
* run optimization procedures
* orchestrate compaction workflows

---

# 34. Example Compaction Procedure

```sql
CALL catalog.system.rewrite_data_files(
  table => 'db.transactions'
)
```

---

# 35. Iceberg Maintenance Operations

## Data File Compaction

Merge small parquet files.

## Manifest Compaction

Optimize metadata manifests.

## Snapshot Expiration

Remove old snapshots.

## Orphan File Cleanup

Delete unused files.

---

# 36. Why Compaction Matters

Compaction is critical for:

* metadata scalability
* query planning performance
* efficient scans
* lower cloud storage overhead

---

# 37. Enterprise Architecture Pattern

Common production architecture:

```text
Streaming Ingestion
       ↓
Small Files
       ↓
Scheduled Compaction Job
       ↓
Optimized Analytical Table
```

---

# 38. Delta vs Iceberg Operational Style

## Delta Lake

Often more tightly managed and automated.

Especially inside Databricks.

Features:

* optimize
* auto compaction
* predictive optimization

---

## Iceberg

Historically more open and decoupled.

Operational responsibility often handled explicitly through:

* Spark jobs
* Airflow
* orchestration systems

---

# 39. Most Important Mental Models

## Spark

```text
Distributed compute framework
```

---

## Photon

```text
Native optimized execution engine
```

---

## Parquet

```text
Physical file format
```

---

## Delta Lake / Iceberg

```text
Open table formats
```

---

## Databricks

```text
Unified data + AI platform
```

---

# 40. Final Architectural Mental Model

```text
Cloud Object Storage
        ↓
Parquet Files
        ↓
Delta Lake / Iceberg
        ↓
Spark / Photon / Trino / Flink
        ↓
Analytics / ML / AI / BI
```

---

# 41. Key Enterprise Insights

## Databricks Strengths

* unified platform
* Spark integration
* Photon acceleration
* AI ecosystem
* operational simplicity

---

## Iceberg Strengths

* openness
* multi-engine interoperability
* scalable metadata architecture
* platform neutrality

---

# 42. Recommended Next Topics

To go deeper into modern data platforms:

1. Spark DAG Internals
2. Catalyst Optimizer
3. AQE (Adaptive Query Execution)
4. Tungsten Engine
5. Delta Internals
6. Iceberg Metadata Internals
7. Data Skipping
8. Z-ordering
9. Vectorized Query Execution
10. Lakehouse Governance
11. RAG + Databricks AI Architecture
12. Streaming Lakehouse Design

---

# 43. Final Summary

Modern lakehouse architecture consists of:

```text
Object Storage
    +
Open Table Formats
    +
Distributed Compute Engines
    +
Governance
    +
AI/Analytics Platforms
```

Databricks, Spark, Delta Lake, Photon, and Iceberg are all key parts of this evolving ecosystem.

The industry trend is moving toward:

```text
Open storage
+
Flexible compute
+
Unified AI/data platforms
```

which is reshaping enterprise data architecture globally.
