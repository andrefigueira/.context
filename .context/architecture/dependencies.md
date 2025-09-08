# Architecture: Dependency Management

This document defines dependency injection patterns and service lifecycle management for [Your Project Name], ensuring consistent, testable, and maintainable code architecture.

## Dependency Injection Pattern

The project uses manual dependency injection through constructor injection, providing explicit control over service dependencies and lifecycles.

## Service Registration

### Service Container Structure
```go
// Container holds all application services and their dependencies
type Container struct {
    // Infrastructure
    DB     *sql.DB
    Cache  *redis.Client
    Logger Logger
    Config *Config
    
    // Repositories
    UserRepo    UserRepository
    ProductRepo ProductRepository
    
    // Services
    UserService    UserService
    ProductService ProductService
    
    // Handlers
    UserHandler    *UserHandler
    ProductHandler *ProductHandler
}

// NewContainer initializes all services with proper dependency injection
func NewContainer(config *Config) (*Container, error) {
    container := &Container{
        Config: config,
    }
    
    if err := container.initInfrastructure(); err != nil {
        return nil, err
    }
    
    container.initRepositories()
    container.initServices()
    container.initHandlers()
    
    return container, nil
}
```

### Infrastructure Initialization
```go
func (c *Container) initInfrastructure() error {
    // Database connection
    db, err := sql.Open("postgres", c.Config.DatabaseURL)
    if err != nil {
        return fmt.Errorf("failed to connect to database: %w", err)
    }
    db.SetMaxOpenConns(c.Config.DB.MaxOpenConns)
    db.SetMaxIdleConns(c.Config.DB.MaxIdleConns)
    c.DB = db
    
    // Cache connection
    cache := redis.NewClient(&redis.Options{
        Addr:     c.Config.RedisURL,
        Password: c.Config.RedisPassword,
        DB:       0,
    })
    c.Cache = cache
    
    // Logger
    c.Logger = NewStructuredLogger(c.Config.LogLevel)
    
    return nil
}
```

### Repository Layer Injection
```go
func (c *Container) initRepositories() {
    c.UserRepo = NewPgUserRepository(c.DB, c.Logger)
    c.ProductRepo = NewPgProductRepository(c.DB, c.Logger)
}

// Repository constructor pattern
func NewPgUserRepository(db *sql.DB, logger Logger) UserRepository {
    return &pgUserRepository{
        db:     db,
        logger: logger.WithComponent("user_repository"),
    }
}
```

### Service Layer Injection
```go
func (c *Container) initServices() {
    c.UserService = NewUserService(
        c.UserRepo,
        NewEmailValidator(),
        c.Cache,
        c.Logger,
    )
    
    c.ProductService = NewProductService(
        c.ProductRepo,
        c.UserService, // Service dependency
        c.Logger,
    )
}

// Service constructor with multiple dependencies
func NewUserService(
    repo UserRepository,
    validator EmailValidator,
    cache Cache,
    logger Logger,
) UserService {
    return &userService{
        repo:      repo,
        validator: validator,
        cache:     cache,
        logger:    logger.WithComponent("user_service"),
    }
}
```

### Handler Layer Injection
```go
func (c *Container) initHandlers() {
    c.UserHandler = NewUserHandler(c.UserService, c.Logger)
    c.ProductHandler = NewProductHandler(
        c.ProductService,
        c.UserService, // Cross-service dependency
        c.Logger,
    )
}
```

## Interface-Based Dependencies

### Service Interfaces
Define clear interfaces for all service dependencies:
```go
// UserService interface defines business operations
type UserService interface {
    CreateUser(ctx context.Context, req CreateUserRequest) (*User, error)
    GetUser(ctx context.Context, userID string) (*User, error)
    UpdateUser(ctx context.Context, userID string, req UpdateUserRequest) (*User, error)
    DeleteUser(ctx context.Context, userID string) error
    ListUsers(ctx context.Context, filter UserFilter) ([]*User, error)
}

// Repository interface abstracts data access
type UserRepository interface {
    Create(ctx context.Context, user *User) (*User, error)
    GetByID(ctx context.Context, id string) (*User, error)
    GetByEmail(ctx context.Context, email string) (*User, error)
    Update(ctx context.Context, user *User) (*User, error)
    Delete(ctx context.Context, id string) error
    List(ctx context.Context, filter UserFilter) ([]*User, error)
}
```

### Cross-Service Dependencies
When services depend on other services, use interfaces to prevent circular dependencies:
```go
// ProductService depends on user operations
type ProductService interface {
    CreateProduct(ctx context.Context, ownerID string, req CreateProductRequest) (*Product, error)
    GetUserProducts(ctx context.Context, userID string) ([]*Product, error)
}

type productService struct {
    repo        ProductRepository
    userService UserService // Interface dependency
    logger      Logger
}

func (s *productService) CreateProduct(ctx context.Context, ownerID string, req CreateProductRequest) (*Product, error) {
    // Validate owner exists
    if _, err := s.userService.GetUser(ctx, ownerID); err != nil {
        return nil, fmt.Errorf("invalid owner: %w", err)
    }
    
    product := &Product{
        OwnerID: ownerID,
        Name:    req.Name,
        Price:   req.Price,
    }
    
    return s.repo.Create(ctx, product)
}
```

