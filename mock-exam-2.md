# Mock Exam 2 — GCP Professional Data Engineer Renewal

**30 questions | Suggested time: 60 minutes | Harder difficulty — many questions combine two topics**

Answers at the bottom. Cover everything below the `---` separator while working.

---

## Questions

**1.** A multinational company processes payroll data for employees across the EU and US. EU employee data must never leave EU regions; US data must stay in US regions. A single BigQuery project is used. Which combination of controls enforces this at the lowest operational overhead?

A) Create separate GCP projects per region with Organization Policy `gcp.resourceLocations` constraints; use separate BigQuery datasets per project
B) Store all data in a single BigQuery multi-region `EU` dataset; apply row-level security to filter US data
C) Use a single BigQuery dataset in the `US` multi-region; use column-level masking to hide EU fields from US users
D) Separate BigQuery datasets in `EU` and `US` multi-regions within the same project; enforce dataset-level IAM to prevent cross-region access

---

**2.** A company uses Cloud KMS CMEK for BigQuery. A security incident requires that all data encrypted with a specific key version be rendered immediately inaccessible — without deleting the underlying BigQuery tables. What is the fastest way to achieve this?

A) Delete the BigQuery dataset containing the tables
B) Disable or destroy the Cloud KMS key version used for encryption
C) Revoke all IAM permissions on the BigQuery dataset
D) Enable VPC Service Controls to block all BigQuery API calls

---

**3.** A data engineer is designing a pipeline to process medical records. Records contain a mix of structured fields (patient ID, diagnosis code) and free-text clinical notes containing PHI. The pipeline must store diagnosis codes in BigQuery for analytics and ensure clinical notes are never stored in plaintext anywhere. Which approach handles both requirements correctly?

A) Store all fields in BigQuery; apply column-level masking on the clinical notes column
B) Use Dataflow: extract structured fields → write to BigQuery; for clinical notes → call Cloud DLP de-identification (with crypto-based tokenization) → write tokenized notes to a separate secured table
C) Store raw records in Cloud Storage with CMEK encryption; query clinical notes via BigQuery external tables
D) Use Cloud Data Fusion to split the record; apply a regex masking rule on the notes field before writing to BigQuery

---

**4.** A streaming pipeline validates incoming records in real-time. Invalid records (missing required fields, wrong data types) should not be written to the main BigQuery table but must be preserved for later investigation and reprocessing. Which Dataflow pattern achieves this?

A) Filter invalid records and log them to Cloud Logging
B) Use a dead-letter Pub/Sub topic; publish invalid records there; write valid records to BigQuery
C) Use a Dataflow side output (additional PCollection) to route invalid records to a separate BigQuery error table while valid records go to the main table
D) Wrap the write in a try/catch block; on exception, skip the record and increment a counter

---

**5.** A data quality team needs to run automated checks weekly on a BigQuery table containing 10 billion rows. Checks include: no nulls in `order_id`, `amount` values within $0–$10,000, and `customer_id` referencing a valid entry in a customers table. Which service runs these checks without building a custom pipeline?

A) Write BigQuery Scheduled Queries that SELECT COUNT(*) for each rule and alert when counts exceed thresholds
B) Configure Dataplex data quality rules (not-null, range, referential integrity) on the BigQuery table; schedule weekly scans; publish results to BigQuery
C) Build a Dataflow pipeline that reads the full table weekly and runs validation logic in Python
D) Use Cloud DLP inspection jobs to scan for constraint violations

---

**6.** A company catalogues all data assets in Dataplex Catalog. Analysts report that when they search for "customer data," too many irrelevant results appear. The data governance team wants to add structured business metadata (data owner, business domain, PII flag) to help refine searches. What is the correct implementation?

A) Add descriptive BigQuery table descriptions using `ALTER TABLE SET OPTIONS (description=...)`
B) Create custom tag templates in Dataplex Catalog with fields for data owner, business domain, and PII flag; apply tags to assets; analysts filter search by tag values
C) Create a separate BigQuery metadata table mapping asset names to business attributes; query it for discovery
D) Use Cloud Asset Inventory custom metadata to annotate assets

---

**7.** A Dataform workspace manages 40 SQL transformation models for a BigQuery data warehouse. The team wants to ensure that when a new model is added, it is automatically tested against assertions before being merged and deployed to production. Which workflow achieves this?

A) Manually run `dataform run` in the console after each model is reviewed
B) Configure Cloud Build to trigger on pull request creation; run `dataform test` in the build step; block merge if assertions fail; on merge to main, trigger `dataform run` against the production compilation profile
C) Use BigQuery Scheduled Queries to run assertion SQL daily after deployment
D) Configure Dataform to auto-run on any file save in the workspace

