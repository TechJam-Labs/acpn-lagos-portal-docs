# User & Pharmacy Data Models

> **Multi-Store Pharmacy Management Data Architecture**

Comprehensive data models supporting complex user-pharmacy relationships, multi-store ownership, resident pharmacist requirements, and hierarchical access control.

---

## Overview

The ACPN system supports sophisticated multi-store pharmacy operations where individual pharmacists can own multiple pharmacies, each requiring a qualified resident pharmacist. This document defines the data structures that enable flexible ownership models while maintaining regulatory compliance.

## Core Relationship Principles

### **Multi-Store Ownership Model**
- One pharmacist can own multiple pharmacy stores
- Each pharmacy requires one designated resident pharmacist
- Owner and resident pharmacist can be the same person
- Staff can work across multiple stores with proper permissions
- Hierarchical access control based on roles and store associations

### **Regulatory Compliance**
- PCN (Pharmacists Council of Nigeria) registration validation
- Resident pharmacist qualification verification
- Store-specific licensing and permits
- Audit trail for all ownership and staff changes

---

## User-Pharmacy Relationship Models

### **Core Relationship Interface**
```typescript
interface UserPharmacyRelationship {
  id: string;
  userId: string;
  pharmacyId: string;
  relationshipType: PharmacyRelationshipType;
  
  // Role and Permissions
  role: PharmacyRole;
  permissions: PharmacyPermission[];
  accessLevel: AccessLevel;
  
  // Employment Details
  employmentDetails?: EmploymentDetails;
  
  // Status and Validity
  status: RelationshipStatus;
  startDate: Date;
  endDate?: Date;
  
  // Approval and Verification
  approvedBy?: string; // User ID
  approvedAt?: Date;
  verificationStatus: VerificationStatus;
  verificationDocuments: string[]; // URLs
  
  // System Fields
  createdAt: Date;
  updatedAt: Date;
  createdBy: string; // User ID
  lastModifiedBy: string; // User ID
}

enum PharmacyRelationshipType {
  OWNER = 'owner',
  RESIDENT_PHARMACIST = 'resident_pharmacist',
  STAFF_PHARMACIST = 'staff_pharmacist',
  PHARMACY_TECHNICIAN = 'pharmacy_technician',
  PHARMACY_ASSISTANT = 'pharmacy_assistant',
  MANAGER = 'manager',
  CASHIER = 'cashier',
  INTERN = 'intern'
}

enum PharmacyRole {
  PRIMARY_OWNER = 'primary_owner',
  CO_OWNER = 'co_owner',
  RESIDENT_PHARMACIST = 'resident_pharmacist',
  RELIEF_PHARMACIST = 'relief_pharmacist',
  SENIOR_PHARMACIST = 'senior_pharmacist',
  JUNIOR_PHARMACIST = 'junior_pharmacist',
  PHARMACY_MANAGER = 'pharmacy_manager',
  SHIFT_SUPERVISOR = 'shift_supervisor',
  SENIOR_TECHNICIAN = 'senior_technician',
  JUNIOR_TECHNICIAN = 'junior_technician',
  PHARMACY_ASSISTANT = 'pharmacy_assistant',
  CASHIER = 'cashier',
  SECURITY_GUARD = 'security_guard',
  CLEANER = 'cleaner',
  INTERN = 'intern'
}

enum AccessLevel {
  FULL_ACCESS = 'full_access',           // All operations
  MANAGEMENT_ACCESS = 'management_access', // Management and operations
  OPERATIONAL_ACCESS = 'operational_access', // Daily operations only
  LIMITED_ACCESS = 'limited_access',     // Specific functions only
  READ_ONLY_ACCESS = 'read_only_access', // View only
  NO_ACCESS = 'no_access'                // Disabled
}

enum RelationshipStatus {
  ACTIVE = 'active',
  INACTIVE = 'inactive',
  SUSPENDED = 'suspended',
  TERMINATED = 'terminated',
  PENDING_APPROVAL = 'pending_approval',
  PENDING_VERIFICATION = 'pending_verification'
}
```

