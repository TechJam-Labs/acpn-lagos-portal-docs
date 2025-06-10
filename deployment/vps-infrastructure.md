# VPS Infrastructure Setup & Management

> **Open-Source, Scalable VPS Architecture for ACPN Lagos Portal**
> 
> Comprehensive guide for deploying and managing a cost-effective, scalable infrastructure using Virtual Private Servers and open-source technologies.

---

## 🎯 **Infrastructure Philosophy**

### **Design Principles**
- **Cost Optimization**: Maximum functionality with minimal infrastructure costs
- **Open Source First**: Leveraging community-driven technologies
- **Horizontal Scalability**: Easy expansion as user base grows
- **High Availability**: Redundancy and failover capabilities
- **Security by Design**: Multi-layered security approach
- **Performance Focus**: Optimized for speed and responsiveness

### **VPS vs Cloud Comparison**
```yaml
VPS Advantages:
  Cost Effectiveness: Predictable monthly costs
  Control: Full server control and customization
  Performance: Dedicated resources without noisy neighbors
  Compliance: Easier data locality and regulatory compliance
  Simplicity: Straightforward deployment and management
  
Traditional Cloud Drawbacks:
  Cost Unpredictability: Variable pricing based on usage
  Vendor Lock-in: Proprietary services and APIs
  Complexity: Steep learning curve for cloud-native services
  Over-engineering: Complex solutions for simple needs
```

---

## 🖥️ **Server Architecture & Specifications**

### **Production Environment Setup**
```yaml
Primary Application Server (VPS-APP-01):
  Provider: DigitalOcean/Linode/Vultr
  CPU: 8 vCPUs (Intel or AMD)
  RAM: 32GB
  Storage: 500GB NVMe SSD
  Network: 1Gbps bandwidth
  OS: Ubuntu Server 22.04 LTS
  Purpose: Main application hosting
  
Database Server (VPS-DB-01):
  Provider: Same as application server
  CPU: 8 vCPUs
  RAM: 32GB
  Storage: 1TB NVMe SSD
  Network: 1Gbps bandwidth
  OS: Ubuntu Server 22.04 LTS
  Purpose: Database hosting (PostgreSQL, MongoDB, Redis)
  
Load Balancer (VPS-LB-01):
  Provider: Same as application server
  CPU: 4 vCPUs
  RAM: 8GB
  Storage: 100GB SSD
  Network: 1Gbps bandwidth
  OS: Ubuntu Server 22.04 LTS
  Purpose: Traffic distribution and SSL termination
  
BigBlueButton Server (VPS-BBB-01):
  Provider: Same as application server
  CPU: 8 vCPUs (minimum for BBB)
  RAM: 32GB (64GB recommended)
  Storage: 1TB SSD (for recordings)
  Network: 1Gbps bandwidth
  OS: Ubuntu Server 20.04 LTS (BBB requirement)
  Purpose: Virtual classroom hosting
```

### **Development Environment Setup**
```yaml
Development Server (VPS-DEV-01):
  CPU: 4 vCPUs
  RAM: 16GB
  Storage: 250GB SSD
  Purpose: Development, testing, and staging
  
Monitoring Server (VPS-MON-01):
  CPU: 2 vCPUs
  RAM: 8GB
  Storage: 200GB SSD
  Purpose: System monitoring and log aggregation
```

---

## 🐳 **Containerization with Docker**

### **Docker Infrastructure**
```yaml
Container Platform:
  Engine: Docker CE (Community Edition)
  Orchestration: Docker Swarm Mode
  Registry: Private Docker Registry
  Networks: Overlay networks for service communication
  Secrets: Docker secrets for sensitive data management
  
Container Services:
  Web Application: Next.js frontend + Node.js backend
  API Gateway: Kong Community Edition
  Databases: PostgreSQL, MongoDB, Redis containers
  Cache: Redis cluster for distributed caching
  Queue: Redis Bull for job processing
  Search: Elasticsearch for full-text search
```

### **Docker Swarm Configuration**
```yaml
Swarm Setup:
  Manager Nodes: 3 (for high availability)
  Worker Nodes: 5+ (for horizontal scaling)
  Network: Encrypted overlay networks
  Load Balancing: Built-in service discovery and load balancing
  Rolling Updates: Zero-downtime deployments
  
Service Scaling:
  Web Services: Auto-scaling based on CPU/memory usage
  Database Services: Vertical scaling with resource limits
  Worker Services: Horizontal scaling for job processing
  Cache Services: Cluster scaling for distributed caching
```

