# Schema Change Data in Data Engineering

This document covers comprehensive strategies for handling schema changes in data engineering systems, including schema evolution, versioning, migration patterns, and compatibility management.

## 🎯 Overview

Schema changes are inevitable in data engineering systems as business requirements evolve, new features are added, and data sources change. Proper handling of schema changes is crucial for maintaining data quality, system reliability, and downstream compatibility.

### Key Concepts

1. **Schema Evolution**: How schemas change over time
2. **Schema Versioning**: Managing multiple schema versions
3. **Schema Migration**: Moving data between schema versions
4. **Schema Compatibility**: Ensuring backward and forward compatibility
5. **Schema Registry**: Centralized schema management

## 🔄 Schema Evolution Patterns

### 1. **Backward Compatible Changes**

#### Adding New Fields
```python
# Schema Version 1.0
user_schema_v1 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},
        {"name": "name", "type": "string"},
        {"name": "email", "type": "string"}
    ]
}

# Schema Version 1.1 - Adding new field (backward compatible)
user_schema_v1_1 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},
        {"name": "name", "type": "string"},
        {"name": "email", "type": "string"},
        {"name": "phone", "type": ["null", "string"], "default": None}  # Optional field
    ]
}

class SchemaEvolutionHandler:
    def __init__(self):
        self.schema_registry = SchemaRegistry()
        self.compatibility_checker = CompatibilityChecker()
    
    def add_field(self, schema, field_name, field_type, default_value=None):
        """Add new field to schema (backward compatible)"""
        if self.compatibility_checker.is_backward_compatible(schema, field_name, field_type):
            new_schema = schema.copy()
            new_schema["fields"].append({
                "name": field_name,
                "type": field_type,
                "default": default_value
            })
            return new_schema
        else:
            raise ValueError("Field addition is not backward compatible")
    
    def process_data_with_schema_evolution(self, data, old_schema, new_schema):
        """Process data with schema evolution"""
        if self.compatibility_checker.is_backward_compatible(old_schema, new_schema):
            # Data can be read with new schema
            return self.read_with_new_schema(data, new_schema)
        else:
            # Need migration
            return self.migrate_data(data, old_schema, new_schema)
```

#### Making Fields Optional
```python
# Schema Version 1.0
user_schema_v1 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},
        {"name": "name", "type": "string"},
        {"name": "email", "type": "string"},
        {"name": "phone", "type": "string"}  # Required field
    ]
}

# Schema Version 1.1 - Making field optional (backward compatible)
user_schema_v1_1 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},
        {"name": "name", "type": "string"},
        {"name": "email", "type": "string"},
        {"name": "phone", "type": ["null", "string"], "default": None}  # Now optional
    ]
}

class OptionalFieldHandler:
    def make_field_optional(self, schema, field_name, default_value=None):
        """Make a field optional (backward compatible)"""
        new_schema = schema.copy()
        for field in new_schema["fields"]:
            if field["name"] == field_name:
                # Change from required to optional
                if isinstance(field["type"], str):
                    field["type"] = ["null", field["type"]]
                field["default"] = default_value
                break
        return new_schema
    
    def handle_optional_field(self, data, field_name, default_value=None):
        """Handle optional field in data processing"""
        if field_name not in data or data[field_name] is None:
            return default_value
        return data[field_name]
```

### 2. **Breaking Changes**

