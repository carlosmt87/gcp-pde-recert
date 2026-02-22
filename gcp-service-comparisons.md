# GCP Service Comparisons — Quick Reference Cheat Sheet

Exam questions frequently test your ability to choose the *right* service for a given
scenario. This sheet organizes the key decision points side-by-side.

---

## 1. Storage: Choosing the Right Database

The single most-tested topic. Pick based on **data model**, **latency**, and **consistency needs**.

| Service | Data Model | Latency | Scale | Consistency | Best For |
|---------|-----------|---------|-------|-------------|----------|
| **BigQuery** | Columnar | Seconds | Petabytes | Eventually consistent reads | OLAP, analytics, data warehouse |
| **BigLake** | Columnar (on GCS files) | Seconds | Petabytes | — | Lakehouse — BigQuery governance on GCS Parquet/ORC |
| **Bigtable** | Wide-column (NoSQL) | <10ms | Petabytes | Strong (single-row) | Time-series, IoT, high-throughput key lookups |
| **Spanner** | Relational | 5–10ms | Unlimited (horizontal) | External (strongest) | Global OLTP, financial systems needing strong consistency across regions |
| **AlloyDB** | Relational (PostgreSQL) | <1ms | Vertical + read pools | Strong | High-performance PostgreSQL OLTP/HTAP; faster than Cloud SQL for analytics |
| **Cloud SQL** | Relational (MySQL/PG/SQL Server) | <5ms | Up to ~100 TB | Strong | Standard regional OLTP; lift-and-shift from on-prem relational DBs |
| **Firestore** | Document (NoSQL) | <10ms | Unlimited | Strong | Mobile/web apps, hierarchical document data, real-time sync |
| **Memorystore** | In-memory (Redis/Memcached) | <1ms | Up to ~300 GB | Eventual (Redis) | Session caching, rate limiting, leaderboards |
| **Cloud Storage** | Object (blobs) | 50–100ms first byte | Unlimited | Strong (for objects) | Unstructured files, data lake landing zone, backups, media |

### Storage Decision Tree

```
Need SQL analytics on large datasets?
  └─ Yes → BigQuery (or BigLake if data stays in GCS)

Need relational OLTP?
  ├─ Global scale, strong consistency → Spanner
  ├─ High-performance PostgreSQL (HTAP) → AlloyDB
  └─ Standard regional (MySQL/PG/SQL Server) → Cloud SQL

Need NoSQL?
  ├─ High throughput, time-series, key lookups → Bigtable
  ├─ Document store / mobile app / real-time sync → Firestore
  └─ In-memory cache → Memorystore

Need object/file storage?
  └─ Cloud Storage
```

---

## 2. Processing: Batch and Streaming

| Service | Paradigm | Interface | Managed? | Best For |
|---------|----------|-----------|----------|----------|
| **Dataflow** | Batch + Streaming | Apache Beam (Java/Python) | Fully serverless | Complex event processing, streaming aggregations, exactly-once semantics |
| **Dataproc** | Batch (+ Streaming) | Spark, Hadoop, Hive, Flink | Managed cluster | Lift-and-shift Spark/Hadoop workloads; teams with existing Spark code |
| **Cloud Data Fusion** | Batch + Streaming | Visual GUI (no-code/low-code) | Fully managed | Teams wanting visual ETL; broad connector library |
| **Dataform** | Batch (ELT only) | SQL | Fully managed | SQL-based ELT inside BigQuery; testing, scheduling, lineage |
| **BigQuery ML** | Batch inference | SQL | Serverless | Running ML training/inference on data already in BigQuery |

### Key Distinctions

- **Dataflow vs. Dataproc**: Dataflow is serverless (no cluster management); Dataproc requires cluster sizing but is better for existing Spark code.
- **Dataflow vs. Data Fusion**: Same outcome, different interface. Data Fusion compiles to Dataflow under the hood but adds visual tooling and pre-built connectors.
- **Dataform vs. Dataflow**: Dataform is SQL-only ELT *inside* BigQuery. Dataflow is code-based and moves data between systems.

