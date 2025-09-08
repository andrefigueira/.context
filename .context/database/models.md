# Database: Data Models

This document defines the data model structures for [Your Project Name], including struct definitions, validation rules, serialization formats, and repository patterns for database interaction.

## Core Data Models

### User Model
Central user entity with comprehensive validation and serialization.

```go
type User struct {
    ID              string     `json:"id" db:"id"`
    Email           string     `json:"email" db:"email" validate:"required,email"`
    Name            string     `json:"name" db:"name" validate:"required,min=1,max=255"`
    PasswordHash    string     `json:"-" db:"password_hash"`
    Status          UserStatus `json:"status" db:"status"`
    EmailVerified   bool       `json:"email_verified" db:"email_verified"`
    EmailVerifiedAt *time.Time `json:"email_verified_at,omitempty" db:"email_verified_at"`
    CreatedAt       time.Time  `json:"created_at" db:"created_at"`
    UpdatedAt       time.Time  `json:"updated_at" db:"updated_at"`
    DeletedAt       *time.Time `json:"deleted_at,omitempty" db:"deleted_at"`
    
    // Associations (loaded separately)
    Profile     *UserProfile `json:"profile,omitempty" db:"-"`
    Roles       []Role       `json:"roles,omitempty" db:"-"`
    Permissions []string     `json:"permissions,omitempty" db:"-"`
}

type UserStatus string

const (
    UserStatusActive    UserStatus = "active"
    UserStatusInactive  UserStatus = "inactive"
    UserStatusSuspended UserStatus = "suspended"
)

// Validation
func (u *User) Validate() error {
    validate := validator.New()
    
    if err := validate.Struct(u); err != nil {
        return fmt.Errorf("user validation failed: %w", err)
    }
    
    // Custom validations
    if len(u.Email) > 255 {
        return errors.New("email too long")
    }
    
    if u.Status != UserStatusActive && u.Status != UserStatusInactive && u.Status != UserStatusSuspended {
        return errors.New("invalid user status")
    }
    
    return nil
}

// Serialization methods
func (u *User) MarshalJSON() ([]byte, error) {
    type Alias User
    return json.Marshal(&struct {
        *Alias
        PasswordHash string `json:"-"` // Never serialize password hash
    }{
        Alias: (*Alias)(u),
    })
}

// Helper methods
func (u *User) IsActive() bool {
    return u.Status == UserStatusActive && u.DeletedAt == nil
}

func (u *User) HasRole(roleName string) bool {
    for _, role := range u.Roles {
        if role.Name == roleName {
            return true
        }
    }
    return false
}

func (u *User) HasPermission(permission string) bool {
    for _, perm := range u.Permissions {
        if perm == permission {
            return true
        }
    }
    return false
}

func (u *User) SetPassword(password string) error {
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        return fmt.Errorf("hash password: %w", err)
    }
    u.PasswordHash = string(hash)
    return nil
}

func (u *User) CheckPassword(password string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(u.PasswordHash), []byte(password))
    return err == nil
}
```

### User Profile Model
Extended user information with flexible preferences.

