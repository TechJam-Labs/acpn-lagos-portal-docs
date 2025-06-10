# Security Architecture

> **Comprehensive Security Framework for ACPN Lagos Portal**

Multi-layered security architecture ensuring data protection, user privacy, and regulatory compliance for the pharmacy management system.

---

## Overview

The ACPN security architecture implements defense-in-depth principles with multiple security layers protecting sensitive pharmaceutical data, user information, and business operations. The system addresses healthcare data protection requirements while maintaining performance and usability.

## Key Security Principles

### **Zero Trust Architecture**
- No implicit trust for any system component
- Continuous verification of all access requests
- Minimal privilege access controls
- Network micro-segmentation

### **Defense in Depth**
- Multiple overlapping security controls
- Layered protection at application, network, and data levels
- Redundant security mechanisms
- Fail-safe defaults

### **Data Protection by Design**
- Privacy considerations embedded in system design
- Data minimization and purpose limitation
- Encryption at rest and in transit
- Regular data auditing and cleanup

---

## Authentication & Authorization

### **Multi-Factor Authentication (MFA)**
```yaml
MFA Configuration:
  Required Roles:
    - Super Admin
    - Pharmacy Owner
    - Resident Pharmacist
  
  Methods:
    - TOTP (Time-based One-Time Password)
    - SMS verification
    - Email verification
    - Backup codes

  Implementation:
    Library: speakeasy
    QR Code Generation: qrcode
    Backup Codes: crypto.randomBytes
```

### **Role-Based Access Control (RBAC)**
```typescript
interface SecurityRole {
  id: string;
  name: string;
  permissions: Permission[];
  restrictions: AccessRestriction[];
  sessionTimeout: number; // minutes
  ipWhitelist?: string[];
}

interface Permission {
  resource: string;
  actions: string[];
  conditions?: PermissionCondition[];
}

interface AccessRestriction {
  type: 'time' | 'location' | 'device' | 'network';
  rule: string;
  isActive: boolean;
}
```

### **JWT Token Security**
```yaml
JWT Configuration:
  Access Token:
    Expiry: 15 minutes
    Algorithm: RS256
    Issuer: acpn-portal
    
  Refresh Token:
    Expiry: 7 days
    Rotation: Enabled
    Storage: HttpOnly Cookie
    
  Security Headers:
    - Secure: true
    - SameSite: Strict
    - HttpOnly: true
```

---

## Data Protection

### **Encryption Standards**

#### **Data at Rest**
```yaml
Database Encryption:
  PostgreSQL:
    Method: AES-256
    Key Management: Environment variables
    Column-level: Sensitive fields only
    
  MongoDB:
    Method: AES-256-GCM
    Key Rotation: Monthly
    Collection-level: User data
    
  File Storage:
    Method: AES-256-CBC
    Compression: Before encryption
    Backup: Encrypted archives
```

#### **Data in Transit**
```yaml
Transport Security:
  TLS Version: 1.3 minimum
  Cipher Suites:
    - TLS_AES_256_GCM_SHA384
    - TLS_CHACHA20_POLY1305_SHA256
    - TLS_AES_128_GCM_SHA256
    
  Certificate Management:
    Provider: Let's Encrypt
    Renewal: Automatic
    HSTS: Enabled (max-age: 31536000)
    
  API Security:
    Authentication: Bearer tokens
    Rate Limiting: Redis-based
    Request Signing: HMAC-SHA256
```

### **Sensitive Data Handling**
```typescript
interface SensitiveDataTypes {
  PII: {
    fields: ['email', 'phone', 'address', 'dateOfBirth'];
    encryption: 'AES-256';
    accessLog: true;
    retention: '7 years';
  };
  
  Financial: {
    fields: ['bankDetails', 'paymentInfo', 'pricing'];
    encryption: 'AES-256';
    accessLog: true;
    retention: '10 years';
  };
  
  Medical: {
    fields: ['prescriptions', 'healthRecords', 'allergies'];
    encryption: 'AES-256';
    accessLog: true;
    retention: 'Indefinite';
  };
}
```

---

## Network Security

### **Firewall Configuration**
```yaml
UFW Rules:
  SSH: 
    Port: 22
    Source: Admin IPs only
    
  HTTP/HTTPS:
    Ports: 80, 443
    Source: Any
    Rate Limit: 1000/minute
    
  Database:
    PostgreSQL: 5432 (Internal only)
    MongoDB: 27017 (Internal only)
    Redis: 6379 (Internal only)
    
  Application:
    API: 3000 (Internal only)
    Monitoring: 9090-9100 (Admin IPs)
```