### **Multi-Store Ownership Model**
```typescript
interface PharmacyOwnership {
  id: string;
  ownerId: string; // User ID
  pharmacyId: string;
  ownershipType: OwnershipType;
  ownershipPercentage: number; // For co-ownership scenarios
  
  // Ownership Details
  ownershipStartDate: Date;
  ownershipEndDate?: Date;
  acquisitionMethod: AcquisitionMethod;
  
  // Legal Documentation
  legalDocuments: LegalDocument[];
  businessRegistration: BusinessRegistration;
  
  // Financial Information
  financialDetails: OwnershipFinancialDetails;
  
  // Status
  status: OwnershipStatus;
  verificationStatus: VerificationStatus;
  
  // Approval Workflow
  approvalWorkflow: ApprovalWorkflow;
  
  // System Fields
  createdAt: Date;
  updatedAt: Date;
  recordedBy: string; // User ID (ACPN admin)
}

enum OwnershipType {
  SOLE_PROPRIETORSHIP = 'sole_proprietorship',
  PARTNERSHIP = 'partnership',
  LIMITED_LIABILITY = 'limited_liability',
  CORPORATE = 'corporate',
  FRANCHISE = 'franchise',
  COOPERATIVE = 'cooperative'
}

enum AcquisitionMethod {
  NEW_ESTABLISHMENT = 'new_establishment',
  PURCHASE = 'purchase',
  INHERITANCE = 'inheritance',
  PARTNERSHIP_ENTRY = 'partnership_entry',
  CORPORATE_ACQUISITION = 'corporate_acquisition',
  FRANCHISE_AGREEMENT = 'franchise_agreement',
  LEASE_TO_OWN = 'lease_to_own'
}

interface BusinessRegistration {
  registrationNumber: string;
  registrationDate: Date;
  registrationAuthority: string;
  registrationType: 'CAC' | 'Business_Name' | 'Partnership' | 'LTD' | 'PLC';
  expiryDate?: Date;
  renewalDate?: Date;
  status: 'active' | 'expired' | 'suspended' | 'cancelled';
  
  // Additional Details
  businessName: string;
  tradingName?: string;
  businessAddress: Address;
  directors?: Director[];
  shareholders?: Shareholder[];
}
```

### **Resident Pharmacist Management**
```typescript
interface ResidentPharmacistAssignment {
  id: string;
  pharmacyId: string;
  pharmacistId: string; // User ID
  
  // Assignment Details
  assignmentType: ResidentPharmacistType;
  startDate: Date;
  endDate?: Date;
  
  // Qualifications
  professionalQualifications: ProfessionalQualification[];
  pcnRegistration: PCNRegistration;
  specializations: string[];
  
  // Responsibilities
  responsibilities: ResidentPharmacistResponsibility[];
  workingHours: WorkingHours;
  availabilitySchedule: AvailabilitySchedule[];
  
  // Backup Coverage
  backupPharmacists: BackupPharmacist[];
  reliefArrangements: ReliefArrangement[];
  
  // Performance and Compliance
  performanceMetrics: PerformanceMetrics;
  complianceRecords: ComplianceRecord[];
  inspectionHistory: InspectionRecord[];
  
  // Status and Verification
  status: ResidentPharmacistStatus;
  verificationStatus: VerificationStatus;
  approvalDate?: Date;
  approvedBy?: string; // ACPN admin user ID
  
  // System Fields
  createdAt: Date;
  updatedAt: Date;
  lastReviewDate?: Date;
  nextReviewDate?: Date;
}

enum ResidentPharmacistType {
  FULL_TIME_RESIDENT = 'full_time_resident',
  PART_TIME_RESIDENT = 'part_time_resident',
  RELIEF_PHARMACIST = 'relief_pharmacist',
  LOCUM_PHARMACIST = 'locum_pharmacist',
  OWNER_RESIDENT = 'owner_resident'
}

enum ResidentPharmacistStatus {
  ACTIVE = 'active',
  TEMPORARY_ABSENCE = 'temporary_absence',
  EXTENDED_LEAVE = 'extended_leave',
  SUSPENDED = 'suspended',
  TERMINATED = 'terminated',
  PENDING_VERIFICATION = 'pending_verification'
}

interface PCNRegistration {
  pcnNumber: string;
  registrationDate: Date;
  expiryDate: Date;
  registrationCategory: PCNCategory;
  practiceScope: PracticeScope[];
  status: PCNStatus;
  
  // Additional Details
  issuingOffice: string;
  renewalHistory: RenewalRecord[];
  disciplinaryHistory: DisciplinaryRecord[];
  continuingEducation: CPDRecord[];
}

enum PCNCategory {
  PHARMACIST = 'pharmacist',
  PHARMACY_TECHNICIAN = 'pharmacy_technician',
  PHARMACY_TECHNOLOGIST = 'pharmacy_technologist',
  PHARMACEUTICAL_ASSISTANT = 'pharmaceutical_assistant'
}

enum PracticeScope {
  COMMUNITY_PHARMACY = 'community_pharmacy',
  HOSPITAL_PHARMACY = 'hospital_pharmacy',
  INDUSTRIAL_PHARMACY = 'industrial_pharmacy',
  ACADEMIC_PHARMACY = 'academic_pharmacy',
  REGULATORY_PHARMACY = 'regulatory_pharmacy',
  CLINICAL_PHARMACY = 'clinical_pharmacy'
}
```

