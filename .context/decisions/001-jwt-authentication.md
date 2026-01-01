# ADR-001: Use JWT for Authentication

## Status
Accepted

## Context
We need to implement authentication for our API that supports:
- Stateless operation for horizontal scaling
- Mobile and web client compatibility
- Reasonable security for user sessions
- Ability to embed user claims for authorization

Alternative approaches considered:
- Session-based authentication with server-side storage
- OAuth2 with third-party provider only
- API keys for all access

## Decision
Implement JWT-based authentication with short-lived access tokens (15 minutes) and longer-lived refresh tokens (7 days) with rotation on use.

Token structure:
- Access token: Contains user ID, email, roles, permissions
- Refresh token: Opaque token stored in database, rotated on each use

## Consequences

### Positive
- Stateless authentication enables easy horizontal scaling
- No server-side session storage required for access tokens
- Claims embedded in token reduce database lookups for authorization
- Refresh token rotation limits impact of token theft
- Standard format with excellent library support

### Negative
- Token revocation requires blacklist infrastructure
- Larger request overhead due to token size in headers
- More complex client-side token management
- Cannot instantly invalidate access tokens (must wait for expiry)
- Updating user permissions requires token refresh

### Neutral
- Need to implement token refresh logic in all clients
- Must secure JWT signing key as critical infrastructure
- Access token lifetime is a security/UX tradeoff

## Implementation Notes
- Use HS256 for signing (symmetric, simpler key management)
- Store refresh tokens in `auth.refresh_tokens` table
- Implement refresh token family tracking for reuse detection
- Set up token blacklist in Redis for immediate revocation when needed