### **DDoS Protection**
```yaml
Protection Layers:
  Level 1: Nginx rate limiting
    - Connection limit: 10/IP
    - Request rate: 100/minute/IP
    
  Level 2: Fail2ban
    - Max attempts: 5
    - Ban time: 1 hour
    - Whitelist: Admin IPs
    
  Level 3: Application-level
    - API rate limiting
    - Suspicious pattern detection
    - Automatic blocking
```

### **VPN Access**
```yaml
VPN Configuration:
  Protocol: WireGuard
  Use Cases:
    - Remote administration
    - Database access
    - Log analysis
    
  Access Control:
    - Key-based authentication
    - IP address verification
    - Session logging
    - Automatic disconnection
```

---

## Application Security

### **Input Validation & Sanitization**
```typescript
interface ValidationRules {
  userInput: {
    maxLength: 255;
    allowedChars: /^[a-zA-Z0-9\s\-_.@]+$/;
    htmlSanitization: true;
    sqlInjectionCheck: true;
  };
  
  fileUpload: {
    maxSize: '10MB';
    allowedTypes: ['image/jpeg', 'image/png', 'application/pdf'];
    virusScanning: true;
    contentValidation: true;
  };
  
  apiEndpoints: {
    rateLimiting: true;
    authenticationRequired: true;
    inputValidation: 'joi';
    outputSanitization: true;
  };
}
```

### **Cross-Site Scripting (XSS) Prevention**
```yaml
XSS Protection:
  Content Security Policy:
    default-src: "'self'"
    script-src: "'self' 'unsafe-inline'"
    style-src: "'self' 'unsafe-inline'"
    img-src: "'self' data: https:"
    
  Output Encoding:
    HTML: Enabled
    JavaScript: Enabled
    CSS: Enabled
    URL: Enabled
    
  Framework Protection:
    React: Built-in XSS protection
    Express: helmet.js middleware
    DOMPurify: Client-side sanitization
```

### **SQL Injection Prevention**
```typescript
interface DatabaseSecurity {
  queryBuilder: 'Knex.js'; // Parameterized queries
  validation: 'Joi'; // Input validation
  escaping: 'pg-escape'; // Additional escaping
  
  restrictions: {
    dynamicQueries: false;
    storedProcedures: false;
    adminQueries: 'whitelist-only';
  };
}
```

---

## API Security

### **Authentication & Rate Limiting**
```yaml
API Security:
  Authentication:
    Method: JWT Bearer tokens
    Validation: Every request
    Refresh: Automatic
    
  Rate Limiting:
    Global: 1000 requests/hour/IP
    Per User: 500 requests/hour
    Per Endpoint: Variable limits
    
  Request Validation:
    Schema: JSON Schema
    Size: Max 1MB
    Timeout: 30 seconds
```

### **API Security Headers**
```typescript
interface SecurityHeaders {
  'X-Content-Type-Options': 'nosniff';
  'X-Frame-Options': 'DENY';
  'X-XSS-Protection': '1; mode=block';
  'Referrer-Policy': 'strict-origin-when-cross-origin';
  'Permissions-Policy': 'geolocation=(), microphone=(), camera=()';
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains';
  'Content-Security-Policy': string; // Defined per endpoint
}
```

---

## Infrastructure Security

### **Container Security**
```yaml
Docker Security:
  Base Images:
    Source: Official repositories only
    Scanning: Trivy vulnerability scanner
    Updates: Weekly security patches
    
  Runtime Security:
    User: Non-root containers
    Capabilities: Minimal required
    Read-only: Root filesystem
    
  Network:
    Isolation: Container networks
    Communication: Service mesh
    Monitoring: Network traffic analysis
```

### **Server Hardening**
```yaml
Ubuntu Hardening:
  OS Updates:
    Security: Automatic
    Kernel: Latest LTS
    Packages: Minimal installation
    
  Services:
    SSH: Key-based only
    Firewall: UFW enabled
    Fail2ban: Active monitoring
    
  Users:
    Root: Disabled
    Sudo: Limited users
    Passwords: Complex policy
    
  Monitoring:
    Logs: Centralized
    Intrusion: AIDE
    Performance: System metrics
```

---

## Compliance & Auditing

