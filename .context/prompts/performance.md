# Prompt: Performance Optimization

## Context Files to Include
```
.context/performance.md
.context/database/schema.md
.context/architecture/patterns.md
```

## Prompt

```
Analyze and optimize performance for:

**Target**: [Component/endpoint/query to optimize]

**Current Performance**:
- Response time: [current]
- Throughput: [current]
- Resource usage: [current]

**Target Performance**:
- Response time: [goal]
- Throughput: [goal]

**Observed Issues**:
[Describe slow behavior, when it occurs]

Analyze for:

1. **Database**
   - Missing indexes
   - N+1 queries
   - Inefficient queries
   - Connection pooling

2. **Caching**
   - What can be cached
   - Cache invalidation strategy
   - Cache hit rates

3. **Code**
   - Unnecessary computation
   - Memory allocation
   - Blocking operations
   - Algorithm complexity

4. **Architecture**
   - Async opportunities
   - Batching potential
   - Parallelization

Provide:
- Root cause analysis
- Specific optimizations (prioritized by impact)
- Expected improvement for each
- Trade-offs to consider
```
