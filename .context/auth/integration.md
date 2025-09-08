# Authentication: Framework Integration

This document details the integration of authentication and authorization into [Your Project Name]'s HTTP framework, middleware chain, and service layer.

## HTTP Middleware Chain

### Middleware Order
The authentication middleware chain follows a specific order to ensure security and performance:

```go
// Router setup with middleware chain
func setupRouter(authService AuthService) *mux.Router {
    router := mux.NewRouter()
    
    // Apply middleware in order of execution
    router.Use(loggingMiddleware)           // 1. Request logging
    router.Use(corsMiddleware)              // 2. CORS headers
    router.Use(rateLimitMiddleware)         // 3. Rate limiting
    router.Use(timeoutMiddleware)           // 4. Request timeout
    
    // Public routes (no auth required)
    publicRouter := router.PathPrefix("/api/v1").Subrouter()
    publicRouter.HandleFunc("/auth/login", handleLogin).Methods("POST")
    publicRouter.HandleFunc("/auth/register", handleRegister).Methods("POST")
    publicRouter.HandleFunc("/auth/refresh", handleRefreshToken).Methods("POST")
    publicRouter.HandleFunc("/health", handleHealth).Methods("GET")
    
    // Protected routes (auth required)
    protectedRouter := router.PathPrefix("/api/v1").Subrouter()
    protectedRouter.Use(authMiddleware(authService))    // 5. Authentication
    protectedRouter.Use(userContextMiddleware)          // 6. User context enrichment
    
    // Role-based routes
    adminRouter := protectedRouter.PathPrefix("/admin").Subrouter()
    adminRouter.Use(requireRole(RoleAdmin))             // 7. Role-based access
    
    return router
}
```

### Core Authentication Middleware
```go
func authMiddleware(authService AuthService) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Skip auth for preflight requests
            if r.Method == "OPTIONS" {
                next.ServeHTTP(w, r)
                return
            }
            
            // Extract token from Authorization header
            token, err := extractBearerToken(r)
            if err != nil {
                writeAuthError(w, "missing_or_invalid_token", "Authorization token is required")
                return
            }
            
            // Validate access token
            claims, err := authService.ValidateAccessToken(r.Context(), token)
            if err != nil {
                var authErr *AuthError
                if errors.As(err, &authErr) {
                    writeAuthError(w, authErr.Code, authErr.Message)
                } else {
                    writeAuthError(w, "token_validation_failed", "Token validation failed")
                }
                return
            }
            
            // Add user context to request
            userCtx := &UserContext{
                UserID:      claims.UserID,
                Email:       claims.Email,
                Roles:       claims.Roles,
                Permissions: claims.Permissions,
                TokenID:     claims.ID,
            }
            
            ctx := context.WithValue(r.Context(), UserContextKey, userCtx)
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}

func extractBearerToken(r *http.Request) (string, error) {
    authHeader := r.Header.Get("Authorization")
    if authHeader == "" {
        return "", errors.New("missing authorization header")
    }
    
    parts := strings.Fields(authHeader)
    if len(parts) != 2 || !strings.EqualFold(parts[0], "bearer") {
        return "", errors.New("invalid authorization header format")
    }
    
    return parts[1], nil
}
```

### Permission-Based Middleware
```go
func requirePermission(permission string) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            userCtx, ok := r.Context().Value(UserContextKey).(*UserContext)
            if !ok {
                writeAuthError(w, "missing_user_context", "User context not found")
                return
            }
            
            if !userCtx.HasPermission(permission) {
                writeAuthError(w, "insufficient_permissions", "Insufficient permissions for this action")
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}

func requireRole(role string) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            userCtx, ok := r.Context().Value(UserContextKey).(*UserContext)
            if !ok {
                writeAuthError(w, "missing_user_context", "User context not found")
                return
            }
            
            if !userCtx.HasRole(role) {
                writeAuthError(w, "insufficient_role", fmt.Sprintf("Role '%s' required", role))
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}
```

## Authentication Handlers

