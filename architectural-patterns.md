# Architectural Patterns — GCP Data Engineer Renewal

Scenario-based exam questions frequently describe a business problem and ask which
*pattern* to apply. Know these by name, tradeoff, and GCP implementation.

---

## 1. ETL vs. ELT

### ETL (Extract → Transform → Load)
Transform data *before* loading it into the destination.

```
Source → [Extract] → [Transform] → [Load] → Destination
              ↑ Dataflow / Data Fusion apply logic here
```

**When to use:** Source data is too raw or large to load into the warehouse efficiently;
transformations require non-SQL logic; destination is not BigQuery.

**GCP implementation:** Dataflow or Cloud Data Fusion reading from a source, transforming
in-flight, and writing to BigQuery or another store.

**Tradeoffs:**
- ✅ Data arrives clean
- ❌ Reprocessing requires re-running the full pipeline
- ❌ Transformations tightly coupled to the pipeline code

---

### ELT (Extract → Load → Transform)
Load raw data first; transform *inside* the destination warehouse.

```
Source → [Extract + Load] → Destination → [Transform with SQL]
                                                ↑ Dataform / BigQuery SQL
```

**When to use:** BigQuery is the destination; team has SQL skills; raw data should be
preserved for reprocessing or auditing.

**GCP implementation:** Dataflow or Data Transfer Service loads raw data into a BigQuery
staging dataset; Dataform manages the SQL transformation DAG into curated datasets.

**Tradeoffs:**
- ✅ Raw data preserved; easy to reprocess
- ✅ Transformations are SQL — version-controlled in Dataform
- ❌ Raw data costs storage; warehouse must handle transformation compute

---

## 2. Lambda Architecture

Serves two layers in parallel to handle both speed and completeness.

```
Source (Pub/Sub)
    ├── Speed Layer  → Dataflow (streaming) → low-latency results (Bigtable / BQ streaming)
    └── Batch Layer  → Dataflow (batch)     → accurate, complete results (BigQuery)
                                                        ↓
                                              Serving Layer merges both
```

**When to use:** You need near-real-time results AND highly accurate historical
aggregations, and you can tolerate maintaining two separate pipelines.

**GCP implementation:**
- Speed layer: Pub/Sub → Dataflow (streaming) → Bigtable or BigQuery (streaming inserts)
- Batch layer: GCS (raw archive) → Dataflow (batch) → BigQuery partitioned tables
- Serving: BigQuery UNION or application logic merges views

**Tradeoffs:**
- ✅ Low latency for recent data; accurate batch for historical
- ❌ Two codebases to maintain (processing logic duplicated)
- ❌ Complex serving layer to merge outputs

---

## 3. Kappa Architecture

Streaming-only — no batch layer. Historical reprocessing done by replaying the stream.

```
Source (Pub/Sub)
    └── Single Streaming Pipeline → Dataflow → BigQuery / Bigtable
              ↑ replay from Pub/Sub retention or GCS archive for reprocessing
```

**When to use:** When the streaming layer can produce results accurate enough for all
use cases; simplicity is preferred over Lambda complexity.

**GCP implementation:**
- Pub/Sub with long message retention (up to 31 days) or GCS as a log archive
- Dataflow handles both real-time processing and historical replay
- One codebase, one pipeline

**Tradeoffs:**
- ✅ Single pipeline — no duplicated logic
- ✅ Simpler to maintain than Lambda
- ❌ Reprocessing large history through streaming can be slow
- ❌ Requires streaming-capable processing (Dataflow, not batch-only tools)

---

## 4. Medallion Architecture (Bronze / Silver / Gold)

A layered data quality pattern within a data lake or lakehouse.

```
Raw sources → [Bronze] → [Silver] → [Gold]
               (raw)      (cleaned)  (business-ready)
```

| Layer | Also Called | Contents | GCP Storage |
|-------|------------|----------|-------------|
| Bronze | Raw zone | Exact copy of source data, no transformation | Cloud Storage bucket (raw zone in Dataplex) |
| Silver | Curated zone | Validated, deduplicated, schema-enforced | Cloud Storage (curated zone) or BigQuery staging dataset |
| Gold | Refined / Consumption | Aggregated, modeled, business-logic applied | BigQuery datasets (fact/dim tables, Dataform outputs) |

**GCP implementation:** Dataplex lakes with raw and curated zones; Dataform manages
Bronze → Silver → Gold SQL transformations in BigQuery.

**Tradeoffs:**
- ✅ Clear data quality SLAs at each layer
- ✅ Easy to reprocess: always replay from Bronze
- ❌ Storage cost for maintaining multiple layers

