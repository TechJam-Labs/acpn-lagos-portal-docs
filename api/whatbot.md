# WhatsApp Bot (WhatBot) API

## 🤖 WhatBot Overview

WhatBot is the WhatsApp automation service that enables real-time product availability queries. It uses Venom-bot to send WhatsApp messages to pharmacies and parse their responses for availability, pricing, and quantity information.

## 📋 Core Endpoints

### POST /api/v1/whatbot/ping-pharmacies

Initiate a WhatsApp ping to nearby pharmacies for product availability.

#### Request Body

```typescript
interface WhatBotPingRequest {
  productSku: string
  geoLocation: {
    latitude: number
    longitude: number
    radius: number // in kilometers
  }
  urgency: 'high' | 'medium' | 'low'
  timeout: number // in seconds (default: 300)
  maxPharmacies?: number // default: 20
  filters?: {
    verifiedOnly?: boolean
    rating?: number // minimum rating
    responseTimeMax?: number // max historical response time
  }
  externalRequestId?: string // From GoMed or other integrations
  requesterInfo?: {
    name: string
    contact: string
    location: string
  }
}
```

#### Example Request

```bash
POST /api/v1/whatbot/ping-pharmacies
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "productSku": "PAR-500-100",
  "geoLocation": {
    "latitude": 6.5244,
    "longitude": 3.3792,
    "radius": 5
  },
  "urgency": "high",
  "timeout": 300,
  "maxPharmacies": 15,
  "filters": {
    "verifiedOnly": true,
    "rating": 4.0,
    "responseTimeMax": 180
  },
  "externalRequestId": "gomed_req_12345",
  "requesterInfo": {
    "name": "HealthPlus Customer",
    "contact": "+234 803 123 4567",
    "location": "Ikoyi, Lagos"
  }
}
```

#### Success Response (201)

```json
{
  "success": true,
  "data": {
    "pingId": "ping_12345abc",
    "productInfo": {
      "sku": "PAR-500-100",
      "name": "Paracetamol 500mg Tablets",
      "form": "Tablet",
      "strength": "500mg",
      "packSize": "100 tablets"
    },
    "pharmaciesContacted": 12,
    "estimatedResponseTime": 180,
    "timeout": 300,
    "status": "pending",
    "messageTemplate": "🏥 Product Inquiry\n\nHello! We need: Paracetamol 500mg (100 tablets)\n\nPlease reply with:\n✅ Available - quantity & price\n❌ Not available\n\nRequest ID: ping_12345abc",
    "pharmacies": [
      {
        "pharmacyId": "pharm_001",
        "name": "HealthPlus Pharmacy",
        "distance": 1.2,
        "whatsappNumber": "+234 803 111 2222",
        "messageSent": true,
        "sentAt": "2024-01-15T10:30:00Z"
      },
      {
        "pharmacyId": "pharm_002",
        "name": "MedCare Pharmacy",
        "distance": 2.1,
        "whatsappNumber": "+234 803 333 4444",
        "messageSent": true,
        "sentAt": "2024-01-15T10:30:01Z"
      }
    ]
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_ping_12345"
  }
}
```

### GET /api/v1/whatbot/ping/{pingId}/status

Get the current status and responses for a WhatsApp ping.

#### Path Parameters

- `pingId` (string): The ping ID returned from the initial request

#### Query Parameters

- `includeDetails` (boolean): Include detailed response parsing (default: false)

#### Example Request