```go
type UserProfile struct {
    UserID      string                 `json:"user_id" db:"user_id"`
    Bio         *string                `json:"bio,omitempty" db:"bio"`
    Location    *string                `json:"location,omitempty" db:"location"`
    Website     *string                `json:"website,omitempty" db:"website" validate:"omitempty,url"`
    AvatarURL   *string                `json:"avatar_url,omitempty" db:"avatar_url" validate:"omitempty,url"`
    DateOfBirth *time.Time             `json:"date_of_birth,omitempty" db:"date_of_birth"`
    Phone       *string                `json:"phone,omitempty" db:"phone" validate:"omitempty,e164"`
    Timezone    string                 `json:"timezone" db:"timezone"`
    Locale      string                 `json:"locale" db:"locale"`
    Preferences map[string]interface{} `json:"preferences" db:"preferences"`
    CreatedAt   time.Time              `json:"created_at" db:"created_at"`
    UpdatedAt   time.Time              `json:"updated_at" db:"updated_at"`
}

// Preferences helper methods
func (p *UserProfile) GetPreference(key string, defaultValue interface{}) interface{} {
    if value, exists := p.Preferences[key]; exists {
        return value
    }
    return defaultValue
}

func (p *UserProfile) SetPreference(key string, value interface{}) {
    if p.Preferences == nil {
        p.Preferences = make(map[string]interface{})
    }
    p.Preferences[key] = value
}

func (p *UserProfile) GetStringPreference(key string, defaultValue string) string {
    value := p.GetPreference(key, defaultValue)
    if str, ok := value.(string); ok {
        return str
    }
    return defaultValue
}

func (p *UserProfile) GetBoolPreference(key string, defaultValue bool) bool {
    value := p.GetPreference(key, defaultValue)
    if b, ok := value.(bool); ok {
        return b
    }
    return defaultValue
}

func (p *UserProfile) Validate() error {
    validate := validator.New()
    
    if err := validate.Struct(p); err != nil {
        return fmt.Errorf("user profile validation failed: %w", err)
    }
    
    // Custom validations
    if p.Bio != nil && len(*p.Bio) > 1000 {
        return errors.New("bio too long (max 1000 characters)")
    }
    
    if p.DateOfBirth != nil {
        now := time.Now()
        if p.DateOfBirth.After(now) {
            return errors.New("date of birth cannot be in the future")
        }
        
        age := now.Sub(*p.DateOfBirth).Hours() / (24 * 365)
        if age > 150 {
            return errors.New("invalid date of birth")
        }
    }
    
    return nil
}
```

### Role and Permission Models

```go
type Role struct {
    ID          string       `json:"id" db:"id"`
    Name        string       `json:"name" db:"name" validate:"required,min=1,max=50"`
    Description *string      `json:"description,omitempty" db:"description"`
    CreatedAt   time.Time    `json:"created_at" db:"created_at"`
    
    // Associations
    Permissions []Permission `json:"permissions,omitempty" db:"-"`
}

type Permission struct {
    ID          string    `json:"id" db:"id"`
    Name        string    `json:"name" db:"name" validate:"required"`
    Description *string   `json:"description,omitempty" db:"description"`
    Resource    string    `json:"resource" db:"resource" validate:"required"`
    Action      string    `json:"action" db:"action" validate:"required"`
    CreatedAt   time.Time `json:"created_at" db:"created_at"`
}

type UserRole struct {
    UserID    string     `json:"user_id" db:"user_id"`
    RoleID    string     `json:"role_id" db:"role_id"`
    GrantedBy *string    `json:"granted_by,omitempty" db:"granted_by"`
    GrantedAt time.Time  `json:"granted_at" db:"granted_at"`
    ExpiresAt *time.Time `json:"expires_at,omitempty" db:"expires_at"`
}

// Helper methods
func (r *Role) HasPermission(permissionName string) bool {
    for _, perm := range r.Permissions {
        if perm.Name == permissionName {
            return true
        }
    }
    return false
}

func (ur *UserRole) IsExpired() bool {
    return ur.ExpiresAt != nil && ur.ExpiresAt.Before(time.Now())
}

func (ur *UserRole) IsValid() bool {
    return !ur.IsExpired()
}
```

### Authentication Token Models

```go
type RefreshToken struct {
    ID          string     `json:"id" db:"id"`
    UserID      string     `json:"user_id" db:"user_id"`
    TokenHash   string     `json:"-" db:"token_hash"`
    ExpiresAt   time.Time  `json:"expires_at" db:"expires_at"`
    CreatedAt   time.Time  `json:"created_at" db:"created_at"`
    RevokedAt   *time.Time `json:"revoked_at,omitempty" db:"revoked_at"`
    ReplacedBy  *string    `json:"replaced_by,omitempty" db:"replaced_by"`
}

func (rt *RefreshToken) IsValid() bool {
    return rt.RevokedAt == nil && rt.ExpiresAt.After(time.Now())
}

func (rt *RefreshToken) IsExpired() bool {
    return rt.ExpiresAt.Before(time.Now())
}

func (rt *RefreshToken) IsRevoked() bool {
    return rt.RevokedAt != nil
}

func (rt *RefreshToken) Revoke() {
    now := time.Now()
    rt.RevokedAt = &now
}

type TokenBlacklist struct {
    TokenID   string    `json:"token_id" db:"token_id"`
    ExpiresAt time.Time `json:"expires_at" db:"expires_at"`
    RevokedAt time.Time `json:"revoked_at" db:"revoked_at"`
    Reason    *string   `json:"reason,omitempty" db:"reason"`
}

func (tb *TokenBlacklist) IsExpired() bool {
    return tb.ExpiresAt.Before(time.Now())
}
```

