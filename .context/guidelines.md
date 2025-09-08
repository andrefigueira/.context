# Development Guidelines

This document establishes development workflows, testing standards, deployment procedures, and cross-cutting concerns for [Your Project Name]. These guidelines ensure consistency, quality, and maintainability across the development team.

## Development Workflow

### Git Workflow
The project follows a modified GitFlow with feature branches and pull request reviews.

#### Branch Strategy
```bash
main                    # Production-ready code
├── develop            # Integration branch for features
├── feature/auth-jwt   # Feature development
├── hotfix/security-patch  # Critical production fixes
└── release/v1.2.0     # Release preparation
```

#### Commit Message Format
```
type(scope): brief description

Longer description explaining the change in more detail.
Include any breaking changes, migration notes, or special
deployment considerations.

Closes #123
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`

**Examples**:
```
feat(auth): add JWT refresh token rotation

Implement automatic refresh token rotation for enhanced security.
Tokens are rotated on each use and old tokens are immediately
invalidated to prevent replay attacks.

Breaking change: Refresh token endpoint now requires token in
request body instead of just cookies.

Closes #456
```

#### Pull Request Process
1. **Create feature branch** from `develop`
2. **Implement changes** following coding standards
3. **Write tests** with minimum 80% coverage
4. **Update documentation** for public APIs
5. **Create pull request** with detailed description
6. **Address review feedback** promptly
7. **Squash and merge** after approval

### Code Review Standards

#### Review Checklist
**Functionality**:
- [ ] Code solves the intended problem
- [ ] Edge cases are handled appropriately
- [ ] Error handling is comprehensive
- [ ] Performance implications considered

**Security**:
- [ ] Input validation implemented
- [ ] Authentication/authorization checked
- [ ] No secrets in code or logs
- [ ] SQL injection prevention

**Code Quality**:
- [ ] Follows established patterns
- [ ] Readable and well-structured
- [ ] Appropriate abstractions
- [ ] No code duplication

**Testing**:
- [ ] Unit tests for business logic
- [ ] Integration tests for critical paths
- [ ] Test coverage meets standards
- [ ] Tests are maintainable

**Documentation**:
- [ ] Public APIs documented
- [ ] Complex logic explained
- [ ] README updated if needed
- [ ] Breaking changes noted

#### Review Response Guidelines
- **Respond within 24 hours** during work days
- **Be constructive** and suggest solutions
- **Explain reasoning** behind feedback
- **Approve small changes** without blocking
- **Request changes** for security or correctness issues

## Testing Standards

### Test Pyramid Strategy
```
        Unit Tests (70%)
    Integration Tests (20%)
End-to-End Tests (10%)
```

### Unit Testing
**Coverage Targets**:
- Business logic: 90%+ coverage
- API handlers: 80%+ coverage
- Utilities: 95%+ coverage
- Overall project: 80%+ coverage

**Testing Patterns**:
```go
func TestUserService_CreateUser(t *testing.T) {
    tests := []struct {
        name          string
        request       CreateUserRequest
        setupMocks    func(*mockUserRepository, *mockValidator)
        expectedError string
        validate      func(*testing.T, *User)
    }{
        {
            name: "successful_creation",
            request: CreateUserRequest{
                Email: "test@example.com",
                Name:  "Test User",
                Password: "SecurePassword123!",
            },
            setupMocks: func(repo *mockUserRepository, validator *mockValidator) {
                validator.On("Validate", mock.Anything).Return(nil)
                repo.On("Create", mock.Anything, mock.MatchedBy(func(u *User) bool {
                    return u.Email == "test@example.com"
                })).Return(&User{
                    ID:    "user-123",
                    Email: "test@example.com",
                    Name:  "Test User",
                }, nil)
            },
            validate: func(t *testing.T, user *User) {
                assert.Equal(t, "user-123", user.ID)
                assert.Equal(t, "test@example.com", user.Email)
                assert.Equal(t, "Test User", user.Name)
                assert.Equal(t, UserStatusActive, user.Status)
            },
        },
        {
            name: "validation_error",
            request: CreateUserRequest{
                Email: "invalid-email",
                Name:  "",
                Password: "weak",
            },
            setupMocks: func(repo *mockUserRepository, validator *mockValidator) {
                validator.On("Validate", mock.Anything).Return(
                    errors.New("validation failed"),
                )
            },
            expectedError: "validation failed",
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Setup
            repo := &mockUserRepository{}
            validator := &mockValidator{}
            logger := &mockLogger{}
            
            if tt.setupMocks != nil {
                tt.setupMocks(repo, validator)
            }
            
            service := NewUserService(repo, validator, logger)
            
            // Execute
            result, err := service.CreateUser(context.Background(), tt.request)
            
            // Assert
            if tt.expectedError != "" {
                assert.Error(t, err)
                assert.Contains(t, err.Error(), tt.expectedError)
                assert.Nil(t, result)
            } else {
                assert.NoError(t, err)
                assert.NotNil(t, result)
                if tt.validate != nil {
                    tt.validate(t, result)
                }
            }
            
            // Verify mocks
            repo.AssertExpectations(t)
            validator.AssertExpectations(t)
        })
    }
}
```

