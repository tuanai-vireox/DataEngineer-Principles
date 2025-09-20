# Data Vault Methodology

This document provides comprehensive coverage of the Data Vault methodology, a modern data modeling approach designed for agile data warehousing, scalability, and flexibility in data engineering systems.

## 🎯 Overview

Data Vault is a hybrid data modeling methodology that combines the best of 3rd Normal Form (3NF) and Star Schema approaches. It's designed to handle the complexity of modern data environments while providing flexibility, scalability, and auditability.

### Key Principles

1. **Business Key Centric**: Focus on business keys and their relationships
2. **Audit Trail**: Complete historical tracking of all data changes
3. **Scalability**: Designed for high-volume, high-velocity data
4. **Flexibility**: Easy to add new data sources and business rules
5. **Agility**: Rapid development and deployment capabilities

### Core Components

- **Hubs**: Business keys and their metadata
- **Links**: Relationships between business keys
- **Satellites**: Descriptive attributes and context
- **Pit Tables**: Point-in-time snapshots for performance
- **Bridge Tables**: Many-to-many relationships

## 🏗️ Data Vault Architecture

### 1. **Hub Tables**

#### Purpose
Hubs store business keys and their metadata, providing a single source of truth for business entities.

```sql
-- Customer Hub
CREATE TABLE HUB_CUSTOMER (
    CUSTOMER_HK VARCHAR(64) PRIMARY KEY,  -- Hash key
    CUSTOMER_BK VARCHAR(50) NOT NULL,     -- Business key
    LOAD_DATE TIMESTAMP NOT NULL,          -- Load timestamp
    RECORD_SOURCE VARCHAR(100) NOT NULL,   -- Source system
    HASH_DIFF VARCHAR(64),                -- Hash of all attributes
    UNIQUE(CUSTOMER_BK, LOAD_DATE, RECORD_SOURCE)
);

-- Product Hub
CREATE TABLE HUB_PRODUCT (
    PRODUCT_HK VARCHAR(64) PRIMARY KEY,
    PRODUCT_BK VARCHAR(50) NOT NULL,
    LOAD_DATE TIMESTAMP NOT NULL,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    HASH_DIFF VARCHAR(64),
    UNIQUE(PRODUCT_BK, LOAD_DATE, RECORD_SOURCE)
);

-- Order Hub
CREATE TABLE HUB_ORDER (
    ORDER_HK VARCHAR(64) PRIMARY KEY,
    ORDER_BK VARCHAR(50) NOT NULL,
    LOAD_DATE TIMESTAMP NOT NULL,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    HASH_DIFF VARCHAR(64),
    UNIQUE(ORDER_BK, LOAD_DATE, RECORD_SOURCE)
);
```

#### Hub Implementation Pattern
```python
class HubTable:
    def __init__(self, table_name: str, business_key: str):
        self.table_name = table_name
        self.business_key = business_key
        self.hash_key = f"{table_name}_HK"
        self.load_date = "LOAD_DATE"
        self.record_source = "RECORD_SOURCE"
    
    def create_table_sql(self):
        """Generate CREATE TABLE SQL for Hub"""
        return f"""
        CREATE TABLE {self.table_name} (
            {self.hash_key} VARCHAR(64) PRIMARY KEY,
            {self.business_key} VARCHAR(50) NOT NULL,
            {self.load_date} TIMESTAMP NOT NULL,
            {self.record_source} VARCHAR(100) NOT NULL,
            HASH_DIFF VARCHAR(64),
            UNIQUE({self.business_key}, {self.load_date}, {self.record_source})
        );
        """
    
    def generate_hash_key(self, business_key: str, load_date: str, record_source: str):
        """Generate hash key for hub record"""
        import hashlib
        key_string = f"{business_key}|{load_date}|{record_source}"
        return hashlib.sha256(key_string.encode()).hexdigest()[:16]
    
    def insert_hub_record(self, business_key: str, record_source: str):
        """Insert new hub record"""
        load_date = datetime.now()
        hash_key = self.generate_hash_key(business_key, str(load_date), record_source)
        
        return {
            self.hash_key: hash_key,
            self.business_key: business_key,
            self.load_date: load_date,
            self.record_source: record_source,
            "HASH_DIFF": self.calculate_hash_diff(business_key, load_date, record_source)
        }
```

### 2. **Link Tables**

#### Purpose
Links represent relationships between business keys, storing many-to-many relationships and their metadata.

```sql
-- Customer-Order Link
CREATE TABLE LINK_CUSTOMER_ORDER (
    CUSTOMER_ORDER_HK VARCHAR(64) PRIMARY KEY,
    CUSTOMER_HK VARCHAR(64) NOT NULL,
    ORDER_HK VARCHAR(64) NOT NULL,
    LOAD_DATE TIMESTAMP NOT NULL,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    HASH_DIFF VARCHAR(64),
    FOREIGN KEY (CUSTOMER_HK) REFERENCES HUB_CUSTOMER(CUSTOMER_HK),
    FOREIGN KEY (ORDER_HK) REFERENCES HUB_ORDER(ORDER_HK),
    UNIQUE(CUSTOMER_HK, ORDER_HK, LOAD_DATE, RECORD_SOURCE)
);

-- Order-Product Link
CREATE TABLE LINK_ORDER_PRODUCT (
    ORDER_PRODUCT_HK VARCHAR(64) PRIMARY KEY,
    ORDER_HK VARCHAR(64) NOT NULL,
    PRODUCT_HK VARCHAR(64) NOT NULL,
    QUANTITY INT,
    UNIT_PRICE DECIMAL(10,2),
    LOAD_DATE TIMESTAMP NOT NULL,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    HASH_DIFF VARCHAR(64),
    FOREIGN KEY (ORDER_HK) REFERENCES HUB_ORDER(ORDER_HK),
    FOREIGN KEY (PRODUCT_HK) REFERENCES HUB_PRODUCT(PRODUCT_HK),
    UNIQUE(ORDER_HK, PRODUCT_HK, LOAD_DATE, RECORD_SOURCE)
);
```