---

## 3. Orchestration

| Service | Complexity | Interface | Execution Model | Best For |
|---------|-----------|-----------|----------------|----------|
| **Cloud Composer** | High | Python DAGs (Apache Airflow) | Managed Airflow cluster | Complex multi-step pipelines, conditional branching, backfilling, cross-service dependencies |
| **Workflows** | Medium | YAML / JSON | Serverless, pay-per-step | Moderate pipelines with sequential/conditional steps; lower cost for simple chains |
| **Cloud Scheduler** | Low | Cron | Triggers only (HTTP, Pub/Sub, jobs) | Time-based triggers; not an orchestrator on its own |
| **Dataform schedules** | Low | Built-in cron | Managed | Scheduling ELT transformation runs within BigQuery |
| **BigQuery Scheduled Queries** | Low | SQL | Managed | Simple recurring SQL jobs; no cross-service coordination |

### Choosing Orchestration

```
Complex DAG with branching, sensors, retries, backfill?
  └─ Cloud Composer

Moderate pipeline (5–15 sequential steps, some conditions)?
  └─ Workflows

Just need to trigger something on a schedule?
  └─ Cloud Scheduler (→ kicks off Composer/Workflows/Cloud Run/etc.)

Everything is SQL in BigQuery?
  └─ Dataform schedules or BigQuery Scheduled Queries
```

---

## 4. Data Governance and Metadata

| Service | Primary Function | Scope |
|---------|-----------------|-------|
| **Dataplex** | Unified data governance — lakes, zones, data quality, lineage | Multi-source (GCS + BQ + AlloyDB) |
| **Dataplex Catalog** | Metadata management, discovery, tagging, search | All GCP data assets |
| **Cloud DLP** | Detect and de-identify sensitive/PII data | BQ, GCS, Datastore, streaming |
| **Policy Tags (Data Catalog)** | Column-level classification for BigQuery security | BigQuery columns |
| **IAM** | Identity-based access control to resources | All GCP resources |
| **VPC Service Controls** | Network perimeter to prevent data exfiltration | GCP API calls |

### Governance Distinctions

- **Dataplex vs. Dataplex Catalog**: Dataplex is the *operational* governance layer (lakes, zones, quality scans, IAM centralization). Dataplex Catalog is the *discovery* layer (metadata, tagging, search).
- **Policy Tags vs. Cloud DLP**: Policy Tags enforce *access control* on columns (who can see what). Cloud DLP *detects and transforms* sensitive data in motion or at rest.
- **VPC Service Controls vs. IAM**: IAM controls *who* can access a resource. VPC SC controls *from where* (prevents exfiltration even by authorized users).

---

## 5. BigQuery Access Control Layers

From coarsest to finest grain:

| Mechanism | Granularity | Use Case |
|-----------|------------|----------|
| IAM on dataset | Dataset | Grant a team access to all tables in a dataset |
| Authorized views | Table/query | Share specific query results without exposing source tables |
| Row-level security (row access policies) | Row | Each user sees only their region/department rows |
| Column-level security (Policy Tags) | Column | Mask or block specific columns (PII, financial) |
| Dynamic data masking | Column (value-level) | Show masked values (NULL, hash) instead of blocking access entirely |

---

## 6. Data Sharing

| Method | Duplication | Discoverability | Cross-org? | Best For |
|--------|------------|----------------|-----------|----------|
| **Analytics Hub listing** | Zero-copy (linked dataset) | Yes (marketplace UI) | Yes | Formal data product sharing at scale, internal or external |
| **Authorized views** | None (live query) | No | No (same org) | Sharing specific query results within an org |
| **IAM on dataset** | None | No | No (requires IAM binding) | Direct cross-project access for known consumers |
| **BigQuery Data Transfer** | Yes (copies data) | No | Yes | When consumer needs an independent copy they control |

---

## 7. ML and AI Services