## Lifecycle Management

### Service Shutdown
```go
// Cleanup implements graceful shutdown for all resources
func (c *Container) Cleanup() error {
    var errors []error
    
    // Close database connection
    if err := c.DB.Close(); err != nil {
        errors = append(errors, fmt.Errorf("database close: %w", err))
    }
    
    // Close cache connection
    if err := c.Cache.Close(); err != nil {
        errors = append(errors, fmt.Errorf("cache close: %w", err))
    }
    
    // Flush logger
    if err := c.Logger.Sync(); err != nil {
        errors = append(errors, fmt.Errorf("logger sync: %w", err))
    }
    
    if len(errors) > 0 {
        return fmt.Errorf("cleanup errors: %v", errors)
    }
    
    return nil
}
```

### Graceful Application Shutdown
```go
func main() {
    container, err := NewContainer(config)
    if err != nil {
        log.Fatal("Failed to initialize container:", err)
    }
    defer container.Cleanup()
    
    // Setup graceful shutdown
    ctx, cancel := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
    defer cancel()
    
    server := &http.Server{
        Addr:    ":8080",
        Handler: setupRoutes(container),
    }
    
    // Start server in goroutine
    go func() {
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatal("Server failed to start:", err)
        }
    }()
    
    // Wait for shutdown signal
    <-ctx.Done()
    
    // Graceful shutdown with timeout
    shutdownCtx, shutdownCancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer shutdownCancel()
    
    if err := server.Shutdown(shutdownCtx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }
}
```

## Testing Patterns

### Mock Dependencies
```go
// Mock implementations for testing
type mockUserRepository struct {
    users map[string]*User
}

func (m *mockUserRepository) Create(ctx context.Context, user *User) (*User, error) {
    user.ID = generateID()
    m.users[user.ID] = user
    return user, nil
}

func (m *mockUserRepository) GetByID(ctx context.Context, id string) (*User, error) {
    user, exists := m.users[id]
    if !exists {
        return nil, ErrUserNotFound
    }
    return user, nil
}

// Test service with mocked dependencies
func TestUserService_CreateUser(t *testing.T) {
    repo := &mockUserRepository{users: make(map[string]*User)}
    validator := &mockEmailValidator{}
    cache := &mockCache{}
    logger := &mockLogger{}
    
    service := NewUserService(repo, validator, cache, logger)
    
    req := CreateUserRequest{
        Email: "test@example.com",
        Name:  "Test User",
    }
    
    user, err := service.CreateUser(context.Background(), req)
    
    assert.NoError(t, err)
    assert.Equal(t, req.Email, user.Email)
    assert.NotEmpty(t, user.ID)
}
```

### Integration Testing Container
```go
// TestContainer for integration tests with real dependencies
func NewTestContainer() (*Container, error) {
    config := &Config{
        DatabaseURL: getTestDatabaseURL(),
        RedisURL:    getTestRedisURL(),
        LogLevel:    "debug",
    }
    
    return NewContainer(config)
}

func TestIntegration_UserFlow(t *testing.T) {
    container, err := NewTestContainer()
    require.NoError(t, err)
    defer container.Cleanup()
    
    // Run migrations
    require.NoError(t, runTestMigrations(container.DB))
    
    // Test full user creation flow
    req := CreateUserRequest{
        Email: "integration@example.com",
        Name:  "Integration Test",
    }
    
    user, err := container.UserService.CreateUser(context.Background(), req)
    require.NoError(t, err)
    
    // Verify user was persisted
    retrieved, err := container.UserService.GetUser(context.Background(), user.ID)
    require.NoError(t, err)
    assert.Equal(t, user.Email, retrieved.Email)
}
```

## Decision History & Trade-offs

### Manual DI vs DI Container
**Decision**: Use manual dependency injection instead of a DI container/framework
**Rationale**:
- Explicit dependencies make code easier to understand and debug
- No magic or reflection-based wiring
- Better IDE support for navigation and refactoring
- Simpler testing setup with no framework-specific mocking

**Trade-offs**:
- More boilerplate code for wiring dependencies
- Constructor parameter explosion for services with many dependencies
- Manual lifecycle management required
- Harder to implement features like lazy loading or scoped instances

### Container Pattern
**Decision**: Use a single container struct to hold all services
**Rationale**:
- Single point of initialization and dependency wiring
- Clear service lifecycle management
- Easy to understand service relationships
- Simplified testing with mock container variants

**Trade-offs**:
- Large container struct with many fields
- Potential circular dependency issues require careful ordering
- All services initialized at startup (no lazy loading)
- Container becomes god object in complex applications

### Interface Segregation
**Decision**: Define focused interfaces for each service responsibility
**Rationale**:
- Enables precise mocking in tests
- Supports interface segregation principle
- Allows for multiple implementations
- Reduces coupling between layers

**Trade-offs**:
- Additional interface definitions increase code volume
- Potential interface explosion for fine-grained responsibilities
- Requires discipline to maintain interface/implementation consistency

---

**Next**: Review [patterns.md](./patterns.md) for coding conventions and implementation patterns.