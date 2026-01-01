# Feature Flags

Strategy for feature flags and gradual rollouts.

## Why Feature Flags

- **Decouple deployment from release** - Deploy code without enabling
- **Gradual rollout** - Release to % of users
- **Quick rollback** - Disable without deploy
- **A/B testing** - Compare variants
- **Kill switch** - Disable problematic features

## Flag Types

### Release Flags
Temporary flags for releasing new features.
- Remove after feature is fully released
- Short lifespan (days to weeks)

### Experiment Flags
For A/B tests and experiments.
- Tied to specific experiments
- Remove after experiment concludes

### Ops Flags
For operational control.
- Enable/disable features at runtime
- May be long-lived

### Permission Flags
Control access to features.
- Based on user attributes
- Often permanent

## Naming Convention

```
{type}_{feature}_{description}

Examples:
release_new_checkout
experiment_pricing_v2
ops_rate_limiting
permission_admin_dashboard
```

## Flag Catalog

| Flag | Type | Description | Default | Owner |
|------|------|-------------|---------|-------|
| `release_new_auth` | Release | New authentication flow | false | @auth-team |
| `ops_maintenance_mode` | Ops | Enable maintenance page | false | @platform |
| [Add your flags here] | | | | |

## Implementation Guidelines

### Check Placement
- Check at decision points, not everywhere
- Keep checks at edges (handlers, not deep in services)
- One check per feature per request path

### Default Values
- Default to **off** for new features
- Default to **on** for kill switches
- Document defaults clearly

### Context for Evaluation
- User ID (for user-based rollout)
- User attributes (plan, role, etc.)
- Request attributes (country, etc.)
- Environment

## Rollout Strategy

### Percentage Rollout
1. Internal users (0%)
2. Beta users (5%)
3. Gradual increase (25%, 50%, 75%)
4. Full rollout (100%)
5. Remove flag

### User Segment Rollout
1. Internal only
2. Beta/early adopters
3. Free tier
4. Paid tier
5. All users

## Lifecycle Management

### Adding Flags
1. Add to this catalog
2. Implement check in code
3. Set initial value (off)
4. Document in PR

### Removing Flags
1. Verify feature stable
2. Remove checks from code
3. Remove from catalog
4. Remove from flag system

### Flag Hygiene
- Review flags monthly
- Remove stale release flags
- Document why long-lived flags exist
- Set expiry dates where possible

## Testing with Flags

- Test both flag states
- CI tests run with flag on AND off
- Integration tests for flag transitions
- Never leave broken code behind "off" flag

## Monitoring

- Alert on flag evaluation failures
- Track flag evaluation counts
- Monitor feature usage by flag state
- Dashboard for flag states

## Anti-Patterns

- **Flag in a flag** - Don't nest flag checks
- **Permanent release flags** - Remove when released
- **Too many flags** - Limit active flags
- **Untested states** - Test all combinations
- **Missing defaults** - Always have fallback
