# Domain Events

Catalog of domain events and their usage patterns.

## What Are Domain Events

Events represent something significant that happened in the system. They enable loose coupling between components.

## Event Naming

Format: `{Entity}{Past Tense Verb}`

Examples:
- `UserRegistered`
- `OrderPlaced`
- `PaymentCompleted`
- `EmailVerified`

## Event Structure

All events include base fields:

| Field | Type | Description |
|-------|------|-------------|
| `eventId` | UUID | Unique event identifier |
| `eventType` | string | Event name |
| `timestamp` | ISO8601 | When event occurred |
| `version` | integer | Event schema version |
| `aggregateId` | UUID | ID of entity that changed |
| `payload` | object | Event-specific data |
| `metadata` | object | Context (userId, requestId) |

## Event Catalog

### User Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `UserRegistered` | New user signup | email, name, userId |
| `UserEmailVerified` | Email confirmation | userId |
| `UserPasswordChanged` | Password update | userId |
| `UserProfileUpdated` | Profile edit | userId, changedFields |
| `UserDeleted` | Account deletion | userId |

### Authentication Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `UserLoggedIn` | Successful login | userId, method |
| `UserLoggedOut` | Logout | userId |
| `LoginFailed` | Failed login attempt | email, reason |
| `TokenRefreshed` | Token refresh | userId |

### [Add your domain events here]

## Event Handlers

Each event can have multiple handlers:

| Event | Handler | Action |
|-------|---------|--------|
| `UserRegistered` | SendWelcomeEmail | Queue welcome email |
| `UserRegistered` | CreateUserProfile | Initialize profile |
| `UserRegistered` | NotifySlack | Post to #new-users |
| `UserRegistered` | UpdateAnalytics | Track signup |

## Publishing Events

### Synchronous (in-process)
- Same transaction as triggering action
- All handlers must succeed
- Use for critical side effects

### Asynchronous (queue)
- Eventually consistent
- Handlers can fail independently
- Use for non-critical side effects

## Event Versioning

When event schema changes:

1. Increment version number
2. Support reading old versions
3. Write new version only
4. Document migration path

## Idempotency

Handlers must be idempotent:
- Same event processed twice = same result
- Use eventId to detect duplicates
- Design for at-least-once delivery

## Event Storage

Events can be:
- **Fire and forget** - Not stored
- **Stored temporarily** - For retry/replay
- **Stored permanently** - Event sourcing

## Adding New Events

1. Add to this catalog
2. Define payload schema
3. Document handlers
4. Consider versioning from start
5. Add monitoring for event processing
