# Error Codes Catalog

Standardized error codes and messages for consistent error handling across the application.

## Error Response Format

All errors follow this structure:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {}
  },
  "requestId": "uuid"
}
```

## Error Code Prefixes

| Prefix | Domain | HTTP Status Range |
|--------|--------|-------------------|
| `AUTH_` | Authentication | 401, 403 |
| `VAL_` | Validation | 400 |
| `USER_` | User management | 400, 404, 409 |
| `RES_` | Resource/general | 404, 409, 410 |
| `RATE_` | Rate limiting | 429 |
| `SYS_` | System/internal | 500, 503 |

## Authentication Errors (AUTH_)

| Code | HTTP | Message | When |
|------|------|---------|------|
| `AUTH_REQUIRED` | 401 | Authentication required | No token provided |
| `AUTH_INVALID_TOKEN` | 401 | Invalid or expired token | Token verification failed |
| `AUTH_EXPIRED_TOKEN` | 401 | Token has expired | Token past expiry |
| `AUTH_INVALID_CREDENTIALS` | 401 | Invalid email or password | Login failed |
| `AUTH_FORBIDDEN` | 403 | Access denied | Insufficient permissions |
| `AUTH_ACCOUNT_LOCKED` | 403 | Account is locked | Too many failed attempts |
| `AUTH_EMAIL_NOT_VERIFIED` | 403 | Email not verified | Requires email verification |

## Validation Errors (VAL_)

| Code | HTTP | Message | When |
|------|------|---------|------|
| `VAL_INVALID_INPUT` | 400 | Validation failed | General validation failure |
| `VAL_MISSING_FIELD` | 400 | Required field missing: {field} | Required field not provided |
| `VAL_INVALID_FORMAT` | 400 | Invalid format for: {field} | Wrong format (email, phone, etc.) |
| `VAL_OUT_OF_RANGE` | 400 | Value out of range for: {field} | Number/length outside bounds |
| `VAL_INVALID_ENUM` | 400 | Invalid value for: {field} | Not in allowed values |

## User Errors (USER_)

| Code | HTTP | Message | When |
|------|------|---------|------|
| `USER_NOT_FOUND` | 404 | User not found | User ID doesn't exist |
| `USER_EMAIL_EXISTS` | 409 | Email already registered | Duplicate email on registration |
| `USER_USERNAME_EXISTS` | 409 | Username already taken | Duplicate username |
| `USER_SUSPENDED` | 403 | Account is suspended | User action on suspended account |

## Resource Errors (RES_)

| Code | HTTP | Message | When |
|------|------|---------|------|
| `RES_NOT_FOUND` | 404 | {Resource} not found | Generic not found |
| `RES_ALREADY_EXISTS` | 409 | {Resource} already exists | Duplicate creation |
| `RES_GONE` | 410 | {Resource} has been deleted | Accessing deleted resource |
| `RES_LOCKED` | 423 | {Resource} is locked | Resource under edit lock |
| `RES_CONFLICT` | 409 | Conflict with current state | Optimistic locking failure |

## Rate Limiting Errors (RATE_)

| Code | HTTP | Message | When |
|------|------|---------|------|
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests | Rate limit hit |
| `RATE_QUOTA_EXCEEDED` | 429 | Usage quota exceeded | API quota exhausted |

## System Errors (SYS_)

| Code | HTTP | Message | When |
|------|------|---------|------|
| `SYS_INTERNAL_ERROR` | 500 | An unexpected error occurred | Unhandled exception |
| `SYS_SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable | Dependency down |
| `SYS_MAINTENANCE` | 503 | System under maintenance | Planned downtime |
| `SYS_TIMEOUT` | 504 | Request timed out | Operation timeout |

## Error Details

Some errors include additional details:

### Validation Error Details
```json
{
  "code": "VAL_INVALID_INPUT",
  "message": "Validation failed",
  "details": {
    "fields": [
      { "field": "email", "message": "Invalid email format" },
      { "field": "password", "message": "Must be at least 12 characters" }
    ]
  }
}
```

### Rate Limit Details
```json
{
  "code": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests",
  "details": {
    "retryAfter": 60,
    "limit": 100,
    "remaining": 0
  }
}
```

## Adding New Error Codes

When adding a new error code:

1. Use appropriate prefix
2. Add to this catalog
3. Include HTTP status
4. Write clear, actionable message
5. Document when it occurs
6. Add to error constants in code