### **Staff Management Model**
```typescript
interface PharmacyStaffAssignment {
  id: string;
  userId: string;
  pharmacyId: string;
  
  // Position Details
  position: StaffPosition;
  department?: string;
  reportingManager?: string; // User ID
  
  // Employment Terms
  employmentType: EmploymentType;
  contractDetails: ContractDetails;
  compensation: CompensationDetails;
  
  // Work Schedule
  workSchedule: WorkSchedule;
  shiftPattern: ShiftPattern;
  
  // Permissions and Access
  systemAccess: SystemAccess;
  physicalAccess: PhysicalAccess;
  
  // Training and Certification
  requiredTraining: TrainingRequirement[];
  completedTraining: TrainingRecord[];
  certifications: CertificationRecord[];
  
  // Performance Management
  performanceGoals: PerformanceGoal[];
  performanceReviews: PerformanceReview[];
  disciplinaryActions: DisciplinaryAction[];
  
  // Status
  status: StaffStatus;
  
  // System Fields
  createdAt: Date;
  updatedAt: Date;
  lastPerformanceReview?: Date;
  nextPerformanceReview?: Date;
}

enum EmploymentType {
  FULL_TIME_PERMANENT = 'full_time_permanent',
  PART_TIME_PERMANENT = 'part_time_permanent',
  CONTRACT = 'contract',
  TEMPORARY = 'temporary',
  INTERNSHIP = 'internship',
  VOLUNTEER = 'volunteer',
  CONSULTANT = 'consultant'
}

interface ContractDetails {
  contractStartDate: Date;
  contractEndDate?: Date;
  probationPeriod?: number; // months
  noticePeriod: number; // days
  
  // Contract Terms
  workingHoursPerWeek: number;
  annualLeaveEntitlement: number; // days
  sickLeaveEntitlement: number; // days
  
  // Termination Clauses
  terminationConditions: string[];
  severanceTerms?: string;
  
  // Contract Documents
  contractDocuments: string[]; // URLs to signed contracts
  amendments: ContractAmendment[];
}

interface SystemAccess {
  hasPortalAccess: boolean;
  permissions: PharmacyPermission[];
  modules: SystemModule[];
  dataAccessLevel: DataAccessLevel;
  
  // Security
  ipRestrictions?: string[];
  timeRestrictions?: TimeRestriction[];
  deviceRestrictions?: DeviceRestriction[];
  
  // Audit
  lastLogin?: Date;
  loginHistory: LoginRecord[];
  accessViolations: AccessViolation[];
}
```

---

## Permission and Access Control Models

