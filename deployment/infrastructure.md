# Infrastructure Setup

## 🏗️ Infrastructure Overview

The ACPN Portal is deployed on a modern, cloud-native infrastructure using AWS services with Kubernetes orchestration. The architecture is designed for high availability, scalability, and security.

## ☁️ AWS Architecture

### Core AWS Services

```mermaid
graph TB
    subgraph "Route 53"
        DNS[DNS Management]
    end
    
    subgraph "CloudFront"
        CDN[Global CDN]
        WAF[Web Application Firewall]
    end
    
    subgraph "Application Load Balancer"
        ALB[ALB with SSL Termination]
    end
    
    subgraph "EKS Cluster"
        K8S[Kubernetes Nodes]
        PODS[Application Pods]
    end
    
    subgraph "Data Services"
        RDS[(RDS PostgreSQL)]
        DOCDB[(DocumentDB)]
        REDIS[(ElastiCache Redis)]
        ES[(Elasticsearch Service)]
    end
    
    subgraph "Storage"
        S3[(S3 Buckets)]
        EFS[(EFS Volumes)]
    end
    
    subgraph "Security"
        IAM[IAM Roles]
        KMS[Key Management]
        SECRETS[Secrets Manager]
    end
    
    subgraph "Monitoring"
        CW[CloudWatch]
        XRAY[X-Ray Tracing]
    end
    
    DNS --> CDN
    CDN --> ALB
    ALB --> K8S
    K8S --> PODS
    PODS --> RDS
    PODS --> DOCDB
    PODS --> REDIS
    PODS --> ES
    PODS --> S3
    PODS --> SECRETS
    
    IAM --> PODS
    KMS --> SECRETS
    CW --> PODS
    XRAY --> PODS
```

### Infrastructure as Code (Terraform)

#### Main Infrastructure Configuration

```hcl
# terraform/main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
  }
  
  backend "s3" {
    bucket         = "acpn-terraform-state"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "acpn-terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = "acpn-portal"
      ManagedBy   = "terraform"
    }
  }
}

# VPC Configuration
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  
  name = "acpn-vpc-${var.environment}"
  cidr = var.vpc_cidr
  
  azs             = var.availability_zones
  private_subnets = var.private_subnets
  public_subnets  = var.public_subnets
  database_subnets = var.database_subnets
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
  enable_dns_hostnames = true
  enable_dns_support = true
  
  # Security
  enable_flow_log = true
  flow_log_destination_type = "cloud-watch-logs"
  
  tags = {
    "kubernetes.io/cluster/acpn-${var.environment}" = "shared"
  }
}

# EKS Cluster
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"
  
  cluster_name    = "acpn-${var.environment}"
  cluster_version = "1.27"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  # Cluster endpoint configuration
  cluster_endpoint_private_access = true
  cluster_endpoint_public_access  = true
  cluster_endpoint_public_access_cidrs = var.allowed_cidr_blocks
  
  # Encryption
  cluster_encryption_config = [
    {
      provider_key_arn = aws_kms_key.eks.arn
      resources        = ["secrets"]
    }
  ]
  
  # Node Groups
  eks_managed_node_groups = {
    main = {
      name = "main-${var.environment}"
      
      instance_types = var.node_instance_types
      capacity_type  = "ON_DEMAND"
      
      min_size     = var.node_group_min_size
      max_size     = var.node_group_max_size
      desired_size = var.node_group_desired_size
      
      # Storage
      disk_size = 50
      disk_type = "gp3"
      
      # Networking
      subnet_ids = module.vpc.private_subnets
      
      # Security
      key_name = aws_key_pair.eks_nodes.key_name
      
      labels = {
        Environment = var.environment
        NodeGroup   = "main"
      }
      
      taints = []
      
      update_config = {
        max_unavailable_percentage = 25
      }
    }
    
    spot = {
      name = "spot-${var.environment}"
      
      instance_types = var.spot_instance_types
      capacity_type  = "SPOT"
      
      min_size     = 0
      max_size     = var.spot_max_size
      desired_size = var.spot_desired_size
      
      labels = {
        Environment = var.environment
        NodeGroup   = "spot"
        WorkloadType = "batch"
      }
      
      taints = [
        {
          key    = "spot"
          value  = "true"
          effect = "NO_SCHEDULE"
        }
      ]
    }
  }
  
  # Add-ons
  cluster_addons = {
    coredns = {
      resolve_conflicts = "OVERWRITE"
    }
    kube-proxy = {}
    vpc-cni = {
      resolve_conflicts = "OVERWRITE"
    }
    aws-ebs-csi-driver = {
      resolve_conflicts = "OVERWRITE"
    }
  }
}
```

