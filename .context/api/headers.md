# API: Headers Documentation

This document defines all HTTP headers used in [Your Project Name] API, including authentication headers, content negotiation, security headers, and custom application headers.

## Authentication Headers

### Authorization Header
**Required for**: All protected endpoints
**Format**: `Authorization: Bearer <access_token>`

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiMTIzIiwiZW1haWwiOiJ1c2VyQGV4YW1wbGUuY29tIiwicm9sZXMiOlsidXNlciJdLCJleHAiOjE2NDI2NzU4MDB9.signature
```

**Error Responses**:
```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="API", error="invalid_token", error_description="The access token is invalid"
```

### Refresh Token Cookie
**Used for**: Token refresh operations
**Format**: HTTP-only secure cookie

```http
Set-Cookie: refresh_token=rt_abc123def456; HttpOnly; Secure; SameSite=Strict; Path=/api/v1/auth; Max-Age=604800
```

## Content Negotiation Headers

### Accept Header
**Purpose**: Specify desired response format and API version
**Default**: `application/json`

```http
Accept: application/json
Accept: application/vnd.yourproject.v1+json
Accept: application/xml
```

### Content-Type Header
**Purpose**: Specify request body format
**Required for**: POST, PUT, PATCH requests with body

```http
Content-Type: application/json
Content-Type: application/x-www-form-urlencoded
Content-Type: multipart/form-data
```

### Accept-Encoding Header
**Purpose**: Specify supported compression algorithms

```http
Accept-Encoding: gzip, deflate, br
```

**Response**:
```http
Content-Encoding: gzip
```

## Security Headers

### X-Request-ID Header
**Purpose**: Request tracing and debugging
**Format**: UUID v4
**Usage**: Include in requests for better support

```http
X-Request-ID: 550e8400-e29b-41d4-a716-446655440000
```

**Response Echo**:
```http
X-Request-ID: 550e8400-e29b-41d4-a716-446655440000
```

### X-API-Key Header (Optional)
**Purpose**: API key authentication for service-to-service calls
**Format**: `X-API-Key: <api_key>`

```http
X-API-Key: ak_live_1234567890abcdef
```

### User-Agent Header
**Purpose**: Client identification and analytics
**Recommended**: Include meaningful client information

```http
User-Agent: YourApp/1.2.3 (iOS 15.0; iPhone13,2)
User-Agent: YourWebApp/2.1.0 (Mozilla/5.0; compatible)
User-Agent: YourCLI/1.0.0 (go1.19; linux/amd64)
```

## Cache Control Headers

### Cache-Control (Response)
**Purpose**: Control client-side caching behavior

```http
# Public endpoints (cacheable)
Cache-Control: public, max-age=3600

# User-specific data (private caching)
Cache-Control: private, max-age=300

# No caching for sensitive data
Cache-Control: no-cache, no-store, must-revalidate
```

### ETag (Response)
**Purpose**: Entity versioning for conditional requests

```http
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

**Conditional Request**:
```http
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

**Response** (304 Not Modified):
```http
HTTP/1.1 304 Not Modified
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

### Last-Modified (Response)
**Purpose**: Resource modification time for conditional requests

```http
Last-Modified: Wed, 15 Jan 2024 10:30:00 GMT
```

**Conditional Request**:
```http
If-Modified-Since: Wed, 15 Jan 2024 10:30:00 GMT
```

## Rate Limiting Headers

### X-RateLimit-* (Response)
**Purpose**: Communicate rate limit status to clients

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642675800
X-RateLimit-Window: 3600
X-RateLimit-Retry-After: 60
```

**Rate Limit Exceeded Response**:
```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1642675800
X-RateLimit-Retry-After: 300
Retry-After: 300

{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Try again in 5 minutes."
  }
}
```

## Pagination Headers

### Link Header (Response)
**Purpose**: Provide pagination navigation links

```http
Link: <https://api.yourproject.com/v1/users?page=1&limit=20>; rel="first",
      <https://api.yourproject.com/v1/users?page=2&limit=20>; rel="next",
      <https://api.yourproject.com/v1/users?page=5&limit=20>; rel="last"
```

### X-Total-Count (Response)
**Purpose**: Total number of items available

```http
X-Total-Count: 1234
X-Page-Count: 62
X-Current-Page: 1
X-Per-Page: 20
```

## CORS Headers

### Access-Control-Allow-* (Response)
**Purpose**: Cross-Origin Resource Sharing configuration

```http
Access-Control-Allow-Origin: https://app.yourproject.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, X-Request-ID
Access-Control-Expose-Headers: X-RateLimit-Remaining, X-Request-ID
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

### Preflight Request Example
**OPTIONS Request**:
```http
OPTIONS /api/v1/users HTTP/1.1
Origin: https://app.yourproject.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type
```

**Preflight Response**:
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.yourproject.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, X-Request-ID
Access-Control-Max-Age: 86400
```

## Custom Application Headers

### X-Client-Version Header
**Purpose**: Client version tracking for compatibility

```http
X-Client-Version: 2.1.0
X-Client-Platform: ios
X-Client-Build: 1234
```

### X-Feature-Flags Header
**Purpose**: Enable/disable features for specific requests

```http
X-Feature-Flags: new-ui=true,beta-features=false
```

### X-Idempotency-Key Header
**Purpose**: Ensure idempotent operations for sensitive actions

```http
X-Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

