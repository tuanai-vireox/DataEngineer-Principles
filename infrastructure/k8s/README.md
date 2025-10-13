# Kubernetes for Data Infrastructure

This document provides comprehensive guidance on using Kubernetes for data infrastructure, covering deployment patterns, best practices, and real-world implementations for data engineering workloads.

## 🏗️ Overview

Kubernetes has become the de facto standard for container orchestration in modern data infrastructure. It provides the foundation for deploying, scaling, and managing data processing applications, streaming platforms, and analytics tools in a cloud-native environment.

### Why Kubernetes for Data Infrastructure?

1. **Scalability**: Auto-scaling based on workload demands
2. **Portability**: Run anywhere (cloud, on-premise, hybrid)
3. **Resource Efficiency**: Better resource utilization and cost optimization
4. **Operational Excellence**: Standardized deployment and management
5. **Ecosystem**: Rich ecosystem of data tools and operators
6. **High Availability**: Built-in fault tolerance and self-healing

## 📊 Data Infrastructure Components on Kubernetes

### Core Data Processing Platforms

#### Apache Spark on Kubernetes
```yaml
# spark-operator.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: spark-operator
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spark-operator
  namespace: spark-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spark-operator
  template:
    metadata:
      labels:
        app: spark-operator
    spec:
      serviceAccountName: spark-operator
      containers:
      - name: spark-operator
        image: gcr.io/spark-operator/spark-operator:v1beta2-1.3.3-3.1.1
        imagePullPolicy: Always
        env:
        - name: WATCH_NAMESPACE
          value: "default"
        - name: SPARK_APP_NAMESPACE
          value: "default"
        - name: INSTALL_CRD
          value: "true"
        - name: OPERATOR_NAME
          value: "spark-operator"
        - name: OPERATOR_IMAGE
          value: "gcr.io/spark-operator/spark-operator:v1beta2-1.3.3-3.1.1"
        - name: OPERATOR_IMAGE_PULL_POLICY
          value: "Always"
        - name: ENABLE_WEBHOOK
          value: "true"
        - name: ENABLE_METRICS
          value: "true"
        - name: METRICS_PORT
          value: "10254"
        - name: METRICS_PATH
          value: "/metrics"
        - name: LOG_LEVEL
          value: "2"
        ports:
        - containerPort: 10254
          name: metrics
        - containerPort: 8080
          name: webhook
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

#### Apache Kafka on Kubernetes
```yaml
# kafka-cluster.yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: kafka-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.5.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
      inter.broker.protocol.version: "3.5"
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 100Gi
        class: fast-ssd
    resources:
      requests:
        memory: 2Gi
        cpu: 1000m
      limits:
        memory: 4Gi
        cpu: 2000m
    jvmOptions:
      -Xms: 1g
      -Xmx: 3g
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 10Gi
      class: fast-ssd
    resources:
      requests:
        memory: 1Gi
        cpu: 500m
      limits:
        memory: 2Gi
        cpu: 1000m
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

#### Apache Flink on Kubernetes
```yaml
# flink-cluster.yaml
apiVersion: flink.apache.org/v1beta1
kind: FlinkDeployment
metadata:
  name: flink-cluster
  namespace: flink
spec:
  image: flink:1.17.1
  flinkVersion: v1_17
  flinkConfiguration:
    taskmanager.numberOfTaskSlots: "4"
    parallelism.default: "2"
    jobmanager.memory.process.size: "1600m"
    taskmanager.memory.process.size: "1728m"
    state.backend: rocksdb
    state.checkpoints.dir: s3://flink-checkpoints/checkpoints
    state.savepoints.dir: s3://flink-checkpoints/savepoints
    s3.endpoint: s3.amazonaws.com
    s3.path.style.access: true
  serviceAccount: flink
  jobManager:
    resource:
      memory: "1600m"
      cpu: 1
  taskManager:
    resource:
      memory: "1728m"
      cpu: 1
    replicas: 2
  podTemplate:
    spec:
      containers:
      - name: flink-main-container
        env:
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: access-key-id
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: secret-access-key
```

## 🔧 Data Processing Workloads

### Spark Job on Kubernetes