#### Database Infrastructure

```hcl
# terraform/database.tf

# RDS PostgreSQL for primary data
resource "aws_db_subnet_group" "main" {
  name       = "acpn-db-subnet-group-${var.environment}"
  subnet_ids = module.vpc.database_subnets
  
  tags = {
    Name = "ACPN DB subnet group"
  }
}

resource "aws_db_instance" "postgresql" {
  identifier = "acpn-postgres-${var.environment}"
  
  # Engine
  engine         = "postgres"
  engine_version = "15.3"
  instance_class = var.postgres_instance_class
  
  # Storage
  allocated_storage     = var.postgres_allocated_storage
  max_allocated_storage = var.postgres_max_allocated_storage
  storage_type          = "gp3"
  storage_encrypted     = true
  kms_key_id           = aws_kms_key.rds.arn
  
  # Database
  db_name  = "acpnportal"
  username = var.postgres_username
  password = var.postgres_password
  
  # Networking
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  publicly_accessible    = false
  
  # Backup
  backup_retention_period = var.postgres_backup_retention
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  # Monitoring
  performance_insights_enabled = true
  monitoring_interval         = 60
  monitoring_role_arn        = aws_iam_role.rds_monitoring.arn
  
  # High Availability
  multi_az = var.environment == "production"
  
  # Security
  deletion_protection = var.environment == "production"
  skip_final_snapshot = var.environment != "production"
  
  tags = {
    Name = "ACPN PostgreSQL ${var.environment}"
  }
}

# DocumentDB for MongoDB workloads
resource "aws_docdb_cluster" "main" {
  cluster_identifier      = "acpn-docdb-${var.environment}"
  engine                 = "docdb"
  
  master_username        = var.docdb_username
  master_password        = var.docdb_password
  
  backup_retention_period = var.docdb_backup_retention
  preferred_backup_window = "03:00-04:00"
  
  db_subnet_group_name   = aws_docdb_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.docdb.id]
  
  storage_encrypted = true
  kms_key_id       = aws_kms_key.docdb.arn
  
  skip_final_snapshot = var.environment != "production"
  
  tags = {
    Name = "ACPN DocumentDB ${var.environment}"
  }
}

resource "aws_docdb_cluster_instance" "main" {
  count              = var.docdb_instance_count
  identifier         = "acpn-docdb-${var.environment}-${count.index}"
  cluster_identifier = aws_docdb_cluster.main.id
  instance_class     = var.docdb_instance_class
}

# ElastiCache Redis for caching
resource "aws_elasticache_subnet_group" "main" {
  name       = "acpn-cache-subnet-${var.environment}"
  subnet_ids = module.vpc.private_subnets
}

resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "acpn-redis-${var.environment}"
  description               = "ACPN Redis cluster"
  
  node_type                 = var.redis_node_type
  port                      = 6379
  parameter_group_name      = "default.redis7"
  
  num_cache_clusters        = var.redis_num_cache_nodes
  
  subnet_group_name         = aws_elasticache_subnet_group.main.name
  security_group_ids        = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  auth_token                = var.redis_auth_token
  
  automatic_failover_enabled = var.environment == "production"
  multi_az_enabled          = var.environment == "production"
  
  tags = {
    Name = "ACPN Redis ${var.environment}"
  }
}
```

#### Security Groups