### **Pharmacy Permission System**
```typescript
interface PharmacyPermission {
  id: string;
  name: string;
  code: string;
  description: string;
  category: PermissionCategory;
  
  // Scope
  scope: PermissionScope;
  resources: string[];
  actions: PermissionAction[];
  
  // Conditions
  conditions?: PermissionCondition[];
  restrictions?: PermissionRestriction[];
  
  // Hierarchy
  parentPermission?: string;
  childPermissions: string[];
  
  // Status
  isActive: boolean;
  requiresApproval: boolean;
  riskLevel: RiskLevel;
  
  // System Fields
  createdAt: Date;
  updatedAt: Date;
}

enum PermissionCategory {
  USER_MANAGEMENT = 'user_management',
  PHARMACY_MANAGEMENT = 'pharmacy_management',
  INVENTORY_MANAGEMENT = 'inventory_management',
  SALES_MANAGEMENT = 'sales_management',
  FINANCIAL_MANAGEMENT = 'financial_management',
  REPORTING = 'reporting',
  SYSTEM_ADMINISTRATION = 'system_administration',
  COMPLIANCE = 'compliance',
  CPD_MANAGEMENT = 'cpd_management'
}

enum PermissionScope {
  SYSTEM_WIDE = 'system_wide',      // All pharmacies
  PHARMACY_CHAIN = 'pharmacy_chain', // All owned pharmacies
  SINGLE_PHARMACY = 'single_pharmacy', // Specific pharmacy
  DEPARTMENT = 'department',         // Department within pharmacy
  PERSONAL = 'personal'             // Own data only
}

enum PermissionAction {
  CREATE = 'create',
  READ = 'read',
  UPDATE = 'update',
  DELETE = 'delete',
  APPROVE = 'approve',
  REJECT = 'reject',
  EXPORT = 'export',
  IMPORT = 'import',
  PRINT = 'print',
  SHARE = 'share'
}

interface PermissionCondition {
  type: ConditionType;
  field: string;
  operator: 'equals' | 'not_equals' | 'greater_than' | 'less_than' | 'contains' | 'in' | 'not_in';
  value: any;
  logicalOperator?: 'AND' | 'OR';
}

enum ConditionType {
  TIME_BASED = 'time_based',
  LOCATION_BASED = 'location_based',
  ROLE_BASED = 'role_based',
  VALUE_BASED = 'value_based',
  STATUS_BASED = 'status_based'
}
```

### **Role-Based Access Control (RBAC)**
```typescript
interface PharmacyRoleDefinition {
  id: string;
  name: string;
  code: string;
  description: string;
  
  // Role Hierarchy
  level: RoleLevel;
  parentRole?: string;
  childRoles: string[];
  
  // Permissions
  defaultPermissions: string[]; // Permission IDs
  conditionalPermissions: ConditionalPermission[];
  
  // Assignments
  eligibilityRequirements: EligibilityRequirement[];
  maxAssignments?: number; // Max users with this role
  
  // Constraints
  mutuallyExclusiveRoles: string[]; // Cannot have these roles simultaneously
  prerequisiteRoles: string[]; // Must have these roles first
  
  // Approval
  requiresApproval: boolean;
  approvalWorkflow?: ApprovalWorkflow;
  
  // System Fields
  isSystemRole: boolean;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

enum RoleLevel {
  OWNER = 1,           // Pharmacy owner
  MANAGEMENT = 2,      // Pharmacy management
  PROFESSIONAL = 3,    // Pharmacists and technicians
  OPERATIONAL = 4,     // General staff
  TEMPORARY = 5        // Interns, volunteers
}

interface ConditionalPermission {
  permissionId: string;
  conditions: PermissionCondition[];
  grantedWhen: 'all_conditions_met' | 'any_condition_met';
  temporary?: boolean;
  expiresAt?: Date;
}

interface EligibilityRequirement {
  type: RequirementType;
  description: string;
  mandatory: boolean;
  validationRule: string;
}

enum RequirementType {
  PROFESSIONAL_QUALIFICATION = 'professional_qualification',
  EXPERIENCE_YEARS = 'experience_years',
  CERTIFICATION = 'certification',
  TRAINING_COMPLETION = 'training_completion',
  BACKGROUND_CHECK = 'background_check',
  AGE_REQUIREMENT = 'age_requirement'
}
```

---

## Multi-Store Management Models

