# Multi-Store Role-Based Access Control

> **RBAC Implementation for Multi-Store Pharmacy Operations**

Comprehensive access control system enabling pharmacists to own and manage multiple stores while maintaining regulatory compliance and security.

---

## Overview

The ACPN multi-store RBAC system addresses the unique challenges of pharmacy chain management where:
- One pharmacist can own multiple pharmacies
- Each pharmacy requires a qualified resident pharmacist  
- Staff can work across multiple locations
- Regulatory compliance must be maintained per store
- Audit trails are required for all cross-store activities

## Core RBAC Principles

### **Store-Level Isolation**
```yaml
Data Isolation:
  - Each pharmacy has separate data boundaries
  - Cross-store access requires explicit permissions
  - Financial data isolated per store
  - Customer data segregated by location

Access Control:
  - Role assignments are store-specific
  - Permissions granted per pharmacy
  - Cross-store roles require additional approval
  - Emergency access procedures defined
```

### **Inheritance Model**
```yaml
Owner Privileges:
  - Automatic access to all owned pharmacies
  - Can delegate management roles
  - Financial oversight across chain
  - Staff hiring/termination authority

Management Hierarchy:
  - Chain Manager: Multi-store operations
  - Store Manager: Single store operations  
  - Shift Supervisor: Time-based authority
  - Department Head: Functional area control
```

---

## Multi-Store Role Definitions

### **Chain-Level Roles**

#### **Chain Owner**
```yaml
Scope: All owned pharmacies
Primary Permissions:
  - Create/modify pharmacy details
  - Hire/terminate staff across chain
  - Financial oversight and reporting
  - Cross-store inventory management
  - Performance analytics
  - Compliance monitoring

Restrictions:
  - Cannot modify other owners' pharmacies
  - Resident pharmacist assignments require ACPN approval
  - Large financial transactions need verification
```

#### **Chain Manager**
```yaml
Scope: Designated pharmacy group
Primary Permissions:
  - Multi-store operational oversight
  - Staff scheduling across locations
  - Inventory coordination
  - Performance monitoring
  - Customer service standards
  - Policy implementation

Restrictions:
  - No ownership modification rights
  - Limited financial authority
  - Cannot hire/fire without owner approval
```

### **Store-Level Roles**

#### **Store Manager**
```yaml
Scope: Single pharmacy location
Primary Permissions:
  - Daily operations management
  - Local staff supervision
  - Customer service oversight
  - Inventory management
  - Local performance reporting
  - Shift scheduling

Restrictions:
  - Limited to assigned store
  - Cannot access other store data
  - Financial limits apply
  - Major decisions require approval
```

#### **Resident Pharmacist**
```yaml
Scope: Assigned pharmacy (regulatory requirement)
Primary Permissions:
  - Professional pharmaceutical oversight
  - Prescription verification and dispensing
  - Clinical consultations
  - Staff supervision (professional matters)
  - Regulatory compliance
  - Quality assurance

Restrictions:
  - Physical presence requirements
  - Limited to assigned pharmacy
  - Cannot delegate professional responsibilities
  - ACPN oversight and auditing
```

---

## Cross-Store Access Management

### **Access Types**

#### **Permanent Cross-Store Access**
```typescript
interface PermanentCrossAccess {
  userId: string;
  primaryStore: string;
  additionalStores: CrossStorePermission[];
  effectiveDate: Date;
  approvedBy: string;
}

interface CrossStorePermission {
  pharmacyId: string;
  role: string;
  permissions: string[];
  limitations: AccessLimitation[];
  schedule?: WorkSchedule;
}
```

#### **Temporary Cross-Store Access**
```typescript
interface TemporaryCrossAccess {
  userId: string;
  fromStore: string;
  toStore: string;
  duration: number; // days
  reason: TemporaryAccessReason;
  permissions: string[];
  approvedBy: string;
  emergencyAccess: boolean;
}

enum TemporaryAccessReason {
  STAFF_SHORTAGE = 'staff_shortage',
  EMERGENCY_COVERAGE = 'emergency_coverage',
  TRAINING_ASSIGNMENT = 'training_assignment',
  SPECIAL_PROJECT = 'special_project',
  VACATION_COVERAGE = 'vacation_coverage'
}
```

