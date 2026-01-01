# Project Glossary

This file defines project-specific terminology. AI should use these terms consistently and understand their specific meaning in this codebase.

## Domain Terms

| Term | Definition | Usage |
|------|------------|-------|
| **Substrate** | The structured documentation system in `.context/` | "Update the substrate when adding new patterns" |
| **Domain** | A logical grouping of related functionality (auth, api, database) | "The auth domain handles all authentication" |
| **Context** | Documentation that provides AI with project understanding | "Include relevant context before generating code" |

## Technical Terms

| Term | Definition | Not to be confused with |
|------|------------|------------------------|
| **Handler** | HTTP request handler (controller layer) | Service (business logic layer) |
| **Service** | Business logic layer component | Handler or Repository |
| **Repository** | Data access layer abstraction | ORM model or direct database access |
| **Model** | Data structure representing domain entity | Database table (which is "schema") |
| **DTO** | Data Transfer Object for API boundaries | Domain Model (internal representation) |
| **Entity** | Domain object with identity and lifecycle | Value Object (identity-less) |

## Project-Specific Terms

| Term | Meaning in this project |
|------|------------------------|
| **User** | Authenticated account holder |
| **Profile** | Extended user information (bio, avatar, preferences) |
| **Session** | Active authentication context (JWT + refresh token pair) |
| **Permission** | Granular access right (e.g., `user:read`) |
| **Role** | Named collection of permissions (e.g., `admin`) |

## Abbreviations

| Abbreviation | Full Form | Context |
|--------------|-----------|---------|
| **DI** | Dependency Injection | Architecture patterns |
| **DTO** | Data Transfer Object | API layer |
| **JWT** | JSON Web Token | Authentication |
| **RBAC** | Role-Based Access Control | Authorization |
| **ORM** | Object-Relational Mapping | Database access |
| **ADR** | Architecture Decision Record | Documentation |

## Status Values

| Status | Meaning | Valid transitions |
|--------|---------|-------------------|
| `active` | Normal operational state | → `inactive`, `suspended` |
| `inactive` | User-initiated deactivation | → `active` |
| `suspended` | Admin-initiated restriction | → `active` |
| `deleted` | Soft-deleted, not recoverable by user | (terminal) |

## Error Code Prefixes

| Prefix | Domain | Example |
|--------|--------|---------|
| `AUTH_` | Authentication errors | `AUTH_INVALID_TOKEN` |
| `USER_` | User management errors | `USER_NOT_FOUND` |
| `VAL_` | Validation errors | `VAL_INVALID_EMAIL` |
| `DB_` | Database errors | `DB_CONNECTION_FAILED` |
| `API_` | API/HTTP errors | `API_RATE_LIMITED` |

---

**Usage**: When generating code or documentation, use these exact terms. Do not invent synonyms or alternative phrasings for defined concepts.
