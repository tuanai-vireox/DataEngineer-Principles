# Terraform for Data Engineering Infrastructure

This document provides comprehensive guidance on using Terraform for Infrastructure as Code (IaC) in data engineering environments, covering fundamentals, best practices, and practical implementations.

## 🎯 Overview

Terraform is an open-source Infrastructure as Code (IaC) tool that enables you to define, provision, and manage cloud infrastructure using declarative configuration files. It's essential for data engineering teams to maintain consistent, reproducible, and scalable infrastructure.

### Key Benefits

1. **Infrastructure as Code**: Version control for infrastructure
2. **Multi-Cloud Support**: Works across AWS, GCP, Azure, and others
3. **State Management**: Tracks infrastructure state and changes
4. **Dependency Management**: Handles resource dependencies automatically
5. **Plan and Apply**: Preview changes before applying them
6. **Modularity**: Reusable modules and components

## 🏗️ Terraform Fundamentals

### Core Concepts

#### 1. **Providers**
```hcl
# Provider Configuration
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# AWS Provider
provider "aws" {
  region = var.aws_region
  profile = var.aws_profile
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = var.project_name
      ManagedBy   = "terraform"
    }
  }
}

# GCP Provider
provider "google" {
  project = var.gcp_project_id
  region  = var.gcp_region
}

# Azure Provider
provider "azurerm" {
  features {}
  subscription_id = var.azure_subscription_id
  tenant_id       = var.azure_tenant_id
}
```

#### 2. **Resources**
```hcl
# AWS S3 Bucket for Data Lake
resource "aws_s3_bucket" "data_lake" {
  bucket = "${var.project_name}-data-lake-${var.environment}"
  
  tags = {
    Name        = "Data Lake"
    Environment = var.environment
    Purpose     = "data-storage"
  }
}

resource "aws_s3_bucket_versioning" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_encryption" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

# GCP Cloud Storage Bucket
resource "google_storage_bucket" "data_lake" {
  name          = "${var.project_name}-data-lake-${var.environment}"
  location      = var.gcp_region
  force_destroy = false
  
  versioning {
    enabled = true
  }
  
  encryption {
    default_kms_key_name = google_kms_crypto_key.data_lake_key.id
  }
  
  lifecycle_rule {
    condition {
      age = 30
    }
    action {
      type = "SetStorageClass"
      storage_class = "NEARLINE"
    }
  }
}

# Azure Data Lake Storage Gen2
resource "azurerm_storage_account" "data_lake" {
  name                     = "${var.project_name}datalake${var.environment}"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  account_kind             = "StorageV2"
  is_hns_enabled           = true
  
  tags = {
    Environment = var.environment
    Purpose     = "data-storage"
  }
}

resource "azurerm_storage_data_lake_gen2_filesystem" "data_lake" {
  name               = "datalake"
  storage_account_id = azurerm_storage_account.data_lake.id
  
  properties = {
    hello = "aGVsbG8="
  }
}
```

#### 3. **Variables**
```hcl
# variables.tf
variable "project_name" {
  description = "Name of the project"
  type        = string
  default     = "data-engineering"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "gcp_project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "gcp_region" {
  description = "GCP region"
  type        = string
  default     = "us-central1"
}

variable "azure_subscription_id" {
  description = "Azure subscription ID"
  type        = string
}

variable "azure_tenant_id" {
  description = "Azure tenant ID"
  type        = string
}

variable "data_retention_days" {
  description = "Data retention period in days"
  type        = number
  default     = 2555 # 7 years
}

variable "allowed_cidr_blocks" {
  description = "List of CIDR blocks allowed to access resources"
  type        = list(string)
  default     = ["10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16"]
}
```