### **Docker Compose Configuration Example**
```yaml
version: '3.8'
services:
  web:
    image: acpn-web:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    networks:
      - acpn-network
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/acpn
    secrets:
      - jwt_secret
      - database_password
  
  api:
    image: acpn-api:latest
    deploy:
      replicas: 3
    networks:
      - acpn-network
    ports:
      - "8000:8000"
    depends_on:
      - database
      - redis
  
  database:
    image: postgres:15
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.database == true
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=acpn
      - POSTGRES_USER=acpn_user
    secrets:
      - postgres_password
    networks:
      - acpn-network
  
  redis:
    image: redis:7-alpine
    deploy:
      replicas: 1
    volumes:
      - redis_data:/data
    networks:
      - acpn-network
  
  nginx:
    image: nginx:alpine
    deploy:
      replicas: 2
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - /etc/letsencrypt:/etc/letsencrypt
    networks:
      - acpn-network
    depends_on:
      - web
      - api

volumes:
  postgres_data:
  redis_data:

networks:
  acpn-network:
    driver: overlay
    encrypted: true

secrets:
  jwt_secret:
    external: true
  database_password:
    external: true
  postgres_password:
    external: true
```

---

## 🔧 **Open Source Technology Stack**

### **Operating System & Base Infrastructure**
```yaml
Operating System:
  Distribution: Ubuntu Server 22.04 LTS
  Kernel: Linux 5.15+ with security updates
  Package Manager: APT with automated security updates
  Init System: systemd for service management
  
System Services:
  SSH: OpenSSH with key-based authentication
  Firewall: UFW (Uncomplicated Firewall)
  Intrusion Detection: fail2ban
  Log Management: rsyslog with logrotate
  Time Synchronization: chrony (NTP)
```

### **Web Server & Reverse Proxy**
```yaml
Nginx Configuration:
  Version: Nginx 1.22+ (latest stable)
  Purpose: Reverse proxy, load balancer, SSL termination
  Features: HTTP/2, gzip compression, caching
  Security: Rate limiting, DDoS protection
  
Load Balancing:
  Algorithm: Round-robin with health checks
  Sticky Sessions: IP hash for session persistence
  Failover: Automatic backend health monitoring
  SSL: Let's Encrypt certificates with auto-renewal
```

### **Database Systems**
```yaml
PostgreSQL 15:
  Purpose: Primary relational database
  Features: ACID compliance, advanced indexing, JSON support
  Backup: pg_dump with automated daily backups
  Replication: Streaming replication for read replicas
  
MongoDB 7.0:
  Purpose: Document storage for content and publications
  Features: GridFS for file storage, full-text search
  Backup: mongodump with automated backup rotation
  Clustering: Replica set configuration
  
Redis 7.0:
  Purpose: Caching, session storage, job queue
  Features: Persistence, clustering, pub/sub
  Configuration: Memory optimization, eviction policies
  Monitoring: Redis Insight for performance monitoring
  
Elasticsearch 8.0:
  Purpose: Full-text search and analytics
  Features: Distributed search, aggregations, analysis
  Configuration: Single-node for development, cluster for production
  Backup: Snapshot and restore to cloud storage
```

### **Application Runtime & Framework**
```yaml
Node.js 20 LTS:
  Runtime: V8 JavaScript engine
  Package Manager: npm with lock files
  Process Manager: PM2 for clustering and monitoring
  Memory Management: Garbage collection optimization
  
Express.js:
  Framework: Web application framework
  Middleware: CORS, helmet, rate limiting
  Authentication: JWT with Passport.js
  Validation: Joi schema validation
  
TypeScript:
  Language: Type-safe JavaScript
  Compilation: ts-node for development, compiled for production
  Configuration: Strict mode with comprehensive type checking
```

---

## 🔴 **BigBlueButton Integration**

### **BigBlueButton Server Setup**
```yaml
BBB Server Requirements:
  OS: Ubuntu 20.04 LTS (specific requirement)
  CPU: 8 cores minimum (16 cores recommended)
  RAM: 32GB minimum (64GB recommended)
  Storage: 1TB SSD for recordings
  Network: 1Gbps with low latency
  
Installation Process:
  1. Clean Ubuntu 20.04 installation
  2. Update system packages and kernel
  3. Install BigBlueButton 2.7 via official script
  4. Configure firewall rules for BBB
  5. Install SSL certificates
  6. Configure recording processing
  7. Set up integration with ACPN portal
```

### **BBB Configuration & Optimization**
```yaml
Server Configuration:
  Max Concurrent Users: 500 (adjustable based on resources)
  Recording: Enabled with automated processing
  Webcam Quality: Configurable quality settings
  Audio: High-quality audio with echo cancellation
  
Performance Optimization:
  CPU Scaling: Governor set to performance mode
  Memory: Swap disabled, memory overcommit settings
  Network: TCP optimization for real-time communication
  Storage: Fast SSD storage for recording processing
  
Security Configuration:
  Firewall: Specific ports for BBB communication
  SSL: Required for WebRTC functionality
  Authentication: Integration with ACPN portal SSO
  Recording Security: Encrypted storage and transmission
```