---

**8.** A company stores raw event logs in Cloud Storage and transforms them into BigQuery tables via Dataflow. After a schema change in the source events, the team needs to understand which downstream BigQuery tables and Looker dashboards will be affected before making the change. Which GCP capability provides this impact analysis?

A) Query `INFORMATION_SCHEMA.COLUMN_FIELD_PATHS` in BigQuery to find all tables with the affected column
B) Search Dataplex Catalog for assets tagged with the source system name
C) Use Dataplex data lineage to trace the downstream impact from the Cloud Storage source through Dataflow to BigQuery tables and their consumers
D) Review Cloud Audit Logs to find which users and services recently accessed the source data

---

**9.** *(Section 2)* A pipeline reads from 5 different Cloud SQL databases (different schemas) and needs to join and aggregate the data before writing to BigQuery. The team wants a visual, low-code tool with pre-built connectors and the ability to run on a schedule. Which tool is most appropriate?

A) Dataflow with custom Python sources for each Cloud SQL instance
B) Dataproc with a PySpark job reading via JDBC
C) Cloud Data Fusion with Cloud SQL connectors, a Joiner transform, and a scheduled trigger
D) Dataform with federated queries across all 5 Cloud SQL instances

---

**10.** *(Section 2)* A company needs to deploy the same Dataflow pipeline to dev, staging, and production environments. Each environment uses different BigQuery datasets and Pub/Sub topics. The pipeline code must be identical across environments; only configuration differs. Which approach best supports this?

A) Maintain three separate copies of the pipeline code with hardcoded environment configs
B) Use Dataflow Flex Templates with runtime parameters for dataset names and topic paths; store environment-specific parameter files in Cloud Storage; deploy via Cloud Build
C) Use environment variables set on the Dataflow workers at launch time
D) Create three separate Cloud Composer DAGs, each with hardcoded environment configs for their respective environments

---

**11.** *(Section 2)* A Cloud Composer DAG runs a Dataflow job that processes files from Cloud Storage. The DAG should only trigger when a new file arrives in a specific GCS bucket prefix, not on a fixed schedule. Which design achieves this with the least operational overhead?

A) Poll the GCS bucket every minute using a `GCSObjectExistenceSensor` in a continuously running DAG
B) Use Eventarc to detect the Cloud Storage object creation event; trigger a Workflows execution or a direct Composer DAG run via the Airflow REST API
C) Use Cloud Scheduler to trigger the DAG every 5 minutes; check for new files at the start of the DAG
D) Configure a Cloud Storage notification to a Pub/Sub topic; run the DAG on a 1-minute schedule using a `PubSubSensor`

---

**12.** A startup needs to store user profile data for a mobile app. Requirements: document-based storage, real-time sync to mobile clients, offline support, and auto-scaling. Which service is correct?

A) Cloud SQL (PostgreSQL)
B) Bigtable
C) Firestore
D) AlloyDB

---

**13.** A data engineer is designing row key structure for a Bigtable table that stores website clickstream events. Events are written at ~500,000/sec. Analysts frequently query all events for a specific `session_id` within a time range. Which row key design is best?

A) `timestamp#session_id` — events ordered by time globally
B) `session_id#timestamp` — all events for a session are co-located; time range scans work efficiently
C) `user_id#timestamp` — events ordered by user and time
D) `random_uuid` — maximally distributes writes across nodes

---

**14.** A company currently runs analytics on Cloud SQL. Query times are degrading as data grows to 5 TB. The team wants to keep the PostgreSQL interface (existing BI tools use PostgreSQL drivers) but needs significantly better analytical query performance. Which migration path is most appropriate?

A) Add read replicas to Cloud SQL to distribute query load
B) Migrate to AlloyDB, which is PostgreSQL-compatible and provides significantly faster analytical query performance
C) Migrate to BigQuery and rewrite all SQL using BigQuery dialect
D) Add a Cloud Spanner instance and use federated queries from BigQuery

---

**15.** A company's data lake contains 3 years of raw log files in Cloud Storage (Parquet format). New data scientists want to run exploratory SQL queries on historical data without copying it into BigQuery. However, the security team requires that column-level access control be applied to mask `ip_address` and `user_agent` fields for non-privileged users. Which solution meets both requirements?

A) Create BigQuery external tables; apply authorized views to omit sensitive columns
B) Create BigLake tables on the Cloud Storage files; apply Policy Tags to `ip_address` and `user_agent` columns; configure dynamic data masking rules
C) Copy the Parquet files into BigQuery native storage; apply column-level security
D) Use Dataproc with Hive Metastore; apply Ranger policies for column masking