#### 4. **Outputs**
```hcl
# outputs.tf
output "data_lake_bucket_name" {
  description = "Name of the data lake S3 bucket"
  value       = aws_s3_bucket.data_lake.bucket
}

output "data_lake_bucket_arn" {
  description = "ARN of the data lake S3 bucket"
  value       = aws_s3_bucket.data_lake.arn
}

output "data_lake_bucket_domain_name" {
  description = "Domain name of the data lake S3 bucket"
  value       = aws_s3_bucket.data_lake.bucket_domain_name
}

output "gcp_data_lake_bucket_name" {
  description = "Name of the GCP data lake bucket"
  value       = google_storage_bucket.data_lake.name
}

output "gcp_data_lake_bucket_url" {
  description = "URL of the GCP data lake bucket"
  value       = google_storage_bucket.data_lake.url
}

output "azure_data_lake_name" {
  description = "Name of the Azure data lake storage account"
  value       = azurerm_storage_account.data_lake.name
}

output "azure_data_lake_primary_endpoint" {
  description = "Primary endpoint of the Azure data lake storage account"
  value       = azurerm_storage_account.data_lake.primary_dfs_endpoint
}
```

## 🏗️ Data Engineering Infrastructure Patterns

### 1. **Data Lake Infrastructure**

#### AWS Data Lake
```hcl
# data-lake-aws.tf
resource "aws_s3_bucket" "data_lake" {
  bucket = "${var.project_name}-data-lake-${var.environment}"
  
  tags = {
    Name        = "Data Lake"
    Environment = var.environment
    Purpose     = "data-storage"
  }
}

# Data Lake Folder Structure
resource "aws_s3_object" "data_lake_folders" {
  for_each = toset([
    "bronze/",
    "silver/",
    "gold/",
    "raw/",
    "processed/",
    "analytics/",
    "ml-models/",
    "temp/"
  ])
  
  bucket = aws_s3_bucket.data_lake.id
  key    = each.value
  source = "/dev/null"
}

# Data Lake Lifecycle Policies
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
    
    transition {
      days          = 365
      storage_class = "DEEP_ARCHIVE"
    }
    
    expiration {
      days = var.data_retention_days
    }
  }
}

# Data Lake Access Logs
resource "aws_s3_bucket" "data_lake_logs" {
  bucket = "${var.project_name}-data-lake-logs-${var.environment}"
}

resource "aws_s3_bucket_logging" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  
  target_bucket = aws_s3_bucket.data_lake_logs.id
  target_prefix = "access-logs/"
}
```

#### GCP Data Lake
```hcl
# data-lake-gcp.tf
resource "google_storage_bucket" "data_lake" {
  name          = "${var.project_name}-data-lake-${var.environment}"
  location      = var.gcp_region
  force_destroy = false
  
  versioning {
    enabled = true
  }
  
  lifecycle_rule {
    condition {
      age = 30
    }
    action {
      type          = "SetStorageClass"
      storage_class = "NEARLINE"
    }
  }
  
  lifecycle_rule {
    condition {
      age = 90
    }
    action {
      type          = "SetStorageClass"
      storage_class = "COLDLINE"
    }
  }
  
  lifecycle_rule {
    condition {
      age = 365
    }
    action {
      type          = "SetStorageClass"
      storage_class = "ARCHIVE"
    }
  }
  
  lifecycle_rule {
    condition {
      age = var.data_retention_days
    }
    action {
      type = "Delete"
    }
  }
}

# Data Lake Folder Structure
resource "google_storage_bucket_object" "data_lake_folders" {
  for_each = toset([
    "bronze/",
    "silver/",
    "gold/",
    "raw/",
    "processed/",
    "analytics/",
    "ml-models/",
    "temp/"
  ])
  
  name    = each.value
  bucket  = google_storage_bucket.data_lake.name
  content = ""
}
```

#### Azure Data Lake
```hcl
# data-lake-azure.tf
resource "azurerm_storage_account" "data_lake" {
  name                     = "${var.project_name}datalake${var.environment}"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  account_kind             = "StorageV2"
  is_hns_enabled           = true
  
  tags = {
    Environment = var.environment
    Purpose     = "data-storage"
  }
}

# Data Lake Containers
resource "azurerm_storage_container" "data_lake_containers" {
  for_each = toset([
    "bronze",
    "silver",
    "gold",
    "raw",
    "processed",
    "analytics",
    "ml-models",
    "temp"
  ])
  
  name                  = each.value
  storage_account_name  = azurerm_storage_account.data_lake.name
  container_access_type = "private"
}

# Data Lake Lifecycle Management
resource "azurerm_storage_management_policy" "data_lake" {
  storage_account_id = azurerm_storage_account.data_lake.id
  
  rule {
    name    = "data_lifecycle"
    enabled = true
    
    filters {
      prefix_match = ["bronze/", "silver/", "gold/"]
      blob_types   = ["blockBlob"]
    }
    
    actions {
      base_blob {
        tier_to_cool_after_days_since_modification_greater_than    = 30
        tier_to_archive_after_days_since_modification_greater_than = 90
        delete_after_days_since_modification_greater_than          = var.data_retention_days
      }
    }
  }
}
```