#### Removing Fields
```python
# Schema Version 1.0
user_schema_v1 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},
        {"name": "name", "type": "string"},
        {"name": "email", "type": "string"},
        {"name": "phone", "type": "string"},
        {"name": "address", "type": "string"}
    ]
}

# Schema Version 2.0 - Removing field (breaking change)
user_schema_v2 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},
        {"name": "name", "type": "string"},
        {"name": "email", "type": "string"}
        # phone and address removed
    ]
}

class BreakingChangeHandler:
    def __init__(self):
        self.migration_strategies = {
            "field_removal": self.handle_field_removal,
            "type_change": self.handle_type_change,
            "field_rename": self.handle_field_rename
        }
    
    def handle_field_removal(self, data, removed_fields):
        """Handle removal of fields"""
        # Archive removed fields before removal
        archived_data = {}
        for field in removed_fields:
            if field in data:
                archived_data[f"{field}_archived"] = data[field]
                del data[field]
        
        return data, archived_data
    
    def migrate_data_for_breaking_change(self, data, old_schema, new_schema):
        """Migrate data for breaking changes"""
        migration_plan = self.create_migration_plan(old_schema, new_schema)
        
        migrated_data = data.copy()
        for step in migration_plan:
            migrated_data = step.apply(migrated_data)
        
        return migrated_data
    
    def create_migration_plan(self, old_schema, new_schema):
        """Create migration plan for breaking changes"""
        plan = []
        
        # Identify removed fields
        old_fields = {field["name"] for field in old_schema["fields"]}
        new_fields = {field["name"] for field in new_schema["fields"]}
        removed_fields = old_fields - new_fields
        
        if removed_fields:
            plan.append(FieldRemovalStep(removed_fields))
        
        # Identify type changes
        for old_field in old_schema["fields"]:
            for new_field in new_schema["fields"]:
                if old_field["name"] == new_field["name"]:
                    if old_field["type"] != new_field["type"]:
                        plan.append(TypeChangeStep(old_field["name"], old_field["type"], new_field["type"]))
        
        return plan
```

#### Changing Field Types
```python
# Schema Version 1.0
user_schema_v1 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "string"},  # String ID
        {"name": "age", "type": "int"},
        {"name": "created_at", "type": "string"}  # String timestamp
    ]
}

# Schema Version 2.0 - Type changes (breaking change)
user_schema_v2 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},  # Long ID
        {"name": "age", "type": "double"},  # Double age
        {"name": "created_at", "type": "long"}  # Long timestamp
    ]
}

class TypeChangeHandler:
    def handle_type_change(self, data, field_name, old_type, new_type):
        """Handle type changes with conversion"""
        if field_name in data:
            value = data[field_name]
            
            # Convert based on type change
            if old_type == "string" and new_type == "long":
                data[field_name] = int(value) if value else 0
            elif old_type == "int" and new_type == "double":
                data[field_name] = float(value) if value else 0.0
            elif old_type == "string" and new_type == "long" and "timestamp" in field_name:
                # Convert timestamp string to long
                data[field_name] = self.convert_timestamp_to_long(value)
        
        return data
    
    def convert_timestamp_to_long(self, timestamp_str):
        """Convert timestamp string to long"""
        from datetime import datetime
        try:
            dt = datetime.fromisoformat(timestamp_str.replace('Z', '+00:00'))
            return int(dt.timestamp() * 1000)  # Convert to milliseconds
        except:
            return 0
```

### 3. **Field Renaming**

```python
# Schema Version 1.0
user_schema_v1 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},
        {"name": "full_name", "type": "string"},
        {"name": "email_address", "type": "string"}
    ]
}

# Schema Version 2.0 - Field renaming (breaking change)
user_schema_v2 = {
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "long"},
        {"name": "name", "type": "string"},  # renamed from full_name
        {"name": "email", "type": "string"}  # renamed from email_address
    ]
}

class FieldRenameHandler:
    def handle_field_rename(self, data, field_mapping):
        """Handle field renaming"""
        renamed_data = data.copy()
        
        for old_name, new_name in field_mapping.items():
            if old_name in renamed_data:
                renamed_data[new_name] = renamed_data[old_name]
                del renamed_data[old_name]
        
        return renamed_data
    
    def create_field_mapping(self, old_schema, new_schema):
        """Create field mapping for renaming"""
        mapping = {}
        
        # Use field aliases or metadata to determine mapping
        for old_field in old_schema["fields"]:
            for new_field in new_schema["fields"]:
                if self.fields_are_equivalent(old_field, new_field):
                    mapping[old_field["name"]] = new_field["name"]
                    break
        
        return mapping
    
    def fields_are_equivalent(self, old_field, new_field):
        """Check if fields are equivalent (same type, different name)"""
        return (old_field["type"] == new_field["type"] and 
                old_field["name"] != new_field["name"])
```

