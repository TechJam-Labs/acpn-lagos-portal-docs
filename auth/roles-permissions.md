# User Roles & Permissions

> **Comprehensive Role-Based Access Control for ACPN Lagos Portal**

Complete permission matrix and role hierarchy supporting multi-store pharmacy operations, resident pharmacist requirements, and ACPN administrative oversight.

---

## Overview

The ACPN system implements a sophisticated role-based access control (RBAC) system that accommodates the complex hierarchy of pharmaceutical practice in Nigeria. The system supports multi-store ownership, regulatory compliance, and hierarchical management while maintaining security and audit requirements.

## Role Hierarchy & Structure

### **Primary Role Categories**

#### **1. ACPN Administrative Roles**
- **Super Admin** - Complete system control and oversight
- **ACPN Admin** - Regional and operational management
- **Compliance Officer** - Regulatory and audit oversight
- **Support Specialist** - User assistance and technical support

#### **2. Pharmacy Ownership Roles**
- **Pharmacy Owner** - Business ownership and management
- **Co-Owner** - Shared ownership and responsibilities
- **Chain Manager** - Multi-store operation oversight

#### **3. Professional Practice Roles**
- **Resident Pharmacist** - Licensed pharmacy oversight
- **Relief Pharmacist** - Temporary professional coverage
- **Senior Pharmacist** - Advanced practice and supervision
- **Junior Pharmacist** - Standard pharmaceutical practice

#### **4. Operational Roles**
- **Pharmacy Manager** - Daily operations management
- **Shift Supervisor** - Shift-level supervision
- **Pharmacy Technician** - Technical pharmaceutical support
- **Pharmacy Assistant** - General pharmacy assistance

#### **5. Support Roles**
- **Cashier** - Transaction processing
- **Inventory Clerk** - Stock management
- **Customer Service** - Client interaction
- **Security Personnel** - Safety and security

#### **6. Training & Development Roles**
- **Intern** - Student pharmacist training
- **Trainee** - Skills development programs
- **Mentor** - Training supervision

---

## Detailed Role Definitions

### **ACPN Administrative Roles**

#### **Super Admin**
```yaml
Role: Super Admin
Code: SUPER_ADMIN
Level: 1 (Highest Authority)
Scope: System-wide

Core Responsibilities:
  - Complete system administration
  - User role management
  - System configuration
  - Security oversight
  - Audit trail management
  - Regulatory compliance enforcement

Key Permissions:
  - Create/modify/delete any user account
  - Assign/revoke any role or permission
  - Access all pharmacy records
  - System configuration changes
  - Database administration
  - Security policy management
  - Generate compliance reports
  - Override system restrictions

Access Restrictions:
  - IP whitelist enforced
  - MFA mandatory
  - Session timeout: 15 minutes
  - All actions logged and monitored

Eligibility Requirements:
  - ACPN Board appointment
  - Advanced cybersecurity training
  - Background security clearance
  - Minimum 10 years pharmaceutical experience
```

#### **ACPN Admin**
```yaml
Role: ACPN Admin
Code: ACPN_ADMIN
Level: 2
Scope: Regional or functional area

Core Responsibilities:
  - Regional pharmacy oversight
  - User verification and approval
  - Compliance monitoring
  - Report generation
  - Educational program management

Key Permissions:
  - Verify pharmacy registrations
  - Approve resident pharmacist assignments
  - Access regional pharmacy data
  - Generate compliance reports
  - Manage CPD programs
  - User support and assistance

Access Restrictions:
  - Regional scope limitations
  - MFA mandatory
  - Session timeout: 30 minutes
  - Supervisor approval for sensitive actions

Eligibility Requirements:
  - ACPN staff appointment
  - Pharmacy degree required
  - 5+ years regulatory experience
  - Compliance training completion
```

### **Pharmacy Ownership Roles**

