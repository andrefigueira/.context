# API: Endpoint Reference

This document provides comprehensive documentation for all REST API endpoints in [Your Project Name], including request/response formats, authentication requirements, and usage examples.

> **Note**: If you already use structured API documentation like Swagger/OpenAPI, you can simply reference that specification here and ensure it's accessible to command line AI tools. This provides machine-readable API definitions that complement the human-readable examples below.

## Base Configuration

### Base URL
```
Production: https://api.yourproject.com/v1
Staging:    https://staging-api.yourproject.com/v1
Local:      http://localhost:8080/api/v1
```

### API Versioning
- **Current Version**: v1
- **Version Header**: `Accept: application/vnd.yourproject.v1+json`
- **URL Versioning**: `/api/v1/` prefix for all endpoints

## Authentication Endpoints

### POST /auth/register
Register a new user account.

**Authentication**: None required

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "name": "John Doe"
}
```

**Response** (201 Created):
```json
{
  "id": "user_12345",
  "email": "user@example.com",
  "name": "John Doe",
  "roles": ["user"],
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Error Responses**:
```json
// 400 Bad Request - Validation Error
{
  "error": {
    "code": "validation_failed",
    "message": "Request validation failed",
    "details": {
      "email": "Invalid email format",
      "password": "Password must be at least 12 characters"
    }
  }
}

// 409 Conflict - Email Already Exists
{
  "error": {
    "code": "duplicate_email",
    "message": "Email already exists"
  }
}
```

### POST /auth/login
Authenticate user and receive access token.

**Authentication**: None required

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```

**Response** (200 OK):
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900,
  "refresh_token": "rt_abc123def456...",
  "user": {
    "id": "user_12345",
    "email": "user@example.com",
    "name": "John Doe",
    "roles": ["user"]
  }
}
```

**Error Responses**:
```json
// 401 Unauthorized - Invalid Credentials
{
  "error": {
    "code": "invalid_credentials",
    "message": "Invalid email or password"
  }
}

// 429 Too Many Requests - Rate Limited
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Too many login attempts, try again later",
    "retry_after": 60
  }
}
```

### POST /auth/refresh
Refresh access token using refresh token.

**Authentication**: Refresh token (cookie or request body)

**Request Body** (optional, if not using cookie):
```json
{
  "refresh_token": "rt_abc123def456..."
}
```

**Response** (200 OK):
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

### POST /auth/logout
Logout user and revoke tokens.

**Authentication**: Bearer token required

**Response** (200 OK):
```json
{
  "message": "Logged out successfully"
}
```

## User Management Endpoints

### GET /users/profile
Get current user profile.

**Authentication**: Bearer token required
**Permissions**: None (own profile)

**Response** (200 OK):
```json
{
  "id": "user_12345",
  "email": "user@example.com",
  "name": "John Doe",
  "roles": ["user"],
  "profile": {
    "bio": "Software developer",
    "location": "San Francisco, CA",
    "website": "https://johndoe.dev"
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

### PUT /users/profile
Update current user profile.

**Authentication**: Bearer token required
**Permissions**: None (own profile)

**Request Body**:
```json
{
  "name": "John Smith",
  "profile": {
    "bio": "Senior Software Developer",
    "location": "Seattle, WA",
    "website": "https://johnsmith.dev"
  }
}
```

**Response** (200 OK):
```json
{
  "id": "user_12345",
  "email": "user@example.com",
  "name": "John Smith",
  "profile": {
    "bio": "Senior Software Developer",
    "location": "Seattle, WA",
    "website": "https://johnsmith.dev"
  },
  "updated_at": "2024-01-15T14:20:00Z"
}
```

### POST /users/change-password
Change user password.

**Authentication**: Bearer token required
**Permissions**: None (own account)

**Request Body**:
```json
{
  "current_password": "OldPassword123!",
  "new_password": "NewSecurePassword456!"
}
```

**Response** (200 OK):
```json
{
  "message": "Password changed successfully"
}
```

**Error Responses**:
```json
// 400 Bad Request - Invalid Current Password
{
  "error": {
    "code": "invalid_current_password",
    "message": "Current password is incorrect"
  }
}
```

### GET /users
List users (admin only).

**Authentication**: Bearer token required
**Permissions**: `admin:panel`

**Query Parameters**:
- `page` (int): Page number (default: 1)
- `limit` (int): Items per page (default: 20, max: 100)
- `search` (string): Search by name or email
- `role` (string): Filter by role
- `status` (string): Filter by status (active, inactive, suspended)

**Example Request**:
```
GET /users?page=1&limit=20&search=john&role=user&status=active
```

**Response** (200 OK):
```json
{
  "users": [
    {
      "id": "user_12345",
      "email": "user@example.com",
      "name": "John Doe",
      "roles": ["user"],
      "status": "active",
      "created_at": "2024-01-15T10:30:00Z",
      "last_active": "2024-01-20T09:15:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_count": 89,
    "has_next": true,
    "has_prev": false
  }
}
```

### GET /users/{user_id}
Get specific user by ID (admin only).

**Authentication**: Bearer token required
**Permissions**: `admin:panel`

**Response** (200 OK):
```json
{
  "id": "user_12345",
  "email": "user@example.com",
  "name": "John Doe",
  "roles": ["user"],
  "status": "active",
  "profile": {
    "bio": "Software developer",
    "location": "San Francisco, CA"
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "last_active": "2024-01-20T09:15:00Z"
}
```

### PUT /users/{user_id}/roles
Update user roles (admin only).

**Authentication**: Bearer token required
**Permissions**: `admin:panel`

**Request Body**:
```json
{
  "roles": ["user", "moderator"]
}
```

**Response** (200 OK):
```json
{
  "id": "user_12345",
  "email": "user@example.com",
  "name": "John Doe",
  "roles": ["user", "moderator"],
  "updated_at": "2024-01-20T10:30:00Z"
}
```

### PUT /users/{user_id}/status
Update user status (admin only).

**Authentication**: Bearer token required
**Permissions**: `admin:panel`

**Request Body**:
```json
{
  "status": "suspended",
  "reason": "Violation of terms of service"
}
```

**Response** (200 OK):
```json
{
  "id": "user_12345",
  "status": "suspended",
  "status_reason": "Violation of terms of service",
  "updated_at": "2024-01-20T10:30:00Z"
}
```

## Health and Monitoring Endpoints

### GET /health
Health check endpoint.

**Authentication**: None required

**Response** (200 OK):
```json
{
  "status": "healthy",
  "timestamp": "2024-01-20T10:30:00Z",
  "version": "1.2.3",
  "services": {
    "database": "healthy",
    "cache": "healthy",
    "queue": "healthy"
  }
}
```

**Response** (503 Service Unavailable):
```json
{
  "status": "unhealthy",
  "timestamp": "2024-01-20T10:30:00Z",
  "version": "1.2.3",
  "services": {
    "database": "unhealthy",
    "cache": "healthy",
    "queue": "healthy"
  }
}
```

### GET /metrics
Prometheus metrics endpoint (monitoring systems only).

**Authentication**: None required (but should be restricted at network level)

**Response** (200 OK):
```
# HELP api_requests_total Total number of API requests
# TYPE api_requests_total counter
api_requests_total{method="GET",path="/users/profile",status="200"} 1234
api_requests_total{method="POST",path="/auth/login",status="200"} 567
api_requests_total{method="POST",path="/auth/login",status="401"} 89

# HELP api_request_duration_seconds API request duration
# TYPE api_request_duration_seconds histogram
api_request_duration_seconds_bucket{method="GET",path="/users/profile",le="0.1"} 100
api_request_duration_seconds_bucket{method="GET",path="/users/profile",le="0.5"} 200
api_request_duration_seconds_bucket{method="GET",path="/users/profile",le="1.0"} 250
api_request_duration_seconds_bucket{method="GET",path="/users/profile",le="+Inf"} 300
```

## Error Response Format

All error responses follow a consistent structure:

```json
{
  "error": {
    "code": "error_code",
    "message": "Human-readable error message",
    "details": {
      "field": "Specific field error message"
    },
    "trace_id": "abc123def456",
    "timestamp": "2024-01-20T10:30:00Z"
  }
}
```

### Common Error Codes
- `validation_failed`: Request validation errors
- `unauthorized`: Authentication required
- `forbidden`: Insufficient permissions
- `not_found`: Resource not found
- `conflict`: Resource already exists
- `rate_limit_exceeded`: Too many requests
- `internal_error`: Server error
- `service_unavailable`: Service temporarily unavailable

## Rate Limiting

### Rate Limit Headers
All responses include rate limit headers:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642675800
X-RateLimit-Window: 3600
```

### Rate Limit Tiers
- **Authentication endpoints**: 5 requests per minute per IP
- **User management**: 60 requests per hour per user
- **Admin endpoints**: 300 requests per hour per user
- **General API**: 1000 requests per hour per user

## Pagination

All list endpoints support cursor-based pagination:

**Query Parameters**:
- `page`: Page number (1-based)
- `limit`: Items per page (max 100)
- `cursor`: Opaque cursor for next page

**Response Format**:
```json
{
  "data": [...],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_count": 89,
    "has_next": true,
    "has_prev": false,
    "next_cursor": "eyJpZCI6IjEyMyJ9",
    "prev_cursor": null
  }
}
```

---

**Next**: Review [headers.md](./headers.md) for HTTP headers documentation and [examples.md](./examples.md) for comprehensive usage examples.