### **Pharmacy Chain Management**
```typescript
interface PharmacyChain {
  id: string;
  name: string;
  description?: string;
  
  // Chain Details
  headquarters: Address;
  establishedDate: Date;
  legalStructure: ChainLegalStructure;
  
  // Ownership
  primaryOwner: string; // User ID
  stakeholders: ChainStakeholder[];
  
  // Operations
  pharmacies: string[]; // Pharmacy IDs
  totalStores: number;
  operatingRegions: Region[];
  
  // Management Structure
  corporateStructure: CorporateStructure;
  managementTeam: ManagementPosition[];
  
  // Policies and Standards
  operatingPolicies: OperatingPolicy[];
  qualityStandards: QualityStandard[];
  
  // Financial
  financialStructure: ChainFinancialStructure;
  
  // System Fields
  status: ChainStatus;
  createdAt: Date;
  updatedAt: Date;
  lastAuditDate?: Date;
}

enum ChainLegalStructure {
  SOLE_PROPRIETORSHIP = 'sole_proprietorship',
  PARTNERSHIP = 'partnership',
  LIMITED_LIABILITY_COMPANY = 'limited_liability_company',
  PUBLIC_LIMITED_COMPANY = 'public_limited_company',
  FRANCHISE = 'franchise',
  COOPERATIVE = 'cooperative'
}

interface ChainStakeholder {
  userId: string;
  stakeholderType: StakeholderType;
  ownershipPercentage?: number;
  investmentAmount?: number;
  role: string;
  responsibilities: string[];
  
  // Rights and Obligations
  votingRights: boolean;
  managementRights: boolean;
  dividendRights: boolean;
  
  // Status
  status: StakeholderStatus;
  joinDate: Date;
  exitDate?: Date;
}

enum StakeholderType {
  MAJORITY_SHAREHOLDER = 'majority_shareholder',
  MINORITY_SHAREHOLDER = 'minority_shareholder',
  SILENT_PARTNER = 'silent_partner',
  MANAGING_PARTNER = 'managing_partner',
  INVESTOR = 'investor',
  FRANCHISEE = 'franchisee'
}
```

### **Cross-Store Staff Management**
```typescript
interface CrossStoreAssignment {
  id: string;
  staffUserId: string;
  primaryPharmacyId: string;
  
  // Cross-Store Access
  accessiblePharmacies: CrossStoreAccess[];
  transferHistory: StoreTransfer[];
  
  // Scheduling
  multiStoreSchedule: MultiStoreSchedule;
  
  // Permissions
  uniformPermissions: boolean; // Same permissions across all stores
  storeSpecificPermissions: StorePermission[];
  
  // Performance
  performanceByStore: StorePerformance[];
  
  // System Fields
  createdAt: Date;
  updatedAt: Date;
  lastUpdate: Date;
}

interface CrossStoreAccess {
  pharmacyId: string;
  accessType: AccessType;
  accessLevel: AccessLevel;
  permissions: string[]; // Permission IDs
  
  // Constraints
  maxHoursPerWeek?: number;
  specificDepartments?: string[];
  timeRestrictions?: TimeRestriction[];
  
  // Status
  status: AccessStatus;
  startDate: Date;
  endDate?: Date;
  
  // Approval
  approvedBy: string; // User ID
  approvalDate: Date;
  approvalNotes?: string;
}

enum AccessType {
  PERMANENT = 'permanent',
  TEMPORARY = 'temporary',
  EMERGENCY = 'emergency',
  TRAINING = 'training',
  RELIEF = 'relief',
  PROJECT_BASED = 'project_based'
}

interface StoreTransfer {
  id: string;
  fromPharmacyId: string;
  toPharmacyId: string;
  transferType: TransferType;
  
  // Transfer Details
  effectiveDate: Date;
  reason: string;
  duration?: number; // days (for temporary transfers)
  
  // Approval
  requestedBy: string; // User ID
  approvedBy: string; // User ID
  approvalDate: Date;
  
  // Status
  status: TransferStatus;
  
  // System Fields
  createdAt: Date;
  completedAt?: Date;
}

enum TransferType {
  PERMANENT = 'permanent',
  TEMPORARY = 'temporary',
  EMERGENCY_COVERAGE = 'emergency_coverage',
  TRAINING_ASSIGNMENT = 'training_assignment',
  PROMOTIONAL = 'promotional'
}
```