```bash
GET /api/v1/whatbot/ping/ping_12345abc/status?includeDetails=true
Authorization: Bearer <access_token>
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "pingId": "ping_12345abc",
    "status": "completed",
    "productSku": "PAR-500-100",
    "startedAt": "2024-01-15T10:30:00Z",
    "completedAt": "2024-01-15T10:32:30Z",
    "timeout": 300,
    "summary": {
      "pharmaciesContacted": 12,
      "responsesReceived": 8,
      "responseRate": 0.67,
      "averageResponseTime": 95,
      "availablePharmacies": 5,
      "unavailablePharmacies": 3
    },
    "responses": [
      {
        "pharmacyId": "pharm_001",
        "pharmacyName": "HealthPlus Pharmacy",
        "whatsappNumber": "+234 803 111 2222",
        "receivedAt": "2024-01-15T10:31:15Z",
        "responseTime": 75,
        "rawMessage": "Available! We have 50 units at ₦250 each. Good quality, expires Dec 2025.",
        "parsed": {
          "availability": "available",
          "quantity": 50,
          "price": 250,
          "currency": "NGN",
          "conditions": "expires Dec 2025",
          "confidence": 0.95
        },
        "status": "processed"
      },
      {
        "pharmacyId": "pharm_002",
        "pharmacyName": "MedCare Pharmacy",
        "whatsappNumber": "+234 803 333 4444",
        "receivedAt": "2024-01-15T10:31:45Z",
        "responseTime": 105,
        "rawMessage": "Sorry, not available at the moment. Will have stock next week.",
        "parsed": {
          "availability": "unavailable",
          "quantity": 0,
          "estimatedAvailability": "next week",
          "confidence": 0.90
        },
        "status": "processed"
      },
      {
        "pharmacyId": "pharm_003",
        "pharmacyName": "QuickMed Pharmacy",
        "whatsappNumber": "+234 803 555 6666",
        "receivedAt": "2024-01-15T10:32:20Z",
        "responseTime": 140,
        "rawMessage": "Yes available. 30 pieces N280 each",
        "parsed": {
          "availability": "available",
          "quantity": 30,
          "price": 280,
          "currency": "NGN",
          "confidence": 0.85
        },
        "status": "processed"
      }
    ],
    "ranking": [
      {
        "pharmacyId": "pharm_001",
        "score": 0.92,
        "factors": {
          "price": 0.95,
          "distance": 0.98,
          "availability": 1.0,
          "responseTime": 0.90,
          "reliability": 0.85
        }
      },
      {
        "pharmacyId": "pharm_003",
        "score": 0.78,
        "factors": {
          "price": 0.80,
          "distance": 0.85,
          "availability": 1.0,
          "responseTime": 0.75,
          "reliability": 0.80
        }
      }
    ]
  },
  "meta": {
    "timestamp": "2024-01-15T10:33:00Z",
    "requestId": "req_status_12345"
  }
}
```

### POST /api/v1/whatbot/receive-response

Internal endpoint for processing incoming WhatsApp responses (webhook).

#### Request Body

```typescript
interface WhatBotIncomingResponse {
  phoneNumber: string
  message: string
  timestamp: string
  messageId: string
  senderName?: string
  messageType: 'text' | 'image' | 'document'
  metadata?: {
    quotedMessage?: string
    forwardedFrom?: string
  }
}
```

#### Example Request

```bash
POST /api/v1/whatbot/receive-response
X-Webhook-Secret: <webhook_secret>
Content-Type: application/json

{
  "phoneNumber": "+234 803 111 2222",
  "message": "Available! We have 50 units at ₦250 each. Good quality, expires Dec 2025.",
  "timestamp": "2024-01-15T10:31:15Z",
  "messageId": "msg_abc123",
  "senderName": "HealthPlus Pharmacy",
  "messageType": "text"
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "processed": true,
    "pharmacyId": "pharm_001",
    "pingId": "ping_12345abc",
    "parsedResponse": {
      "availability": "available",
      "quantity": 50,
      "price": 250,
      "currency": "NGN",
      "conditions": "expires Dec 2025",
      "confidence": 0.95
    },
    "acknowledgmentSent": true
  },
  "meta": {
    "timestamp": "2024-01-15T10:31:16Z",
    "requestId": "req_receive_12345"
  }
}
```

### POST /api/v1/whatbot/send-custom-message

Send a custom WhatsApp message to specific pharmacies.

#### Request Body

