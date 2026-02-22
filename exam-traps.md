# Exam Traps — Common Wrong Answers

These are the decisions the GCP PDE exam specifically uses to trip people up. For each
trap, the pattern is: what the question looks like, what the wrong instinct is, and why
the correct answer wins.

---

## 1. Bigtable vs. Spanner

**The trap:** Both are managed, highly scalable, and support very high throughput.
Questions will describe scale + throughput and many people default to Bigtable.

**The tell:** Look for words like:
- "strongly consistent *across regions*" → **Spanner**
- "relational", "ACID", "multi-row transactions" → **Spanner**
- "key-value", "time-series", "sub-10ms single-row lookups", "IoT" → **Bigtable**

**Why Spanner wins when consistency is required:**
Bigtable is strong-consistent for single-row reads but does NOT support multi-row ACID
transactions or SQL joins. Spanner provides external consistency (stronger than
serializable) across regions. If the scenario says "global inventory must be updated
atomically" — that's Spanner.

**Rule:** Throughput + NoSQL → Bigtable. Throughput + relational + global consistency → Spanner.

---

## 2. Dataflow vs. Dataproc

**The trap:** Both process large datasets. Dataproc feels "more powerful" because it's
a full cluster.

**The tell:**
- "Existing Spark/Hadoop code", "Hive", "lift-and-shift from on-prem" → **Dataproc**
- "Serverless", "no cluster management", "streaming", "exactly-once" → **Dataflow**
- "Apache Beam" → **Dataflow** (Dataflow *is* managed Beam)

**Why Dataflow usually wins on GCP-native scenarios:**
The exam favors managed, serverless solutions. Dataflow autoscales, requires no cluster
sizing, and is the recommended service for new GCP-native pipelines. Dataproc is the
right answer specifically when Spark/Hadoop is already in use.

**Rule:** New pipeline on GCP → Dataflow. Migrating Spark workload → Dataproc.

---

## 3. Cloud Composer vs. Workflows

**The trap:** Both orchestrate multi-step pipelines. Workflows sounds simpler so people
assume it's always the right answer for "lightweight" cases.

**The tell:** Look for complexity signals:
- "Conditional branching", "sensors", "backfilling", "retry on failure with custom logic", "10+ tasks" → **Cloud Composer**
- "Sequential steps", "moderate conditions", "serverless", "cost-sensitive", "<10 steps" → **Workflows**

**The key question:** Does the scenario involve Airflow-specific features (sensors,
`BranchPythonOperator`, DAG backfill, complex Python logic)? → Composer. Otherwise
evaluate Workflows first.

**Rule:** If you'd need Airflow to do it, use Composer. If a YAML step definition
covers it, use Workflows.

---

## 4. Cloud DLP vs. Policy Tags (Column-Level Security)

**The trap:** Both relate to protecting sensitive data. Many people conflate "data
protection" with Cloud DLP.

| | Cloud DLP | Policy Tags |
|-|-----------|------------|
| **Action** | Detects and *transforms* data (masks, tokenizes, redacts) | Controls *access* to columns |
| **When data changes** | Yes — modifies values | No — data unchanged; access denied or masked at query time |
| **Ongoing protection** | Scan jobs run periodically or in pipelines | Enforced at every query, always |
| **Best for** | De-identifying data before sharing, scanning for PII, stream de-identification | Column-level access control in BigQuery |

**The tell:**
- "Detect PII in existing tables and remove it" → **Cloud DLP** (inspection + de-identification job)
- "Prevent certain users from seeing the `ssn` column" → **Policy Tags** (column-level security)
- "Mask the column value differently per user group" → **Policy Tags + dynamic data masking**

**Rule:** DLP = detect and transform data. Policy Tags = control who can see what column.

---

## 5. VPC Service Controls vs. IAM

**The trap:** IAM denies access to unauthorized users — surely that's enough?

**The key distinction:**
- IAM answers: *"Is this identity allowed to perform this action?"*
- VPC Service Controls answers: *"Is this request coming from an approved context (network, project)?"*

