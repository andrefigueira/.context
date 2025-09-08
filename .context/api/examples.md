# API: Usage Examples

This document provides comprehensive examples of [Your Project Name] API usage, including authentication flows, common operations, error handling, and integration patterns for various client types.

## Authentication Flow Examples

### Complete Registration and Login Flow

#### 1. User Registration
```bash
# Register new user
curl -X POST https://api.yourproject.com/v1/auth/register \
  -H "Content-Type: application/json" \
  -H "X-Request-ID: 550e8400-e29b-41d4-a716-446655440000" \
  -d '{
    "email": "john.doe@example.com",
    "password": "SecurePassword123!",
    "name": "John Doe"
  }'
```

**Response**:
```json
{
  "id": "user_abc123",
  "email": "john.doe@example.com",
  "name": "John Doe",
  "roles": ["user"],
  "created_at": "2024-01-20T10:30:00Z",
  "updated_at": "2024-01-20T10:30:00Z"
}
```

#### 2. User Login
```bash
# Login to get access token
curl -X POST https://api.yourproject.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "SecurePassword123!"
  }'
```

**Response**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900,
  "refresh_token": "rt_def456ghi789...",
  "user": {
    "id": "user_abc123",
    "email": "john.doe@example.com",
    "name": "John Doe",
    "roles": ["user"]
  }
}
```

#### 3. Using Access Token
```bash
# Make authenticated request
curl -X GET https://api.yourproject.com/v1/users/profile \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "X-Request-ID: 550e8400-e29b-41d4-a716-446655440001"
```

#### 4. Token Refresh
```bash
# Refresh expired access token
curl -X POST https://api.yourproject.com/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "rt_def456ghi789..."
  }'
```

**Response**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

## User Management Examples

### Profile Management

#### Get User Profile
```bash
curl -X GET https://api.yourproject.com/v1/users/profile \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Update User Profile
```bash
curl -X PUT https://api.yourproject.com/v1/users/profile \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Smith",
    "profile": {
      "bio": "Senior Software Developer at TechCorp",
      "location": "Seattle, WA",
      "website": "https://johnsmith.dev"
    }
  }'
```

#### Change Password
```bash
curl -X POST https://api.yourproject.com/v1/users/change-password \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "current_password": "SecurePassword123!",
    "new_password": "NewSecurePassword456!"
  }'
```

### Admin Operations

#### List All Users (Admin)
```bash
curl -X GET "https://api.yourproject.com/v1/users?page=1&limit=20&search=john&role=user" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response**:
```json
{
  "users": [
    {
      "id": "user_abc123",
      "email": "john.doe@example.com",
      "name": "John Doe",
      "roles": ["user"],
      "status": "active",
      "created_at": "2024-01-15T10:30:00Z",
      "last_active": "2024-01-20T09:15:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_count": 89,
    "has_next": true,
    "has_prev": false,
    "next_cursor": "eyJpZCI6IjEyMyJ9",
    "prev_cursor": null
  }
}
```

#### Update User Roles
```bash
curl -X PUT https://api.yourproject.com/v1/users/user_abc123/roles \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "roles": ["user", "moderator"]
  }'
```

#### Suspend User Account
```bash
curl -X PUT https://api.yourproject.com/v1/users/user_abc123/status \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "status": "suspended",
    "reason": "Violation of terms of service"
  }'
```

## Error Handling Examples

### Validation Errors
```bash
# Invalid email format
curl -X POST https://api.yourproject.com/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "invalid-email",
    "password": "short",
    "name": ""
  }'
```

**Response** (400 Bad Request):
```json
{
  "error": {
    "code": "validation_failed",
    "message": "Request validation failed",
    "details": {
      "email": "Invalid email format",
      "password": "Password must be at least 12 characters",
      "name": "Name is required"
    },
    "trace_id": "abc123def456",
    "timestamp": "2024-01-20T10:30:00Z"
  }
}
```

### Authentication Errors
```bash
# Invalid credentials
curl -X POST https://api.yourproject.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "wrongpassword"
  }'
```

**Response** (401 Unauthorized):
```json
{
  "error": {
    "code": "invalid_credentials",
    "message": "Invalid email or password",
    "trace_id": "def456ghi789",
    "timestamp": "2024-01-20T10:30:00Z"
  }
}
```

### Rate Limiting
```bash
# Too many requests
curl -X POST https://api.yourproject.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "john.doe@example.com", "password": "password"}'
```

**Response** (429 Too Many Requests):
```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1642675800
Retry-After: 300