### **Permission Inheritance Rules**

```yaml
Cross-Store Permission Rules:
  
  Automatic Inheritance:
    - Owner gets full access to all owned stores
    - Basic safety and emergency permissions
    - View-only access to public information
    
  Explicit Grant Required:
    - Operational access to other stores
    - Financial data access
    - Staff management permissions
    - Customer data access
    
  Never Inherited:
    - Professional responsibilities (Resident Pharmacist)
    - Administrative system access
    - Owner-level permissions
    - Sensitive compliance data
```

---

## Implementation Architecture

### **Database Schema Design**

#### **Core RBAC Tables**
```sql
-- User-Store-Role Mapping
CREATE TABLE user_store_roles (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    pharmacy_id UUID REFERENCES pharmacies(id),
    role_id UUID REFERENCES roles(id),
    is_primary BOOLEAN DEFAULT false,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active',
    approved_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Cross-Store Access Permissions
CREATE TABLE cross_store_access (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    source_pharmacy_id UUID REFERENCES pharmacies(id),
    target_pharmacy_id UUID REFERENCES pharmacies(id),
    access_type VARCHAR(20), -- 'permanent', 'temporary', 'emergency'
    permissions JSONB,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP,
    approved_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Permission Inheritance Rules
CREATE TABLE permission_inheritance (
    id UUID PRIMARY KEY,
    parent_role_id UUID REFERENCES roles(id),
    child_role_id UUID REFERENCES roles(id),
    pharmacy_scope VARCHAR(20), -- 'same_store', 'chain_wide', 'system_wide'
    inheritance_type VARCHAR(20), -- 'automatic', 'conditional', 'explicit'
    conditions JSONB
);
```

### **API Implementation**

#### **Access Control Middleware**
```typescript
class MultiStoreRBAC {
    async checkPermission(
        userId: string, 
        pharmacyId: string, 
        permission: string
    ): Promise<boolean> {
        // 1. Get user's direct roles for this pharmacy
        const directRoles = await this.getUserStoreRoles(userId, pharmacyId);
        
        // 2. Check for inherited permissions (owner, chain manager)
        const inheritedPermissions = await this.getInheritedPermissions(userId, pharmacyId);
        
        // 3. Check for temporary cross-store access
        const temporaryAccess = await this.getTemporaryAccess(userId, pharmacyId);
        
        // 4. Evaluate permission against all sources
        return this.evaluatePermission(permission, {
            directRoles,
            inheritedPermissions,
            temporaryAccess
        });
    }
    
    async grantCrossStoreAccess(
        userId: string,
        fromPharmacy: string,
        toPharmacy: string,
        permissions: string[],
        duration?: number
    ): Promise<void> {
        // Validation and approval workflow
        await this.validateCrossStoreRequest(userId, fromPharmacy, toPharmacy);
        
        // Create access record
        await this.createCrossStoreAccess({
            userId,
            fromPharmacy,
            toPharmacy,
            permissions,
            duration,
            approvedBy: this.getCurrentUser().id
        });
        
        // Audit logging
        await this.logAccessGrant(userId, fromPharmacy, toPharmacy, permissions);
    }
}
```

---

## Workflow Examples

### **Scenario 1: Owner Managing Multiple Stores**

```yaml
User: Dr. Sarah Okafor (Pharmacy Owner)
Owns: 3 pharmacies in Lagos

Access Pattern:
  Primary Store: Lagos Island Pharmacy
    Role: Owner + Resident Pharmacist
    Permissions: Full operational control
    
  Secondary Stores: Victoria Island & Ikeja branches
    Role: Owner
    Permissions: 
      - Financial oversight
      - Staff management
      - Performance monitoring
      - Inventory coordination
    
  Restrictions:
    - Cannot be resident pharmacist at multiple locations
    - Professional duties limited to primary store
    - Must appoint qualified resident pharmacists for other stores
```