VPC SC prevents data exfiltration even by *authorized* users. An engineer with
`bigquery.dataViewer` could copy data to their personal GCP project — IAM allows it,
but VPC SC can block it.

**The tell:** Any scenario involving:
- "Prevent data exfiltration" → **VPC Service Controls**
- "Authorized users must not be able to move data outside the approved boundary" → **VPC SC**
- "Control who can access the resource" → **IAM**

---

## 6. Analytics Hub vs. Authorized Views vs. IAM Dataset Access

**The trap:** Authorized views feel like the "proper BigQuery way" to share data.

| | Analytics Hub | Authorized View | IAM on Dataset |
|-|--------------|-----------------|----------------|
| Discoverability | Yes (marketplace) | No | No |
| Cross-org sharing | Yes | No | No (requires org trust) |
| Self-service subscribe | Yes | No | No |
| Data duplication | Zero-copy | None (live query) | None |
| Best for | Formal data products at scale | Sharing a specific query result within org | Direct cross-project access for known teams |

**The tell:**
- "200 customers / 50 teams need to discover and subscribe to data" → **Analytics Hub**
- "Share a specific view of a table without exposing source" → **Authorized view**
- "Grant project X access to our BigQuery dataset" → **IAM**

---

## 7. On-Demand vs. BigQuery Reservations (Editions)

**The trap:** On-demand sounds cheaper because you only pay for what you scan.

**When on-demand is actually more expensive:** Consistent, high-volume workloads. A
team running 8 hours of queries daily will pay more per-slot on on-demand than on a
committed reservation.

**When on-demand wins:** Sporadic, unpredictable workloads (dev, ad-hoc exploration).

**The slot contention trap:** On-demand queries share a slot pool with fair scheduling.
A heavy batch job can starve interactive queries. Reservations with separate allocations
for BI vs. ETL solve this — the exam loves this scenario.

**Rule:**
- Predictable workload + production SLA → Editions with reservations
- Sporadic / dev / exploration → on-demand
- BI queries being slowed by ETL jobs → separate reservations

---

## 8. Dataform vs. Dataflow

**The trap:** Both "transform data" so people mix them up under time pressure.

| | Dataform | Dataflow |
|-|---------|---------|
| Language | SQL | Java / Python (Apache Beam) |
| Where it runs | Inside BigQuery | Managed Beam workers |
| Data movement | No — ELT within BQ | Yes — moves data between systems |
| Streaming | No | Yes |
| Best for | SQL transformations on data already in BQ | Processing data in transit; streaming; non-SQL logic |

**The tell:**
- "Transform data already in BigQuery using SQL" → **Dataform**
- "Read from Pub/Sub, enrich, write to BigQuery" → **Dataflow**
- "Version-controlled SQL transformations with tests" → **Dataform**

---

## 9. AlloyDB vs. Cloud SQL

**The trap:** Both are managed relational databases on GCP. Cloud SQL is more familiar.

**When AlloyDB wins:**
- The scenario mentions "high-performance PostgreSQL" or "analytical queries on operational data" (HTAP)
- Performance requirements exceed what Cloud SQL can provide
- The scenario mentions vector similarity search (AlloyDB supports `pgvector`)
- "4x faster read performance" or "complex queries on transactional data"

**When Cloud SQL wins:**
- Standard OLTP with no extreme performance requirements
- MySQL, SQL Server, or PostgreSQL with no special capabilities needed
- Lift-and-shift from on-prem MySQL/PostgreSQL

**Rule:** Standard relational OLTP → Cloud SQL. High-performance PostgreSQL / HTAP / vectors → AlloyDB.

---

## 10. Bigtable Row Key Design (Common Wrong Answers)

**The trap:** Using a sequential key like `timestamp` or `user_id` as a plain row key
causes hotspotting — all writes go to the same node.

**Common exam scenario:** "Queries need the last 24 hours of events per device; writes
are 100K/sec."

