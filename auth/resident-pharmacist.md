# Resident Pharmacist Management

> **Professional Compliance & Authentication System**

Comprehensive system managing resident pharmacist requirements, authentication, and regulatory compliance for multi-store pharmacy operations.

---

## Overview

The Resident Pharmacist Management System ensures each pharmacy location meets ACPN regulatory requirements by maintaining qualified pharmaceutical oversight, managing professional credentials, and ensuring compliance with operational standards.

### **Regulatory Requirements**
- Each pharmacy MUST have a designated resident pharmacist
- Physical presence requirements during operational hours
- Professional licensing and continuing education compliance
- ACPN oversight and reporting obligations
- Emergency coverage procedures

---

## Core Features

### **Professional Authentication**
```yaml
Credential Verification:
  - ACPN registration number validation
  - Professional license verification
  - Educational qualification checks
  - Continuing education compliance
  - Disciplinary record review

Multi-Factor Authentication:
  - Biometric verification (fingerprint/face)
  - Professional PIN authentication
  - SMS/Email verification
  - Hardware token support
  - Time-based access controls
```

### **Presence Management**
```yaml
Physical Presence Tracking:
  - Check-in/check-out timestamps
  - Location-based verification
  - IP address monitoring
  - Device authentication
  - Emergency override procedures

Operational Coverage:
  - Minimum presence requirements
  - Coverage scheduling
  - Temporary absence protocols
  - Backup pharmacist assignments
  - Emergency contact procedures
```

---

## System Architecture

### **Database Schema**

#### **Resident Pharmacist Records**
```sql
-- Resident Pharmacist Assignments
CREATE TABLE resident_pharmacists (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    pharmacy_id UUID REFERENCES pharmacies(id),
    acpn_registration VARCHAR(20) NOT NULL UNIQUE,
    license_number VARCHAR(50) NOT NULL,
    assignment_date DATE NOT NULL,
    termination_date DATE,
    status VARCHAR(20) DEFAULT 'active',
    is_primary BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Professional Credentials
CREATE TABLE pharmacist_credentials (
    id UUID PRIMARY KEY,
    pharmacist_id UUID REFERENCES resident_pharmacists(id),
    credential_type VARCHAR(50), -- 'license', 'certification', 'education'
    issuing_authority VARCHAR(100),
    credential_number VARCHAR(100),
    issue_date DATE,
    expiry_date DATE,
    status VARCHAR(20) DEFAULT 'active',
    document_url TEXT,
    verification_status VARCHAR(20) DEFAULT 'pending',
    verified_by UUID REFERENCES users(id),
    verified_at TIMESTAMP
);

-- Presence Tracking
CREATE TABLE pharmacist_presence (
    id UUID PRIMARY KEY,
    pharmacist_id UUID REFERENCES resident_pharmacists(id),
    pharmacy_id UUID REFERENCES pharmacies(id),
    check_in_time TIMESTAMP NOT NULL,
    check_out_time TIMESTAMP,
    presence_type VARCHAR(20), -- 'regular', 'emergency', 'coverage'
    location_verified BOOLEAN DEFAULT false,
    ip_address INET,
    device_id VARCHAR(100),
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Coverage Arrangements
CREATE TABLE pharmacist_coverage (
    id UUID PRIMARY KEY,
    primary_pharmacist_id UUID REFERENCES resident_pharmacists(id),
    covering_pharmacist_id UUID REFERENCES resident_pharmacists(id),
    pharmacy_id UUID REFERENCES pharmacies(id),
    coverage_start TIMESTAMP NOT NULL,
    coverage_end TIMESTAMP NOT NULL,
    reason VARCHAR(100),
    approved_by UUID REFERENCES users(id),
    status VARCHAR(20) DEFAULT 'active',
    emergency_coverage BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### **API Implementation**

#### **Authentication Service**
```typescript
class ResidentPharmacistAuth {
    async authenticatePharmacist(
        acpnNumber: string, 
        pharmacyId: string, 
        authMethod: AuthMethod
    ): Promise<AuthResult> {
        // 1. Verify ACPN registration
        const pharmacist = await this.verifyACPNRegistration(acpnNumber);
        if (!pharmacist) {
            throw new Error('Invalid ACPN registration');
        }
        
        // 2. Check pharmacy assignment
        const assignment = await this.getPharmacyAssignment(pharmacist.id, pharmacyId);
        if (!assignment || assignment.status !== 'active') {
            throw new Error('No active assignment to this pharmacy');
        }
        
        // 3. Verify credentials status
        await this.verifyCredentialsStatus(pharmacist.id);
        
        // 4. Perform multi-factor authentication
        const authResult = await this.performMFA(pharmacist.id, authMethod);
        
        // 5. Log authentication attempt
        await this.logAuthentication(pharmacist.id, pharmacyId, authResult);
        
        return authResult;
    }
    