### Integration Testing
**Database Tests**:
```go
func TestIntegration_UserRepository(t *testing.T) {
    if testing.Short() {
        t.Skip("Skipping integration test")
    }
    
    // Setup test database
    db := setupTestDB(t)
    defer cleanupTestDB(t, db)
    
    repo := NewPgUserRepository(db, testLogger)
    
    t.Run("create_and_get_user", func(t *testing.T) {
        // Create user
        user := &User{
            Email: "integration@example.com",
            Name:  "Integration Test",
            Status: UserStatusActive,
        }
        
        created, err := repo.Create(context.Background(), user)
        require.NoError(t, err)
        require.NotEmpty(t, created.ID)
        
        // Get user
        retrieved, err := repo.GetByID(context.Background(), created.ID)
        require.NoError(t, err)
        assert.Equal(t, created.Email, retrieved.Email)
        assert.Equal(t, created.Name, retrieved.Name)
    })
}

func setupTestDB(t *testing.T) *sql.DB {
    dbURL := os.Getenv("TEST_DATABASE_URL")
    if dbURL == "" {
        t.Skip("TEST_DATABASE_URL not set")
    }
    
    db, err := sql.Open("postgres", dbURL)
    require.NoError(t, err)
    
    // Run migrations
    migrator := NewMigrator(db, "migrations", testLogger)
    require.NoError(t, migrator.Migrate())
    
    return db
}

func cleanupTestDB(t *testing.T, db *sql.DB) {
    // Truncate all tables
    tables := []string{"users", "user_profiles", "refresh_tokens"}
    for _, table := range tables {
        _, err := db.Exec(fmt.Sprintf("TRUNCATE TABLE %s CASCADE", table))
        require.NoError(t, err)
    }
    
    db.Close()
}
```

### API Testing
```go
func TestAPI_UserEndpoints(t *testing.T) {
    // Setup test server
    server := setupTestServer(t)
    defer server.Close()
    
    client := &http.Client{Timeout: 10 * time.Second}
    
    t.Run("user_registration_flow", func(t *testing.T) {
        // Register user
        regReq := CreateUserRequest{
            Email:    "api@example.com",
            Name:     "API Test",
            Password: "SecurePassword123!",
        }
        
        regResp := makeRequest(t, client, "POST", server.URL+"/auth/register", regReq)
        assert.Equal(t, http.StatusCreated, regResp.StatusCode)
        
        var user User
        err := json.NewDecoder(regResp.Body).Decode(&user)
        require.NoError(t, err)
        assert.Equal(t, regReq.Email, user.Email)
        
        // Login
        loginReq := LoginRequest{
            Email:    regReq.Email,
            Password: regReq.Password,
        }
        
        loginResp := makeRequest(t, client, "POST", server.URL+"/auth/login", loginReq)
        assert.Equal(t, http.StatusOK, loginResp.StatusCode)
        
        var loginResult LoginResponse
        err = json.NewDecoder(loginResp.Body).Decode(&loginResult)
        require.NoError(t, err)
        assert.NotEmpty(t, loginResult.AccessToken)
        
        // Get profile
        req, _ := http.NewRequest("GET", server.URL+"/users/profile", nil)
        req.Header.Set("Authorization", "Bearer "+loginResult.AccessToken)
        
        profileResp, err := client.Do(req)
        require.NoError(t, err)
        assert.Equal(t, http.StatusOK, profileResp.StatusCode)
    })
}
```