| Key Design | Problem | Use When |
|-----------|---------|---------|
| `timestamp` alone | Hotspot on latest node | Never for write-heavy time-series |
| `device_id` alone | Hotspot if one device is very active | Only for lookup-by-device with no time range |
| `device_id#timestamp` | Good for "all events for device X in range" | Time-series per entity |
| `device_id#reverse_timestamp` | Retrieves *most recent* first efficiently | Latest-N queries per device |
| Salted prefix (e.g., `hash%device_id#ts`) | Distributes writes across nodes | High write throughput with many devices |

---

## 11. BigQuery Partitioning vs. Clustering

**The trap:** Adding clustering everywhere; forgetting that clustering alone doesn't
prune partitions.

| | Partitioning | Clustering |
|-|-------------|-----------|
| Data organization | Physical split into separate segments by column value | Sorted within each partition by column(s) |
| Query cost reduction | Yes — unread partitions not scanned | Yes — blocks skipped within a partition |
| Best column type | Date/timestamp, integer range, ingestion time | High-cardinality columns (user_id, region, category) |
| Combine? | Yes — partition first, then cluster within partitions | Yes |

**Rule:** Partition on the column most used in `WHERE` date filters. Cluster on
high-cardinality columns frequently used in filters or `GROUP BY`.

---

## 12. Pub/Sub Pull vs. Push Subscriptions

**The trap:** Push sounds more "real-time" so people choose it for streaming scenarios.

| | Pull | Push |
|-|------|------|
| Consumer initiates | Yes — consumer calls `pull` API | No — Pub/Sub sends HTTP POST to endpoint |
| Best for | Dataflow, batch consumers, variable rate processing | Webhooks, Cloud Run, Cloud Functions, HTTP endpoints |
| Backpressure | Consumer controls rate | Pub/Sub retries on non-200 response |
| Auth | Consumer authenticates to Pub/Sub | Pub/Sub authenticates to your endpoint |

**The tell:**
- "Dataflow pipeline reads from Pub/Sub" → **Pull** (Dataflow uses pull internally)
- "Trigger a Cloud Function on each message" → **Push** (HTTP trigger to Cloud Function URL)

---

## 13. `WRITE_APPEND` vs. `WRITE_TRUNCATE` in BigQuery Load Jobs

**The trap:** `WRITE_APPEND` sounds safe ("just add more data"). But for idempotent
pipelines it causes duplicates on reprocessing.

**Rule:**
- Reprocessing / backfilling → `WRITE_TRUNCATE` (on the specific partition, not the whole table)
- Accumulating data that should never be overwritten → `WRITE_APPEND`
- Daily pipeline that must be re-runnable → `WRITE_TRUNCATE` with partition decorator

---

## Quick-Scan Trigger Words

| See this in a question... | Think... |
|--------------------------|---------|
| "global", "multi-region", "strongly consistent OLTP" | Spanner |
| "time-series", "IoT", "high throughput NoSQL", "sub-10ms key lookup" | Bigtable |
| "existing Spark/Hadoop", "Hive", "lift-and-shift" | Dataproc |
| "serverless", "streaming", "exactly-once", "Apache Beam" | Dataflow |
| "SQL ELT", "version-controlled transformations", "assertions" | Dataform |
| "discover datasets", "tag templates", "metadata search" | Dataplex Catalog |
| "prevent data exfiltration", "authorized users moving data outside boundary" | VPC Service Controls |
| "200 customers subscribe to data", "data marketplace", "zero-copy sharing" | Analytics Hub |
| "column-level access control", "nullify sensitive column" | Policy Tags + dynamic masking |
| "detect PII", "de-identify", "tokenize", "redact" | Cloud DLP |
| "complex branching", "sensors", "backfill historical runs" | Cloud Composer |
| "sequential steps", "serverless orchestration", "low cost" | Workflows |
| "query GCS Parquet with BQ governance" | BigLake |
| "high-performance PostgreSQL", "HTAP", "pgvector" | AlloyDB |
| "embeddings", "semantic search", "RAG", "vector similarity" | Vertex AI Embeddings + Vector Search |
