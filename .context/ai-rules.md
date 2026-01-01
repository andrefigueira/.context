# AI Rules & Constraints

This file defines hard rules and constraints for AI code generation. These are non-negotiable standards that must be followed in all generated code.

## Code Style Rules

### Naming Conventions
```yaml
variables: camelCase
functions: camelCase
classes: PascalCase
constants: SCREAMING_SNAKE_CASE
files: kebab-case
database_tables: snake_case
database_columns: snake_case
api_endpoints: kebab-case
```

### Language-Specific Rules

**TypeScript:**
- Never use `any` - use `unknown` and narrow, or define proper types
- Always use strict mode
- Prefer `interface` over `type` for object shapes
- Use `const` by default, `let` when reassignment needed, never `var`

**Go:**
- Always handle errors explicitly, never ignore with `_`
- Use meaningful variable names, not single letters (except loop counters)
- Return early to avoid deep nesting

**Python:**
- Use type hints on all function signatures
- Use dataclasses or Pydantic for data structures
- Never use mutable default arguments

## Architectural Rules

### Always
- Use dependency injection for services
- Validate all external input at system boundaries
- Log errors with context (user ID, request ID, operation)
- Use transactions for multi-step database operations
- Return specific error types, not generic errors

### Never
- Put business logic in controllers/handlers
- Store secrets in code or config files
- Use raw SQL without parameterized queries
- Catch exceptions and silently ignore them
- Use global mutable state

## Security Rules

### Authentication
- Never store plaintext passwords
- Always use constant-time comparison for secrets
- Invalidate sessions on password change
- Implement rate limiting on auth endpoints

### Data Handling
- Sanitize all user input before display
- Use parameterized queries exclusively
- Never log sensitive data (passwords, tokens, PII)
- Encrypt sensitive data at rest

### API Security
- Validate Content-Type headers
- Implement CORS properly (not `*` in production)
- Use HTTPS only
- Set security headers (CSP, X-Frame-Options, etc.)

## Testing Rules

- Unit tests for all business logic
- Integration tests for API endpoints
- No test should depend on another test's state
- Mock external services, don't call them
- Test error paths, not just happy paths

## Documentation Rules

- Public functions must have doc comments
- Complex logic requires inline comments explaining "why"
- API endpoints must have request/response examples
- Breaking changes must be documented

## Dependency Rules

- Prefer standard library over external packages
- Evaluate security and maintenance status before adding dependencies
- Pin dependency versions exactly
- Update dependencies monthly with testing

---

**Note**: These rules override any conflicting patterns. When in doubt, follow these constraints.
