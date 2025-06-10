# Authentication API

## 🔐 Authentication Overview

The ACPN Portal uses JWT-based authentication with role-based access control (RBAC). Authentication supports multiple user types including pharmacists, doctors, hospitals, and administrators.

## 📋 Endpoints

### POST /api/v1/auth/register

Register a new user account with role-specific verification.

#### Request Body

```typescript
interface RegisterRequest {
  email: string
  password: string
  confirmPassword: string
  role: 'pharmacist' | 'doctor' | 'hospital'
  userData: {
    firstName: string
    lastName: string
    phoneNumber: string
    address: Address
  }
  verification: {
    pcnNumber?: string        // Required for pharmacists
    medicalLicense?: string   // Required for doctors
    facilityCode?: string     // Required for hospitals
  }
  acceptTerms: boolean
  acceptPrivacy: boolean
}

interface Address {
  street: string
  city: string
  state: string
  country: string
  postalCode?: string
  coordinates?: {
    latitude: number
    longitude: number
  }
}
```

#### Example Request

```bash
POST /api/v1/auth/register
Content-Type: application/json

{
  "email": "john.doe@healthplus.com",
  "password": "SecurePass123!",
  "confirmPassword": "SecurePass123!",
  "role": "pharmacist",
  "userData": {
    "firstName": "John",
    "lastName": "Doe",
    "phoneNumber": "+234 803 123 4567",
    "address": {
      "street": "123 Pharmacy Street",
      "city": "Lagos",
      "state": "Lagos",
      "country": "Nigeria",
      "coordinates": {
        "latitude": 6.5244,
        "longitude": 3.3792
      }
    }
  },
  "verification": {
    "pcnNumber": "PCN-12345-LG"
  },
  "acceptTerms": true,
  "acceptPrivacy": true
}
```

#### Success Response (201)

```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_12345",
      "email": "john.doe@healthplus.com",
      "role": "pharmacist",
      "verified": false,
      "createdAt": "2024-01-15T10:30:00Z"
    },
    "verificationRequired": true,
    "nextSteps": [
      "verify_email",
      "verify_pcn",
      "complete_profile"
    ]
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_reg_12345"
  }
}
```

#### Error Response (422)

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Registration validation failed",
    "details": {
      "email": ["Email already exists"],
      "pcnNumber": ["PCN number is invalid or already registered"],
      "password": ["Password must contain at least 8 characters with uppercase, lowercase, numbers and special characters"]
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_reg_12345"
  }
}
```

### POST /api/v1/auth/login

Authenticate user and receive access tokens.

#### Request Body

```typescript
interface LoginRequest {
  email: string
  password: string
  rememberMe?: boolean
  deviceInfo?: {
    userAgent: string
    platform: string
    ipAddress: string
  }
  location?: {
    latitude: number
    longitude: number
  }
}
```

#### Example Request

```bash
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "john.doe@healthplus.com",
  "password": "SecurePass123!",
  "rememberMe": true,
  "deviceInfo": {
    "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    "platform": "web",
    "ipAddress": "192.168.1.100"
  },
  "location": {
    "latitude": 6.5244,
    "longitude": 3.3792
  }
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_12345",
      "email": "john.doe@healthplus.com",
      "role": "pharmacist",
      "firstName": "John",
      "lastName": "Doe",
      "verified": true,
      "pharmacyId": "pharm_67890",
      "permissions": [
        "pharmacy:read",
        "pharmacy:write",
        "marketplace:access",
        "events:register"
      ],
      "lastLogin": "2024-01-15T10:30:00Z",
      "profileComplete": true
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "rt_abc123def456...",
      "expiresIn": 3600,
      "tokenType": "Bearer"
    },
    "session": {
      "id": "sess_xyz789",
      "expiresAt": "2024-01-15T11:30:00Z"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_login_12345"
  }
}
```

#### Error Response (401)

```json
{
  "success": false,
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password",
    "details": {
      "attempts": 3,
      "remainingAttempts": 2,
      "lockoutTime": null
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_login_12345"
  }
}
```

### POST /api/v1/auth/refresh

Refresh expired access token using refresh token.

#### Request Body

```typescript
interface RefreshRequest {
  refreshToken: string
}
```

#### Example Request

```bash
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "rt_abc123def456..."
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "rt_new123def456...",
      "expiresIn": 3600,
      "tokenType": "Bearer"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T11:30:00Z",
    "requestId": "req_refresh_12345"
  }
}
```

### POST /api/v1/auth/logout

Logout user and invalidate tokens.

#### Request Headers

```bash
Authorization: Bearer <access_token>
```

#### Request Body

```typescript
interface LogoutRequest {
  refreshToken?: string
  logoutAllDevices?: boolean
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "message": "Successfully logged out",
    "tokensInvalidated": 2
  },
  "meta": {
    "timestamp": "2024-01-15T12:00:00Z",
    "requestId": "req_logout_12345"
  }
}
```

### POST /api/v1/auth/verify-email

Verify user email address with verification code.

#### Request Body

```typescript
interface VerifyEmailRequest {
  email: string
  code: string
}
```

#### Example Request

```bash
POST /api/v1/auth/verify-email
Content-Type: application/json