### 2. **Data Processing Infrastructure**

#### Apache Spark on AWS EMR
```hcl
# spark-emr-aws.tf
resource "aws_emr_cluster" "spark_cluster" {
  name          = "${var.project_name}-spark-cluster-${var.environment}"
  release_label = "emr-6.15.0"
  applications  = ["Spark", "Hadoop", "Hive"]
  
  ec2_attributes {
    subnet_id                         = aws_subnet.private.id
    emr_managed_master_security_group = aws_security_group.emr_master.id
    emr_managed_slave_security_group  = aws_security_group.emr_slave.id
    instance_profile                  = aws_iam_instance_profile.emr_profile.arn
  }
  
  master_instance_group {
    instance_type = var.spark_master_instance_type
    instance_count = 1
  }
  
  core_instance_group {
    instance_type  = var.spark_core_instance_type
    instance_count = var.spark_core_instance_count
    
    ebs_config {
      size                 = 100
      type                 = "gp3"
      volumes_per_instance = 1
    }
  }
  
  configurations_json = jsonencode([
    {
      "Classification": "spark-defaults",
      "Properties": {
        "spark.sql.adaptive.enabled": "true",
        "spark.sql.adaptive.coalescePartitions.enabled": "true",
        "spark.serializer": "org.apache.spark.serializer.KryoSerializer",
        "spark.sql.execution.arrow.pyspark.enabled": "true"
      }
    }
  ])
  
  log_uri = "s3://${aws_s3_bucket.data_lake_logs.bucket}/emr-logs/"
  
  service_role = aws_iam_role.emr_service_role.arn
  
  tags = {
    Name        = "Spark Cluster"
    Environment = var.environment
    Purpose     = "data-processing"
  }
}

# EMR Security Groups
resource "aws_security_group" "emr_master" {
  name_prefix = "${var.project_name}-emr-master-${var.environment}"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "EMR Master Security Group"
  }
}

resource "aws_security_group" "emr_slave" {
  name_prefix = "${var.project_name}-emr-slave-${var.environment}"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port = 0
    to_port   = 65535
    protocol  = "tcp"
    security_groups = [aws_security_group.emr_master.id]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "EMR Slave Security Group"
  }
}
```

#### Apache Spark on GCP Dataproc
```hcl
# spark-dataproc-gcp.tf
resource "google_dataproc_cluster" "spark_cluster" {
  name   = "${var.project_name}-spark-cluster-${var.environment}"
  region = var.gcp_region
  
  cluster_config {
    staging_bucket = google_storage_bucket.dataproc_staging.name
    
    master_config {
      num_instances = 1
      machine_type  = var.spark_master_machine_type
      disk_config {
        boot_disk_type    = "pd-ssd"
        boot_disk_size_gb = 100
      }
    }
    
    worker_config {
      num_instances = var.spark_worker_instance_count
      machine_type  = var.spark_worker_machine_type
      disk_config {
        boot_disk_type    = "pd-standard"
        boot_disk_size_gb = 100
        num_local_ssds    = 1
      }
    }
    
    software_config {
      image_version = "2.0-debian10"
      override_properties = {
        "spark:spark.sql.adaptive.enabled" = "true"
        "spark:spark.sql.adaptive.coalescePartitions.enabled" = "true"
        "spark:spark.serializer" = "org.apache.spark.serializer.KryoSerializer"
      }
    }
    
    gce_cluster_config {
      network    = google_compute_network.main.name
      subnetwork = google_compute_subnetwork.private.name
      
      service_account = google_service_account.dataproc.email
      service_account_scopes = [
        "https://www.googleapis.com/auth/cloud-platform"
      ]
    }
  }
  
  labels = {
    environment = var.environment
    purpose     = "data-processing"
  }
}

# Dataproc Staging Bucket
resource "google_storage_bucket" "dataproc_staging" {
  name          = "${var.project_name}-dataproc-staging-${var.environment}"
  location      = var.gcp_region
  force_destroy = true
}
```

