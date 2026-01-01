# Performance Standards

Performance budgets, guidelines, and optimization priorities.

## Performance Budgets

### API Response Times

| Endpoint Type | P50 Target | P95 Target | P99 Max |
|---------------|------------|------------|---------|
| Health check | < 10ms | < 50ms | < 100ms |
| Read (single) | < 50ms | < 100ms | < 200ms |
| Read (list) | < 100ms | < 200ms | < 500ms |
| Write (simple) | < 100ms | < 200ms | < 500ms |
| Write (complex) | < 200ms | < 500ms | < 1s |
| Batch operation | < 1s | < 2s | < 5s |

### Frontend Metrics (Core Web Vitals)

| Metric | Good | Needs Work | Poor |
|--------|------|------------|------|
| LCP (Largest Contentful Paint) | < 2.5s | < 4s | > 4s |
| FID (First Input Delay) | < 100ms | < 300ms | > 300ms |
| CLS (Cumulative Layout Shift) | < 0.1 | < 0.25 | > 0.25 |
| INP (Interaction to Next Paint) | < 200ms | < 500ms | > 500ms |

### Resource Limits

| Resource | Limit |
|----------|-------|
| API response size | < 1MB |
| Single query time | < 100ms |
| Memory per request | < 50MB |
| File upload | < 10MB (configurable) |

## Optimization Priorities

### Priority 1: Database
Most performance issues are database-related.

- **Use indexes** for all query WHERE/ORDER clauses
- **Avoid N+1 queries** - use joins or batch loading
- **Paginate** all list endpoints
- **Use connection pooling** appropriately
- **Query only needed columns** for large tables

### Priority 2: Caching
Cache aggressively at appropriate layers.

- **Response caching** for static/semi-static data
- **Query caching** for expensive computations
- **Session caching** for user data
- **CDN** for static assets

### Priority 3: Async Processing
Move slow operations out of request path.

- **Queue** email sending
- **Queue** notifications
- **Queue** report generation
- **Batch** bulk operations

## Red Flags

Warning signs that indicate performance problems:

- [ ] Queries without LIMIT
- [ ] SELECT * on large tables
- [ ] Loops with database calls inside
- [ ] Synchronous external API calls
- [ ] Loading entire files into memory
- [ ] Regex on user input without limits
- [ ] Unbounded list endpoints

## Measurement

### What to Measure
- Request latency (P50, P95, P99)
- Throughput (requests/second)
- Error rate
- Database query time
- External service latency
- Memory usage
- CPU usage

### How to Measure
- Application metrics (Prometheus, StatsD)
- APM tools (DataDog, New Relic)
- Database query logs
- Load testing (k6, Locust)

## Load Testing Guidelines

### Before Launch
- Test at 2x expected load
- Identify breaking point
- Document capacity limits

### Regularly
- Run load tests on staging
- Compare against baseline
- Catch regressions early

## Performance Review Checklist

When reviewing code for performance:

- [ ] Database queries have appropriate indexes?
- [ ] No N+1 queries?
- [ ] Lists are paginated?
- [ ] Heavy operations are async?
- [ ] Caching used where appropriate?
- [ ] Response sizes reasonable?
- [ ] No unbounded operations?
