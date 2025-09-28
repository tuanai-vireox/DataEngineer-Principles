# Apache Spark Performance Tuning

This document provides comprehensive guidance on optimizing Apache Spark performance for data engineering workloads, covering configuration tuning, resource optimization, and best practices.

## 🎯 Overview

Spark performance tuning involves optimizing various aspects of the Spark application to achieve better throughput, lower latency, and efficient resource utilization. This guide covers the key areas that impact performance.

## ⚙️ Configuration Tuning

### 1. **Memory Configuration**

#### Driver Memory
```python
# Driver memory configuration
def configure_driver_memory():
    """
    Configure driver memory based on workload requirements
    """
    
    # For small datasets (< 1GB)
    spark.conf.set("spark.driver.memory", "2g")
    
    # For medium datasets (1-10GB)
    spark.conf.set("spark.driver.memory", "4g")
    
    # For large datasets (> 10GB)
    spark.conf.set("spark.driver.memory", "8g")
    
    # Driver memory fraction for caching
    spark.conf.set("spark.driver.memoryFraction", "0.6")
    
    return "Driver memory configured"
```

#### Executor Memory
```python
# Executor memory configuration
def configure_executor_memory():
    """
    Configure executor memory and memory fractions
    """
    
    # Total executor memory
    spark.conf.set("spark.executor.memory", "4g")
    
    # Memory fraction for caching
    spark.conf.set("spark.executor.memoryFraction", "0.6")
    
    # Memory fraction for shuffle operations
    spark.conf.set("spark.shuffle.memoryFraction", "0.2")
    
    # Memory fraction for storage
    spark.conf.set("spark.storage.memoryFraction", "0.6")
    
    return "Executor memory configured"
```

### 2. **CPU Configuration**

#### Executor Cores
```python
# CPU configuration
def configure_cpu():
    """
    Configure CPU resources for executors
    """
    
    # Number of cores per executor
    spark.conf.set("spark.executor.cores", "4")
    
    # Total number of cores
    spark.conf.set("spark.cores.max", "200")
    
    # Number of executors
    spark.conf.set("spark.executor.instances", "50")
    
    return "CPU configuration set"
```

### 3. **Shuffle Configuration**

#### Shuffle Partitions
```python
# Shuffle partitions configuration
def configure_shuffle_partitions():
    """
    Configure shuffle partitions for optimal performance
    """
    
    # Rule of thumb: 2-3x the number of cores
    num_cores = 200
    optimal_partitions = num_cores * 2
    
    spark.conf.set("spark.sql.shuffle.partitions", str(optimal_partitions))
    
    # For specific operations
    df.groupBy("key").agg({"value": "sum"}).repartition(optimal_partitions)
    
    return "Shuffle partitions configured"
```

#### Shuffle Compression
```python
# Shuffle compression configuration
def configure_shuffle_compression():
    """
    Enable compression to reduce network I/O
    """
    
    # Enable shuffle compression
    spark.conf.set("spark.shuffle.compress", "true")
    spark.conf.set("spark.shuffle.spill.compress", "true")
    
    # Use efficient compression codec
    spark.conf.set("spark.io.compression.codec", "snappy")
    
    return "Shuffle compression configured"
```

## 🚀 Performance Optimization Techniques

### 1. **Data Caching**

#### Cache Strategies
```python
# Data caching strategies
def implement_caching_strategies():
    """
    Implement different caching strategies based on data access patterns
    """
    
    # Cache frequently accessed data
    df.cache()
    
    # Persist with different storage levels
    df.persist(StorageLevel.MEMORY_AND_DISK_SER)
    
    # Unpersist when no longer needed
    df.unpersist()
    
    # Check cache status
    print(f"Cache status: {df.is_cached}")
    
    return "Caching strategies implemented"
```

#### Storage Levels
```python
# Storage levels for different use cases
def choose_storage_levels():
    """
    Choose appropriate storage levels based on data characteristics
    """
    
    from pyspark import StorageLevel
    
    # For small datasets that fit in memory
    df.persist(StorageLevel.MEMORY_ONLY)
    
    # For large datasets with serialization
    df.persist(StorageLevel.MEMORY_ONLY_SER)
    
    # For datasets that might not fit in memory
    df.persist(StorageLevel.MEMORY_AND_DISK)
    
    # For large datasets with serialization and disk spill
    df.persist(StorageLevel.MEMORY_AND_DISK_SER)
    
    # For datasets that are expensive to compute
    df.persist(StorageLevel.DISK_ONLY)
    
    return "Storage levels configured"
```

### 2. **Data Partitioning**