#### **Pharmacy Owner**
```yaml
Role: Pharmacy Owner
Code: PHARMACY_OWNER
Level: 3
Scope: Owned pharmacies only

Core Responsibilities:
  - Business operations management
  - Staff hiring and management
  - Financial oversight
  - Regulatory compliance
  - Customer service standards

Key Permissions:
  - Manage owned pharmacy details
  - Hire/terminate staff
  - Assign roles (within restrictions)
  - Access financial reports
  - Inventory management
  - Customer data access
  - Order management
  - Performance analytics

Access Restrictions:
  - Limited to owned pharmacies
  - Cannot modify critical system settings
  - Staff role assignments require verification
  - Financial transactions require approval

Eligibility Requirements:
  - Valid pharmacy license
  - Business registration documents
  - PCN registration
  - Background verification
  - Financial capability assessment
```

#### **Chain Manager**
```yaml
Role: Chain Manager
Code: CHAIN_MANAGER
Level: 3
Scope: Pharmacy chain

Core Responsibilities:
  - Multi-store operations coordination
  - Chain-wide policy implementation
  - Performance monitoring
  - Resource allocation
  - Staff coordination across stores

Key Permissions:
  - Access all chain pharmacy data
  - Cross-store staff management
  - Chain-wide inventory oversight
  - Performance analytics
  - Policy implementation
  - Resource allocation

Access Restrictions:
  - Limited to designated pharmacy chain
  - Cannot modify ownership structures
  - Major decisions require owner approval

Eligibility Requirements:
  - Pharmacy degree
  - 5+ years management experience
  - Chain owner appointment
  - Management training completion
```

### **Professional Practice Roles**

#### **Resident Pharmacist**
```yaml
Role: Resident Pharmacist
Code: RESIDENT_PHARMACIST
Level: 4
Scope: Assigned pharmacy

Core Responsibilities:
  - Professional pharmaceutical oversight
  - Prescription verification
  - Clinical consultation
  - Regulatory compliance
  - Staff supervision
  - Quality assurance

Key Permissions:
  - Prescription management
  - Clinical consultations
  - Drug information provision
  - Staff supervision
  - Compliance monitoring
  - Inventory oversight
  - Customer counseling
  - Educational activities

Access Restrictions:
  - Limited to assigned pharmacy
  - Cannot modify ownership information
  - Financial access restricted
  - Must maintain physical presence requirements

Eligibility Requirements:
  - Valid PCN registration
  - Pharmacy degree
  - 2+ years experience
  - Clean disciplinary record
  - Continuing education compliance
  - Physical presence commitment
```

#### **Senior Pharmacist**
```yaml
Role: Senior Pharmacist
Code: SENIOR_PHARMACIST
Level: 5
Scope: Assigned pharmacy

Core Responsibilities:
  - Advanced pharmaceutical care
  - Junior staff mentoring
  - Specialized consultations
  - Research activities
  - Quality improvement

Key Permissions:
  - Complex prescription handling
  - Clinical decision making
  - Staff training
  - Research data access
  - Quality metrics review
  - Specialized consultations

Access Restrictions:
  - Assigned pharmacy scope
  - Supervision requirements met
  - Regular performance reviews

Eligibility Requirements:
  - Valid PCN registration
  - Advanced pharmacy qualifications
  - 5+ years experience
  - Specialization certification
  - Mentoring training
```

### **Operational Roles**

#### **Pharmacy Manager**
```yaml
Role: Pharmacy Manager
Code: PHARMACY_MANAGER
Level: 6
Scope: Assigned pharmacy

Core Responsibilities:
  - Daily operations management
  - Staff scheduling
  - Customer service oversight
  - Inventory coordination
  - Performance monitoring

Key Permissions:
  - Staff schedule management
  - Inventory ordering
  - Customer service oversight
  - Performance reporting
  - Vendor coordination
  - Basic financial reports

Access Restrictions:
  - Operational functions only
  - No prescription authority
  - Limited financial access
  - Requires pharmacist oversight

Eligibility Requirements:
  - Management experience
  - Customer service training
  - Inventory management certification
  - Leadership skills assessment
```