    async recordPresence(
        pharmacistId: string, 
        pharmacyId: string, 
        presenceData: PresenceData
    ): Promise<void> {
        // Validate location
        const locationValid = await this.validateLocation(
            presenceData.coordinates, 
            pharmacyId
        );
        
        // Record presence
        await this.createPresenceRecord({
            pharmacistId,
            pharmacyId,
            checkInTime: new Date(),
            locationVerified: locationValid,
            ...presenceData
        });
        
        // Update pharmacy operational status
        await this.updatePharmacyOperationalStatus(pharmacyId, 'pharmacist_present');
        
        // Notify stakeholders
        await this.notifyPresenceChange(pharmacistId, pharmacyId, 'check_in');
    }
}
```

#### **Compliance Monitoring**
```typescript
class ComplianceMonitor {
    async checkPresenceCompliance(pharmacyId: string): Promise<ComplianceReport> {
        const today = new Date();
        const operatingHours = await this.getOperatingHours(pharmacyId);
        
        // Check current presence
        const currentPresence = await this.getCurrentPresence(pharmacyId);
        
        // Calculate required vs actual presence
        const requiredHours = this.calculateRequiredHours(operatingHours);
        const actualHours = await this.getActualPresenceHours(pharmacyId, today);
        
        // Generate compliance report
        return {
            pharmacyId,
            date: today,
            requiredHours,
            actualHours,
            compliancePercentage: (actualHours / requiredHours) * 100,
            currentStatus: currentPresence ? 'compliant' : 'non_compliant',
            violations: await this.getViolations(pharmacyId, today),
            recommendations: this.generateRecommendations(requiredHours, actualHours)
        };
    }
    
    async monitorCredentialExpiry(): Promise<ExpiryAlert[]> {
        const expiringCredentials = await this.getExpiringCredentials(30); // 30 days ahead
        
        const alerts: ExpiryAlert[] = [];
        
        for (const credential of expiringCredentials) {
            const daysUntilExpiry = this.calculateDaysUntilExpiry(credential.expiryDate);
            
            alerts.push({
                pharmacistId: credential.pharmacistId,
                credentialType: credential.type,
                expiryDate: credential.expiryDate,
                daysRemaining: daysUntilExpiry,
                urgencyLevel: this.getUrgencyLevel(daysUntilExpiry),
                actionRequired: this.getRequiredAction(credential.type, daysUntilExpiry)
            });
        }
        
        return alerts;
    }
}
```

---

## Workflow Management

### **Pharmacist Assignment Process**

#### **New Assignment Workflow**
```yaml
Step 1: Qualification Verification
  - ACPN registration validation
  - Professional license verification
  - Educational credential review
  - Background check completion
  - Reference verification

Step 2: Pharmacy Onboarding
  - Pharmacy-specific training
  - System access setup
  - Local procedure familiarization
  - Emergency protocol training
  - Compliance orientation

Step 3: Formal Assignment
  - Assignment documentation
  - ACPN notification
  - Insurance verification
  - Access credential issuance
  - Operational handover

Step 4: Monitoring Setup
  - Presence tracking activation
  - Performance metric baseline
  - Compliance monitoring initiation
  - Regular review scheduling
  - Feedback mechanism setup
```

#### **Coverage Arrangements**
```yaml
Temporary Coverage:
  Duration: Up to 30 days
  Requirements:
    - Qualified pharmacist available
    - ACPN notification within 24 hours
    - Documentation of arrangement
    - Approval from pharmacy owner
    - Emergency contact information

Extended Coverage:
  Duration: 30+ days
  Requirements:
    - Formal assignment process
    - ACPN approval required
    - Full credential verification
    - Insurance transfer
    - Comprehensive handover

Emergency Coverage:
  Duration: Up to 72 hours
  Requirements:
    - Immediate ACPN notification
    - Qualified pharmacist on-site
    - Emergency documentation
    - Post-incident reporting
    - Correction plan submission
```

### **Compliance Scenarios**

#### **Scenario 1: Planned Absence**
```yaml
Situation: Resident pharmacist taking vacation
Duration: 2 weeks
Requirements:
  - 14 days advance notice
  - Coverage pharmacist identified
  - ACPN notification submitted
  - Handover documentation complete
  - Emergency contacts updated

Process:
  1. Submit absence request
  2. Identify qualified coverage
  3. Complete handover checklist
  4. Notify ACPN and stakeholders
  5. Update system access
  6. Monitor coverage compliance
  7. Complete return transition
```

#### **Scenario 2: Emergency Absence**
```yaml
Situation: Resident pharmacist medical emergency
Duration: Immediate, undefined
Requirements:
  - Immediate pharmacy closure or coverage
  - Emergency ACPN notification
  - Rapid coverage arrangement
  - Documentation of circumstances
  - Recovery and return planning

Process:
  1. Assess immediate pharmacy needs
  2. Secure emergency coverage or close
  3. Notify ACPN within 2 hours
  4. Document emergency circumstances
  5. Arrange temporary coverage
  6. Plan permanent solution
  7. Monitor compliance during transition
```

#### **Scenario 3: Credential Expiry**
```yaml
Situation: Professional license expiring
Timeline: 30 days notice required
Requirements:
  - Renewal application submitted
  - Continuing education completed
  - Documentation updated
  - System access maintained
  - Compliance monitoring continued

