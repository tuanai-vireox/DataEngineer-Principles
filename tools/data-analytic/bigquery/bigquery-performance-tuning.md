# BigQuery Performance Tuning

This document provides comprehensive guidance on optimizing BigQuery performance for data engineering workloads, covering query optimization, data modeling, and cost management strategies.

## 🎯 Overview

BigQuery performance tuning involves optimizing various aspects of queries, data layout, and resource utilization to achieve better performance, lower costs, and efficient data processing. This guide covers the key areas that impact BigQuery performance.

## ⚙️ Query Optimization

### 1. **Query Structure Optimization**

#### Optimize SELECT Statements
```sql
-- Use specific column names instead of SELECT *
SELECT id, name, amount, category
FROM sales_data
WHERE date >= '2023-01-01';

-- Instead of
SELECT *
FROM sales_data
WHERE date >= '2023-01-01';
```

#### Optimize WHERE Clauses
```sql
-- Apply filters early and use appropriate data types
SELECT id, name, amount
FROM sales_data
WHERE date >= DATE('2023-01-01')  -- Use DATE function
  AND amount > 1000
  AND category = 'Electronics';

-- Use IN instead of multiple OR conditions
SELECT id, name, amount
FROM sales_data
WHERE category IN ('Electronics', 'Clothing', 'Books');

-- Instead of
SELECT id, name, amount
FROM sales_data
WHERE category = 'Electronics' 
   OR category = 'Clothing' 
   OR category = 'Books';
```

#### Optimize JOIN Operations
```sql
-- Use appropriate JOIN types
SELECT a.id, b.name, c.category
FROM sales_data a
INNER JOIN customers b ON a.customer_id = b.id
INNER JOIN categories c ON a.category_id = c.id;

-- Use JOIN hints for optimization
SELECT /*+ JOIN_ORDER(a, b, c) */
  a.id, b.name, c.category
FROM large_table a
JOIN medium_table b ON a.id = b.id
JOIN small_table c ON b.category_id = c.id;
```

### 2. **Aggregation Optimization**

#### Use APPROX Functions
```sql
-- Use approximate functions for better performance
SELECT 
  category,
  APPROX_COUNT_DISTINCT(customer_id) as unique_customers,
  APPROX_QUANTILES(amount, 100)[OFFSET(50)] as median_amount
FROM sales_data
GROUP BY category;

-- Instead of exact functions
SELECT 
  category,
  COUNT(DISTINCT customer_id) as unique_customers,
  PERCENTILE_CONT(amount, 0.5) OVER (PARTITION BY category) as median_amount
FROM sales_data
GROUP BY category;
```

#### Optimize GROUP BY
```sql
-- Use HAVING instead of WHERE for aggregated data
SELECT category, COUNT(*) as count, SUM(amount) as total
FROM sales_data
GROUP BY category
HAVING COUNT(*) > 100;

-- Use ROLLUP for hierarchical aggregations
SELECT 
  region,
  category,
  COUNT(*) as count,
  SUM(amount) as total
FROM sales_data
GROUP BY ROLLUP(region, category);
```

### 3. **Window Function Optimization**

#### Use PARTITION BY Effectively
```sql
-- Use PARTITION BY to reduce computation scope
SELECT 
  id,
  name,
  amount,
  ROW_NUMBER() OVER (PARTITION BY category ORDER BY amount DESC) as rank
FROM sales_data;

-- Instead of global window
SELECT 
  id,
  name,
  amount,
  ROW_NUMBER() OVER (ORDER BY amount DESC) as rank
FROM sales_data;
```

#### Optimize Window Function Performance
```sql
-- Use appropriate window frame
SELECT 
  id,
  name,
  amount,
  SUM(amount) OVER (
    PARTITION BY category 
    ORDER BY date 
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) as rolling_7day_sum
FROM sales_data;
```

## 📊 Data Layout Optimization

### 1. **Partitioning Strategies**