## 📋 Schema Versioning Strategies

### 1. **Semantic Versioning**

```python
class SchemaVersion:
    def __init__(self, major: int, minor: int, patch: int):
        self.major = major
        self.minor = minor
        self.patch = patch
    
    def __str__(self):
        return f"{self.major}.{self.minor}.{self.patch}"
    
    def is_backward_compatible(self, other_version):
        """Check if this version is backward compatible with other version"""
        if self.major != other_version.major:
            return False  # Major version changes are breaking
        return self.minor >= other_version.minor
    
    def is_forward_compatible(self, other_version):
        """Check if this version is forward compatible with other version"""
        if self.major != other_version.major:
            return False  # Major version changes are breaking
        return self.minor <= other_version.minor

class SchemaVersionManager:
    def __init__(self):
        self.schemas = {}
        self.version_history = []
    
    def register_schema(self, version: SchemaVersion, schema: dict):
        """Register a new schema version"""
        self.schemas[str(version)] = schema
        self.version_history.append(version)
        self.version_history.sort(key=lambda v: (v.major, v.minor, v.patch))
    
    def get_schema(self, version: str):
        """Get schema by version"""
        return self.schemas.get(version)
    
    def get_latest_version(self):
        """Get latest schema version"""
        return self.version_history[-1] if self.version_history else None
    
    def find_compatible_version(self, target_version: str):
        """Find compatible version for migration"""
        target = SchemaVersion.from_string(target_version)
        
        for version in reversed(self.version_history):
            if version.is_backward_compatible(target):
                return version
        
        return None
```

### 2. **Schema Registry Integration**

```python
class SchemaRegistry:
    def __init__(self, registry_url: str):
        self.registry_url = registry_url
        self.client = SchemaRegistryClient(registry_url)
    
    def register_schema(self, subject: str, schema: dict, version: str):
        """Register schema in registry"""
        schema_id = self.client.register_schema(subject, schema)
        
        # Store version information
        self.store_version_info(subject, version, schema_id)
        
        return schema_id
    
    def get_schema(self, subject: str, version: str = "latest"):
        """Get schema from registry"""
        if version == "latest":
            return self.client.get_latest_schema(subject)
        else:
            return self.client.get_schema_by_version(subject, version)
    
    def check_compatibility(self, subject: str, new_schema: dict):
        """Check compatibility of new schema"""
        return self.client.test_compatibility(subject, new_schema)
    
    def get_schema_evolution_history(self, subject: str):
        """Get schema evolution history"""
        return self.client.get_schema_versions(subject)

class SchemaRegistryClient:
    def __init__(self, registry_url: str):
        self.registry_url = registry_url
        self.session = requests.Session()
    
    def register_schema(self, subject: str, schema: dict):
        """Register schema and get schema ID"""
        url = f"{self.registry_url}/subjects/{subject}/versions"
        response = self.session.post(url, json={"schema": json.dumps(schema)})
        response.raise_for_status()
        return response.json()["id"]
    
    def get_latest_schema(self, subject: str):
        """Get latest schema for subject"""
        url = f"{self.registry_url}/subjects/{subject}/versions/latest"
        response = self.session.get(url)
        response.raise_for_status()
        return response.json()
    
    def test_compatibility(self, subject: str, new_schema: dict):
        """Test compatibility of new schema"""
        url = f"{self.registry_url}/compatibility/subjects/{subject}/versions/latest"
        response = self.session.post(url, json={"schema": json.dumps(new_schema)})
        response.raise_for_status()
        return response.json()["is_compatible"]
```

## 🔄 Schema Migration Patterns

### 1. **Blue-Green Migration**