#### Link Implementation Pattern
```python
class LinkTable:
    def __init__(self, table_name: str, parent_hubs: list):
        self.table_name = table_name
        self.parent_hubs = parent_hubs
        self.hash_key = f"{table_name}_HK"
        self.load_date = "LOAD_DATE"
        self.record_source = "RECORD_SOURCE"
    
    def create_table_sql(self):
        """Generate CREATE TABLE SQL for Link"""
        foreign_keys = []
        unique_constraints = []
        
        for hub in self.parent_hubs:
            foreign_keys.append(f"FOREIGN KEY ({hub}_HK) REFERENCES HUB_{hub}({hub}_HK)")
            unique_constraints.append(f"{hub}_HK")
        
        return f"""
        CREATE TABLE {self.table_name} (
            {self.hash_key} VARCHAR(64) PRIMARY KEY,
            {', '.join([f"{hub}_HK VARCHAR(64) NOT NULL" for hub in self.parent_hubs])},
            {self.load_date} TIMESTAMP NOT NULL,
            {self.record_source} VARCHAR(100) NOT NULL,
            HASH_DIFF VARCHAR(64),
            {', '.join(foreign_keys)},
            UNIQUE({', '.join(unique_constraints)}, {self.load_date}, {self.record_source})
        );
        """
    
    def generate_link_hash_key(self, parent_keys: dict, load_date: str, record_source: str):
        """Generate hash key for link record"""
        import hashlib
        key_parts = [str(parent_keys[hub]) for hub in self.parent_hubs]
        key_string = "|".join(key_parts + [load_date, record_source])
        return hashlib.sha256(key_string.encode()).hexdigest()[:16]
    
    def insert_link_record(self, parent_keys: dict, record_source: str, attributes: dict = None):
        """Insert new link record"""
        load_date = datetime.now()
        hash_key = self.generate_link_hash_key(parent_keys, str(load_date), record_source)
        
        record = {
            self.hash_key: hash_key,
            self.load_date: load_date,
            self.record_source: record_source,
            "HASH_DIFF": self.calculate_hash_diff(parent_keys, load_date, record_source)
        }
        
        # Add parent hub keys
        for hub in self.parent_hubs:
            record[f"{hub}_HK"] = parent_keys[hub]
        
        # Add additional attributes
        if attributes:
            record.update(attributes)
        
        return record
```

### 3. **Satellite Tables**

#### Purpose
Satellites store descriptive attributes and context for business keys, providing historical tracking of attribute changes.

```sql
-- Customer Satellite (Demographics)
CREATE TABLE SAT_CUSTOMER_DEMOGRAPHICS (
    CUSTOMER_HK VARCHAR(64) NOT NULL,
    LOAD_DATE TIMESTAMP NOT NULL,
    LOAD_END_DATE TIMESTAMP,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    HASH_DIFF VARCHAR(64),
    CUSTOMER_NAME VARCHAR(100),
    EMAIL VARCHAR(100),
    PHONE VARCHAR(20),
    DATE_OF_BIRTH DATE,
    GENDER VARCHAR(10),
    ADDRESS_LINE1 VARCHAR(200),
    ADDRESS_LINE2 VARCHAR(200),
    CITY VARCHAR(100),
    STATE VARCHAR(50),
    ZIP_CODE VARCHAR(20),
    COUNTRY VARCHAR(50),
    PRIMARY KEY (CUSTOMER_HK, LOAD_DATE, RECORD_SOURCE),
    FOREIGN KEY (CUSTOMER_HK) REFERENCES HUB_CUSTOMER(CUSTOMER_HK)
);

-- Customer Satellite (Preferences)
CREATE TABLE SAT_CUSTOMER_PREFERENCES (
    CUSTOMER_HK VARCHAR(64) NOT NULL,
    LOAD_DATE TIMESTAMP NOT NULL,
    LOAD_END_DATE TIMESTAMP,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    HASH_DIFF VARCHAR(64),
    PREFERRED_LANGUAGE VARCHAR(10),
    MARKETING_CONSENT BOOLEAN,
    NEWSLETTER_SUBSCRIPTION BOOLEAN,
    COMMUNICATION_PREFERENCE VARCHAR(20),
    PRIMARY KEY (CUSTOMER_HK, LOAD_DATE, RECORD_SOURCE),
    FOREIGN KEY (CUSTOMER_HK) REFERENCES HUB_CUSTOMER(CUSTOMER_HK)
);

-- Product Satellite (Details)
CREATE TABLE SAT_PRODUCT_DETAILS (
    PRODUCT_HK VARCHAR(64) NOT NULL,
    LOAD_DATE TIMESTAMP NOT NULL,
    LOAD_END_DATE TIMESTAMP,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    HASH_DIFF VARCHAR(64),
    PRODUCT_NAME VARCHAR(200),
    DESCRIPTION TEXT,
    CATEGORY VARCHAR(100),
    BRAND VARCHAR(100),
    SKU VARCHAR(50),
    PRICE DECIMAL(10,2),
    CURRENCY VARCHAR(3),
    WEIGHT DECIMAL(8,2),
    DIMENSIONS VARCHAR(50),
    PRIMARY KEY (PRODUCT_HK, LOAD_DATE, RECORD_SOURCE),
    FOREIGN KEY (PRODUCT_HK) REFERENCES HUB_PRODUCT(PRODUCT_HK)
);
```