#### Apache Spark on Azure Synapse
```hcl
# spark-synapse-azure.tf
resource "azurerm_synapse_workspace" "main" {
  name                                 = "${var.project_name}-synapse-${var.environment}"
  resource_group_name                  = azurerm_resource_group.main.name
  location                            = azurerm_resource_group.main.location
  storage_data_lake_gen2_filesystem_id = azurerm_storage_data_lake_gen2_filesystem.data_lake.id
  sql_administrator_login              = "sqladminuser"
  sql_administrator_login_password     = var.sql_admin_password
  
  tags = {
    Environment = var.environment
    Purpose     = "data-processing"
  }
}

resource "azurerm_synapse_spark_pool" "spark_pool" {
  name                 = "${var.project_name}-spark-pool-${var.environment}"
  synapse_workspace_id = azurerm_synapse_workspace.main.id
  node_size_family     = "MemoryOptimized"
  node_size            = "Small"
  node_count           = var.spark_node_count
  
  auto_scale {
    max_node_count = var.spark_max_node_count
    min_node_count = var.spark_min_node_count
  }
  
  auto_pause {
    delay_in_minutes = 15
  }
  
  spark_version = "3.3"
  
  tags = {
    Environment = var.environment
    Purpose     = "data-processing"
  }
}
```

### 3. **Data Streaming Infrastructure**

#### Apache Kafka on AWS MSK
```hcl
# kafka-msk-aws.tf
resource "aws_msk_cluster" "kafka_cluster" {
  cluster_name           = "${var.project_name}-kafka-${var.environment}"
  kafka_version          = "2.8.1"
  number_of_broker_nodes = var.kafka_broker_count
  
  broker_node_group_info {
    instance_type   = var.kafka_instance_type
    ebs_volume_size = 100
    client_subnets  = [aws_subnet.private.id, aws_subnet.private_2.id]
    security_groups = [aws_security_group.kafka.id]
  }
  
  configuration_info {
    arn      = aws_msk_configuration.kafka_config.arn
    revision = aws_msk_configuration.kafka_config.latest_revision
  }
  
  encryption_info {
    encryption_at_rest_kms_key_id = aws_kms_key.kafka.arn
    encryption_in_transit {
      client_broker = "TLS"
      in_cluster    = true
    }
  }
  
  logging_info {
    broker_logs {
      cloudwatch_logs {
        enabled   = true
        log_group = aws_cloudwatch_log_group.kafka.name
      }
      firehose {
        enabled = false
      }
      s3 {
        enabled = false
      }
    }
  }
  
  tags = {
    Name        = "Kafka Cluster"
    Environment = var.environment
    Purpose     = "data-streaming"
  }
}

# Kafka Configuration
resource "aws_msk_configuration" "kafka_config" {
  kafka_versions = ["2.8.1"]
  name           = "${var.project_name}-kafka-config-${var.environment}"
  
  server_properties = <<PROPERTIES
auto.create.topics.enable=true
default.replication.factor=3
min.insync.replicas=2
num.partitions=3
log.retention.hours=168
log.segment.bytes=1073741824
log.cleanup.policy=delete
compression.type=snappy
PROPERTIES
}

# Kafka Security Group
resource "aws_security_group" "kafka" {
  name_prefix = "${var.project_name}-kafka-${var.environment}"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 9092
    to_port     = 9092
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
  }
  
  ingress {
    from_port   = 9094
    to_port     = 9094
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "Kafka Security Group"
  }
}
```