```python
class BlueGreenMigration:
    def __init__(self):
        self.blue_environment = "blue"
        self.green_environment = "green"
        self.current_environment = self.blue_environment
    
    def migrate_schema(self, new_schema: dict):
        """Perform blue-green schema migration"""
        # 1. Deploy new schema to inactive environment
        inactive_env = self.get_inactive_environment()
        self.deploy_schema_to_environment(inactive_env, new_schema)
        
        # 2. Migrate data to new schema
        self.migrate_data_to_new_schema(inactive_env, new_schema)
        
        # 3. Validate new environment
        if self.validate_environment(inactive_env):
            # 4. Switch traffic to new environment
            self.switch_traffic(inactive_env)
            self.current_environment = inactive_env
        else:
            # 5. Rollback if validation fails
            self.rollback_migration(inactive_env)
    
    def get_inactive_environment(self):
        """Get inactive environment for migration"""
        return self.green_environment if self.current_environment == self.blue_environment else self.blue_environment
    
    def migrate_data_to_new_schema(self, environment: str, new_schema: dict):
        """Migrate data to new schema format"""
        # Read data from current environment
        current_data = self.read_data_from_environment(self.current_environment)
        
        # Transform data to new schema
        migrated_data = self.transform_data_to_schema(current_data, new_schema)
        
        # Write data to new environment
        self.write_data_to_environment(environment, migrated_data)
    
    def switch_traffic(self, new_environment: str):
        """Switch traffic to new environment"""
        # Update load balancer configuration
        self.update_load_balancer_config(new_environment)
        
        # Update application configuration
        self.update_application_config(new_environment)
        
        # Verify traffic is flowing to new environment
        self.verify_traffic_switch(new_environment)
```

### 2. **Canary Migration**

```python
class CanaryMigration:
    def __init__(self):
        self.migration_percentage = 0
        self.max_migration_percentage = 100
        self.migration_step = 10
    
    def migrate_schema_canary(self, new_schema: dict):
        """Perform canary schema migration"""
        # Start with small percentage
        self.migration_percentage = self.migration_step
        
        while self.migration_percentage <= self.max_migration_percentage:
            # Migrate percentage of data
            self.migrate_percentage_of_data(new_schema, self.migration_percentage)
            
            # Monitor for issues
            if self.monitor_migration_health():
                # Increase migration percentage
                self.migration_percentage += self.migration_step
            else:
                # Rollback migration
                self.rollback_migration()
                break
        
        # Complete migration
        if self.migration_percentage >= self.max_migration_percentage:
            self.complete_migration()
    
    def migrate_percentage_of_data(self, new_schema: dict, percentage: int):
        """Migrate percentage of data to new schema"""
        # Select percentage of data to migrate
        data_to_migrate = self.select_data_for_migration(percentage)
        
        # Transform data to new schema
        migrated_data = self.transform_data_to_schema(data_to_migrate, new_schema)
        
        # Write migrated data
        self.write_migrated_data(migrated_data)
    
    def select_data_for_migration(self, percentage: int):
        """Select data for migration based on percentage"""
        # Use consistent hashing to select data
        total_data = self.get_total_data_count()
        data_to_migrate_count = int(total_data * percentage / 100)
        
        # Select data using consistent hashing
        return self.select_data_by_hash(data_to_migrate_count)
    
    def monitor_migration_health(self):
        """Monitor health of migration"""
        # Check error rates
        error_rate = self.get_error_rate()
        if error_rate > self.max_error_rate:
            return False
        
        # Check performance metrics
        performance_metrics = self.get_performance_metrics()
        if performance_metrics["latency"] > self.max_latency:
            return False
        
        # Check data quality
        data_quality = self.check_data_quality()
        if data_quality < self.min_data_quality:
            return False
        
        return True
```

### 3. **Rolling Migration**