#### Satellite Implementation Pattern
```python
class SatelliteTable:
    def __init__(self, table_name: str, parent_hub: str, attributes: list):
        self.table_name = table_name
        self.parent_hub = parent_hub
        self.attributes = attributes
        self.parent_key = f"{parent_hub}_HK"
        self.load_date = "LOAD_DATE"
        self.load_end_date = "LOAD_END_DATE"
        self.record_source = "RECORD_SOURCE"
        self.hash_diff = "HASH_DIFF"
    
    def create_table_sql(self):
        """Generate CREATE TABLE SQL for Satellite"""
        attribute_definitions = []
        for attr in self.attributes:
            attr_type = self.get_attribute_type(attr)
            attribute_definitions.append(f"{attr} {attr_type}")
        
        return f"""
        CREATE TABLE {self.table_name} (
            {self.parent_key} VARCHAR(64) NOT NULL,
            {self.load_date} TIMESTAMP NOT NULL,
            {self.load_end_date} TIMESTAMP,
            {self.record_source} VARCHAR(100) NOT NULL,
            {self.hash_diff} VARCHAR(64),
            {', '.join(attribute_definitions)},
            PRIMARY KEY ({self.parent_key}, {self.load_date}, {self.record_source}),
            FOREIGN KEY ({self.parent_key}) REFERENCES HUB_{self.parent_hub}({self.parent_key})
        );
        """
    
    def get_attribute_type(self, attribute: str):
        """Get SQL data type for attribute"""
        type_mapping = {
            "name": "VARCHAR(100)",
            "email": "VARCHAR(100)",
            "phone": "VARCHAR(20)",
            "date": "DATE",
            "timestamp": "TIMESTAMP",
            "amount": "DECIMAL(10,2)",
            "quantity": "INT",
            "description": "TEXT",
            "boolean": "BOOLEAN"
        }
        
        # Simple type inference based on attribute name
        for key, sql_type in type_mapping.items():
            if key in attribute.lower():
                return sql_type
        
        return "VARCHAR(255)"  # Default type
    
    def insert_satellite_record(self, parent_key: str, attributes: dict, record_source: str):
        """Insert new satellite record"""
        load_date = datetime.now()
        
        # Check if this is an update to existing record
        existing_record = self.get_latest_record(parent_key)
        if existing_record and self.has_changes(existing_record, attributes):
            # Close previous record
            self.close_previous_record(parent_key, load_date)
        
        record = {
            self.parent_key: parent_key,
            self.load_date: load_date,
            self.load_end_date: None,  # Open record
            self.record_source: record_source,
            self.hash_diff: self.calculate_hash_diff(attributes)
        }
        
        # Add attributes
        record.update(attributes)
        
        return record
    
    def has_changes(self, existing_record: dict, new_attributes: dict):
        """Check if there are changes in attributes"""
        for attr in self.attributes:
            if attr in new_attributes and attr in existing_record:
                if existing_record[attr] != new_attributes[attr]:
                    return True
        return False
    
    def close_previous_record(self, parent_key: str, new_load_date: datetime):
        """Close previous satellite record"""
        update_sql = f"""
        UPDATE {self.table_name} 
        SET {self.load_end_date} = %s 
        WHERE {self.parent_key} = %s 
        AND {self.load_end_date} IS NULL
        """
        # Execute update with new_load_date and parent_key
```

### 4. **Pit Tables (Point-in-Time)**

#### Purpose
Pit tables provide point-in-time snapshots for performance optimization, reducing the need for complex joins across historical data.

```sql
-- Customer Point-in-Time Table
CREATE TABLE PIT_CUSTOMER (
    CUSTOMER_HK VARCHAR(64) NOT NULL,
    EFFECTIVE_DATE DATE NOT NULL,
    DEMOGRAPHICS_LOAD_DATE TIMESTAMP,
    PREFERENCES_LOAD_DATE TIMESTAMP,
    PRIMARY KEY (CUSTOMER_HK, EFFECTIVE_DATE),
    FOREIGN KEY (CUSTOMER_HK) REFERENCES HUB_CUSTOMER(CUSTOMER_HK)
);

-- Order Point-in-Time Table
CREATE TABLE PIT_ORDER (
    ORDER_HK VARCHAR(64) NOT NULL,
    EFFECTIVE_DATE DATE NOT NULL,
    ORDER_DETAILS_LOAD_DATE TIMESTAMP,
    ORDER_STATUS_LOAD_DATE TIMESTAMP,
    PRIMARY KEY (ORDER_HK, EFFECTIVE_DATE),
    FOREIGN KEY (ORDER_HK) REFERENCES HUB_ORDER(ORDER_HK)
);
```