#### Apache Kafka on GCP Pub/Sub
```hcl
# kafka-pubsub-gcp.tf
resource "google_pubsub_topic" "data_streams" {
  for_each = toset([
    "raw-data",
    "processed-data",
    "analytics-events",
    "ml-features"
  ])
  
  name = "${var.project_name}-${each.value}-${var.environment}"
  
  message_retention_duration = "604800s" # 7 days
  
  labels = {
    environment = var.environment
    purpose     = "data-streaming"
  }
}

resource "google_pubsub_subscription" "data_streams" {
  for_each = google_pubsub_topic.data_streams
  
  name  = "${each.value.name}-subscription"
  topic = each.value.name
  
  ack_deadline_seconds = 20
  
  retry_policy {
    minimum_backoff = "10s"
    maximum_backoff = "600s"
  }
  
  dead_letter_policy {
    dead_letter_topic     = google_pubsub_topic.dead_letter.id
    max_delivery_attempts = 5
  }
}

resource "google_pubsub_topic" "dead_letter" {
  name = "${var.project_name}-dead-letter-${var.environment}"
  
  labels = {
    environment = var.environment
    purpose     = "error-handling"
  }
}
```

### 4. **Data Orchestration Infrastructure**

#### Apache Airflow on AWS
```hcl
# airflow-aws.tf
resource "aws_mwaa_environment" "airflow" {
  name         = "${var.project_name}-airflow-${var.environment}"
  airflow_version = "2.6.3"
  
  environment_class = var.airflow_environment_class
  
  execution_role_arn = aws_iam_role.airflow_execution_role.arn
  
  network_configuration {
    security_group_ids = [aws_security_group.airflow.id]
    subnet_ids         = [aws_subnet.private.id, aws_subnet.private_2.id]
  }
  
  source_bucket_arn = aws_s3_bucket.airflow_dags.arn
  
  logging_configuration {
    dag_processing_logs {
      enabled   = true
      log_level = "INFO"
    }
    
    scheduler_logs {
      enabled   = true
      log_level = "INFO"
    }
    
    task_logs {
      enabled   = true
      log_level = "INFO"
    }
    
    webserver_logs {
      enabled   = true
      log_level = "INFO"
    }
    
    worker_logs {
      enabled   = true
      log_level = "INFO"
    }
  }
  
  webserver_access_mode = "PRIVATE_ONLY"
  
  tags = {
    Name        = "Airflow Environment"
    Environment = var.environment
    Purpose     = "data-orchestration"
  }
}

# Airflow DAGs Bucket
resource "aws_s3_bucket" "airflow_dags" {
  bucket = "${var.project_name}-airflow-dags-${var.environment}"
  
  tags = {
    Name        = "Airflow DAGs"
    Environment = var.environment
    Purpose     = "data-orchestration"
  }
}

# Airflow Security Group
resource "aws_security_group" "airflow" {
  name_prefix = "${var.project_name}-airflow-${var.environment}"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "Airflow Security Group"
  }
}
```

## 🔧 Terraform Modules

### 1. **Data Lake Module**
```hcl
# modules/data-lake/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

resource "aws_s3_bucket" "data_lake" {
  bucket = "${var.project_name}-data-lake-${var.environment}"
  
  tags = merge(var.tags, {
    Name    = "Data Lake"
    Purpose = "data-storage"
  })
}

resource "aws_s3_bucket_versioning" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_encryption" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
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
      days = var.data_retention_days
    }
  }
}

# modules/data-lake/variables.tf
variable "project_name" {
  description = "Name of the project"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "data_retention_days" {
  description = "Data retention period in days"
  type        = number
  default     = 2555
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}

# modules/data-lake/outputs.tf
output "bucket_name" {
  description = "Name of the data lake S3 bucket"
  value       = aws_s3_bucket.data_lake.bucket
}

output "bucket_arn" {
  description = "ARN of the data lake S3 bucket"
  value       = aws_s3_bucket.data_lake.arn
}

output "bucket_domain_name" {
  description = "Domain name of the data lake S3 bucket"
  value       = aws_s3_bucket.data_lake.bucket_domain_name
}
```

