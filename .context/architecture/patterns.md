# Architecture: Implementation Patterns

This document defines coding standards, design patterns, and implementation conventions for [Your Project Name], ensuring consistency and maintainability across the codebase.

## Code Organization Patterns

### Package Structure
```
cmd/
├── api/          # Main application entrypoint
│   └── main.go
├── migrate/      # Database migration tool
│   └── main.go
└── worker/       # Background job processor
    └── main.go

internal/
├── api/          # HTTP handlers and middleware
├── auth/         # Authentication logic
├── config/       # Configuration management
├── database/     # Data access layer
├── models/       # Domain models
├── services/     # Business logic
└── utils/        # Shared utilities

migrations/       # Database schema changes
docs/            # API documentation
```

### File Naming Conventions
- **Interfaces**: `user_service.go` (defines UserService interface)
- **Implementations**: `user_service_impl.go` or `pg_user_repository.go`
- **Tests**: `user_service_test.go`
- **Mocks**: `user_service_mock.go` or `mocks/user_service.go`

## Error Handling Patterns

### Domain Error Types
```go
// Base error structure for domain-specific errors
type DomainError struct {
    Code       string                 `json:"code"`
    Message    string                 `json:"message"`
    Details    map[string]interface{} `json:"details,omitempty"`
    Cause      error                  `json:"-"`
    StatusCode int                    `json:"-"`
}

func (e *DomainError) Error() string {
    if e.Cause != nil {
        return fmt.Sprintf("%s: %s", e.Message, e.Cause.Error())
    }
    return e.Message
}

func (e *DomainError) Unwrap() error {
    return e.Cause
}

// Predefined domain errors
var (
    ErrUserNotFound = &DomainError{
        Code:       "USER_NOT_FOUND",
        Message:    "User not found",
        StatusCode: 404,
    }
    
    ErrInvalidEmail = &DomainError{
        Code:       "INVALID_EMAIL",
        Message:    "Invalid email format",
        StatusCode: 400,
    }
    
    ErrDuplicateEmail = &DomainError{
        Code:       "DUPLICATE_EMAIL",
        Message:    "Email already exists",
        StatusCode: 409,
    }
)
```

### Error Wrapping and Context
```go
// Service layer error handling
func (s *userService) CreateUser(ctx context.Context, req CreateUserRequest) (*User, error) {
    // Validation
    if err := s.validator.Validate(req); err != nil {
        return nil, &DomainError{
            Code:       "VALIDATION_FAILED",
            Message:    "User validation failed",
            Details:    map[string]interface{}{"validation_errors": err},
            Cause:      err,
            StatusCode: 400,
        }
    }
    
    // Repository call
    user, err := s.repo.Create(ctx, &User{
        Email: req.Email,
        Name:  req.Name,
    })
    if err != nil {
        // Check for specific database errors
        if isDuplicateKeyError(err) {
            return nil, &DomainError{
                Code:       "DUPLICATE_EMAIL",
                Message:    "Email already exists",
                Cause:      err,
                StatusCode: 409,
            }
        }
        
        // Generic database error
        return nil, fmt.Errorf("failed to create user: %w", err)
    }
    
    return user, nil
}
```

### HTTP Error Handling
```go
// Handler error response pattern
func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        h.writeError(w, &DomainError{
            Code:       "INVALID_JSON",
            Message:    "Invalid JSON in request body",
            Cause:      err,
            StatusCode: 400,
        })
        return
    }
    
    user, err := h.userService.CreateUser(r.Context(), req)
    if err != nil {
        h.writeError(w, err)
        return
    }
    
    h.writeJSON(w, http.StatusCreated, user)
}

func (h *BaseHandler) writeError(w http.ResponseWriter, err error) {
    var domainErr *DomainError
    if errors.As(err, &domainErr) {
        h.writeJSON(w, domainErr.StatusCode, domainErr)
        return
    }
    
    // Unknown error - return generic 500
    genericErr := &DomainError{
        Code:    "INTERNAL_ERROR",
        Message: "An internal error occurred",
    }
    h.writeJSON(w, 500, genericErr)
    
    // Log the actual error for debugging
    h.logger.Error("Unhandled error", "error", err)
}
```

## Validation Patterns

