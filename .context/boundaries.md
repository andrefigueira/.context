# Boundaries & Ownership

This file defines what AI should and should not modify, and clarifies ownership of different parts of the codebase.

## Do Not Modify

These files/areas require human review and should not be changed by AI without explicit request:

### Critical Infrastructure
| Path | Reason |
|------|--------|
| `*.env*` | Contains secrets and environment-specific config |
| `docker-compose.prod.yml` | Production infrastructure |
| `k8s/` | Kubernetes manifests affect production |
| `terraform/` | Infrastructure as code, affects cloud resources |
| `.github/workflows/` | CI/CD pipelines, security implications |

### Security-Sensitive
| Path | Reason |
|------|--------|
| `auth/` | Authentication logic requires security review |
| `**/middleware/auth*` | Token validation, session handling |
| `**/crypto*` | Encryption, hashing implementations |
| `**/secrets*` | Secret management code |

### Generated Files
| Path | Reason |
|------|--------|
| `*.gen.go` | Generated code, modify source instead |
| `*.generated.ts` | Generated from schema |
| `dist/`, `build/` | Build output |
| `node_modules/`, `vendor/` | Dependencies |

### Database Migrations
| Path | Reason |
|------|--------|
| `migrations/*.sql` | Existing migrations are immutable |

**Exception**: Creating NEW migration files is allowed and encouraged.

## Modification Guidelines

### Allowed With Caution
| Path | Guideline |
|------|-----------|
| `api/` handlers | Follow existing patterns exactly |
| `services/` | Add tests for any new logic |
| `repositories/` | Don't change query patterns without reason |
| `models/` | Ensure database schema compatibility |

### Freely Modifiable
| Path | Notes |
|------|-------|
| `tests/` | Add and modify tests freely |
| `docs/` | Documentation improvements welcome |
| `.context/` | Keep substrate updated |
| `scripts/` | Development tooling |

## Ownership Map

Different areas have different owners who should review changes:

| Area | Owner | Review Required For |
|------|-------|---------------------|
| Authentication | Security Team | Any auth logic changes |
| API Contracts | API Team | Endpoint changes, breaking changes |
| Database Schema | Backend Lead | Schema migrations |
| Infrastructure | DevOps | Docker, K8s, CI/CD |
| Frontend | Frontend Team | UI components, state management |

## Cross-Cutting Concerns

When modifying these, consider impact across the codebase:

| Concern | Files Affected | Coordination Needed |
|---------|---------------|---------------------|
| Error handling | All handlers, services | Maintain consistency |
| Logging | All files | Follow log level conventions |
| Metrics | Services, handlers | Use established metric names |
| Configuration | Config loaders, all consumers | Don't break existing config |

## Breaking Changes

Before making breaking changes:

1. **API Changes**: Check for consumers, version appropriately
2. **Database Changes**: Ensure backward-compatible migration
3. **Config Changes**: Provide migration path or defaults
4. **Interface Changes**: Update all implementations

## When Unsure

If uncertain whether a change is allowed:
1. Ask before modifying security-sensitive code
2. Create smallest possible change
3. Add comprehensive tests
4. Document the change rationale
5. Flag for human review

---

**Note**: This file protects the codebase from unintended modifications. When in doubt, err on the side of caution and ask for clarification.