```hcl
# terraform/security.tf

# EKS Cluster Security Group
resource "aws_security_group" "eks_cluster" {
  name_prefix = "acpn-eks-cluster-${var.environment}"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "acpn-eks-cluster-sg-${var.environment}"
  }
}

# RDS Security Group
resource "aws_security_group" "rds" {
  name_prefix = "acpn-rds-${var.environment}"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    description     = "PostgreSQL from EKS"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [module.eks.node_security_group_id]
  }
  
  tags = {
    Name = "acpn-rds-sg-${var.environment}"
  }
}

# DocumentDB Security Group
resource "aws_security_group" "docdb" {
  name_prefix = "acpn-docdb-${var.environment}"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    description     = "MongoDB from EKS"
    from_port       = 27017
    to_port         = 27017
    protocol        = "tcp"
    security_groups = [module.eks.node_security_group_id]
  }
  
  tags = {
    Name = "acpn-docdb-sg-${var.environment}"
  }
}

# Redis Security Group
resource "aws_security_group" "redis" {
  name_prefix = "acpn-redis-${var.environment}"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    description     = "Redis from EKS"
    from_port       = 6379
    to_port         = 6379
    protocol        = "tcp"
    security_groups = [module.eks.node_security_group_id]
  }
  
  tags = {
    Name = "acpn-redis-sg-${var.environment}"
  }
}
```

## 🐳 Kubernetes Configuration

### Namespace and Resource Quotas

```yaml
# k8s/namespaces/acpn-portal.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: acpn-portal
  labels:
    name: acpn-portal
    environment: production
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: acpn-portal-quota
  namespace: acpn-portal
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    persistentvolumeclaims: "10"
    pods: "50"
    services: "20"
    secrets: "30"
    configmaps: "30"
---
apiVersion: v1
kind: LimitRange
metadata:
  name: acpn-portal-limits
  namespace: acpn-portal
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
```

### ConfigMaps and Secrets

```yaml
# k8s/config/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: acpn-config
  namespace: acpn-portal
data:
  # Application Configuration
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  PORT: "3000"
  
  # Database Configuration
  POSTGRES_HOST: "acpn-postgres-production.cluster-xyz.us-west-2.rds.amazonaws.com"
  POSTGRES_PORT: "5432"
  POSTGRES_DATABASE: "acpnportal"
  
  MONGODB_HOST: "acpn-docdb-production.cluster-xyz.docdb.us-west-2.amazonaws.com"
  MONGODB_PORT: "27017"
  MONGODB_DATABASE: "acpnportal"
  
  REDIS_HOST: "acpn-redis-production.cache.amazonaws.com"
  REDIS_PORT: "6379"
  
  # External Services
  GOMED_API_URL: "https://api.gomed.ng/v1"
  WHATSAPP_API_URL: "https://api.whatsapp.com/v1"
  
  # Feature Flags
  ENABLE_GOMED_INTEGRATION: "true"
  ENABLE_WHATBOT: "true"
  ENABLE_ANALYTICS: "true"
  
  # Rate Limiting
  RATE_LIMIT_WINDOW: "900000"  # 15 minutes
  RATE_LIMIT_MAX: "100"
  
  # File Upload
  MAX_FILE_SIZE: "10485760"  # 10MB
  ALLOWED_FILE_TYPES: "jpg,jpeg,png,pdf,doc,docx"
---
apiVersion: v1
kind: Secret
metadata:
  name: acpn-secrets
  namespace: acpn-portal
type: Opaque
data:
  # Database Credentials (base64 encoded)
  POSTGRES_USERNAME: cGhhcm1hY3lfdXNlcg==
  POSTGRES_PASSWORD: UGFzcHdvcmQxMjM=
  MONGODB_USERNAME: bW9uZ29fdXNlcg==
  MONGODB_PASSWORD: UGFzcHdvcmQxMjM=
  REDIS_PASSWORD: UmVkaXNQYXNzd29yZDEyMw==
  
  # API Keys
  JWT_SECRET: c3VwZXJfc2VjcmV0X2p3dF9rZXk=
  GOMED_API_KEY: Z29tZWRfYXBpX2tleV8xMjM=
  GOMED_SECRET_KEY: Z29tZWRfc2VjcmV0X2tleV8xMjM=
  WHATSAPP_API_TOKEN: d2hhdHNhcHBfYXBpX3Rva2VuXzEyMw==
  
  # Payment Gateway
  PAYSTACK_SECRET_KEY: cGF5c3RhY2tfc2VjcmV0X2tleQ==
  FLUTTERWAVE_SECRET_KEY: Zmx1dHRlcndhdmVfc2VjcmV0X2tleQ==
  
  # Email Service
  SENDGRID_API_KEY: c2VuZGdyaWRfYXBpX2tleQ==
  
  # Encryption
  ENCRYPTION_KEY: ZW5jcnlwdGlvbl9rZXlfMzJfY2hhcnM=
```

