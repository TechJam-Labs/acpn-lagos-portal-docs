# GoMed Integration API

## 🔌 Integration Overview

The GoMed integration enables seamless connectivity between the ACPN Portal and GoMed's e-commerce platform. This allows automatic pharmacist onboarding, product availability queries, and order fulfillment coordination.

## 🔑 Authentication for GoMed

GoMed uses API key authentication for all requests:

```bash
X-API-Key: gomed_key_abc123def456
X-Client-ID: gomed_client_12345
X-Signature: sha256_hash_of_request_body
```

## 📋 Core Integration Endpoints

### POST /api/v1/integrations/gomed/onboard-pharmacist

Automatically onboard an ACPN member to GoMed platform.

#### Request Body

```typescript
interface GoMedOnboardingRequest {
  acpnUserId: string
  pharmacistData: {
    pcnNumber: string
    firstName: string
    lastName: string
    email: string
    phoneNumber: string
    pharmacyDetails: {
      name: string
      address: string
      coordinates: {
        latitude: number
        longitude: number
      }
      licenseNumber: string
      operatingHours: {
        [day: string]: {
          open: string
          close: string
          isOpen: boolean
        }
      }
    }
  }
  tierLevel: 'standard' | 'premium' | 'priority'
  autoActivate: boolean
}
```

#### Example Request

```bash
POST /api/v1/integrations/gomed/onboard-pharmacist
X-API-Key: gomed_key_abc123def456
X-Client-ID: gomed_client_12345
Content-Type: application/json

{
  "acpnUserId": "user_12345",
  "pharmacistData": {
    "pcnNumber": "PCN-12345-LG",
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@healthplus.com",
    "phoneNumber": "+234 803 123 4567",
    "pharmacyDetails": {
      "name": "HealthPlus Pharmacy",
      "address": "123 Pharmacy Street, Ikoyi, Lagos",
      "coordinates": {
        "latitude": 6.5244,
        "longitude": 3.3792
      },
      "licenseNumber": "LIC-HP-2024-001",
      "operatingHours": {
        "monday": { "open": "08:00", "close": "20:00", "isOpen": true },
        "tuesday": { "open": "08:00", "close": "20:00", "isOpen": true },
        "wednesday": { "open": "08:00", "close": "20:00", "isOpen": true },
        "thursday": { "open": "08:00", "close": "20:00", "isOpen": true },
        "friday": { "open": "08:00", "close": "20:00", "isOpen": true },
        "saturday": { "open": "09:00", "close": "18:00", "isOpen": true },
        "sunday": { "open": "10:00", "close": "16:00", "isOpen": true }
      }
    }
  },
  "tierLevel": "premium",
  "autoActivate": true
}
```

#### Success Response (201)

```json
{
  "success": true,
  "data": {
    "gomedAccountId": "gomed_pharm_67890",
    "acpnUserId": "user_12345",
    "onboardingStatus": "completed",
    "tierLevel": "premium",
    "accountDetails": {
      "username": "healthplus_pharmacy",
      "profileUrl": "https://gomed.ng/pharmacy/healthplus_pharmacy",
      "dashboardUrl": "https://seller.gomed.ng/dashboard",
      "apiCredentials": {
        "sellerId": "seller_hp_12345",
        "apiKey": "***masked***"
      }
    },
    "benefits": {
      "commissionRate": 0.085,
      "priorityListing": true,
      "advancedAnalytics": true,
      "dedicatedSupport": true
    },
    "nextSteps": [
      "complete_profile",
      "upload_products",
      "verify_banking_details",
      "setup_delivery_zones"
    ],
    "activatedAt": "2024-01-15T10:30:00Z"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_onboard_12345"
  }
}
```

### POST /api/v1/integrations/gomed/product-availability-query

Query product availability across ACPN pharmacies for GoMed orders.

#### Request Body

```typescript
interface ProductAvailabilityQuery {
  orderId: string
  customerId: string
  products: Array<{
    sku: string
    name: string
    quantity: number
    maxPrice?: number
  }>
  deliveryLocation: {
    latitude: number
    longitude: number
    address: string
    city: string
    state: string
  }
  urgency: 'immediate' | 'same_day' | 'next_day' | 'flexible'
  customerPreferences?: {
    preferredPharmacies?: string[]
    maxDistance?: number
    priceRange?: 'budget' | 'mid' | 'premium'
  }
  timeoutSeconds: number
}
```