### Login Handler
```go
type LoginRequest struct {
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
}

type LoginResponse struct {
    AccessToken  string `json:"access_token"`
    TokenType    string `json:"token_type"`
    ExpiresIn    int    `json:"expires_in"`
    RefreshToken string `json:"refresh_token,omitempty"`
    User         *User  `json:"user"`
}

func (h *AuthHandler) HandleLogin(w http.ResponseWriter, r *http.Request) {
    var req LoginRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        h.writeError(w, &ValidationError{
            Code:    "invalid_request_body",
            Message: "Invalid JSON in request body",
        })
        return
    }
    
    if err := h.validator.Validate(req); err != nil {
        h.writeError(w, &ValidationError{
            Code:    "validation_failed",
            Message: "Request validation failed",
            Details: err,
        })
        return
    }
    
    // Authenticate user
    tokenPair, user, err := h.authService.Authenticate(r.Context(), req.Email, req.Password)
    if err != nil {
        var authErr *AuthError
        if errors.As(err, &authErr) {
            h.writeError(w, authErr)
        } else {
            h.logger.Error("Authentication failed", "error", err, "email", req.Email)
            h.writeError(w, &AuthError{
                Code:    "authentication_failed",
                Message: "Authentication failed",
            })
        }
        return
    }
    
    // Set refresh token as HTTP-only cookie
    http.SetCookie(w, &http.Cookie{
        Name:     "refresh_token",
        Value:    tokenPair.RefreshToken,
        HttpOnly: true,
        Secure:   h.config.HTTPS,
        SameSite: http.SameSiteStrictMode,
        Expires:  time.Now().Add(7 * 24 * time.Hour),
        Path:     "/api/v1/auth",
    })
    
    response := LoginResponse{
        AccessToken:  tokenPair.AccessToken,
        TokenType:    "Bearer",
        ExpiresIn:    int(h.config.AccessTokenTTL.Seconds()),
        RefreshToken: tokenPair.RefreshToken, // Also in response for mobile clients
        User:         user,
    }
    
    h.writeJSON(w, http.StatusOK, response)
    h.logger.Info("User authenticated successfully", "user_id", user.ID, "email", user.Email)
}
```

### Refresh Token Handler
```go
func (h *AuthHandler) HandleRefreshToken(w http.ResponseWriter, r *http.Request) {
    // Try to get refresh token from cookie first
    refreshToken := ""
    if cookie, err := r.Cookie("refresh_token"); err == nil {
        refreshToken = cookie.Value
    }
    
    // Fallback to request body for mobile clients
    if refreshToken == "" {
        var req struct {
            RefreshToken string `json:"refresh_token"`
        }
        if err := json.NewDecoder(r.Body).Decode(&req); err == nil {
            refreshToken = req.RefreshToken
        }
    }
    
    if refreshToken == "" {
        h.writeError(w, &AuthError{
            Code:    "missing_refresh_token",
            Message: "Refresh token is required",
        })
        return
    }
    
    // Generate new token pair
    newTokenPair, err := h.authService.RefreshTokens(r.Context(), refreshToken)
    if err != nil {
        var authErr *AuthError
        if errors.As(err, &authErr) {
            h.writeError(w, authErr)
        } else {
            h.logger.Error("Token refresh failed", "error", err)
            h.writeError(w, &AuthError{
                Code:    "token_refresh_failed",
                Message: "Failed to refresh token",
            })
        }
        return
    }
    
    // Update refresh token cookie
    http.SetCookie(w, &http.Cookie{
        Name:     "refresh_token",
        Value:    newTokenPair.RefreshToken,
        HttpOnly: true,
        Secure:   h.config.HTTPS,
        SameSite: http.SameSiteStrictMode,
        Expires:  time.Now().Add(7 * 24 * time.Hour),
        Path:     "/api/v1/auth",
    })
    
    response := struct {
        AccessToken string `json:"access_token"`
        TokenType   string `json:"token_type"`
        ExpiresIn   int    `json:"expires_in"`
    }{
        AccessToken: newTokenPair.AccessToken,
        TokenType:   "Bearer",
        ExpiresIn:   int(h.config.AccessTokenTTL.Seconds()),
    }
    
    h.writeJSON(w, http.StatusOK, response)
}
```

### Logout Handler
```go
func (h *AuthHandler) HandleLogout(w http.ResponseWriter, r *http.Request) {
    userCtx, ok := r.Context().Value(UserContextKey).(*UserContext)
    if !ok {
        // User not authenticated, but still clear any cookies
        h.clearAuthCookies(w)
        h.writeJSON(w, http.StatusOK, map[string]string{"message": "Logged out successfully"})
        return
    }
    
    // Get refresh token for revocation
    refreshToken := ""
    if cookie, err := r.Cookie("refresh_token"); err == nil {
        refreshToken = cookie.Value
    }
    
    // Revoke tokens
    if err := h.authService.RevokeTokens(r.Context(), userCtx.TokenID, refreshToken); err != nil {
        h.logger.Error("Failed to revoke tokens", "error", err, "user_id", userCtx.UserID)
        // Continue with logout even if revocation fails
    }
    
    // Clear authentication cookies
    h.clearAuthCookies(w)
    
    h.writeJSON(w, http.StatusOK, map[string]string{"message": "Logged out successfully"})
    h.logger.Info("User logged out", "user_id", userCtx.UserID)
}

func (h *AuthHandler) clearAuthCookies(w http.ResponseWriter) {
    http.SetCookie(w, &http.Cookie{
        Name:     "refresh_token",
        Value:    "",
        HttpOnly: true,
        Secure:   h.config.HTTPS,
        SameSite: http.SameSiteStrictMode,
        Expires:  time.Unix(0, 0),
        Path:     "/api/v1/auth",
    })
}
```

