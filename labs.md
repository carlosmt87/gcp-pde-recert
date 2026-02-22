# Hands-On Labs Map

Mapped to each exam subsection. All labs are on [Google Cloud Skills Boost](https://www.cloudskillsboost.google).
Search by the lab title to find it. Priority column indicates how closely the lab
matches renewal exam content: **High** = directly tested, **Medium** = supporting knowledge.

---

## Section 1 — Designing Data Processing Systems

### 1.1 Security and Compliance

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Getting Started with Cloud Data Loss Prevention | DLP inspection jobs, de-identification, info types |
| High | BigQuery: Qwik Start - Console | BigQuery basics, dataset creation with region selection |
| High | VPC Service Controls | Creating service perimeters, testing access restrictions |
| Medium | Cloud KMS: Qwik Start | Key rings, key versions, encryption/decryption |
| Medium | IAM: Qwik Start | Roles, service accounts, IAM policy binding |

### 1.2 Reliability and Fidelity

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Dataform: Qwik Start | Creating models, assertions, running workflows in BigQuery |
| High | Dataflow: Qwik Start - Templates | Running pre-built Dataflow templates |
| High | Building a Data Pipeline using Dataflow | Custom Beam pipelines, PCollections, transforms |
| Medium | Cloud Data Fusion: Qwik Start | Visual ETL pipeline creation, Wrangler |

### 1.3 Flexibility and Portability

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Dataplex: Qwik Start | Creating lakes, zones, and assets |
| High | Tagging Dataplex Assets | Tag templates, applying tags, searching catalog |
| High | Exploring Data with Dataplex | Data discovery, profiling, quality rules |
| Medium | Data Catalog: Qwik Start | Metadata search (predecessor to Dataplex Catalog) |

---

## Section 2 — Ingesting and Processing the Data

### 2.1 Planning the Data Pipelines

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Cloud Composer: Qwik Start | Creating a DAG, triggering Dataflow from Composer |
| High | Orchestrating the Cloud with Workflows | Workflow steps, conditions, Cloud Run integration |
| Medium | Pub/Sub: Qwik Start - Console | Topics, subscriptions, publishing and pulling messages |

### 2.2 Building the Pipelines

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Building a Streaming Data Pipeline using Dataflow | Pub/Sub → Dataflow → BigQuery streaming pipeline |
| High | Enrich a Dataflow Pipeline with AI | Calling AI APIs (NLP, Vision) within a Dataflow DoFn |
| High | Cloud Data Fusion: Building and Executing a Pipeline | Joiner, Aggregator, and scheduling |
| Medium | Using the Natural Language API from Google Docs | Natural Language API entity/sentiment extraction |

### 2.3 Deploying and Operationalizing the Pipelines

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | CI/CD for Data Processing | Cloud Build trigger, Dataflow Flex Template build and deploy |
| High | Dataflow Flex Templates: Qwik Start | Building a Flex Template, parameterized deployment |
| Medium | Continuous Delivery with Google Cloud Deploy | CD pipeline concepts applicable to data deployments |

---

## Section 3 — Storing the Data

### 3.1 Selecting Storage Systems

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | BigLake: Qwik Start | Creating BigLake tables on GCS; querying Parquet files |
| High | Cloud Bigtable: Qwik Start | Creating instances, tables, row key design |
| High | Cloud Spanner: Qwik Start | Instance creation, schema, global distribution |
| High | AlloyDB: Qwik Start | Cluster creation, PostgreSQL compatibility, performance |
| Medium | Cloud SQL: Qwik Start | MySQL/PostgreSQL instance setup, connections |
| Medium | Firestore: Qwik Start | Document collections, queries, real-time listeners |
| Medium | Cloud Storage: Qwik Start | Bucket creation, storage classes, lifecycle rules |
| Medium | Memorystore: Qwik Start | Redis instance, read/write operations |

### 3.3 Using a Data Lake

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Build a Data Lake on GCP | End-to-end: GCS landing zone → Dataplex → BigQuery |
| High | Dataplex: Manage Data in a Data Lake | Zone configuration, asset registration, IAM |
| High | Cloud Storage: Qwik Start - Cloud Console | Lifecycle rules, storage class transitions |

### 3.4 Designing for a Data Platform

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Analytics Hub: Qwik Start | Creating an exchange, publishing a listing, subscribing |
| High | Build a Data Mesh with Dataplex | Domain lakes, federated governance, cross-domain discovery |
| Medium | Dataplex: Implementing Data Lineage Tracking | Lineage graph, tracking across transformations |

---

## Section 4 — Preparing and Using Data for Analysis

### 4.1 Preparing Data for Visualization

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | BigQuery: Column-Level Security | Policy Tags, taxonomy, fine-grained reader role |
| High | BigQuery: Row-Level Security | Row access policies, group-based filtering |
| High | BigQuery: Dynamic Data Masking | Masking rules (nullify, hash, default value) |
| Medium | Looker Studio and BigQuery: Qwik Start | Connecting Looker Studio to BigQuery |

### 4.2 Preparing Data for AI and ML

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | BigQuery ML: Predict Visitor Purchases | CREATE MODEL, ML.PREDICT for logistic regression |
| High | Get Started with Vertex AI Feature Store | Feature definitions, batch ingestion, online serving |
| High | Vertex AI: Qwik Start | Training jobs, model registry, endpoints |
| High | Building a RAG Pipeline with Vertex AI | Document extraction, embedding, Vector Search, Gemini |
| High | Using Gemini with BigQuery | ML.GENERATE_TEXT, remote models, LLM-assisted SQL |
| Medium | Vertex AI Vector Search: Qwik Start | Index creation, querying with embeddings |
| Medium | Document AI: Qwik Start | PDF text extraction for unstructured data prep |

### 4.3 Sharing Data

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Analytics Hub: Publishing and Subscribing | Full publisher + subscriber workflow |
| Medium | Sharing BigQuery Data and Insights | Authorized views, dataset-level IAM |

---

## Section 5 — Maintaining and Automating Data Workloads

### 5.2 Designing Automation and Repeatability

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Cloud Composer: DAG Scheduling and Backfill | `schedule_interval`, `catchup`, Jinja templating |
| High | Dataform: Scheduling Workflow Invocations | Release configurations, cron scheduling |
| Medium | Cloud Scheduler: Qwik Start | Cron jobs, HTTP targets, Pub/Sub targets |

### 5.3 Organizing Workloads Based on Business Requirements

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | BigQuery Reservations | Creating reservations, assignments, slot commitments |
| High | Managing BigQuery Workloads with Reservations | Idle slot sharing, autoscaling, workload separation |
| Medium | BigQuery: Qwik Start - Command Line | `bq` CLI, job management |

### 5.4 Monitoring and Troubleshooting

| Priority | Lab Title | Key Skills |
|----------|-----------|-----------|
| High | Monitoring a Dataflow Pipeline | System lag, backlog metrics, Cloud Monitoring dashboards |
| High | BigQuery: Controlling Costs | INFORMATION_SCHEMA.JOBS, bytes processed analysis |
| High | Cloud Logging: Qwik Start | Log explorer, log-based metrics, sinks |
| Medium | Cloud Monitoring: Qwik Start | Dashboards, alerting policies, notification channels |
| Medium | Error Reporting: Qwik Start | Exception grouping, error trends |

---

## Recommended Learning Paths on Skills Boost

These curated paths bundle multiple labs in the right order:

| Path | Relevance |
|------|-----------|
| **Cloud Data Engineer** learning path | Directly aligned with PDE exam |
| **Preparing for the Professional Data Engineer Exam** | Exam-specific review course |
| **BigQuery for Data Warehousing** | Deep dive on BigQuery (25% of exam) |
| **Machine Learning on Google Cloud** | Vertex AI, BigQuery ML, feature engineering |
| **Data Governance on Google Cloud** | Dataplex, DLP, IAM, VPC SC |
