# Debezium Documentation

This document provides comprehensive guidance on Debezium, a distributed platform for change data capture (CDC) that monitors databases and streams change events to other systems.

## 🎯 Overview

Debezium is an open-source distributed platform for change data capture (CDC). It captures row-level changes in databases and streams them to Apache Kafka, enabling real-time data synchronization and event-driven architectures.

### Key Benefits

1. **Real-time Change Capture**: Monitor database changes in real-time
2. **Low Latency**: Minimal impact on source database performance
3. **Reliable Delivery**: Exactly-once semantics with Kafka
4. **Schema Evolution**: Automatic schema registry integration
5. **Multiple Database Support**: MySQL, PostgreSQL, MongoDB, SQL Server, Oracle
6. **Scalable Architecture**: Distributed and fault-tolerant

## 🏗️ Debezium Architecture

### Core Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    Debezium Architecture                       │
├─────────────────────────────────────────────────────────────────┤
│  Source Database    │  Debezium Connector  │  Apache Kafka     │
│  ├── MySQL          │  ├── MySQL Connector │  ├── Change Events │
│  ├── PostgreSQL     │  ├── PG Connector    │  ├── Schema Events │
│  ├── MongoDB        │  ├── MongoDB Conn.   │  └── Heartbeat     │
│  └── SQL Server     │  └── SQL Server Conn.│                   │
├─────────────────────────────────────────────────────────────────┤
│  Kafka Connect      │  Schema Registry     │  Target Systems   │
│  ├── Worker Nodes   │  ├── Avro Schemas    │  ├── Data Lake     │
│  ├── REST API       │  ├── JSON Schemas    │  ├── Data Warehouse│
│  └── Configuration  │  └── Schema Evolution│  └── Analytics     │
└─────────────────────────────────────────────────────────────────┘
```

### Change Data Capture Process

1. **Database Monitoring**: Debezium monitors database transaction logs
2. **Change Detection**: Captures INSERT, UPDATE, DELETE operations
3. **Event Generation**: Creates structured change events
4. **Kafka Publishing**: Streams events to Kafka topics
5. **Schema Management**: Maintains schema evolution history

## 🔧 Debezium Connectors

### 1. **MySQL Connector**

#### Configuration
```json
{
  "name": "mysql-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "mysql-server",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "password",
    "database.server.id": "184054",
    "database.server.name": "mysql-server",
    "database.whitelist": "inventory",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "mysql-history",
    "include.schema.changes": "true",
    "snapshot.mode": "initial",
    "snapshot.locking.mode": "minimal",
    "provide.transaction.metadata": "true",
    "max.batch.size": "2048",
    "max.queue.size": "8192",
    "poll.interval.ms": "1000",
    "heartbeat.interval.ms": "30000",
    "heartbeat.topics.prefix": "__debezium-heartbeat"
  }
}
```

#### Advanced Configuration
```json
{
  "name": "mysql-connector-advanced",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "mysql-server",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "password",
    "database.server.id": "184054",
    "database.server.name": "mysql-server",
    "database.whitelist": "inventory",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "mysql-history",
    
    "snapshot.mode": "when_needed",
    "snapshot.locking.mode": "minimal",
    "snapshot.select.statement.overrides": "inventory.customers",
    "snapshot.select.statement.overrides.inventory.customers": "SELECT * FROM inventory.customers WHERE created_at > '2023-01-01'",
    
    "include.schema.changes": "true",
    "provide.transaction.metadata": "true",
    "tombstones.on.delete": "false",
    
    "max.batch.size": "2048",
    "max.queue.size": "8192",
    "poll.interval.ms": "1000",
    "heartbeat.interval.ms": "30000",
    
    "decimal.handling.mode": "precise",
    "time.precision.mode": "adaptive",
    "bigint.unsigned.handling.mode": "long",
    
    "transforms": "route",
    "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
    "transforms.route.regex": "mysql-server.inventory.(.*)",
    "transforms.route.replacement": "inventory-$1"
  }
}
```

### 2. **PostgreSQL Connector**

#### Configuration
```json
{
  "name": "postgres-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres-server",
    "database.port": "5432",
    "database.user": "debezium",
    "database.password": "password",
    "database.dbname": "postgres",
    "database.server.name": "postgres-server",
    "schema.whitelist": "public",
    "slot.name": "debezium_slot",
    "publication.name": "debezium_publication",
    "plugin.name": "pgoutput",
    "snapshot.mode": "initial",
    "snapshot.locking.mode": "minimal",
    "include.schema.changes": "true",
    "provide.transaction.metadata": "true",
    "max.batch.size": "2048",
    "max.queue.size": "8192",
    "poll.interval.ms": "1000",
    "heartbeat.interval.ms": "30000"
  }
}
```

#### Logical Replication Setup
```sql
-- Create replication user
CREATE USER debezium WITH REPLICATION LOGIN PASSWORD 'password';

