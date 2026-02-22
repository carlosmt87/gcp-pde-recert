# Mock Exam — GCP Professional Data Engineer Renewal

**30 questions | Suggested time: 60 minutes | Answers at the bottom**

Questions are weighted by section: S1 (8), S2 (3), S3 (8), S4 (7), S5 (4).
Cover the answer key below the line while working through questions.

---

## Questions

**1.** A company must store customer records for EU users exclusively within the EU to comply with GDPR. A data engineer is setting up the GCP project. Which two controls should they implement? *(Choose two)*

A) Create BigQuery datasets in the `EU` multi-region
B) Create BigQuery datasets in the `US` multi-region and use authorized views to restrict access
C) Apply an Organization Policy with `gcp.resourceLocations` constraint limiting resource creation to EU regions
D) Enable Cloud DLP on all datasets to redact PII fields
E) Use VPC Service Controls to prevent data from leaving the EU project

---

**2.** A healthcare organization needs to prove to auditors which users accessed PHI data in BigQuery during the past 90 days, including what queries were run. Which GCP service provides this evidence?

A) Cloud Monitoring with custom metrics
B) BigQuery INFORMATION_SCHEMA.JOBS_BY_PROJECT
C) Cloud Audit Logs (Data Access logs for BigQuery)
D) BigQuery Admin Panel job history

---

**3.** A data pipeline must ensure that encryption keys for BigQuery are rotated every 90 days and controlled by the company, not Google. Which solution is correct?

A) Enable Google-managed encryption on the BigQuery dataset
B) Use BigQuery column encryption with AEAD SQL functions
C) Configure CMEK via Cloud KMS on the BigQuery dataset with a 90-day rotation schedule
D) Store data in Cloud Storage with server-side encryption and query via external tables

---

**4.** A team needs to clean and validate customer records before loading into BigQuery. Validation rules include: no nulls in `customer_id`, email format matching a regex, and row counts matching the source. The team uses SQL and wants the checks version-controlled and automated. Which tool fits best?

A) Cloud Data Fusion with a custom validation plugin
B) Dataform assertions defined alongside the transformation SQL
C) A Dataflow pipeline with custom Python validation logic
D) BigQuery Scheduled Queries with manual COUNT checks

---

**5.** A streaming Dataflow pipeline is processing IoT sensor events. Events can arrive up to 15 minutes late due to network delays. The pipeline computes 30-minute totals per device. Which configuration handles late data correctly?

A) Fixed 30-minute windows; discard all late elements
B) Session windows with a 15-minute gap duration
C) Fixed 30-minute windows with allowed lateness of 15 minutes
D) Sliding windows of 15 minutes sliding every 30 minutes

---

**6.** A company's data lake on Cloud Storage has grown to 500+ buckets across 20 teams. Analysts cannot find relevant datasets and there is no consistent tagging or access control. Which GCP service most directly solves all three problems?

A) Create a centralized BigQuery dataset and copy all GCS files into tables
B) Implement Dataplex with lakes and zones; enable automatic discovery; manage IAM at the zone level
C) Configure a Cloud Storage bucket inventory export to BigQuery for tracking
D) Use Cloud Asset Inventory to list all buckets

---

**7.** A financial firm needs column-level security on a BigQuery table so that the `account_number` column is only visible to users with the `finance-privileged` group membership, while all other users see NULL. Which feature implements this without duplicating tables?

A) Create separate BigQuery views omitting the column for unprivileged users
B) Apply a row access policy filtering by user group
C) Configure BigQuery dynamic data masking with a Policy Tag and a "nullify" masking rule; grant fine-grained reader to `finance-privileged`
D) Use Cloud DLP to scan and redact the column daily

---

**8.** A data engineer needs to enable self-service discovery of data assets across 10 GCP projects for business analysts. Analysts should be able to search for tables by business domain, sensitivity level, and data owner. Which is the most appropriate solution?

A) Maintain a shared spreadsheet with dataset metadata updated monthly
B) Create BigQuery table descriptions using `ALTER TABLE SET OPTIONS` and use INFORMATION_SCHEMA for search
C) Define tag templates in Dataplex Catalog; apply tags to assets; grant analysts the Catalog Viewer role
D) Use Cloud Asset Inventory with a custom metadata export pipeline

---

**9.** *(Section 2)* A pipeline reads raw JSON events from Pub/Sub, extracts named entities from text fields using a pre-trained model, and writes enriched records to BigQuery. Which architecture has the least operational overhead?