#### Date Partitioning
```sql
-- Create date-partitioned table
CREATE TABLE sales_data_partitioned
(
  id INT64,
  name STRING,
  amount NUMERIC,
  category STRING,
  region STRING,
  date DATE
)
PARTITION BY DATE(date)
OPTIONS(
  partition_expiration_days = 365,
  require_partition_filter = true
);

-- Query with partition filter
SELECT category, COUNT(*) as count
FROM sales_data_partitioned
WHERE date >= '2023-01-01'  -- Partition filter
  AND date < '2023-02-01'
GROUP BY category;
```

#### Integer Range Partitioning
```sql
-- Create integer range partitioned table
CREATE TABLE sales_data_int_partitioned
(
  id INT64,
  name STRING,
  amount NUMERIC,
  category STRING,
  region STRING,
  date DATE
)
PARTITION BY RANGE_BUCKET(id, GENERATE_ARRAY(0, 1000000, 10000))
OPTIONS(require_partition_filter = true);

-- Query with partition filter
SELECT category, COUNT(*) as count
FROM sales_data_int_partitioned
WHERE id >= 10000 AND id < 20000  -- Partition filter
GROUP BY category;
```

### 2. **Clustering Strategies**

#### Single Column Clustering
```sql
-- Create clustered table
CREATE TABLE sales_data_clustered
(
  id INT64,
  name STRING,
  amount NUMERIC,
  category STRING,
  region STRING,
  date DATE
)
CLUSTER BY category;

-- Queries on clustered column are more efficient
SELECT category, COUNT(*) as count
FROM sales_data_clustered
WHERE category = 'Electronics'  -- Uses clustering
GROUP BY category;
```

#### Multi-Column Clustering
```sql
-- Create multi-column clustered table
CREATE TABLE sales_data_multi_clustered
(
  id INT64,
  name STRING,
  amount NUMERIC,
  category STRING,
  region STRING,
  date DATE
)
CLUSTER BY category, region;

-- Queries on clustered columns are more efficient
SELECT category, region, COUNT(*) as count
FROM sales_data_multi_clustered
WHERE category = 'Electronics' 
  AND region = 'North America'  -- Uses clustering
GROUP BY category, region;
```

#### Combined Partitioning and Clustering
```sql
-- Best of both worlds
CREATE TABLE sales_data_optimized
(
  id INT64,
  name STRING,
  amount NUMERIC,
  category STRING,
  region STRING,
  date DATE
)
PARTITION BY DATE(date)
CLUSTER BY category, region
OPTIONS(
  partition_expiration_days = 365,
  require_partition_filter = true
);

-- Highly optimized queries
SELECT category, region, COUNT(*) as count
FROM sales_data_optimized
WHERE date >= '2023-01-01'  -- Partition filter
  AND category = 'Electronics'  -- Clustering
  AND region = 'North America'  -- Clustering
GROUP BY category, region;
```

### 3. **Data Type Optimization**

#### Use Appropriate Data Types
```sql
-- Use appropriate data types for better performance
CREATE TABLE optimized_table
(
  id INT64,                    -- Use INT64 for integers
  name STRING,                 -- Use STRING for text
  amount NUMERIC,              -- Use NUMERIC for precise decimals
  price FLOAT64,               -- Use FLOAT64 for approximate decimals
  is_active BOOL,              -- Use BOOL for boolean values
  created_at TIMESTAMP,        -- Use TIMESTAMP for timestamps
  metadata JSON                -- Use JSON for structured data
);
```

#### Optimize String Operations
```sql
-- Use efficient string operations
SELECT 
  id,
  name,
  -- Use STRPOS instead of REGEXP_CONTAINS when possible
  CASE 
    WHEN STRPOS(name, 'Electronics') > 0 THEN 'Electronics'
    WHEN STRPOS(name, 'Clothing') > 0 THEN 'Clothing'
    ELSE 'Other'
  END as category
FROM sales_data;

-- Use CONCAT instead of || for better performance
SELECT 
  id,
  CONCAT(first_name, ' ', last_name) as full_name
FROM customers;
```

