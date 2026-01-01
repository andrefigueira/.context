# Monitoring & Observability

Standards for logging, metrics, and observability.

## Three Pillars

1. **Logs** - What happened
2. **Metrics** - How much/how fast
3. **Traces** - Request flow across services

## Logging Standards

### Log Levels

| Level | Use For | Example |
|-------|---------|---------|
| `error` | Errors requiring attention | Database connection failed |
| `warn` | Concerning but handled | Rate limit approaching |
| `info` | Notable events | User registered |
| `debug` | Debugging details | Query executed |

### What to Log

**Always Log**
- Request received (info)
- Request completed (info)
- Errors with context (error)
- Authentication failures (warn)
- Authorization failures (warn)
- External service calls (debug)

**Never Log**
- Passwords or secrets
- Full credit card numbers
- Personal health information
- Session tokens
- API keys

### Log Format

Use structured JSON logging:

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "info",
  "message": "User registered",
  "requestId": "abc-123",
  "userId": "user_456",
  "email": "u***@example.com",
  "duration_ms": 45
}
```

### Context to Include

- `requestId` - Correlate logs for a request
- `userId` - Who triggered the action
- `action` - What operation
- `duration_ms` - How long it took
- `error` - Error details (for errors)

## Metrics Standards

### Naming Convention

```
{namespace}_{subsystem}_{name}_{unit}

Examples:
api_http_requests_total
api_http_request_duration_seconds
db_query_duration_seconds
cache_hits_total
```

### Key Metrics to Track

**RED Metrics (for services)**
- **R**ate - Requests per second
- **E**rrors - Error rate
- **D**uration - Latency

**USE Metrics (for resources)**
- **U**tilization - % capacity used
- **S**aturation - Queue depth
- **E**rrors - Error count

### Standard Metrics

| Metric | Type | Labels |
|--------|------|--------|
| `http_requests_total` | Counter | method, path, status |
| `http_request_duration_seconds` | Histogram | method, path |
| `db_query_duration_seconds` | Histogram | query_type |
| `cache_hits_total` | Counter | cache_name |
| `cache_misses_total` | Counter | cache_name |
| `external_api_requests_total` | Counter | service, status |
| `external_api_duration_seconds` | Histogram | service |

## Alerting

### Alert Severity

| Severity | Response Time | Examples |
|----------|---------------|----------|
| Critical | Immediate | Service down, data loss |
| High | < 1 hour | Error rate > 5% |
| Medium | < 4 hours | Latency degradation |
| Low | Next business day | Disk 70% full |

### Alert Best Practices

- Alert on symptoms, not causes
- Include runbook links
- Avoid alert fatigue
- Test alerts regularly
- Page only for actionable issues

## Dashboards

### Standard Dashboards

1. **Overview**
   - Request rate
   - Error rate
   - Latency (P50, P95, P99)
   - Active users

2. **API Performance**
   - Requests by endpoint
   - Latency by endpoint
   - Errors by endpoint

3. **Database**
   - Query latency
   - Connection pool usage
   - Slow queries

4. **Infrastructure**
   - CPU/Memory usage
   - Disk usage
   - Network I/O

## Tracing

### When to Trace

- Cross-service requests
- Database queries
- External API calls
- Cache operations
- Queue operations

### Span Naming

```
{component}.{operation}

Examples:
http.request
db.query
cache.get
external.payment_api
queue.publish
```

## Health Checks

### Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/health` | Basic liveness check |
| `/ready` | Full readiness check |

### Readiness Checks

- Database connection
- Cache connection
- External service availability
- Sufficient resources