---

## Permission Matrix

### **System Administration Permissions**

| Permission | Super Admin | ACPN Admin | Owner | Resident | Manager | Staff |
|------------|-------------|------------|-------|----------|---------|-------|
| **User Management** |
| Create User Account | ✅ | ✅ | ⚠️¹ | ❌ | ❌ | ❌ |
| Modify User Roles | ✅ | ✅ | ⚠️² | ❌ | ❌ | ❌ |
| Delete User Account | ✅ | ⚠️³ | ⚠️⁴ | ❌ | ❌ | ❌ |
| View User Details | ✅ | ✅ | ⚠️⁵ | ⚠️⁶ | ⚠️⁶ | ⚠️⁷ |
| **System Configuration** |
| Modify System Settings | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Configure Integrations | ✅ | ⚠️⁸ | ❌ | ❌ | ❌ | ❌ |
| Manage API Keys | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Security & Audit** |
| View Audit Logs | ✅ | ✅ | ⚠️⁹ | ❌ | ❌ | ❌ |
| Security Monitoring | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Generate Compliance Reports | ✅ | ✅ | ⚠️¹⁰ | ❌ | ❌ | ❌ |

### **Pharmacy Management Permissions**

| Permission | Super Admin | ACPN Admin | Owner | Resident | Manager | Staff |
|------------|-------------|------------|-------|----------|---------|-------|
| **Pharmacy Operations** |
| Create Pharmacy | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Modify Pharmacy Details | ✅ | ✅ | ✅ | ⚠️¹¹ | ⚠️¹² | ❌ |
| Delete Pharmacy | ✅ | ⚠️¹³ | ❌ | ❌ | ❌ | ❌ |
| View Pharmacy Data | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️¹⁴ |
| **Staff Management** |
| Hire Staff | ✅ | ❌ | ✅ | ⚠️¹⁵ | ⚠️¹⁶ | ❌ |
| Terminate Staff | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| Assign Roles | ✅ | ✅ | ⚠️¹⁷ | ❌ | ❌ | ❌ |
| Schedule Management | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ |

### **Clinical & Professional Permissions**

| Permission | Super Admin | ACPN Admin | Owner | Resident | Senior Pharm | Junior Pharm |
|------------|-------------|------------|-------|----------|--------------|--------------|
| **Prescription Management** |
| Verify Prescriptions | ✅ | ❌ | ⚠️¹⁸ | ✅ | ✅ | ⚠️¹⁹ |
| Dispense Medications | ✅ | ❌ | ⚠️¹⁸ | ✅ | ✅ | ⚠️²⁰ |
| Clinical Consultations | ❌ | ❌ | ⚠️¹⁸ | ✅ | ✅ | ⚠️²¹ |
| **Drug Information** |
| Access Drug Database | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Update Drug Information | ✅ | ✅ | ❌ | ⚠️²² | ⚠️²² | ❌ |
| Clinical Decision Support | ✅ | ❌ | ❌ | ✅ | ✅ | ⚠️²³ |

### **Financial & Business Permissions**

| Permission | Super Admin | ACPN Admin | Owner | Manager | Cashier | Staff |
|------------|-------------|------------|-------|---------|---------|-------|
| **Financial Management** |
| View Revenue Reports | ✅ | ⚠️²⁴ | ✅ | ⚠️²⁵ | ❌ | ❌ |
| Process Payments | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Refund Management | ✅ | ❌ | ✅ | ✅ | ⚠️²⁶ | ❌ |
| **Inventory Management** |
| Order Stock | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Stock Adjustments | ✅ | ❌ | ✅ | ✅ | ✅ | ⚠️²⁷ |
| Supplier Management | ✅ | ❌ | ✅ | ⚠️²⁸ | ❌ | ❌ |

