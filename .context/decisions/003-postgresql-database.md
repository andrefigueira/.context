# ADR-003: Use PostgreSQL as Primary Database

## Status
Accepted

## Context
We need a primary database that supports:
- Relational data with complex queries
- ACID transactions for data integrity
- JSON storage for flexible schema portions
- Good performance at expected scale (millions of rows)
- Strong ecosystem and tooling

Alternative approaches considered:
- MySQL/MariaDB
- MongoDB (document store)
- CockroachDB (distributed SQL)

## Decision
Use PostgreSQL 14+ as the primary database with:
- Structured relational tables for core entities
- JSONB columns for flexible metadata/preferences
- UUID primary keys for distribution-readiness
- Connection pooling via PgBouncer

## Consequences

### Positive
- Mature, battle-tested database with excellent reliability
- Rich feature set (JSONB, full-text search, CTEs, window functions)
- Strong ACID guarantees for data integrity
- Excellent tooling ecosystem (pg_dump, pgAdmin, etc.)
- Good performance with proper indexing
- Large community and extensive documentation

### Negative
- Vertical scaling has limits (though sufficient for our scale)
- Requires more operational expertise than managed NoSQL
- Schema migrations require careful planning
- Connection limits need management at scale

### Neutral
- Need to manage connection pooling
- Must plan backup and recovery procedures
- Index maintenance requires attention
- Monitoring setup required

## Implementation Notes
- Use UUIDs (`gen_random_uuid()`) for primary keys
- Implement soft deletes with `deleted_at` timestamp
- Use JSONB for user preferences and flexible metadata
- Partition large tables (audit_logs) by time
- Set up read replicas when read traffic justifies