## 💰 Cost Optimization

### 1. **Query Cost Management**

#### Use LIMIT to Reduce Costs
```sql
-- Use LIMIT to reduce data processing
SELECT id, name, amount
FROM sales_data
ORDER BY amount DESC
LIMIT 100;  -- Process only top 100 records

-- Use LIMIT in subqueries
SELECT category, count
FROM (
  SELECT category, COUNT(*) as count
  FROM sales_data
  GROUP BY category
  ORDER BY count DESC
  LIMIT 10
);
```

#### Use Caching Effectively
```sql
-- Enable query result caching
SELECT /*+ USE_CACHED_RESULT */
  category, COUNT(*) as count
FROM sales_data
GROUP BY category;

-- Check cache hit rate
SELECT
  job_id,
  creation_time,
  query,
  cache_hit,
  total_bytes_processed
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY creation_time DESC;
```

### 2. **Storage Cost Optimization**

#### Use Appropriate Storage Classes
```sql
-- Use long-term storage for infrequently accessed data
CREATE TABLE archive_data
(
  id INT64,
  name STRING,
  amount NUMERIC,
  date DATE
)
PARTITION BY DATE(date)
OPTIONS(
  partition_expiration_days = 365,
  require_partition_filter = true
);

-- Move old data to long-term storage
ALTER TABLE archive_data
SET OPTIONS(
  partition_expiration_days = 365
);
```

#### Optimize Data Compression
```sql
-- Use efficient data formats
-- BigQuery automatically compresses data
-- Use appropriate data types to minimize storage

-- Example: Use DATE instead of STRING for dates
CREATE TABLE optimized_dates
(
  id INT64,
  event_date DATE,        -- More efficient than STRING
  event_timestamp TIMESTAMP
);
```

### 3. **Slot Usage Optimization**

#### Monitor Slot Usage
```sql
-- Monitor slot usage
SELECT
  job_id,
  creation_time,
  query,
  total_slot_ms,
  total_bytes_processed,
  total_bytes_billed,
  -- Calculate slot efficiency
  total_slot_ms / (total_bytes_processed / 1024 / 1024) as slot_ms_per_mb
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY total_slot_ms DESC;
```

#### Optimize Slot Usage
```sql
-- Use appropriate slot allocation
-- For on-demand pricing: BigQuery automatically allocates slots
-- For flat-rate pricing: Monitor and adjust slot allocation

-- Check current slot usage
SELECT
  TIMESTAMP_TRUNC(creation_time, HOUR) as hour,
  COUNT(*) as query_count,
  SUM(total_slot_ms) as total_slot_ms,
  AVG(total_slot_ms) as avg_slot_ms
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY hour
ORDER BY hour;
```

## 🔧 Advanced Optimization Techniques

### 1. **Materialized Views**

#### Create Materialized Views
```sql
-- Create materialized view for frequently accessed aggregations
CREATE MATERIALIZED VIEW sales_summary
AS
SELECT 
  category,
  region,
  DATE(date) as date,
  COUNT(*) as count,
  SUM(amount) as total_amount,
  AVG(amount) as avg_amount,
  MIN(amount) as min_amount,
  MAX(amount) as max_amount
FROM sales_data
GROUP BY category, region, DATE(date);

-- Query materialized view (no computation needed)
SELECT category, SUM(total_amount) as category_total
FROM sales_summary
WHERE date >= '2023-01-01'
GROUP BY category;
```

#### Refresh Materialized Views
```sql
-- Refresh materialized view
CALL BQ.REFRESH_MATERIALIZED_VIEW('project.dataset.sales_summary');

-- Check materialized view status
SELECT
  table_name,
  last_refresh_time,
  refresh_watermark
FROM `project.dataset.INFORMATION_SCHEMA.MATERIALIZED_VIEWS`;
```

### 2. **Table-Valued Functions**