#### Pit Table Implementation
```python
class PitTable:
    def __init__(self, table_name: str, parent_hub: str, satellites: list):
        self.table_name = table_name
        self.parent_hub = parent_hub
        self.satellites = satellites
        self.parent_key = f"{parent_hub}_HK"
        self.effective_date = "EFFECTIVE_DATE"
    
    def create_table_sql(self):
        """Generate CREATE TABLE SQL for Pit Table"""
        satellite_columns = []
        for satellite in self.satellites:
            column_name = f"{satellite}_LOAD_DATE"
            satellite_columns.append(f"{column_name} TIMESTAMP")
        
        return f"""
        CREATE TABLE {self.table_name} (
            {self.parent_key} VARCHAR(64) NOT NULL,
            {self.effective_date} DATE NOT NULL,
            {', '.join(satellite_columns)},
            PRIMARY KEY ({self.parent_key}, {self.effective_date}),
            FOREIGN KEY ({self.parent_key}) REFERENCES HUB_{self.parent_hub}({self.parent_key})
        );
        """
    
    def generate_pit_snapshot(self, effective_date: date):
        """Generate point-in-time snapshot for given date"""
        pit_records = []
        
        # Get all hub records
        hub_records = self.get_hub_records()
        
        for hub_record in hub_records:
            pit_record = {
                self.parent_key: hub_record[self.parent_key],
                self.effective_date: effective_date
            }
            
            # Get latest satellite data for each satellite
            for satellite in self.satellites:
                latest_satellite = self.get_latest_satellite_data(
                    hub_record[self.parent_key], 
                    satellite, 
                    effective_date
                )
                if latest_satellite:
                    pit_record[f"{satellite}_LOAD_DATE"] = latest_satellite["LOAD_DATE"]
                else:
                    pit_record[f"{satellite}_LOAD_DATE"] = None
            
            pit_records.append(pit_record)
        
        return pit_records
```

### 5. **Bridge Tables**

#### Purpose
Bridge tables handle many-to-many relationships and provide denormalized views for performance optimization.

```sql
-- Customer-Product Bridge (Customer Preferences)
CREATE TABLE BRIDGE_CUSTOMER_PRODUCT_PREFERENCE (
    CUSTOMER_HK VARCHAR(64) NOT NULL,
    PRODUCT_HK VARCHAR(64) NOT NULL,
    PREFERENCE_SCORE DECIMAL(3,2),
    LAST_UPDATED TIMESTAMP,
    LOAD_DATE TIMESTAMP NOT NULL,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    PRIMARY KEY (CUSTOMER_HK, PRODUCT_HK, LOAD_DATE),
    FOREIGN KEY (CUSTOMER_HK) REFERENCES HUB_CUSTOMER(CUSTOMER_HK),
    FOREIGN KEY (PRODUCT_HK) REFERENCES HUB_PRODUCT(PRODUCT_HK)
);

-- Order-Product Bridge (Order Items)
CREATE TABLE BRIDGE_ORDER_PRODUCT_ITEM (
    ORDER_HK VARCHAR(64) NOT NULL,
    PRODUCT_HK VARCHAR(64) NOT NULL,
    QUANTITY INT,
    UNIT_PRICE DECIMAL(10,2),
    TOTAL_PRICE DECIMAL(10,2),
    DISCOUNT_PERCENTAGE DECIMAL(5,2),
    LOAD_DATE TIMESTAMP NOT NULL,
    RECORD_SOURCE VARCHAR(100) NOT NULL,
    PRIMARY KEY (ORDER_HK, PRODUCT_HK, LOAD_DATE),
    FOREIGN KEY (ORDER_HK) REFERENCES HUB_ORDER(ORDER_HK),
    FOREIGN KEY (PRODUCT_HK) REFERENCES HUB_PRODUCT(PRODUCT_HK)
);
```

## 🔄 Data Vault Implementation Patterns

### 1. **ETL Process for Data Vault**

```python
class DataVaultETL:
    def __init__(self, source_system: str):
        self.source_system = source_system
        self.hub_processor = HubProcessor()
        self.link_processor = LinkProcessor()
        self.satellite_processor = SatelliteProcessor()
    
    def process_source_data(self, source_data: list):
        """Process source data into Data Vault structure"""
        results = {
            "hubs": [],
            "links": [],
            "satellites": []
        }
        
        for record in source_data:
            # Process hubs
            hub_records = self.hub_processor.process_record(record, self.source_system)
            results["hubs"].extend(hub_records)
            
            # Process links
            link_records = self.link_processor.process_record(record, self.source_system)
            results["links"].extend(link_records)
            
            # Process satellites
            satellite_records = self.satellite_processor.process_record(record, self.source_system)
            results["satellites"].extend(satellite_records)
        
        return results
    
    def load_to_data_vault(self, processed_data: dict):
        """Load processed data to Data Vault tables"""
        # Load hubs
        for hub_record in processed_data["hubs"]:
            self.load_hub_record(hub_record)
        
        # Load links
        for link_record in processed_data["links"]:
            self.load_link_record(link_record)
        
        # Load satellites
        for satellite_record in processed_data["satellites"]:
            self.load_satellite_record(satellite_record)
    
    def load_hub_record(self, hub_record: dict):
        """Load hub record with duplicate checking"""
        # Check if hub record already exists
        existing = self.check_hub_exists(hub_record)
        if not existing:
            self.insert_hub_record(hub_record)
        elif self.has_hub_changes(existing, hub_record):
            self.update_hub_record(hub_record)
    
    def load_satellite_record(self, satellite_record: dict):
        """Load satellite record with change detection"""
        # Check if satellite record has changes
        if self.has_satellite_changes(satellite_record):
            # Close previous record
            self.close_previous_satellite_record(satellite_record)
            # Insert new record
            self.insert_satellite_record(satellite_record)
```

