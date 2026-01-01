# Anti-Patterns

This file documents patterns to avoid. When generating code, never produce these patterns. When reviewing code, flag these as issues.

## Architecture Anti-Patterns

### God Service
```go
// BAD: Service does everything
type UserService struct {
    db *sql.DB
}

func (s *UserService) CreateUser(...)    { /* direct DB access */ }
func (s *UserService) SendEmail(...)     { /* email logic here */ }
func (s *UserService) ProcessPayment(...) { /* payment logic here */ }
func (s *UserService) GenerateReport(...) { /* reporting here */ }
```

```go
// GOOD: Single responsibility, delegated concerns
type UserService struct {
    repo   UserRepository
    events EventPublisher
}

func (s *UserService) CreateUser(...) {
    user := s.repo.Create(...)
    s.events.Publish(UserCreatedEvent{...})
    return user
}
```

### Leaky Abstraction
```typescript
// BAD: Repository exposes database details
interface UserRepository {
  findBySQL(query: string): Promise<User[]>;
  getConnection(): DatabaseConnection;
}

// GOOD: Repository hides implementation
interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<User>;
}
```

### Circular Dependencies
```
// BAD: A depends on B, B depends on A
UserService → AuthService → UserService

// GOOD: Introduce abstraction or event
UserService → AuthService
UserService ← UserCreatedEvent ← AuthService
```

## Code Anti-Patterns

### Stringly Typed
```python
# BAD: Using strings for structured data
def get_user_status(user):
    return "active"  # Magic string

def check_status(status):
    if status == "actve":  # Typo won't be caught
        return True

# GOOD: Use enums or constants
class UserStatus(Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"

def get_user_status(user) -> UserStatus:
    return UserStatus.ACTIVE
```

### Boolean Blindness
```java
// BAD: What do these booleans mean?
createUser(name, email, true, false, true);

// GOOD: Use named parameters or builder
CreateUserRequest.builder()
    .name(name)
    .email(email)
    .sendWelcomeEmail(true)
    .requireEmailVerification(false)
    .isAdmin(true)
    .build();
```

### Primitive Obsession
```typescript
// BAD: Primitives for domain concepts
function createUser(email: string, age: number, balance: number) { }

// GOOD: Domain types
function createUser(email: Email, age: Age, balance: Money) { }
```

### Silent Failure
```go
// BAD: Errors ignored
result, _ := riskyOperation()
return result

// GOOD: Errors handled
result, err := riskyOperation()
if err != nil {
    return nil, fmt.Errorf("risky operation failed: %w", err)
}
return result, nil
```

## Security Anti-Patterns

### Hardcoded Secrets
```python
# BAD: Never do this
API_KEY = "sk-1234567890abcdef"
DB_PASSWORD = "admin123"

# GOOD: Environment variables
API_KEY = os.environ["API_KEY"]
DB_PASSWORD = os.environ["DB_PASSWORD"]
```

### SQL Injection
```java
// BAD: String concatenation
String query = "SELECT * FROM users WHERE id = " + userId;

// GOOD: Parameterized query
String query = "SELECT * FROM users WHERE id = ?";
preparedStatement.setString(1, userId);
```

### Exposing Stack Traces
```typescript
// BAD: Leaking internals
catch (error) {
  res.status(500).json({ error: error.stack });
}

// GOOD: Generic message, log details
catch (error) {
  logger.error('Operation failed', { error, requestId });
  res.status(500).json({ error: 'Internal server error', requestId });
}
```

### Trusting Client Data
```go
// BAD: Using user-provided ID for authorization
userID := r.FormValue("user_id")
user := db.GetUser(userID)

// GOOD: Use authenticated user context
userID := r.Context().Value("authenticated_user_id")
user := db.GetUser(userID)
```

## Testing Anti-Patterns

### Testing Implementation Details
```typescript
// BAD: Testing private methods or internal state
expect(service._internalCache.size).toBe(5);
expect(service.privateMethod()).toBe(true);

// GOOD: Testing public behavior
expect(await service.getUser(id)).toEqual(expectedUser);
```

### Flaky Tests
```python
# BAD: Depends on timing or external state
def test_user_creation():
    user = create_user()
    time.sleep(2)  # Hope the async process finished
    assert user.email_sent == True

# GOOD: Explicit waits or mocking
def test_user_creation():
    email_service = Mock()
    user = create_user(email_service=email_service)
    email_service.send.assert_called_once()
```

### Test Interdependence
```java
// BAD: Tests depend on execution order
@Test void test1_createUser() { /* creates user */ }
@Test void test2_getUser() { /* assumes user from test1 exists */ }

// GOOD: Each test is independent
@Test void createUser_success() { /* creates and verifies */ }
@Test void getUser_success() { /* creates own user, then gets */ }
```

## Database Anti-Patterns

### N+1 Queries
```python
# BAD: Query per user
users = db.query("SELECT * FROM users")
for user in users:
    profile = db.query(f"SELECT * FROM profiles WHERE user_id = {user.id}")

# GOOD: Single query with join
users = db.query("""
    SELECT u.*, p.* FROM users u
    LEFT JOIN profiles p ON p.user_id = u.id
""")
```

### Unbounded Queries
```sql
-- BAD: No limit
SELECT * FROM audit_logs WHERE user_id = ?;

-- GOOD: Always paginate
SELECT * FROM audit_logs WHERE user_id = ? LIMIT 100 OFFSET 0;
```

---

**Usage**: Before generating code, mentally check against these patterns. If you catch yourself producing any of these, stop and refactor.