---

**16.** A data platform serves 20 internal teams via Analytics Hub. One team's data product listing was recently updated to add new columns. The platform team needs to understand which subscriber teams are actively using the linked dataset and how frequently before deprecating old columns. Which approach provides this information?

A) Email all 20 subscriber teams asking if they use the listing
B) Query `INFORMATION_SCHEMA.JOBS_BY_PROJECT` in each subscriber's project to see if queries reference the linked dataset
C) Review Cloud Audit Logs (Data Access logs) on the publisher's BigQuery dataset to see which linked datasets are being queried and by whom
D) Check Analytics Hub subscription metadata for last-access timestamps

---

**17.** A data engineer needs to implement a change data capture (CDC) pattern: as rows are inserted or updated in a Cloud SQL operational database, changes should appear in BigQuery within 5 minutes. Which architecture achieves this with the least custom code?

A) Set up a Cloud Composer DAG to run a `SELECT * WHERE updated_at > last_run` query every 5 minutes and MERGE results into BigQuery
B) Use Datastream to replicate Cloud SQL changes to Cloud Storage or BigQuery directly via CDC log reading; near-real-time latency
C) Use Cloud Data Fusion with a delta load pipeline on a 5-minute schedule
D) Write a Cloud SQL stored procedure that triggers an HTTP call to a Cloud Function which writes changes to BigQuery

---

**18.** A company is building a data mesh. Each of 8 domain teams publishes data products via Analytics Hub. The central data governance team needs to ensure all published datasets have: a data owner tag, a sensitivity classification tag, and pass a minimum data quality score before being published. How should this governance be enforced?

A) Send a monthly email to domain teams asking them to self-certify compliance
B) Use Dataplex Catalog tag templates for ownership and sensitivity; configure Dataplex data quality scans with a minimum score threshold; integrate both checks into a Cloud Build CI/CD pipeline that must pass before an Analytics Hub listing can be published
C) Require all domain teams to fill out a Google Form before publishing
D) Use Organization Policy to restrict Analytics Hub listing creation to specific approved projects only

---

**19.** A financial services company wants to run SQL analytics on data stored across BigQuery (transactions), Cloud Storage (Parquet audit logs), and AlloyDB (customer profiles) — all in a single query without ETL. Which BigQuery capability enables cross-source queries?

A) BigQuery Data Transfer Service to copy all data into BigQuery first
B) BigQuery federated queries: use BigQuery Omni for Cloud Storage, authorized BigQuery connections for AlloyDB, and native tables for transactions — JOIN across all three in one SQL statement
C) Cloud Data Fusion pipeline to merge all sources into a single BigQuery table nightly
D) Dataproc with Spark SQL reading from all three sources simultaneously

---

**20.** *(Section 4)* A data scientist wants to run unsupervised clustering on 50 million customer records in BigQuery to identify customer segments without writing Python code. Which approach is correct?

A) Export data to CSV; run scikit-learn KMeans locally; import cluster labels back to BigQuery
B) Use BigQuery ML: `CREATE MODEL` with `model_type='KMEANS'`; run `ML.PREDICT` to assign cluster labels; store results back in BigQuery
C) Use Vertex AI AutoML Tables to train a clustering model
D) Use Dataflow to compute pairwise distances and assign cluster labels

---

**21.** *(Section 4)* A company processes customer support calls (audio files in Cloud Storage) and needs to: (1) transcribe audio to text, (2) detect sentiment per transcript, (3) store results in BigQuery for analysis. Which managed pipeline achieves all three steps with no custom ML model development?

A) Deploy a custom speech-to-text model on Vertex AI; call Cloud Natural Language API; write to BigQuery
B) Use Dataflow: call Speech-to-Text API per audio file → call Cloud Natural Language API for sentiment → write structured output to BigQuery
C) Use Cloud Data Fusion with a custom audio processing plugin
D) Load audio files into BigQuery BLOB columns; use BigQuery ML remote models for transcription and sentiment

---

**22.** *(Section 4)* A company wants to build a semantic search system over their 500,000-document knowledge base stored as text in Cloud Storage. Users should be able to search in natural language and retrieve the most relevant documents. Which is the correct architecture?

A) Create a BigQuery full-text search index using `SEARCH()` function on a table containing document text
B) Extract text → chunk documents → generate embeddings with Vertex AI Embeddings API → store in Vertex AI Vector Search index → at query time: embed query → find top-k similar chunks → return source documents
C) Use Cloud Search (enterprise search) to index documents and expose a search API
D) Load documents into Firestore; use Firestore's built-in text search capability

