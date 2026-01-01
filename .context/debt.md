# Technical Debt Registry

This file documents known technical debt. It helps AI understand what shortcuts exist, why they were taken, and what the proper solution would be.

## How to Use This File

When generating code:
- Be aware of existing debt to avoid compounding it
- Don't add new instances of documented debt patterns
- If fixing debt is in scope, reference the "Proper Solution"

## Active Debt

### HIGH Priority

#### DEBT-001: Missing Input Validation on Internal APIs
- **Location**: `internal/api/handlers/`
- **Description**: Internal service-to-service APIs skip input validation assuming trusted callers
- **Why it exists**: Speed of initial development, "internal only" assumption
- **Risk**: Service compromise could exploit unvalidated endpoints
- **Proper solution**: Add validation middleware to all endpoints regardless of caller
- **Effort**: Medium (2-3 days)
- **Created**: 2024-01-15

#### DEBT-002: Synchronous Email Sending
- **Location**: `services/user/registration.go`
- **Description**: Welcome emails sent synchronously during registration
- **Why it exists**: Quick implementation, email service was fast initially
- **Risk**: Slow email service blocks registration, poor UX
- **Proper solution**: Queue emails for async processing via message queue
- **Effort**: Medium (needs queue infrastructure)
- **Created**: 2024-01-20

### MEDIUM Priority

#### DEBT-003: Hardcoded Pagination Limits
- **Location**: Various repository files
- **Description**: `LIMIT 100` hardcoded in queries instead of configurable
- **Why it exists**: Quick fix during development
- **Risk**: Inflexible, can't adjust without code change
- **Proper solution**: Move to configuration with sensible defaults
- **Effort**: Low (1 day)
- **Created**: 2024-02-01

#### DEBT-004: No Database Connection Retry
- **Location**: `internal/database/connection.go`
- **Description**: Application fails immediately if database unavailable at startup
- **Why it exists**: Simple implementation
- **Risk**: Deployment ordering issues, transient failures cause crashes
- **Proper solution**: Exponential backoff retry with circuit breaker
- **Effort**: Low (half day)
- **Created**: 2024-02-01

### LOW Priority

#### DEBT-005: Inconsistent Error Messages
- **Location**: Various handlers
- **Description**: Some errors return detailed messages, others are generic
- **Why it exists**: Different developers, no enforced standard
- **Risk**: Inconsistent API experience, potential info leakage
- **Proper solution**: Standardize error response format, review all handlers
- **Effort**: Medium (needs audit and refactor)
- **Created**: 2024-02-15

## Resolved Debt

| ID | Description | Resolution | Date |
|----|-------------|------------|------|
| DEBT-000 | No authentication | Implemented JWT auth (ADR-001) | 2024-01-15 |

## Adding New Debt

When intentionally taking on debt:
1. Add entry here with unique ID
2. Document the "why" clearly
3. Describe proper solution
4. Estimate effort
5. Assign priority (HIGH/MEDIUM/LOW)
6. Reference in code comment: `// DEBT-XXX: [brief description]`

## Debt Review Schedule

- **Weekly**: Review HIGH priority items
- **Monthly**: Review MEDIUM priority items
- **Quarterly**: Review LOW priority, archive stale items