### **Healthcare Data Compliance**
```yaml
Compliance Standards:
  NDPR: Nigerian Data Protection Regulation
    - Consent management
    - Data subject rights
    - Breach notification
    
  ISO 27001: Information Security
    - Risk management
    - Security controls
    - Continuous improvement
    
  Healthcare: Medical data protection
    - Patient confidentiality
    - Access controls
    - Audit trails
```

### **Audit Logging**
```typescript
interface AuditLog {
  timestamp: Date;
  userId: string;
  action: string;
  resource: string;
  ipAddress: string;
  userAgent: string;
  result: 'success' | 'failure';
  risk: 'low' | 'medium' | 'high';
  metadata: Record<string, any>;
}

interface AuditEvents {
  authentication: ['login', 'logout', 'failed_login', 'password_change'];
  authorization: ['access_granted', 'access_denied', 'role_change'];
  data: ['create', 'read', 'update', 'delete', 'export'];
  system: ['backup', 'restore', 'configuration_change'];
}
```

---

## Incident Response

### **Security Incident Classification**
```yaml
Incident Levels:
  Level 1 - Low:
    Examples: Failed login attempts, minor policy violations
    Response: Automated logging, monitoring
    Escalation: None required
    
  Level 2 - Medium:
    Examples: Suspicious access patterns, configuration changes
    Response: Alert security team, investigate
    Escalation: Security manager within 4 hours
    
  Level 3 - High:
    Examples: Data breach attempt, system compromise
    Response: Immediate investigation, contain threat
    Escalation: Management within 1 hour
    
  Level 4 - Critical:
    Examples: Confirmed data breach, system takeover
    Response: Emergency response, external assistance
    Escalation: Immediate notification to all stakeholders
```

### **Response Procedures**
```yaml
Incident Response:
  Detection:
    - Automated monitoring alerts
    - User reports
    - Security scanning
    
  Containment:
    - Isolate affected systems
    - Preserve evidence
    - Prevent further damage
    
  Recovery:
    - Restore from clean backups
    - Apply security patches
    - Verify system integrity
    
  Lessons Learned:
    - Document incident
    - Update procedures
    - Training improvements
```

---

## Security Monitoring

### **Real-time Monitoring**
```yaml
Monitoring Stack:
  SIEM: ELK Stack
    - Log aggregation
    - Pattern detection
    - Alert generation
    
  Metrics: Prometheus + Grafana
    - Performance monitoring
    - Security metrics
    - Dashboard visualization
    
  Alerts:
    - Failed authentication attempts
    - Unusual access patterns
    - System vulnerabilities
    - Performance anomalies
```

### **Vulnerability Management**
```yaml
Vulnerability Scanning:
  Frequency: Weekly
  Tools:
    - OpenVAS: Network scanning
    - OWASP ZAP: Web application
    - Trivy: Container images
    
  Process:
    1. Automated scanning
    2. Risk assessment
    3. Prioritization
    4. Remediation
    5. Verification
    
  SLA:
    - Critical: 24 hours
    - High: 72 hours
    - Medium: 1 week
    - Low: Next maintenance window
```

---

## Business Continuity

### **Backup Security**
```yaml
Backup Strategy:
  Encryption: AES-256
  Storage: Multiple locations
  Testing: Monthly restoration tests
  Retention: 
    - Daily: 30 days
    - Weekly: 12 weeks
    - Monthly: 12 months
    - Yearly: 7 years
    
  Access Control:
    - Role-based permissions
    - Multi-person authorization
    - Audit logging
    - Physical security
```

### **Disaster Recovery**
```yaml
DR Planning:
  RTO: 4 hours (Recovery Time Objective)
  RPO: 1 hour (Recovery Point Objective)
  
  Procedures:
    - Automated failover
    - Data synchronization
    - Service restoration
    - Communication plan
    
  Testing:
    - Quarterly DR drills
    - Annual full simulation
    - Documentation updates
    - Staff training
```

---

## Security Training

### **Staff Security Awareness**
```yaml
Training Program:
  Frequency: Quarterly
  Topics:
    - Password security
    - Phishing recognition
    - Social engineering
    - Incident reporting
    
  Assessment:
    - Knowledge tests
    - Simulated attacks
    - Compliance tracking
    - Remedial training
```

---

## Related Documentation

- [Authentication & Authorization](../auth/overview.md)
- [VPS Infrastructure](../deployment/vps-infrastructure.md)
- [Data Models](../data/core-models.md)
- [API Security](../api/overview.md)

This security architecture provides comprehensive protection for the ACPN Lagos Portal while maintaining usability and performance for legitimate users.