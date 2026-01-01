# Environment Variables

Documentation of all environment variables used in the application. This file documents the variables, not their values.

## Required Variables

### Application
| Variable | Description | Example Format |
|----------|-------------|----------------|
| `NODE_ENV` | Environment mode | `development`, `staging`, `production` |
| `PORT` | Server port | `3000` |
| `APP_URL` | Public application URL | `https://api.example.com` |

### Database
| Variable | Description | Example Format |
|----------|-------------|----------------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@host:5432/db` |
| `DATABASE_POOL_MIN` | Minimum pool connections | `2` |
| `DATABASE_POOL_MAX` | Maximum pool connections | `10` |

### Authentication
| Variable | Description | Example Format |
|----------|-------------|----------------|
| `JWT_SECRET` | Secret for signing JWTs | 32+ character random string |
| `JWT_ACCESS_EXPIRY` | Access token lifetime | `15m` |
| `JWT_REFRESH_EXPIRY` | Refresh token lifetime | `7d` |

### External Services
| Variable | Description | Example Format |
|----------|-------------|----------------|
| `REDIS_URL` | Redis connection string | `redis://localhost:6379` |
| `SMTP_HOST` | Email server host | `smtp.sendgrid.net` |
| `SMTP_PORT` | Email server port | `587` |
| `SMTP_USER` | Email username | API key or username |
| `SMTP_PASS` | Email password | API key or password |

## Optional Variables

### Observability
| Variable | Description | Default |
|----------|-------------|---------|
| `LOG_LEVEL` | Logging verbosity | `info` |
| `LOG_FORMAT` | Log output format | `json` |
| `SENTRY_DSN` | Error tracking DSN | none |
| `METRICS_PORT` | Prometheus metrics port | `9090` |

### Feature Flags
| Variable | Description | Default |
|----------|-------------|---------|
| `FEATURE_NEW_AUTH` | Enable new auth flow | `false` |
| `FEATURE_DARK_MODE` | Enable dark mode | `false` |

### Rate Limiting
| Variable | Description | Default |
|----------|-------------|---------|
| `RATE_LIMIT_WINDOW` | Rate limit window (seconds) | `60` |
| `RATE_LIMIT_MAX` | Max requests per window | `100` |

## Environment-Specific Defaults

### Development
- Relaxed rate limits
- Verbose logging
- Debug mode enabled
- Local services

### Staging
- Production-like config
- Debug logging
- Test external services

### Production
- Strict rate limits
- Error-only logging
- All security features enabled
- Production external services

## Adding New Variables

When adding a new environment variable:

1. Add to this documentation
2. Add to `.env.example` with placeholder value
3. Add validation in config loader
4. Provide sensible default if optional
5. Update deployment documentation

## Security Notes

- Never commit actual values to version control
- Use secrets manager in production
- Rotate secrets regularly
- Audit access to production secrets
- Different secrets per environment