### Application Deployments

```yaml
# k8s/deployments/user-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: acpn-portal
  labels:
    app: user-service
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: user-service
      version: v1
  template:
    metadata:
      labels:
        app: user-service
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: acpn-service-account
      containers:
      - name: user-service
        image: acpn/user-service:v1.2.3
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
          name: http
        - containerPort: 9090
          name: metrics
        env:
        - name: SERVICE_NAME
          value: "user-service"
        envFrom:
        - configMapRef:
            name: acpn-config
        - secretRef:
            name: acpn-secrets
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        volumeMounts:
        - name: logs
          mountPath: /app/logs
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: logs
        emptyDir: {}
      - name: tmp
        emptyDir: {}
      nodeSelector:
        kubernetes.io/arch: amd64
      tolerations:
      - key: "spot"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
```

### Services and Ingress

```yaml
# k8s/services/user-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: acpn-portal
  labels:
    app: user-service
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
    name: http
  - port: 9090
    targetPort: 9090
    protocol: TCP
    name: metrics
  selector:
    app: user-service
---
# k8s/ingress/main-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: acpn-portal-ingress
  namespace: acpn-portal
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://portal.acpnlagos.org"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization"
spec:
  tls:
  - hosts:
    - api.acpnlagos.org
    secretName: acpn-api-tls
  rules:
  - host: api.acpnlagos.org
    http:
      paths:
      - path: /api/v1/auth
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /api/v1/users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /api/v1/pharmacies
        pathType: Prefix
        backend:
          service:
            name: pharmacy-service
            port:
              number: 80
      - path: /api/v1/products
        pathType: Prefix
        backend:
          service:
            name: product-service
            port:
              number: 80
      - path: /api/v1/marketplace
        pathType: Prefix
        backend:
          service:
            name: marketplace-service
            port:
              number: 80
      - path: /api/v1/events
        pathType: Prefix
        backend:
          service:
            name: event-service
            port:
              number: 80
      - path: /api/v1/payments
        pathType: Prefix
        backend:
          service:
            name: payment-service
            port:
              number: 80
      - path: /api/v1/integrations
        pathType: Prefix
        backend:
          service:
            name: integration-service
            port:
              number: 80
      - path: /api/v1/whatbot
        pathType: Prefix
        backend:
          service:
            name: whatbot-service
            port:
              number: 80
```

## 🔒 Security Configuration

### RBAC (Role-Based Access Control)

```yaml
# k8s/rbac/service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: acpn-service-account
  namespace: acpn-portal
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: acpn-portal
  name: acpn-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: acpn-role-binding
  namespace: acpn-portal
subjects:
- kind: ServiceAccount
  name: acpn-service-account
  namespace: acpn-portal
roleRef:
  kind: Role
  name: acpn-role
  apiGroup: rbac.authorization.k8s.io
```

### Network Policies

```yaml
# k8s/security/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: acpn-network-policy
  namespace: acpn-portal
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: nginx-ingress
    - namespaceSelector:
        matchLabels:
          name: acpn-portal
    ports:
    - protocol: TCP
      port: 3000
  egress:
  - to: []  # Allow all egress traffic
    ports:
    - protocol: TCP
      port: 443  # HTTPS
    - protocol: TCP
      port: 80   # HTTP
    - protocol: TCP
      port: 5432 # PostgreSQL
    - protocol: TCP
      port: 27017 # MongoDB
    - protocol: TCP
      port: 6379  # Redis
    - protocol: UDP
      port: 53    # DNS
```

