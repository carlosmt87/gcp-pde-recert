# BigQuery SQL Patterns — Quick Reference

Exam scenarios often require recognizing the correct SQL pattern, not just the correct
service. This sheet covers the patterns that appear most frequently.

---

## 1. Partition Management

### Query with partition pruning (always do this)
```sql
-- Good: partition filter applied — only scans relevant partitions
SELECT order_id, amount
FROM `project.dataset.orders`
WHERE DATE(order_date) = '2024-01-15';

-- Bad: no partition filter — full table scan
SELECT order_id, amount
FROM `project.dataset.orders`
WHERE EXTRACT(YEAR FROM order_date) = 2024;
```

### Overwrite a single partition (idempotent daily pipeline)
```sql
-- In bq CLI or job config:
-- destination: project.dataset.table$20240115
-- write_disposition: WRITE_TRUNCATE

-- Equivalent via DML:
DELETE FROM `project.dataset.orders`
WHERE DATE(order_date) = '2024-01-15';

INSERT INTO `project.dataset.orders`
SELECT * FROM `project.dataset.orders_staging`
WHERE DATE(order_date) = '2024-01-15';
```

### Check partition metadata
```sql
SELECT
  partition_id,
  total_rows,
  total_logical_bytes,
  last_modified_time
FROM `project.dataset.INFORMATION_SCHEMA.PARTITIONS`
WHERE table_name = 'orders'
ORDER BY partition_id DESC;
```

---

## 2. MERGE (Upsert Pattern)

The standard pattern for idempotent incremental loads:

```sql
MERGE `project.dataset.customers` AS target
USING `project.dataset.customers_staging` AS source
ON target.customer_id = source.customer_id

WHEN MATCHED AND source.updated_at > target.updated_at THEN
  UPDATE SET
    target.name = source.name,
    target.email = source.email,
    target.updated_at = source.updated_at

WHEN NOT MATCHED THEN
  INSERT (customer_id, name, email, updated_at)
  VALUES (source.customer_id, source.name, source.email, source.updated_at)

WHEN NOT MATCHED BY SOURCE THEN
  DELETE;  -- optional: remove records deleted from source
```

**Key exam points:**
- `WHEN MATCHED` — row exists in target; update it
- `WHEN NOT MATCHED` — row is new in source; insert it
- `WHEN NOT MATCHED BY SOURCE` — row only in target (deleted from source); delete it
- MERGE is atomic — safe to re-run (idempotent when keyed correctly)

---

## 3. Window Functions

```sql
-- ROW_NUMBER: deduplicate — keep latest record per customer
SELECT * EXCEPT(rn)
FROM (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY updated_at DESC
    ) AS rn
  FROM `project.dataset.customers`
)
WHERE rn = 1;

-- LAG/LEAD: compare to previous/next row (e.g., day-over-day change)
SELECT
  sale_date,
  revenue,
  LAG(revenue, 1) OVER (ORDER BY sale_date) AS prev_day_revenue,
  revenue - LAG(revenue, 1) OVER (ORDER BY sale_date) AS day_over_day_change
FROM `project.dataset.daily_sales`;

-- Running total
SELECT
  sale_date,
  revenue,
  SUM(revenue) OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM `project.dataset.daily_sales`;

-- Moving average (7-day)
SELECT
  sale_date,
  revenue,
  AVG(revenue) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS moving_avg_7d
FROM `project.dataset.daily_sales`;
```

---

## 4. INFORMATION_SCHEMA — Job History & Monitoring