A) Dataproc cluster running a PySpark job with Spark NLP
B) A Dataflow pipeline with a DoFn calling the Cloud Natural Language API per element
C) Cloud Data Fusion with a custom NLP Java plugin
D) A Cloud Run service triggered by Pub/Sub that calls the NLP API and writes to BigQuery

---

**10.** *(Section 2)* A team wants to deploy their Dataflow pipeline automatically whenever code is merged to `main` in GitHub. The pipeline should be tested in a staging environment before promotion to production. Which services form the correct CI/CD chain?

A) Cloud Scheduler → Cloud Functions → Dataflow
B) GitHub → Cloud Build trigger → run tests → build Flex Template → deploy to staging → promote to prod
C) GitHub → Cloud Composer DAG that runs `gcloud dataflow jobs run`
D) Cloud Source Repositories → Artifact Registry → manual `gcloud` deploy

---

**11.** *(Section 2)* A data pipeline orchestrated by Cloud Composer runs 8 sequential tasks across Dataflow, BigQuery, and Cloud Storage. Task 4 must only run if Task 3 produced more than 1,000 output rows. Which Airflow feature handles this?

A) `TimeSensor` waiting for a fixed delay after Task 3
B) `BranchPythonOperator` checking the row count and routing to Task 4 or a skip task
C) Setting `retries=0` on Task 4 so it fails fast if conditions aren't met
D) Using a `TriggerDagRunOperator` to launch a separate DAG for Task 4

---

**12.** A company needs to store 5 years of financial transaction records for regulatory compliance. Records are accessed heavily for the first 30 days, occasionally up to 1 year, and almost never after that. Which Cloud Storage configuration minimizes cost?

A) Store all data in Standard storage; delete after 5 years
B) Lifecycle rule: transition to Nearline at 30 days, Coldline at 365 days, Archive at 1825 days (5 years), then delete
C) Store all data in Archive storage from the start
D) Use Nearline storage with no lifecycle rules; manually delete old data

---

**13.** A global e-commerce company needs sub-10ms inventory lookups for 2 billion product SKUs. Inventory is updated by thousands of warehouses concurrently. Queries are always by a single SKU key. Which storage service is correct?

A) BigQuery with a clustered table on `sku_id`
B) Cloud SQL with an index on `sku_id`
C) Cloud Spanner with a primary key on `sku_id`
D) Bigtable with a row key of `sku_id`

---

**14.** A data engineering team wants to query Parquet files stored in Cloud Storage using BigQuery SQL, while also enforcing column-level access control on sensitive columns in those files — without copying data into BigQuery native storage. Which feature enables this?

A) BigQuery external tables (standard)
B) BigQuery federated queries against Cloud Storage
C) BigLake tables on Cloud Storage with Policy Tags
D) Dataproc with Hive Metastore pointing at the GCS bucket

---

**15.** A data platform team wants to share a curated BigQuery dataset with 50 internal teams across 15 GCP projects. Teams should be able to self-discover and subscribe to the dataset. The platform team must maintain ownership and schema control. Which approach scales best?

A) Grant `roles/bigquery.dataViewer` to each team's service account individually
B) Publish the dataset as a listing in an Analytics Hub exchange; teams subscribe to create linked datasets
C) Export the dataset to Cloud Storage and share the bucket with all teams
D) Create authorized views in each team's project

---

**16.** A retail company stores point-of-sale transaction data in BigQuery partitioned by `sale_date`. A nightly pipeline appends the previous day's data. A bug caused duplicate records on 3 specific dates. After fixing the bug, how should the engineer safely reprocess only those 3 dates?

A) Truncate the entire table and reload all historical data
B) Run `DELETE FROM table WHERE sale_date IN (...)` then re-append the 3 days
C) Use `WRITE_TRUNCATE` with partition decorator to overwrite each affected date partition individually
D) Use `INSERT OVERWRITE` to replace the entire table with corrected data

---

**17.** A company uses Dataplex to manage their data lake. Raw zone contains unvalidated logs; curated zone contains validated, business-ready datasets. A new analyst team should read curated data but must not access raw zone data containing PII. How should access be configured?

A) Grant `roles/storage.objectViewer` on curated zone buckets; deny on raw zone buckets
B) Grant `roles/dataplex.dataReader` at the curated zone level; grant no permissions on the raw zone
C) Use a BigQuery authorized view of curated data for the analyst team
D) Apply VPC Service Controls to block raw zone access by IP

---

**18.** A data platform needs to support SQL analytics on BigQuery tables AND on Parquet files in Cloud Storage, with unified data lineage and governance across both. Which combination of services forms the correct foundation?