#### Example Request

```bash
POST /api/v1/integrations/gomed/product-availability-query
X-API-Key: gomed_key_abc123def456
X-Client-ID: gomed_client_12345
Content-Type: application/json

{
  "orderId": "gomed_order_789012",
  "customerId": "gomed_customer_345678",
  "products": [
    {
      "sku": "PAR-500-100",
      "name": "Paracetamol 500mg Tablets (100 count)",
      "quantity": 2,
      "maxPrice": 300
    },
    {
      "sku": "IBU-400-50",
      "name": "Ibuprofen 400mg Tablets (50 count)",
      "quantity": 1,
      "maxPrice": 500
    }
  ],
  "deliveryLocation": {
    "latitude": 6.5244,
    "longitude": 3.3792,
    "address": "456 Customer Street",
    "city": "Lagos",
    "state": "Lagos"
  },
  "urgency": "same_day",
  "customerPreferences": {
    "maxDistance": 10,
    "priceRange": "mid"
  },
  "timeoutSeconds": 300
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "queryId": "query_gomed_12345",
    "orderId": "gomed_order_789012",
    "status": "completed",
    "queryStartedAt": "2024-01-15T10:30:00Z",
    "queryCompletedAt": "2024-01-15T10:32:45Z",
    "totalResponseTime": 165,
    "pharmaciesQueried": 15,
    "pharmaciesResponded": 11,
    "responseRate": 0.73,
    "availabilityResults": [
      {
        "pharmacyId": "pharm_001",
        "gomedSellerId": "seller_hp_12345",
        "pharmacyName": "HealthPlus Pharmacy",
        "distance": 1.2,
        "rating": 4.8,
        "responseTime": 85,
        "products": [
          {
            "sku": "PAR-500-100",
            "available": true,
            "quantity": 50,
            "unitPrice": 275,
            "totalPrice": 550,
            "expiryInfo": "Dec 2025",
            "condition": "excellent"
          },
          {
            "sku": "IBU-400-50",
            "available": true,
            "quantity": 25,
            "unitPrice": 480,
            "totalPrice": 480,
            "expiryInfo": "Mar 2026",
            "condition": "excellent"
          }
        ],
        "totalOrderValue": 1030,
        "estimatedDeliveryTime": "2-3 hours",
        "deliveryFee": 500,
        "paymentMethods": ["card", "bank_transfer", "pos"],
        "score": 0.94
      },
      {
        "pharmacyId": "pharm_003",
        "gomedSellerId": "seller_qm_67890",
        "pharmacyName": "QuickMed Pharmacy",
        "distance": 2.8,
        "rating": 4.5,
        "responseTime": 120,
        "products": [
          {
            "sku": "PAR-500-100",
            "available": true,
            "quantity": 30,
            "unitPrice": 290,
            "totalPrice": 580,
            "expiryInfo": "Oct 2025",
            "condition": "good"
          },
          {
            "sku": "IBU-400-50",
            "available": false,
            "reason": "out_of_stock",
            "estimatedRestockDate": "2024-01-20"
          }
        ],
        "partialFulfillment": true,
        "totalOrderValue": 580,
        "estimatedDeliveryTime": "3-4 hours",
        "deliveryFee": 600,
        "score": 0.76
      }
    ],
    "recommendations": {
      "bestOverall": "pharm_001",
      "bestPrice": "pharm_001",
      "fastestDelivery": "pharm_001",
      "nearestLocation": "pharm_001"
    },
    "aggregatedPricing": {
      "PAR-500-100": {
        "minPrice": 275,
        "maxPrice": 290,
        "averagePrice": 282.5,
        "availableFrom": 3
      },
      "IBU-400-50": {
        "minPrice": 480,
        "maxPrice": 480,
        "averagePrice": 480,
        "availableFrom": 1
      }
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:33:00Z",
    "requestId": "req_availability_12345"
  }
}
```

### POST /api/v1/integrations/gomed/order-placement

Place an order with selected pharmacy through GoMed integration.

#### Request Body