```python
class RollingMigration:
    def __init__(self):
        self.migration_batches = []
        self.current_batch = 0
    
    def migrate_schema_rolling(self, new_schema: dict):
        """Perform rolling schema migration"""
        # Create migration batches
        self.create_migration_batches(new_schema)
        
        # Process each batch
        for batch in self.migration_batches:
            self.process_migration_batch(batch)
            
            # Wait for batch to complete
            self.wait_for_batch_completion(batch)
            
            # Validate batch
            if not self.validate_batch(batch):
                self.rollback_batch(batch)
                break
        
        # Complete migration
        self.complete_migration()
    
    def create_migration_batches(self, new_schema: dict):
        """Create migration batches"""
        # Divide data into batches
        total_data = self.get_total_data_count()
        batch_size = self.calculate_batch_size(total_data)
        
        for i in range(0, total_data, batch_size):
            batch = MigrationBatch(
                start_index=i,
                end_index=min(i + batch_size, total_data),
                new_schema=new_schema
            )
            self.migration_batches.append(batch)
    
    def process_migration_batch(self, batch: MigrationBatch):
        """Process migration batch"""
        # Read batch data
        batch_data = self.read_batch_data(batch)
        
        # Transform data to new schema
        migrated_data = self.transform_data_to_schema(batch_data, batch.new_schema)
        
        # Write migrated data
        self.write_migrated_batch_data(migrated_data, batch)
    
    def validate_batch(self, batch: MigrationBatch):
        """Validate migration batch"""
        # Check data integrity
        if not self.check_data_integrity(batch):
            return False
        
        # Check performance
        if not self.check_batch_performance(batch):
            return False
        
        # Check business logic
        if not self.check_business_logic(batch):
            return False
        
        return True

class MigrationBatch:
    def __init__(self, start_index: int, end_index: int, new_schema: dict):
        self.start_index = start_index
        self.end_index = end_index
        self.new_schema = new_schema
        self.status = "pending"
        self.start_time = None
        self.end_time = None
        self.error_count = 0
```

## 🔗 Schema Compatibility Management

### 1. **Backward Compatibility**

```python
class BackwardCompatibilityChecker:
    def __init__(self):
        self.compatibility_rules = {
            "add_field": self.check_add_field_compatibility,
            "remove_field": self.check_remove_field_compatibility,
            "change_type": self.check_type_change_compatibility,
            "rename_field": self.check_rename_field_compatibility
        }
    
    def check_compatibility(self, old_schema: dict, new_schema: dict):
        """Check backward compatibility between schemas"""
        compatibility_report = {
            "is_compatible": True,
            "issues": [],
            "warnings": []
        }
        
        # Check each field change
        for change in self.identify_changes(old_schema, new_schema):
            rule = self.compatibility_rules.get(change["type"])
            if rule:
                result = rule(change, old_schema, new_schema)
                if not result["is_compatible"]:
                    compatibility_report["is_compatible"] = False
                    compatibility_report["issues"].extend(result["issues"])
                compatibility_report["warnings"].extend(result["warnings"])
        
        return compatibility_report
    
    def check_add_field_compatibility(self, change, old_schema, new_schema):
        """Check compatibility for adding fields"""
        result = {"is_compatible": True, "issues": [], "warnings": []}
        
        new_field = change["new_field"]
        
        # Adding required field is breaking change
        if new_field.get("default") is None and "null" not in new_field["type"]:
            result["is_compatible"] = False
            result["issues"].append(f"Adding required field '{new_field['name']}' is breaking change")
        
        # Adding optional field is compatible
        if new_field.get("default") is not None or "null" in new_field["type"]:
            result["warnings"].append(f"Adding optional field '{new_field['name']}' is compatible")
        
        return result
    
    def check_remove_field_compatibility(self, change, old_schema, new_schema):
        """Check compatibility for removing fields"""
        result = {"is_compatible": False, "issues": [], "warnings": []}
        
        removed_field = change["removed_field"]
        result["issues"].append(f"Removing field '{removed_field['name']}' is breaking change")
        
        return result
    
    def check_type_change_compatibility(self, change, old_schema, new_schema):
        """Check compatibility for type changes"""
        result = {"is_compatible": True, "issues": [], "warnings": []}
        
        old_type = change["old_type"]
        new_type = change["new_type"]
        
        # Check if type change is compatible
        if not self.is_type_change_compatible(old_type, new_type):
            result["is_compatible"] = False
            result["issues"].append(f"Type change from '{old_type}' to '{new_type}' is breaking change")
        else:
            result["warnings"].append(f"Type change from '{old_type}' to '{new_type}' is compatible")
        
        return result
    
    def is_type_change_compatible(self, old_type, new_type):
        """Check if type change is compatible"""
        # Widening changes are compatible
        compatible_changes = {
            "int": ["long", "float", "double"],
            "long": ["float", "double"],
            "float": ["double"],
            "string": ["bytes"]
        }
        
        return new_type in compatible_changes.get(old_type, [])
```

