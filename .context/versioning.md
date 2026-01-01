# API Versioning Strategy

How we version APIs and handle breaking changes.

## Versioning Approach

We use **URL path versioning**:

```
/api/v1/users
/api/v2/users
```

### Why URL Versioning
- Explicit and visible
- Easy to route
- Cache-friendly
- Simple for clients

### Alternatives Considered
- Header versioning: Hidden, harder to test
- Query parameter: Not RESTful
- Content negotiation: Complex

## Version Lifecycle

```
Active → Deprecated → Sunset → Removed
```

| Stage | Description | Duration |
|-------|-------------|----------|
| **Active** | Current recommended version | Until next version |
| **Deprecated** | Still works, not recommended | 6 months minimum |
| **Sunset** | Final warning, reduced support | 3 months |
| **Removed** | Returns 410 Gone | Permanent |

## What Constitutes Breaking Change

### Breaking (requires new version)
- Removing an endpoint
- Removing a field from response
- Changing field type
- Changing field meaning
- Requiring new input field
- Changing authentication method
- Changing error codes

### Non-Breaking (same version)
- Adding optional field to request
- Adding field to response
- Adding new endpoint
- Adding new optional parameter
- Performance improvements
- Bug fixes

## Deprecation Process

### 1. Announce
- Update API documentation
- Add deprecation notice to responses
- Email API consumers
- Blog post for public APIs

### 2. Add Headers
```
Deprecation: true
Sunset: Sat, 1 Jun 2025 00:00:00 GMT
Link: <https://docs.example.com/migration>; rel="deprecation"
```

### 3. Monitor
- Track usage of deprecated endpoints
- Reach out to heavy users
- Provide migration support

### 4. Remove
- Return 410 Gone after sunset
- Include migration information in response
- Monitor for unexpected traffic

## Migration Support

When deprecating, provide:

1. **Migration guide** - Step-by-step instructions
2. **Changelog** - What changed between versions
3. **Comparison** - Old vs new examples
4. **Timeline** - Key dates
5. **Support** - How to get help

## Version Documentation

Each version maintains its own documentation:

```
/docs/api/v1/
/docs/api/v2/
```

Documentation includes:
- All endpoints for that version
- Version-specific behavior
- Migration notes from previous version

## Client Guidelines

Communicate to API consumers:

1. Always specify version explicitly
2. Don't use "latest" or unversioned
3. Subscribe to deprecation notices
4. Plan migration time in roadmap
5. Test against new versions early

## Internal Versioning

For internal APIs:
- Less formal process
- Shorter deprecation windows
- Coordinate with consuming teams
- Can use feature flags for transitions

## Current Versions

| Version | Status | Sunset Date | Notes |
|---------|--------|-------------|-------|
| v1 | Deprecated | 2025-06-01 | Migrate to v2 |
| v2 | Active | - | Current version |

## Adding New Version

1. Create new version in code
2. Copy documentation, update for changes
3. Write migration guide
4. Update API clients/SDKs
5. Announce new version
6. Begin deprecation of old version
