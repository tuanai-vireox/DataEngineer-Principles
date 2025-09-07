# Data Security and Governance

Data security and governance are critical components of modern data architecture, ensuring that data is protected, compliant, and properly managed throughout its lifecycle. This comprehensive approach covers data protection, access control, compliance, and governance frameworks.

## 🛡️ Data Security Framework

### Core Security Principles

1. **Confidentiality**: Protect data from unauthorized access
2. **Integrity**: Ensure data accuracy and consistency
3. **Availability**: Maintain data accessibility for authorized users
4. **Accountability**: Track and audit data access and modifications
5. **Compliance**: Meet regulatory and industry requirements

### Security Layers

#### 1. Network Security
- **Firewalls**: Control network traffic and access
- **VPN**: Secure remote access to data systems
- **Network Segmentation**: Isolate sensitive data systems
- **DDoS Protection**: Protect against distributed denial-of-service attacks

#### 2. Application Security
- **Authentication**: Verify user identity
- **Authorization**: Control user access to resources
- **Input Validation**: Prevent injection attacks
- **Secure APIs**: Protect API endpoints and data

#### 3. Data Security
- **Encryption**: Protect data at rest and in transit
- **Data Masking**: Hide sensitive data in non-production environments
- **Tokenization**: Replace sensitive data with tokens
- **Data Loss Prevention**: Monitor and prevent data exfiltration

#### 4. Infrastructure Security
- **Access Controls**: Restrict physical and logical access
- **Monitoring**: Continuous security monitoring
- **Vulnerability Management**: Regular security assessments
- **Incident Response**: Plan for security incidents

## 🔐 Data Protection Strategies

### Encryption

#### Data at Rest
- **Database Encryption**: Transparent Data Encryption (TDE)
- **File System Encryption**: Encrypt storage volumes
- **Object Storage Encryption**: S3, Azure Blob, GCS encryption
- **Key Management**: Centralized key management systems

#### Data in Transit
- **TLS/SSL**: Secure communication protocols
- **VPN**: Encrypted network connections
- **API Security**: OAuth, JWT tokens
- **Message Encryption**: Encrypt data in message queues

#### Implementation Example
```python
# Example: Data encryption with AWS KMS
import boto3
from cryptography.fernet import Fernet
import base64

class DataEncryption:
    def __init__(self, kms_key_id):
        self.kms_client = boto3.client('kms')
        self.kms_key_id = kms_key_id
    
    def encrypt_data(self, plaintext):
        # Generate data key
        response = self.kms_client.generate_data_key(
            KeyId=self.kms_key_id,
            KeySpec='AES_256'
        )
        
        # Encrypt data
        cipher = Fernet(base64.b64encode(response['Plaintext']))
        encrypted_data = cipher.encrypt(plaintext.encode())
        
        return {
            'encrypted_data': encrypted_data,
            'encrypted_key': response['CiphertextBlob']
        }
    
    def decrypt_data(self, encrypted_data, encrypted_key):
        # Decrypt data key
        response = self.kms_client.decrypt(
            CiphertextBlob=encrypted_key
        )
        
        # Decrypt data
        cipher = Fernet(base64.b64encode(response['Plaintext']))
        decrypted_data = cipher.decrypt(encrypted_data)
        
        return decrypted_data.decode()
```

### Data Masking and Anonymization

#### Static Data Masking
- **Production Data**: Mask sensitive data in production
- **Test Data**: Create masked copies for testing
- **Development Data**: Provide safe data for development

#### Dynamic Data Masking
- **Real-time Masking**: Mask data during query execution
- **Role-based Masking**: Different masking based on user roles
- **Context-aware Masking**: Mask based on query context

#### Implementation Example
```python
# Example: Data masking implementation
import hashlib
import re
from typing import Any, Dict

class DataMasker:
    def __init__(self):
        self.masking_rules = {
            'email': self.mask_email,
            'phone': self.mask_phone,
            'ssn': self.mask_ssn,
            'credit_card': self.mask_credit_card
        }
    
    def mask_email(self, email: str) -> str:
        if '@' in email:
            local, domain = email.split('@', 1)
            masked_local = local[0] + '*' * (len(local) - 2) + local[-1]
            return f"{masked_local}@{domain}"
        return email
    
    def mask_phone(self, phone: str) -> str:
        # Remove non-digits
        digits = re.sub(r'\D', '', phone)
        if len(digits) == 10:
            return f"***-***-{digits[-4:]}"
        return phone
    
    def mask_ssn(self, ssn: str) -> str:
        digits = re.sub(r'\D', '', ssn)
        if len(digits) == 9:
            return f"***-**-{digits[-4:]}"
        return ssn
    
    def mask_credit_card(self, card: str) -> str:
        digits = re.sub(r'\D', '', card)
        if len(digits) >= 4:
            return f"****-****-****-{digits[-4:]}"
        return card
    
    def mask_data(self, data: Dict[str, Any], fields_to_mask: list) -> Dict[str, Any]:
        masked_data = data.copy()
        for field in fields_to_mask:
            if field in masked_data and field in self.masking_rules:
                masked_data[field] = self.masking_rules[field](str(masked_data[field]))
        return masked_data
```