{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Too many login attempts. Try again in 5 minutes.",
    "retry_after": 300,
    "trace_id": "ghi789jkl012",
    "timestamp": "2024-01-20T10:30:00Z"
  }
}
```

## Client Implementation Examples

### JavaScript/TypeScript Client
```typescript
class APIClient {
  private baseURL: string;
  private accessToken: string | null = null;
  private refreshToken: string | null = null;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }

  async login(email: string, password: string): Promise<LoginResponse> {
    const response = await fetch(`${this.baseURL}/auth/login`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Request-ID': this.generateRequestId(),
      },
      body: JSON.stringify({ email, password }),
    });

    if (!response.ok) {
      throw await this.handleErrorResponse(response);
    }

    const data = await response.json();
    this.accessToken = data.access_token;
    this.refreshToken = data.refresh_token;
    
    return data;
  }

  async makeAuthenticatedRequest<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    if (!this.accessToken) {
      throw new Error('Not authenticated');
    }

    const response = await fetch(`${this.baseURL}${endpoint}`, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${this.accessToken}`,
        'X-Request-ID': this.generateRequestId(),
      },
    });

    if (response.status === 401) {
      // Token might be expired, try refresh
      await this.refreshAccessToken();
      return this.makeAuthenticatedRequest(endpoint, options);
    }

    if (!response.ok) {
      throw await this.handleErrorResponse(response);
    }

    return response.json();
  }

  private async refreshAccessToken(): Promise<void> {
    if (!this.refreshToken) {
      throw new Error('No refresh token available');
    }

    const response = await fetch(`${this.baseURL}/auth/refresh`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ refresh_token: this.refreshToken }),
    });

    if (!response.ok) {
      // Refresh failed, user needs to login again
      this.accessToken = null;
      this.refreshToken = null;
      throw new Error('Session expired, please login again');
    }

    const data = await response.json();
    this.accessToken = data.access_token;
  }

  private generateRequestId(): string {
    return crypto.randomUUID();
  }

  private async handleErrorResponse(response: Response): Promise<Error> {
    const errorData = await response.json();
    return new APIError(
      errorData.error.code,
      errorData.error.message,
      response.status,
      errorData.error.trace_id
    );
  }
}

// Usage example
const client = new APIClient('https://api.yourproject.com/v1');

try {
  await client.login('john.doe@example.com', 'password');
  const profile = await client.makeAuthenticatedRequest('/users/profile');
  console.log('User profile:', profile);
} catch (error) {
  console.error('API error:', error.message);
}
```

### Python Client
```python
import requests
import uuid
import time
from typing import Optional, Dict, Any

class APIClient:
    def __init__(self, base_url: str):
        self.base_url = base_url
        self.access_token: Optional[str] = None
        self.refresh_token: Optional[str] = None
        self.session = requests.Session()

    def login(self, email: str, password: str) -> Dict[str, Any]:
        response = self.session.post(
            f"{self.base_url}/auth/login",
            json={"email": email, "password": password},
            headers={"X-Request-ID": str(uuid.uuid4())}
        )
        
        response.raise_for_status()
        data = response.json()
        
        self.access_token = data["access_token"]
        self.refresh_token = data["refresh_token"]
        
        return data

    def make_authenticated_request(
        self, 
        method: str, 
        endpoint: str, 
        **kwargs
    ) -> Dict[str, Any]:
        if not self.access_token:
            raise ValueError("Not authenticated")

        headers = kwargs.get("headers", {})
        headers.update({
            "Authorization": f"Bearer {self.access_token}",
            "X-Request-ID": str(uuid.uuid4())
        })
        kwargs["headers"] = headers

        response = self.session.request(
            method,
            f"{self.base_url}{endpoint}",
            **kwargs
        )

        if response.status_code == 401:
            # Try to refresh token
            self._refresh_access_token()
            # Retry the original request
            headers["Authorization"] = f"Bearer {self.access_token}"
            response = self.session.request(
                method,
                f"{self.base_url}{endpoint}",
                **kwargs
            )

        response.raise_for_status()
        return response.json()

    def _refresh_access_token(self):
        if not self.refresh_token:
            raise ValueError("No refresh token available")

        response = self.session.post(
            f"{self.base_url}/auth/refresh",
            json={"refresh_token": self.refresh_token}
        )

        if response.status_code != 200:
            self.access_token = None
            self.refresh_token = None
            raise ValueError("Session expired, please login again")

        data = response.json()
        self.access_token = data["access_token"]

    def get_profile(self) -> Dict[str, Any]:
        return self.make_authenticated_request("GET", "/users/profile")

    def update_profile(self, profile_data: Dict[str, Any]) -> Dict[str, Any]:
        return self.make_authenticated_request(
            "PUT", 
            "/users/profile", 
            json=profile_data
        )

# Usage example
client = APIClient("https://api.yourproject.com/v1")

try:
    client.login("john.doe@example.com", "password")
    profile = client.get_profile()
    print(f"User profile: {profile}")
    
    # Update profile
    updated_profile = client.update_profile({
        "name": "John Smith",
        "profile": {"bio": "Updated bio"}
    })
    print(f"Updated profile: {updated_profile}")
    
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")
except ValueError as e:
    print(f"Authentication error: {e}")
```

### Go Client
```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "time"
    "github.com/google/uuid"
)

type APIClient struct {
    baseURL      string
    httpClient   *http.Client
    accessToken  string
    refreshToken string
}

type LoginRequest struct {
    Email    string `json:"email"`
    Password string `json:"password"`
}

type LoginResponse struct {
    AccessToken  string `json:"access_token"`
    RefreshToken string `json:"refresh_token"`
    TokenType    string `json:"token_type"`
    ExpiresIn    int    `json:"expires_in"`
    User         User   `json:"user"`
}

func NewAPIClient(baseURL string) *APIClient {
    return &APIClient{
        baseURL: baseURL,
        httpClient: &http.Client{
            Timeout: 30 * time.Second,
        },
    }
}

func (c *APIClient) Login(email, password string) (*LoginResponse, error) {
    reqBody := LoginRequest{
        Email:    email,
        Password: password,
    }
    
    reqJSON, err := json.Marshal(reqBody)
    if err != nil {
        return nil, fmt.Errorf("marshal request: %w", err)
    }
    
    req, err := http.NewRequest("POST", c.baseURL+"/auth/login", bytes.NewBuffer(reqJSON))
    if err != nil {
        return nil, fmt.Errorf("create request: %w", err)
    }
    
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("X-Request-ID", uuid.New().String())
    
    resp, err := c.httpClient.Do(req)
    if err != nil {
        return nil, fmt.Errorf("make request: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, c.handleErrorResponse(resp)
    }
    
    var loginResp LoginResponse
    if err := json.NewDecoder(resp.Body).Decode(&loginResp); err != nil {
        return nil, fmt.Errorf("decode response: %w", err)
    }
    
    c.accessToken = loginResp.AccessToken
    c.refreshToken = loginResp.RefreshToken
    
    return &loginResp, nil
}

func (c *APIClient) MakeAuthenticatedRequest(method, endpoint string, body interface{}) (*http.Response, error) {
    if c.accessToken == "" {
        return nil, fmt.Errorf("not authenticated")
    }
    
    var reqBody io.Reader
    if body != nil {
        bodyJSON, err := json.Marshal(body)
        if err != nil {
            return nil, fmt.Errorf("marshal request body: %w", err)
        }
        reqBody = bytes.NewBuffer(bodyJSON)
    }
    
    req, err := http.NewRequest(method, c.baseURL+endpoint, reqBody)
    if err != nil {
        return nil, fmt.Errorf("create request: %w", err)
    }
    
    req.Header.Set("Authorization", "Bearer "+c.accessToken)
    req.Header.Set("X-Request-ID", uuid.New().String())
    if body != nil {
        req.Header.Set("Content-Type", "application/json")
    }
    
    resp, err := c.httpClient.Do(req)
    if err != nil {
        return nil, fmt.Errorf("make request: %w", err)
    }
    
    if resp.StatusCode == http.StatusUnauthorized {
        resp.Body.Close()
        // Try to refresh token
        if err := c.refreshAccessToken(); err != nil {
            return nil, err
        }
        // Retry original request
        return c.MakeAuthenticatedRequest(method, endpoint, body)
    }
    
    return resp, nil
}

func (c *APIClient) refreshAccessToken() error {
    if c.refreshToken == "" {
        return fmt.Errorf("no refresh token available")
    }
    
    reqBody := map[string]string{
        "refresh_token": c.refreshToken,
    }
    
    reqJSON, _ := json.Marshal(reqBody)
    req, _ := http.NewRequest("POST", c.baseURL+"/auth/refresh", bytes.NewBuffer(reqJSON))
    req.Header.Set("Content-Type", "application/json")
    
    resp, err := c.httpClient.Do(req)
    if err != nil {
        return fmt.Errorf("refresh request failed: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        c.accessToken = ""
        c.refreshToken = ""
        return fmt.Errorf("session expired, please login again")
    }
    
    var refreshResp struct {
        AccessToken string `json:"access_token"`
    }
    
    if err := json.NewDecoder(resp.Body).Decode(&refreshResp); err != nil {
        return fmt.Errorf("decode refresh response: %w", err)
    }
    
    c.accessToken = refreshResp.AccessToken
    return nil
}

// Usage example
func main() {
    client := NewAPIClient("https://api.yourproject.com/v1")
    
    // Login
    loginResp, err := client.Login("john.doe@example.com", "password")
    if err != nil {
        log.Fatal("Login failed:", err)
    }
    
    fmt.Printf("Logged in as: %s\n", loginResp.User.Name)
    
    // Get profile
    resp, err := client.MakeAuthenticatedRequest("GET", "/users/profile", nil)
    if err != nil {
        log.Fatal("Get profile failed:", err)
    }
    defer resp.Body.Close()
    
    var profile User
    json.NewDecoder(resp.Body).Decode(&profile)
    fmt.Printf("Profile: %+v\n", profile)
}
```

## Testing Examples

### API Testing with curl
```bash
#!/bin/bash

# Test script for API endpoints
BASE_URL="https://api.yourproject.com/v1"
EMAIL="test@example.com"
PASSWORD="TestPassword123!"

echo "=== API Test Suite ==="

# Test 1: User Registration
echo "Testing user registration..."
REGISTER_RESPONSE=$(curl -s -X POST "$BASE_URL/auth/register" \
  -H "Content-Type: application/json" \
  -d "{\"email\":\"$EMAIL\",\"password\":\"$PASSWORD\",\"name\":\"Test User\"}")

if echo "$REGISTER_RESPONSE" | grep -q "id"; then
  echo "✅ Registration successful"
else
  echo "❌ Registration failed: $REGISTER_RESPONSE"
  exit 1
fi

# Test 2: User Login
echo "Testing user login..."
LOGIN_RESPONSE=$(curl -s -X POST "$BASE_URL/auth/login" \
  -H "Content-Type: application/json" \
  -d "{\"email\":\"$EMAIL\",\"password\":\"$PASSWORD\"}")

ACCESS_TOKEN=$(echo "$LOGIN_RESPONSE" | jq -r '.access_token')

if [ "$ACCESS_TOKEN" != "null" ] && [ "$ACCESS_TOKEN" != "" ]; then
  echo "✅ Login successful"
else
  echo "❌ Login failed: $LOGIN_RESPONSE"
  exit 1
fi

# Test 3: Get Profile
echo "Testing get profile..."
PROFILE_RESPONSE=$(curl -s -X GET "$BASE_URL/users/profile" \
  -H "Authorization: Bearer $ACCESS_TOKEN")

if echo "$PROFILE_RESPONSE" | grep -q "id"; then
  echo "✅ Get profile successful"
else
  echo "❌ Get profile failed: $PROFILE_RESPONSE"
fi

# Test 4: Update Profile
echo "Testing update profile..."
UPDATE_RESPONSE=$(curl -s -X PUT "$BASE_URL/users/profile" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated Test User","profile":{"bio":"Test bio"}}')

if echo "$UPDATE_RESPONSE" | grep -q "Updated Test User"; then
  echo "✅ Update profile successful"
else
  echo "❌ Update profile failed: $UPDATE_RESPONSE"
fi

echo "=== All tests completed ==="
```

This completes the API documentation domain with comprehensive examples for authentication, user management, error handling, and client implementations across multiple programming languages.