```typescript
interface CustomMessageRequest {
  pharmacyIds: string[]
  message: string
  messageType?: 'text' | 'template'
  templateId?: string
  templateData?: Record<string, any>
  priority: 'high' | 'medium' | 'low'
  scheduledAt?: string // ISO datetime
}
```

#### Example Request

```bash
POST /api/v1/whatbot/send-custom-message
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "pharmacyIds": ["pharm_001", "pharm_002", "pharm_003"],
  "message": "🎉 New product announcement: We have special pricing on Vitamin C tablets this week. Contact us for bulk orders!",
  "messageType": "text",
  "priority": "medium"
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "messageId": "custom_msg_12345",
    "pharmaciesContacted": 3,
    "messagesSent": 3,
    "messagesFailed": 0,
    "status": "sent",
    "deliveryTracking": [
      {
        "pharmacyId": "pharm_001",
        "whatsappNumber": "+234 803 111 2222",
        "status": "sent",
        "sentAt": "2024-01-15T11:00:00Z"
      },
      {
        "pharmacyId": "pharm_002",
        "whatsappNumber": "+234 803 333 4444",
        "status": "sent",
        "sentAt": "2024-01-15T11:00:01Z"
      },
      {
        "pharmacyId": "pharm_003",
        "whatsappNumber": "+234 803 555 6666",
        "status": "sent",
        "sentAt": "2024-01-15T11:00:02Z"
      }
    ]
  },
  "meta": {
    "timestamp": "2024-01-15T11:00:00Z",
    "requestId": "req_custom_12345"
  }
}
```

### GET /api/v1/whatbot/analytics

Get WhatsApp bot performance analytics.

#### Query Parameters

- `startDate` (string): Start date (ISO format)
- `endDate` (string): End date (ISO format)
- `pharmacyId` (string, optional): Filter by specific pharmacy
- `productSku` (string, optional): Filter by specific product

#### Example Request

```bash
GET /api/v1/whatbot/analytics?startDate=2024-01-01&endDate=2024-01-31&pharmacyId=pharm_001
Authorization: Bearer <access_token>
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "period": {
      "startDate": "2024-01-01T00:00:00Z",
      "endDate": "2024-01-31T23:59:59Z"
    },
    "summary": {
      "totalPings": 1250,
      "totalMessages": 15000,
      "averageResponseRate": 0.72,
      "averageResponseTime": 145,
      "successfulTransactions": 890,
      "conversionRate": 0.71
    },
    "pharmacyPerformance": [
      {
        "pharmacyId": "pharm_001",
        "pharmacyName": "HealthPlus Pharmacy",
        "messagesReceived": 45,
        "responsesGiven": 42,
        "responseRate": 0.93,
        "averageResponseTime": 85,
        "availabilityRate": 0.78,
        "rating": 4.8
      }
    ],
    "productTrends": [
      {
        "productSku": "PAR-500-100",
        "productName": "Paracetamol 500mg",
        "requestCount": 125,
        "availabilityRate": 0.85,
        "averagePrice": 275,
        "priceRange": {
          "min": 220,
          "max": 320
        }
      }
    ],
    "timeAnalysis": {
      "peakHours": [9, 10, 11, 14, 15, 16],
      "bestResponseTimes": {
        "morning": 95,
        "afternoon": 120,
        "evening": 165
      }
    }
  },
  "meta": {
    "timestamp": "2024-01-15T11:30:00Z",
    "requestId": "req_analytics_12345"
  }
}
```

## 🤖 Message Templates

### Standard Product Inquiry Template

```
🏥 Product Inquiry

Hello! We need: {productName}

Please reply with:
✅ Available - quantity & price
❌ Not available

Urgent: {urgency}
Customer: {customerLocation}

Request ID: {pingId}
```

### Follow-up Template

```
⏰ Follow-up Reminder

Still need: {productName}

Please respond if available:
- Quantity available
- Price per unit
- Expiry date

Request ID: {pingId}
```

### Acknowledgment Template

```
✅ Thank you for responding!

Your availability info has been recorded.
Customer will be notified.

ACPN Lagos Portal
```

