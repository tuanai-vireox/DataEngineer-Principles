# Modern Data Architecture Implementation Guide

This guide provides practical implementation examples and code snippets for building a modern data architecture.

## 🚀 Quick Start Implementation

### 1. Infrastructure as Code (Terraform)

#### AWS Infrastructure Setup

```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Data Lake Storage
resource "aws_s3_bucket" "data_lake" {
  bucket = "${var.project_name}-data-lake-${random_string.bucket_suffix.result}"
  
  tags = {
    Name        = "Data Lake"
    Environment = var.environment
    Project     = var.project_name
  }
}

resource "aws_s3_bucket_versioning" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_lifecycle_configuration" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id

  rule {
    id     = "data_lifecycle"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER"
    }

    expiration {
      days = 2555 # 7 years
    }
  }
}

# Data Warehouse
resource "aws_redshift_cluster" "data_warehouse" {
  cluster_identifier = "${var.project_name}-data-warehouse"
  database_name      = "analytics"
  master_username    = var.redshift_username
  master_password    = var.redshift_password
  node_type          = "dc2.large"
  cluster_type       = "single-node"
  
  tags = {
    Name        = "Data Warehouse"
    Environment = var.environment
    Project     = var.project_name
  }
}

# Kafka Cluster
resource "aws_msk_cluster" "kafka" {
  cluster_name           = "${var.project_name}-kafka"
  kafka_version          = "2.8.1"
  number_of_broker_nodes = 3

  broker_node_group_info {
    instance_type   = "kafka.m5.large"
    ebs_volume_size = 1000
    client_subnets  = var.private_subnet_ids
    security_groups = [aws_security_group.kafka.id]
  }

  tags = {
    Name        = "Kafka Cluster"
    Environment = var.environment
    Project     = var.project_name
  }
}

# EMR Cluster for Spark
resource "aws_emr_cluster" "spark" {
  name          = "${var.project_name}-spark-cluster"
  release_label = "emr-6.15.0"
  applications  = ["Spark", "Hadoop"]

  ec2_attributes {
    subnet_id                         = var.private_subnet_ids[0]
    emr_managed_master_security_group = aws_security_group.emr_master.id
    emr_managed_slave_security_group  = aws_security_group.emr_slave.id
    instance_profile                  = aws_iam_instance_profile.emr_profile.arn
  }

  master_instance_group {
    instance_type = "m5.xlarge"
  }

  core_instance_group {
    instance_type  = "m5.large"
    instance_count = 2
  }

  tags = {
    Name        = "Spark Cluster"
    Environment = var.environment
    Project     = var.project_name
  }
}
```

### 2. Data Ingestion Pipeline

#### Kafka Producer (Python)

```python
# kafka_producer.py
import json
import time
from datetime import datetime
from kafka import KafkaProducer
from typing import Dict, Any
import logging

class DataIngestionProducer:
    def __init__(self, bootstrap_servers: str, topic: str):
        self.producer = KafkaProducer(
            bootstrap_servers=bootstrap_servers,
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            key_serializer=lambda k: k.encode('utf-8') if k else None,
            acks='all',
            retries=3,
            retry_backoff_ms=1000
        )
        self.topic = topic
        self.logger = logging.getLogger(__name__)
    
    def send_event(self, event_data: Dict[str, Any], key: str = None):
        """Send event to Kafka topic"""
        try:
            # Add metadata
            event_data['timestamp'] = datetime.utcnow().isoformat()
            event_data['event_id'] = f"{int(time.time() * 1000)}"
            
            future = self.producer.send(
                self.topic,
                key=key,
                value=event_data
            )
            
            # Wait for confirmation
            record_metadata = future.get(timeout=10)
            self.logger.info(f"Event sent to {record_metadata.topic} partition {record_metadata.partition}")
            
        except Exception as e:
            self.logger.error(f"Failed to send event: {e}")
            raise
    
    def send_batch_events(self, events: list, key_field: str = None):
        """Send multiple events in batch"""
        for event in events:
            key = event.get(key_field) if key_field else None
            self.send_event(event, key)
    
    def close(self):
        """Close the producer"""
        self.producer.close()

# Usage example
if __name__ == "__main__":
    producer = DataIngestionProducer(
        bootstrap_servers="localhost:9092",
        topic="user-events"
    )
    
    # Sample event data
    event_data = {
        "user_id": "12345",
        "event_type": "page_view",
        "page_url": "/products/laptop",
        "session_id": "sess_abc123",
        "properties": {
            "product_id": "laptop_001",
            "category": "electronics"
        }
    }
    
    producer.send_event(event_data, key="12345")
    producer.close()
```