A) Cloud Storage + Dataproc + Data Catalog (legacy)
B) Cloud Storage + BigQuery + Dataplex + BigLake
C) Cloud Storage + BigQuery + Cloud Monitoring + IAM
D) AlloyDB + BigQuery + Pub/Sub + Cloud Composer

---

**19.** A company wants to implement a federated data governance model where each of 5 business units owns and manages their data independently in separate GCP projects, but a central team can enforce data quality standards and make all assets discoverable company-wide. Which architecture is most appropriate?

A) Centralize all data in one BigQuery project managed by the governance team
B) Each domain creates a Dataplex lake in their project; central team uses shared Dataplex Catalog tag templates, quality rules, and an Analytics Hub exchange
C) Use separate Cloud Storage buckets per domain with cross-project IAM bindings
D) Replicate all domain data to a central project nightly using Cloud Composer

---

**20.** *(Section 4)* A fraud detection model uses a 30-day rolling average transaction amount as a key feature. The data science team reports that model performance is worse in production than in testing, despite using the same raw data. What is the most likely cause and solution?

A) The model is overfitting; retrain with more regularization
B) Training-serving skew — features are computed differently at train vs. serve time; implement Vertex AI Feature Store to ensure consistent feature computation for both
C) Insufficient training data; add more historical records
D) The serving latency is too high; optimize the prediction endpoint

---

**21.** *(Section 4)* A company needs to build a RAG-based chatbot that answers questions using proprietary internal documents stored as PDFs in Cloud Storage. Which sequence correctly implements the retrieval component?

A) Load PDFs into BigQuery BLOB columns; query at runtime with SQL; pass to Gemini
B) Extract text via Document AI → chunk → generate embeddings with Vertex AI Embeddings API → store in Vertex AI Vector Search → at query time embed question → retrieve top-k → pass to Gemini
C) Convert PDFs to images → store in GCS → use Gemini Vision to read all PDFs at query time
D) Index PDFs with Cloud Search → retrieve via Cloud Search API → pass results to Gemini

---

**22.** *(Section 4)* A data analyst team uses Looker Studio dashboards connected to BigQuery. Regional managers should automatically see only their own region's data without the dashboard developer building separate dashboards per region. Which BigQuery feature enables this?

A) Create one authorized view per region and connect each view to a separate dashboard
B) Use column-level security with Policy Tags to hide non-relevant region columns
C) Implement BigQuery row-level security with a row access policy filtering by the querying user's group/region attribute
D) Use Cloud DLP to redact rows belonging to other regions before the dashboard queries

---

**23.** *(Section 4)* A financial data vendor wants to distribute daily market data to 300 enterprise customers, each on GCP. Customers should query the data in their own BigQuery projects, paying their own query costs, without the vendor duplicating data 300 times. Which solution is correct?

A) Grant each customer's service account `roles/bigquery.dataViewer` on the vendor's dataset
B) Publish the dataset as an Analytics Hub listing; customers subscribe and receive linked datasets in their own projects — zero-copy, customer-billed queries
C) Use BigQuery Data Transfer Service to copy the dataset to each customer's project daily
D) Export the dataset to Cloud Storage and share the bucket publicly

---

**24.** *(Section 4)* A machine learning pipeline prepares customer features in BigQuery for both model training (monthly batch) and real-time fraud scoring (sub-100ms latency). Which architecture correctly serves both use cases?

A) Export features from BigQuery to a CSV for training; build a separate real-time feature computation service for scoring
B) Use Vertex AI Feature Store: batch export for training jobs; online serving for real-time scoring — using the same feature definitions
C) Store features in Cloud SQL for training; cache in Memorystore for scoring
D) Use BigQuery ML for both training and real-time inference

---

**25.** *(Section 5)* A daily pipeline appends sales data to a BigQuery partitioned table. After a bug fix, the team needs to safely re-run the pipeline for any past date without producing duplicates. Which design pattern makes the pipeline idempotent?

A) Use `WRITE_APPEND` mode; de-duplicate with a follow-up SQL job
B) Use `WRITE_TRUNCATE` at the partition level (partition decorator overwrite) for each run date
C) Drop the entire table before each run and reload all historical data
D) Use `MERGE` statements only when the date is explicitly in the past 7 days

---

**26.** *(Section 5)* A company runs BigQuery for two workloads: interactive BI dashboards (business hours, latency-sensitive) and nightly batch ETL (11pm–3am, tolerates slower performance). They use on-demand pricing and BI queries are slow during busy periods. What resolves this at lowest cost?

A) Switch all queries to on-demand with higher concurrent slot limits
B) Purchase BigQuery Enterprise Edition slots; create separate BI and ETL reservations with idle slot sharing from ETL to BI during business hours
C) Schedule BI dashboards to query only during off-peak hours
D) Use BigQuery BI Engine for dashboards; keep ETL on on-demand