```yaml
# spark-job.yaml
apiVersion: "sparkoperator.k8s.io/v1beta2"
kind: SparkApplication
metadata:
  name: etl-job
  namespace: default
spec:
  type: Scala
  mode: cluster
  image: "gcr.io/spark-operator/spark:v3.1.1"
  imagePullPolicy: Always
  mainClass: com.example.ETLJob
  mainApplicationFile: "s3://spark-jars/etl-job.jar"
  sparkVersion: "3.1.1"
  restartPolicy:
    type: OnFailure
    onFailureRetries: 3
    onFailureRetryInterval: 10
    onSubmissionFailureRetries: 5
    onSubmissionFailureRetryInterval: 20
  driver:
    cores: 1
    coreLimit: "1200m"
    memory: "512m"
    labels:
      version: 3.1.1
    serviceAccount: spark
    env:
    - name: AWS_ACCESS_KEY_ID
      valueFrom:
        secretKeyRef:
          name: aws-credentials
          key: access-key-id
    - name: AWS_SECRET_ACCESS_KEY
      valueFrom:
        secretKeyRef:
          name: aws-credentials
          key: secret-access-key
  executor:
    cores: 2
    instances: 3
    memory: "1g"
    labels:
      version: 3.1.1
    env:
    - name: AWS_ACCESS_KEY_ID
      valueFrom:
        secretKeyRef:
          name: aws-credentials
          key: access-key-id
    - name: AWS_SECRET_ACCESS_KEY
      valueFrom:
        secretKeyRef:
          name: aws-credentials
          key: secret-access-key
```

### Airflow on Kubernetes

```yaml
# airflow-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: airflow-webserver
  namespace: airflow
spec:
  replicas: 2
  selector:
    matchLabels:
      app: airflow-webserver
  template:
    metadata:
      labels:
        app: airflow-webserver
    spec:
      serviceAccountName: airflow
      containers:
      - name: webserver
        image: apache/airflow:2.7.0
        command: ["airflow", "webserver"]
        args: ["--port", "8080"]
        ports:
        - containerPort: 8080
        env:
        - name: AIRFLOW__CORE__EXECUTOR
          value: "KubernetesExecutor"
        - name: AIRFLOW__KUBERNETES__NAMESPACE
          value: "airflow"
        - name: AIRFLOW__KUBERNETES__WORKER_CONTAINER_REPOSITORY
          value: "apache/airflow"
        - name: AIRFLOW__KUBERNETES__WORKER_CONTAINER_TAG
          value: "2.7.0"
        - name: AIRFLOW__KUBERNETES__DELETE_WORKER_PODS
          value: "True"
        - name: AIRFLOW__KUBERNETES__DELETE_WORKER_PODS_ON_FAILURE
          value: "True"
        - name: AIRFLOW__KUBERNETES__WORKER_PODS_CREATION_BATCH_SIZE
          value: "1"
        - name: AIRFLOW__KUBERNETES__WORKER_PODS_DELETION_BATCH_SIZE
          value: "1"
        - name: AIRFLOW__CORE__SQL_ALCHEMY_CONN
          valueFrom:
            secretKeyRef:
              name: airflow-secrets
              key: sql-alchemy-conn
        - name: AIRFLOW__CELERY__BROKER_URL
          valueFrom:
            secretKeyRef:
              name: airflow-secrets
              key: broker-url
        - name: AIRFLOW__CELERY__RESULT_BACKEND
          valueFrom:
            secretKeyRef:
              name: airflow-secrets
              key: result-backend
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        volumeMounts:
        - name: airflow-dags
          mountPath: /opt/airflow/dags
        - name: airflow-logs
          mountPath: /opt/airflow/logs
      volumes:
      - name: airflow-dags
        persistentVolumeClaim:
          claimName: airflow-dags-pvc
      - name: airflow-logs
        persistentVolumeClaim:
          claimName: airflow-logs-pvc
```

## 🗄️ Data Storage on Kubernetes

### StatefulSets for Data Storage

```yaml
# postgres-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: data
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: "analytics"
        - name: POSTGRES_USER
          value: "postgres"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - exec pg_isready -U postgres -h 127.0.0.1 -p 5432
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - exec pg_isready -U postgres -h 127.0.0.1 -p 5432
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
      storageClassName: fast-ssd
```

### Redis Cluster