### 2. **Change Data Capture (CDC) for Data Vault**

```python
class DataVaultCDC:
    def __init__(self):
        self.change_detector = ChangeDetector()
        self.hash_calculator = HashCalculator()
    
    def detect_changes(self, current_record: dict, previous_record: dict):
        """Detect changes between current and previous records"""
        changes = {
            "hub_changes": [],
            "link_changes": [],
            "satellite_changes": []
        }
        
        # Detect hub changes
        hub_changes = self.change_detector.detect_hub_changes(current_record, previous_record)
        if hub_changes:
            changes["hub_changes"].append(hub_changes)
        
        # Detect link changes
        link_changes = self.change_detector.detect_link_changes(current_record, previous_record)
        if link_changes:
            changes["link_changes"].append(link_changes)
        
        # Detect satellite changes
        satellite_changes = self.change_detector.detect_satellite_changes(current_record, previous_record)
        if satellite_changes:
            changes["satellite_changes"].append(satellite_changes)
        
        return changes
    
    def process_cdc_changes(self, changes: dict):
        """Process CDC changes into Data Vault"""
        for change_type, change_list in changes.items():
            for change in change_list:
                if change_type == "hub_changes":
                    self.process_hub_change(change)
                elif change_type == "link_changes":
                    self.process_link_change(change)
                elif change_type == "satellite_changes":
                    self.process_satellite_change(change)
    
    def process_satellite_change(self, change: dict):
        """Process satellite change with proper historical tracking"""
        # Check if this is a new record or update
        if change["change_type"] == "INSERT":
            self.insert_new_satellite_record(change["record"])
        elif change["change_type"] == "UPDATE":
            # Close previous record
            self.close_previous_satellite_record(change["parent_key"])
            # Insert new record
            self.insert_new_satellite_record(change["record"])
        elif change["change_type"] == "DELETE":
            # Close previous record
            self.close_previous_satellite_record(change["parent_key"])
```

### 3. **Data Vault Automation**

```python
class DataVaultAutomation:
    def __init__(self):
        self.metadata_manager = MetadataManager()
        self.code_generator = CodeGenerator()
        self.deployment_manager = DeploymentManager()
    
    def auto_generate_data_vault(self, source_metadata: dict):
        """Automatically generate Data Vault structure from source metadata"""
        # Analyze source metadata
        analysis = self.analyze_source_metadata(source_metadata)
        
        # Generate hubs
        hubs = self.generate_hubs(analysis["business_keys"])
        
        # Generate links
        links = self.generate_links(analysis["relationships"])
        
        # Generate satellites
        satellites = self.generate_satellites(analysis["attributes"])
        
        # Generate DDL scripts
        ddl_scripts = self.generate_ddl_scripts(hubs, links, satellites)
        
        # Generate ETL code
        etl_code = self.generate_etl_code(hubs, links, satellites)
        
        return {
            "ddl_scripts": ddl_scripts,
            "etl_code": etl_code,
            "metadata": self.create_data_vault_metadata(hubs, links, satellites)
        }
    
    def analyze_source_metadata(self, source_metadata: dict):
        """Analyze source metadata to identify Data Vault components"""
        analysis = {
            "business_keys": [],
            "relationships": [],
            "attributes": []
        }
        
        for table_name, table_metadata in source_metadata.items():
            # Identify business keys
            business_keys = self.identify_business_keys(table_metadata)
            analysis["business_keys"].extend(business_keys)
            
            # Identify relationships
            relationships = self.identify_relationships(table_metadata)
            analysis["relationships"].extend(relationships)
            
            # Identify attributes
            attributes = self.identify_attributes(table_metadata)
            analysis["attributes"].extend(attributes)
        
        return analysis
    
    def generate_hubs(self, business_keys: list):
        """Generate hub definitions from business keys"""
        hubs = []
        
        for business_key in business_keys:
            hub = {
                "name": f"HUB_{business_key['entity_name']}",
                "business_key": business_key["key_name"],
                "attributes": business_key["attributes"]
            }
            hubs.append(hub)
        
        return hubs
    
    def generate_links(self, relationships: list):
        """Generate link definitions from relationships"""
        links = []
        
        for relationship in relationships:
            link = {
                "name": f"LINK_{relationship['name']}",
                "parent_hubs": relationship["parent_hubs"],
                "attributes": relationship.get("attributes", [])
            }
            links.append(link)
        
        return links
    
    def generate_satellites(self, attributes: list):
        """Generate satellite definitions from attributes"""
        satellites = []
        
        # Group attributes by entity and context
        grouped_attributes = self.group_attributes_by_context(attributes)
        
        for context, attr_group in grouped_attributes.items():
            satellite = {
                "name": f"SAT_{context}",
                "parent_hub": attr_group["parent_hub"],
                "attributes": attr_group["attributes"]
            }
            satellites.append(satellite)
        
        return satellites
```