---

**23.** *(Section 4)* A healthcare company needs to share de-identified patient datasets with 3 academic research institutions, each in a different GCP organization. The data must remain within GCP and researchers must pay for their own query costs. The company needs to track who is accessing what data. Which solution is most appropriate?

A) Copy the de-identified data to each research institution's GCP project using BigQuery Data Transfer Service
B) Publish the de-identified datasets via Analytics Hub; each institution subscribes to create a linked dataset in their own project; Cloud Audit Logs on the publisher dataset track access
C) Grant each institution's service account `roles/bigquery.dataViewer` on the source dataset via cross-organization IAM
D) Export data to Cloud Storage and share via signed URLs with 30-day expiry

---

**24.** *(Section 4)* A retail company uses Looker Studio connected to BigQuery for regional sales dashboards. The `sales` table has 3 billion rows and queries are timing out. The table is partitioned by `sale_date` and clustered by `region`. The dashboards always filter by the last 30 days and a specific region. Which optimization most directly improves dashboard query performance?

A) Add a BigQuery materialized view pre-aggregating sales by date and region; connect Looker Studio to the materialized view
B) Switch from BigQuery to Bigtable for dashboard queries
C) Enable BigQuery BI Engine with a memory reservation; BI Engine will cache dashboard queries in-memory for sub-second response
D) Move the table to BigQuery native storage from external tables

---

**25.** *(Section 5)* A company runs a Dataform pipeline that rebuilds 15 BigQuery tables daily. Some tables take 45 minutes to build; others take 2 minutes. Currently all tables are rebuilt sequentially. How should the engineer optimize this?

A) Split the 15 tables into separate Dataform workspaces and run them independently
B) Dataform automatically parallelizes table builds based on the dependency graph — ensure dependency declarations (`ref()`) are correctly defined so Dataform can identify which tables have no dependencies on each other and build them concurrently
C) Use Cloud Composer to trigger each table build as a separate DAG task with parallel task execution
D) Increase BigQuery slot reservations to process each table faster sequentially

---

**26.** *(Section 5)* A data team has a BigQuery reservation with 500 baseline slots. During month-end reporting, 12 heavy analytical jobs run simultaneously and all compete for slots, causing slowdowns. During normal days, the reservation is underutilized. What is the most cost-effective solution?

A) Purchase 1,000 more baseline slots to handle peak load
B) Configure autoscaling on the reservation: keep 500 baseline slots; set an autoscaling maximum of 2,000 slots; the reservation will burst automatically during month-end and scale back during normal operations
C) Schedule month-end reports to run sequentially, one at a time
D) Switch to on-demand pricing for the month-end period

---

**27.** *(Section 5)* A data pipeline writes partitioned data to BigQuery daily. The team wants an automated alert if a partition is missing (i.e., a day's data didn't load) within 2 hours of the expected completion time. Which monitoring approach achieves this?

A) Set a Cloud Monitoring uptime check on the pipeline endpoint
B) Create a BigQuery Scheduled Query that checks `INFORMATION_SCHEMA.PARTITIONS` for yesterday's partition; if missing, write to a log; set a log-based metric and Cloud Monitoring alert on that metric
C) Monitor the Dataflow job completion status in Cloud Monitoring; alert if the job doesn't appear in the last 24 hours
D) Set a BigQuery quota alert for daily loaded bytes falling below a threshold

---

**28.** *(Section 5)* A Cloud Composer environment is running Airflow 2.x. A DAG that processes 1,000 files fails when the Dataflow operator times out after 30 minutes — but the Dataflow job itself succeeds after 45 minutes. How should the engineer fix this without changing the underlying Dataflow job?

A) Increase the Composer environment's machine type to allow longer task execution
B) Increase the `execution_timeout` parameter on the Dataflow operator in the DAG; or switch to an asynchronous operator pattern that submits the Dataflow job and polls for completion separately
C) Split the 1,000 files into batches of 100 and process each batch as a separate DAG run
D) Add a `retries=3` parameter so the operator retries after each timeout

---

**29.** A data engineer notices that BigQuery on-demand query costs doubled last month. Using `INFORMATION_SCHEMA`, they find that 80% of the bytes processed came from 3 users running unoptimized queries with no partition filters. Beyond fixing the queries, which preventive control stops this from recurring for all future queries on that table?