```yaml
# redis-cluster.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
  namespace: data
spec:
  serviceName: redis-cluster
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        command:
        - redis-server
        - /etc/redis/redis.conf
        - --cluster-announce-ip
        - $(POD_IP)
        - --cluster-announce-port
        - "6379"
        - --cluster-announce-bus-port
        - "16379"
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        volumeMounts:
        - name: redis-config
          mountPath: /etc/redis
        - name: redis-data
          mountPath: /data
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
      volumes:
      - name: redis-config
        configMap:
          name: redis-config
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

## 🔄 Data Pipeline Orchestration

### Argo Workflows for Data Pipelines

```yaml
# data-pipeline-workflow.yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  name: data-pipeline
  namespace: data
spec:
  entrypoint: data-pipeline
  templates:
  - name: data-pipeline
    dag:
      tasks:
      - name: extract-data
        template: extract
      - name: transform-data
        template: transform
        dependencies: [extract-data]
      - name: load-data
        template: load
        dependencies: [transform-data]
      - name: validate-data
        template: validate
        dependencies: [load-data]
  
  - name: extract
    container:
      image: python:3.9
      command: [python]
      source: |
        import requests
        import pandas as pd
        
        # Extract data from API
        response = requests.get('https://api.example.com/data')
        data = response.json()
        
        # Save to S3
        df = pd.DataFrame(data)
        df.to_parquet('s3://data-lake/bronze/extracted_data.parquet')
      env:
      - name: AWS_ACCESS_KEY_ID
        valueFrom:
          secretKeyRef:
            name: aws-credentials
            key: access-key-id
      - name: AWS_SECRET_ACCESS_KEY
        valueFrom:
          secretKeyRef:
            name: aws-credentials
            key: secret-access-key
      resources:
        requests:
          memory: "512Mi"
          cpu: "200m"
        limits:
          memory: "1Gi"
          cpu: "500m"
  
  - name: transform
    container:
      image: apache/spark:3.1.1
      command: [spark-submit]
      args:
      - --class
      - com.example.TransformJob
      - --master
      - k8s://https://kubernetes.default.svc
      - --deploy-mode
      - cluster
      - --conf
      - spark.kubernetes.authenticate.driver.serviceAccountName=spark
      - --conf
      - spark.kubernetes.container.image=apache/spark:3.1.1
      - s3://spark-jars/transform-job.jar
      env:
      - name: AWS_ACCESS_KEY_ID
        valueFrom:
          secretKeyRef:
            name: aws-credentials
            key: access-key-id
      - name: AWS_SECRET_ACCESS_KEY
        valueFrom:
          secretKeyRef:
            name: aws-credentials
            key: secret-access-key
      resources:
        requests:
          memory: "2Gi"
          cpu: "1000m"
        limits:
          memory: "4Gi"
          cpu: "2000m"
  
  - name: load
    container:
      image: python:3.9
      command: [python]
      source: |
        import pandas as pd
        from sqlalchemy import create_engine
        
        # Load data to data warehouse
        df = pd.read_parquet('s3://data-lake/silver/transformed_data.parquet')
        
        engine = create_engine('postgresql://user:pass@postgres:5432/analytics')
        df.to_sql('processed_data', engine, if_exists='replace', index=False)
      env:
      - name: AWS_ACCESS_KEY_ID
        valueFrom:
          secretKeyRef:
            name: aws-credentials
            key: access-key-id
      - name: AWS_SECRET_ACCESS_KEY
        valueFrom:
          secretKeyRef:
            name: aws-credentials
            key: secret-access-key
      resources:
        requests:
          memory: "512Mi"
          cpu: "200m"
        limits:
          memory: "1Gi"
          cpu: "500m"
  
  - name: validate
    container:
      image: python:3.9
      command: [python]
      source: |
        import pandas as pd
        from sqlalchemy import create_engine
        
        # Validate data quality
        engine = create_engine('postgresql://user:pass@postgres:5432/analytics')
        df = pd.read_sql('SELECT * FROM processed_data', engine)
        
        # Data quality checks
        assert df.isnull().sum().sum() == 0, "Null values found"
        assert len(df) > 0, "No data loaded"
        assert df['amount'].min() >= 0, "Negative amounts found"
        
        print("Data validation passed!")
      resources:
        requests:
          memory: "256Mi"
          cpu: "100m"
        limits:
          memory: "512Mi"
          cpu: "300m"