## 📊 Data Vault Modeling Techniques

### 1. **Business Key Identification**

```python
class BusinessKeyIdentifier:
    def __init__(self):
        self.key_patterns = {
            "customer": ["customer_id", "cust_id", "client_id"],
            "product": ["product_id", "prod_id", "item_id"],
            "order": ["order_id", "ord_id", "transaction_id"],
            "employee": ["employee_id", "emp_id", "staff_id"]
        }
    
    def identify_business_keys(self, table_metadata: dict):
        """Identify business keys from table metadata"""
        business_keys = []
        
        for table_name, columns in table_metadata.items():
            # Look for primary key patterns
            primary_keys = self.find_primary_keys(columns)
            
            for pk in primary_keys:
                business_key = self.analyze_business_key(pk, table_name)
                if business_key:
                    business_keys.append(business_key)
        
        return business_keys
    
    def find_primary_keys(self, columns: list):
        """Find primary key columns"""
        primary_keys = []
        
        for column in columns:
            if column.get("is_primary_key") or column.get("constraint_type") == "PRIMARY KEY":
                primary_keys.append(column)
        
        return primary_keys
    
    def analyze_business_key(self, column: dict, table_name: str):
        """Analyze if column represents a business key"""
        column_name = column["name"].lower()
        
        # Check against known patterns
        for entity_type, patterns in self.key_patterns.items():
            for pattern in patterns:
                if pattern in column_name:
                    return {
                        "entity_name": entity_type.upper(),
                        "key_name": column["name"],
                        "data_type": column["data_type"],
                        "table_name": table_name
                    }
        
        # Check for generic ID patterns
        if column_name.endswith("_id") and len(column_name) > 3:
            entity_name = column_name.replace("_id", "").upper()
            return {
                "entity_name": entity_name,
                "key_name": column["name"],
                "data_type": column["data_type"],
                "table_name": table_name
            }
        
        return None
```

### 2. **Relationship Identification**

```python
class RelationshipIdentifier:
    def __init__(self):
        self.relationship_patterns = {
            "many_to_many": self.identify_many_to_many,
            "one_to_many": self.identify_one_to_many,
            "one_to_one": self.identify_one_to_one
        }
    
    def identify_relationships(self, table_metadata: dict):
        """Identify relationships between tables"""
        relationships = []
        
        # Find foreign key relationships
        foreign_keys = self.find_foreign_keys(table_metadata)
        
        for fk in foreign_keys:
            relationship = self.analyze_relationship(fk, table_metadata)
            if relationship:
                relationships.append(relationship)
        
        return relationships
    
    def find_foreign_keys(self, table_metadata: dict):
        """Find foreign key constraints"""
        foreign_keys = []
        
        for table_name, columns in table_metadata.items():
            for column in columns:
                if column.get("constraint_type") == "FOREIGN KEY":
                    foreign_keys.append({
                        "table_name": table_name,
                        "column_name": column["name"],
                        "referenced_table": column.get("referenced_table"),
                        "referenced_column": column.get("referenced_column")
                    })
        
        return foreign_keys
    
    def analyze_relationship(self, fk: dict, table_metadata: dict):
        """Analyze foreign key relationship"""
        # Determine relationship type
        relationship_type = self.determine_relationship_type(fk, table_metadata)
        
        if relationship_type == "many_to_many":
            return self.create_many_to_many_relationship(fk)
        elif relationship_type == "one_to_many":
            return self.create_one_to_many_relationship(fk)
        elif relationship_type == "one_to_one":
            return self.create_one_to_one_relationship(fk)
        
        return None
    
    def determine_relationship_type(self, fk: dict, table_metadata: dict):
        """Determine the type of relationship"""
        # Check if this is a junction table (many-to-many)
        if self.is_junction_table(fk["table_name"], table_metadata):
            return "many_to_many"
        
        # Check cardinality
        cardinality = self.check_cardinality(fk)
        if cardinality == "1:1":
            return "one_to_one"
        else:
            return "one_to_many"
    
    def is_junction_table(self, table_name: str, table_metadata: dict):
        """Check if table is a junction table"""
        table_columns = table_metadata.get(table_name, [])
        
        # Junction tables typically have only foreign keys and no other attributes
        fk_count = sum(1 for col in table_columns if col.get("constraint_type") == "FOREIGN KEY")
        total_columns = len(table_columns)
        
        # If most columns are foreign keys, it's likely a junction table
        return fk_count >= total_columns * 0.8
```

### 3. **Attribute Grouping and Context**