---

## Permission Footnotes

¹ Owner can create staff accounts for their pharmacies only  
² Owner can assign limited roles (staff-level only)  
³ ACPN Admin can delete with supervisor approval  
⁴ Owner can delete staff accounts only  
⁵ Owner can view staff details in their pharmacies  
⁶ Limited to same pharmacy staff  
⁷ Can view basic contact information only  
⁸ ACPN Admin can configure regional integrations  
⁹ Owner can view audit logs for their pharmacies  
¹⁰ Owner can generate basic compliance reports  
¹¹ Resident can update operational details only  
¹² Manager can update basic operational information  
¹³ ACPN Admin requires board approval for deletion  
¹⁴ Staff can view limited operational data  
¹⁵ Resident can recommend staff hiring  
¹⁶ Manager can recommend hiring with approval  
¹⁷ Owner limited to non-professional roles  
¹⁸ If owner is also a licensed pharmacist  
¹⁹ Junior pharmacist with supervision  
²⁰ Junior pharmacist with resident oversight  
²¹ Junior pharmacist for basic consultations  
²² Professional staff can suggest updates  
²³ Junior pharmacist with guidance  
²⁴ ACPN Admin can view for compliance purposes  
²⁵ Manager can view operational reports  
²⁶ Cashier can process approved refunds  
²⁷ Staff can adjust with supervisor approval  
²⁸ Manager can coordinate with existing suppliers  

---

## Role Assignment Rules

### **Assignment Constraints**

#### **Mutual Exclusivity Rules**
```yaml
Cannot Hold Simultaneously:
  - Pharmacy Owner + Staff Employee (same pharmacy)
  - Resident Pharmacist + Relief Pharmacist (same period)
  - ACPN Admin + Pharmacy Owner
  - Super Admin + Any pharmacy role

Conditional Assignments:
  - Owner can be Resident Pharmacist (if qualified)
  - Pharmacist can work multiple stores (with approvals)
  - Manager role requires owner approval
```

#### **Prerequisite Requirements**
```yaml
Senior Pharmacist:
  Prerequisites:
    - Valid PCN registration
    - 5+ years experience
    - Advanced qualifications
    - Performance evaluation

Chain Manager:
  Prerequisites:
    - Management experience
    - Owner appointment
    - Multiple store operations knowledge
    - Leadership training

Resident Pharmacist:
  Prerequisites:
    - Current PCN registration
    - Clean disciplinary record
    - Physical presence commitment
    - Continuing education compliance
```

#### **Approval Workflows**
```yaml
Role Assignment Approval:
  Level 1 (Auto-approved):
    - Basic staff roles
    - Non-critical operational roles
    
  Level 2 (Manager approval):
    - Professional roles
    - Supervisory positions
    - Cross-store assignments
    
  Level 3 (Owner approval):
    - Management roles
    - Financial access roles
    - Multi-store access
    
  Level 4 (ACPN approval):
    - Resident pharmacist assignment
    - Professional certifications
    - Compliance roles
```

---

## Dynamic Permission System

### **Context-Aware Permissions**

#### **Time-Based Permissions**
```typescript
interface TimeBasedPermission {
  permission: string;
  allowedHours: TimeRange[];
  timezone: string;
  exceptions: ExceptionRule[];
}

// Example: Prescription verification only during business hours
const prescriptionPermission: TimeBasedPermission = {
  permission: 'verify_prescriptions',
  allowedHours: [
    { start: '08:00', end: '20:00', days: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'] },
    { start: '09:00', end: '18:00', days: ['Saturday'] },
    { start: '10:00', end: '16:00', days: ['Sunday'] }
  ],
  timezone: 'Africa/Lagos',
  exceptions: [
    { type: 'emergency', override: true },
    { type: 'holiday', restrict: true }
  ]
};
```