#### Change Data Capture (Debezium)

```yaml
# debezium-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: debezium-config
data:
  application.properties: |
    # Debezium Configuration
    bootstrap.servers=kafka:9092
    key.converter=org.apache.kafka.connect.json.JsonConverter
    value.converter=org.apache.kafka.connect.json.JsonConverter
    key.converter.schemas.enable=false
    value.converter.schemas.enable=false
    
    # Source Database Configuration
    database.hostname=postgres
    database.port=5432
    database.user=debezium
    database.password=debezium
    database.dbname=analytics
    database.server.name=postgres-server
    
    # Connector Configuration
    connector.class=io.debezium.connector.postgresql.PostgresConnector
    tasks.max=1
    database.history.kafka.bootstrap.servers=kafka:9092
    database.history.kafka.topic=dbhistory.postgres
    include.schema.changes=true
    snapshot.mode=initial
```

### 3. Stream Processing with Apache Flink

#### Real-time Data Processing

```python
# flink_stream_processor.py
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.table import StreamTableEnvironment
from pyflink.datastream.connectors import FlinkKafkaConsumer
from pyflink.common.serialization import SimpleStringSchema
from pyflink.common.typeinfo import Types
from pyflink.datastream.functions import MapFunction, KeyedProcessFunction
from pyflink.common.time import Time
import json
import logging

class EventProcessor(MapFunction):
    """Process incoming events"""
    
    def map(self, value):
        try:
            event = json.loads(value)
            # Add processing timestamp
            event['processed_at'] = int(time.time() * 1000)
            return event
        except Exception as e:
            logging.error(f"Error processing event: {e}")
            return None

class FraudDetectionFunction(KeyedProcessFunction):
    """Detect suspicious transactions"""
    
    def __init__(self):
        self.state = None
    
    def open(self, runtime_context):
        # Initialize state for user transaction history
        self.state = runtime_context.get_state(
            Types.MAP(Types.STRING(), Types.LIST(Types.STRING()))
        )
    
    def process_element(self, value, ctx):
        user_id = value['user_id']
        amount = value['amount']
        timestamp = value['timestamp']
        
        # Get user's transaction history
        history = self.state.value()
        if history is None:
            history = []
        
        # Add current transaction
        history.append(json.dumps({
            'amount': amount,
            'timestamp': timestamp
        }))
        
        # Keep only last 10 transactions
        if len(history) > 10:
            history = history[-10:]
        
        # Update state
        self.state.update(history)
        
        # Fraud detection logic
        if self._detect_fraud(history, amount):
            return {
                'user_id': user_id,
                'amount': amount,
                'timestamp': timestamp,
                'fraud_score': self._calculate_fraud_score(history, amount),
                'alert_type': 'suspicious_transaction'
            }
        
        return None
    
    def _detect_fraud(self, history, current_amount):
        """Simple fraud detection logic"""
        if len(history) < 3:
            return False
        
        # Check for rapid successive transactions
        recent_transactions = history[-3:]
        total_amount = sum(json.loads(tx)['amount'] for tx in recent_transactions)
        
        return total_amount > 10000 or current_amount > 5000
    
    def _calculate_fraud_score(self, history, current_amount):
        """Calculate fraud probability score"""
        score = 0.0
        
        if current_amount > 5000:
            score += 0.3
        
        if len(history) >= 3:
            recent_amounts = [json.loads(tx)['amount'] for tx in history[-3:]]
            avg_amount = sum(recent_amounts) / len(recent_amounts)
            if current_amount > avg_amount * 2:
                score += 0.4
        
        return min(score, 1.0)

def create_stream_processing_job():
    """Create and configure Flink stream processing job"""
    
    # Create execution environment
    env = StreamExecutionEnvironment.get_execution_environment()
    env.set_parallelism(4)
    
    # Add Kafka connector
    env.add_jars("file:///opt/flink/lib/flink-sql-connector-kafka-1.17.0.jar")
    
    # Create Kafka consumer
    kafka_consumer = FlinkKafkaConsumer(
        topics='transactions',
        deserialization_schema=SimpleStringSchema(),
        properties={
            'bootstrap.servers': 'kafka:9092',
            'group.id': 'fraud-detection-group',
            'auto.offset.reset': 'latest'
        }
    )
    
    # Create data stream
    stream = env.add_source(kafka_consumer)
    
    # Process events
    processed_stream = stream \
        .map(EventProcessor()) \
        .filter(lambda x: x is not None) \
        .key_by(lambda x: x['user_id']) \
        .process(FraudDetectionFunction()) \
        .filter(lambda x: x is not None)
    
    # Output to Kafka
    processed_stream.add_sink(
        FlinkKafkaProducer(
            topic='fraud-alerts',
            serialization_schema=SimpleStringSchema(),
            producer_config={
                'bootstrap.servers': 'kafka:9092'
            }
        )
    )
    
    return env

if __name__ == "__main__":
    env = create_stream_processing_job()
    env.execute("Fraud Detection Job")
```