| Service | When to Use |
|---------|------------|
| **BigQuery ML** | Data already in BQ; SQL team; batch training and inference |
| **Vertex AI Training** | Custom TF/PyTorch/sklearn; GPU/TPU needed; large-scale training |
| **Vertex AI Endpoints** | Real-time (online) inference for custom models |
| **Vertex AI Batch Prediction** | Offline inference on large datasets with custom models |
| **Vertex AI Feature Store** | Consistent features across training and online serving (prevents skew) |
| **Vertex AI Embeddings API** | Generate text/image embeddings for semantic search or RAG |
| **Vertex AI Vector Search** | Managed ANN index for embedding similarity search (RAG retrieval) |
| **Pre-trained APIs** (Vision, NLP, Speech, Translation) | No training needed; call API directly; enrichment in Dataflow pipelines |

### RAG Architecture on GCP

```
Unstructured docs (PDF/text/images in GCS)
  → Extract text (Vision AI OCR / Document AI)
  → Chunk into segments
  → Embed chunks (Vertex AI Embeddings API: text-embedding-gecko)
  → Store in vector index (Vertex AI Vector Search or BigQuery VECTOR or AlloyDB pgvector)

At query time:
  → Embed user question
  → Retrieve top-k similar chunks (vector similarity search)
  → Pass chunks + question as context to Gemini (Vertex AI)
  → Return grounded response
```

---

## 8. BigQuery Pricing Model Comparison

| Model | Cost Driver | Predictability | Best For |
|-------|------------|---------------|----------|
| **On-demand** | Per TB scanned | Unpredictable | Sporadic workloads, dev/test |
| **Standard Edition** (Editions) | Per slot-hour (autoscaling) | Bounded by slot cap | Variable workloads needing cost ceiling |
| **Enterprise Edition** | Per slot-hour + features | Bounded | Production analytics, needs CMEK/BI Engine |
| **Enterprise Plus** | Per slot-hour + all features | Bounded | Mission-critical; 7-day time travel, table clones |
| **Committed (1yr/3yr)** | Discounted slot-hour | Fixed | Stable, predictable baseline workloads |

### Slot Concepts

- **Slot**: 1 virtual CPU for query execution. Complex queries fan out across many slots in parallel.
- **Reservation**: Named pool of slots assigned to a project/folder/org.
- **Assignment**: Maps a project to a reservation (without one → on-demand pricing).
- **Idle slot sharing**: Unused slots in one reservation flow to another — maximize utilization without waste.
- **Autoscaling max**: Cap how many extra slots a reservation can burst to — prevents runaway costs.

---

## 9. Streaming: Pub/Sub + Dataflow Patterns

| Scenario | Pattern |
|----------|---------|
| Fan-out (one message → many consumers) | Multiple Pub/Sub subscriptions on one topic |
| Ordered processing | Pub/Sub message ordering keys + Dataflow per-key ordering |
| Deduplication | Dataflow exactly-once semantics; or dedup by message ID in Bigtable |
| Late data | Dataflow allowed lateness + windowing |
| Dead-letter queue | Pub/Sub dead-letter topic for unprocessable messages |
| Backpressure / consumer lag | Monitor `subscription/oldest_unacked_message_age` |

---

## 10. Cloud Composer vs. Workflows — Side-by-Side

| | Cloud Composer | Workflows |
|--|---------------|-----------|
| **Based on** | Apache Airflow | Google Workflows DSL |
| **Language** | Python | YAML / JSON |
| **Startup time** | Minutes (cluster) | Seconds (serverless) |
| **Cost** | Hourly (always-on env) | Per-step execution |
| **Backfill support** | Yes (native Airflow) | Manual |
| **Branching/conditionals** | Full Python logic | Supported (limited) |
| **Sensors (wait for condition)** | Yes | Yes (polling via callbacks) |
| **DAG complexity** | Unlimited | Best for <20 steps |
| **Managed UI** | Airflow UI | Workflows console |