```sql
-- Find slow queries in the last 24 hours
SELECT
  job_id,
  user_email,
  TIMESTAMP_DIFF(end_time, start_time, SECOND) AS duration_seconds,
  total_bytes_processed / POW(1024, 3) AS gb_processed,
  total_slot_ms / 1000 AS slot_seconds,
  query
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE
  creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
  AND job_type = 'QUERY'
  AND state = 'DONE'
ORDER BY duration_seconds DESC
LIMIT 20;

-- Find queries that scanned the most data
SELECT
  user_email,
  COUNT(*) AS query_count,
  SUM(total_bytes_processed) / POW(1024, 4) AS total_tb_processed
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY user_email
ORDER BY total_tb_processed DESC;

-- Find failed jobs
SELECT
  job_id,
  user_email,
  error_result.reason,
  error_result.message,
  query
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE
  creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
  AND error_result IS NOT NULL;

-- Table storage sizes
SELECT
  table_name,
  total_rows,
  total_logical_bytes / POW(1024, 3) AS size_gb,
  total_partitions
FROM `project.dataset.INFORMATION_SCHEMA.TABLE_STORAGE`
ORDER BY total_logical_bytes DESC;
```

---

## 5. BigQuery ML

### Train a model
```sql
-- Logistic regression (classification)
CREATE OR REPLACE MODEL `project.dataset.churn_model`
OPTIONS (
  model_type = 'LOGISTIC_REG',
  input_label_cols = ['churned'],
  auto_class_weights = TRUE
) AS
SELECT
  tenure_months,
  monthly_spend,
  support_tickets,
  churned
FROM `project.dataset.customer_features`
WHERE DATE(snapshot_date) < '2024-01-01';

-- Other model types: 'LINEAR_REG', 'KMEANS', 'XGBOOST',
--                   'AUTOML_CLASSIFIER', 'AUTOML_REGRESSOR'
```

### Evaluate a model
```sql
SELECT *
FROM ML.EVALUATE(MODEL `project.dataset.churn_model`,
  (SELECT tenure_months, monthly_spend, support_tickets, churned
   FROM `project.dataset.customer_features`
   WHERE DATE(snapshot_date) >= '2024-01-01')
);
```

### Batch prediction
```sql
SELECT
  customer_id,
  predicted_churned,
  predicted_churned_probs
FROM ML.PREDICT(MODEL `project.dataset.churn_model`,
  (SELECT customer_id, tenure_months, monthly_spend, support_tickets
   FROM `project.dataset.current_customers`)
);
```

### Generate embeddings (text)
```sql
-- Requires a remote model connected to Vertex AI
CREATE OR REPLACE MODEL `project.dataset.embedding_model`
REMOTE WITH CONNECTION `us.vertex-ai-connection`
OPTIONS (endpoint = 'text-embedding-005');

SELECT *
FROM ML.GENERATE_EMBEDDING(
  MODEL `project.dataset.embedding_model`,
  (SELECT product_id, description AS content FROM `project.dataset.products`)
);
```

### Gemini integration (ML.GENERATE_TEXT)
```sql
CREATE OR REPLACE MODEL `project.dataset.gemini_model`
REMOTE WITH CONNECTION `us.vertex-ai-connection`
OPTIONS (endpoint = 'gemini-1.5-flash');

SELECT
  ticket_id,
  ml_generate_text_result['candidates'][0]['content']['parts'][0]['text'] AS summary
FROM ML.GENERATE_TEXT(
  MODEL `project.dataset.gemini_model`,
  (SELECT
    ticket_id,
    CONCAT('Summarize this support ticket in one sentence: ', description) AS prompt
   FROM `project.dataset.support_tickets`),
  STRUCT(0.2 AS temperature, 100 AS max_output_tokens)
);
```

---

## 6. Vector Search

```sql
-- Store embeddings in a BigQuery table
-- Assumes embeddings column is ARRAY<FLOAT64> or JSON

-- Find top-5 most similar products to a query embedding
SELECT
  base.product_id,
  base.description,
  distance
FROM VECTOR_SEARCH(
  TABLE `project.dataset.product_embeddings`,
  'embedding',
  (SELECT embedding FROM `project.dataset.query_embeddings` WHERE query_id = 'q1'),
  top_k => 5,
  distance_type => 'COSINE'
);
```

---

## 7. Row-Level Security