## Request/Response Models

### Authentication DTOs

```go
// Registration request
type CreateUserRequest struct {
    Email    string `json:"email" validate:"required,email,max=255"`
    Name     string `json:"name" validate:"required,min=1,max=255"`
    Password string `json:"password" validate:"required,min=12,max=128"`
}

func (r *CreateUserRequest) Validate() error {
    validate := validator.New()
    
    if err := validate.Struct(r); err != nil {
        return fmt.Errorf("create user request validation failed: %w", err)
    }
    
    // Password strength validation
    if !isStrongPassword(r.Password) {
        return errors.New("password does not meet strength requirements")
    }
    
    return nil
}

func (r *CreateUserRequest) ToUser() *User {
    return &User{
        ID:            generateUUID(),
        Email:         strings.ToLower(strings.TrimSpace(r.Email)),
        Name:          strings.TrimSpace(r.Name),
        Status:        UserStatusActive,
        EmailVerified: false,
        CreatedAt:     time.Now(),
        UpdatedAt:     time.Now(),
    }
}

// Login request
type LoginRequest struct {
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required"`
}

// Login response
type LoginResponse struct {
    AccessToken  string `json:"access_token"`
    TokenType    string `json:"token_type"`
    ExpiresIn    int    `json:"expires_in"`
    RefreshToken string `json:"refresh_token,omitempty"`
    User         *User  `json:"user"`
}

// Profile update request
type UpdateProfileRequest struct {
    Name    *string                `json:"name,omitempty" validate:"omitempty,min=1,max=255"`
    Profile *UpdateUserProfileData `json:"profile,omitempty"`
}

type UpdateUserProfileData struct {
    Bio         *string                `json:"bio,omitempty" validate:"omitempty,max=1000"`
    Location    *string                `json:"location,omitempty" validate:"omitempty,max=255"`
    Website     *string                `json:"website,omitempty" validate:"omitempty,url"`
    Phone       *string                `json:"phone,omitempty" validate:"omitempty,e164"`
    Timezone    *string                `json:"timezone,omitempty"`
    Locale      *string                `json:"locale,omitempty"`
    Preferences map[string]interface{} `json:"preferences,omitempty"`
}

// Password change request
type ChangePasswordRequest struct {
    CurrentPassword string `json:"current_password" validate:"required"`
    NewPassword     string `json:"new_password" validate:"required,min=12,max=128"`
}

func (r *ChangePasswordRequest) Validate() error {
    validate := validator.New()
    
    if err := validate.Struct(r); err != nil {
        return fmt.Errorf("change password request validation failed: %w", err)
    }
    
    if r.CurrentPassword == r.NewPassword {
        return errors.New("new password must be different from current password")
    }
    
    if !isStrongPassword(r.NewPassword) {
        return errors.New("new password does not meet strength requirements")
    }
    
    return nil
}
```

### Admin DTOs

```go
// User list request with filtering
type ListUsersRequest struct {
    Page   int         `json:"page" validate:"min=1"`
    Limit  int         `json:"limit" validate:"min=1,max=100"`
    Search string      `json:"search,omitempty" validate:"max=255"`
    Role   string      `json:"role,omitempty"`
    Status UserStatus  `json:"status,omitempty"`
    Sort   string      `json:"sort,omitempty" validate:"oneof=name email created_at updated_at"`
    Order  string      `json:"order,omitempty" validate:"oneof=asc desc"`
}

