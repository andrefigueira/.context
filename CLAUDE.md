# Claude Code Configuration

This file is the entry point for Claude Code users. Claude Code automatically reads this file and uses it to understand how to work with this project.

## About This Project

This repository teaches the **.context method** (Substrate Methodology), a documentation-as-code-as-context approach. The actual documentation lives in the `.context/` folder. This `CLAUDE.md` file tells Claude Code how to use it.

## How This Works

```
┌─────────────────────────────────────────────────────┐
│                  .context/ folder                    │
│         (Your structured documentation)              │
│                                                      │
│  substrate.md, architecture/, auth/, api/, etc.     │
└─────────────────────────────────────────────────────┘
                         ▲
                         │
         ┌───────────────┴───────────────┐
         │                               │
    ┌────┴────┐                    ┌─────┴─────┐
    │CLAUDE.md│                    │ agents.md │
    │(you are │                    │           │
    │  here)  │                    │  Other    │
    │         │                    │ AI Tools  │
    │(auto)   │                    │ (manual)  │
    └─────────┘                    └───────────┘
```

**CLAUDE.md** (this file) is for Claude Code. It's auto-loaded at session start.

**agents.md** is for other AI tools (ChatGPT, Cursor, Copilot). It explains how to manually include `.context/` files in prompts.

Both point to the same `.context/` documentation.

## Documentation Index

### Entry Point
- `.context/substrate.md` - **Start here.** Methodology overview and navigation guide.

### AI-Specific Context (Read These First)
| File | Purpose |
|------|---------|
| `.context/ai-rules.md` | Hard constraints and non-negotiable standards for code generation |
| `.context/glossary.md` | Project-specific terminology to use consistently |
| `.context/anti-patterns.md` | What NOT to do, with bad/good code examples |
| `.context/boundaries.md` | What you should and should not modify |
| `.context/debt.md` | Known technical debt to avoid compounding |

### Architecture Decision Records
| File | Purpose |
|------|---------|
| `.context/decisions/README.md` | ADR template and index |
| `.context/decisions/001-jwt-authentication.md` | Why JWT was chosen |
| `.context/decisions/002-repository-pattern.md` | Why repository pattern |
| `.context/decisions/003-postgresql-database.md` | Why PostgreSQL |

### Architecture Domain
| File | Purpose |
|------|---------|
| `.context/architecture/overview.md` | System architecture, layered design, Mermaid diagrams |
| `.context/architecture/dependencies.md` | Dependency injection patterns |
| `.context/architecture/patterns.md` | Code organization, error handling, naming conventions |

### Authentication Domain
| File | Purpose |
|------|---------|
| `.context/auth/overview.md` | JWT auth flow, token strategy, RBAC model |
| `.context/auth/integration.md` | HTTP middleware, framework integration |
| `.context/auth/security.md` | Password hashing (Argon2id), threat mitigation |

### API Domain
| File | Purpose |
|------|---------|
| `.context/api/endpoints.md` | REST endpoint reference, request/response formats |
| `.context/api/headers.md` | HTTP headers, CORS configuration |
| `.context/api/examples.md` | Client implementation examples |

### Database Domain
| File | Purpose |
|------|---------|
| `.context/database/schema.md` | PostgreSQL schema, ERD diagrams, indexes |
| `.context/database/models.md` | Data models, validation rules |
| `.context/database/migrations.md` | Migration strategy, rollback procedures |

### UI Domain
| File | Purpose |
|------|---------|
| `.context/ui/overview.md` | Design tokens, component architecture, accessibility |
| `.context/ui/patterns.md` | Component implementation, state management |

### SEO Domain
| File | Purpose |
|------|---------|
| `.context/seo/overview.md` | Meta tags, structured data schemas, Core Web Vitals |

### Cross-Cutting
| File | Purpose |
|------|---------|
| `.context/guidelines.md` | Git workflow, testing standards, deployment, code review |

## Instructions for Claude Code

### Before Generating Code
1. Read `.context/substrate.md` first for project orientation
2. Read domain-specific files relevant to the task
3. Follow patterns documented in `.context/architecture/patterns.md`

### Task-Specific Context

**Authentication work:**
Read `.context/auth/overview.md`, `.context/auth/security.md`, `.context/auth/integration.md`

**API development:**
Read `.context/api/endpoints.md`, `.context/api/examples.md`, `.context/architecture/patterns.md`

**Database work:**
Read `.context/database/schema.md`, `.context/database/models.md`

**Frontend/UI work:**
Read `.context/ui/overview.md`, `.context/ui/patterns.md`

**SEO implementation:**
Read `.context/seo/overview.md`

### Code Standards
- Follow coding patterns from `.context/architecture/patterns.md`
- Implement security measures from `.context/auth/security.md`
- Use data models from `.context/database/models.md`
- Match existing naming conventions in the codebase
- Follow the test pyramid strategy from `.context/guidelines.md`

### Do Not
- Generate code without reading relevant context files first
- Ignore security patterns and constraints
- Create new architectural patterns without documentation
- Assume implementation details not explicitly documented
- Override established conventions without clear rationale

### When Making Changes
- Update relevant `.context/` files when making architectural decisions
- Document trade-offs and rationale in "Decision History" sections
- Keep code examples in documentation current and functional