#### Create Table-Valued Functions
```sql
-- Create table-valued function for complex logic
CREATE OR REPLACE FUNCTION get_top_customers(min_amount NUMERIC)
RETURNS TABLE(customer_id INT64, total_amount NUMERIC, order_count INT64)
AS (
  SELECT 
    customer_id, 
    SUM(amount) as total_amount,
    COUNT(*) as order_count
  FROM sales_data
  GROUP BY customer_id
  HAVING SUM(amount) >= min_amount
);

-- Use function in queries
SELECT * FROM get_top_customers(10000);
```

### 3. **Custom Functions (UDFs)**

#### Create UDFs for Complex Logic
```sql
-- Create UDF for complex calculations
CREATE OR REPLACE FUNCTION calculate_discount(amount NUMERIC, category STRING)
RETURNS NUMERIC
AS (
  CASE 
    WHEN category = 'Electronics' THEN amount * 0.1
    WHEN category = 'Clothing' THEN amount * 0.15
    WHEN category = 'Books' THEN amount * 0.2
    ELSE amount * 0.05
  END
);

-- Use UDF in queries
SELECT 
  id, 
  amount, 
  category,
  calculate_discount(amount, category) as discount,
  amount - calculate_discount(amount, category) as final_amount
FROM sales_data;
```

## 📈 Performance Monitoring

### 1. **Query Performance Analysis**

#### Analyze Query Execution
```sql
-- Get detailed query execution information
SELECT
  job_id,
  creation_time,
  query,
  total_slot_ms,
  total_bytes_processed,
  total_bytes_billed,
  cache_hit,
  -- Calculate performance metrics
  total_bytes_processed / total_slot_ms as bytes_per_slot_ms,
  total_bytes_billed / total_bytes_processed as billing_ratio
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY total_slot_ms DESC;
```

#### Identify Performance Bottlenecks
```sql
-- Find queries with high slot usage
SELECT
  job_id,
  creation_time,
  query,
  total_slot_ms,
  total_bytes_processed,
  -- Identify expensive operations
  CASE 
    WHEN query LIKE '%CROSS JOIN%' THEN 'Cross Join'
    WHEN query LIKE '%ORDER BY%' THEN 'Order By'
    WHEN query LIKE '%DISTINCT%' THEN 'Distinct'
    WHEN query LIKE '%GROUP BY%' THEN 'Group By'
    WHEN query LIKE '%JOIN%' THEN 'Join'
    ELSE 'Other'
  END as operation_type
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE total_slot_ms > 1000000  -- > 1M slot-ms
ORDER BY total_slot_ms DESC;
```

### 2. **Cost Analysis**

#### Analyze Query Costs
```sql
-- Calculate query costs
SELECT
  job_id,
  creation_time,
  query,
  total_bytes_processed,
  total_bytes_billed,
  -- Cost calculation (assuming $5 per TB)
  (total_bytes_billed / 1024 / 1024 / 1024 / 1024) * 5 as estimated_cost_usd,
  -- Efficiency ratio
  total_bytes_billed / total_bytes_processed as billing_efficiency
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY estimated_cost_usd DESC;
```

#### Monitor Cost Trends
```sql
-- Monitor cost trends over time
SELECT
  TIMESTAMP_TRUNC(creation_time, HOUR) as hour,
  COUNT(*) as query_count,
  SUM(total_bytes_billed) as total_bytes_billed,
  -- Calculate hourly cost
  (SUM(total_bytes_billed) / 1024 / 1024 / 1024 / 1024) * 5 as hourly_cost_usd
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY hour
ORDER BY hour;
```

### 3. **Resource Utilization**

#### Monitor Slot Usage
```sql
-- Monitor slot usage patterns
SELECT
  TIMESTAMP_TRUNC(creation_time, HOUR) as hour,
  COUNT(*) as query_count,
  SUM(total_slot_ms) as total_slot_ms,
  AVG(total_slot_ms) as avg_slot_ms,
  MAX(total_slot_ms) as max_slot_ms,
  -- Calculate slot utilization
  SUM(total_slot_ms) / 3600000 as slot_hours  -- Convert to slot-hours
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY hour
ORDER BY hour;
```