### Tokenization

#### Benefits
- **Data Protection**: Replace sensitive data with tokens
- **Compliance**: Meet PCI DSS, GDPR requirements
- **Analytics**: Enable analytics on tokenized data
- **Reversibility**: Tokens can be reversed when needed

#### Implementation
```python
# Example: Tokenization implementation
import uuid
import hashlib
from typing import Dict, Any

class Tokenizer:
    def __init__(self, secret_key: str):
        self.secret_key = secret_key
        self.token_map = {}
    
    def tokenize(self, sensitive_data: str) -> str:
        # Create deterministic token
        token_input = f"{sensitive_data}{self.secret_key}"
        token = hashlib.sha256(token_input.encode()).hexdigest()[:16]
        
        # Store mapping for detokenization
        self.token_map[token] = sensitive_data
        
        return token
    
    def detokenize(self, token: str) -> str:
        return self.token_map.get(token, token)
    
    def tokenize_record(self, record: Dict[str, Any], sensitive_fields: list) -> Dict[str, Any]:
        tokenized_record = record.copy()
        for field in sensitive_fields:
            if field in tokenized_record:
                tokenized_record[field] = self.tokenize(str(tokenized_record[field]))
        return tokenized_record
```

## 👥 Access Control and Identity Management

### Authentication Methods

#### Multi-Factor Authentication (MFA)
- **Something you know**: Password, PIN
- **Something you have**: Token, mobile device
- **Something you are**: Biometric authentication

#### Single Sign-On (SSO)
- **SAML**: Security Assertion Markup Language
- **OAuth 2.0**: Authorization framework
- **OpenID Connect**: Identity layer on OAuth 2.0
- **LDAP**: Lightweight Directory Access Protocol

### Authorization Models

#### Role-Based Access Control (RBAC)
```python
# Example: RBAC implementation
from enum import Enum
from typing import List, Set

class Role(Enum):
    ADMIN = "admin"
    ANALYST = "analyst"
    VIEWER = "viewer"
    DATA_SCIENTIST = "data_scientist"

class Permission(Enum):
    READ = "read"
    WRITE = "write"
    DELETE = "delete"
    ADMIN = "admin"

class RBAC:
    def __init__(self):
        self.role_permissions = {
            Role.ADMIN: {Permission.READ, Permission.WRITE, Permission.DELETE, Permission.ADMIN},
            Role.ANALYST: {Permission.READ, Permission.WRITE},
            Role.DATA_SCIENTIST: {Permission.READ, Permission.WRITE},
            Role.VIEWER: {Permission.READ}
        }
        self.user_roles = {}
    
    def assign_role(self, user_id: str, role: Role):
        if user_id not in self.user_roles:
            self.user_roles[user_id] = set()
        self.user_roles[user_id].add(role)
    
    def has_permission(self, user_id: str, permission: Permission) -> bool:
        if user_id not in self.user_roles:
            return False
        
        user_permissions = set()
        for role in self.user_roles[user_id]:
            user_permissions.update(self.role_permissions.get(role, set()))
        
        return permission in user_permissions
```

#### Attribute-Based Access Control (ABAC)
- **Attributes**: User, resource, environment attributes
- **Policies**: Rule-based access decisions
- **Context**: Dynamic access control based on context
- **Flexibility**: Fine-grained access control

### Data Access Controls

#### Row-Level Security (RLS)
```sql
-- Example: Row-level security in PostgreSQL
CREATE POLICY user_data_policy ON user_data
    FOR ALL TO authenticated_users
    USING (user_id = current_user_id());

-- Enable RLS on table
ALTER TABLE user_data ENABLE ROW LEVEL SECURITY;
```

#### Column-Level Security
```sql
-- Example: Column-level security
CREATE VIEW user_data_secure AS
SELECT 
    user_id,
    CASE 
        WHEN current_user_role() = 'admin' THEN email
        ELSE '***@***.***'
    END as email,
    name,
    created_at
FROM user_data;
```

## 📋 Data Governance Framework

### Governance Components

#### 1. Data Stewardship
- **Data Owners**: Business owners of data
- **Data Stewards**: Technical custodians of data
- **Data Custodians**: IT teams managing data systems
- **Data Users**: End users consuming data