### 2. **Spark Cluster Module**
```hcl
# modules/spark-cluster/main.tf
resource "aws_emr_cluster" "spark_cluster" {
  name          = "${var.project_name}-spark-cluster-${var.environment}"
  release_label = var.emr_release_label
  applications  = ["Spark", "Hadoop", "Hive"]
  
  ec2_attributes {
    subnet_id                         = var.subnet_id
    emr_managed_master_security_group = aws_security_group.emr_master.id
    emr_managed_slave_security_group  = aws_security_group.emr_slave.id
    instance_profile                  = aws_iam_instance_profile.emr_profile.arn
  }
  
  master_instance_group {
    instance_type  = var.master_instance_type
    instance_count = 1
  }
  
  core_instance_group {
    instance_type  = var.core_instance_type
    instance_count = var.core_instance_count
    
    ebs_config {
      size                 = var.ebs_volume_size
      type                 = var.ebs_volume_type
      volumes_per_instance = 1
    }
  }
  
  configurations_json = jsonencode(var.spark_configurations)
  
  log_uri = var.log_uri
  
  service_role = aws_iam_role.emr_service_role.arn
  
  tags = merge(var.tags, {
    Name    = "Spark Cluster"
    Purpose = "data-processing"
  })
}

# modules/spark-cluster/variables.tf
variable "project_name" {
  description = "Name of the project"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "subnet_id" {
  description = "Subnet ID for EMR cluster"
  type        = string
}

variable "emr_release_label" {
  description = "EMR release label"
  type        = string
  default     = "emr-6.15.0"
}

variable "master_instance_type" {
  description = "Instance type for master node"
  type        = string
  default     = "m5.xlarge"
}

variable "core_instance_type" {
  description = "Instance type for core nodes"
  type        = string
  default     = "m5.large"
}

variable "core_instance_count" {
  description = "Number of core instances"
  type        = number
  default     = 2
}

variable "ebs_volume_size" {
  description = "EBS volume size in GB"
  type        = number
  default     = 100
}

variable "ebs_volume_type" {
  description = "EBS volume type"
  type        = string
  default     = "gp3"
}

variable "spark_configurations" {
  description = "Spark configurations"
  type        = list(map(string))
  default = [
    {
      "Classification": "spark-defaults",
      "Properties": {
        "spark.sql.adaptive.enabled": "true",
        "spark.sql.adaptive.coalescePartitions.enabled": "true",
        "spark.serializer": "org.apache.spark.serializer.KryoSerializer"
      }
    }
  ]
}

variable "log_uri" {
  description = "S3 URI for EMR logs"
  type        = string
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

### 3. **Module Usage**
```hcl
# main.tf
module "data_lake" {
  source = "./modules/data-lake"
  
  project_name        = var.project_name
  environment         = var.environment
  data_retention_days = var.data_retention_days
  
  tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
  }
}

module "spark_cluster" {
  source = "./modules/spark-cluster"
  
  project_name        = var.project_name
  environment         = var.environment
  subnet_id           = aws_subnet.private.id
  log_uri             = "s3://${module.data_lake.bucket_name}/emr-logs/"
  core_instance_count = var.spark_core_instance_count
  
  tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
  }
}
```

## 🚀 Terraform Best Practices

### 1. **State Management**
```hcl
# terraform.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "data-engineering/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

### 2. **Environment Management**
```hcl
# environments/dev/terraform.tfvars
project_name = "data-engineering"
environment  = "dev"
aws_region   = "us-east-1"

spark_core_instance_count = 2
spark_master_instance_type = "m5.large"
spark_core_instance_type = "m5.large"

data_retention_days = 365

allowed_cidr_blocks = [
  "10.0.0.0/8",
  "172.16.0.0/12"
]

# environments/prod/terraform.tfvars
project_name = "data-engineering"
environment  = "prod"
aws_region   = "us-east-1"

spark_core_instance_count = 5
spark_master_instance_type = "m5.xlarge"
spark_core_instance_type = "m5.xlarge"

data_retention_days = 2555

allowed_cidr_blocks = [
  "10.0.0.0/8"
]
```