```sql
-- Create a row access policy: each user sees only their region's rows
CREATE ROW ACCESS POLICY sales_region_filter
ON `project.dataset.sales`
GRANT TO ('group:emea-analysts@company.com')
FILTER USING (region = 'EMEA');

CREATE ROW ACCESS POLICY sales_region_filter_apac
ON `project.dataset.sales`
GRANT TO ('group:apac-analysts@company.com')
FILTER USING (region = 'APAC');

-- Grant full access to admins (no filter)
CREATE ROW ACCESS POLICY sales_admin_access
ON `project.dataset.sales`
GRANT TO ('group:data-admins@company.com')
FILTER USING (TRUE);
```

---

## 8. Authorized Views

```sql
-- Create a view that omits sensitive columns
CREATE OR REPLACE VIEW `project.dataset.orders_public` AS
SELECT
  order_id,
  product_id,
  quantity,
  order_date,
  region
  -- omits: customer_name, customer_email, payment_info
FROM `project.dataset.orders`;

-- Then authorize the view to access the source table:
-- In the source dataset settings, add the view as an authorized view.
-- (Done via Console or bq CLI, not SQL)
```

---

## 9. Time Travel & Table Snapshots

```sql
-- Query a table as it was 1 hour ago
SELECT *
FROM `project.dataset.orders`
FOR SYSTEM_TIME AS OF TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR);

-- Restore a table to a prior state (using DML + time travel)
CREATE OR REPLACE TABLE `project.dataset.orders` AS
SELECT *
FROM `project.dataset.orders`
FOR SYSTEM_TIME AS OF '2024-01-15 10:00:00 UTC';

-- Create a table snapshot (point-in-time copy, storage-efficient)
CREATE SNAPSHOT TABLE `project.dataset.orders_snapshot_20240115`
CLONE `project.dataset.orders`
FOR SYSTEM_TIME AS OF '2024-01-15 00:00:00 UTC'
OPTIONS (expiration_timestamp = TIMESTAMP '2024-02-15 00:00:00 UTC');
```

---

## 10. DML Patterns

```sql
-- DELETE with subquery
DELETE FROM `project.dataset.orders`
WHERE order_id IN (
  SELECT order_id FROM `project.dataset.cancelled_orders`
);

-- UPDATE with join
UPDATE `project.dataset.customers` c
SET c.tier = s.new_tier
FROM `project.dataset.tier_updates` s
WHERE c.customer_id = s.customer_id;

-- Conditional UPDATE
UPDATE `project.dataset.inventory`
SET
  stock_count = stock_count - quantity_sold,
  last_updated = CURRENT_TIMESTAMP()
WHERE product_id = 'P001'
  AND stock_count >= quantity_sold;
```

---

## 11. Useful Date/Time Patterns

```sql
-- Partition by ingestion date in queries
WHERE _PARTITIONDATE = DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)

-- Truncate to week/month for aggregation
SELECT
  DATE_TRUNC(sale_date, WEEK) AS week_start,
  SUM(revenue) AS weekly_revenue
FROM `project.dataset.sales`
GROUP BY week_start;

-- Generate a date spine (series of dates)
SELECT day
FROM UNNEST(
  GENERATE_DATE_ARRAY('2024-01-01', '2024-12-31', INTERVAL 1 DAY)
) AS day;

-- Parse and format timestamps
SELECT
  PARSE_TIMESTAMP('%Y-%m-%dT%H:%M:%SZ', raw_timestamp) AS parsed_ts,
  FORMAT_TIMESTAMP('%Y-%m-%d', event_time, 'America/New_York') AS local_date
FROM `project.dataset.events`;
```

---

## 12. Schema & Table Management

```sql
-- Add a column
ALTER TABLE `project.dataset.orders`
ADD COLUMN IF NOT EXISTS discount_pct FLOAT64;

-- Create a partitioned + clustered table
CREATE TABLE `project.dataset.events`
(
  event_id STRING,
  user_id STRING,
  event_type STRING,
  event_time TIMESTAMP,
  payload JSON
)
PARTITION BY DATE(event_time)
CLUSTER BY user_id, event_type
OPTIONS (
  partition_expiration_days = 365,
  require_partition_filter = TRUE
);

-- Table with row-level TTL (delete old rows automatically)
ALTER TABLE `project.dataset.events`
SET OPTIONS (
  partition_expiration_days = 90
);
```