{
  "email": "john.doe@healthplus.com",
  "code": "123456"
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "verified": true,
    "user": {
      "id": "user_12345",
      "emailVerified": true,
      "verifiedAt": "2024-01-15T10:35:00Z"
    },
    "nextSteps": ["verify_pcn", "complete_profile"]
  },
  "meta": {
    "timestamp": "2024-01-15T10:35:00Z",
    "requestId": "req_verify_12345"
  }
}
```

### POST /api/v1/auth/resend-verification

Resend email verification code.

#### Request Body

```typescript
interface ResendVerificationRequest {
  email: string
  type: 'email' | 'sms'
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "sent": true,
    "method": "email",
    "expiresIn": 600,
    "canResendAfter": 60
  },
  "meta": {
    "timestamp": "2024-01-15T10:40:00Z",
    "requestId": "req_resend_12345"
  }
}
```

### POST /api/v1/auth/forgot-password

Initiate password reset process.

#### Request Body

```typescript
interface ForgotPasswordRequest {
  email: string
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "message": "Password reset instructions sent to email",
    "resetTokenSent": true,
    "expiresIn": 1800
  },
  "meta": {
    "timestamp": "2024-01-15T10:45:00Z",
    "requestId": "req_forgot_12345"
  }
}
```

### POST /api/v1/auth/reset-password

Reset password using reset token.

#### Request Body

```typescript
interface ResetPasswordRequest {
  token: string
  newPassword: string
  confirmPassword: string
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "passwordReset": true,
    "message": "Password successfully reset",
    "autoLogin": false
  },
  "meta": {
    "timestamp": "2024-01-15T10:50:00Z",
    "requestId": "req_reset_12345"
  }
}
```

### POST /api/v1/auth/change-password

Change password for authenticated user.

#### Request Headers

```bash
Authorization: Bearer <access_token>
```

#### Request Body

```typescript
interface ChangePasswordRequest {
  currentPassword: string
  newPassword: string
  confirmPassword: string
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "passwordChanged": true,
    "invalidatedSessions": 3,
    "message": "Password successfully changed"
  },
  "meta": {
    "timestamp": "2024-01-15T11:00:00Z",
    "requestId": "req_change_12345"
  }
}
```

### POST /api/v1/auth/verify-geolocation

Verify pharmacy location for pharmacist accounts.

#### Request Headers

```bash
Authorization: Bearer <access_token>
```

#### Request Body

```typescript
interface VerifyGeolocationRequest {
  coordinates: {
    latitude: number
    longitude: number
    accuracy: number
  }
  address: string
  pharmacyId?: string
  verificationMethod: 'gps' | 'manual' | 'document'
  supportingDocuments?: string[]
}
```

#### Example Request

```bash
POST /api/v1/auth/verify-geolocation
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "coordinates": {
    "latitude": 6.5244,
    "longitude": 3.3792,
    "accuracy": 10
  },
  "address": "123 Pharmacy Street, Ikoyi, Lagos",
  "pharmacyId": "pharm_67890",
  "verificationMethod": "gps"
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "verified": true,
    "confidence": 0.95,
    "verificationId": "geo_verify_12345",
    "pharmacy": {
      "id": "pharm_67890",
      "locationVerified": true,
      "verifiedAt": "2024-01-15T11:10:00Z",
      "coordinates": {
        "latitude": 6.5244,
        "longitude": 3.3792
      }
    },
    "nextSteps": ["complete_profile"]
  },
  "meta": {
    "timestamp": "2024-01-15T11:10:00Z",
    "requestId": "req_geo_12345"
  }
}
```

### GET /api/v1/auth/profile

Get current user profile information.

#### Request Headers

```bash
Authorization: Bearer <access_token>
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_12345",
      "email": "john.doe@healthplus.com",
      "role": "pharmacist",
      "firstName": "John",
      "lastName": "Doe",
      "phoneNumber": "+234 803 123 4567",
      "profileImage": "https://storage.acpnlagos.org/profiles/user_12345.jpg",
      "verified": true,
      "emailVerified": true,
      "phoneVerified": true,
      "locationVerified": true,
      "profileComplete": true,
      "createdAt": "2024-01-10T09:00:00Z",
      "lastLogin": "2024-01-15T10:30:00Z",
      "permissions": [
        "pharmacy:read",
        "pharmacy:write",
        "marketplace:access",
        "events:register"
      ],
      "pharmacy": {
        "id": "pharm_67890",
        "name": "HealthPlus Pharmacy",
        "role": "owner"
      },
      "preferences": {
        "language": "en",
        "timezone": "Africa/Lagos",
        "notifications": {
          "email": true,
          "sms": false,
          "push": true
        }
      }
    }
  },
  "meta": {
    "timestamp": "2024-01-15T11:15:00Z",
    "requestId": "req_profile_12345"
  }
}
```

### POST /api/v1/auth/2fa/setup

Set up two-factor authentication.

#### Request Headers

```bash
Authorization: Bearer <access_token>
```

#### Request Body

```typescript
interface Setup2FARequest {
  method: 'totp' | 'sms'
  phoneNumber?: string  // Required for SMS
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "method": "totp",
    "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgA...",
    "secret": "JBSWY3DPEHPK3PXP",
    "backupCodes": [
      "12345678",
      "87654321",
      "13579246",
      "24681357"
    ]
  },
  "meta": {
    "timestamp": "2024-01-15T11:20:00Z",
    "requestId": "req_2fa_setup_12345"
  }
}
```

### POST /api/v1/auth/2fa/verify

Verify two-factor authentication code.

#### Request Headers

```bash
Authorization: Bearer <access_token>
```

#### Request Body

```typescript
interface Verify2FARequest {
  code: string
  method: 'totp' | 'sms' | 'backup'
}
```

#### Success Response (200)

```json
{
  "success": true,
  "data": {
    "verified": true,
    "user": {
      "id": "user_12345",
      "twoFactorEnabled": true,
      "enabledAt": "2024-01-15T11:25:00Z"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T11:25:00Z",
    "requestId": "req_2fa_verify_12345"
  }
}
```

## 🔒 JWT Token Structure

### Access Token Payload

```typescript
interface JWTPayload {
  sub: string              // User ID
  email: string
  role: 'pharmacist' | 'doctor' | 'hospital' | 'admin'
  permissions: string[]
  pharmacyId?: string      // For pharmacists
  verified: boolean
  sessionId: string
  iat: number             // Issued at
  exp: number             // Expires at
  iss: string             // Issuer
  aud: string             // Audience
}
```

### Example Decoded JWT

```json
{
  "sub": "user_12345",
  "email": "john.doe@healthplus.com",
  "role": "pharmacist",
  "permissions": [
    "pharmacy:read",
    "pharmacy:write",
    "marketplace:access"
  ],
  "pharmacyId": "pharm_67890",
  "verified": true,
  "sessionId": "sess_xyz789",
  "iat": 1642248000,
  "exp": 1642251600,
  "iss": "acpn-lagos-api",
  "aud": "acpn-lagos-portal"
}
```

## 🛡️ Security Considerations

### Password Requirements

- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character
- Cannot be a common password
- Cannot contain user's email or name

### Account Lockout

- 5 failed login attempts trigger lockout
- Lockout duration: 15 minutes (progressive)
- Account can be unlocked via email

### Session Management

- Access token expires in 1 hour
- Refresh token expires in 30 days (or 1 year if "remember me")
- Maximum 5 concurrent sessions per user
- Sessions are invalidated on password change

### Rate Limiting

- Login: 10 attempts per IP per minute
- Registration: 5 attempts per IP per hour
- Password reset: 3 attempts per IP per hour
- Verification codes: 5 requests per user per hour

## 🔍 Error Codes

| Code | Description |
|------|-------------|
| `VALIDATION_ERROR` | Request validation failed |
| `INVALID_CREDENTIALS` | Wrong email/password |
| `ACCOUNT_LOCKED` | Too many failed attempts |
| `ACCOUNT_SUSPENDED` | Account suspended by admin |
| `EMAIL_NOT_VERIFIED` | Email verification required |
| `VERIFICATION_EXPIRED` | Verification code expired |
| `VERIFICATION_INVALID` | Invalid verification code |
| `TOKEN_EXPIRED` | JWT token expired |
| `TOKEN_INVALID` | Invalid JWT token |
| `REFRESH_TOKEN_EXPIRED` | Refresh token expired |
| `2FA_REQUIRED` | Two-factor authentication required |
| `2FA_INVALID` | Invalid 2FA code |
| `PERMISSION_DENIED` | Insufficient permissions |
| `PCN_VERIFICATION_FAILED` | PCN number verification failed |
| `GEOLOCATION_VERIFICATION_FAILED` | Location verification failed | 