---

## 5. Data Mesh

An organizational and architectural pattern treating data as a product owned by domain teams.

**Four principles:**
1. **Domain ownership** — each business domain owns its data end-to-end
2. **Data as a product** — data is treated with product thinking (SLAs, docs, discoverability)
3. **Self-serve infrastructure** — central platform team provides tooling; domains use it independently
4. **Federated governance** — central standards + policies, distributed enforcement

```
Domain A (GCP Project A)          Domain B (GCP Project B)
├── Dataplex Lake                  ├── Dataplex Lake
├── BigQuery datasets              ├── BigQuery datasets
└── Analytics Hub listings  ──→   └── Subscribe to Domain A listings

Central Platform Team
├── Dataplex Catalog (org-wide discovery)
├── Shared tag templates (data owner, sensitivity, domain)
├── Org-level IAM policies
└── Analytics Hub exchange
```

**GCP implementation:**
- Each domain: Dataplex lake in their GCP project; publish data products via Analytics Hub
- Central: Dataplex Catalog for cross-domain discovery; shared tag taxonomy; org-level policies

**Tradeoffs:**
- ✅ Scales across many domains without central bottleneck
- ✅ Domain teams accountable for their data quality
- ❌ Requires cultural change; domains must treat data as a product
- ❌ Governance coordination overhead

---

## 6. Lakehouse

Combines the scale and cost of a data lake (object storage) with the query performance
and governance of a data warehouse.

```
Cloud Storage (Parquet / ORC / Avro files)
         ↓
     BigLake layer
     (unified metadata + access control)
         ↓
  BigQuery query engine (SQL on files)
```

**GCP implementation:** BigLake tables defined over Cloud Storage files; BigQuery
queries them with full SQL; Policy Tags enforce column-level security on the files
themselves (not just BigQuery-native tables).

**Key distinction from a plain data lake:** A data lake stores files with no governance.
A lakehouse adds schema enforcement, ACID transactions (via formats like Iceberg/Delta),
and fine-grained access control directly on the files.

**Tradeoffs:**
- ✅ No data movement — query files in place
- ✅ BigQuery-style governance without copying data into BQ storage
- ✅ Cheaper storage than BigQuery native at massive scale
- ❌ Query performance lower than native BigQuery storage for hot data

---

## 7. Event-Driven / Streaming-First Architecture

Treat all data as a stream of events; batch is just a special case of stream processing.

```
Producers → Pub/Sub → Dataflow (streaming) → Sinks
                           ↓
                    Windowing, aggregations,
                    enrichment, stateful ops
                           ↓
               BigQuery | Bigtable | Cloud Storage
```

**Key Pub/Sub patterns:**

| Pattern | Implementation |
|---------|---------------|
| Fan-out (1 topic → N consumers) | Multiple subscriptions on one topic |
| Dead-letter queue | Configure dead-letter topic on subscription |
| Ordered processing | Enable message ordering; use ordering keys |
| At-least-once → exactly-once | Dataflow exactly-once + idempotent sinks |
| Consumer lag monitoring | `subscription/oldest_unacked_message_age` metric |

**When to use:** Low-latency requirements; decoupled producers/consumers; audit trails;
real-time ML inference triggers.

---

## 8. Serving Layer Patterns

How processed data is exposed to consumers.

| Pattern | GCP Implementation | Best For |
|---------|-------------------|----------|
| Batch serving | BigQuery tables / views | Analytics, reporting, ML batch inference |
| Online serving | Bigtable (key lookup) | App APIs, sub-10ms key-value lookups |
| Cached serving | Memorystore (Redis) | Frequently-accessed aggregates, sessions |
| Federated serving | BigQuery federated queries / BigLake | Query across storage systems without moving data |
| Shared data product | Analytics Hub linked datasets | Cross-team / cross-org data sharing |

---

## Pattern Selection Guide

```
Real-time results needed?
  ├── Yes, and accuracy matters for history too → Lambda (streaming + batch)
  └── Yes, and stream replay is acceptable for history → Kappa

All analytics in BigQuery?
  ├── Team uses SQL → ELT + Dataform (medallion layers)
  └── Team uses code → ETL with Dataflow

Multi-team / multi-domain organization?
  └── Data Mesh (Dataplex + Analytics Hub)

Want to query GCS files with BQ governance?
  └── Lakehouse with BigLake

High-throughput event processing?
  └── Event-driven: Pub/Sub → Dataflow → sinks
```