Process:
  1. Receive expiry notification
  2. Initiate renewal process
  3. Complete required education
  4. Submit renewal application
  5. Provide temporary documentation
  6. Update system records
  7. Confirm renewal completion
```

---

## Security Features

### **Multi-Factor Authentication**
```yaml
Primary Authentication:
  - ACPN registration number
  - Personal identification number
  - Professional license verification
  - Biometric confirmation

Secondary Verification:
  - SMS verification code
  - Email confirmation
  - Mobile app authentication
  - Hardware security key

Location Verification:
  - GPS coordinates
  - IP address validation
  - WiFi network detection
  - Bluetooth beacon proximity
```

### **Access Control**
```yaml
Professional Privileges:
  - Prescription verification
  - Clinical consultations
  - Professional supervision
  - Compliance oversight
  - Regulatory reporting

Operational Limitations:
  - Pharmacy-specific access
  - Time-based restrictions
  - Function-based permissions
  - Emergency override controls
  - Audit trail maintenance
```

---

## Monitoring & Reporting

### **Real-Time Dashboards**

#### **Pharmacy Owner Dashboard**
```yaml
Current Status:
  - Resident pharmacist presence
  - Operational compliance status
  - Coverage arrangements
  - Credential expiry alerts
  - Performance metrics

Quick Actions:
  - Request coverage
  - View compliance reports
  - Contact pharmacist
  - Access emergency procedures
  - Review audit logs
```

#### **ACPN Oversight Dashboard**
```yaml
Regional Overview:
  - Pharmacy compliance rates
  - Pharmacist assignments
  - Credential status summary
  - Violation tracking
  - Coverage arrangements

Regulatory Tools:
  - Compliance reporting
  - Violation management
  - Pharmacist verification
  - Audit scheduling
  - Policy enforcement
```

### **Compliance Reports**

#### **Daily Presence Report**
```yaml
Report Content:
  - Pharmacist check-in/out times
  - Total presence duration
  - Coverage periods
  - Compliance percentage
  - Violations or issues

Recipients:
  - Pharmacy owner
  - ACPN oversight team
  - Regional compliance officer
  - Insurance providers
  - Audit trail system
```

#### **Monthly Compliance Summary**
```yaml
Report Content:
  - Overall compliance metrics
  - Pharmacist performance
  - Credential status updates
  - Training completion
  - Improvement recommendations

Analysis:
  - Trend identification
  - Risk assessment
  - Performance benchmarking
  - Cost impact analysis
  - Strategic recommendations
```

---

## Integration Points

### **ACPN Systems**
```yaml
Data Exchange:
  - Registration verification
  - Credential status updates
  - Compliance reporting
  - Violation notifications
  - Educational tracking

API Endpoints:
  - /api/acpn/verify-pharmacist
  - /api/acpn/update-assignment
  - /api/acpn/report-compliance
  - /api/acpn/credential-status
  - /api/acpn/emergency-notification
```

### **Pharmacy Management System**
```yaml
Operational Integration:
  - Staff scheduling
  - Prescription management
  - Inventory oversight
  - Customer service
  - Financial reporting

Workflow Automation:
  - Presence-based access control
  - Automatic compliance checking
  - Alert generation
  - Report scheduling
  - Emergency procedures
```

---

## Troubleshooting

### **Common Issues**

#### **Authentication Failures**
```yaml
Symptom: Pharmacist cannot authenticate
Diagnosis:
  1. Verify ACPN registration status
  2. Check pharmacy assignment validity
  3. Confirm credential expiry dates
  4. Test authentication methods
  5. Review system access logs

Resolution:
  - Update registration records
  - Renew expired credentials
  - Reset authentication factors
  - Contact ACPN support
  - Emergency access provision
```

#### **Presence Tracking Issues**
```yaml
Symptom: Location verification failing
Diagnosis:
  1. Check GPS signal strength
  2. Verify pharmacy location data
  3. Test IP address detection
  4. Confirm device authentication
  5. Review network connectivity

Resolution:
  - Calibrate location settings
  - Update pharmacy coordinates
  - Whitelist IP addresses
  - Re-register devices
  - Manual presence override
```

#### **Compliance Violations**
```yaml
Symptom: Repeated presence violations
Diagnosis:
  1. Review presence patterns
  2. Check coverage arrangements
  3. Verify operational hours
  4. Assess staff schedules
  5. Identify systemic issues

Resolution:
  - Adjust coverage schedules
  - Improve staff planning
  - Implement backup procedures
  - Enhanced monitoring
  - ACPN consultation
```

---

## Related Documentation

- [Multi-Store RBAC](multi-store-rbac.md)
- [User Roles & Permissions](roles-permissions.md)
- [Multi-Store Management](../pharmacy/multi-store.md)
- [Security Architecture](../architecture/security.md)
- [CPD System](../cpd/overview.md)

This system ensures regulatory compliance while maintaining operational efficiency and professional standards across all pharmacy locations.