### **BBB API Integration**
```yaml
API Endpoints:
  Create Meeting: Programmatic meeting creation
  Join Meeting: Secure meeting join URLs
  End Meeting: Administrative meeting termination
  Recording Management: Recording processing and retrieval
  
Integration Features:
  Single Sign-On: ACPN portal authentication
  Automatic Recording: Session recording for later access
  Attendance Tracking: Participant monitoring and logging
  CPD Credit Assignment: Automatic credit calculation
  
Webhook Configuration:
  Meeting Events: Start, end, participant join/leave
  Recording Events: Processing complete, ready for download
  Error Handling: Failed meeting creation, processing errors
```

---

## 📊 **Monitoring & Observability**

### **System Monitoring Stack**
```yaml
Prometheus:
  Purpose: Metrics collection and alerting
  Configuration: Node exporter, custom application metrics
  Storage: Time-series data with configurable retention
  Alerting: AlertManager for notification routing
  
Grafana:
  Purpose: Visualization and dashboards
  Data Sources: Prometheus, application databases
  Dashboards: System metrics, application performance
  Alerting: Visual alerts with notification channels
  
ELK Stack (Elasticsearch, Logstash, Kibana):
  Purpose: Centralized logging and analysis
  Log Sources: Application logs, system logs, web server logs
  Processing: Log parsing, enrichment, and indexing
  Visualization: Kibana dashboards for log analysis
```

### **Application Performance Monitoring**
```yaml
Custom Metrics:
  Response Times: API endpoint performance
  Error Rates: Application error tracking
  User Activity: Feature usage and engagement
  Database Performance: Query execution times
  
Health Checks:
  Service Health: Application service status
  Database Health: Connection and query performance
  External Services: GoMed API, WhatsApp API status
  Resource Usage: CPU, memory, disk utilization
  
Alerting Rules:
  Critical: Service down, database unreachable
  Warning: High response times, elevated error rates
  Info: Resource usage thresholds, backup completion
```

---

## 🔒 **Security Hardening**

### **System Security**
```yaml
Firewall Configuration:
  UFW: Uncomplicated Firewall with strict rules
  Allowed Ports: 22 (SSH), 80 (HTTP), 443 (HTTPS), BBB ports
  Rate Limiting: Connection rate limiting by IP
  DDoS Protection: Basic protection with fail2ban
  
SSH Security:
  Key-based Authentication: Disable password authentication
  Non-standard Port: Change default SSH port
  Connection Limits: Limit concurrent connections
  Two-Factor Authentication: Optional 2FA for admin access
  
Intrusion Detection:
  fail2ban: Automatic IP blocking for suspicious activity
  Log Monitoring: Real-time log analysis for threats
  File Integrity: AIDE for system file monitoring
  Rootkit Detection: chkrootkit for malware scanning
```

### **Application Security**
```yaml
SSL/TLS Configuration:
  Certificates: Let's Encrypt with automated renewal
  Cipher Suites: Strong encryption with forward secrecy
  HSTS: HTTP Strict Transport Security headers
  Certificate Pinning: Public key pinning for mobile apps
  
Data Protection:
  Encryption at Rest: Database encryption for sensitive data
  Encryption in Transit: TLS 1.3 for all communications
  Key Management: Hardware security module (HSM) or encrypted storage
  Backup Encryption: Encrypted backup storage
  
Access Control:
  Role-Based Access: Granular permission system
  Session Management: Secure session handling with timeout
  API Security: Rate limiting, input validation, authentication
  Audit Logging: Comprehensive logging of sensitive operations
```

---

## 🔄 **Backup & Disaster Recovery**

### **Backup Strategy**
```yaml
Database Backups:
  PostgreSQL: Daily pg_dump with point-in-time recovery
  MongoDB: Daily mongodump with binary log shipping
  Redis: RDB snapshots with AOF persistence
  Schedule: Automated daily backups at 2:00 AM
  
File System Backups:
  Application Code: Git repository with tagged releases
  User Uploads: Daily incremental backups
  Configuration Files: Version-controlled configuration management
  SSL Certificates: Automatic backup of Let's Encrypt certificates
  
Backup Storage:
  Local Storage: 7 days of local backup retention
  Cloud Storage: 30 days of cloud backup retention
  Geographic Distribution: Backups stored in different regions
  Encryption: All backups encrypted with AES-256
```

### **Disaster Recovery Plan**
```yaml
Recovery Scenarios:
  Hardware Failure: Server replacement and data restoration
  Data Corruption: Point-in-time database recovery
  Security Breach: Incident response and system rebuild
  Network Outage: Failover to backup data center
  
Recovery Procedures:
  RTO (Recovery Time Objective): 4 hours maximum
  RPO (Recovery Point Objective): 1 hour maximum
  Backup Validation: Monthly restore testing
  Documentation: Step-by-step recovery procedures
  
Failover Strategy:
  Database Failover: Automated promotion of read replicas
  Application Failover: Load balancer health check routing
  DNS Failover: Automated DNS record updates
  Communication: Automated status page updates
```