```python
class AttributeGrouper:
    def __init__(self):
        self.context_patterns = {
            "demographics": ["name", "age", "gender", "birth", "address", "phone", "email"],
            "preferences": ["preference", "setting", "option", "choice", "favorite"],
            "status": ["status", "state", "condition", "phase", "stage"],
            "financial": ["price", "cost", "amount", "balance", "payment", "billing"],
            "temporal": ["date", "time", "created", "updated", "modified", "expired"]
        }
    
    def group_attributes_by_context(self, attributes: list):
        """Group attributes by business context"""
        grouped_attributes = {}
        
        for attr in attributes:
            context = self.determine_attribute_context(attr)
            if context not in grouped_attributes:
                grouped_attributes[context] = {
                    "parent_hub": attr["parent_hub"],
                    "attributes": []
                }
            grouped_attributes[context]["attributes"].append(attr)
        
        return grouped_attributes
    
    def determine_attribute_context(self, attribute: dict):
        """Determine the business context of an attribute"""
        attr_name = attribute["name"].lower()
        
        # Check against context patterns
        for context, patterns in self.context_patterns.items():
            for pattern in patterns:
                if pattern in attr_name:
                    return context
        
        # Default context based on attribute type
        if attribute["data_type"] in ["DATE", "TIMESTAMP"]:
            return "temporal"
        elif attribute["data_type"] in ["DECIMAL", "NUMERIC", "MONEY"]:
            return "financial"
        else:
            return "general"
    
    def create_satellite_groups(self, grouped_attributes: dict):
        """Create satellite groups from attribute groups"""
        satellite_groups = []
        
        for context, attr_group in grouped_attributes.items():
            # Create satellite name
            satellite_name = f"SAT_{attr_group['parent_hub']}_{context.upper()}"
            
            satellite_group = {
                "name": satellite_name,
                "parent_hub": attr_group["parent_hub"],
                "context": context,
                "attributes": attr_group["attributes"]
            }
            
            satellite_groups.append(satellite_group)
        
        return satellite_groups
```

## 🎯 Data Vault Best Practices

### 1. **Naming Conventions**

```python
class DataVaultNamingConventions:
    def __init__(self):
        self.prefixes = {
            "hub": "HUB_",
            "link": "LINK_",
            "satellite": "SAT_",
            "pit": "PIT_",
            "bridge": "BRIDGE_"
        }
        
        self.suffixes = {
            "hash_key": "_HK",
            "business_key": "_BK",
            "load_date": "_LOAD_DATE",
            "record_source": "_RECORD_SOURCE"
        }
    
    def generate_hub_name(self, entity_name: str):
        """Generate hub table name"""
        return f"{self.prefixes['hub']}{entity_name.upper()}"
    
    def generate_link_name(self, relationship_name: str):
        """Generate link table name"""
        return f"{self.prefixes['link']}{relationship_name.upper()}"
    
    def generate_satellite_name(self, parent_hub: str, context: str):
        """Generate satellite table name"""
        return f"{self.prefixes['satellite']}{parent_hub.upper()}_{context.upper()}"
    
    def generate_column_name(self, column_type: str, entity_name: str = None):
        """Generate column name following conventions"""
        if column_type == "hash_key":
            return f"{entity_name.upper()}{self.suffixes['hash_key']}"
        elif column_type == "business_key":
            return f"{entity_name.upper()}{self.suffixes['business_key']}"
        elif column_type == "load_date":
            return self.suffixes["load_date"]
        elif column_type == "record_source":
            return self.suffixes["record_source"]
        
        return column_type.upper()
```

### 2. **Performance Optimization**

```python
class DataVaultPerformanceOptimizer:
    def __init__(self):
        self.index_strategies = {
            "hub": self.create_hub_indexes,
            "link": self.create_link_indexes,
            "satellite": self.create_satellite_indexes
        }
    
    def optimize_data_vault_performance(self, data_vault_structure: dict):
        """Optimize Data Vault performance"""
        optimizations = {
            "indexes": [],
            "partitions": [],
            "compression": []
        }
        
        # Create indexes
        for table_type, tables in data_vault_structure.items():
            for table in tables:
                indexes = self.index_strategies[table_type](table)
                optimizations["indexes"].extend(indexes)
        
        # Create partitions
        partitions = self.create_partition_strategy(data_vault_structure)
        optimizations["partitions"].extend(partitions)
        
        # Apply compression
        compression = self.create_compression_strategy(data_vault_structure)
        optimizations["compression"].extend(compression)
        
        return optimizations
    
    def create_hub_indexes(self, hub_table: dict):
        """Create indexes for hub tables"""
        indexes = [
            f"CREATE INDEX idx_{hub_table['name']}_bk ON {hub_table['name']} ({hub_table['business_key']});",
            f"CREATE INDEX idx_{hub_table['name']}_load_date ON {hub_table['name']} (LOAD_DATE);",
            f"CREATE INDEX idx_{hub_table['name']}_record_source ON {hub_table['name']} (RECORD_SOURCE);"
        ]
        return indexes
    
    def create_link_indexes(self, link_table: dict):
        """Create indexes for link tables"""
        indexes = [
            f"CREATE INDEX idx_{link_table['name']}_parent_hubs ON {link_table['name']} ({', '.join(link_table['parent_hubs'])});",
            f"CREATE INDEX idx_{link_table['name']}_load_date ON {link_table['name']} (LOAD_DATE);"
        ]
        return indexes
    
    def create_satellite_indexes(self, satellite_table: dict):
        """Create indexes for satellite tables"""
        indexes = [
            f"CREATE INDEX idx_{satellite_table['name']}_parent_hub ON {satellite_table['name']} ({satellite_table['parent_hub']}_HK);",
            f"CREATE INDEX idx_{satellite_table['name']}_load_date ON {satellite_table['name']} (LOAD_DATE);",
            f"CREATE INDEX idx_{satellite_table['name']}_load_end_date ON {satellite_table['name']} (LOAD_END_DATE);"
        ]
        return indexes
    
    def create_partition_strategy(self, data_vault_structure: dict):
        """Create partitioning strategy for Data Vault tables"""
        partitions = []
        
        # Partition satellites by load_date
        for satellite in data_vault_structure.get("satellites", []):
            partition_sql = f"""
            ALTER TABLE {satellite['name']} 
            PARTITION BY RANGE (LOAD_DATE) (
                PARTITION p_2023 VALUES LESS THAN ('2024-01-01'),
                PARTITION p_2024 VALUES LESS THAN ('2025-01-01'),
                PARTITION p_future VALUES LESS THAN MAXVALUE
            );
            """
            partitions.append(partition_sql)
        
        return partitions
```