---

**27.** *(Section 5)* A data engineer needs to find all BigQuery queries from the past 7 days that took longer than 5 minutes to execute, the user who ran them, and the bytes processed. Which is the most efficient approach?

A) Manually review the BigQuery Admin Panel job history and export to CSV
B) Set up a Cloud Monitoring alert for slow queries and check historical alerts
C) Query `INFORMATION_SCHEMA.JOBS_BY_PROJECT` filtering on `total_slot_ms`, `creation_time`, and `user_email`
D) Enable BigQuery Data Access audit logs and search Cloud Logging with a filter

---

**28.** *(Section 5)* A Cloud Composer DAG fails at task 5 of 10. The engineer needs to identify the root cause. Task 5 calls a Dataflow job. What is the most effective troubleshooting sequence?

A) Restart the full DAG run and monitor if it fails again
B) In the Airflow UI, click the failed task → view logs; check the Dataflow job UI for the specific job launched by task 5; cross-reference with Cloud Logging using the Dataflow job ID
C) Check Cloud Monitoring for Composer environment CPU utilization
D) Query `INFORMATION_SCHEMA.JOBS_BY_PROJECT` for slow BigQuery queries in the same time window

---

**29.** A startup uses BigQuery on-demand pricing. A data scientist accidentally ran a query that scanned 50 TB, generating a $250 charge. The team wants to prevent individual queries from scanning more than 1 TB. Which control implements this?

A) Switch to slot reservations with a 1 TB autoscaling maximum
B) Set a custom quota for `bigquery.googleapis.com/quota/query/usage` in Cloud Quotas
C) Configure a maximum bytes billed limit per query using `maximumBytesBilled` in the query job configuration, or enforce it via an Organization Policy
D) Use VPC Service Controls to block queries larger than 1 TB

---

**30.** A company ingests real-time events from Pub/Sub into BigQuery via Dataflow. Users report that data visible in BigQuery dashboards is increasingly delayed. The delay started after recent traffic doubled. Which metric and action address this?

A) Check BigQuery `INFORMATION_SCHEMA.JOBS_BY_PROJECT` for slow queries; add clustering to the BigQuery table
B) Check Dataflow `system_lag` and `backlog_elements`; if lag is growing, increase Dataflow max workers to scale up processing throughput
C) Check Pub/Sub `topic/send_message_count`; reduce the publisher rate
D) Check Cloud Composer DAG run duration; optimize task dependencies

---
---

## Answer Key

| Q | Answer | Section |
|---|--------|---------|
| 1 | **A, C** | 1.1 |
| 2 | **C** | 1.1 |
| 3 | **C** | 1.1 |
| 4 | **B** | 1.2 |
| 5 | **C** | 1.2 |
| 6 | **B** | 1.3 |
| 7 | **C** | 1.3 |
| 8 | **C** | 1.3 |
| 9 | **B** | 2.2 |
| 10 | **B** | 2.3 |
| 11 | **B** | 2.1 |
| 12 | **B** | 3.3 |
| 13 | **D** | 3.1 |
| 14 | **C** | 3.1 |
| 15 | **B** | 4.3 |
| 16 | **C** | 5.2 |
| 17 | **B** | 3.3 |
| 18 | **B** | 3.4 |
| 19 | **B** | 3.4 |
| 20 | **B** | 4.2 |
| 21 | **B** | 4.2 |
| 22 | **C** | 4.1 |
| 23 | **B** | 4.3 |
| 24 | **B** | 4.2 |
| 25 | **B** | 5.2 |
| 26 | **B** | 5.3 |
| 27 | **C** | 5.4 |
| 28 | **B** | 5.4 |
| 29 | **C** | 5.3 |
| 30 | **B** | 5.4 |

---

## Score Interpretation

| Score | Readiness |
|-------|-----------|
| 27–30 (90%+) | Strong — ready to schedule the exam |
| 24–26 (80–89%) | Good — review missed sections and retake |
| 20–23 (67–79%) | Moderate — focused study needed on weak sections |
| <20 (<67%) | More study needed — revisit section files |

## Scoring by Section

After marking answers, tally by section to identify weak areas:

| Section | Questions | My Score |
|---------|-----------|----------|
| 1 — Designing systems | 1–8 | /8 |
| 2 — Ingesting & processing | 9–11 | /3 |
| 3 — Storing | 12–19 | /8 |
| 4 — Analysis & ML | 20–24 | /5 |
| 5 — Maintaining | 25–30 | /6 |
| **Total** | | **/30** |
