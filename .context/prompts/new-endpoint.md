# Prompt: New API Endpoint

## Context Files to Include
```
.context/api/endpoints.md
.context/architecture/patterns.md
.context/database/schema.md (if data access needed)
.context/auth/overview.md (if authentication needed)
```

## Prompt

```
Create a new API endpoint with the following requirements:

**Endpoint**: [METHOD] /api/[path]
**Purpose**: [What this endpoint does]
**Authentication**: [Required/Optional/None]
**Authorization**: [What permissions needed, if any]

**Request**:
- Body/Query params: [Describe expected input]
- Validation rules: [Any specific validation]

**Response**:
- Success: [What to return]
- Errors: [Expected error cases]

**Business Logic**:
[Describe what should happen]

Follow the patterns in the included context files. Ensure:
1. Input validation at the handler level
2. Business logic in the service layer
3. Data access through repository pattern
4. Consistent error handling
5. Appropriate logging
```