-- Grant necessary permissions
GRANT SELECT ON ALL TABLES IN SCHEMA public TO debezium;
GRANT USAGE ON SCHEMA public TO debezium;

-- Create replication slot
SELECT pg_create_logical_replication_slot('debezium_slot', 'pgoutput');

-- Create publication
CREATE PUBLICATION debezium_publication FOR ALL TABLES;
```

### 3. **MongoDB Connector**

#### Configuration
```json
{
  "name": "mongodb-connector",
  "config": {
    "connector.class": "io.debezium.connector.mongodb.MongoDbConnector",
    "mongodb.hosts": "rs0/mongodb1:27017,mongodb2:27017,mongodb3:27017",
    "mongodb.user": "debezium",
    "mongodb.password": "password",
    "mongodb.name": "mongodb-server",
    "collection.whitelist": "inventory.products,inventory.customers",
    "snapshot.mode": "initial",
    "snapshot.fetch.size": "1000",
    "include.schema.changes": "true",
    "provide.transaction.metadata": "true",
    "max.batch.size": "2048",
    "max.queue.size": "8192",
    "poll.interval.ms": "1000",
    "heartbeat.interval.ms": "30000"
  }
}
```

## 🚀 Implementation Examples

### 1. **Docker Compose Setup**

#### Complete Debezium Stack
```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9101:9101"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost

  schema-registry:
    image: confluentinc/cp-schema-registry:7.4.0
    hostname: schema-registry
    container_name: schema-registry
    depends_on:
      - kafka
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: 'kafka:29092'

  kafka-connect:
    image: debezium/connect:2.4
    hostname: kafka-connect
    container_name: kafka-connect
    depends_on:
      - kafka
      - schema-registry
    ports:
      - "8083:8083"
    environment:
      BOOTSTRAP_SERVERS: kafka:29092
      GROUP_ID: 1
      CONFIG_STORAGE_TOPIC: my_connect_configs
      OFFSET_STORAGE_TOPIC: my_connect_offsets
      STATUS_STORAGE_TOPIC: my_connect_statuses
      CONFIG_STORAGE_REPLICATION_FACTOR: 1
      OFFSET_STORAGE_REPLICATION_FACTOR: 1
      STATUS_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_VALUE_CONVERTER: io.confluent.connect.avro.AvroConverter
      CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL: http://schema-registry:8081
      CONNECT_INTERNAL_KEY_CONVERTER: "org.apache.kafka.connect.json.JsonConverter"
      CONNECT_INTERNAL_VALUE_CONVERTER: "org.apache.kafka.connect.json.JsonConverter"
      CONNECT_REST_ADVERTISED_HOST_NAME: kafka-connect
      CONNECT_REST_PORT: 8083
      CONNECT_JMX_PORT: 9102
      CONNECT_JMX_HOSTNAME: localhost
      CONNECT_LOG4J_LOGGERS: org.apache.zookeeper=ERROR,org.I0Itec.zkclient=ERROR,org.reflections=ERROR

  mysql:
    image: mysql:8.0
    hostname: mysql
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_USER: debezium
      MYSQL_PASSWORD: password
      MYSQL_DATABASE: inventory
    command: >
      --server-id=1
      --log-bin=mysql-bin
      --binlog-format=row
      --binlog-row-image=full
      --expire-logs-days=7
      --binlog-do-db=inventory
      --gtid-mode=on
      --enforce-gtid-consistency=on

  postgres:
    image: postgres:15
    hostname: postgres
    container_name: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: debezium
      POSTGRES_PASSWORD: password
    command: >
      postgres
      -c wal_level=logical
      -c max_wal_senders=10
      -c max_replication_slots=10
      -c max_connections=200

  mongodb:
    image: mongo:6.0
    hostname: mongodb
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: debezium
      MONGO_INITDB_ROOT_PASSWORD: password
    command: mongod --replSet rs0 --bind_ip_all

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    hostname: kafka-ui
    container_name: kafka-ui
    depends_on:
      - kafka
      - schema-registry
      - kafka-connect
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
      KAFKA_CLUSTERS_0_ZOOKEEPER: zookeeper:2181
      KAFKA_CLUSTERS_0_SCHEMAREGISTRY: http://schema-registry:8081
      KAFKA_CLUSTERS_0_KAFKACONNECT_0_NAME: debezium
      KAFKA_CLUSTERS_0_KAFKACONNECT_0_ADDRESS: http://kafka-connect:8083