### 3. **Data Quality and Governance**

```python
class DataVaultDataQuality:
    def __init__(self):
        self.quality_rules = {
            "hub_quality": self.check_hub_quality,
            "link_quality": self.check_link_quality,
            "satellite_quality": self.check_satellite_quality
        }
    
    def validate_data_vault_quality(self, data_vault_data: dict):
        """Validate Data Vault data quality"""
        quality_report = {
            "overall_score": 0,
            "hub_quality": {},
            "link_quality": {},
            "satellite_quality": {},
            "issues": []
        }
        
        # Check hub quality
        hub_quality = self.check_hub_quality(data_vault_data.get("hubs", []))
        quality_report["hub_quality"] = hub_quality
        
        # Check link quality
        link_quality = self.check_link_quality(data_vault_data.get("links", []))
        quality_report["link_quality"] = link_quality
        
        # Check satellite quality
        satellite_quality = self.check_satellite_quality(data_vault_data.get("satellites", []))
        quality_report["satellite_quality"] = satellite_quality
        
        # Calculate overall score
        quality_report["overall_score"] = self.calculate_overall_score(
            hub_quality, link_quality, satellite_quality
        )
        
        return quality_report
    
    def check_hub_quality(self, hub_data: list):
        """Check hub data quality"""
        quality_metrics = {
            "duplicate_business_keys": 0,
            "missing_business_keys": 0,
            "invalid_hash_keys": 0,
            "orphaned_records": 0
        }
        
        business_keys = set()
        hash_keys = set()
        
        for hub_record in hub_data:
            # Check for duplicate business keys
            if hub_record["business_key"] in business_keys:
                quality_metrics["duplicate_business_keys"] += 1
            else:
                business_keys.add(hub_record["business_key"])
            
            # Check for missing business keys
            if not hub_record["business_key"]:
                quality_metrics["missing_business_keys"] += 1
            
            # Check hash key validity
            if not self.is_valid_hash_key(hub_record["hash_key"]):
                quality_metrics["invalid_hash_keys"] += 1
            else:
                hash_keys.add(hub_record["hash_key"])
        
        return quality_metrics
    
    def check_link_quality(self, link_data: list):
        """Check link data quality"""
        quality_metrics = {
            "orphaned_links": 0,
            "duplicate_relationships": 0,
            "invalid_foreign_keys": 0
        }
        
        relationships = set()
        
        for link_record in link_data:
            # Check for duplicate relationships
            relationship_key = tuple(link_record["parent_keys"].values())
            if relationship_key in relationships:
                quality_metrics["duplicate_relationships"] += 1
            else:
                relationships.add(relationship_key)
            
            # Check for orphaned links
            if self.is_orphaned_link(link_record):
                quality_metrics["orphaned_links"] += 1
        
        return quality_metrics
    
    def check_satellite_quality(self, satellite_data: list):
        """Check satellite data quality"""
        quality_metrics = {
            "orphaned_satellites": 0,
            "duplicate_attributes": 0,
            "invalid_load_dates": 0
        }
        
        for satellite_record in satellite_data:
            # Check for orphaned satellites
            if self.is_orphaned_satellite(satellite_record):
                quality_metrics["orphaned_satellites"] += 1
            
            # Check load date validity
            if not self.is_valid_load_date(satellite_record["load_date"]):
                quality_metrics["invalid_load_dates"] += 1
        
        return quality_metrics
    
    def is_valid_hash_key(self, hash_key: str):
        """Check if hash key is valid"""
        return hash_key and len(hash_key) == 16 and hash_key.isalnum()
    
    def is_orphaned_link(self, link_record: dict):
        """Check if link record is orphaned"""
        # Check if parent hub records exist
        for parent_hub, parent_key in link_record["parent_keys"].items():
            if not self.hub_record_exists(parent_hub, parent_key):
                return True
        return False
    
    def is_orphaned_satellite(self, satellite_record: dict):
        """Check if satellite record is orphaned"""
        parent_hub = satellite_record["parent_hub"]
        parent_key = satellite_record["parent_key"]
        return not self.hub_record_exists(parent_hub, parent_key)
```

## 🔗 Related Concepts

- [Data Warehouse](../datawarehouse/README.md)
- [Data Lake](../datalake/README.md)
- [Data Lakehouse](../datalakehouse/README.md)
- [Schema Change](../schema-change/README.md)
- [Data Modeling Patterns](../../design-pattern/README.md)

---

*Data Vault methodology provides a robust, scalable approach to data warehousing that handles complex business requirements while maintaining data integrity and auditability. Focus on proper business key identification, relationship modeling, and data quality to ensure successful implementation.*