---

## Compliance and Audit Models

### **Regulatory Compliance Tracking**
```typescript
interface ComplianceRecord {
  id: string;
  pharmacyId: string;
  complianceType: ComplianceType;
  
  // Compliance Details
  requirement: string;
  description: string;
  regulatoryAuthority: string;
  
  // Status
  status: ComplianceStatus;
  lastAssessmentDate: Date;
  nextAssessmentDate: Date;
  
  // Evidence
  evidenceDocuments: string[]; // URLs
  certifications: string[]; // URLs
  
  // Responsible Parties
  responsiblePerson: string; // User ID
  verifiedBy?: string; // User ID
  
  // Action Items
  actionItems: ComplianceActionItem[];
  
  // System Fields
  createdAt: Date;
  updatedAt: Date;
  lastReviewDate?: Date;
}

enum ComplianceType {
  PHARMACY_LICENSE = 'pharmacy_license',
  DRUG_LICENSE = 'drug_license',
  RESIDENT_PHARMACIST = 'resident_pharmacist',
  CONTROLLED_SUBSTANCES = 'controlled_substances',
  STORAGE_CONDITIONS = 'storage_conditions',
  RECORD_KEEPING = 'record_keeping',
  STAFF_QUALIFICATIONS = 'staff_qualifications',
  SAFETY_PROTOCOLS = 'safety_protocols',
  ENVIRONMENTAL = 'environmental'
}

interface ComplianceActionItem {
  id: string;
  description: string;
  priority: Priority;
  dueDate: Date;
  assignedTo: string; // User ID
  status: ActionItemStatus;
  
  // Progress Tracking
  progressNotes: ProgressNote[];
  
  // Completion
  completedDate?: Date;
  completedBy?: string; // User ID
  verificationRequired: boolean;
  verifiedBy?: string; // User ID
}
```

### **Audit Trail System**
```typescript
interface UserPharmacyAuditLog {
  id: string;
  
  // Entity Information
  entityType: 'user' | 'pharmacy' | 'relationship' | 'permission' | 'role';
  entityId: string;
  
  // Action Details
  action: AuditAction;
  description: string;
  
  // User Context
  performedBy: string; // User ID
  onBehalfOf?: string; // User ID (for delegated actions)
  
  // Changes
  oldValues?: Record<string, any>;
  newValues?: Record<string, any>;
  changedFields: string[];
  
  // Context
  pharmacyId?: string;
  ipAddress: string;
  userAgent: string;
  sessionId: string;
  
  // Risk Assessment
  riskLevel: RiskLevel;
  flaggedForReview: boolean;
  reviewedBy?: string; // User ID
  reviewNotes?: string;
  
  // System Fields
  timestamp: Date;
  source: AuditSource;
  
  // Compliance
  retentionPeriod: number; // years
  archived: boolean;
}

enum AuditAction {
  CREATE = 'create',
  UPDATE = 'update',
  DELETE = 'delete',
  VIEW = 'view',
  APPROVE = 'approve',
  REJECT = 'reject',
  SUSPEND = 'suspend',
  ACTIVATE = 'activate',
  TRANSFER = 'transfer',
  ASSIGN = 'assign',
  REMOVE = 'remove',
  LOGIN = 'login',
  LOGOUT = 'logout',
  PERMISSION_CHANGE = 'permission_change',
  ROLE_CHANGE = 'role_change'
}

enum RiskLevel {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  CRITICAL = 'critical'
}

enum AuditSource {
  WEB_PORTAL = 'web_portal',
  MOBILE_APP = 'mobile_app',
  API = 'api',
  SYSTEM = 'system',
  ADMIN_PANEL = 'admin_panel',
  BULK_IMPORT = 'bulk_import'
}
```

---

## Related Documentation

- [Core Data Models](core-models.md)
- [Multi-Store Management](../pharmacy/multi-store.md)
- [Authentication & Authorization](../auth/overview.md)
- [Security Architecture](../architecture/security.md)

This comprehensive data model supports sophisticated multi-store pharmacy operations while maintaining regulatory compliance and detailed audit trails for all user-pharmacy relationships. 