#### **Location-Based Permissions**
```typescript
interface LocationBasedPermission {
  permission: string;
  allowedLocations: LocationRule[];
  proximityRequired: boolean;
  gpsVerification: boolean;
}

// Example: Inventory management requires physical presence
const inventoryPermission: LocationBasedPermission = {
  permission: 'manage_inventory',
  allowedLocations: [
    { type: 'pharmacy_premises', radius: 50 }, // 50 meters
    { type: 'authorized_warehouse', radius: 100 }
  ],
  proximityRequired: true,
  gpsVerification: true
};
```

#### **Value-Based Permissions**
```typescript
interface ValueBasedPermission {
  permission: string;
  thresholds: ThresholdRule[];
  approvalRequired: boolean;
  escalationRules: EscalationRule[];
}

// Example: Financial transactions have value limits
const paymentPermission: ValueBasedPermission = {
  permission: 'process_payment',
  thresholds: [
    { role: 'cashier', maxAmount: 50000, currency: 'NGN' },
    { role: 'manager', maxAmount: 200000, currency: 'NGN' },
    { role: 'owner', maxAmount: 1000000, currency: 'NGN' }
  ],
  approvalRequired: true,
  escalationRules: [
    { amount: 500000, requiresApproval: 'manager' },
    { amount: 1000000, requiresApproval: 'owner' }
  ]
};
```

---

## Emergency Access Procedures

### **Emergency Override System**
```yaml
Emergency Scenarios:
  - Medical emergencies requiring immediate prescription access
  - System failures preventing normal operations
  - Security incidents requiring elevated access
  - Regulatory compliance urgent requirements

Override Permissions:
  - Temporary elevated access (max 24 hours)
  - Detailed audit logging required
  - Automatic notification to supervisors
  - Post-incident review mandatory

Approval Process:
  1. Emergency declaration with justification
  2. Automatic supervisor notification
  3. Time-limited access grant
  4. Real-time monitoring of activities
  5. Post-emergency access review
  6. Documentation and lessons learned
```

### **Backup Role Assignments**
```yaml
Backup Coverage:
  Resident Pharmacist Absence:
    - Relief pharmacist activation
    - Temporary role assignment
    - Limited operational scope
    - ACPN notification required
    
  Management Absence:
    - Deputy manager activation
    - Restricted authority levels
    - Owner notification required
    - Maximum coverage period: 30 days
    
  Owner Absence:
    - Designated successor activation
    - Legal documentation required
    - ACPN approval for extended periods
    - Financial oversight maintained
```

---

## Compliance & Audit Features

### **Role Compliance Monitoring**
```yaml
Compliance Checks:
  Daily:
    - Active role verification
    - Permission usage monitoring
    - Suspicious activity detection
    
  Weekly:
    - Role assignment reviews
    - Permission effectiveness analysis
    - User activity patterns
    
  Monthly:
    - Comprehensive role audits
    - Permission optimization
    - Security assessment
    
  Quarterly:
    - Role hierarchy review
    - Business requirement alignment
    - Regulatory compliance check
```

### **Audit Trail Requirements**
```yaml
Audit Logging:
  Role Changes:
    - Role assignment/removal
    - Permission modifications
    - Approval workflows
    - Emergency overrides
    
  Access Events:
    - Login/logout activities
    - Permission usage
    - Failed access attempts
    - Privilege escalations
    
  Data Access:
    - Sensitive data viewing
    - Financial information access
    - Personal data handling
    - Export/download activities
```

---

## Related Documentation

- [Multi-Store RBAC](multi-store-rbac.md)
- [Security Architecture](../architecture/security.md)
- [User-Pharmacy Relationships](../data/user-pharmacy.md)
- [Authentication Systems](jwt.md)

This comprehensive role and permission system ensures secure, compliant, and efficient operations across all ACPN pharmacy operations while maintaining the flexibility required for diverse business models.