```typescript
interface GoMedOrderPlacement {
  queryId: string
  orderId: string
  selectedPharmacyId: string
  customerInfo: {
    name: string
    phone: string
    email: string
    address: string
  }
  orderItems: Array<{
    sku: string
    quantity: number
    agreedPrice: number
  }>
  paymentMethod: 'card' | 'bank_transfer' | 'pos' | 'cash_on_delivery'
  deliveryOptions: {
    type: 'pickup' | 'delivery'
    urgency: 'immediate' | 'same_day' | 'next_day'
    preferredTime?: string
    deliveryInstructions?: string
  }
  totalAmount: number
  platformFee: number
  deliveryFee: number
}
```

#### Example Request

```bash
POST /api/v1/integrations/gomed/order-placement
X-API-Key: gomed_key_abc123def456
X-Client-ID: gomed_client_12345
Content-Type: application/json

{
  "queryId": "query_gomed_12345",
  "orderId": "gomed_order_789012",
  "selectedPharmacyId": "pharm_001",
  "customerInfo": {
    "name": "Jane Customer",
    "phone": "+234 803 987 6543",
    "email": "jane@example.com",
    "address": "456 Customer Street, Ikoyi, Lagos"
  },
  "orderItems": [
    {
      "sku": "PAR-500-100",
      "quantity": 2,
      "agreedPrice": 275
    },
    {
      "sku": "IBU-400-50",
      "quantity": 1,
      "agreedPrice": 480
    }
  ],
  "paymentMethod": "card",
  "deliveryOptions": {
    "type": "delivery",
    "urgency": "same_day",
    "preferredTime": "14:00-16:00",
    "deliveryInstructions": "Call on arrival"
  },
  "totalAmount": 1030,
  "platformFee": 103,
  "deliveryFee": 500
}
```

#### Success Response (201)

```json
{
  "success": true,
  "data": {
    "acpnOrderId": "acpn_order_12345",
    "gomedOrderId": "gomed_order_789012",
    "pharmacyOrderId": "pharm_001_ord_67890",
    "status": "confirmed",
    "pharmacy": {
      "id": "pharm_001",
      "name": "HealthPlus Pharmacy",
      "contact": "+234 803 111 2222",
      "address": "123 Pharmacy Street, Ikoyi, Lagos"
    },
    "orderSummary": {
      "items": [
        {
          "sku": "PAR-500-100",
          "name": "Paracetamol 500mg Tablets",
          "quantity": 2,
          "unitPrice": 275,
          "totalPrice": 550
        },
        {
          "sku": "IBU-400-50",
          "name": "Ibuprofen 400mg Tablets",
          "quantity": 1,
          "unitPrice": 480,
          "totalPrice": 480
        }
      ],
      "subtotal": 1030,
      "platformFee": 103,
      "deliveryFee": 500,
      "totalAmount": 1633
    },
    "fulfillment": {
      "type": "delivery",
      "estimatedTime": "2-3 hours",
      "trackingId": "track_acpn_12345",
      "driverInfo": {
        "assigned": false,
        "willAssignAt": "2024-01-15T11:00:00Z"
      }
    },
    "payment": {
      "method": "card",
      "status": "pending",
      "paymentUrl": "https://payment.gomed.ng/pay/order_789012"
    },
    "timeline": {
      "orderPlaced": "2024-01-15T10:35:00Z",
      "pharmacyConfirmed": "2024-01-15T10:35:30Z",
      "estimatedPreparation": "2024-01-15T11:30:00Z",
      "estimatedDelivery": "2024-01-15T13:30:00Z"
    },
    "notifications": {
      "smsToCustomer": true,
      "whatsappToPharmacy": true,
      "emailConfirmation": true
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:35:00Z",
    "requestId": "req_order_12345"
  }
}
```

### GET /api/v1/integrations/gomed/order-status/{orderId}

Get real-time order status and tracking information.

#### Path Parameters

- `orderId` (string): GoMed order ID

#### Example Request