#### Partitioning Strategies
```python
# Data partitioning strategies
def implement_partitioning_strategies():
    """
    Implement different partitioning strategies for optimal performance
    """
    
    # Hash partitioning
    df.repartition(10, "key")
    
    # Range partitioning
    df.repartition(10, col("key"))
    
    # Custom partitioning
    df.repartition(10, "key").write.partitionBy("date")
    
    # Coalesce to reduce partitions
    df.coalesce(5)
    
    return "Partitioning strategies implemented"
```

#### Partition Optimization
```python
# Partition optimization
def optimize_partitions():
    """
    Optimize partition size and number for better performance
    """
    
    # Optimal partition size: 128MB - 1GB
    target_partition_size = 256 * 1024 * 1024  # 256MB
    
    # Calculate optimal number of partitions
    total_size = df.count() * 8  # Assuming 8 bytes per row
    optimal_partitions = max(1, total_size // target_partition_size)
    
    # Repartition to optimal size
    df.repartition(optimal_partitions)
    
    return "Partitions optimized"
```

### 3. **Join Optimization**

#### Broadcast Joins
```python
# Broadcast join optimization
def optimize_joins():
    """
    Use broadcast joins for small tables
    """
    
    # Set broadcast threshold
    spark.conf.set("spark.sql.autoBroadcastJoinThreshold", "10MB")
    
    # Manual broadcast join
    small_df = spark.read.parquet("small_table.parquet")
    large_df = spark.read.parquet("large_table.parquet")
    
    # Broadcast join
    result = large_df.join(
        broadcast(small_df), 
        "key", 
        "inner"
    )
    
    return "Joins optimized"
```

#### Bucketed Joins
```python
# Bucketed joins
def implement_bucketed_joins():
    """
    Use bucketing to avoid shuffling during joins
    """
    
    # Create bucketed table
    df.write \
      .mode("overwrite") \
      .bucketBy(10, "key") \
      .sortBy("key") \
      .saveAsTable("bucketed_table")
    
    # Join bucketed tables
    bucketed_df1 = spark.table("bucketed_table1")
    bucketed_df2 = spark.table("bucketed_table2")
    
    result = bucketed_df1.join(bucketed_df2, "key")
    
    return "Bucketed joins implemented"
```

### 4. **Query Optimization**

#### Catalyst Optimizer
```python
# Catalyst optimizer configuration
def configure_catalyst_optimizer():
    """
    Configure Catalyst optimizer for better query performance
    """
    
    # Enable adaptive query execution
    spark.conf.set("spark.sql.adaptive.enabled", "true")
    spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
    spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
    
    # Enable cost-based optimizer
    spark.conf.set("spark.sql.cbo.enabled", "true")
    spark.conf.set("spark.sql.cbo.joinReorder.enabled", "true")
    
    # Enable column pruning
    spark.conf.set("spark.sql.optimizer.metadataOnly", "true")
    
    return "Catalyst optimizer configured"
```

#### Predicate Pushdown
```python
# Predicate pushdown optimization
def implement_predicate_pushdown():
    """
    Implement predicate pushdown for better performance
    """
    
    # Filter early in the pipeline
    df.filter(col("date") >= "2023-01-01") \
      .filter(col("amount") > 100) \
      .groupBy("category") \
      .sum("amount")
    
    # Use column pruning
    df.select("key", "value").filter(col("value") > 100)
    
    return "Predicate pushdown implemented"
```

## 📊 Resource Management

### 1. **Dynamic Allocation**

#### Enable Dynamic Allocation
```python
# Dynamic allocation configuration
def configure_dynamic_allocation():
    """
    Configure dynamic allocation for better resource utilization
    """
    
    # Enable dynamic allocation
    spark.conf.set("spark.dynamicAllocation.enabled", "true")
    
    # Set allocation parameters
    spark.conf.set("spark.dynamicAllocation.minExecutors", "1")
    spark.conf.set("spark.dynamicAllocation.maxExecutors", "100")
    spark.conf.set("spark.dynamicAllocation.initialExecutors", "10")
    
    # Set allocation intervals
    spark.conf.set("spark.dynamicAllocation.schedulerBacklogTimeout", "1s")
    spark.conf.set("spark.dynamicAllocation.sustainedSchedulerBacklogTimeout", "10s")
    
    return "Dynamic allocation configured"
```

### 2. **Resource Monitoring**

#### Monitor Resource Usage
```python
# Resource monitoring
def monitor_resources():
    """
    Monitor resource usage and performance metrics
    """
    
    # Monitor executor memory usage
    def check_executor_memory():
        # Check Spark UI for memory usage
        # Monitor GC pressure
        # Check for spill events
        pass
    
    # Monitor CPU usage
    def check_cpu_usage():
        # Check CPU utilization
        # Monitor task scheduling
        # Check for CPU bottlenecks
        pass
    
    # Monitor network usage
    def check_network_usage():
        # Check shuffle read/write metrics
        # Monitor network I/O
        # Check for network bottlenecks
        pass
    
    return "Resource monitoring implemented"
```

