# API Overview

## 🌐 Base URL and Versioning

```
Production: https://api.acpnlagos.org/v1
Staging: https://staging-api.acpnlagos.org/v1
Development: http://localhost:4000/api/v1
```

## 🔑 Authentication

All API requests require authentication via JWT tokens in the Authorization header:

```bash
Authorization: Bearer <jwt_token>
```

### API Key for External Services

External integrations (like GoMed) use API keys:

```bash
X-API-Key: <api_key>
X-Client-ID: <client_id>
```

## 📊 Standard Response Format

All API responses follow this structure:

```typescript
interface APIResponse<T> {
  success: boolean
  data?: T
  error?: {
    code: string
    message: string
    details?: any
    field?: string
  }
  meta?: {
    pagination?: PaginationMeta
    filters?: FilterMeta
    timestamp: string
    requestId: string
    version: string
  }
}
```

### Success Response Example

```json
{
  "success": true,
  "data": {
    "id": "pharm_12345",
    "name": "HealthPlus Pharmacy",
    "email": "manager@healthplus.com"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_abc123def456",
    "version": "1.0"
  }
}
```

### Error Response Example

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": ["Email format is invalid"],
      "phoneNumber": ["Phone number is required"]
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_abc123def456",
    "version": "1.0"
  }
}
```

## 📄 Pagination

List endpoints support pagination:

```typescript
interface PaginationMeta {
  page: number
  limit: number
  total: number
  totalPages: number
  hasNext: boolean
  hasPrev: boolean
}
```

### Pagination Parameters

```bash
GET /api/v1/pharmacies?page=2&limit=20&sort=name&order=asc
```

### Pagination Response

```json
{
  "success": true,
  "data": [
    {
      "id": "pharm_001",
      "name": "Pharmacy A"
    }
  ],
  "meta": {
    "pagination": {
      "page": 2,
      "limit": 20,
      "total": 150,
      "totalPages": 8,
      "hasNext": true,
      "hasPrev": true
    }
  }
}
```

## 🔍 Filtering and Searching

### Query Parameters

```bash
GET /api/v1/products?search=paracetamol&category=analgesics&manufacturer=GSK&in_stock=true
```

### Advanced Filtering

```typescript
interface FilterMeta {
  applied: {
    [key: string]: any
  }
  available: {
    [key: string]: FilterOption[]
  }
}

interface FilterOption {
  value: string
  label: string
  count: number
}
```

## ⚡ Rate Limiting

Rate limits are applied per endpoint:

| Endpoint Type | Limit | Window |
|---------------|-------|--------|
| Authentication | 10 requests | 1 minute |
| General API | 100 requests | 1 minute |
| Search | 200 requests | 1 minute |
| WhatsApp Bot | 1000 requests | 1 minute |
| File Upload | 10 requests | 5 minutes |

### Rate Limit Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642248000
Retry-After: 60
```

## 🔒 Security Headers

All responses include security headers:

```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
```

## 📝 Content Types

Supported content types:

- **JSON**: `application/json` (default)
- **Form Data**: `multipart/form-data` (file uploads)
- **URL Encoded**: `application/x-www-form-urlencoded`

## 🌍 Localization

API supports multiple languages via the `Accept-Language` header:

```bash
Accept-Language: en-US,en;q=0.9,yo;q=0.8
```

Supported languages:
- `en` - English (default)
- `yo` - Yoruba
- `ha` - Hausa
- `ig` - Igbo

## 📊 HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Missing/invalid authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource already exists |
| 422 | Unprocessable Entity | Validation errors |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Maintenance mode |

## 🔄 Idempotency

POST requests support idempotency keys to prevent duplicate operations:

```bash
POST /api/v1/marketplace/requests
Idempotency-Key: req_12345_67890
Content-Type: application/json
```

## 📱 Webhook Support

The system supports webhooks for real-time notifications:

### Webhook Configuration

```typescript
interface WebhookConfig {
  url: string
  events: WebhookEvent[]
  secret: string
  active: boolean
  retries: number
}

type WebhookEvent = 
  | 'user.created'
  | 'pharmacy.verified'
  | 'marketplace.request.created'
  | 'marketplace.offer.received'
  | 'payment.completed'
  | 'event.registered'
  | 'whatbot.response.received'
  | 'gomed.order.created'
```

### Webhook Payload

```json
{
  "event": "marketplace.request.created",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "requestId": "req_12345",
    "pharmacyId": "pharm_67890",
    "productSku": "PAR-500-100",
    "quantity": 50
  },
  "signature": "sha256=abc123def456..."
}
```

## 🧪 Testing and Mock Data

### Test Environment

```bash
Base URL: https://test-api.acpnlagos.org/v1
Test API Key: test_key_12345abcdef
```

### Mock Data Endpoints

```bash
GET /api/v1/mock/users
GET /api/v1/mock/pharmacies
GET /api/v1/mock/products
GET /api/v1/mock/marketplace-requests
```

## 📈 API Analytics

Track API usage via custom headers:

```bash
X-Client-Version: 1.2.3
X-Platform: web|mobile|api
X-Source: dashboard|marketplace|mobile-app
```

## 🔧 Development Tools

### SDKs and Libraries

```bash
# JavaScript/TypeScript
npm install @acpn-lagos/api-client

# Python
pip install acpn-lagos-api

# PHP
composer require acpn-lagos/api-client
```

### Postman Collection

Download the complete Postman collection:
[ACPN API Collection](https://api.acpnlagos.org/docs/postman/collection.json)

### OpenAPI Specification

View the interactive API documentation:
[API Documentation](https://api.acpnlagos.org/docs)

Download OpenAPI spec:
[OpenAPI 3.0 Spec](https://api.acpnlagos.org/docs/openapi.json)

## 🚨 Error Handling Best Practices

### Retry Logic

Implement exponential backoff for retries:

```javascript
const retryConfig = {
  maxRetries: 3,
  baseDelay: 1000, // 1 second
  maxDelay: 10000, // 10 seconds
  retryableStatuses: [429, 500, 502, 503, 504]
}
```

### Circuit Breaker Pattern

Implement circuit breakers for external service calls:

```javascript
const circuitBreakerConfig = {
  errorThresholdPercentage: 50,
  requestVolumeThreshold: 20,
  sleepWindowInMilliseconds: 60000
}
```

## 📚 Additional Resources

- [Authentication Guide](auth.md)
- [User Management API](users.md)
- [Pharmacy Management API](pharmacy.md)
- [Marketplace API](marketplace.md)
- [WhatsApp Bot API](whatbot.md)
- [GoMed Integration API](gomed.md)
- [Events & Payments API](events-payments.md) 