```bash
GET /api/v1/integrations/gomed/order-status/gomed_order_789012
X-API-Key: gomed_key_abc123def456
X-Client-ID: gomed_client_12345
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "orderId": "gomed_order_789012",
    "acpnOrderId": "acpn_order_12345",
    "status": "in_progress",
    "substatus": "being_prepared",
    "lastUpdated": "2024-01-15T11:15:00Z",
    "pharmacy": {
      "id": "pharm_001",
      "name": "HealthPlus Pharmacy",
      "contact": "+234 803 111 2222"
    },
    "timeline": [
      {
        "stage": "order_placed",
        "status": "completed",
        "timestamp": "2024-01-15T10:35:00Z",
        "description": "Order placed by customer"
      },
      {
        "stage": "pharmacy_confirmed",
        "status": "completed",
        "timestamp": "2024-01-15T10:35:30Z",
        "description": "Pharmacy confirmed order and availability"
      },
      {
        "stage": "payment_processed",
        "status": "completed",
        "timestamp": "2024-01-15T10:40:00Z",
        "description": "Payment successfully processed"
      },
      {
        "stage": "preparation_started",
        "status": "completed",
        "timestamp": "2024-01-15T11:00:00Z",
        "description": "Pharmacy started preparing order"
      },
      {
        "stage": "preparation_complete",
        "status": "in_progress",
        "estimatedCompletion": "2024-01-15T11:30:00Z",
        "description": "Order being prepared"
      },
      {
        "stage": "driver_assigned",
        "status": "pending",
        "estimatedStart": "2024-01-15T11:30:00Z",
        "description": "Driver will be assigned for delivery"
      },
      {
        "stage": "out_for_delivery",
        "status": "pending",
        "description": "Order out for delivery"
      },
      {
        "stage": "delivered",
        "status": "pending",
        "description": "Order delivered to customer"
      }
    ],
    "delivery": {
      "type": "delivery",
      "status": "preparing",
      "estimatedDeliveryTime": "2024-01-15T13:30:00Z",
      "trackingId": "track_acpn_12345",
      "driver": null,
      "deliveryFee": 500
    },
    "contact": {
      "pharmacyPhone": "+234 803 111 2222",
      "customerServicePhone": "+234 803 450 6789",
      "supportEmail": "support@gomed.ng"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T11:15:00Z",
    "requestId": "req_status_12345"
  }
}
```

### POST /api/v1/integrations/gomed/sync-catalog

Sync product catalog between ACPN and GoMed platforms.

#### Request Body

```typescript
interface CatalogSyncRequest {
  syncType: 'full' | 'incremental' | 'verify'
  lastSyncTime?: string
  categories?: string[]
  pharmacyIds?: string[]
  dryRun?: boolean
}
```

#### Example Request

```bash
POST /api/v1/integrations/gomed/sync-catalog
X-API-Key: gomed_key_abc123def456
X-Client-ID: gomed_client_12345
Content-Type: application/json

{
  "syncType": "incremental",
  "lastSyncTime": "2024-01-15T00:00:00Z",
  "categories": ["analgesics", "antibiotics", "vitamins"],
  "dryRun": false
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "syncId": "sync_12345abc",
    "syncType": "incremental",
    "startedAt": "2024-01-15T11:20:00Z",
    "completedAt": "2024-01-15T11:25:30Z",
    "duration": 330,
    "summary": {
      "productsProcessed": 1250,
      "productsAdded": 45,
      "productsUpdated": 128,
      "productsRemoved": 12,
      "categoriesProcessed": 3,
      "pharmaciesAffected": 25,
      "errors": 2
    },
    "changes": {
      "newProducts": [
        {
          "sku": "VIT-C-1000-30",
          "name": "Vitamin C 1000mg Tablets (30 count)",
          "category": "vitamins",
          "addedBy": 5,
          "averagePrice": 850
        }
      ],
      "updatedProducts": [
        {
          "sku": "PAR-500-100",
          "name": "Paracetamol 500mg Tablets",
          "changes": {
            "averagePrice": { "from": 275, "to": 280 },
            "availability": { "from": "85%", "to": "92%" }
          }
        }
      ],
      "removedProducts": [
        {
          "sku": "OLD-PROD-123",
          "name": "Discontinued Product",
          "reason": "manufacturer_discontinued"
        }
      ]
    },
    "errors": [
      {
        "sku": "ERR-PROD-001",
        "error": "Invalid price format",
        "pharmacyId": "pharm_error_01"
      }
    ],
    "nextSyncRecommended": "2024-01-16T11:20:00Z"
  },
  "meta": {
    "timestamp": "2024-01-15T11:25:30Z",
    "requestId": "req_sync_12345"
  }
}
```