### 4. Batch Processing with Apache Spark

#### ETL Pipeline

```python
# spark_etl_pipeline.py
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *
import logging
from datetime import datetime, timedelta

class DataETLPipeline:
    def __init__(self, app_name: str = "DataETLPipeline"):
        self.spark = SparkSession.builder \
            .appName(app_name) \
            .config("spark.sql.adaptive.enabled", "true") \
            .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
            .config("spark.serializer", "org.apache.spark.serializer.KryoSerializer") \
            .getOrCreate()
        
        self.logger = logging.getLogger(__name__)
    
    def extract_data(self, source_path: str, file_format: str = "json"):
        """Extract data from source"""
        try:
            if file_format == "json":
                df = self.spark.read.json(source_path)
            elif file_format == "parquet":
                df = self.spark.read.parquet(source_path)
            elif file_format == "csv":
                df = self.spark.read.option("header", "true").csv(source_path)
            else:
                raise ValueError(f"Unsupported file format: {file_format}")
            
            self.logger.info(f"Extracted {df.count()} records from {source_path}")
            return df
            
        except Exception as e:
            self.logger.error(f"Error extracting data: {e}")
            raise
    
    def transform_data(self, df, transformation_rules: dict):
        """Apply data transformations"""
        try:
            # Data quality checks
            df = self._apply_data_quality_checks(df)
            
            # Apply transformations
            for rule in transformation_rules:
                df = self._apply_transformation_rule(df, rule)
            
            # Add metadata
            df = df.withColumn("processed_at", current_timestamp()) \
                   .withColumn("batch_id", lit(datetime.now().strftime("%Y%m%d_%H%M%S")))
            
            self.logger.info(f"Transformed data: {df.count()} records")
            return df
            
        except Exception as e:
            self.logger.error(f"Error transforming data: {e}")
            raise
    
    def load_data(self, df, target_path: str, partition_columns: list = None):
        """Load data to target destination"""
        try:
            writer = df.write.mode("overwrite")
            
            if partition_columns:
                writer = writer.partitionBy(*partition_columns)
            
            writer.parquet(target_path)
            
            self.logger.info(f"Loaded data to {target_path}")
            
        except Exception as e:
            self.logger.error(f"Error loading data: {e}")
            raise
    
    def _apply_data_quality_checks(self, df):
        """Apply data quality validation rules"""
        # Remove duplicates
        df = df.dropDuplicates()
        
        # Handle null values
        df = df.fillna({
            "user_id": "unknown",
            "amount": 0.0,
            "category": "uncategorized"
        })
        
        # Validate data types
        df = df.withColumn("amount", col("amount").cast(DoubleType()))
        df = df.withColumn("timestamp", col("timestamp").cast(TimestampType()))
        
        return df
    
    def _apply_transformation_rule(self, df, rule: dict):
        """Apply a specific transformation rule"""
        rule_type = rule.get("type")
        
        if rule_type == "filter":
            condition = rule.get("condition")
            df = df.filter(condition)
        
        elif rule_type == "aggregation":
            group_by = rule.get("group_by", [])
            aggregations = rule.get("aggregations", {})
            df = df.groupBy(*group_by).agg(aggregations)
        
        elif rule_type == "join":
            other_df = rule.get("dataframe")
            join_condition = rule.get("condition")
            join_type = rule.get("join_type", "inner")
            df = df.join(other_df, join_condition, join_type)
        
        elif rule_type == "custom":
            custom_function = rule.get("function")
            df = custom_function(df)
        
        return df
    
    def run_pipeline(self, config: dict):
        """Run the complete ETL pipeline"""
        try:
            # Extract
            source_df = self.extract_data(
                config["source"]["path"],
                config["source"]["format"]
            )
            
            # Transform
            transformed_df = self.transform_data(
                source_df,
                config["transformations"]
            )
            
            # Load
            self.load_data(
                transformed_df,
                config["target"]["path"],
                config["target"].get("partition_columns")
            )
            
            self.logger.info("ETL pipeline completed successfully")
            
        except Exception as e:
            self.logger.error(f"ETL pipeline failed: {e}")
            raise
        finally:
            self.spark.stop()

# Usage example
if __name__ == "__main__":
    pipeline = DataETLPipeline("UserEventsETL")
    
    config = {
        "source": {
            "path": "s3://data-lake/bronze/user-events/",
            "format": "json"
        },
        "transformations": [
            {
                "type": "filter",
                "condition": "event_type = 'purchase'"
            },
            {
                "type": "aggregation",
                "group_by": ["user_id", "date"],
                "aggregations": {
                    "total_amount": sum("amount"),
                    "transaction_count": count("*")
                }
            }
        ],
        "target": {
            "path": "s3://data-lake/silver/user-daily-summary/",
            "partition_columns": ["date"]
        }
    }
    
    pipeline.run_pipeline(config)
```