A) Grant only `roles/bigquery.dataViewer` to the 3 users instead of `roles/bigquery.user`
B) Set `require_partition_filter = TRUE` on the table — BigQuery will reject any query that doesn't include a partition filter, forcing all users to prune partitions
C) Enable BigQuery audit logging to detect unfiltered queries in real-time
D) Apply a maximum bytes billed quota of 100 GB per user per day

---

**30.** A real-time fraud detection system processes 50,000 payment transactions per second. Each transaction must be scored by an ML model within 200ms. Feature computation (30-day aggregates per customer) is expensive. What architecture minimizes end-to-end latency?

A) For each transaction: query BigQuery in real-time to compute 30-day aggregates → call Vertex AI for scoring → return result
B) Pre-compute 30-day features for all customers nightly via batch job → store in Vertex AI Feature Store → at inference time: fetch features from Feature Store online serving (<10ms) → call Vertex AI Endpoint for scoring
C) For each transaction: compute features with a Dataflow streaming pipeline → write to Cloud SQL → query Cloud SQL for scoring
D) Use BigQuery ML `ML.PREDICT` at inference time — it handles feature computation and scoring in one SQL call within the 200ms SLA

---
---

## Answer Key

| Q | Answer | Topic |
|---|--------|-------|
| 1 | **D** | 1.1 — Data sovereignty + BigQuery regional datasets |
| 2 | **B** | 1.1 — CMEK key destruction for emergency access revocation |
| 3 | **B** | 1.1 + 1.2 — DLP de-identification in a Dataflow pipeline |
| 4 | **C** | 1.2 — Dataflow side outputs for error routing |
| 5 | **B** | 1.2 — Dataplex data quality rules |
| 6 | **B** | 1.3 — Dataplex Catalog tag templates |
| 7 | **B** | 1.2 + 2.3 — Dataform assertions in CI/CD via Cloud Build |
| 8 | **C** | 1.3 — Dataplex data lineage for impact analysis |
| 9 | **C** | 2.2 — Cloud Data Fusion for multi-source visual ETL |
| 10 | **B** | 2.3 — Dataflow Flex Templates for multi-environment CI/CD |
| 11 | **B** | 2.3 — Event-driven Composer triggering via Eventarc |
| 12 | **C** | 3.1 — Firestore for mobile/document/real-time sync |
| 13 | **B** | 3.1 — Bigtable row key design for session + time range |
| 14 | **B** | 3.1 — AlloyDB for high-performance PostgreSQL analytics |
| 15 | **B** | 3.1 + 3.3 — BigLake + Policy Tags on GCS Parquet files |
| 16 | **C** | 3.4 — Audit logs to track Analytics Hub linked dataset usage |
| 17 | **B** | 3.1 — Datastream for CDC from Cloud SQL to BigQuery |
| 18 | **B** | 3.4 — Dataplex governance + CI/CD gate for data mesh |
| 19 | **B** | 3.4 — BigQuery federated queries across multiple sources |
| 20 | **B** | 4.2 — BigQuery ML KMeans clustering |
| 21 | **B** | 4.2 — Dataflow + pre-trained APIs (Speech-to-Text + NLP) |
| 22 | **B** | 4.2 — RAG pipeline with Vertex AI embeddings + Vector Search |
| 23 | **B** | 4.3 — Analytics Hub for cross-org sharing with audit trail |
| 24 | **C** | 4.1 — BigQuery BI Engine for dashboard performance |
| 25 | **B** | 5.2 — Dataform parallel execution via dependency graph |
| 26 | **B** | 5.3 — BigQuery reservation autoscaling for peak loads |
| 27 | **B** | 5.4 — Log-based metrics + alerting for data freshness |
| 28 | **B** | 5.4 — Composer operator timeout configuration |
| 29 | **B** | 5.3 + 5.4 — `require_partition_filter` as a preventive control |
| 30 | **B** | 4.2 + 5.3 — Feature Store online serving for low-latency ML |

---

## Score & Diagnosis

| Score | Readiness |
|-------|-----------|
| 27–30 (90%+) | Exam-ready |
| 24–26 (80–89%) | Review weak sections; retake in a few days |
| 20–23 (67–79%) | Revisit subsection files for missed topics |
| <20 (<67%) | Re-study before sitting the exam |

## Per-Section Breakdown

| Section | Questions | My Score |
|---------|-----------|----------|
| 1 — Designing systems | 1–8 | /8 |
| 2 — Ingesting & processing | 9–11 | /3 |
| 3 — Storing | 12–19 | /8 |
| 4 — Analysis & ML | 20–24 | /5 |
| 5 — Maintaining | 25–30 | /6 |
| **Total** | | **/30** |