### Input Validation
```go
// Validation interface
type Validator interface {
    Validate(interface{}) error
}

// Validation implementation using struct tags
type structValidator struct{}

func (v *structValidator) Validate(s interface{}) error {
    return validator.New().Struct(s)
}

// Request structure with validation tags
type CreateUserRequest struct {
    Email string `json:"email" validate:"required,email"`
    Name  string `json:"name" validate:"required,min=2,max=100"`
    Age   int    `json:"age" validate:"min=0,max=150"`
}

// Business rule validation
type UserValidator struct {
    repo UserRepository
}

func (v *UserValidator) ValidateCreateUser(ctx context.Context, req CreateUserRequest) error {
    // Check email uniqueness
    if _, err := v.repo.GetByEmail(ctx, req.Email); err == nil {
        return ErrDuplicateEmail
    }
    
    // Business rule: minimum age for account creation
    if req.Age < 13 {
        return &DomainError{
            Code:    "AGE_RESTRICTION",
            Message: "Users must be at least 13 years old",
        }
    }
    
    return nil
}
```

## Repository Patterns

### Repository Interface
```go
// Generic repository pattern
type Repository[T any, ID comparable] interface {
    Create(ctx context.Context, entity *T) (*T, error)
    GetByID(ctx context.Context, id ID) (*T, error)
    Update(ctx context.Context, entity *T) (*T, error)
    Delete(ctx context.Context, id ID) error
    List(ctx context.Context, filter ListFilter) ([]*T, error)
}

// Specific repository with domain methods
type UserRepository interface {
    Repository[User, string]
    GetByEmail(ctx context.Context, email string) (*User, error)
    GetByStatus(ctx context.Context, status UserStatus) ([]*User, error)
}
```

### Repository Implementation
```go
type pgUserRepository struct {
    db     *sql.DB
    logger Logger
}

func NewPgUserRepository(db *sql.DB, logger Logger) UserRepository {
    return &pgUserRepository{
        db:     db,
        logger: logger.WithComponent("user_repository"),
    }
}

func (r *pgUserRepository) Create(ctx context.Context, user *User) (*User, error) {
    query := `
        INSERT INTO users (email, name, status, created_at) 
        VALUES ($1, $2, $3, $4) 
        RETURNING id, updated_at`
    
    err := r.db.QueryRowContext(ctx, query, 
        user.Email, user.Name, user.Status, user.CreatedAt).
        Scan(&user.ID, &user.UpdatedAt)
    
    if err != nil {
        r.logger.Error("Failed to create user", "error", err, "email", user.Email)
        return nil, fmt.Errorf("create user: %w", err)
    }
    
    r.logger.Info("User created", "user_id", user.ID, "email", user.Email)
    return user, nil
}

func (r *pgUserRepository) GetByEmail(ctx context.Context, email string) (*User, error) {
    query := `
        SELECT id, email, name, status, created_at, updated_at 
        FROM users 
        WHERE email = $1`
    
    var user User
    err := r.db.QueryRowContext(ctx, query, email).Scan(
        &user.ID, &user.Email, &user.Name, 
        &user.Status, &user.CreatedAt, &user.UpdatedAt,
    )
    
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrUserNotFound
        }
        return nil, fmt.Errorf("get user by email: %w", err)
    }
    
    return &user, nil
}
```

## Service Patterns

### Service Interface and Implementation
```go
// Service interface
type UserService interface {
    CreateUser(ctx context.Context, req CreateUserRequest) (*User, error)
    GetUser(ctx context.Context, userID string) (*User, error)
    UpdateUser(ctx context.Context, userID string, req UpdateUserRequest) (*User, error)
    DeleteUser(ctx context.Context, userID string) error
    ListUsers(ctx context.Context, filter UserFilter) (*UserList, error)
}

// Service implementation
type userService struct {
    repo      UserRepository
    validator UserValidator
    cache     Cache
    logger    Logger
}

func NewUserService(repo UserRepository, validator UserValidator, cache Cache, logger Logger) UserService {
    return &userService{
        repo:      repo,
        validator: validator,
        cache:     cache,
        logger:    logger.WithComponent("user_service"),
    }
}

func (s *userService) CreateUser(ctx context.Context, req CreateUserRequest) (*User, error) {
    // Input validation
    if err := s.validator.ValidateCreateUser(ctx, req); err != nil {
        return nil, err
    }
    
    // Create user entity
    user := &User{
        Email:     req.Email,
        Name:      req.Name,
        Status:    UserStatusActive,
        CreatedAt: time.Now(),
    }
    
    // Persist to database
    createdUser, err := s.repo.Create(ctx, user)
    if err != nil {
        s.logger.Error("Failed to create user", "error", err, "email", req.Email)
        return nil, err
    }
    
    // Cache the created user
    cacheKey := fmt.Sprintf("user:%s", createdUser.ID)
    if err := s.cache.Set(ctx, cacheKey, createdUser, 1*time.Hour); err != nil {
        s.logger.Warn("Failed to cache user", "error", err, "user_id", createdUser.ID)
        // Don't fail the request for cache errors
    }
    
    s.logger.Info("User created successfully", "user_id", createdUser.ID, "email", createdUser.Email)
    return createdUser, nil
}
```

