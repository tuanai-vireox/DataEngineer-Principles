# Data Governance & Compliance

Data Governance & Compliance is a comprehensive framework that ensures data is managed, protected, and used in accordance with organizational policies, industry standards, and regulatory requirements. This documentation covers governance frameworks, compliance regulations, data quality management, and implementation strategies.

## 📋 Table of Contents

1. [Data Governance Framework](#data-governance-framework)
2. [Compliance Regulations](#compliance-regulations)
3. [Data Quality Management](#data-quality-management)
4. [Data Lineage & Cataloging](#data-lineage--cataloging)
5. [Data Privacy & Protection](#data-privacy--protection)
6. [Data Retention & Lifecycle](#data-retention--lifecycle)
7. [Access Control & Classification](#access-control--classification)
8. [Compliance Monitoring](#compliance-monitoring)
9. [Implementation Examples](#implementation-examples)
10. [Best Practices](#best-practices)

## 🏛️ Data Governance Framework

### Core Principles

1. **Accountability**: Clear ownership and responsibility for data assets
2. **Transparency**: Visible and understandable data processes
3. **Integrity**: Data accuracy, completeness, and consistency
4. **Protection**: Security and privacy of data assets
5. **Compliance**: Adherence to regulations and standards
6. **Quality**: High standards for data quality
7. **Value**: Maximize business value from data

### Governance Components

#### 1. Data Stewardship Model

```python
# Data Stewardship Framework
from dataclasses import dataclass
from typing import List, Optional
from enum import Enum
from datetime import datetime

class StewardshipRole(Enum):
    DATA_OWNER = "data_owner"           # Business owner, accountable for data
    DATA_STEWARD = "data_steward"       # Technical custodian, manages data quality
    DATA_CUSTODIAN = "data_custodian"   # IT team, manages infrastructure
    DATA_USER = "data_user"            # End user, consumes data

@dataclass
class DataSteward:
    name: str
    email: str
    role: StewardshipRole
    domain: str
    responsibilities: List[str]
    assigned_date: datetime

class DataStewardship:
    def __init__(self):
        self.stewards = {}
        self.data_asset_ownership = {}
    
    def assign_steward(self, asset_name: str, steward: DataSteward):
        """Assign a data steward to a data asset"""
        if asset_name not in self.data_asset_ownership:
            self.data_asset_ownership[asset_name] = []
        
        self.data_asset_ownership[asset_name].append(steward)
        self.stewards[steward.email] = steward
    
    def get_stewards(self, asset_name: str) -> List[DataSteward]:
        """Get all stewards for a data asset"""
        return self.data_asset_ownership.get(asset_name, [])
    
    def get_owner(self, asset_name: str) -> Optional[DataSteward]:
        """Get the data owner for an asset"""
        stewards = self.get_stewards(asset_name)
        owners = [s for s in stewards if s.role == StewardshipRole.DATA_OWNER]
        return owners[0] if owners else None
```

#### 2. Data Governance Council

- **Executive Sponsor**: C-level executive champion
- **Data Governance Officer**: Leads governance program
- **Business Representatives**: Domain experts from business units
- **IT Representatives**: Technical infrastructure experts
- **Legal & Compliance**: Regulatory and legal advisors
- **Security Team**: Information security experts

#### 3. Governance Policies

```python
# Data Governance Policies
from dataclasses import dataclass
from typing import Dict, List, Any
from datetime import datetime, timedelta

@dataclass
class DataPolicy:
    policy_id: str
    name: str
    description: str
    category: str  # e.g., "access_control", "data_quality", "retention"
    rules: Dict[str, Any]
    effective_date: datetime
    review_date: datetime
    owner: str
    status: str  # "draft", "active", "archived"

class PolicyManagement:
    def __init__(self):
        self.policies = {}
    
    def create_policy(self, policy: DataPolicy):
        """Create a new data governance policy"""
        self.policies[policy.policy_id] = policy
    
    def get_policy(self, policy_id: str) -> DataPolicy:
        """Retrieve a policy by ID"""
        return self.policies.get(policy_id)
    
    def get_policies_by_category(self, category: str) -> List[DataPolicy]:
        """Get all policies in a category"""
        return [p for p in self.policies.values() if p.category == category]
    
    def check_policy_compliance(self, asset_name: str, action: str) -> bool:
        """Check if an action complies with policies"""
        relevant_policies = self._get_relevant_policies(asset_name, action)
        
        for policy in relevant_policies:
            if not self._evaluate_policy(policy, asset_name, action):
                return False
        
        return True
    
    def _get_relevant_policies(self, asset_name: str, action: str) -> List[DataPolicy]:
        """Get policies relevant to asset and action"""
        # Implementation to filter relevant policies
        return list(self.policies.values())
    
    def _evaluate_policy(self, policy: DataPolicy, asset_name: str, action: str) -> bool:
        """Evaluate if action complies with policy"""
        # Implementation to evaluate policy rules
        return True
```

### Governance Maturity Model

1. **Level 1 - Initial**: Ad-hoc processes, no formal governance
2. **Level 2 - Managed**: Basic policies and procedures in place
3. **Level 3 - Defined**: Formal governance framework established
4. **Level 4 - Quantitatively Managed**: Metrics-driven governance
5. **Level 5 - Optimizing**: Continuous improvement and innovation

## 📜 Compliance Regulations

### GDPR (General Data Protection Regulation)

#### Key Requirements

1. **Lawful Basis for Processing**
   - Consent
   - Contractual necessity
   - Legal obligation
   - Vital interests
   - Public task
   - Legitimate interests

2. **Data Subject Rights**
   - Right to access
   - Right to rectification
   - Right to erasure (right to be forgotten)
   - Right to restrict processing
   - Right to data portability
   - Right to object
   - Rights related to automated decision-making

3. **Data Protection Principles**
   - Lawfulness, fairness, and transparency
   - Purpose limitation
   - Data minimization
   - Accuracy
   - Storage limitation
   - Integrity and confidentiality
   - Accountability

#### GDPR Implementation

```python
# GDPR Compliance Framework
from typing import Dict, List, Any, Optional
from datetime import datetime, timedelta
from enum import Enum
import json

class ProcessingPurpose(Enum):
    MARKETING = "marketing"
    ANALYTICS = "analytics"
    SERVICE_DELIVERY = "service_delivery"
    LEGAL_COMPLIANCE = "legal_compliance"

class LawfulBasis(Enum):
    CONSENT = "consent"
    CONTRACT = "contract"
    LEGAL_OBLIGATION = "legal_obligation"
    VITAL_INTERESTS = "vital_interests"
    PUBLIC_TASK = "public_task"
    LEGITIMATE_INTERESTS = "legitimate_interests"

@dataclass
class ConsentRecord:
    user_id: str
    purpose: ProcessingPurpose
    lawful_basis: LawfulBasis
    consent_given: bool
    consent_date: datetime
    consent_method: str  # "explicit", "opt_in", "opt_out"
    withdrawal_date: Optional[datetime] = None

class GDPRCompliance:
    def __init__(self):
        self.consent_records = {}  # user_id -> List[ConsentRecord]
        self.processing_activities = []
        self.data_retention_policies = {}
        self.data_breach_log = []
    
    def record_consent(self, user_id: str, purpose: ProcessingPurpose, 
                      lawful_basis: LawfulBasis, consent_given: bool):
        """Record user consent for data processing"""
        if user_id not in self.consent_records:
            self.consent_records[user_id] = []
        
        consent = ConsentRecord(
            user_id=user_id,
            purpose=purpose,
            lawful_basis=lawful_basis,
            consent_given=consent_given,
            consent_date=datetime.now(),
            consent_method="explicit"
        )
        
        self.consent_records[user_id].append(consent)
    
    def withdraw_consent(self, user_id: str, purpose: ProcessingPurpose):
        """Handle consent withdrawal"""
        if user_id in self.consent_records:
            for consent in self.consent_records[user_id]:
                if consent.purpose == purpose and consent.consent_given:
                    consent.withdrawal_date = datetime.now()
                    consent.consent_given = False
    
    def has_valid_consent(self, user_id: str, purpose: ProcessingPurpose) -> bool:
        """Check if user has valid consent for processing purpose"""
        if user_id not in self.consent_records:
            return False
        
        for consent in self.consent_records[user_id]:
            if (consent.purpose == purpose and 
                consent.consent_given and 
                consent.withdrawal_date is None):
                return True
        
        return False
    
    def handle_data_subject_request(self, user_id: str, request_type: str) -> Dict[str, Any]:
        """Handle GDPR data subject requests"""
        if request_type == "access":
            return self._handle_access_request(user_id)
        elif request_type == "rectification":
            return self._handle_rectification_request(user_id)
        elif request_type == "erasure":
            return self._handle_erasure_request(user_id)
        elif request_type == "portability":
            return self._handle_portability_request(user_id)
        elif request_type == "restrict":
            return self._handle_restriction_request(user_id)
        else:
            return {"error": "Invalid request type"}
    
    def _handle_access_request(self, user_id: str) -> Dict[str, Any]:
        """Right to access - provide all personal data"""
        return {
            "user_id": user_id,
            "personal_data": self._get_user_data(user_id),
            "processing_purposes": self._get_processing_purposes(user_id),
            "data_sources": self._get_data_sources(user_id),
            "retention_period": self._get_retention_info(user_id),
            "third_parties": self._get_third_party_sharing(user_id)
        }
    
    def _handle_erasure_request(self, user_id: str) -> Dict[str, Any]:
        """Right to erasure - delete user data"""
        deleted_data = self._delete_user_data(user_id)
        return {
            "user_id": user_id,
            "deleted_data": deleted_data,
            "deletion_date": datetime.now().isoformat(),
            "status": "completed"
        }
    
    def _handle_portability_request(self, user_id: str) -> Dict[str, Any]:
        """Right to data portability - export data in machine-readable format"""
        user_data = self._get_user_data(user_id)
        return {
            "user_id": user_id,
            "data": user_data,
            "format": "json",
            "export_date": datetime.now().isoformat()
        }
    
    def record_data_breach(self, breach_details: Dict[str, Any]):
        """Record and report data breach (required within 72 hours)"""
        breach_record = {
            "breach_id": f"BR-{datetime.now().strftime('%Y%m%d%H%M%S')}",
            "timestamp": datetime.now().isoformat(),
            "details": breach_details,
            "reported_to_authority": False,
            "notified_data_subjects": False
        }
        
        self.data_breach_log.append(breach_record)
        
        # Auto-report if required
        if breach_details.get("severity") == "high":
            self._report_to_authority(breach_record)
    
    def _get_user_data(self, user_id: str) -> Dict[str, Any]:
        """Retrieve all user data"""
        # Implementation to fetch user data from all systems
        return {}
    
    def _delete_user_data(self, user_id: str) -> List[str]:
        """Delete user data from all systems"""
        # Implementation to delete user data
        return []
    
    def _get_processing_purposes(self, user_id: str) -> List[str]:
        """Get all processing purposes for user"""
        if user_id in self.consent_records:
            return [c.purpose.value for c in self.consent_records[user_id]]
        return []
    
    def _get_data_sources(self, user_id: str) -> List[str]:
        """Get all data sources for user"""
        # Implementation
        return []
    
    def _get_retention_info(self, user_id: str) -> Dict[str, Any]:
        """Get data retention information"""
        # Implementation
        return {}
    
    def _get_third_party_sharing(self, user_id: str) -> List[str]:
        """Get third parties with whom data is shared"""
        # Implementation
        return []
    
    def _report_to_authority(self, breach_record: Dict[str, Any]):
        """Report breach to supervisory authority"""
        # Implementation to report to GDPR authority
        breach_record["reported_to_authority"] = True
```

### CCPA (California Consumer Privacy Act)

#### Key Requirements

1. **Consumer Rights**
   - Right to know what personal information is collected
   - Right to know if personal information is sold or disclosed
   - Right to say no to the sale of personal information
   - Right to access personal information
   - Right to equal service and price

2. **Business Obligations**
   - Privacy policy disclosure
   - Opt-out mechanisms
   - Verification processes
   - Non-discrimination

#### CCPA Implementation

```python
# CCPA Compliance Framework
class CCPACompliance:
    def __init__(self):
        self.data_collection_log = {}
        self.data_sales_log = {}
        self.opt_out_requests = {}
    
    def record_data_collection(self, consumer_id: str, data_categories: List[str]):
        """Record data collection activities"""
        if consumer_id not in self.data_collection_log:
            self.data_collection_log[consumer_id] = []
        
        self.data_collection_log[consumer_id].append({
            "timestamp": datetime.now().isoformat(),
            "data_categories": data_categories
        })
    
    def record_data_sale(self, consumer_id: str, third_party: str, data_categories: List[str]):
        """Record data sales to third parties"""
        if consumer_id not in self.data_sales_log:
            self.data_sales_log[consumer_id] = []
        
        self.data_sales_log[consumer_id].append({
            "timestamp": datetime.now().isoformat(),
            "third_party": third_party,
            "data_categories": data_categories
        })
    
    def handle_opt_out(self, consumer_id: str):
        """Handle consumer opt-out request"""
        self.opt_out_requests[consumer_id] = {
            "opt_out_date": datetime.now().isoformat(),
            "status": "active"
        }
    
    def can_sell_data(self, consumer_id: str) -> bool:
        """Check if data can be sold for consumer"""
        return consumer_id not in self.opt_out_requests
    
    def get_disclosure_information(self, consumer_id: str) -> Dict[str, Any]:
        """Provide disclosure information to consumer"""
        return {
            "consumer_id": consumer_id,
            "collected_categories": self.data_collection_log.get(consumer_id, []),
            "sold_categories": self.data_sales_log.get(consumer_id, []),
            "opt_out_status": consumer_id in self.opt_out_requests
        }
```

### HIPAA (Health Insurance Portability and Accountability Act)

#### Key Requirements

1. **Protected Health Information (PHI)**
   - Individually identifiable health information
   - Created, received, maintained, or transmitted
   - By covered entities or business associates

2. **Administrative Safeguards**
   - Security management process
   - Assigned security responsibility
   - Workforce security
   - Information access management
   - Security awareness and training

3. **Physical Safeguards**
   - Facility access controls
   - Workstation use
   - Workstation security
   - Device and media controls

4. **Technical Safeguards**
   - Access control
   - Audit controls
   - Integrity controls
   - Transmission security

#### HIPAA Implementation

```python
# HIPAA Compliance Framework
class HIPAACompliance:
    def __init__(self):
        self.phi_access_log = []
        self.business_associates = {}
        self.encryption_keys = {}
    
    def log_phi_access(self, user_id: str, phi_id: str, purpose: str, 
                      access_type: str):
        """Log all PHI access for audit trail"""
        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "user_id": user_id,
            "phi_id": phi_id,
            "purpose": purpose,
            "access_type": access_type
        }
        self.phi_access_log.append(log_entry)
    
    def encrypt_phi(self, phi_data: str, encryption_key: str) -> str:
        """Encrypt PHI data"""
        # Implementation for encryption
        return f"encrypted_{phi_data}"
    
    def decrypt_phi(self, encrypted_data: str, encryption_key: str) -> str:
        """Decrypt PHI data"""
        # Implementation for decryption
        return encrypted_data.replace("encrypted_", "")
    
    def validate_minimum_necessary(self, user_id: str, requested_phi: List[str]) -> bool:
        """Validate minimum necessary rule"""
        # Check if user has access to requested PHI
        # Implementation
        return True
    
    def generate_breach_report(self, breach_details: Dict[str, Any]) -> Dict[str, Any]:
        """Generate breach report for HIPAA compliance"""
        return {
            "breach_id": f"HIPAA-BR-{datetime.now().strftime('%Y%m%d%H%M%S')}",
            "timestamp": datetime.now().isoformat(),
            "details": breach_details,
            "affected_individuals": breach_details.get("affected_count", 0),
            "notification_required": breach_details.get("affected_count", 0) > 500
        }
```

### PCI DSS (Payment Card Industry Data Security Standard)

#### Key Requirements

1. **Build and Maintain Secure Network**
2. **Protect Cardholder Data**
3. **Maintain Vulnerability Management Program**
4. **Implement Strong Access Control**
5. **Monitor and Test Networks**
6. **Maintain Information Security Policy**

#### PCI DSS Implementation

```python
# PCI DSS Compliance Framework
class PCIDSSCompliance:
    def __init__(self):
        self.cardholder_data = {}
        self.access_logs = []
        self.network_segments = {}
    
    def tokenize_card_data(self, card_number: str) -> str:
        """Tokenize cardholder data"""
        # Implementation for tokenization
        return f"TOKEN_{hash(card_number)}"
    
    def mask_card_data(self, card_number: str) -> str:
        """Mask card data for display"""
        if len(card_number) >= 4:
            return f"****-****-****-{card_number[-4:]}"
        return "****"
    
    def validate_network_segmentation(self, network_id: str) -> bool:
        """Validate network segmentation for cardholder data"""
        # Implementation
        return True
    
    def log_cardholder_access(self, user_id: str, action: str, card_token: str):
        """Log all access to cardholder data"""
        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "user_id": user_id,
            "action": action,
            "card_token": card_token
        }
        self.access_logs.append(log_entry)
```

## 📊 Data Quality Management

### Data Quality Dimensions

1. **Accuracy**: Data correctly represents real-world entities
2. **Completeness**: All required data is present
3. **Consistency**: Data is consistent across systems
4. **Timeliness**: Data is available when needed
5. **Validity**: Data conforms to defined rules
6. **Uniqueness**: No duplicate records
7. **Integrity**: Data relationships are maintained

### Data Quality Framework

```python
# Data Quality Management Framework
from typing import Dict, List, Any, Callable
from dataclasses import dataclass
from enum import Enum

class QualityDimension(Enum):
    ACCURACY = "accuracy"
    COMPLETENESS = "completeness"
    CONSISTENCY = "consistency"
    TIMELINESS = "timeliness"
    VALIDITY = "validity"
    UNIQUENESS = "uniqueness"
    INTEGRITY = "integrity"

@dataclass
class QualityRule:
    rule_id: str
    name: str
    dimension: QualityDimension
    field: str
    validation_function: Callable
    severity: str  # "critical", "warning", "info"
    description: str

@dataclass
class QualityIssue:
    rule_id: str
    record_id: str
    field: str
    issue_type: str
    severity: str
    message: str
    timestamp: datetime

class DataQualityManager:
    def __init__(self):
        self.quality_rules = {}
        self.quality_issues = []
        self.quality_metrics = {}
    
    def add_quality_rule(self, rule: QualityRule):
        """Add a data quality rule"""
        self.quality_rules[rule.rule_id] = rule
    
    def validate_data(self, data: List[Dict[str, Any]], 
                     dataset_name: str) -> Dict[str, Any]:
        """Validate data against quality rules"""
        issues = []
        total_records = len(data)
        valid_records = 0
        
        for record in data:
            record_valid = True
            for rule_id, rule in self.quality_rules.items():
                if rule.field in record:
                    try:
                        if not rule.validation_function(record[rule.field]):
                            issue = QualityIssue(
                                rule_id=rule_id,
                                record_id=str(record.get("id", "unknown")),
                                field=rule.field,
                                issue_type=rule.dimension.value,
                                severity=rule.severity,
                                message=f"{rule.name}: {rule.description}",
                                timestamp=datetime.now()
                            )
                            issues.append(issue)
                            record_valid = False
                    except Exception as e:
                        issue = QualityIssue(
                            rule_id=rule_id,
                            record_id=str(record.get("id", "unknown")),
                            field=rule.field,
                            issue_type="validation_error",
                            severity="critical",
                            message=f"Validation error: {str(e)}",
                            timestamp=datetime.now()
                        )
                        issues.append(issue)
                        record_valid = False
            
            if record_valid:
                valid_records += 1
        
        quality_score = (valid_records / total_records) * 100 if total_records > 0 else 0
        
        result = {
            "dataset_name": dataset_name,
            "total_records": total_records,
            "valid_records": valid_records,
            "invalid_records": total_records - valid_records,
            "quality_score": quality_score,
            "issues": issues,
            "timestamp": datetime.now().isoformat()
        }
        
        self.quality_issues.extend(issues)
        self.quality_metrics[dataset_name] = result
        
        return result
    
    def get_quality_report(self, dataset_name: str) -> Dict[str, Any]:
        """Get quality report for a dataset"""
        return self.quality_metrics.get(dataset_name, {})
    
    def get_quality_trends(self, dataset_name: str, days: int = 30) -> Dict[str, Any]:
        """Get quality trends over time"""
        # Implementation to track quality over time
        return {}

# Example Quality Rules
def validate_email(email: str) -> bool:
    """Validate email format"""
    import re
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

def validate_phone(phone: str) -> bool:
    """Validate phone number format"""
    import re
    pattern = r'^\+?1?\d{9,15}$'
    return bool(re.match(pattern, phone.replace("-", "").replace(" ", "")))

def validate_not_null(value: Any) -> bool:
    """Validate field is not null"""
    return value is not None and value != ""

# Usage Example
quality_manager = DataQualityManager()

# Add quality rules
quality_manager.add_quality_rule(QualityRule(
    rule_id="email_format",
    name="Email Format Validation",
    dimension=QualityDimension.VALIDITY,
    field="email",
    validation_function=validate_email,
    severity="critical",
    description="Email must be in valid format"
))

quality_manager.add_quality_rule(QualityRule(
    rule_id="phone_format",
    name="Phone Format Validation",
    dimension=QualityDimension.VALIDITY,
    field="phone",
    validation_function=validate_phone,
    severity="warning",
    description="Phone must be in valid format"
))

quality_manager.add_quality_rule(QualityRule(
    rule_id="name_required",
    name="Name Completeness",
    dimension=QualityDimension.COMPLETENESS,
    field="name",
    validation_function=validate_not_null,
    severity="critical",
    description="Name field is required"
))
```

## 🔗 Data Lineage & Cataloging

### Data Lineage

Data lineage tracks the flow of data from source to destination, including all transformations and processes.

```python
# Data Lineage Framework
from typing import Dict, List, Optional
from dataclasses import dataclass
from datetime import datetime

@dataclass
class DataLineageNode:
    node_id: str
    name: str
    type: str  # "source", "transformation", "destination"
    location: str
    schema: Dict[str, Any]
    metadata: Dict[str, Any]

@dataclass
class DataLineageEdge:
    source_node_id: str
    target_node_id: str
    transformation: str
    timestamp: datetime
    user: str

class DataLineageTracker:
    def __init__(self):
        self.nodes = {}
        self.edges = []
    
    def register_node(self, node: DataLineageNode):
        """Register a data node in lineage"""
        self.nodes[node.node_id] = node
    
    def add_lineage(self, edge: DataLineageEdge):
        """Add lineage relationship between nodes"""
        self.edges.append(edge)
    
    def get_upstream_lineage(self, node_id: str) -> List[DataLineageNode]:
        """Get all upstream nodes (sources)"""
        upstream_nodes = []
        visited = set()
        
        def traverse(node_id: str):
            if node_id in visited:
                return
            
            visited.add(node_id)
            
            for edge in self.edges:
                if edge.target_node_id == node_id:
                    source_node = self.nodes.get(edge.source_node_id)
                    if source_node:
                        upstream_nodes.append(source_node)
                        traverse(edge.source_node_id)
        
        traverse(node_id)
        return upstream_nodes
    
    def get_downstream_lineage(self, node_id: str) -> List[DataLineageNode]:
        """Get all downstream nodes (destinations)"""
        downstream_nodes = []
        visited = set()
        
        def traverse(node_id: str):
            if node_id in visited:
                return
            
            visited.add(node_id)
            
            for edge in self.edges:
                if edge.source_node_id == node_id:
                    target_node = self.nodes.get(edge.target_node_id)
                    if target_node:
                        downstream_nodes.append(target_node)
                        traverse(edge.target_node_id)
        
        traverse(node_id)
        return downstream_nodes
    
    def get_full_lineage(self, node_id: str) -> Dict[str, List[DataLineageNode]]:
        """Get complete lineage (upstream and downstream)"""
        return {
            "upstream": self.get_upstream_lineage(node_id),
            "downstream": self.get_downstream_lineage(node_id)
        }
    
    def get_impact_analysis(self, node_id: str) -> Dict[str, Any]:
        """Analyze impact of changes to a node"""
        downstream = self.get_downstream_lineage(node_id)
        return {
            "node_id": node_id,
            "affected_nodes": len(downstream),
            "downstream_nodes": [n.name for n in downstream],
            "risk_level": "high" if len(downstream) > 10 else "medium" if len(downstream) > 5 else "low"
        }
```

### Data Catalog

```python
# Data Catalog Framework
@dataclass
class DataAsset:
    asset_id: str
    name: str
    description: str
    type: str  # "table", "view", "file", "stream"
    location: str
    schema: Dict[str, Any]
    owner: str
    steward: str
    classification: str  # "public", "internal", "confidential", "restricted"
    tags: List[str]
    created_date: datetime
    updated_date: datetime
    quality_score: float
    usage_stats: Dict[str, Any]

class DataCatalog:
    def __init__(self):
        self.assets = {}
        self.search_index = {}
    
    def register_asset(self, asset: DataAsset):
        """Register a data asset in catalog"""
        self.assets[asset.asset_id] = asset
        self._update_search_index(asset)
    
    def search_assets(self, query: str, filters: Dict[str, Any] = None) -> List[DataAsset]:
        """Search data assets"""
        results = []
        
        for asset in self.assets.values():
            if self._matches_query(asset, query, filters):
                results.append(asset)
        
        return results
    
    def _matches_query(self, asset: DataAsset, query: str, filters: Dict[str, Any]) -> bool:
        """Check if asset matches search query and filters"""
        # Text search
        query_lower = query.lower()
        if (query_lower in asset.name.lower() or 
            query_lower in asset.description.lower() or
            any(query_lower in tag.lower() for tag in asset.tags)):
            pass
        else:
            return False
        
        # Apply filters
        if filters:
            if "classification" in filters and asset.classification != filters["classification"]:
                return False
            if "owner" in filters and asset.owner != filters["owner"]:
                return False
            if "type" in filters and asset.type != filters["type"]:
                return False
        
        return True
    
    def get_asset(self, asset_id: str) -> Optional[DataAsset]:
        """Get asset by ID"""
        return self.assets.get(asset_id)
    
    def _update_search_index(self, asset: DataAsset):
        """Update search index for asset"""
        # Implementation for search indexing
        pass
```

## 🔒 Data Privacy & Protection

### Data Classification

```python
# Data Classification Framework
class DataClassification(Enum):
    PUBLIC = "public"                    # No restrictions
    INTERNAL = "internal"               # Internal use only
    CONFIDENTIAL = "confidential"       # Restricted access
    RESTRICTED = "restricted"           # Highly restricted
    TOP_SECRET = "top_secret"           # Maximum security

class DataClassificationManager:
    def __init__(self):
        self.classification_rules = {}
        self.classified_data = {}
    
    def classify_data(self, data_asset: str, content: Dict[str, Any]) -> DataClassification:
        """Classify data based on content"""
        # Check for PII
        if self._contains_pii(content):
            return DataClassification.CONFIDENTIAL
        
        # Check for PHI
        if self._contains_phi(content):
            return DataClassification.RESTRICTED
        
        # Check for financial data
        if self._contains_financial_data(content):
            return DataClassification.CONFIDENTIAL
        
        # Default classification
        return DataClassification.INTERNAL
    
    def _contains_pii(self, content: Dict[str, Any]) -> bool:
        """Check if content contains PII"""
        pii_fields = ["ssn", "email", "phone", "address", "date_of_birth"]
        return any(field in content for field in pii_fields)
    
    def _contains_phi(self, content: Dict[str, Any]) -> bool:
        """Check if content contains PHI"""
        phi_fields = ["medical_record", "diagnosis", "treatment", "prescription"]
        return any(field in content for field in phi_fields)
    
    def _contains_financial_data(self, content: Dict[str, Any]) -> bool:
        """Check if content contains financial data"""
        financial_fields = ["credit_card", "bank_account", "transaction"]
        return any(field in content for field in financial_fields)
```

### Privacy Impact Assessment

```python
# Privacy Impact Assessment
@dataclass
class PrivacyImpactAssessment:
    assessment_id: str
    project_name: str
    data_types: List[str]
    data_subjects: List[str]
    processing_purposes: List[str]
    lawful_basis: str
    data_sharing: List[str]
    retention_period: int
    security_measures: List[str]
    risk_level: str  # "low", "medium", "high"
    mitigation_measures: List[str]
    approval_status: str  # "pending", "approved", "rejected"
    assessment_date: datetime

class PrivacyImpactAssessmentManager:
    def __init__(self):
        self.assessments = {}
    
    def create_assessment(self, assessment: PrivacyImpactAssessment):
        """Create a privacy impact assessment"""
        self.assessments[assessment.assessment_id] = assessment
    
    def assess_privacy_risk(self, assessment: PrivacyImpactAssessment) -> str:
        """Assess privacy risk level"""
        risk_score = 0
        
        # Data sensitivity
        if any(dt in ["PII", "PHI", "financial"] for dt in assessment.data_types):
            risk_score += 3
        elif any(dt in ["internal", "business"] for dt in assessment.data_types):
            risk_score += 1
        
        # Data volume
        # Assuming large volume increases risk
        risk_score += 1
        
        # Data sharing
        if len(assessment.data_sharing) > 0:
            risk_score += 2
        
        # Determine risk level
        if risk_score >= 5:
            return "high"
        elif risk_score >= 3:
            return "medium"
        else:
            return "low"
```

## ⏰ Data Retention & Lifecycle

### Retention Policies

```python
# Data Retention Framework
@dataclass
class RetentionPolicy:
    policy_id: str
    data_category: str
    retention_period_days: int
    retention_rule: str  # "legal", "business", "regulatory"
    disposal_method: str  # "delete", "archive", "anonymize"
    applicable_regulations: List[str]
    effective_date: datetime
    review_date: datetime

class DataRetentionManager:
    def __init__(self):
        self.retention_policies = {}
        self.data_lifecycle = {}
    
    def create_retention_policy(self, policy: RetentionPolicy):
        """Create a data retention policy"""
        self.retention_policies[policy.policy_id] = policy
    
    def get_retention_policy(self, data_category: str) -> Optional[RetentionPolicy]:
        """Get retention policy for data category"""
        for policy in self.retention_policies.values():
            if policy.data_category == data_category:
                return policy
        return None
    
    def check_retention_compliance(self, data_asset: str, 
                                  creation_date: datetime) -> Dict[str, Any]:
        """Check if data asset complies with retention policy"""
        # Get policy for asset
        policy = self._get_policy_for_asset(data_asset)
        
        if not policy:
            return {"status": "no_policy", "action": "review_required"}
        
        age_days = (datetime.now() - creation_date).days
        
        if age_days > policy.retention_period_days:
            return {
                "status": "retention_exceeded",
                "action": policy.disposal_method,
                "age_days": age_days,
                "retention_period": policy.retention_period_days
            }
        else:
            return {
                "status": "compliant",
                "remaining_days": policy.retention_period_days - age_days
            }
    
    def _get_policy_for_asset(self, data_asset: str) -> Optional[RetentionPolicy]:
        """Get retention policy for data asset"""
        # Implementation to map asset to policy
        return None
    
    def execute_data_disposal(self, data_asset: str, method: str):
        """Execute data disposal based on method"""
        if method == "delete":
            self._delete_data(data_asset)
        elif method == "archive":
            self._archive_data(data_asset)
        elif method == "anonymize":
            self._anonymize_data(data_asset)
    
    def _delete_data(self, data_asset: str):
        """Delete data asset"""
        # Implementation
        pass
    
    def _archive_data(self, data_asset: str):
        """Archive data asset"""
        # Implementation
        pass
    
    def _anonymize_data(self, data_asset: str):
        """Anonymize data asset"""
        # Implementation
        pass
```

## 🔐 Access Control & Classification

### Role-Based Access Control (RBAC)

```python
# Enhanced RBAC for Data Governance
from enum import Enum
from typing import Set, List

class DataRole(Enum):
    DATA_ADMIN = "data_admin"
    DATA_STEWARD = "data_steward"
    DATA_ANALYST = "data_analyst"
    DATA_SCIENTIST = "data_scientist"
    DATA_VIEWER = "data_viewer"
    EXTERNAL_USER = "external_user"

class DataPermission(Enum):
    READ = "read"
    WRITE = "write"
    DELETE = "delete"
    EXPORT = "export"
    SHARE = "share"
    ADMIN = "admin"

class DataRBAC:
    def __init__(self):
        self.role_permissions = {
            DataRole.DATA_ADMIN: {
                DataPermission.READ, DataPermission.WRITE, 
                DataPermission.DELETE, DataPermission.EXPORT,
                DataPermission.SHARE, DataPermission.ADMIN
            },
            DataRole.DATA_STEWARD: {
                DataPermission.READ, DataPermission.WRITE,
                DataPermission.EXPORT
            },
            DataRole.DATA_ANALYST: {
                DataPermission.READ, DataPermission.EXPORT
            },
            DataRole.DATA_SCIENTIST: {
                DataPermission.READ, DataPermission.WRITE,
                DataPermission.EXPORT
            },
            DataRole.DATA_VIEWER: {
                DataPermission.READ
            },
            DataRole.EXTERNAL_USER: {
                DataPermission.READ
            }
        }
        
        self.user_roles = {}
        self.asset_permissions = {}  # asset_id -> Set[DataPermission]
    
    def assign_role(self, user_id: str, role: DataRole):
        """Assign role to user"""
        if user_id not in self.user_roles:
            self.user_roles[user_id] = set()
        self.user_roles[user_id].add(role)
    
    def has_permission(self, user_id: str, asset_id: str, 
                      permission: DataPermission) -> bool:
        """Check if user has permission for asset"""
        # Check role-based permissions
        user_permissions = set()
        if user_id in self.user_roles:
            for role in self.user_roles[user_id]:
                user_permissions.update(self.role_permissions.get(role, set()))
        
        # Check asset-specific permissions
        asset_perms = self.asset_permissions.get(asset_id, set())
        
        # User needs permission from either role or asset-specific
        return permission in user_permissions or permission in asset_perms
    
    def grant_asset_permission(self, asset_id: str, permission: DataPermission):
        """Grant permission to asset"""
        if asset_id not in self.asset_permissions:
            self.asset_permissions[asset_id] = set()
        self.asset_permissions[asset_id].add(permission)
```

## 📈 Compliance Monitoring

### Compliance Dashboard

```python
# Compliance Monitoring Framework
@dataclass
class ComplianceMetric:
    metric_id: str
    name: str
    category: str  # "gdpr", "ccpa", "hipaa", "pci_dss"
    current_value: float
    target_value: float
    status: str  # "compliant", "warning", "non_compliant"
    last_updated: datetime

class ComplianceMonitor:
    def __init__(self):
        self.metrics = {}
        self.compliance_reports = []
    
    def track_metric(self, metric: ComplianceMetric):
        """Track compliance metric"""
        self.metrics[metric.metric_id] = metric
    
    def calculate_compliance_score(self, regulation: str) -> float:
        """Calculate overall compliance score for regulation"""
        relevant_metrics = [
            m for m in self.metrics.values() 
            if m.category == regulation
        ]
        
        if not relevant_metrics:
            return 0.0
        
        compliant_count = sum(
            1 for m in relevant_metrics 
            if m.status == "compliant"
        )
        
        return (compliant_count / len(relevant_metrics)) * 100
    
    def generate_compliance_report(self, regulation: str) -> Dict[str, Any]:
        """Generate compliance report"""
        score = self.calculate_compliance_score(regulation)
        relevant_metrics = [
            m for m in self.metrics.values() 
            if m.category == regulation
        ]
        
        report = {
            "regulation": regulation,
            "compliance_score": score,
            "status": "compliant" if score >= 90 else "warning" if score >= 70 else "non_compliant",
            "metrics": [
                {
                    "name": m.name,
                    "current_value": m.current_value,
                    "target_value": m.target_value,
                    "status": m.status
                }
                for m in relevant_metrics
            ],
            "generated_date": datetime.now().isoformat()
        }
        
        self.compliance_reports.append(report)
        return report
```

## 🚀 Best Practices

### Governance Best Practices

1. **Establish Clear Ownership**
   - Define data owners and stewards
   - Create RACI matrix for data assets
   - Regular governance council meetings

2. **Document Everything**
   - Maintain data dictionary
   - Document data lineage
   - Keep policy documentation current

3. **Implement Data Quality Checks**
   - Automated quality validation
   - Quality scorecards and dashboards
   - Continuous monitoring

4. **Enforce Access Controls**
   - Principle of least privilege
   - Regular access reviews
   - Audit all data access

5. **Maintain Compliance**
   - Regular compliance assessments
   - Automated compliance monitoring
   - Training and awareness programs

### Compliance Best Practices

1. **Understand Regulations**
   - Map data to applicable regulations
   - Regular regulatory updates
   - Legal and compliance consultation

2. **Implement Privacy by Design**
   - Build privacy into systems
   - Minimize data collection
   - Default privacy settings

3. **Maintain Audit Trails**
   - Log all data access
   - Track data changes
   - Retain audit logs

4. **Regular Assessments**
   - Quarterly compliance reviews
   - Annual comprehensive audits
   - Continuous improvement

5. **Incident Response**
   - Breach detection and response plan
   - Notification procedures
   - Recovery and remediation

## 🔗 Related Concepts

- [Data Security](./security/) - Security and protection mechanisms
- [Data Lakehouse](../datalakehouse/) - Modern data architecture
- [Data Mesh](../datamesh/) - Decentralized data architecture
- [Schema Change](../schema-change/) - Schema evolution and versioning
- [Design Patterns](../../design-pattern/) - Architectural patterns