## Service Integration

### User Context Helper
```go
type UserContext struct {
    UserID      string    `json:"user_id"`
    Email       string    `json:"email"`
    Roles       []string  `json:"roles"`
    Permissions []string  `json:"permissions"`
    TokenID     string    `json:"token_id"`
}

func (uc *UserContext) HasRole(role string) bool {
    for _, r := range uc.Roles {
        if r == role {
            return true
        }
    }
    return false
}

func (uc *UserContext) HasPermission(permission string) bool {
    for _, p := range uc.Permissions {
        if p == permission {
            return true
        }
    }
    return false
}

func (uc *UserContext) HasAnyRole(roles ...string) bool {
    for _, role := range roles {
        if uc.HasRole(role) {
            return true
        }
    }
    return false
}

func (uc *UserContext) HasAnyPermission(permissions ...string) bool {
    for _, permission := range permissions {
        if uc.HasPermission(permission) {
            return true
        }
    }
    return false
}

// Helper function to extract user context from request context
func GetUserContext(ctx context.Context) (*UserContext, bool) {
    userCtx, ok := ctx.Value(UserContextKey).(*UserContext)
    return userCtx, ok
}
```

### Service Layer Integration
```go
// Example service method using user context
func (s *userService) UpdateProfile(ctx context.Context, req UpdateProfileRequest) (*User, error) {
    userCtx, ok := GetUserContext(ctx)
    if !ok {
        return nil, ErrUnauthorized
    }
    
    // Users can only update their own profile unless they're admin
    if req.UserID != userCtx.UserID && !userCtx.HasRole(RoleAdmin) {
        return nil, ErrForbidden
    }
    
    // Validate request
    if err := s.validator.ValidateUpdateProfile(req); err != nil {
        return nil, err
    }
    
    // Update user
    user, err := s.repo.GetByID(ctx, req.UserID)
    if err != nil {
        return nil, err
    }
    
    user.Name = req.Name
    user.UpdatedAt = time.Now()
    
    return s.repo.Update(ctx, user)
}
```

## Error Handling

### Authentication Errors
```go
type AuthError struct {
    Code       string `json:"code"`
    Message    string `json:"message"`
    StatusCode int    `json:"-"`
}

func (e *AuthError) Error() string {
    return e.Message
}

var (
    ErrInvalidCredentials = &AuthError{
        Code:       "invalid_credentials",
        Message:    "Invalid email or password",
        StatusCode: http.StatusUnauthorized,
    }
    
    ErrTokenExpired = &AuthError{
        Code:       "token_expired",
        Message:    "Access token has expired",
        StatusCode: http.StatusUnauthorized,
    }
    
    ErrTokenInvalid = &AuthError{
        Code:       "token_invalid",
        Message:    "Access token is invalid",
        StatusCode: http.StatusUnauthorized,
    }
    
    ErrInsufficientPermissions = &AuthError{
        Code:       "insufficient_permissions",
        Message:    "Insufficient permissions for this action",
        StatusCode: http.StatusForbidden,
    }
)

func writeAuthError(w http.ResponseWriter, code, message string) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusUnauthorized)
    
    errorResponse := map[string]interface{}{
        "error": map[string]string{
            "code":    code,
            "message": message,
        },
    }
    
    json.NewEncoder(w).Encode(errorResponse)
}
```

## Decision History & Trade-offs

### Middleware Chain Order
**Decision**: Apply authentication middleware after rate limiting but before authorization
**Rationale**:
- Rate limiting protects against brute force attacks before authentication
- Authentication establishes user context for subsequent middleware
- Authorization middleware depends on user context from authentication
- Timeout middleware ensures requests don't hang during authentication

**Trade-offs**:
- Unauthenticated requests still consume authentication processing time
- Complex middleware chain requires careful testing
- Order dependencies make middleware less modular

### Cookie + Header Token Strategy
**Decision**: Support both HTTP-only cookies and Authorization headers for refresh tokens
**Rationale**:
- Cookies provide security against XSS attacks for web applications
- Authorization headers support mobile and API clients
- Fallback mechanism provides flexibility for different client types
- Refresh tokens have longer lifetime and need secure storage

**Trade-offs**:
- Increased complexity in token handling logic
- Potential security issues if not implemented correctly
- CORS complications with cookies
- Duplicate token storage on client side

### User Context Enrichment
**Decision**: Add user context to request context after authentication
**Rationale**:
- Provides consistent access to user information across handlers
- Eliminates need to re-parse tokens in business logic
- Enables fine-grained permission checking
- Simplifies testing with context injection

**Trade-offs**:
- Context values are not type-safe without careful helper functions
- Additional memory overhead for each request
- Potential for context value conflicts
- Testing complexity with context setup

---

**Next**: Review [security.md](./security.md) for comprehensive security measures and threat mitigation strategies.