## Deployment Procedures

### Environment Strategy
```
Development → Staging → Production
     ↓           ↓         ↓
   Feature    Integration  Live
   Testing     Testing    Traffic
```

### Deployment Pipeline
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
      redis:
        image: redis:7
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Go
        uses: actions/setup-go@v3
        with:
          go-version: '1.19'
      
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/go/pkg/mod
          key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
      
      - name: Install dependencies
        run: go mod download
      
      - name: Run tests
        run: |
          go test ./... -v -race -coverprofile=coverage.out
          go tool cover -html=coverage.out -o coverage.html
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: coverage.out
  
  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Build binary
        run: |
          CGO_ENABLED=0 GOOS=linux go build -o app ./cmd/api
      
      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.REGISTRY }}/app:${{ github.sha }} .
          docker push ${{ secrets.REGISTRY }}/app:${{ github.sha }}
  
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - name: Deploy to staging
        run: |
          # Deploy using your preferred method (k8s, docker-compose, etc.)
          kubectl set image deployment/app app=${{ secrets.REGISTRY }}/app:${{ github.sha }}
          kubectl rollout status deployment/app
  
  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Deploy to production
        run: |
          # Production deployment with additional safety checks
          # Blue-green deployment or rolling update
          echo "Deploying to production..."
```

### Release Process
1. **Create release branch** from `develop`
2. **Update version numbers** and changelog
3. **Run full test suite** including load tests
4. **Deploy to staging** for final validation
5. **Create release PR** to `main`
6. **Deploy to production** after merge
7. **Tag release** with semantic version
8. **Monitor deployment** metrics and logs

### Rollback Procedures
```bash
#!/bin/bash
# rollback.sh - Production rollback script

set -euo pipefail

PREVIOUS_VERSION="${1:-}"
if [ -z "$PREVIOUS_VERSION" ]; then
    echo "Usage: $0 <previous-version>"
    echo "Example: $0 v1.2.3"
    exit 1
fi

echo "=== Rolling back to $PREVIOUS_VERSION ==="

# Database rollback (if needed)
if [ "${ROLLBACK_DB:-false}" = "true" ]; then
    echo "Rolling back database..."
    ./migrate -database "$DATABASE_URL" -path ./migrations down -limit 1
fi

# Application rollback
echo "Rolling back application..."
kubectl set image deployment/app app=$REGISTRY/app:$PREVIOUS_VERSION
kubectl rollout status deployment/app

# Verify rollback
echo "Verifying rollback..."
kubectl get pods -l app=myapp
curl -f http://app.example.com/health

echo "✅ Rollback completed successfully"
```

## Performance Standards

### Response Time Targets
- **API endpoints**: 95th percentile < 200ms
- **Database queries**: 95th percentile < 50ms
- **Authentication**: 99th percentile < 100ms
- **Health checks**: 99th percentile < 10ms

### Scaling Guidelines
- **Horizontal scaling**: Stateless application design
- **Database scaling**: Read replicas for read-heavy workloads
- **Cache utilization**: Redis for session and frequently accessed data
- **CDN usage**: Static assets and API responses when appropriate

### Monitoring Requirements
```go
// Metrics collection example
type Metrics struct {
    requestDuration   prometheus.HistogramVec
    requestCount      prometheus.CounterVec
    activeConnections prometheus.Gauge
    dbConnections     prometheus.Gauge
}

func (m *Metrics) RecordRequest(method, path string, statusCode int, duration time.Duration) {
    m.requestDuration.WithLabelValues(method, path, strconv.Itoa(statusCode)).Observe(duration.Seconds())
    m.requestCount.WithLabelValues(method, path, strconv.Itoa(statusCode)).Inc()
}

func metricsMiddleware(metrics *Metrics) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            wrapped := &responseWriter{ResponseWriter: w}
            next.ServeHTTP(wrapped, r)
            
            duration := time.Since(start)
            metrics.RecordRequest(r.Method, r.URL.Path, wrapped.statusCode, duration)
        })
    }
}
```

## Security Guidelines

### Code Security
- **Input validation**: All user input validated and sanitized
- **SQL injection**: Use parameterized queries exclusively
- **XSS prevention**: Proper output encoding and CSP headers
- **Authentication**: Multi-factor authentication for admin accounts
- **Authorization**: Principle of least privilege

### Dependency Management
```bash
# Regular security audits
go list -json -m all | nancy sleuth