### 3. **Security Best Practices**
```hcl
# security.tf
# KMS Key for encryption
resource "aws_kms_key" "data_encryption" {
  description             = "KMS key for data encryption"
  deletion_window_in_days = 7
  
  tags = {
    Name        = "Data Encryption Key"
    Environment = var.environment
    Purpose     = "encryption"
  }
}

resource "aws_kms_alias" "data_encryption" {
  name          = "alias/data-encryption-${var.environment}"
  target_key_id = aws_kms_key.data_encryption.key_id
}

# IAM Roles with least privilege
resource "aws_iam_role" "data_processing_role" {
  name = "${var.project_name}-data-processing-${var.environment}"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
  
  tags = {
    Name        = "Data Processing Role"
    Environment = var.environment
  }
}

resource "aws_iam_role_policy" "data_processing_policy" {
  name = "${var.project_name}-data-processing-${var.environment}"
  role = aws_iam_role.data_processing_role.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]
        Resource = [
          "${aws_s3_bucket.data_lake.arn}/*"
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "s3:ListBucket"
        ]
        Resource = [
          aws_s3_bucket.data_lake.arn
        ]
      }
    ]
  })
}
```

### 4. **Monitoring and Logging**
```hcl
# monitoring.tf
# CloudWatch Log Groups
resource "aws_cloudwatch_log_group" "data_processing" {
  name              = "/aws/data-processing/${var.environment}"
  retention_in_days = 30
  
  tags = {
    Name        = "Data Processing Logs"
    Environment = var.environment
  }
}

# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "high_cpu_utilization" {
  alarm_name          = "${var.project_name}-high-cpu-${var.environment}"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "This metric monitors ec2 cpu utilization"
  
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.data_processing.name
  }
  
  tags = {
    Name        = "High CPU Utilization"
    Environment = var.environment
  }
}

# SNS Topic for alerts
resource "aws_sns_topic" "alerts" {
  name = "${var.project_name}-alerts-${var.environment}"
  
  tags = {
    Name        = "Alerts Topic"
    Environment = var.environment
  }
}

resource "aws_sns_topic_subscription" "alerts_email" {
  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "email"
  endpoint  = var.alert_email
}
```

### 5. **Cost Optimization**
```hcl
# cost-optimization.tf
# S3 Intelligent Tiering
resource "aws_s3_bucket_intelligent_tiering_configuration" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  name   = "EntireBucket"
  
  status = "Enabled"
  
  tiering {
    access_tier = "ARCHIVE_ACCESS"
    days        = 90
  }
  
  tiering {
    access_tier = "DEEP_ARCHIVE_ACCESS"
    days        = 180
  }
}

# Spot Instances for EMR
resource "aws_emr_cluster" "spark_cluster_spot" {
  name          = "${var.project_name}-spark-cluster-spot-${var.environment}"
  release_label = "emr-6.15.0"
  applications  = ["Spark", "Hadoop", "Hive"]
  
  ec2_attributes {
    subnet_id                         = aws_subnet.private.id
    emr_managed_master_security_group = aws_security_group.emr_master.id
    emr_managed_slave_security_group  = aws_security_group.emr_slave.id
    instance_profile                  = aws_iam_instance_profile.emr_profile.arn
  }
  
  master_instance_group {
    instance_type  = var.spark_master_instance_type
    instance_count = 1
  }
  
  core_instance_group {
    instance_type  = var.spark_core_instance_type
    instance_count = var.spark_core_instance_count
    
    ebs_config {
      size                 = 100
      type                 = "gp3"
      volumes_per_instance = 1
    }
  }
  
  # Use spot instances for cost optimization
  instance_fleets {
    instance_fleet_type = "CORE"
    target_on_demand_capacity = 0
    target_spot_capacity = var.spark_core_instance_count
    
    instance_type_configs {
      instance_type     = var.spark_core_instance_type
      weighted_capacity = 1
    }
  }
  
  service_role = aws_iam_role.emr_service_role.arn
  
  tags = {
    Name        = "Spark Cluster (Spot)"
    Environment = var.environment
    Purpose     = "data-processing"
  }
}
```

## 🔗 Related Concepts

- [Kubernetes Infrastructure](../k8s/README.md)
- [Docker Infrastructure](../docker/README.md)
- [Data Lakehouse Architecture](../../concepts/datalakehouse/README.md)
- [Modern Data Architecture](../../architecture-designs/modern-data-architecture.md)

---

*Terraform enables data engineering teams to manage infrastructure as code, ensuring consistency, reproducibility, and scalability across environments. Success requires proper state management, security practices, and cost optimization strategies.*