### Pod Security Standards

```yaml
# k8s/security/pod-security-policy.yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  namespace: acpn-portal
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1001
    runAsGroup: 1001
    fsGroup: 1001
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: acpn/user-service:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /app/cache
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

## 📊 Monitoring Setup

### Prometheus Configuration

```yaml
# k8s/monitoring/prometheus.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    rule_files:
      - "/etc/prometheus/rules/*.yml"
    
    scrape_configs:
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
    
    - job_name: 'kubernetes-services'
      kubernetes_sd_configs:
      - role: service
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_service_name

    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          - alertmanager:9093
```

### Grafana Dashboards

```json
{
  "dashboard": {
    "id": null,
    "title": "ACPN Portal - Application Metrics",
    "tags": ["acpn", "application"],
    "timezone": "UTC",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{namespace=\"acpn-portal\"}[5m])) by (service)",
            "legendFormat": "{{service}}"
          }
        ],
        "yAxes": [
          {
            "label": "Requests/sec",
            "min": 0
          }
        ]
      },
      {
        "id": 2,
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{namespace=\"acpn-portal\"}[5m])) by (le, service))",
            "legendFormat": "95th percentile - {{service}}"
          },
          {
            "expr": "histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket{namespace=\"acpn-portal\"}[5m])) by (le, service))",
            "legendFormat": "50th percentile - {{service}}"
          }
        ],
        "yAxes": [
          {
            "label": "Seconds",
            "min": 0
          }
        ]
      },
      {
        "id": 3,
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{namespace=\"acpn-portal\", status=~\"5..\"}[5m])) by (service) / sum(rate(http_requests_total{namespace=\"acpn-portal\"}[5m])) by (service)",
            "legendFormat": "Error rate - {{service}}"
          }
        ],
        "yAxes": [
          {
            "label": "Error rate",
            "min": 0,
            "max": 1
          }
        ]
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "30s"
  }
}
```

## 🚀 Deployment Pipeline

### Helm Charts

```yaml
# helm/acpn-portal/Chart.yaml
apiVersion: v2
name: acpn-portal
description: A Helm chart for ACPN Portal
type: application
version: 0.1.0
appVersion: "1.0.0"

dependencies:
- name: postgresql
  version: 11.9.13
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled
- name: redis
  version: 17.3.7
  repository: https://charts.bitnami.com/bitnami
  condition: redis.enabled
```

```yaml
# helm/acpn-portal/values.yaml
# Default values for acpn-portal
replicaCount: 3

image:
  repository: acpn/portal
  pullPolicy: IfNotPresent
  tag: ""

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations: {}

podSecurityContext:
  fsGroup: 1001
  runAsNonRoot: true
  runAsUser: 1001

securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1001

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
  - host: api.acpnlagos.org
    paths:
    - path: /
      pathType: Prefix
  tls:
  - secretName: acpn-api-tls
    hosts:
    - api.acpnlagos.org

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app.kubernetes.io/name
            operator: In
            values:
            - acpn-portal
        topologyKey: kubernetes.io/hostname

# External services
postgresql:
  enabled: false
  external:
    host: acpn-postgres-production.cluster-xyz.us-west-2.rds.amazonaws.com
    port: 5432
    database: acpnportal

redis:
  enabled: false
  external:
    host: acpn-redis-production.cache.amazonaws.com
    port: 6379

# Configuration
config:
  nodeEnv: production
  logLevel: info
  enableGomedIntegration: true
  enableWhatbot: true

# Secrets (references to existing secrets)
secrets:
  databaseUrl: "postgresql://username:password@host:5432/database"
  jwtSecret: "super_secret_jwt_key"
  gomedApiKey: "gomed_api_key"
```

This comprehensive infrastructure setup provides a production-ready, secure, and scalable foundation for the ACPN Portal, with proper monitoring, security, and deployment automation. 