#### 2. Data Quality Management
- **Data Profiling**: Understand data characteristics
- **Data Validation**: Ensure data accuracy and completeness
- **Data Cleansing**: Fix data quality issues
- **Data Monitoring**: Continuous quality monitoring

#### 3. Data Lineage
- **Source Tracking**: Track data from source to destination
- **Transformation Tracking**: Monitor data transformations
- **Impact Analysis**: Understand impact of changes
- **Compliance**: Meet regulatory requirements

#### 4. Metadata Management
- **Data Catalog**: Centralized metadata repository
- **Business Glossary**: Common business terms
- **Technical Metadata**: Schema, structure, format
- **Operational Metadata**: Usage, performance, quality

### Implementation Example
```python
# Example: Data governance framework
from dataclasses import dataclass
from typing import List, Dict, Any
from datetime import datetime
import json

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

@dataclass
class DataLineage:
    source_system: str
    target_system: str
    transformation: str
    timestamp: datetime
    user: str

class DataGovernance:
    def __init__(self):
        self.data_assets = {}
        self.data_lineage = []
        self.data_quality_rules = {}
    
    def register_data_asset(self, asset: DataAsset):
        self.data_assets[asset.name] = asset
    
    def add_lineage(self, lineage: DataLineage):
        self.data_lineage.append(lineage)
    
    def get_data_asset(self, name: str) -> DataAsset:
        return self.data_assets.get(name)
    
    def get_lineage(self, asset_name: str) -> List[DataLineage]:
        return [lineage for lineage in self.data_lineage 
                if lineage.source_system == asset_name or lineage.target_system == asset_name]
    
    def validate_data_quality(self, data: List[Dict], rules: Dict[str, Any]) -> Dict[str, Any]:
        results = {
            'total_records': len(data),
            'valid_records': 0,
            'invalid_records': 0,
            'quality_score': 0.0,
            'issues': []
        }
        
        for record in data:
            is_valid = True
            for field, rule in rules.items():
                if field in record:
                    if not self._validate_field(record[field], rule):
                        is_valid = False
                        results['issues'].append(f"Field {field} failed validation")
            
            if is_valid:
                results['valid_records'] += 1
            else:
                results['invalid_records'] += 1
        
        results['quality_score'] = results['valid_records'] / results['total_records']
        return results
    
    def _validate_field(self, value: Any, rule: Dict[str, Any]) -> bool:
        if 'required' in rule and rule['required'] and value is None:
            return False
        
        if 'type' in rule:
            if rule['type'] == 'string' and not isinstance(value, str):
                return False
            elif rule['type'] == 'number' and not isinstance(value, (int, float)):
                return False
        
        if 'pattern' in rule and isinstance(value, str):
            import re
            if not re.match(rule['pattern'], value):
                return False
        
        return True
```

## 📊 Compliance and Regulations

### Key Regulations

#### GDPR (General Data Protection Regulation)
- **Data Subject Rights**: Right to access, rectification, erasure
- **Consent Management**: Explicit consent for data processing
- **Data Portability**: Right to data portability
- **Privacy by Design**: Build privacy into systems

#### CCPA (California Consumer Privacy Act)
- **Consumer Rights**: Right to know, delete, opt-out
- **Data Categories**: Personal information categories
- **Business Obligations**: Disclosure and compliance requirements
- **Enforcement**: Civil penalties and enforcement

#### HIPAA (Health Insurance Portability and Accountability Act)
- **Protected Health Information**: PHI protection requirements
- **Administrative Safeguards**: Policies and procedures
- **Physical Safeguards**: Physical access controls
- **Technical Safeguards**: Technical security measures

#### PCI DSS (Payment Card Industry Data Security Standard)
- **Cardholder Data**: Protection of payment card data
- **Security Requirements**: 12 security requirements
- **Compliance Validation**: Regular compliance assessments
- **Incident Response**: Breach response procedures