# Update dependencies monthly
go get -u ./...
go mod tidy

# Check for vulnerabilities
govulncheck ./...
```

### Secret Management
- **Environment variables**: For configuration
- **Secret rotation**: Automated rotation of API keys and tokens  
- **Access logs**: Audit all secret access
- **Encryption**: All secrets encrypted at rest

## Documentation Standards

### Code Documentation
```go
// Package user provides user management functionality including
// registration, authentication, and profile management.
//
// The package follows a layered architecture with clear separation
// between HTTP handlers, business logic, and data access.
package user

// UserService defines the interface for user management operations.
// 
// All methods require a context for request tracing and cancellation.
// Authentication and authorization are handled by the caller.
type UserService interface {
    // CreateUser registers a new user account with the provided information.
    // Returns ErrDuplicateEmail if the email is already registered.
    // Returns ErrInvalidInput if the request fails validation.
    CreateUser(ctx context.Context, req CreateUserRequest) (*User, error)
    
    // GetUser retrieves a user by their unique identifier.
    // Returns ErrUserNotFound if the user does not exist.
    GetUser(ctx context.Context, userID string) (*User, error)
}

// CreateUserRequest contains the information needed to register a new user.
type CreateUserRequest struct {
    Email    string `json:"email" validate:"required,email"`
    Name     string `json:"name" validate:"required,min=1,max=255"`
    Password string `json:"password" validate:"required,min=12"`
}
```

### API Documentation
- **OpenAPI/Swagger**: Generated from code annotations
- **Postman collections**: For manual testing and examples
- **Integration guides**: Step-by-step client integration
- **Rate limiting**: Document all rate limits and quotas

### Architecture Decision Records
```markdown
# ADR-001: Use JWT for Authentication

## Status
Accepted

## Context
We need to implement authentication for our API that supports:
- Stateless operation for horizontal scaling
- Mobile and web client compatibility
- Reasonable security for user sessions

## Decision
Implement JWT-based authentication with refresh token rotation.

## Consequences
**Positive:**
- Stateless authentication enables easy horizontal scaling
- Standard format with good library support
- Embedded claims reduce database lookups

**Negative:**
- Token revocation requires additional infrastructure
- Larger request size due to token overhead
- Complex client-side token management

## Implementation Notes
- Access tokens: 15 minute lifetime
- Refresh tokens: 7 day lifetime with automatic rotation
- Blacklist for immediate token revocation
```

## Decision History & Trade-offs

### Testing Strategy Choice
**Decision**: Adopt test pyramid with emphasis on unit tests (70/20/10 split)
**Rationale**:
- Unit tests provide fast feedback and high coverage
- Integration tests catch service boundary issues
- E2E tests validate critical user journeys
- Balanced approach between speed and confidence

**Trade-offs**:
- Heavy unit testing requires good mocking practices
- Integration tests need database setup overhead
- E2E tests are slower and more flaky
- Maintenance overhead increases with more test types

### Deployment Pipeline Complexity
**Decision**: Multi-stage pipeline with automated testing and manual production approval
**Rationale**:
- Automated testing catches regressions early
- Staging environment validates integration
- Manual approval gate for production safety
- Rollback capabilities for quick recovery

**Trade-offs**:
- Complex pipeline increases maintenance burden
- Multiple environments increase infrastructure costs
- Manual approval can slow emergency deployments
- Pipeline failures can block all deployments

### Documentation Requirements
**Decision**: Require documentation updates for all public API changes
**Rationale**:
- API documentation prevents integration issues
- Code comments aid future maintenance
- Architecture decisions preserve institutional knowledge
- User guides reduce support burden

**Trade-offs**:
- Documentation overhead slows feature development
- Keeping docs in sync with code requires discipline
- Over-documentation can become noise
- Multiple doc formats increase maintenance complexity

---

This completes the comprehensive Substrate Methodology template with all core domains documented: architecture, authentication, API, database, and development guidelines. The documentation provides a complete foundation for any software project implementing the "Documentation as Code as Context" approach.