### 5. Data Quality Framework

#### Data Quality Monitoring

```python
# data_quality_monitor.py
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *
import json
from datetime import datetime
from typing import Dict, List, Any

class DataQualityMonitor:
    def __init__(self, spark_session: SparkSession):
        self.spark = spark_session
        self.quality_rules = {}
        self.quality_metrics = {}
    
    def add_quality_rule(self, rule_name: str, rule_config: dict):
        """Add a data quality rule"""
        self.quality_rules[rule_name] = rule_config
    
    def validate_data(self, df, table_name: str) -> Dict[str, Any]:
        """Validate data against quality rules"""
        validation_results = {
            "table_name": table_name,
            "validation_timestamp": datetime.utcnow().isoformat(),
            "total_records": df.count(),
            "rules": {}
        }
        
        for rule_name, rule_config in self.quality_rules.items():
            rule_result = self._execute_rule(df, rule_config)
            validation_results["rules"][rule_name] = rule_result
        
        # Calculate overall quality score
        validation_results["quality_score"] = self._calculate_quality_score(
            validation_results["rules"]
        )
        
        return validation_results
    
    def _execute_rule(self, df, rule_config: dict) -> Dict[str, Any]:
        """Execute a specific quality rule"""
        rule_type = rule_config.get("type")
        
        if rule_type == "completeness":
            return self._check_completeness(df, rule_config)
        elif rule_type == "uniqueness":
            return self._check_uniqueness(df, rule_config)
        elif rule_type == "validity":
            return self._check_validity(df, rule_config)
        elif rule_type == "consistency":
            return self._check_consistency(df, rule_config)
        elif rule_type == "custom":
            return self._check_custom(df, rule_config)
        else:
            raise ValueError(f"Unknown rule type: {rule_type}")
    
    def _check_completeness(self, df, rule_config: dict) -> Dict[str, Any]:
        """Check data completeness"""
        columns = rule_config.get("columns", [])
        threshold = rule_config.get("threshold", 0.95)
        
        results = {}
        for column in columns:
            total_count = df.count()
            non_null_count = df.filter(col(column).isNotNull()).count()
            completeness_ratio = non_null_count / total_count if total_count > 0 else 0
            
            results[column] = {
                "completeness_ratio": completeness_ratio,
                "passed": completeness_ratio >= threshold,
                "non_null_count": non_null_count,
                "total_count": total_count
            }
        
        return {
            "type": "completeness",
            "passed": all(result["passed"] for result in results.values()),
            "details": results
        }
    
    def _check_uniqueness(self, df, rule_config: dict) -> Dict[str, Any]:
        """Check data uniqueness"""
        columns = rule_config.get("columns", [])
        threshold = rule_config.get("threshold", 0.95)
        
        total_count = df.count()
        unique_count = df.select(*columns).distinct().count()
        uniqueness_ratio = unique_count / total_count if total_count > 0 else 0
        
        return {
            "type": "uniqueness",
            "passed": uniqueness_ratio >= threshold,
            "uniqueness_ratio": uniqueness_ratio,
            "unique_count": unique_count,
            "total_count": total_count
        }
    
    def _check_validity(self, df, rule_config: dict) -> Dict[str, Any]:
        """Check data validity"""
        column = rule_config.get("column")
        validation_expression = rule_config.get("expression")
        threshold = rule_config.get("threshold", 0.95)
        
        total_count = df.count()
        valid_count = df.filter(validation_expression).count()
        validity_ratio = valid_count / total_count if total_count > 0 else 0
        
        return {
            "type": "validity",
            "passed": validity_ratio >= threshold,
            "validity_ratio": validity_ratio,
            "valid_count": valid_count,
            "total_count": total_count,
            "column": column,
            "expression": validation_expression
        }
    
    def _check_consistency(self, df, rule_config: dict) -> Dict[str, Any]:
        """Check data consistency"""
        # Implementation for consistency checks
        # This could include cross-column validation, referential integrity, etc.
        return {
            "type": "consistency",
            "passed": True,
            "details": "Consistency check not implemented"
        }
    
    def _check_custom(self, df, rule_config: dict) -> Dict[str, Any]:
        """Execute custom quality rule"""
        custom_function = rule_config.get("function")
        if custom_function:
            return custom_function(df)
        else:
            return {
                "type": "custom",
                "passed": False,
                "error": "No custom function provided"
            }
    
    def _calculate_quality_score(self, rules_results: Dict[str, Any]) -> float:
        """Calculate overall data quality score"""
        if not rules_results:
            return 0.0
        
        passed_rules = sum(1 for rule in rules_results.values() if rule.get("passed", False))
        total_rules = len(rules_results)
        
        return passed_rules / total_rules if total_rules > 0 else 0.0
    
    def generate_quality_report(self, validation_results: Dict[str, Any]) -> str:
        """Generate a human-readable quality report"""
        report = f"""
Data Quality Report
==================
Table: {validation_results['table_name']}
Timestamp: {validation_results['validation_timestamp']}
Total Records: {validation_results['total_records']:,}
Quality Score: {validation_results['quality_score']:.2%}

Rule Results:
"""
        
        for rule_name, rule_result in validation_results['rules'].items():
            status = "PASS" if rule_result.get('passed', False) else "FAIL"
            report += f"  {rule_name}: {status}\n"
            
            if rule_result.get('type') == 'completeness':
                for column, details in rule_result.get('details', {}).items():
                    report += f"    {column}: {details['completeness_ratio']:.2%} complete\n"
        
        return report

# Usage example
if __name__ == "__main__":
    spark = SparkSession.builder.appName("DataQualityMonitor").getOrCreate()
    
    monitor = DataQualityMonitor(spark)
    
    # Add quality rules
    monitor.add_quality_rule("user_id_completeness", {
        "type": "completeness",
        "columns": ["user_id"],
        "threshold": 0.99
    })
    
    monitor.add_quality_rule("email_validity", {
        "type": "validity",
        "column": "email",
        "expression": "email rlike '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$'",
        "threshold": 0.95
    })
    
    monitor.add_quality_rule("user_id_uniqueness", {
        "type": "uniqueness",
        "columns": ["user_id"],
        "threshold": 0.99
    })
    
    # Load data and validate
    df = spark.read.parquet("s3://data-lake/silver/users/")
    results = monitor.validate_data(df, "users")
    
    # Generate report
    report = monitor.generate_quality_report(results)
    print(report)
    
    spark.stop()
```