**Usage Example**:
```http
POST /api/v1/payments HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
X-Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

{
  "amount": 1000,
  "currency": "USD",
  "description": "Payment for order #1234"
}
```

## Error Response Headers

### WWW-Authenticate Header
**Purpose**: Indicate authentication method for 401 responses

```http
WWW-Authenticate: Bearer realm="API"
WWW-Authenticate: Bearer realm="API", error="invalid_token", error_description="Token has expired"
```

### X-Error-Code Header
**Purpose**: Machine-readable error identification

```http
X-Error-Code: VALIDATION_FAILED
X-Error-Details: email:required,password:min_length
```

## Response Time Headers

### X-Response-Time Header
**Purpose**: Server processing time for performance monitoring

```http
X-Response-Time: 0.123s
X-DB-Time: 0.045s
X-Cache-Status: HIT
```

### Server Header
**Purpose**: Server identification (minimal information for security)

```http
Server: YourProject/1.0
```

## Health Check Headers

### X-Health-Check Header
**Purpose**: Identify health check requests for logging/monitoring

```http
X-Health-Check: true
X-Monitoring-Source: datadog
```

## Implementation Examples

### Middleware Header Processing
```go
func headerMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Add security headers
        w.Header().Set("X-Content-Type-Options", "nosniff")
        w.Header().Set("X-Frame-Options", "DENY")
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        
        // Add CORS headers
        origin := r.Header.Get("Origin")
        if isAllowedOrigin(origin) {
            w.Header().Set("Access-Control-Allow-Origin", origin)
            w.Header().Set("Access-Control-Allow-Credentials", "true")
        }
        
        // Add request ID
        requestID := r.Header.Get("X-Request-ID")
        if requestID == "" {
            requestID = generateUUID()
        }
        w.Header().Set("X-Request-ID", requestID)
        
        // Add response time tracking
        start := time.Now()
        next.ServeHTTP(w, r)
        
        duration := time.Since(start)
        w.Header().Set("X-Response-Time", fmt.Sprintf("%.3fs", duration.Seconds()))
    })
}
```

### Rate Limiting Headers
```go
func addRateLimitHeaders(w http.ResponseWriter, limit, remaining int, resetTime time.Time) {
    w.Header().Set("X-RateLimit-Limit", strconv.Itoa(limit))
    w.Header().Set("X-RateLimit-Remaining", strconv.Itoa(remaining))
    w.Header().Set("X-RateLimit-Reset", strconv.FormatInt(resetTime.Unix(), 10))
    w.Header().Set("X-RateLimit-Window", "3600")
    
    if remaining == 0 {
        retryAfter := int(time.Until(resetTime).Seconds())
        w.Header().Set("Retry-After", strconv.Itoa(retryAfter))
        w.Header().Set("X-RateLimit-Retry-After", strconv.Itoa(retryAfter))
    }
}
```

## Header Validation

### Required Headers Validation
```go
type RequiredHeaders struct {
    ContentType   bool
    Authorization bool
    UserAgent     bool
    AcceptVersion bool
}

func validateHeaders(r *http.Request, required RequiredHeaders) error {
    if required.ContentType && r.Header.Get("Content-Type") == "" {
        return &HeaderError{
            Code:    "missing_content_type",
            Message: "Content-Type header is required",
        }
    }
    
    if required.Authorization && r.Header.Get("Authorization") == "" {
        return &HeaderError{
            Code:    "missing_authorization",
            Message: "Authorization header is required",
        }
    }
    
    if required.UserAgent && r.Header.Get("User-Agent") == "" {
        return &HeaderError{
            Code:    "missing_user_agent",
            Message: "User-Agent header is required",
        }
    }
    
    return nil
}
```

## Decision History & Trade-offs

### Authorization Header Format
**Decision**: Use standard Bearer token format instead of custom header
**Rationale**:
- Standard HTTP authentication mechanism
- Excellent tooling and library support
- Compatible with OAuth 2.0 and OpenID Connect
- Clear separation between authentication and other headers

**Trade-offs**:
- Bearer tokens visible in logs and debugging tools
- Requires HTTPS for security
- No built-in token rotation mechanism in header

### Custom X-Request-ID Header
**Decision**: Implement request ID tracking for all requests
**Rationale**:
- Essential for distributed system debugging
- Enables request tracing across services
- Improves customer support capabilities
- Minimal performance overhead

**Trade-offs**:
- Additional header processing overhead
- Client burden to include meaningful request IDs
- Potential privacy implications with request tracking

### Rate Limiting Header Strategy
**Decision**: Include comprehensive rate limiting information in response headers
**Rationale**:
- Enables intelligent client retry behavior
- Transparent rate limiting implementation
- Helps developers optimize their API usage
- Industry standard approach

**Trade-offs**:
- Additional response header overhead
- Reveals rate limiting implementation details
- Complex header calculation for different rate limit types

---

**Next**: Review [examples.md](./examples.md) for comprehensive API usage examples and integration patterns.