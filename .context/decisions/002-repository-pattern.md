# ADR-002: Implement Repository Pattern

## Status
Accepted

## Context
We need a consistent approach to data access that:
- Enables unit testing without database dependencies
- Provides a clean abstraction over database operations
- Allows future database changes with minimal code impact
- Keeps business logic free of data access concerns

Alternative approaches considered:
- Direct database access in services
- Active Record pattern
- Query builder abstraction only

## Decision
Implement the Repository pattern with:
- Interface definitions for each aggregate root
- Concrete implementations for PostgreSQL
- Dependency injection of repositories into services

Repository responsibilities:
- CRUD operations for aggregates
- Query methods returning domain objects
- Transaction management at repository boundary

Service responsibilities:
- Business logic and validation
- Orchestrating multiple repository calls
- Publishing domain events

## Consequences

### Positive
- Services are testable with mock repositories
- Database implementation details hidden from business logic
- Easy to add caching layer between service and repository
- Clear separation of concerns
- Potential to swap database implementations

### Negative
- Additional abstraction layer increases code volume
- Risk of leaky abstractions for complex queries
- Must maintain interface and implementation in sync
- May lead to anemic domain models if not careful

### Neutral
- Team must learn and follow the pattern consistently
- Complex queries may require custom repository methods
- Aggregate boundaries need careful design

## Implementation Notes
- One repository per aggregate root (User, not UserProfile separately)
- Use constructor injection for repositories in services
- Keep repository methods focused (no god methods)
- For complex read queries, consider separate read models/CQRS