## 🔧 Advanced Optimization

### 1. **Serialization**

#### Kryo Serialization
```python
# Kryo serialization configuration
def configure_kryo_serialization():
    """
    Configure Kryo serialization for better performance
    """
    
    # Enable Kryo serialization
    spark.conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    
    # Register custom classes
    spark.conf.set("spark.kryo.registrationRequired", "true")
    spark.conf.set("spark.kryo.unsafe", "true")
    
    # Configure Kryo buffer size
    spark.conf.set("spark.kryoserializer.buffer.max", "64m")
    
    return "Kryo serialization configured"
```

### 2. **Garbage Collection**

#### GC Optimization
```python
# Garbage collection optimization
def optimize_garbage_collection():
    """
    Optimize garbage collection for better performance
    """
    
    # Use G1GC for large heaps
    spark.conf.set("spark.executor.extraJavaOptions", 
                   "-XX:+UseG1GC -XX:+UnlockExperimentalVMOptions -XX:+UseStringDeduplication")
    
    # Tune GC parameters
    spark.conf.set("spark.executor.extraJavaOptions",
                   "-XX:MaxGCPauseMillis=200 -XX:G1HeapRegionSize=16m")
    
    # Monitor GC performance
    spark.conf.set("spark.executor.extraJavaOptions",
                   "-XX:+PrintGCDetails -XX:+PrintGCTimeStamps")
    
    return "Garbage collection optimized"
```

### 3. **Data Skew Handling**

#### Detect and Handle Skew
```python
# Data skew handling
def handle_data_skew():
    """
    Detect and handle data skew for better performance
    """
    
    # Detect skew
    def detect_skew(df, key_column):
        partition_counts = df.groupBy(key_column).count().collect()
        counts = [row['count'] for row in partition_counts]
        return max(counts) / min(counts) if min(counts) > 0 else float('inf')
    
    # Handle skew with salting
    def handle_skew_with_salting(df, key_column):
        # Add random salt
        salted_df = df.withColumn(
            "salted_key", 
            concat(col(key_column), lit("_"), (rand() * 100).cast("int"))
        )
        
        # Perform aggregation
        aggregated_df = salted_df.groupBy("salted_key").agg(
            sum("value").alias("sum_value")
        )
        
        # Remove salt and aggregate
        final_df = aggregated_df.withColumn(
            "original_key", 
            split(col("salted_key"), "_")[0]
        ).groupBy("original_key").agg(
            sum("sum_value").alias("total_value")
        )
        
        return final_df
    
    return "Data skew handling implemented"
```

## 📈 Performance Monitoring

### 1. **Metrics Collection**

#### Collect Performance Metrics
```python
# Performance metrics collection
def collect_performance_metrics():
    """
    Collect and analyze performance metrics
    """
    
    # Enable detailed metrics
    spark.conf.set("spark.sql.adaptive.enabled", "true")
    spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
    spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
    
    # Monitor in Spark UI
    # - Job execution time
    # - Stage execution time
    # - Task execution time
    # - Shuffle read/write metrics
    # - Memory usage
    # - GC metrics
    
    return "Performance metrics collection enabled"
```

### 2. **Performance Profiling**

#### Profile Spark Applications
```python
# Performance profiling
def profile_spark_application():
    """
    Profile Spark application for performance bottlenecks
    """
    
    # Use Spark UI for profiling
    # - Analyze job DAG
    # - Check stage dependencies
    # - Monitor task distribution
    # - Analyze shuffle metrics
    
    # Use external profiling tools
    # - JProfiler
    # - YourKit
    # - VisualVM
    
    return "Performance profiling implemented"
```

## 🎯 Best Practices

### 1. **Configuration Best Practices**
- Set appropriate memory fractions
- Use dynamic allocation
- Enable compression
- Configure serialization

### 2. **Data Processing Best Practices**
- Cache frequently accessed data
- Use appropriate partitioning
- Optimize joins
- Handle data skew

### 3. **Resource Management Best Practices**
- Monitor resource usage
- Tune garbage collection
- Optimize network I/O
- Use efficient data formats

### 4. **Monitoring Best Practices**
- Collect performance metrics
- Profile applications regularly
- Monitor resource utilization
- Set up alerting

## 🔗 Related Documentation

- [Spark Shuffling](./spark-shuffling.md)
- [Data Partitioning Strategies](./data-partitioning.md)
- [Join Optimization](./join-optimization.md)
- [Memory Management](./memory-management.md)

---

*Regular performance tuning and monitoring are essential for maintaining optimal Spark application performance. This guide provides a foundation for understanding and implementing performance optimization strategies.*