```

## 📊 Monitoring and Observability

### Prometheus Monitoring Stack

```yaml
# prometheus-monitoring.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    rule_files:
      - "alert_rules.yml"
    
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              - alertmanager:9093
    
    scrape_configs:
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
            action: replace
            target_label: __metrics_path__
            regex: (.+)
          - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
            action: replace
            regex: ([^:]+)(?::\d+)?;(\d+)
            replacement: $1:$2
            target_label: __address__
          - action: labelmap
            regex: __meta_kubernetes_pod_label_(.+)
          - source_labels: [__meta_kubernetes_namespace]
            action: replace
            target_label: kubernetes_namespace
          - source_labels: [__meta_kubernetes_pod_name]
            action: replace
            target_label: kubernetes_pod_name
      
      - job_name: 'spark-applications'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_label_spark_role]
            action: keep
            regex: driver
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
            action: replace
            regex: ([^:]+)(?::\d+)?;(\d+)
            replacement: $1:$2
            target_label: __address__
      
      - job_name: 'kafka-cluster'
        static_configs:
          - targets: ['kafka-cluster-kafka-bootstrap:9092']
        metrics_path: /metrics
        scrape_interval: 30s
      
      - job_name: 'flink-cluster'
        static_configs:
          - targets: ['flink-cluster-rest:8081']
        metrics_path: /metrics
        scrape_interval: 30s

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:v2.40.0
        ports:
        - containerPort: 9090
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus/'
          - '--web.console.libraries=/etc/prometheus/console_libraries'
          - '--web.console.templates=/etc/prometheus/consoles'
          - '--storage.tsdb.retention.time=200h'
          - '--web.enable-lifecycle'
        volumeMounts:
        - name: prometheus-config-volume
          mountPath: /etc/prometheus/
        - name: prometheus-storage-volume
          mountPath: /prometheus/
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
      volumes:
      - name: prometheus-config-volume
        configMap:
          defaultMode: 420
          name: prometheus-config
      - name: prometheus-storage-volume
        persistentVolumeClaim:
          claimName: prometheus-pvc
```

### Grafana Dashboards

```yaml
# grafana-dashboard.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard-data-pipeline
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  data-pipeline-dashboard.json: |
    {
      "dashboard": {
        "id": null,
        "title": "Data Pipeline Monitoring",
        "tags": ["data", "pipeline"],
        "timezone": "browser",
        "panels": [
          {
            "id": 1,
            "title": "Pipeline Success Rate",
            "type": "stat",
            "targets": [
              {
                "expr": "rate(pipeline_jobs_completed_total{status=\"success\"}[5m]) / rate(pipeline_jobs_completed_total[5m]) * 100",
                "legendFormat": "Success Rate %"
              }
            ],
            "fieldConfig": {
              "defaults": {
                "unit": "percent",
                "min": 0,
                "max": 100,
                "thresholds": {
                  "steps": [
                    {"color": "red", "value": 0},
                    {"color": "yellow", "value": 80},
                    {"color": "green", "value": 95}
                  ]
                }
              }
            }
          },
          {
            "id": 2,
            "title": "Data Processing Throughput",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(data_records_processed_total[5m])",
                "legendFormat": "Records/sec"
              }
            ],
            "yAxes": [
              {
                "label": "Records per second",
                "min": 0
              }
            ]
          },
          {
            "id": 3,
            "title": "Spark Application Status",
            "type": "table",
            "targets": [
              {
                "expr": "spark_application_info",
                "format": "table"
              }
            ]
          },
          {
            "id": 4,
            "title": "Kafka Consumer Lag",
            "type": "graph",
            "targets": [
              {
                "expr": "kafka_consumer_lag_sum",
                "legendFormat": "{{topic}} - {{consumer_group}}"
              }
            ]
          }
        ],
        "time": {
          "from": "now-1h",
          "to": "now"
        },
        "refresh": "30s"
      }
    }
```

## 🔒 Security and Access Control

### RBAC for Data Applications

```yaml
# data-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: spark-service-account
  namespace: data
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: data
  name: spark-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: spark-role-binding
  namespace: data
subjects:
- kind: ServiceAccount
  name: spark-service-account
  namespace: data
roleRef:
  kind: Role
  name: spark-role
  apiGroup: rbac.authorization.k8s.io
```

### Network Policies

```yaml
# network-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: data-pipeline-network-policy
  namespace: data
spec:
  podSelector:
    matchLabels:
      app: data-pipeline
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    - podSelector:
        matchLabels:
          app: airflow
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: data
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector:
        matchLabels:
          name: kafka
    ports:
    - protocol: TCP
      port: 9092
  - to: []
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 80
```

## 🚀 Auto-scaling and Resource Management

### Horizontal Pod Autoscaler

```yaml
# hpa-spark-driver.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: spark-driver-hpa
  namespace: data
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: spark-driver
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