```

### 2. **Connector Management**

#### Python Connector Management
```python
import requests
import json
from typing import Dict, Any, List

class DebeziumConnectorManager:
    def __init__(self, connect_url: str = "http://localhost:8083"):
        self.connect_url = connect_url
        self.headers = {"Content-Type": "application/json"}
    
    def create_connector(self, connector_name: str, config: Dict[str, Any]) -> Dict[str, Any]:
        """Create a new Debezium connector"""
        url = f"{self.connect_url}/connectors"
        payload = {
            "name": connector_name,
            "config": config
        }
        
        response = requests.post(url, headers=self.headers, json=payload)
        response.raise_for_status()
        return response.json()
    
    def get_connector(self, connector_name: str) -> Dict[str, Any]:
        """Get connector information"""
        url = f"{self.connect_url}/connectors/{connector_name}"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def get_connector_status(self, connector_name: str) -> Dict[str, Any]:
        """Get connector status"""
        url = f"{self.connect_url}/connectors/{connector_name}/status"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def update_connector(self, connector_name: str, config: Dict[str, Any]) -> Dict[str, Any]:
        """Update connector configuration"""
        url = f"{self.connect_url}/connectors/{connector_name}/config"
        response = requests.put(url, headers=self.headers, json=config)
        response.raise_for_status()
        return response.json()
    
    def delete_connector(self, connector_name: str) -> Dict[str, Any]:
        """Delete connector"""
        url = f"{self.connect_url}/connectors/{connector_name}"
        response = requests.delete(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def restart_connector(self, connector_name: str) -> Dict[str, Any]:
        """Restart connector"""
        url = f"{self.connect_url}/connectors/{connector_name}/restart"
        response = requests.post(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def pause_connector(self, connector_name: str) -> Dict[str, Any]:
        """Pause connector"""
        url = f"{self.connect_url}/connectors/{connector_name}/pause"
        response = requests.put(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def resume_connector(self, connector_name: str) -> Dict[str, Any]:
        """Resume connector"""
        url = f"{self.connect_url}/connectors/{connector_name}/resume"
        response = requests.put(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def list_connectors(self) -> List[str]:
        """List all connectors"""
        url = f"{self.connect_url}/connectors"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def get_connector_tasks(self, connector_name: str) -> List[Dict[str, Any]]:
        """Get connector tasks"""
        url = f"{self.connect_url}/connectors/{connector_name}/tasks"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()

# Usage Example
def main():
    manager = DebeziumConnectorManager()
    
    # MySQL Connector Configuration
    mysql_config = {
        "connector.class": "io.debezium.connector.mysql.MySqlConnector",
        "database.hostname": "mysql",
        "database.port": "3306",
        "database.user": "debezium",
        "database.password": "password",
        "database.server.id": "184054",
        "database.server.name": "mysql-server",
        "database.whitelist": "inventory",
        "database.history.kafka.bootstrap.servers": "kafka:29092",
        "database.history.kafka.topic": "mysql-history",
        "include.schema.changes": "true",
        "snapshot.mode": "initial"
    }
    
    # Create MySQL connector
    try:
        result = manager.create_connector("mysql-connector", mysql_config)
        print(f"Connector created: {result}")
    except requests.exceptions.HTTPError as e:
        print(f"Error creating connector: {e}")
    
    # Check connector status
    try:
        status = manager.get_connector_status("mysql-connector")
        print(f"Connector status: {status}")
    except requests.exceptions.HTTPError as e:
        print(f"Error getting status: {e}")

if __name__ == "__main__":
    main()
```

### 3. **Event Processing**

#### Kafka Consumer for Debezium Events
```python
from kafka import KafkaConsumer
import json
from typing import Dict, Any, Optional

class DebeziumEventProcessor:
    def __init__(self, bootstrap_servers: str = "localhost:9092"):
        self.bootstrap_servers = bootstrap_servers
        self.consumer = None
    
    def create_consumer(self, topic: str, group_id: str = "debezium-processor"):
        """Create Kafka consumer for Debezium events"""
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=self.bootstrap_servers,
            group_id=group_id,
            value_deserializer=lambda x: json.loads(x.decode('utf-8')),
            auto_offset_reset='earliest',
            enable_auto_commit=True,
            auto_commit_interval_ms=1000
        )
        return self.consumer
    
    def process_events(self, topic: str, processor_func: callable):
        """Process Debezium events from topic"""
        consumer = self.create_consumer(topic)
        
        try:
            for message in consumer:
                event = message.value
                self.process_debezium_event(event, processor_func)
        except KeyboardInterrupt:
            print("Stopping consumer...")
        finally:
            consumer.close()
    
    def process_debezium_event(self, event: Dict[str, Any], processor_func: callable):
        """Process individual Debezium event"""
        # Extract event metadata
        source = event.get('source', {})
        op = event.get('op', '')
        ts_ms = event.get('ts_ms', 0)
        
        # Extract table information
        database = source.get('db', '')
        table = source.get('table', '')
        schema = source.get('schema', '')
        
        # Process based on operation type
        if op == 'c':  # Create/Insert
            data = event.get('after', {})
            processor_func('INSERT', database, schema, table, data, ts_ms)
        elif op == 'u':  # Update
            before = event.get('before', {})
            after = event.get('after', {})
            processor_func('UPDATE', database, schema, table, after, ts_ms, before)
        elif op == 'd':  # Delete
            data = event.get('before', {})
            processor_func('DELETE', database, schema, table, data, ts_ms)
        elif op == 'r':  # Read (Snapshot)
            data = event.get('after', {})
            processor_func('SNAPSHOT', database, schema, table, data, ts_ms)
        else:
            print(f"Unknown operation: {op}")

# Example event processor
def example_processor(operation: str, database: str, schema: str, 
                     table: str, data: Dict[str, Any], ts_ms: int, 
                     before_data: Optional[Dict[str, Any]] = None):
    """Example event processor function"""
    print(f"Operation: {operation}")
    print(f"Database: {database}")
    print(f"Schema: {schema}")
    print(f"Table: {table}")
    print(f"Timestamp: {ts_ms}")
    print(f"Data: {data}")
    
    if before_data:
        print(f"Before: {before_data}")
    
    # Add your processing logic here
    if operation == 'INSERT':
        # Handle insert
        pass
    elif operation == 'UPDATE':
        # Handle update
        pass
    elif operation == 'DELETE':
        # Handle delete
        pass
    elif operation == 'SNAPSHOT':
        # Handle snapshot
        pass

# Usage
if __name__ == "__main__":
    processor = DebeziumEventProcessor()
    processor.process_events("mysql-server.inventory.customers", example_processor)
```

## 🎯 Best Practices

### 1. **Performance Optimization**

#### Connector Tuning
```json
{
  "max.batch.size": "2048",
  "max.queue.size": "8192",
  "poll.interval.ms": "1000",
  "heartbeat.interval.ms": "30000",
  "snapshot.fetch.size": "1000",
  "query.fetch.size": "1000"
}
```

#### Kafka Tuning
```properties
# Kafka Producer Settings
batch.size=16384
linger.ms=5
compression.type=snappy
acks=all
retries=3

# Kafka Consumer Settings
fetch.min.bytes=1
fetch.max.wait.ms=500
max.partition.fetch.bytes=1048576
session.timeout.ms=30000
```

### 2. **Monitoring and Alerting**

#### Health Checks
```python
import requests
import time
from typing import Dict, Any

class DebeziumHealthMonitor:
    def __init__(self, connect_url: str = "http://localhost:8083"):
        self.connect_url = connect_url
    
    def check_connector_health(self, connector_name: str) -> Dict[str, Any]:
        """Check connector health status"""
        try:
            status = requests.get(f"{self.connect_url}/connectors/{connector_name}/status")
            status.raise_for_status()
            return status.json()
        except requests.exceptions.RequestException as e:
            return {"error": str(e), "healthy": False}
    
    def check_kafka_connect_health(self) -> Dict[str, Any]:
        """Check Kafka Connect cluster health"""
        try:
            response = requests.get(f"{self.connect_url}")
            response.raise_for_status()
            return {"healthy": True, "status": "running"}
        except requests.exceptions.RequestException as e:
            return {"healthy": False, "error": str(e)}
    
    def monitor_connector_lag(self, connector_name: str) -> Dict[str, Any]:
        """Monitor connector lag"""
        status = self.check_connector_health(connector_name)
        
        if "error" in status:
            return status
        
        tasks = status.get("tasks", [])
        total_lag = 0
        
        for task in tasks:
            if task.get("state") == "RUNNING":
                # Get task metrics (requires JMX or custom metrics)
                lag = self.get_task_lag(task.get("id"))
                total_lag += lag
        
        return {
            "connector": connector_name,
            "total_lag": total_lag,
            "healthy": total_lag < 1000  # Threshold for healthy lag
        }
    
    def get_task_lag(self, task_id: str) -> int:
        """Get lag for specific task"""
        # Implementation depends on your monitoring setup
        # This could query JMX metrics or custom metrics endpoint
        return 0
```

### 3. **Error Handling and Recovery**

#### Connector Error Recovery
```python
class DebeziumErrorHandler:
    def __init__(self, manager: DebeziumConnectorManager):
        self.manager = manager
    
    def handle_connector_failure(self, connector_name: str, max_retries: int = 3):
        """Handle connector failure with retry logic"""
        for attempt in range(max_retries):
            try:
                # Check connector status
                status = self.manager.get_connector_status(connector_name)
                
                # Check if connector is in failed state
                if status.get("connector", {}).get("state") == "FAILED":
                    print(f"Connector {connector_name} failed, attempting restart...")
                    self.manager.restart_connector(connector_name)
                    
                    # Wait and check if restart was successful
                    time.sleep(30)
                    new_status = self.manager.get_connector_status(connector_name)
                    
                    if new_status.get("connector", {}).get("state") == "RUNNING":
                        print(f"Connector {connector_name} restarted successfully")
                        return True
                
                # Check task failures
                tasks = status.get("tasks", [])
                for task in tasks:
                    if task.get("state") == "FAILED":
                        print(f"Task {task.get('id')} failed, restarting connector...")
                        self.manager.restart_connector(connector_name)
                        time.sleep(30)
                        break
                
                return True
                
            except Exception as e:
                print(f"Attempt {attempt + 1} failed: {e}")
                if attempt < max_retries - 1:
                    time.sleep(60)  # Wait before retry
        
        print(f"Failed to recover connector {connector_name} after {max_retries} attempts")
        return False
    
    def monitor_and_recover(self, connector_names: list):
        """Monitor multiple connectors and recover failures"""
        while True:
            for connector_name in connector_names:
                try:
                    status = self.manager.get_connector_status(connector_name)
                    connector_state = status.get("connector", {}).get("state")
                    
                    if connector_state == "FAILED":
                        print(f"Detected failure in {connector_name}")
                        self.handle_connector_failure(connector_name)
                    
                except Exception as e:
                    print(f"Error monitoring {connector_name}: {e}")
            
            time.sleep(60)  # Check every minute
```

## 🔗 Related Concepts

- [Data Ingestion Patterns](../../design-pattern/README.md)
- [Kafka Documentation](../data-streaming/README.md)
- [Change Data Capture](../../concepts/README.md)
- [Real-time Data Processing](../../architecture-designs/modern-data-architecture.md)

---

*Debezium provides a robust foundation for change data capture in modern data architectures. Success requires proper configuration, monitoring, and error handling to ensure reliable real-time data synchronization.*