## 📊 Analytics and Intelligence

### POST /api/v1/integrations/gomed/intelligence/market-insights

Push market intelligence data to GoMed for product decisions.

#### Request Body

```typescript
interface MarketIntelligenceData {
  reportType: 'demand_analysis' | 'price_trends' | 'availability_gaps' | 'new_products'
  timeframe: 'daily' | 'weekly' | 'monthly'
  data: {
    demandedProducts: Array<{
      sku?: string
      productName: string
      requestCount: number
      fulfillmentRate: number
      averagePrice: number
      geographicDemand: {
        [location: string]: number
      }
      seasonality: {
        [month: string]: number
      }
    }>
    gapAnalysis: Array<{
      productName: string
      currentAvailability: number
      projectedDemand: number
      potentialRevenue: number
      recommendedAction: string
    }>
    pricingIntelligence: Array<{
      sku: string
      marketPrice: {
        min: number
        max: number
        average: number
        median: number
      }
      priceElasticity: number
      competitorAnalysis: {
        [competitor: string]: number
      }
    }>
  }
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "reportId": "intel_report_12345",
    "reportType": "demand_analysis",
    "processedAt": "2024-01-15T12:00:00Z",
    "insights": {
      "highDemandProducts": [
        {
          "productName": "Paracetamol 500mg",
          "demandGrowth": 0.15,
          "recommendation": "Increase inventory by 25%"
        }
      ],
      "marketOpportunities": [
        {
          "category": "Vitamins",
          "potentialRevenue": 2500000,
          "currentGap": 0.35,
          "recommendation": "Expand vitamin product range"
        }
      ],
      "actionItems": [
        "Consider bulk procurement program for high-demand analgesics",
        "Explore partnerships for vitamin suppliers",
        "Implement dynamic pricing for seasonal products"
      ]
    },
    "gomedIntegration": {
      "dataShared": true,
      "categoriesUpdated": 5,
      "pricingModelsAdjusted": 12,
      "newProductSuggestions": 8
    }
  },
  "meta": {
    "timestamp": "2024-01-15T12:00:00Z",
    "requestId": "req_intelligence_12345"
  }
}
```

## 🔒 Security and Compliance

### API Security

- All requests are authenticated with API keys and signatures
- Request signatures use HMAC-SHA256 with shared secret
- Rate limiting: 1000 requests per hour per API key
- IP whitelisting for production environments

### Data Privacy

- Customer data is only shared with explicit consent
- All data transmission uses TLS 1.3 encryption
- Data retention policies align with NDPR requirements
- PII is anonymized in analytics reports

### Error Handling

```typescript
interface GoMedErrorResponse {
  success: false
  error: {
    code: string
    message: string
    details?: any
  }
  meta: {
    timestamp: string
    requestId: string
  }
}
```

### Common Error Codes

| Code | Description | HTTP Status |
|------|-------------|-------------|
| `GOMED_API_KEY_INVALID` | Invalid API key | 401 |
| `GOMED_SIGNATURE_MISMATCH` | Request signature invalid | 401 |
| `GOMED_RATE_LIMIT_EXCEEDED` | Too many requests | 429 |
| `GOMED_PHARMACY_NOT_FOUND` | Pharmacy not exists on GoMed | 404 |
| `GOMED_PRODUCT_NOT_MAPPED` | Product SKU not mapped | 422 |
| `GOMED_ORDER_PROCESSING_ERROR` | Order processing failed | 500 |
| `GOMED_SYNC_CONFLICT` | Catalog sync conflict | 409 |

## 📈 Performance Monitoring

### SLA Metrics

| Operation | Target Response Time | Availability |
|-----------|---------------------|-------------|
| Product Availability Query | < 5 seconds | 99.9% |
| Order Placement | < 3 seconds | 99.9% |
| Order Status Update | < 1 second | 99.9% |
| Catalog Sync | < 60 seconds | 99.5% |

### Monitoring Endpoints

```bash
GET /api/v1/integrations/gomed/health
GET /api/v1/integrations/gomed/metrics
GET /api/v1/integrations/gomed/status
``` 