func (r *ListUsersRequest) SetDefaults() {
    if r.Page == 0 {
        r.Page = 1
    }
    if r.Limit == 0 {
        r.Limit = 20
    }
    if r.Sort == "" {
        r.Sort = "created_at"
    }
    if r.Order == "" {
        r.Order = "desc"
    }
}

// Paginated response
type PaginatedUsersResponse struct {
    Users      []*User    `json:"users"`
    Pagination Pagination `json:"pagination"`
}

type Pagination struct {
    CurrentPage int    `json:"current_page"`
    PerPage     int    `json:"per_page"`
    TotalPages  int    `json:"total_pages"`
    TotalCount  int64  `json:"total_count"`
    HasNext     bool   `json:"has_next"`
    HasPrev     bool   `json:"has_prev"`
    NextCursor  string `json:"next_cursor,omitempty"`
    PrevCursor  string `json:"prev_cursor,omitempty"`
}

// Update user roles request
type UpdateUserRolesRequest struct {
    Roles []string `json:"roles" validate:"required,min=1"`
}

// Update user status request
type UpdateUserStatusRequest struct {
    Status UserStatus `json:"status" validate:"required"`
    Reason string     `json:"reason,omitempty" validate:"max=500"`
}
```

## Audit and Logging Models

```go
type AuditLog struct {
    ID           string                 `json:"id" db:"id"`
    UserID       *string                `json:"user_id,omitempty" db:"user_id"`
    Action       string                 `json:"action" db:"action"`
    ResourceType string                 `json:"resource_type" db:"resource_type"`
    ResourceID   *string                `json:"resource_id,omitempty" db:"resource_id"`
    OldValues    map[string]interface{} `json:"old_values,omitempty" db:"old_values"`
    NewValues    map[string]interface{} `json:"new_values,omitempty" db:"new_values"`
    IPAddress    *string                `json:"ip_address,omitempty" db:"ip_address"`
    UserAgent    *string                `json:"user_agent,omitempty" db:"user_agent"`
    Success      bool                   `json:"success" db:"success"`
    ErrorMessage *string                `json:"error_message,omitempty" db:"error_message"`
    Metadata     map[string]interface{} `json:"metadata,omitempty" db:"metadata"`
    CreatedAt    time.Time              `json:"created_at" db:"created_at"`
}

// Factory method for creating audit logs
func NewAuditLog(userID *string, action, resourceType string, resourceID *string) *AuditLog {
    return &AuditLog{
        ID:           generateUUID(),
        UserID:       userID,
        Action:       action,
        ResourceType: resourceType,
        ResourceID:   resourceID,
        Success:      true,
        Metadata:     make(map[string]interface{}),
        CreatedAt:    time.Now(),
    }
}

func (al *AuditLog) WithIPAddress(ip string) *AuditLog {
    al.IPAddress = &ip
    return al
}

func (al *AuditLog) WithUserAgent(ua string) *AuditLog {
    al.UserAgent = &ua
    return al
}

func (al *AuditLog) WithOldValues(values map[string]interface{}) *AuditLog {
    al.OldValues = values
    return al
}

func (al *AuditLog) WithNewValues(values map[string]interface{}) *AuditLog {
    al.NewValues = values
    return al
}

func (al *AuditLog) WithError(err error) *AuditLog {
    al.Success = false
    errMsg := err.Error()
    al.ErrorMessage = &errMsg
    return al
}

func (al *AuditLog) WithMetadata(key string, value interface{}) *AuditLog {
    if al.Metadata == nil {
        al.Metadata = make(map[string]interface{})
    }
    al.Metadata[key] = value
    return al
}
```

## Model Validation and Helpers

### Validation Functions

```go
func isStrongPassword(password string) bool {
    if len(password) < 12 {
        return false
    }
    
    checks := []struct {
        pattern *regexp.Regexp
        name    string
    }{
        {regexp.MustCompile(`[A-Z]`), "uppercase"},
        {regexp.MustCompile(`[a-z]`), "lowercase"},
        {regexp.MustCompile(`[0-9]`), "number"},
        {regexp.MustCompile(`[^A-Za-z0-9]`), "special"},
    }
    
    for _, check := range checks {
        if !check.pattern.MatchString(password) {
            return false
        }
    }
    
    return true
}

