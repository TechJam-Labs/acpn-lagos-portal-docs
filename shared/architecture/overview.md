# System Architecture Overview

> **Copyright © 2024 TechJamLabs**  
> Website: [www.techjamlabs.com](https://www.techjamlabs.com)  
> Phone: +234 201 330 9089 | +1 206 710 0170  
> **Authors:** Ben Adenle, Odinaka Solomon, Dare Oloruntoba

> **ACPN Lagos Portal - Comprehensive Digital Ecosystem Architecture**
> 
> A scalable, open-source VPS-based platform designed for pharmaceutical practice management, continuing professional development, and digital transformation.

---

## 🏗️ **Comprehensive System Architecture**

### **High-Level System Architecture**

```mermaid
graph TB
    subgraph "External Layer"
        EXT1[Users/Pharmacists]
        EXT2[Mobile Apps]
        EXT3[Third-party APIs]
        EXT4[GoMed Platform]
        EXT5[WhatsApp Business]
    end
    
    subgraph "CDN & Load Balancing"
        CDN[Cloudflare CDN]
        LB[Nginx Load Balancer]
        WAF[Web Application Firewall]
    end
    
    subgraph "API Gateway Layer"
        AG[API Gateway - Kong]
        AUTH[Authentication Service]
        RATE[Rate Limiting]
        LOG[Request Logging]
    end
    
    subgraph "Application Services Layer"
        USER[User Management Service]
        PHARM[Pharmacy Management Service]
        CPD[CPD Learning Service]
        EVENT[Events Management Service]
        MARKET[Marketplace Service]
        TRADE[Wholesale-Retail Trade Service]
        NOTIF[Notification Service]
        FILE[File Management Service]
    end
    
    subgraph "Integration Layer"
        BBB[BigBlueButton Integration]
        WHAT[WhatsApp Bot Integration]
        PAY[Payment Gateway Integration]
        SMS[SMS Gateway Integration]
        EMAIL[Email Service Integration]
    end
    
    subgraph "Data Layer"
        PG[(PostgreSQL - Primary DB)]
        MONGO[(MongoDB - Documents)]
        REDIS[(Redis - Cache/Queue)]
        ES[(Elasticsearch - Search)]
        FS[File Storage]
    end
    
    subgraph "Infrastructure Layer"
        DOCKER[Docker Swarm]
        MONITOR[Monitoring Stack]
        BACKUP[Backup System]
        SEC[Security Services]
    end
    
    EXT1 --> CDN
    EXT2 --> CDN
    EXT3 --> AG
    EXT4 --> AG
    EXT5 --> WHAT
    
    CDN --> LB
    LB --> WAF
    WAF --> AG
    
    AG --> AUTH
    AG --> USER
    AG --> PHARM
    AG --> CPD
    AG --> EVENT
    AG --> MARKET
    AG --> TRADE
    
    USER --> PG
    PHARM --> PG
    CPD --> PG
    CPD --> BBB
    EVENT --> PG
    MARKET --> PG
    MARKET --> MONGO
    TRADE --> PG
    TRADE --> MONGO
    
    NOTIF --> EMAIL
    NOTIF --> SMS
    NOTIF --> WHAT
    
    ALL_SERVICES --> REDIS
    ALL_SERVICES --> ES
    FILE --> FS
    
    DOCKER --> MONITOR
    DOCKER --> BACKUP
    DOCKER --> SEC
```

### **Microservices Architecture Detail**

```mermaid
graph LR
    subgraph "Frontend Applications"
        WEB[Web Application]
        PWA[Progressive Web App]
        MOBILE[Mobile App]
        ADMIN[Admin Dashboard]
    end
    
    subgraph "API Gateway & Security"
        KONG[Kong API Gateway]
        JWT[JWT Authentication]
        RBAC[Role-Based Access Control]
        OAUTH[OAuth2 Provider]
    end
    
    subgraph "Core Business Services"
        USER_SVC[User Service]
        PHARMACY_SVC[Pharmacy Service]
        CPD_SVC[CPD Service]
        EVENT_SVC[Event Service]
        MARKETPLACE_SVC[Marketplace Service]
        TRADE_SVC[Trade Service]
    end
    
    subgraph "Supporting Services"
        NOTIFICATION_SVC[Notification Service]
        FILE_SVC[File Service]
        ANALYTICS_SVC[Analytics Service]
        AUDIT_SVC[Audit Service]
        REPORTING_SVC[Reporting Service]
    end
    
    subgraph "External Integrations"
        BBB_INT[BigBlueButton]
        WHATSAPP_INT[WhatsApp Bot]
        PAYMENT_INT[Payment Gateway]
        PCN_INT[PCN Database]
        NAFDAC_INT[NAFDAC Registry]
    end
    
    WEB --> KONG
    PWA --> KONG
    MOBILE --> KONG
    ADMIN --> KONG
    
    KONG --> JWT
    KONG --> RBAC
    
    KONG --> USER_SVC
    KONG --> PHARMACY_SVC
    KONG --> CPD_SVC
    KONG --> EVENT_SVC
    KONG --> MARKETPLACE_SVC
    KONG --> TRADE_SVC
    
    USER_SVC --> NOTIFICATION_SVC
    PHARMACY_SVC --> FILE_SVC
    CPD_SVC --> BBB_INT
    EVENT_SVC --> PAYMENT_INT
    MARKETPLACE_SVC --> WHATSAPP_INT
    TRADE_SVC --> ANALYTICS_SVC
    
    ALL_SERVICES --> AUDIT_SVC
    ANALYTICS_SVC --> REPORTING_SVC
```

---

## 🏗️ **Architectural Philosophy**

### **Design Principles**
- **Scalability First**: Horizontal scaling capabilities using containerized microservices
- **Open Source Foundation**: Built entirely on open-source technologies for cost-effectiveness and flexibility
- **VPS-Optimized**: Designed for Virtual Private Server deployment with resource efficiency
- **Modular Architecture**: Loosely coupled services enabling independent scaling and maintenance
- **Security by Design**: Multi-layered security approach with role-based access control
- **Performance Optimized**: Caching strategies and optimized data flows for responsive user experience

### **Core Architectural Patterns**
- **Microservices Architecture**: Independent, deployable services
- **Event-Driven Communication**: Asynchronous messaging between services
- **API-First Design**: RESTful APIs with GraphQL for complex queries
- **Domain-Driven Design**: Business logic organized around pharmaceutical domains
- **CQRS Pattern**: Command Query Responsibility Segregation for optimal performance

---

## 🗄️ **Data Architecture & Database Design**

### **Database Architecture Overview**

```mermaid
graph TB
    subgraph "Application Layer"
        APP1[User Management App]
        APP2[Pharmacy Management App]
        APP3[CPD Learning App]
        APP4[Events Management App]
        APP5[Marketplace App]
        APP6[Trade Management App]
    end
    
    subgraph "Data Access Layer"
        ORM[TypeORM/Prisma]
        CACHE[Redis Cache Layer]
        QUEUE[Redis Queue]
        SEARCH[Elasticsearch]
    end
    
    subgraph "Primary Database - PostgreSQL"
        PG_USERS[(Users & Authentication)]
        PG_PHARMACY[(Pharmacy Data)]
        PG_CPD[(CPD Records)]
        PG_EVENTS[(Events & Registrations)]
        PG_TRANSACTIONS[(Financial Transactions)]
        PG_AUDIT[(Audit Logs)]
    end
    
    subgraph "Document Database - MongoDB"
        MONGO_PRODUCTS[(Product Catalogs)]
        MONGO_CONTENT[(Learning Content)]
        MONGO_LOGS[(Activity Logs)]
        MONGO_ANALYTICS[(Analytics Data)]
    end
    
    subgraph "File Storage"
        FS_DOCS[Documents]
        FS_MEDIA[Media Files]
        FS_CERTS[Certificates]
        FS_BACKUP[Backup Files]
    end
    
    APP1 --> ORM
    APP2 --> ORM
    APP3 --> ORM
    APP4 --> ORM
    APP5 --> ORM
    APP6 --> ORM
    
    ORM --> CACHE
    ORM --> PG_USERS
    ORM --> PG_PHARMACY
    ORM --> PG_CPD
    ORM --> PG_EVENTS
    ORM --> PG_TRANSACTIONS
    ORM --> PG_AUDIT
    
    ORM --> MONGO_PRODUCTS
    ORM --> MONGO_CONTENT
    ORM --> MONGO_LOGS
    ORM --> MONGO_ANALYTICS
    
    APP1 --> FS_DOCS
    APP3 --> FS_MEDIA
    APP4 --> FS_CERTS
    QUEUE --> FS_BACKUP
    
    SEARCH --> PG_USERS
    SEARCH --> MONGO_PRODUCTS
    SEARCH --> MONGO_CONTENT
```

### **Core Data Relationships**

```mermaid
erDiagram
    USER ||--o{ USER_PROFILE : has
    USER ||--o{ PHARMACY_OWNERSHIP : owns
    USER ||--o{ CPD_ENROLLMENT : enrolls
    USER ||--o{ EVENT_REGISTRATION : registers
    USER ||--o{ MARKETPLACE_ORDER : places
    
    PHARMACY ||--o{ PHARMACY_STAFF : employs
    PHARMACY ||--o{ INVENTORY : maintains
    PHARMACY ||--o{ TRADE_RELATIONSHIP : participates
    
    CPD_COURSE ||--o{ CPD_ENROLLMENT : has
    CPD_COURSE ||--o{ CPD_ASSESSMENT : includes
    CPD_ENROLLMENT ||--o{ CPD_CERTIFICATE : generates
    
    EVENT ||--o{ EVENT_REGISTRATION : accepts
    EVENT ||--o{ EVENT_SESSION : contains
    EVENT_REGISTRATION ||--o{ PAYMENT : requires
    
    MARKETPLACE_REQUEST ||--o{ MARKETPLACE_OFFER : receives
    MARKETPLACE_OFFER ||--o{ MARKETPLACE_ORDER : becomes
    
    WHOLESALE_PRODUCT ||--o{ TRADE_ORDER_ITEM : included_in
    TRADE_ORDER ||--o{ TRADE_ORDER_ITEM : contains
    TRADE_ORDER ||--o{ TRADE_PAYMENT : requires
    
    USER {
        string id PK
        string email
        string role
        datetime created_at
        datetime updated_at
    }
    
    PHARMACY {
        string id PK
        string name
        string license_number
        string address
        string owner_id FK
        datetime created_at
    }
    
    CPD_COURSE {
        string id PK
        string title
        int credit_hours
        string content_url
        datetime created_at
    }
    
    EVENT {
        string id PK
        string title
        datetime start_date
        datetime end_date
        string location
        decimal price
    }
    
    MARKETPLACE_REQUEST {
        string id PK
        string title
        string description
        int quantity
        string urgency
        string requester_id FK
    }
    
    TRADE_ORDER {
        string id PK
        string buyer_id FK
        string seller_id FK
        decimal total_amount
        string status
        datetime created_at
    }
```

---

## 🔒 **Security Architecture**

### **Multi-Layer Security Model**

```mermaid
graph TB
    subgraph "External Security"
        CF[Cloudflare Protection]
        DDOS[DDoS Protection]
        WAF[Web Application Firewall]
        GEO[Geo-blocking]
    end
    
    subgraph "Network Security"
        FW[UFW Firewall]
        FAIL2BAN[Fail2ban]
        VPN[VPN Access]
        WHITELIST[IP Whitelisting]
    end
    
    subgraph "Application Security"
        JWT_AUTH[JWT Authentication]
        RBAC_SYS[RBAC System]
        OAUTH[OAuth2/OIDC]
        2FA[Two-Factor Auth]
        SESSION[Session Management]
    end
    
    subgraph "Data Security"
        ENCRYPT[Data Encryption]
        HASH[Password Hashing]
        BACKUP_ENC[Backup Encryption]
        KEY_MGMT[Key Management]
    end
    
    subgraph "API Security"
        RATE_LIMIT[Rate Limiting]
        API_KEY[API Key Management]
        CORS[CORS Policy]
        CSRF[CSRF Protection]
    end
    
    subgraph "Monitoring & Compliance"
        AUDIT_LOG[Audit Logging]
        SIEM[Security Monitoring]
        COMPLIANCE[Compliance Checks]
        INCIDENT[Incident Response]
    end
    
    CF --> FW
    DDOS --> FW
    WAF --> FW
    GEO --> FW
    
    FW --> JWT_AUTH
    FAIL2BAN --> JWT_AUTH
    
    JWT_AUTH --> RBAC_SYS
    OAUTH --> 2FA
    SESSION --> ENCRYPT
    
    RATE_LIMIT --> API_KEY
    CORS --> CSRF
    
    ALL_LAYERS --> AUDIT_LOG
    AUDIT_LOG --> SIEM
    SIEM --> COMPLIANCE
    COMPLIANCE --> INCIDENT
```

### **Authentication & Authorization Flow**

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant AG as API Gateway
    participant AS as Auth Service
    participant DB as Database
    participant APP as Application Service
    
    U->>FE: Login Request
    FE->>AG: POST /auth/login
    AG->>AS: Validate Credentials
    AS->>DB: Check User
    DB-->>AS: User Data
    AS->>AS: Validate Password
    AS->>AS: Generate JWT
    AS->>AS: Log Audit Event
    AS-->>AG: JWT Token + Refresh Token
    AG-->>FE: Authentication Response
    FE->>FE: Store Tokens
    
    U->>FE: Access Protected Resource
    FE->>AG: Request with JWT
    AG->>AG: Validate JWT
    AG->>AS: Check Permissions
    AS->>DB: Get User Roles
    DB-->>AS: Role Data
    AS-->>AG: Authorization Result
    AG->>APP: Forward Request
    APP-->>AG: Response
    AG-->>FE: Protected Data
```

---

## 🔗 **Integration Architecture**

### **External Integrations Overview**

```mermaid
graph TB
    subgraph "ACPN Portal Core"
        CORE[Core Application]
        API_GW[API Gateway]
        WEBHOOK[Webhook Handler]
        QUEUE[Message Queue]
    end
    
    subgraph "Learning & CPD Integrations"
        BBB[BigBlueButton Server]
        SCORM[SCORM Player]
        CERT[Certificate Generator]
        PCN_API[PCN Database API]
    end
    
    subgraph "Communication Integrations"
        WHATSAPP[WhatsApp Business API]
        EMAIL_SVC[Email Service - SendGrid]
        SMS_SVC[SMS Gateway]
        PUSH[Push Notification Service]
    end
    
    subgraph "Financial Integrations"
        PAYSTACK[Paystack Payment]
        FLUTTERWAVE[Flutterwave Payment]
        BANK_API[Bank APIs]
        INVOICE[Invoice Generator]
    end
    
    subgraph "Marketplace Integrations"
        GOMED[GoMed Platform]
        NAFDAC[NAFDAC Registry]
        CUSTOMS[Nigeria Customs]
        LOGISTICS[Logistics Partners]
    end
    
    subgraph "Monitoring & Analytics"
        PROMETHEUS[Prometheus]
        GRAFANA[Grafana]
        ELK[ELK Stack]
        SENTRY[Sentry Error Tracking]
    end
    
    CORE --> API_GW
    API_GW --> BBB
    API_GW --> WHATSAPP
    API_GW --> PAYSTACK
    API_GW --> GOMED
    
    WEBHOOK --> QUEUE
    QUEUE --> CORE
    
    CORE --> PCN_API
    CORE --> EMAIL_SVC
    CORE --> SMS_SVC
    CORE --> NAFDAC
    
    PROMETHEUS --> GRAFANA
    CORE --> ELK
    CORE --> SENTRY
```

### **WhatsApp Bot Integration Flow**

```mermaid
sequenceDiagram
    participant GM as GoMed
    participant API as ACPN API
    participant WB as WhatsApp Bot
    participant P as Pharmacy
    participant AI as AI Parser
    participant DB as Database
    
    GM->>API: Product Availability Request
    API->>DB: Get Nearby Pharmacies
    DB-->>API: Pharmacy List
    API->>WB: Generate WhatsApp Messages
    WB->>P: Send Availability Query
    
    P->>WB: Respond with Availability
    WB->>AI: Parse Response
    AI->>API: Structured Data
    API->>DB: Store Response
    API->>GM: Aggregated Results
    
    Note over GM,DB: Real-time inventory sync
    Note over WB,AI: Natural language processing
```

---

## 🖥️ **VPS Infrastructure Architecture**

### **Server Configuration**
```yaml
Primary VPS Configuration:
  CPU: 8 cores (scalable to 16 cores)
  RAM: 32GB (scalable to 64GB)
  Storage: 500GB SSD (expandable with block storage)
  Network: 1Gbps dedicated bandwidth
  OS: Ubuntu Server 22.04 LTS
  
Load Balancer VPS:
  CPU: 4 cores
  RAM: 8GB
  Network: Load balancing across multiple application servers
  
Database VPS (separate for security):
  CPU: 8 cores
  RAM: 32GB
  Storage: 1TB SSD with automated backups
```

### **Infrastructure Deployment Architecture**

```mermaid
graph TB
    subgraph "Internet Layer"
        USERS[Users/Clients]
        MOBILE[Mobile Apps]
        API_CLIENTS[API Clients]
    end
    
    subgraph "CDN & Edge Layer"
        CLOUDFLARE[Cloudflare CDN]
        EDGE_CACHE[Edge Caching]
        DNS[DNS Resolution]
    end
    
    subgraph "Load Balancing Layer"
        LB1[Primary Load Balancer]
        LB2[Secondary Load Balancer]
        HEALTH[Health Checks]
    end
    
    subgraph "VPS Infrastructure"
        subgraph "Production Cluster"
            VPS1[VPS-01: Web Servers]
            VPS2[VPS-02: API Services]
            VPS3[VPS-03: Background Jobs]
            VPS4[VPS-04: File Storage]
        end
        
        subgraph "Database Cluster"
            VPS5[VPS-05: PostgreSQL Primary]
            VPS6[VPS-06: PostgreSQL Replica]
            VPS7[VPS-07: MongoDB Cluster]
            VPS8[VPS-08: Redis Cluster]
        end
        
        subgraph "Integration Services"
            VPS9[VPS-09: BigBlueButton]
            VPS10[VPS-10: Monitoring Stack]
            VPS11[VPS-11: Backup Services]
        end
    end
    
    subgraph "External Services"
        WHATSAPP_API[WhatsApp Business API]
        PAYMENT_GW[Payment Gateways]
        SMS_GW[SMS Gateway]
        EMAIL_SVC[Email Service]
    end
    
    USERS --> CLOUDFLARE
    MOBILE --> CLOUDFLARE
    API_CLIENTS --> CLOUDFLARE
    
    CLOUDFLARE --> LB1
    CLOUDFLARE --> LB2
    
    LB1 --> VPS1
    LB1 --> VPS2
    LB2 --> VPS1
    LB2 --> VPS2
    
    VPS1 --> VPS5
    VPS2 --> VPS5
    VPS3 --> VPS6
    
    VPS2 --> VPS7
    VPS3 --> VPS8
    
    VPS2 --> WHATSAPP_API
    VPS2 --> PAYMENT_GW
    VPS3 --> SMS_GW
    VPS3 --> EMAIL_SVC
    
    VPS10 --> VPS1
    VPS10 --> VPS2
    VPS10 --> VPS3
    VPS11 --> VPS5
    VPS11 --> VPS6
```

### **Container Orchestration Architecture**

```mermaid
graph TB
    subgraph "Docker Swarm Cluster"
        subgraph "Manager Nodes"
            MGR1[Manager-01]
            MGR2[Manager-02]
            MGR3[Manager-03]
        end
        
        subgraph "Worker Nodes"
            WORK1[Worker-01]
            WORK2[Worker-02]
            WORK3[Worker-03]
            WORK4[Worker-04]
            WORK5[Worker-05]
        end
        
        subgraph "Service Stack"
            WEB_SVC[Web Service Stack]
            API_SVC[API Service Stack]
            BG_SVC[Background Service Stack]
            DB_SVC[Database Service Stack]
        end
        
        subgraph "Storage & Networking"
            OVERLAY[Overlay Network]
            VOLUMES[Docker Volumes]
            SECRETS[Docker Secrets]
            CONFIGS[Docker Configs]
        end
    end
    
    subgraph "Service Discovery"
        CONSUL[Consul Service Discovery]
        DNS_SD[DNS Service Discovery]
        LOAD_BAL[Internal Load Balancing]
    end
    
    subgraph "Monitoring & Logging"
        PROMETHEUS[Prometheus]
        GRAFANA[Grafana]
        LOKI[Loki Logging]
        ALERTMANAGER[Alert Manager]
    end
    
    MGR1 --> WORK1
    MGR1 --> WORK2
    MGR2 --> WORK3
    MGR2 --> WORK4
    MGR3 --> WORK5
    
    WEB_SVC --> WORK1
    API_SVC --> WORK2
    API_SVC --> WORK3
    BG_SVC --> WORK4
    DB_SVC --> WORK5
    
    ALL_SERVICES --> OVERLAY
    ALL_SERVICES --> VOLUMES
    ALL_SERVICES --> SECRETS
    
    CONSUL --> DNS_SD
    DNS_SD --> LOAD_BAL
    
    PROMETHEUS --> GRAFANA
    LOKI --> GRAFANA
    PROMETHEUS --> ALERTMANAGER
```

### **Service Mesh & Communication**

```mermaid
graph LR
    subgraph "Frontend Services"
        WEB[Web Application]
        PWA[Progressive Web App]
        ADMIN[Admin Dashboard]
    end
    
    subgraph "API Gateway Layer"
        KONG[Kong API Gateway]
        AUTH[Authentication Service]
        RATE_LIMITER[Rate Limiter]
    end
    
    subgraph "Business Logic Services"
        USER_SVC[User Service]
        PHARMACY_SVC[Pharmacy Service]
        CPD_SVC[CPD Service]
        EVENT_SVC[Event Service]
        MARKETPLACE_SVC[Marketplace Service]
        TRADE_SVC[Trade Service]
    end
    
    subgraph "Data Services"
        USER_DB[User Database]
        PHARMACY_DB[Pharmacy Database]
        CPD_DB[CPD Database]
        EVENT_DB[Event Database]
        MARKETPLACE_DB[Marketplace Database]
        TRADE_DB[Trade Database]
    end
    
    subgraph "Supporting Services"
        NOTIFICATION[Notification Service]
        FILE_STORAGE[File Storage Service]
        ANALYTICS[Analytics Service]
        AUDIT[Audit Service]
        CACHE[Cache Service]
        QUEUE[Queue Service]
    end
    
    WEB --> KONG
    PWA --> KONG
    ADMIN --> KONG
    
    KONG --> AUTH
    KONG --> RATE_LIMITER
    
    KONG --> USER_SVC
    KONG --> PHARMACY_SVC
    KONG --> CPD_SVC
    KONG --> EVENT_SVC
    KONG --> MARKETPLACE_SVC
    KONG --> TRADE_SVC
    
    USER_SVC --> USER_DB
    PHARMACY_SVC --> PHARMACY_DB
    CPD_SVC --> CPD_DB
    EVENT_SVC --> EVENT_DB
    MARKETPLACE_SVC --> MARKETPLACE_DB
    TRADE_SVC --> TRADE_DB
    
    ALL_BUSINESS_SERVICES --> NOTIFICATION
    ALL_BUSINESS_SERVICES --> FILE_STORAGE
    ALL_BUSINESS_SERVICES --> ANALYTICS
    ALL_BUSINESS_SERVICES --> AUDIT
    ALL_BUSINESS_SERVICES --> CACHE
    ALL_BUSINESS_SERVICES --> QUEUE
```

### **Network Architecture**
```
Internet Layer
    ↓
[Cloudflare CDN] → [Edge Caching] → [DDoS Protection]
    ↓                        ↓                ↓
[DNS Resolution]     [WAF Filtering]   [Geo-blocking]
    ↓                        ↓                ↓
Load Balancer Layer
    ↓
[Primary LB (Nginx)] ↔ [Secondary LB (HAProxy)]
    ↓                        ↓
[Health Checks]      [SSL Termination]
    ↓                        ↓
Application Layer
    ↓
[Docker Swarm Cluster] → [Service Discovery] → [Internal Load Balancing]
    ↓                        ↓                        ↓
[Web Services]       [API Services]           [Background Services]
    ↓                        ↓                        ↓
Data Layer
    ↓
[PostgreSQL Cluster] + [MongoDB Cluster] + [Redis Cluster] + [File Storage]
    ↓                        ↓                   ↓              ↓
[Primary DB]         [Document Store]    [Cache/Queue]   [Media Files]
[Replica DB]         [Analytics Data]    [Sessions]      [Documents]
```

---

## 🔧 **Technology Stack**

### **Core Infrastructure**
```yaml
Operating System: Ubuntu Server 22.04 LTS
Containerization: Docker CE + Docker Swarm
Web Server: Nginx (reverse proxy + load balancer)
SSL/TLS: Let's Encrypt with automated renewal
Firewall: UFW (Uncomplicated Firewall) + fail2ban
Process Management: systemd + PM2 for Node.js
```

### **Application Layer**
```yaml
Backend Runtime: Node.js 20 LTS
Framework: Express.js with TypeScript
API Gateway: Kong Community Edition
Authentication: JWT + Passport.js
Session Management: Redis-based sessions
File Uploads: Multer with local storage + backup
```

### **Database Layer**
```yaml
Primary Database: PostgreSQL 15 (user data, transactions)
Document Store: MongoDB 7.0 (content, publications)
Cache Layer: Redis 7.0 (session, cache, job queue)
Search Engine: Elasticsearch 8.0 (full-text search)
Queue System: Redis Bull (background jobs)
```

### **Frontend Stack**
```yaml
Framework: Next.js 15 with React 19
Language: TypeScript
Styling: Tailwind CSS 4
State Management: Zustand + React Query
PWA: Next.js PWA with offline support
Build Tool: Turbopack
```

---

## 🎓 **CPD & Learning Management Architecture**

### **BigBlueButton Integration**
```yaml
BBB Server Configuration:
  Version: BigBlueButton 2.7
  Installation: Dedicated VPS instance
  CPU: 8 cores (minimum)
  RAM: 16GB (recommended 32GB)
  Network: High bandwidth for video streaming
  
Integration Components:
  BBB API: RESTful API integration
  Recording Processing: Automated post-session processing
  Content Delivery: Nginx for recorded session delivery
  Authentication: ACPN portal SSO integration
```

### **Learning Platform Components**
```yaml
Course Management Service:
  Purpose: Course creation, management, and delivery
  Database: PostgreSQL for structured course data
  Files: Local storage for course materials
  Features: SCORM compliance, progress tracking
  
Assessment Engine:
  Purpose: Quizzes, assignments, and evaluations
  Database: PostgreSQL for assessment data
  Features: Automated grading, certificate generation
  Integration: CPD credit calculation and tracking
  
Content Delivery Network:
  Purpose: Fast delivery of learning materials
  Technology: Nginx with caching
  Features: Video streaming, document delivery
  Optimization: Gzip compression, browser caching
```

### **CPD Credit System**
```yaml
Credit Tracking Service:
  Database: PostgreSQL for credit records
  Features: Automatic credit calculation
  Compliance: PCN requirements integration
  Reporting: Individual and aggregate analytics
  
Certificate Generation:
  Technology: PDFKit for dynamic certificates
  Storage: Local file system with backups
  Features: Digital signatures, verification codes
  Integration: Email delivery system
```

---

## 🏪 **Multi-Store Management Architecture**

### **Store Hierarchy System**
```yaml
Ownership Model:
  Primary Pharmacist: Can own multiple stores
  Resident Pharmacist: Assigned to specific store
  Staff Members: Store-specific access only
  
Store Registration:
  GPS Verification: Geolocation validation
  PCN Integration: Pharmacist license verification
  Document Management: Store permits and certificates
  Compliance Tracking: Regulatory requirement monitoring
```

### **Multi-Store Data Architecture**
```yaml
Database Design:
  Users Table: Global user accounts
  Pharmacies Table: Individual store records
  Store_Pharmacists: Many-to-many relationship
  Store_Staff: Store-specific staff assignments
  Inventory: Store-specific product tracking
  
Access Control:
  Role-Based: Different permissions per store
  Data Isolation: Store-specific data access
  Cross-Store: Owner-level aggregated views
  Audit Trail: All store operations logged
```

### **Inventory Management**
```yaml
Store-Level Inventory:
  Real-time stock tracking per store
  Automated reorder notifications
  Cross-store transfer capabilities
  Centralized procurement options
  
Integration Points:
  GoMed Sync: Store-specific product availability
  WhatsApp Bot: Store inventory queries
  Analytics: Cross-store performance comparison
  Compliance: Controlled substance tracking
```

---

## 🔌 **Integration Architecture**

### **GoMed Platform Integration**
```yaml
Integration Type: RESTful API
Authentication: OAuth 2.0 + API keys
Data Sync: Real-time inventory synchronization
Store Mapping: Each ACPN store → GoMed seller account
Order Flow: ACPN → GoMed → Customer fulfillment

Publication Sharing:
  Permission System: ACPN super admin controlled
  Content Types: Guidelines, regulations, updates
  Access Control: Role-based GoMed user access
  Sync Frequency: Real-time for critical updates
```

### **WhatsApp Bot (WhatBot) Architecture**
```yaml
Bot Framework: Venom-bot (open source)
Message Queue: Redis Bull for message processing
Business Logic: Separate microservice
Database: MongoDB for conversation logs
Response Engine: AI-powered query understanding

Features:
  Product Availability: Real-time store inventory
  CPD Content: Daily educational content delivery
  Order Updates: Purchase confirmation and tracking
  Emergency Alerts: Critical regulatory notifications
```

### **BigBlueButton Integration Flow**
```yaml
Session Management:
  Create Session: ACPN portal → BBB API
  User Authentication: SSO token validation
  Join Session: Direct BBB room access
  Recording: Automated post-session processing
  
Content Pipeline:
  Live Session → Recording → Processing → CDN → Portal
  Metadata: Session details, attendance, analytics
  Integration: CPD credit automatic assignment
```

---

## 🔐 **Security Architecture**

### **Multi-Layer Security**
```yaml
Network Security:
  Firewall: UFW with strict ingress rules
  DDoS Protection: Cloudflare + rate limiting
  VPN Access: WireGuard for admin access
  SSL/TLS: Let's Encrypt with HSTS headers
  
Application Security:
  Authentication: JWT with refresh tokens
  Authorization: RBAC with store-level permissions
  Input Validation: Joi schema validation
  SQL Injection: Parameterized queries only
  XSS Protection: Content Security Policy headers
  
Data Security:
  Encryption at Rest: PostgreSQL + MongoDB encryption
  Encryption in Transit: TLS 1.3 minimum
  Sensitive Data: Environment variables + Docker secrets
  Audit Logging: All sensitive operations logged
```

### **Backup & Recovery**
```yaml
Database Backups:
  Frequency: Daily automated backups
  Retention: 30 days local, 90 days off-site
  Testing: Monthly restore verification
  Recovery Time: < 4 hours for full restore
  
File System Backups:
  User Uploads: Real-time replication
  Application Code: Git-based version control
  Configuration: Infrastructure as code
  
Disaster Recovery:
  RTO (Recovery Time Objective): 4 hours
  RPO (Recovery Point Objective): 1 hour
  Failover: Automated secondary VPS activation
```

---

## 📊 **Data Flow Architecture**

### **Core Data Flows**

#### **User Registration & Multi-Store Setup**
```
User Registration → PCN Verification → Store Registration 
→ GPS Verification → Resident Pharmacist Assignment 
→ GoMed Account Creation → WhatsApp Bot Setup
```

#### **CPD Learning Flow**
```
Course Enrollment → BigBlueButton Session Creation 
→ Live Learning/Recording → Assessment Completion 
→ CPD Credit Assignment → Certificate Generation 
→ WhatsApp Notification
```

#### **Marketplace Order Flow**
```
Customer Order (GoMed) → WhatsApp Product Query 
→ Store Inventory Check → Availability Response 
→ Order Confirmation → Fulfillment Tracking 
→ Completion Notification
```

#### **Publication Distribution**
```
Content Creation → ACPN Admin Approval 
→ Permission Tagging → WhatsApp Distribution 
→ GoMed Sharing (if permitted) → Engagement Analytics
```

### **Real-Time Data Synchronization**
```yaml
Inventory Sync:
  ACPN Store Updates → GoMed Platform
  Frequency: Real-time via webhooks
  Conflict Resolution: Last-write-wins with audit trail
  
CPD Progress:
  Learning Activity → Real-time Progress Update
  Credit Calculation → Automatic Transcript Update
  Certificate Generation → Immediate Availability
  
WhatsApp Interactions:
  Query Reception → Context Analysis → Database Lookup
  Response Generation → Delivery Confirmation → Analytics
```

---

## 🚀 **Scalability Design**

### **Horizontal Scaling Strategy**
```yaml
Application Scaling:
  Load Balancer: Nginx with round-robin
  App Servers: Docker Swarm auto-scaling
  Database: Read replicas for query optimization
  Cache: Redis cluster for distributed caching
  
BigBlueButton Scaling:
  Multiple BBB Instances: Load-balanced across servers
  Recording Processing: Distributed across worker nodes
  Content Delivery: CDN for optimized media delivery
  
WhatsApp Bot Scaling:
  Message Queue: Redis Bull with multiple workers
  Conversation State: Distributed across Redis cluster
  Response Generation: Parallel processing workers
```

### **Performance Optimization**
```yaml
Database Optimization:
  Indexing: Strategic index placement
  Query Optimization: Explain plan analysis
  Connection Pooling: PgBouncer for PostgreSQL
  Partitioning: Time-based table partitioning
  
Caching Strategy:
  Redis: Session, frequently accessed data
  Nginx: Static content caching
  Application: In-memory caching for hot paths
  CDN: Global content distribution
  
Resource Management:
  CPU: Auto-scaling based on utilization
  Memory: Optimized container resource limits
  Storage: Automated cleanup and archival
  Network: Bandwidth monitoring and optimization
```

---

## 🔄 **Monitoring & Observability**

### **System Monitoring**
```yaml
Infrastructure Monitoring:
  Tool: Prometheus + Grafana
  Metrics: CPU, memory, disk, network
  Alerts: Slack/email for critical issues
  Dashboards: Real-time system status
  
Application Monitoring:
  APM: Custom metrics collection
  Error Tracking: Centralized error logging
  Performance: Response time monitoring
  User Analytics: Feature usage tracking
  
Database Monitoring:
  PostgreSQL: pg_stat_statements analysis
  MongoDB: Performance profiling
  Redis: Memory usage and hit rates
  Elasticsearch: Cluster health monitoring
```

### **Logging Strategy**
```yaml
Centralized Logging:
  Tool: ELK Stack (Elasticsearch, Logstash, Kibana)
  Application Logs: Structured JSON logging
  Access Logs: Nginx request logging
  Security Logs: Authentication and authorization events
  Audit Logs: All sensitive operations
  
Log Management:
  Retention: 90 days for application logs
  Security Logs: 1 year retention
  Rotation: Daily log rotation
  Compression: Gzip compression for archived logs
```

---

## 🔄 **Development & Deployment Workflow**

### **CI/CD Pipeline**
```yaml
Source Control: Git with feature branching
CI/CD Tool: GitHub Actions (free tier)
Testing: Automated unit, integration, and e2e tests
Building: Docker image creation and registry push
Deployment: Rolling updates via Docker Swarm
Rollback: Automatic rollback on deployment failure
```

### **Environment Management**
```yaml
Development: Local Docker Compose environment
Staging: Scaled-down production replica
Production: Full VPS cluster deployment
Configuration: Environment-specific Docker secrets
Database: Separate database instances per environment
```

---

## 📈 **Future Scalability Considerations**

### **Growth Planning**
```yaml
User Scaling:
  Target: 10,000+ registered pharmacists
  Stores: 5,000+ pharmacy locations
  Concurrent Users: 1,000+ simultaneous
  
Content Scaling:
  Courses: 1,000+ CPD courses
  Publications: 10,000+ documents
  Videos: Terabytes of recorded content
  
Geographic Scaling:
  Multi-State: Additional state chapters
  International: West African expansion
  Compliance: Multi-jurisdiction regulatory support
```

### **Technology Evolution**
```yaml
Microservices Evolution:
  Current: Monolithic with service separation
  Future: Full microservices architecture
  Communication: Event-driven architecture
  
Container Orchestration:
  Current: Docker Swarm
  Future: Kubernetes migration option
  Cloud: Multi-cloud deployment capability
  
Performance Enhancement:
  Caching: Advanced caching strategies
  Database: Sharding for massive scale
  CDN: Global content delivery optimization
```

---

This architecture provides a robust, scalable foundation for the ACPN Lagos Portal, supporting all current requirements while maintaining flexibility for future growth and enhancement.