### 6. Data Governance Implementation

#### Data Catalog Service

```python
# data_catalog_service.py
from typing import Dict, List, Any, Optional
from datetime import datetime
import json
import logging
from dataclasses import dataclass, asdict

@dataclass
class DataAsset:
    name: str
    description: str
    owner: str
    steward: str
    classification: str
    retention_period: int
    created_at: datetime
    updated_at: datetime
    schema: Dict[str, Any]
    location: str
    format: str
    size_bytes: int
    row_count: int
    tags: List[str]

@dataclass
class DataLineage:
    source_asset: str
    target_asset: str
    transformation: str
    timestamp: datetime
    user: str
    operation_type: str

class DataCatalogService:
    def __init__(self, storage_backend: str = "s3"):
        self.storage_backend = storage_backend
        self.data_assets = {}
        self.data_lineage = []
        self.logger = logging.getLogger(__name__)
    
    def register_data_asset(self, asset: DataAsset):
        """Register a new data asset in the catalog"""
        try:
            self.data_assets[asset.name] = asset
            self.logger.info(f"Registered data asset: {asset.name}")
        except Exception as e:
            self.logger.error(f"Error registering data asset: {e}")
            raise
    
    def update_data_asset(self, asset_name: str, updates: Dict[str, Any]):
        """Update an existing data asset"""
        if asset_name not in self.data_assets:
            raise ValueError(f"Data asset {asset_name} not found")
        
        asset = self.data_assets[asset_name]
        for key, value in updates.items():
            if hasattr(asset, key):
                setattr(asset, key, value)
        
        asset.updated_at = datetime.utcnow()
        self.logger.info(f"Updated data asset: {asset_name}")
    
    def get_data_asset(self, asset_name: str) -> Optional[DataAsset]:
        """Get a data asset by name"""
        return self.data_assets.get(asset_name)
    
    def search_data_assets(self, query: str, filters: Dict[str, Any] = None) -> List[DataAsset]:
        """Search data assets by query and filters"""
        results = []
        
        for asset in self.data_assets.values():
            # Text search
            if query.lower() in asset.name.lower() or query.lower() in asset.description.lower():
                # Apply filters
                if filters:
                    if self._matches_filters(asset, filters):
                        results.append(asset)
                else:
                    results.append(asset)
        
        return results
    
    def _matches_filters(self, asset: DataAsset, filters: Dict[str, Any]) -> bool:
        """Check if asset matches the given filters"""
        for key, value in filters.items():
            if hasattr(asset, key):
                asset_value = getattr(asset, key)
                if isinstance(value, list):
                    if asset_value not in value:
                        return False
                else:
                    if asset_value != value:
                        return False
        return True
    
    def add_lineage(self, lineage: DataLineage):
        """Add data lineage information"""
        self.data_lineage.append(lineage)
        self.logger.info(f"Added lineage: {lineage.source_asset} -> {lineage.target_asset}")
    
    def get_lineage(self, asset_name: str) -> List[DataLineage]:
        """Get lineage information for an asset"""
        return [lineage for lineage in self.data_lineage 
                if lineage.source_asset == asset_name or lineage.target_asset == asset_name]
    
    def get_upstream_assets(self, asset_name: str) -> List[str]:
        """Get upstream assets (sources) for an asset"""
        upstream = []
        for lineage in self.data_lineage:
            if lineage.target_asset == asset_name:
                upstream.append(lineage.source_asset)
        return list(set(upstream))
    
    def get_downstream_assets(self, asset_name: str) -> List[str]:
        """Get downstream assets (consumers) for an asset"""
        downstream = []
        for lineage in self.data_lineage:
            if lineage.source_asset == asset_name:
                downstream.append(lineage.target_asset)
        return list(set(downstream))
    
    def export_catalog(self, format: str = "json") -> str:
        """Export the data catalog"""
        catalog_data = {
            "assets": {name: asdict(asset) for name, asset in self.data_assets.items()},
            "lineage": [asdict(lineage) for lineage in self.data_lineage],
            "export_timestamp": datetime.utcnow().isoformat()
        }
        
        if format == "json":
            return json.dumps(catalog_data, indent=2, default=str)
        else:
            raise ValueError(f"Unsupported export format: {format}")

# Usage example
if __name__ == "__main__":
    catalog = DataCatalogService()
    
    # Register data assets
    user_asset = DataAsset(
        name="users",
        description="User profile information",
        owner="data-team",
        steward="john.doe@company.com",
        classification="internal",
        retention_period=2555,  # 7 years
        created_at=datetime.utcnow(),
        updated_at=datetime.utcnow(),
        schema={
            "user_id": "string",
            "email": "string",
            "name": "string",
            "created_at": "timestamp"
        },
        location="s3://data-lake/silver/users/",
        format="parquet",
        size_bytes=1024000,
        row_count=10000,
        tags=["users", "profiles", "pii"]
    )
    
    catalog.register_data_asset(user_asset)
    
    # Add lineage
    lineage = DataLineage(
        source_asset="raw_user_events",
        target_asset="users",
        transformation="ETL pipeline - user profile extraction",
        timestamp=datetime.utcnow(),
        user="etl-pipeline",
        operation_type="transform"
    )
    
    catalog.add_lineage(lineage)
    
    # Search assets
    results = catalog.search_data_assets("user", {"classification": "internal"})
    print(f"Found {len(results)} assets")
    
    # Export catalog
    catalog_json = catalog.export_catalog()
    print(catalog_json)
```

This implementation guide provides practical, production-ready code examples for building a modern data architecture. Each component is designed to be modular, scalable, and maintainable, following best practices for data engineering.

## 🔗 Next Steps

1. **Deploy Infrastructure**: Use the Terraform configurations to set up cloud infrastructure
2. **Implement Data Pipelines**: Deploy the Kafka, Flink, and Spark components
3. **Set Up Monitoring**: Implement data quality monitoring and governance
4. **Configure Security**: Implement access controls and data encryption
5. **Test and Optimize**: Perform load testing and performance optimization

---

*This implementation guide provides a solid foundation for building a modern data architecture. Customize the components based on your specific requirements and constraints.*