### **Scenario 2: Staff Working Across Stores**

```yaml
User: Pharm. James Adebayo (Senior Pharmacist)
Primary Assignment: Victoria Island Pharmacy

Cross-Store Access Request:
  Target: Lagos Island Pharmacy
  Duration: 2 weeks (vacation coverage)
  Reason: Resident pharmacist on leave
  
Approval Workflow:
  1. Request submitted with justification
  2. Owner approval required
  3. ACPN notification (resident pharmacist change)
  4. Temporary role assignment
  5. Audit trail maintained
  
Granted Permissions:
  - Prescription verification
  - Clinical consultations
  - Staff supervision
  - Compliance oversight
  
Restrictions:
  - Limited to professional duties
  - No financial authority
  - Regular check-ins required
  - End date strictly enforced
```

### **Scenario 3: Chain Manager Operations**

```yaml
User: Mrs. Funmi Adesanya (Chain Manager)
Scope: 5-store pharmacy chain in Abuja

Daily Operations:
  Morning Routine:
    - Check overnight reports from all stores
    - Review inventory levels across chain
    - Monitor staff attendance
    - Identify operational issues
    
  Cross-Store Activities:
    - Coordinate inventory transfers
    - Implement chain-wide policies
    - Monitor performance metrics
    - Ensure compliance standards
    
  Access Permissions:
    - Read/write: Operational data
    - Read-only: Financial summaries
    - No access: Sensitive financial details
    - No access: Other chains' data
```

---

## Security Considerations

### **Data Isolation**
```yaml
Pharmacy Data Boundaries:
  Financial Data:
    - Separate encryption keys per store
    - Access logging for all financial queries
    - Regular access pattern analysis
    
  Customer Data:
    - Store-specific customer records
    - Cross-store customer linking requires consent
    - Privacy compliance per location
    
  Staff Data:
    - Employment records per store
    - Performance data isolated
    - Salary information restricted to owners
```

### **Audit Requirements**
```yaml
Cross-Store Activities:
  Required Logging:
    - All cross-store data access
    - Permission grants and revocations
    - Role changes and assignments
    - Emergency access usage
    
  Monitoring Alerts:
    - Unusual cross-store access patterns
    - Multiple simultaneous store access
    - Extended cross-store sessions
    - Permission escalation attempts
    
  Compliance Reporting:
    - Monthly cross-store access reports
    - Quarterly permission audits
    - Annual security assessments
    - Regulatory compliance checks
```

---

## Troubleshooting

### **Common Issues**

#### **Access Denied Errors**
```yaml
Symptom: User cannot access second store
Diagnosis Steps:
  1. Verify user has valid cross-store permission
  2. Check permission expiration dates
  3. Confirm store ownership/management chain
  4. Review recent permission changes
  
Resolution:
  - Grant explicit cross-store access
  - Extend permission duration
  - Update role assignments
  - Contact system administrator
```

#### **Permission Inheritance Problems**
```yaml
Symptom: Owner cannot access owned pharmacy
Diagnosis Steps:
  1. Verify ownership records in database
  2. Check role assignment accuracy
  3. Review permission inheritance rules
  4. Confirm user account status
  
Resolution:
  - Update ownership records
  - Reassign owner role
  - Refresh permission cache
  - Manual permission grant (temporary)
```

---

## Related Documentation

- [User Roles & Permissions](roles-permissions.md)
- [Multi-Store Management](../pharmacy/multi-store.md)
- [Security Architecture](../architecture/security.md)
- [Authentication Systems](jwt.md)

This multi-store RBAC system ensures secure, compliant, and efficient management of pharmacy chains while maintaining regulatory requirements and audit trails.