### Vertical Pod Autoscaler

```yaml
# vpa-spark-executor.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: spark-executor-vpa
  namespace: data
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: spark-executor
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: spark-executor
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 4
        memory: 8Gi
      controlledResources: ["cpu", "memory"]
```

## 📈 Best Practices

### Resource Management

1. **Resource Requests and Limits**
   - Always set resource requests and limits
   - Use resource quotas to prevent resource starvation
   - Monitor resource utilization and adjust accordingly

2. **Node Affinity and Taints**
   - Use node affinity for data-intensive workloads
   - Apply taints to dedicated nodes for specific workloads
   - Consider spot instances for batch processing

3. **Storage Management**
   - Use appropriate storage classes for different workloads
   - Implement backup and disaster recovery strategies
   - Monitor storage usage and implement cleanup policies

### Security Best Practices

1. **Service Accounts**
   - Use dedicated service accounts for each application
   - Implement least privilege access
   - Regularly rotate service account tokens

2. **Secrets Management**
   - Use Kubernetes secrets or external secret management
   - Encrypt secrets at rest and in transit
   - Implement secret rotation policies

3. **Network Security**
   - Use network policies to restrict traffic
   - Implement service mesh for advanced networking
   - Monitor network traffic and detect anomalies

### Monitoring and Observability

1. **Comprehensive Monitoring**
   - Monitor application metrics, infrastructure metrics, and business metrics
   - Set up alerting for critical issues
   - Implement distributed tracing for complex workflows

2. **Log Management**
   - Centralize logs using ELK stack or similar
   - Implement log aggregation and analysis
   - Set up log-based alerting

3. **Performance Monitoring**
   - Monitor application performance and resource utilization
   - Implement performance testing and benchmarking
   - Use profiling tools to identify bottlenecks

## 🔧 Troubleshooting Common Issues

### Spark on Kubernetes Issues

1. **Driver Pod Stuck in Pending**
   - Check resource requests and node capacity
   - Verify node selectors and affinity rules
   - Check for resource quotas

2. **Executor Pods Failing**
   - Check executor resource limits
   - Verify network connectivity
   - Check for image pull issues

3. **Slow Job Performance**
   - Optimize resource allocation
   - Check data locality
   - Review Spark configuration

### Kafka on Kubernetes Issues

1. **Pod Startup Issues**
   - Check persistent volume claims
   - Verify resource requests
   - Check for port conflicts

2. **Data Loss**
   - Verify replication factor settings
   - Check min.insync.replicas configuration
   - Monitor broker health

3. **Performance Issues**
   - Check disk I/O performance
   - Verify network configuration
   - Review Kafka configuration

## 🚀 Deployment Strategies

### GitOps with ArgoCD

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: data-infrastructure
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/data-infrastructure-k8s
    targetRevision: HEAD
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: data
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
```

### Helm Charts for Data Applications

```yaml
# values.yaml for Spark Helm Chart
spark:
  image:
    repository: apache/spark
    tag: "3.1.1"
    pullPolicy: IfNotPresent
  
  serviceAccount:
    create: true
    name: spark
  
  driver:
    cores: 1
    memory: "1g"
    serviceType: ClusterIP
  
  executor:
    cores: 2
    instances: 3
    memory: "2g"
  
  conf:
    "spark.kubernetes.authenticate.driver.serviceAccountName": "spark"
    "spark.kubernetes.container.image": "apache/spark:3.1.1"
    "spark.sql.adaptive.enabled": "true"
    "spark.sql.adaptive.coalescePartitions.enabled": "true"
  
  resources:
    driver:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
    executor:
      requests:
        memory: "2Gi"
        cpu: "1000m"
      limits:
        memory: "4Gi"
        cpu: "2000m"
```

## 🔗 Related Resources

- [Modern Data Architecture](../architecture-designs/modern-data-architecture.md)
- [Design Patterns](../design-pattern/README.md)
- [Data Governance](../concepts/security/README.md)
- [Interview Questions](../interview-questions/README.md)

---

*This document provides comprehensive guidance for implementing data infrastructure on Kubernetes. Regular updates and best practices should be incorporated based on evolving requirements and technology advancements.*