---

## 📈 **Scaling Strategy**

### **Horizontal Scaling**
```yaml
Application Scaling:
  Containerization: Docker containers for easy scaling
  Load Balancing: Nginx with multiple backend servers
  Session Management: Stateless application design
  Microservices: Service separation for independent scaling
  
Database Scaling:
  Read Replicas: PostgreSQL streaming replication
  Sharding: Horizontal partitioning for large datasets
  Caching: Redis cluster for distributed caching
  Connection Pooling: PgBouncer for connection management
  
Infrastructure Scaling:
  Auto-scaling: Automated server provisioning
  Resource Monitoring: Proactive capacity planning
  Cost Optimization: Right-sizing instances for workload
```

### **Performance Optimization**
```yaml
Application Optimization:
  Code Profiling: Regular performance analysis
  Database Optimization: Query optimization and indexing
  Caching Strategy: Multi-layer caching implementation
  CDN: Content delivery network for static assets
  
System Optimization:
  Kernel Tuning: Network and I/O optimization
  Memory Management: Tuned memory allocation
  CPU Optimization: Process scheduling and affinity
  Storage: I/O optimization and SSD configuration
```

---

## 🚀 **Deployment Pipeline**

### **CI/CD with GitHub Actions**
```yaml
Pipeline Stages:
  1. Source Control: Git push triggers pipeline
  2. Build Phase: Docker image creation and testing
  3. Test Phase: Automated unit and integration tests
  4. Security Scan: Vulnerability assessment
  5. Deployment: Rolling update to production
  
Deployment Strategy:
  Blue-Green: Zero-downtime deployment
  Canary: Gradual rollout for risk mitigation
  Rollback: Automated rollback on deployment failure
  Health Checks: Post-deployment verification
```

### **Infrastructure as Code**
```yaml
Configuration Management:
  Ansible: Server configuration and application deployment
  Docker Compose: Container orchestration and networking
  Environment Variables: Environment-specific configuration
  Secrets Management: Encrypted secrets storage
  
Version Control:
  Git: Infrastructure code version control
  Branching: Feature branches for configuration changes
  Code Review: Peer review for infrastructure changes
  Documentation: Inline documentation and README files
```

---

## 💰 **Cost Optimization**

### **Resource Management**
```yaml
Cost Optimization Strategies:
  Right-sizing: Match resources to actual usage
  Reserved Instances: Long-term commitments for discounts
  Spot Instances: Use spot pricing for non-critical workloads
  Resource Scheduling: Shut down non-production resources
  
Monitoring & Analysis:
  Cost Tracking: Monthly cost analysis and reporting
  Resource Utilization: Identify underutilized resources
  Performance vs Cost: Balance performance with cost
  Optimization Recommendations: Regular cost optimization reviews
```

### **Estimated Monthly Costs**
```yaml
Production Environment:
  Primary Application Server (8 CPU, 32GB RAM): $240/month
  Database Server (8 CPU, 32GB RAM): $240/month
  Load Balancer (4 CPU, 8GB RAM): $80/month
  BigBlueButton Server (8 CPU, 32GB RAM): $240/month
  Development Server (4 CPU, 16GB RAM): $120/month
  Monitoring Server (2 CPU, 8GB RAM): $40/month
  
Additional Costs:
  Domain Names: $20/year
  SSL Certificates: Free (Let's Encrypt)
  Cloud Storage (Backups): $50/month
  CDN (Optional): $30/month
  
Total Estimated Cost: $960/month + $50 backup storage
```

---

## 🔮 **Future Scaling Considerations**

### **Growth Planning**
```yaml
User Growth Projections:
  Year 1: 1,000 active users
  Year 2: 5,000 active users
  Year 3: 10,000+ active users
  
Infrastructure Scaling:
  Database: Horizontal sharding implementation
  Application: Microservices architecture adoption
  BigBlueButton: Multi-server cluster deployment
  Geographic: Multi-region deployment for global reach
```

### **Technology Evolution**
```yaml
Container Orchestration:
  Current: Docker Swarm for simplicity
  Future: Kubernetes migration for advanced features
  Benefits: Better auto-scaling, service mesh, advanced networking
  
Cloud Integration:
  Hybrid Approach: VPS + selective cloud services
  Benefits: Cost optimization with cloud flexibility
  Services: Object storage, managed databases, CDN
```

---

This VPS infrastructure provides a robust, scalable, and cost-effective foundation for the ACPN Lagos Portal while maintaining the flexibility to grow and evolve with changing requirements.