## 🔍 Response Parsing Algorithm

### Keywords Detection

```typescript
interface ResponseKeywords {
  availability: {
    available: ['available', 'yes', 'have', 'stock', 'in stock', 'got it', 'we have']
    unavailable: ['not available', 'no', 'out of stock', 'finished', 'none', 'don\'t have']
    partial: ['limited', 'few', 'small quantity', 'running low']
  }
  quantity: {
    patterns: [
      /(\d+)\s*(units?|pieces?|bottles?|boxes?|packs?|tablets?)/i,
      /quantity[:\s]*(\d+)/i,
      /(\d+)\s*available/i,
      /have\s*(\d+)/i
    ]
  }
  price: {
    patterns: [
      /₦\s*(\d+(?:,\d{3})*(?:\.\d{2})?)/,
      /naira\s*(\d+)/i,
      /(\d+)\s*naira/i,
      /price[:\s]*₦?\s*(\d+)/i,
      /(\d+)\s*each/i
    ]
  }
  conditions: {
    expiry: [/expires?\s*(\w+\s*\d{4})/i, /expiry[:\s]*(\w+\s*\d{4})/i],
    quality: ['good quality', 'excellent', 'fresh', 'new stock'],
    delivery: ['same day', 'next day', 'immediate', 'pickup']
  }
}
```

### Confidence Scoring

```typescript
interface ConfidenceFactors {
  keyword_match: number      // 0.4 weight
  number_extraction: number  // 0.3 weight
  message_structure: number  // 0.2 weight
  pharmacy_history: number   // 0.1 weight
}

function calculateConfidence(factors: ConfidenceFactors): number {
  return (
    factors.keyword_match * 0.4 +
    factors.number_extraction * 0.3 +
    factors.message_structure * 0.2 +
    factors.pharmacy_history * 0.1
  )
}
```

## 📊 Performance Metrics

### Response Time Benchmarks

| Urgency Level | Target Response Time | Timeout |
|---------------|---------------------|---------|
| High | < 60 seconds | 180 seconds |
| Medium | < 120 seconds | 300 seconds |
| Low | < 300 seconds | 600 seconds |

### Success Rate Targets

| Metric | Target | Acceptable |
|--------|--------|------------|
| Message Delivery | > 98% | > 95% |
| Response Rate | > 75% | > 60% |
| Parse Accuracy | > 90% | > 80% |
| False Positives | < 5% | < 10% |

## 🚨 Error Handling

### Common Error Codes

| Code | Description | Resolution |
|------|-------------|------------|
| `WHATBOT_UNAVAILABLE` | WhatsApp bot service down | Retry after 5 minutes |
| `PHONE_NUMBER_INVALID` | Invalid pharmacy WhatsApp number | Update pharmacy contact |
| `MESSAGE_SEND_FAILED` | Failed to send WhatsApp message | Check network connectivity |
| `PARSING_ERROR` | Failed to parse response | Manual review required |
| `TIMEOUT_EXCEEDED` | No responses within timeout | Extend timeout or retry |
| `RATE_LIMIT_EXCEEDED` | Too many messages sent | Wait for rate limit reset |

### Retry Logic

```typescript
interface RetryConfig {
  maxRetries: 3
  baseDelay: 5000  // 5 seconds
  maxDelay: 30000  // 30 seconds
  retryableErrors: [
    'WHATBOT_UNAVAILABLE',
    'MESSAGE_SEND_FAILED',
    'NETWORK_ERROR'
  ]
}
```

## 🔒 Security & Privacy

### Message Encryption

All WhatsApp messages are end-to-end encrypted by WhatsApp's infrastructure.

### Data Retention

- Ping requests: 30 days
- Response messages: 7 days (anonymized after processing)
- Analytics data: 1 year (aggregated, non-identifiable)

### Privacy Compliance

- No personal customer data sent in messages
- Pharmacy responses are only shared with authorized requesters
- All data handling complies with NDPR and WhatsApp Business policies 