# Approved Dependencies

Approved packages and libraries with rationale. Use these before adding new dependencies.

## Principles

1. **Standard library first** - Use built-in solutions when sufficient
2. **Proven over novel** - Prefer battle-tested packages
3. **Minimal dependencies** - Each addition is maintenance burden
4. **Security matters** - Check for known vulnerabilities
5. **Active maintenance** - Avoid abandoned packages

## Evaluation Criteria

Before adding a new dependency, consider:

- [ ] Is this functionality in standard library?
- [ ] Can we implement it in < 100 lines?
- [ ] Is package actively maintained?
- [ ] Are there known security issues?
- [ ] What's the dependency tree size?
- [ ] Is it widely used (community trust)?
- [ ] License compatible?

## Approved Packages by Category

### HTTP Framework
| Package | Use For | Why This One |
|---------|---------|--------------|
| Express (Node) | HTTP server | Standard, huge ecosystem |
| Fastify (Node) | High-perf HTTP | When Express too slow |
| Chi (Go) | HTTP router | Lightweight, idiomatic |
| FastAPI (Python) | HTTP API | Modern, type hints, fast |

### Database
| Package | Use For | Why This One |
|---------|---------|--------------|
| Prisma (Node) | ORM | Type-safe, great DX |
| pg (Node) | Raw PostgreSQL | When ORM overkill |
| sqlx (Go) | SQL with types | Compile-time checked |
| SQLAlchemy (Python) | ORM | Standard, flexible |

### Validation
| Package | Use For | Why This One |
|---------|---------|--------------|
| Zod (Node) | Schema validation | Type inference, composable |
| Pydantic (Python) | Data validation | Standard for FastAPI |
| validator (Go) | Struct validation | Widely used, extensible |

### Authentication
| Package | Use For | Why This One |
|---------|---------|--------------|
| jose (Node) | JWT | Spec-compliant, maintained |
| golang-jwt (Go) | JWT | Standard Go choice |
| PyJWT (Python) | JWT | Simple, reliable |
| argon2 | Password hashing | Current best algorithm |

### Testing
| Package | Use For | Why This One |
|---------|---------|--------------|
| Jest (Node) | Testing | Standard, great DX |
| Vitest (Node) | Testing | Faster Jest alternative |
| testify (Go) | Assertions | Readable assertions |
| pytest (Python) | Testing | Standard, powerful |

### Logging
| Package | Use For | Why This One |
|---------|---------|--------------|
| pino (Node) | Logging | Fast, JSON output |
| zerolog (Go) | Logging | Zero allocation |
| structlog (Python) | Logging | Structured logging |

### HTTP Client
| Package | Use For | Why This One |
|---------|---------|--------------|
| axios (Node) | HTTP client | When fetch insufficient |
| Standard library | HTTP client | Usually enough |

## Adding New Dependencies

1. Check if approved alternative exists
2. Evaluate against criteria above
3. Document rationale
4. Add to this file
5. Review with team

## Dependency Updates

- **Weekly**: Check for security updates
- **Monthly**: Review non-security updates
- **Quarterly**: Audit unused dependencies

## Banned Packages

Packages we explicitly avoid:

| Package | Reason | Alternative |
|---------|--------|-------------|
| moment.js | Bloated, deprecated | date-fns, dayjs |
| request | Deprecated | axios, fetch |
| lodash (full) | Usually overkill | lodash-es (specific imports) |

## Security

- Run `npm audit` / `pip check` / etc. in CI
- Address critical vulnerabilities immediately
- High vulnerabilities within 1 week
- Review dependency licenses
