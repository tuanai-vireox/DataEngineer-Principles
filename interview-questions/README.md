# Data Engineering Interview Questions

This document contains comprehensive interview questions covering data governance, data processing technologies, and modern data architecture concepts. These questions are designed to assess both theoretical knowledge and practical experience.

## 📋 Table of Contents

1. [Data Governance Questions](#data-governance-questions)
2. [Apache Spark Questions](#apache-spark-questions)
3. [Apache Flink Questions](#apache-flink-questions)
4. [BigQuery Questions](#bigquery-questions)
5. [SQL Questions](#sql-questions)
6. [System Design Questions](#system-design-questions)
7. [Scenario-Based Questions](#scenario-based-questions)

---

## 🛡️ Data Governance Questions

### Basic Level

**Q1: What is data governance and why is it important?**
- **Expected Answer**: Data governance is the overall management of data availability, usability, integrity, and security in an organization. It's important because it ensures data quality, compliance with regulations, reduces risks, and enables better decision-making.

**Q2: What are the key components of a data governance framework?**
- **Expected Answer**: 
  - Data stewardship and ownership
  - Data quality management
  - Data lineage and metadata management
  - Data security and privacy
  - Compliance and regulatory adherence
  - Data catalog and discovery

**Q3: Explain the difference between data owner, data steward, and data custodian.**
- **Expected Answer**:
  - **Data Owner**: Business executive responsible for data strategy and decisions
  - **Data Steward**: Subject matter expert who ensures data quality and business rules
  - **Data Custodian**: IT professional who manages the technical aspects of data storage and processing

### Intermediate Level

**Q4: How would you implement data lineage tracking in a data pipeline?**
- **Expected Answer**: 
  - Use metadata management tools like Apache Atlas, Collibra, or custom solutions
  - Capture transformation logic and data flow
  - Implement automated lineage discovery
  - Store lineage information in a graph database
  - Provide lineage visualization for impact analysis

**Q5: What is the difference between data masking and data anonymization?**
- **Expected Answer**:
  - **Data Masking**: Replaces sensitive data with realistic but fake data (reversible)
  - **Data Anonymization**: Removes or modifies data to prevent identification (irreversible)
  - Masking preserves data format and relationships, anonymization focuses on privacy protection

**Q6: How would you handle GDPR compliance in a data warehouse?**
- **Expected Answer**:
  - Implement data subject rights (access, rectification, erasure, portability)
  - Maintain consent records and data processing purposes
  - Implement data retention policies
  - Provide data portability mechanisms
  - Conduct privacy impact assessments

### Advanced Level

**Q7: Design a data quality framework for a real-time streaming pipeline.**
- **Expected Answer**:
  - Implement schema validation at ingestion
  - Use statistical process control for anomaly detection
  - Implement data quality rules engine
  - Create data quality dashboards and alerts
  - Handle data quality issues in real-time (quarantine, alert, or auto-fix)

**Q8: How would you implement fine-grained access control in a data lake?**
- **Expected Answer**:
  - Use Apache Ranger or similar tools for policy management
  - Implement row-level and column-level security
  - Use attribute-based access control (ABAC)
  - Implement data classification and tagging
  - Use encryption and tokenization for sensitive data

---

## ⚡ Apache Spark Questions

### Basic Level

**Q9: What is Apache Spark and what are its main components?**
- **Expected Answer**: Apache Spark is a unified analytics engine for large-scale data processing. Main components:
  - **Spark Core**: Basic functionality and RDD API
  - **Spark SQL**: SQL interface and DataFrame API
  - **Spark Streaming**: Real-time data processing
  - **MLlib**: Machine learning library
  - **GraphX**: Graph processing

**Q10: Explain the difference between RDD, DataFrame, and Dataset in Spark.**
- **Expected Answer**:
  - **RDD**: Resilient Distributed Dataset - low-level API, immutable distributed collection
  - **DataFrame**: High-level API with schema, optimized execution plans
  - **Dataset**: Type-safe API combining benefits of RDD and DataFrame

**Q11: What is the difference between transformations and actions in Spark?**
- **Expected Answer**:
  - **Transformations**: Lazy operations that create new RDDs (map, filter, groupBy)
  - **Actions**: Operations that trigger computation and return results (collect, count, save)

### Intermediate Level

**Q12: How does Spark handle data partitioning and why is it important?**
- **Expected Answer**:
  - Data is divided into partitions across cluster nodes
  - Enables parallel processing and fault tolerance
  - Partitioning strategies: hash, range, custom
  - Affects performance - avoid data skew and ensure even distribution

**Q13: Explain Spark's execution model and the role of DAG.**
- **Expected Answer**:
  - Spark creates a Directed Acyclic Graph (DAG) of operations
  - DAG Scheduler optimizes execution plan
  - Tasks are distributed across executors
  - Enables optimization like predicate pushdown and column pruning

**Q14: How would you optimize a Spark job that's running slowly?**
- **Expected Answer**:
  - Check for data skew and repartition if needed
  - Optimize partitioning strategy
  - Use broadcast joins for small tables
  - Cache frequently used DataFrames
  - Tune executor memory and cores
  - Use appropriate file formats (Parquet, Delta)

### Advanced Level

**Q15: Design a Spark streaming application for real-time fraud detection.**
- **Expected Answer**:
  - Use Structured Streaming with Kafka as source
  - Implement windowing for time-based aggregations
  - Use MLlib for fraud detection models
  - Implement checkpointing for fault tolerance
  - Use watermarking for late data handling
  - Output to multiple sinks (alerts, database, dashboard)

**Q16: How would you handle data skew in a Spark job?**
- **Expected Answer**:
  - Identify skewed keys using sampling
  - Use salting technique to distribute skewed data
  - Implement custom partitioning strategies
  - Use broadcast joins for small skewed tables
  - Consider two-phase aggregation approach

---

## 🌊 Apache Flink Questions

### Basic Level

**Q17: What is Apache Flink and how does it differ from Spark Streaming?**
- **Expected Answer**: Apache Flink is a stream processing framework with true streaming capabilities. Differences:
  - **Flink**: True streaming, low latency, event-time processing
  - **Spark Streaming**: Micro-batch processing, higher latency
  - **Flink**: Better for real-time applications
  - **Spark**: Better for batch processing and ML workloads

**Q18: Explain the concept of event time vs processing time in Flink.**
- **Expected Answer**:
  - **Event Time**: When the event actually occurred (embedded in data)
  - **Processing Time**: When the event is processed by Flink
  - Event time is more accurate for analytics but requires handling late data
  - Processing time is simpler but less accurate

**Q19: What are watermarks in Flink and why are they important?**
- **Expected Answer**: Watermarks are timestamps that indicate the progress of event time. They:
  - Allow handling of late-arriving data
  - Enable windowing operations
  - Provide a mechanism to trigger window computations
  - Help balance between latency and completeness

### Intermediate Level

**Q20: How does Flink handle state management and checkpointing?**
- **Expected Answer**:
  - Flink maintains state for stateful operations
  - Checkpointing creates consistent snapshots of state
  - Uses distributed snapshots algorithm
  - Supports exactly-once processing guarantees
  - State can be stored in memory, RocksDB, or external systems

**Q21: Design a Flink application for real-time recommendation engine.**
- **Expected Answer**:
  - Use Kafka as data source for user events
  - Implement user behavior tracking with state
  - Use CEP for pattern detection
  - Implement machine learning models for recommendations
  - Output recommendations to Kafka or database
  - Use windowing for time-based features

**Q22: How would you handle backpressure in Flink?**
- **Expected Answer**:
  - Monitor backpressure using Flink metrics
  - Implement dynamic scaling based on backpressure
  - Use buffering and batching strategies
  - Optimize operator parallelism
  - Consider using async I/O for external calls

### Advanced Level

**Q23: Implement a Flink CEP pattern for detecting suspicious transactions.**
- **Expected Answer**:
```java
Pattern<Transaction, ?> suspiciousPattern = Pattern.<Transaction>begin("first")
    .where(new SimpleCondition<Transaction>() {
        @Override
        public boolean filter(Transaction transaction) {
            return transaction.getAmount() > 1000;
        }
    })
    .followedBy("second")
    .where(new SimpleCondition<Transaction>() {
        @Override
        public boolean filter(Transaction transaction) {
            return transaction.getAmount() > 1000;
        }
    })
    .within(Time.minutes(5));
```

**Q24: How would you implement exactly-once processing in Flink with Kafka?**
- **Expected Answer**:
  - Enable Kafka exactly-once producer
  - Use Flink's two-phase commit sink
  - Implement idempotent operations
  - Use transactional writes to external systems
  - Handle failures and recovery scenarios

---

## 🔍 BigQuery Questions

### Basic Level

**Q25: What is BigQuery and what are its key features?**
- **Expected Answer**: BigQuery is Google's serverless data warehouse. Key features:
  - Serverless and fully managed
  - Petabyte-scale analytics
  - SQL interface
  - Real-time analytics
  - Machine learning integration
  - Pay-per-query pricing

**Q26: Explain BigQuery's pricing model.**
- **Expected Answer**:
  - **On-demand**: Pay per query (per TB processed)
  - **Flat-rate**: Monthly subscription for predictable workloads
  - **Storage**: Pay for data stored
  - **Streaming inserts**: Pay per row inserted
  - **Slots**: Reserved compute capacity

**Q27: What are BigQuery's data types and limitations?**
- **Expected Answer**:
  - **Data Types**: STRING, BYTES, INTEGER, FLOAT, BOOLEAN, TIMESTAMP, DATE, TIME, DATETIME, GEOGRAPHY, JSON, ARRAY, STRUCT
  - **Limitations**: 10MB per row, 100MB per query, 1000 columns per table

### Intermediate Level

**Q28: How would you optimize a slow BigQuery query?**
- **Expected Answer**:
  - Use appropriate data types
  - Implement table partitioning and clustering
  - Optimize JOIN operations
  - Use column pruning and predicate pushdown
  - Consider materialized views
  - Use approximate aggregation functions
  - Optimize subqueries and CTEs

**Q29: Explain BigQuery's partitioning and clustering strategies.**
- **Expected Answer**:
  - **Partitioning**: Divide table by time, integer range, or ingestion time
  - **Clustering**: Organize data by up to 4 columns
  - **Benefits**: Reduced query costs, improved performance
  - **Best Practices**: Partition by time, cluster by frequently filtered columns

**Q30: How would you implement data quality checks in BigQuery?**
- **Expected Answer**:
  - Use data quality functions (IS_NULL, IS_INF, etc.)
  - Implement custom data quality rules
  - Use BigQuery ML for anomaly detection
  - Create data quality dashboards
  - Implement automated data quality monitoring

### Advanced Level

**Q31: Design a real-time analytics pipeline using BigQuery.**
- **Expected Answer**:
  - Use Cloud Pub/Sub for data ingestion
  - Implement Cloud Functions for data transformation
  - Use BigQuery streaming inserts for real-time data
  - Implement materialized views for performance
  - Use BigQuery ML for real-time predictions
  - Create real-time dashboards with Data Studio

**Q32: How would you handle data governance in BigQuery?**
- **Expected Answer**:
  - Implement column-level security
  - Use data classification and labeling
  - Implement row-level security
  - Use IAM for access control
  - Implement data lineage tracking
  - Use audit logs for compliance

---

## 🗃️ SQL Questions

### Basic Level

**Q33: Write a SQL query to find the second highest salary from an employee table.**
- **Expected Answer**:
```sql
-- Method 1: Using window function
SELECT salary
FROM (
    SELECT salary, ROW_NUMBER() OVER (ORDER BY salary DESC) as rn
    FROM employees
) ranked
WHERE rn = 2;

-- Method 2: Using subquery
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Q34: Explain the difference between INNER JOIN, LEFT JOIN, and RIGHT JOIN.**
- **Expected Answer**:
  - **INNER JOIN**: Returns only matching records from both tables
  - **LEFT JOIN**: Returns all records from left table and matching records from right table
  - **RIGHT JOIN**: Returns all records from right table and matching records from left table
  - **FULL OUTER JOIN**: Returns all records from both tables

**Q35: What is the difference between WHERE and HAVING clauses?**
- **Expected Answer**:
  - **WHERE**: Filters rows before grouping
  - **HAVING**: Filters groups after grouping
  - WHERE cannot use aggregate functions
  - HAVING can use aggregate functions

### Intermediate Level

**Q36: Write a SQL query to find customers who have made purchases in all product categories.**
- **Expected Answer**:
```sql
SELECT customer_id
FROM (
    SELECT DISTINCT customer_id, category_id
    FROM purchases p
    JOIN products pr ON p.product_id = pr.product_id
) customer_categories
GROUP BY customer_id
HAVING COUNT(DISTINCT category_id) = (
    SELECT COUNT(DISTINCT category_id) FROM products
);
```

**Q37: Explain window functions and provide examples.**
- **Expected Answer**: Window functions perform calculations across a set of rows related to the current row.
```sql
-- ROW_NUMBER: Assigns sequential numbers
SELECT name, salary, ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- RANK: Assigns ranks with gaps for ties
SELECT name, salary, RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- LAG/LEAD: Access previous/next row values
SELECT name, salary, LAG(salary) OVER (ORDER BY salary) as prev_salary
FROM employees;
```

**Q38: How would you optimize a slow SQL query?**
- **Expected Answer**:
  - Analyze execution plan
  - Add appropriate indexes
  - Optimize JOIN conditions
  - Use appropriate data types
  - Avoid SELECT *
  - Use LIMIT for large result sets
  - Consider query rewriting

### Advanced Level

**Q39: Write a SQL query to find the running total of sales by month.**
- **Expected Answer**:
```sql
SELECT 
    month,
    sales,
    SUM(sales) OVER (ORDER BY month ROWS UNBOUNDED PRECEDING) as running_total
FROM monthly_sales
ORDER BY month;
```

**Q40: Design a SQL solution for handling slowly changing dimensions (SCD Type 2).**
- **Expected Answer**:
```sql
-- Create SCD Type 2 table
CREATE TABLE customer_scd (
    customer_id INT,
    name VARCHAR(100),
    email VARCHAR(100),
    address VARCHAR(200),
    effective_date DATE,
    end_date DATE,
    is_current BOOLEAN
);

-- Insert new record
INSERT INTO customer_scd (customer_id, name, email, address, effective_date, end_date, is_current)
VALUES (1, 'John Doe', 'john@email.com', '123 Main St', '2023-01-01', '9999-12-31', TRUE);

-- Update existing record (SCD Type 2)
UPDATE customer_scd 
SET end_date = '2023-06-30', is_current = FALSE
WHERE customer_id = 1 AND is_current = TRUE;

INSERT INTO customer_scd (customer_id, name, email, address, effective_date, end_date, is_current)
VALUES (1, 'John Doe', 'john@email.com', '456 Oak Ave', '2023-07-01', '9999-12-31', TRUE);
```

---

## 🏗️ System Design Questions

### Data Pipeline Design

**Q41: Design a real-time data pipeline for processing e-commerce transactions.**
- **Expected Answer**:
  - **Data Sources**: Web applications, mobile apps, POS systems
  - **Ingestion**: Apache Kafka for event streaming
  - **Processing**: Apache Flink for real-time processing
  - **Storage**: Data lake (S3) for raw data, data warehouse (BigQuery) for analytics
  - **Monitoring**: Real-time dashboards and alerts
  - **Security**: Encryption, access controls, audit logging

**Q42: How would you design a data lake architecture for a large enterprise?**
- **Expected Answer**:
  - **Storage**: S3/ADLS/GCS with lifecycle policies
  - **Processing**: Spark, Flink for batch and streaming
  - **Catalog**: Apache Atlas or AWS Glue for metadata
  - **Security**: Apache Ranger for access control
  - **Governance**: Data quality, lineage, and compliance
  - **Layers**: Bronze (raw), Silver (cleaned), Gold (business-ready)

**Q43: Design a machine learning pipeline for fraud detection.**
- **Expected Answer**:
  - **Data Collection**: Real-time transaction data
  - **Feature Engineering**: Historical patterns, user behavior
  - **Model Training**: Offline training with historical data
  - **Model Serving**: Real-time inference with low latency
  - **Monitoring**: Model performance and drift detection
  - **Feedback Loop**: Continuous learning from new data

### Scalability and Performance

**Q44: How would you handle data skew in a distributed system?**
- **Expected Answer**:
  - **Detection**: Monitor partition sizes and processing times
  - **Prevention**: Use salting techniques, custom partitioning
  - **Mitigation**: Two-phase aggregation, broadcast joins
  - **Monitoring**: Set up alerts for skewed partitions
  - **Optimization**: Dynamic repartitioning based on data distribution

**Q45: Design a system for processing 1TB of data daily with 99.9% uptime.**
- **Expected Answer**:
  - **Architecture**: Distributed processing with redundancy
  - **Storage**: Distributed file system with replication
  - **Processing**: Spark with auto-scaling
  - **Monitoring**: Comprehensive monitoring and alerting
  - **Recovery**: Automated failover and recovery procedures
  - **Testing**: Regular disaster recovery testing

---

## 🎯 Scenario-Based Questions

### Data Quality Issues

**Q46: You discover that 30% of your customer data has missing email addresses. How would you handle this?**
- **Expected Answer**:
  - **Immediate**: Implement data validation at ingestion
  - **Short-term**: Data cleansing and enrichment
  - **Long-term**: Improve data collection processes
  - **Monitoring**: Set up data quality dashboards
  - **Governance**: Establish data quality standards

**Q47: A critical data pipeline fails at 2 AM. Walk me through your response.**
- **Expected Answer**:
  - **Immediate**: Assess impact and notify stakeholders
  - **Investigation**: Check logs, metrics, and system health
  - **Resolution**: Fix the issue or implement workaround
  - **Recovery**: Restart pipeline and verify data integrity
  - **Post-mortem**: Document incident and implement prevention measures

### Compliance and Security

**Q48: How would you ensure GDPR compliance for a data warehouse containing EU customer data?**
- **Expected Answer**:
  - **Data Mapping**: Identify all EU personal data
  - **Consent Management**: Track and manage consent
  - **Data Subject Rights**: Implement access, rectification, erasure, portability
  - **Security**: Encryption, access controls, audit logging
  - **Retention**: Implement data retention policies
  - **Documentation**: Maintain compliance documentation

**Q49: You need to migrate sensitive data from on-premises to cloud. How would you approach this?**
- **Expected Answer**:
  - **Assessment**: Inventory and classify sensitive data
  - **Security**: Implement encryption in transit and at rest
  - **Compliance**: Ensure regulatory compliance
  - **Testing**: Test migration process with non-sensitive data
  - **Monitoring**: Monitor migration process and data integrity
  - **Documentation**: Document security measures and procedures

### Performance Optimization

**Q50: Your Spark job is taking 4 hours to process 100GB of data. How would you optimize it?**
- **Expected Answer**:
  - **Analysis**: Profile the job to identify bottlenecks
  - **Partitioning**: Optimize data partitioning strategy
  - **Caching**: Cache frequently used DataFrames
  - **Joins**: Optimize join strategies and use broadcast joins
  - **File Format**: Use columnar formats like Parquet
  - **Resources**: Tune executor memory, cores, and parallelism
  - **Code**: Optimize transformations and avoid unnecessary operations

---

## 📝 Interview Tips

### Preparation
1. **Understand the basics**: Master fundamental concepts
2. **Practice coding**: Write code for common scenarios
3. **Study real-world examples**: Understand industry use cases
4. **Prepare examples**: Have specific examples from your experience
5. **Know the tools**: Understand the technologies you'll be working with

### During the Interview
1. **Ask clarifying questions**: Understand requirements before answering
2. **Think out loud**: Explain your thought process
3. **Consider trade-offs**: Discuss pros and cons of different approaches
4. **Be specific**: Provide concrete examples and code
5. **Show problem-solving**: Demonstrate analytical thinking

### Common Mistakes to Avoid
1. **Jumping to solutions**: Take time to understand the problem
2. **Ignoring edge cases**: Consider error handling and edge cases
3. **Not considering scalability**: Think about performance and scale
4. **Lack of examples**: Provide concrete examples from experience
5. **Poor communication**: Explain concepts clearly and concisely

---

## 🔗 Related Resources

- [Data Governance Best Practices](../concepts/security/)
- [Apache Spark Documentation](https://spark.apache.org/docs/)
- [Apache Flink Documentation](https://flink.apache.org/docs/)
- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [SQL Best Practices](https://www.sqlstyle.guide/)

---

*This document is continuously updated with new questions and scenarios. Feel free to contribute additional questions or improve existing ones.*