### 2. **Forward Compatibility**

```python
class ForwardCompatibilityChecker:
    def __init__(self):
        self.forward_compatibility_rules = {
            "ignore_unknown_fields": True,
            "default_values": True,
            "type_promotion": True
        }
    
    def check_forward_compatibility(self, old_schema: dict, new_schema: dict):
        """Check forward compatibility between schemas"""
        compatibility_report = {
            "is_compatible": True,
            "issues": [],
            "warnings": []
        }
        
        # Check if old schema can read new data
        for field in new_schema["fields"]:
            if not self.can_read_field(field, old_schema):
                compatibility_report["is_compatible"] = False
                compatibility_report["issues"].append(f"Cannot read new field '{field['name']}' with old schema")
        
        return compatibility_report
    
    def can_read_field(self, field, old_schema):
        """Check if old schema can read new field"""
        # Find corresponding field in old schema
        old_field = self.find_field_in_schema(field["name"], old_schema)
        
        if not old_field:
            # New field - check if we can ignore it
            return self.forward_compatibility_rules["ignore_unknown_fields"]
        
        # Check type compatibility
        return self.is_type_forward_compatible(old_field["type"], field["type"])
    
    def is_type_forward_compatible(self, old_type, new_type):
        """Check if type change is forward compatible"""
        # Narrowing changes are forward compatible
        forward_compatible_changes = {
            "long": ["int"],
            "double": ["float", "long", "int"],
            "float": ["int", "long"],
            "bytes": ["string"]
        }
        
        return new_type in forward_compatible_changes.get(old_type, [])
```

## 🛠️ Schema Change Implementation

### 1. **Schema Change Handler**

```python
class SchemaChangeHandler:
    def __init__(self):
        self.schema_registry = SchemaRegistry()
        self.migration_engine = MigrationEngine()
        self.compatibility_checker = CompatibilityChecker()
        self.data_validator = DataValidator()
    
    def handle_schema_change(self, subject: str, new_schema: dict, change_type: str):
        """Handle schema change"""
        # Get current schema
        current_schema = self.schema_registry.get_latest_schema(subject)
        
        # Check compatibility
        compatibility_report = self.compatibility_checker.check_compatibility(
            current_schema, new_schema
        )
        
        if not compatibility_report["is_compatible"]:
            # Handle breaking changes
            return self.handle_breaking_change(subject, new_schema, compatibility_report)
        else:
            # Handle non-breaking changes
            return self.handle_non_breaking_change(subject, new_schema)
    
    def handle_breaking_change(self, subject: str, new_schema: dict, compatibility_report: dict):
        """Handle breaking schema changes"""
        # Create migration plan
        migration_plan = self.create_migration_plan(subject, new_schema)
        
        # Execute migration
        migration_result = self.execute_migration(migration_plan)
        
        # Update schema registry
        self.schema_registry.register_schema(subject, new_schema)
        
        return migration_result
    
    def handle_non_breaking_change(self, subject: str, new_schema: dict):
        """Handle non-breaking schema changes"""
        # Register new schema
        schema_id = self.schema_registry.register_schema(subject, new_schema)
        
        # Update data processing pipelines
        self.update_data_pipelines(subject, new_schema)
        
        return {"status": "success", "schema_id": schema_id}
    
    def create_migration_plan(self, subject: str, new_schema: dict):
        """Create migration plan for schema change"""
        current_schema = self.schema_registry.get_latest_schema(subject)
        
        # Identify changes
        changes = self.identify_schema_changes(current_schema, new_schema)
        
        # Create migration steps
        migration_steps = []
        for change in changes:
            step = self.create_migration_step(change)
            migration_steps.append(step)
        
        return MigrationPlan(subject, migration_steps)
    
    def execute_migration(self, migration_plan: MigrationPlan):
        """Execute migration plan"""
        results = []
        
        for step in migration_plan.steps:
            try:
                result = step.execute()
                results.append(result)
            except Exception as e:
                # Handle migration failure
                self.handle_migration_failure(step, e)
                raise
        
        return MigrationResult(migration_plan.subject, results)
```