### Compliance Implementation
```python
# Example: GDPR compliance framework
from typing import Dict, List, Any
from datetime import datetime, timedelta
import json

class GDPRCompliance:
    def __init__(self):
        self.consent_records = {}
        self.data_processing_records = {}
        self.data_retention_policies = {}
    
    def record_consent(self, user_id: str, purpose: str, consent_given: bool):
        if user_id not in self.consent_records:
            self.consent_records[user_id] = {}
        
        self.consent_records[user_id][purpose] = {
            'consent_given': consent_given,
            'timestamp': datetime.now(),
            'purpose': purpose
        }
    
    def has_valid_consent(self, user_id: str, purpose: str) -> bool:
        if user_id not in self.consent_records:
            return False
        
        consent_record = self.consent_records[user_id].get(purpose)
        if not consent_record:
            return False
        
        # Check if consent is still valid (e.g., not expired)
        consent_age = datetime.now() - consent_record['timestamp']
        if consent_age > timedelta(days=365):  # Consent expires after 1 year
            return False
        
        return consent_record['consent_given']
    
    def process_data_subject_request(self, user_id: str, request_type: str) -> Dict[str, Any]:
        if request_type == 'access':
            return self._handle_access_request(user_id)
        elif request_type == 'rectification':
            return self._handle_rectification_request(user_id)
        elif request_type == 'erasure':
            return self._handle_erasure_request(user_id)
        elif request_type == 'portability':
            return self._handle_portability_request(user_id)
        else:
            return {'error': 'Invalid request type'}
    
    def _handle_access_request(self, user_id: str) -> Dict[str, Any]:
        # Return all personal data for the user
        return {
            'user_id': user_id,
            'personal_data': self._get_user_data(user_id),
            'processing_purposes': self._get_processing_purposes(user_id),
            'data_retention': self._get_retention_info(user_id)
        }
    
    def _handle_erasure_request(self, user_id: str) -> Dict[str, Any]:
        # Delete user data (right to be forgotten)
        deleted_data = self._delete_user_data(user_id)
        return {
            'user_id': user_id,
            'deleted_data': deleted_data,
            'timestamp': datetime.now()
        }
```

## 🔍 Monitoring and Auditing

### Security Monitoring

#### Log Management
- **Centralized Logging**: Aggregate logs from all systems
- **Log Analysis**: Analyze logs for security events
- **Real-time Monitoring**: Monitor for security incidents
- **Log Retention**: Maintain logs for compliance

#### Security Information and Event Management (SIEM)
- **Event Correlation**: Correlate events across systems
- **Threat Detection**: Detect security threats
- **Incident Response**: Automated incident response
- **Compliance Reporting**: Generate compliance reports

### Audit Trail
```python
# Example: Audit trail implementation
from typing import Dict, Any, List
from datetime import datetime
import json

class AuditTrail:
    def __init__(self):
        self.audit_logs = []
    
    def log_event(self, user_id: str, action: str, resource: str, 
                  details: Dict[str, Any] = None):
        audit_event = {
            'timestamp': datetime.now().isoformat(),
            'user_id': user_id,
            'action': action,
            'resource': resource,
            'details': details or {},
            'ip_address': self._get_client_ip(),
            'user_agent': self._get_user_agent()
        }
        
        self.audit_logs.append(audit_event)
        self._persist_audit_event(audit_event)
    
    def get_audit_logs(self, user_id: str = None, action: str = None, 
                      start_date: datetime = None, end_date: datetime = None) -> List[Dict]:
        filtered_logs = self.audit_logs
        
        if user_id:
            filtered_logs = [log for log in filtered_logs if log['user_id'] == user_id]
        
        if action:
            filtered_logs = [log for log in filtered_logs if log['action'] == action]
        
        if start_date:
            filtered_logs = [log for log in filtered_logs 
                           if datetime.fromisoformat(log['timestamp']) >= start_date]
        
        if end_date:
            filtered_logs = [log for log in filtered_logs 
                           if datetime.fromisoformat(log['timestamp']) <= end_date]
        
        return filtered_logs
    
    def _persist_audit_event(self, event: Dict[str, Any]):
        # Persist to database or log file
        with open('audit.log', 'a') as f:
            f.write(json.dumps(event) + '\n')
    
    def _get_client_ip(self) -> str:
        # Implementation to get client IP
        return "127.0.0.1"
    
    def _get_user_agent(self) -> str:
        # Implementation to get user agent
        return "Unknown"
```

## 🚀 Best Practices

### Security Best Practices
1. **Defense in Depth**: Multiple layers of security
2. **Least Privilege**: Minimum necessary access
3. **Regular Updates**: Keep systems and software updated
4. **Security Training**: Educate users on security practices
5. **Incident Response**: Plan for security incidents

### Governance Best Practices
1. **Clear Ownership**: Define data ownership and stewardship
2. **Documentation**: Maintain comprehensive documentation
3. **Regular Reviews**: Periodic governance reviews
4. **Training**: Train staff on governance practices
5. **Continuous Improvement**: Evolve governance practices

### Compliance Best Practices
1. **Understand Requirements**: Know applicable regulations
2. **Implement Controls**: Put appropriate controls in place
3. **Monitor Compliance**: Continuous compliance monitoring
4. **Document Everything**: Maintain compliance documentation
5. **Regular Assessments**: Periodic compliance assessments

## 🔗 Related Concepts

- [Data Lake](../datalake/) - Raw data storage and processing
- [Data Warehouse](../datawarehouse/) - Structured analytics storage
- [Data Mart](../datamart/) - Department-specific data subsets
- [OLAP vs OLTP](../OLAP/) - Analytical vs transactional processing
- [Design Patterns](../../design-pattern/) - Common architectural patterns
