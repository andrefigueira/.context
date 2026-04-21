# Rule: Auth

Rules for authentication, authorization, sessions, and security middleware.

## Applies to
`src/auth/**`, `src/middleware/auth*`, `src/security/**`

## Hard rules
1. Never store plaintext passwords. Use the project's hashing helper
2. Use constant-time comparison for any secret, token, or hash
3. Invalidate all sessions on password change
4. Rate-limit every auth endpoint (login, register, password reset, refresh)
5. Never log passwords, tokens, session IDs, or PII
6. Tokens are opaque to clients. Do not leak token payload contents in responses
7. CSRF protection on all state-changing endpoints that accept cookies
8. Refresh tokens rotate on use; old refresh tokens are invalidated immediately

## Patterns

Protected handler gate:

```ts
export const handler = withAuth(async (req, { user }) => {
  // user is guaranteed present and valid here
});
```

## Load on demand
- Auth flow overview → `.context/auth/overview.md`
- Security model and threat mitigations → `.context/auth/security.md`
- Framework integration patterns → `.context/auth/integration.md`
- Decision record for auth method → `.context/decisions/001-jwt-authentication.md`