func validateTimezone(timezone string) bool {
    _, err := time.LoadLocation(timezone)
    return err == nil
}

func validateLocale(locale string) bool {
    validLocales := map[string]bool{
        "en": true, "en-US": true, "en-GB": true,
        "es": true, "es-ES": true, "es-MX": true,
        "fr": true, "fr-FR": true, "fr-CA": true,
        "de": true, "de-DE": true, "de-AT": true,
        "it": true, "it-IT": true,
        "pt": true, "pt-BR": true, "pt-PT": true,
        "zh": true, "zh-CN": true, "zh-TW": true,
        "ja": true, "ja-JP": true,
        "ko": true, "ko-KR": true,
        "ru": true, "ru-RU": true,
        "ar": true, "ar-SA": true,
    }
    
    return validLocales[locale]
}
```

### Utility Functions

```go
func generateUUID() string {
    return uuid.New().String()
}

func sanitizeEmail(email string) string {
    return strings.ToLower(strings.TrimSpace(email))
}

func sanitizeName(name string) string {
    // Remove extra whitespace and trim
    name = strings.TrimSpace(name)
    name = regexp.MustCompile(`\s+`).ReplaceAllString(name, " ")
    
    // Remove potentially dangerous characters
    name = regexp.MustCompile(`[<>\"'&]`).ReplaceAllString(name, "")
    
    return name
}

func maskEmail(email string) string {
    parts := strings.Split(email, "@")
    if len(parts) != 2 {
        return "***"
    }
    
    username := parts[0]
    domain := parts[1]
    
    if len(username) <= 2 {
        return "***@" + domain
    }
    
    masked := username[:1] + strings.Repeat("*", len(username)-2) + username[len(username)-1:]
    return masked + "@" + domain
}

func truncateString(s string, maxLength int) string {
    if len(s) <= maxLength {
        return s
    }
    return s[:maxLength-3] + "..."
}
```

## Decision History & Trade-offs

### JSON Tags vs Database Tags
**Decision**: Use separate `json` and `db` struct tags instead of single tag
**Rationale**:
- Clear separation between API serialization and database mapping
- Different naming conventions (camelCase vs snake_case)
- Ability to hide database fields from JSON (like password_hash)
- Better maintainability when API and DB schemas differ

**Trade-offs**:
- More verbose struct definitions
- Potential for inconsistency between tag types
- Additional maintenance when field names change
- Risk of forgetting to update both tag types

### Pointer Fields for Optional Data
**Decision**: Use pointers for optional fields instead of default values
**Rationale**:
- Clear distinction between nil (not set) and zero value (explicitly set)
- Better JSON marshaling behavior (omitempty works correctly)
- Database NULL handling is explicit
- Prevents accidental overwrites of existing data

**Trade-offs**:
- More complex code with nil checking
- Potential for nil pointer panics if not handled carefully
- Additional memory overhead for pointer storage
- More verbose code when accessing optional fields

### Embedded vs Separate Validation
**Decision**: Use struct tags with go-validator instead of custom validation methods
**Rationale**:
- Declarative validation rules are easier to understand
- Standard library with good community support
- Consistent validation patterns across the application
- Automatic validation error formatting

**Trade-offs**:
- Limited flexibility for complex business rule validation
- Struct tag validation is checked at runtime, not compile time
- Custom validation functions still needed for business logic
- Potential performance overhead for reflection-based validation

### Model Factory Methods
**Decision**: Provide factory methods for creating models with proper defaults
**Rationale**:
- Ensures models are created with proper initial state
- Reduces boilerplate in application code
- Centralizes default value logic
- Makes testing easier with consistent model creation

**Trade-offs**:
- Additional methods to maintain for each model
- Risk of factory methods becoming out of sync with struct changes
- Potential confusion about when to use factory vs direct instantiation
- More complex when models have many optional parameters

---

**Next**: Review [migrations.md](./migrations.md) for schema change management and migration strategies.