## 🎯 Best Practices

### 1. **Query Design**
- Use specific column names instead of SELECT *
- Apply filters early in the query
- Use appropriate JOIN types and order
- Leverage window functions instead of self-joins

### 2. **Data Layout**
- Use partitioning for time-series data
- Use clustering for frequently queried columns
- Combine partitioning and clustering for optimal performance
- Use appropriate data types

### 3. **Cost Management**
- Use LIMIT to reduce data processing
- Leverage query result caching
- Use approximate functions when precision isn't critical
- Monitor and analyze costs regularly

### 4. **Performance Monitoring**
- Monitor slot usage and costs
- Identify expensive queries
- Analyze query execution plans
- Set up alerts for performance issues

## 🔧 Troubleshooting Common Issues

### 1. **High Slot Usage**

#### Identify the Cause
```sql
-- Check for expensive operations
SELECT
  job_id,
  query,
  total_slot_ms,
  total_bytes_processed,
  -- Look for expensive operations
  CASE 
    WHEN query LIKE '%CROSS JOIN%' THEN 'Cross Join'
    WHEN query LIKE '%ORDER BY%' THEN 'Order By'
    WHEN query LIKE '%DISTINCT%' THEN 'Distinct'
    WHEN query LIKE '%GROUP BY%' THEN 'Group By'
    ELSE 'Other'
  END as operation_type
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE total_slot_ms > 1000000
ORDER BY total_slot_ms DESC;
```

#### Optimization Strategies
```sql
-- Replace expensive operations
-- Instead of CROSS JOIN
SELECT a.id, b.name
FROM table_a a
CROSS JOIN table_b b;

-- Use INNER JOIN with proper conditions
SELECT a.id, b.name
FROM table_a a
INNER JOIN table_b b ON a.category = b.category;
```

### 2. **Slow Query Performance**

#### Analyze Query Execution
```sql
-- Get detailed execution information
SELECT
  job_id,
  creation_time,
  query,
  total_slot_ms,
  total_bytes_processed,
  total_bytes_billed,
  -- Performance metrics
  total_slot_ms / (total_bytes_processed / 1024 / 1024) as slot_ms_per_mb
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY slot_ms_per_mb DESC;
```

#### Optimization Techniques
```sql
-- Use clustering for better performance
SELECT category, COUNT(*) as count
FROM sales_data_clustered
WHERE category = 'Electronics'  -- Uses clustering
GROUP BY category;

-- Use partitioning for time-based queries
SELECT category, COUNT(*) as count
FROM sales_data_partitioned
WHERE date >= '2023-01-01'  -- Uses partitioning
GROUP BY category;
```

### 3. **High Costs**

#### Analyze Cost Drivers
```sql
-- Identify cost drivers
SELECT
  job_id,
  creation_time,
  query,
  total_bytes_processed,
  total_bytes_billed,
  -- Calculate cost efficiency
  total_bytes_billed / total_bytes_processed as billing_ratio,
  -- Estimate cost
  (total_bytes_billed / 1024 / 1024 / 1024 / 1024) * 5 as estimated_cost_usd
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY estimated_cost_usd DESC;
```

#### Cost Optimization Strategies
```sql
-- Use LIMIT to reduce costs
SELECT id, name, amount
FROM sales_data
ORDER BY amount DESC
LIMIT 100;  -- Process only top 100 records

-- Use approximate functions
SELECT category, APPROX_COUNT_DISTINCT(customer_id) as unique_customers
FROM sales_data
GROUP BY category;
```

## 🔗 Related Documentation

- [BigQuery Shuffling and Optimization](./bigquery-shuffling-optimization.md)
- [Data Partitioning Strategies](./data-partitioning.md)
- [Query Optimization](./query-optimization.md)
- [Cost Optimization](./cost-optimization.md)

---

*Regular performance tuning and monitoring are essential for maintaining optimal BigQuery performance and cost efficiency. This guide provides a foundation for understanding and implementing performance optimization strategies.*