## Testing Patterns

### Unit Test Structure
```go
func TestUserService_CreateUser(t *testing.T) {
    tests := []struct {
        name          string
        request       CreateUserRequest
        setupMocks    func(*mockUserRepository, *mockUserValidator, *mockCache)
        expectedError string
        validate      func(*testing.T, *User)
    }{
        {
            name: "successful_creation",
            request: CreateUserRequest{
                Email: "test@example.com",
                Name:  "Test User",
            },
            setupMocks: func(repo *mockUserRepository, validator *mockUserValidator, cache *mockCache) {
                validator.On("ValidateCreateUser", mock.Anything, mock.Anything).Return(nil)
                repo.On("Create", mock.Anything, mock.Anything).Return(&User{
                    ID:    "user-123",
                    Email: "test@example.com",
                    Name:  "Test User",
                }, nil)
                cache.On("Set", mock.Anything, mock.Anything, mock.Anything, mock.Anything).Return(nil)
            },
            validate: func(t *testing.T, user *User) {
                assert.Equal(t, "user-123", user.ID)
                assert.Equal(t, "test@example.com", user.Email)
            },
        },
        {
            name: "validation_error",
            request: CreateUserRequest{
                Email: "invalid-email",
                Name:  "Test User",
            },
            setupMocks: func(repo *mockUserRepository, validator *mockUserValidator, cache *mockCache) {
                validator.On("ValidateCreateUser", mock.Anything, mock.Anything).Return(ErrInvalidEmail)
            },
            expectedError: "INVALID_EMAIL",
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Setup mocks
            repo := &mockUserRepository{}
            validator := &mockUserValidator{}
            cache := &mockCache{}
            logger := &mockLogger{}
            
            tt.setupMocks(repo, validator, cache)
            
            // Create service
            service := NewUserService(repo, validator, cache, logger)
            
            // Execute
            user, err := service.CreateUser(context.Background(), tt.request)
            
            // Assertions
            if tt.expectedError != "" {
                require.Error(t, err)
                var domainErr *DomainError
                require.True(t, errors.As(err, &domainErr))
                assert.Equal(t, tt.expectedError, domainErr.Code)
            } else {
                require.NoError(t, err)
                require.NotNil(t, user)
                if tt.validate != nil {
                    tt.validate(t, user)
                }
            }
            
            // Verify mocks
            repo.AssertExpectations(t)
            validator.AssertExpectations(t)
            cache.AssertExpectations(t)
        })
    }
}
```

## Decision History & Trade-offs

### Repository Pattern Implementation
**Decision**: Use repository pattern with interface-first design
**Rationale**: 
- Abstracts data access implementation from business logic
- Enables comprehensive testing with mock implementations
- Supports multiple storage backends
- Clear separation of concerns

**Trade-offs**:
- Additional abstraction complexity
- Potential for leaky abstractions with complex queries
- More code to maintain (interfaces + implementations)

### Error Handling Strategy
**Decision**: Use structured domain errors with HTTP status mapping
**Rationale**:
- Consistent error responses across API
- Clear distinction between domain and infrastructure errors
- Structured error information for API clients
- Proper HTTP status code mapping

**Trade-offs**:
- More complex error handling logic
- Risk of error information leakage
- Additional error type definitions to maintain

### Validation Approach
**Decision**: Combine struct tag validation with business rule validation
**Rationale**:
- Struct tags handle basic format validation efficiently
- Business rule validation handles domain-specific constraints
- Clear separation between syntactic and semantic validation
- Reusable validation logic

**Trade-offs**:
- Two-layer validation increases complexity
- Potential for validation logic duplication
- Requires careful coordination between validation layers

---

**Next**: Review [../auth/overview.md](../auth/overview.md) for authentication and authorization patterns.