### 2. **Data Validation and Quality**

```python
class SchemaDataValidator:
    def __init__(self):
        self.validators = {
            "required_fields": self.validate_required_fields,
            "type_validation": self.validate_field_types,
            "constraint_validation": self.validate_constraints,
            "format_validation": self.validate_formats
        }
    
    def validate_data_against_schema(self, data: dict, schema: dict):
        """Validate data against schema"""
        validation_result = {
            "is_valid": True,
            "errors": [],
            "warnings": []
        }
        
        # Validate each field
        for field in schema["fields"]:
            field_result = self.validate_field(data, field)
            if not field_result["is_valid"]:
                validation_result["is_valid"] = False
                validation_result["errors"].extend(field_result["errors"])
            validation_result["warnings"].extend(field_result["warnings"])
        
        return validation_result
    
    def validate_field(self, data: dict, field: dict):
        """Validate individual field"""
        field_name = field["name"]
        field_type = field["type"]
        
        result = {"is_valid": True, "errors": [], "warnings": []}
        
        # Check if field exists
        if field_name not in data:
            if field.get("default") is not None:
                # Use default value
                data[field_name] = field["default"]
            elif "null" not in field_type:
                result["is_valid"] = False
                result["errors"].append(f"Required field '{field_name}' is missing")
            return result
        
        # Validate field value
        value = data[field_name]
        if value is None:
            if "null" not in field_type:
                result["is_valid"] = False
                result["errors"].append(f"Field '{field_name}' cannot be null")
            return result
        
        # Validate type
        if not self.validate_field_type(value, field_type):
            result["is_valid"] = False
            result["errors"].append(f"Field '{field_name}' has invalid type")
        
        # Validate constraints
        if field.get("constraints"):
            constraint_result = self.validate_constraints(value, field["constraints"])
            if not constraint_result["is_valid"]:
                result["is_valid"] = False
                result["errors"].extend(constraint_result["errors"])
        
        return result
    
    def validate_field_type(self, value, field_type):
        """Validate field type"""
        if isinstance(field_type, list):
            # Union type
            return any(self.validate_field_type(value, t) for t in field_type)
        
        type_validators = {
            "string": lambda v: isinstance(v, str),
            "int": lambda v: isinstance(v, int),
            "long": lambda v: isinstance(v, int),
            "float": lambda v: isinstance(v, float),
            "double": lambda v: isinstance(v, float),
            "boolean": lambda v: isinstance(v, bool),
            "null": lambda v: v is None
        }
        
        validator = type_validators.get(field_type)
        return validator(value) if validator else False
```

## 🔗 Related Concepts

- [Data Architecture Patterns](../../architecture-designs/modern-data-architecture.md)
- [Data Quality and Governance](../security/README.md)
- [Schema Registry](../../tools/data-ingestion/schema-registry/README.md)
- [Data Migration Patterns](../../design-pattern/README.md)

---

*Proper schema change management is essential for maintaining data quality and system reliability. Focus on compatibility, migration strategies, and validation to ensure smooth schema evolution.*
