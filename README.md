# The .context method

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Stars](https://img.shields.io/github/stars/andrefigueira/.context?style=social)](https://github.com/andrefigueira/.context/stargazers)
[![Documentation](https://img.shields.io/badge/docs-substrate-blue.svg)](./.context/)

> **TL;DR**: Create a `.context/` folder in your project with structured markdown documentation. AI tools read it to understand your codebase, reducing hallucinations and producing code that follows your patterns. This repo is the template.

A complete "Documentation as Code as Context" template implementing the Substrate Methodology. Transform any software project into a self-documenting, AI-optimized codebase with modular, Git-native documentation that serves as a living knowledge base.

## What is the Substrate Methodology?

The Substrate Methodology solves the critical problem of outdated documentation and AI hallucinations by creating a structured, domain-organized documentation system in a `.context/` directory. This approach:

- **Eliminates documentation drift** through Git-native versioning
- **Reduces AI hallucinations** with structured context
- **Accelerates onboarding** with comprehensive domain knowledge
- **Captures decision history** for future reference
- **Scales development teams** with consistent patterns

## Quick Start

1. **Clone and customize:**
   ```bash
   git clone https://github.com/andrefigueira/.context.git your-project
   cd your-project
   rm -rf .git && git init
   ```

2. **Customize the template:**
   - Replace `[Your Project Name]` placeholders
   - Update code examples for your stack
   - Modify domains in `.context/` to match your architecture

3. **Start documenting:**
   ```bash
   # Edit the entry point
   vim .context/substrate.md
   
   # Add your first domain documentation
   vim .context/architecture/overview.md
   ```

4. **Generate your substrate with AI:**
   Use the comprehensive AI prompt below to create domain-specific documentation for your project. See [AI-Assisted Substrate Generation](#ai-assisted-substrate-generation) section for the complete prompt.

## Why This Matters

Modern software development faces a documentation crisis:
- Most developers work with outdated or incomplete docs
- AI tools hallucinate when missing context
- Knowledge silos slow team velocity
- Onboarding takes weeks instead of days

The Substrate Methodology transforms documentation from a burden into a force multiplier, creating a comprehensive knowledge base that grows with your codebase.

## How the .context Method Works with AI Tools

The `.context/` folder is the core of this methodology. It contains your structured documentation. But different AI tools need different ways to discover and use this context.

### The Entry Point Problem

AI tools need to know your `.context/` folder exists and how to use it. This is solved differently depending on your tool:

| AI Tool | Entry Point | How It Works |
|---------|-------------|--------------|
| **Claude Code** (Anthropic CLI) | `CLAUDE.md` | Auto-loaded at session start. Claude Code reads this file automatically and follows its instructions to reference `.context/` files. |
| **Other AI tools** (ChatGPT, Cursor, Copilot, generic Claude) | `agents.md` | Manual inclusion. You copy relevant `.context/` files into your prompts following the patterns in agents.md. |

### Why Two Files?

**CLAUDE.md** is a Claude Code feature. When you start a Claude Code session, it automatically reads `CLAUDE.md` from your project root and treats it as persistent instructions. This means Claude Code will automatically know to check `.context/` files before generating code.

**agents.md** exists for AI tools that don't have this auto-discovery feature. It documents how to manually feed context into your prompts for consistent results.

### The Relationship

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
    │         │                    │           │
    │ Claude  │                    │  Other    │
    │  Code   │                    │ AI Tools  │
    │(auto)   │                    │ (manual)  │
    └─────────┘                    └───────────┘
```

Both files point to the same `.context/` documentation. They're just different bridges for different tools.

## Hot Tip: Iterate on Your .context

Your `.context/` folder is a living document. Once you've created it, have your AI agent review and validate it against your actual codebase. This catches gaps, outdated information, and opportunities to add richer context.

Use this prompt to validate and improve your substrate:

```
Review my .context/ documentation against the actual codebase. For each domain file:

1. **Accuracy check**: Does the documentation match the current implementation? Flag any outdated patterns, deprecated approaches, or missing features.

2. **Completeness check**: What's documented in the code but missing from .context/? Look for:
   - Undocumented API endpoints
   - Missing error handling patterns
   - Security measures not captured
   - Database fields/tables not in schema docs
   - UI components without documentation

3. **Richness check**: Where could the documentation be more useful? Consider:
   - Adding more code examples
   - Including edge cases and error scenarios
   - Documenting "why" decisions were made (Decision History sections)
   - Adding Mermaid diagrams for complex flows

4. **Consistency check**: Are naming conventions, patterns, and terminology consistent across all .context/ files?

Provide a prioritized list of improvements with specific suggestions for each.
```

Run this periodically (monthly or after major features) to keep your substrate accurate and valuable.

## Real-World Example

See the methodology in action: [.context-designs](https://github.com/andrefigueira/.context-designs) - A complete UI component library built with Tailwind CSS using the .context method. This project demonstrates how documentation-as-context enables consistent design system implementation and AI-assisted component generation.

## Structure Overview

```
README.md                     # Project introduction and quick start
CLAUDE.md                     # Claude Code configuration (use this if using Claude Code CLI)
agents.md                     # AI agent usage patterns (use this for other AI tools)
.context/
├── substrate.md              # Entry point and methodology guide
├── architecture/             # System design and patterns
│   ├── overview.md          # System architecture with diagrams
│   ├── dependencies.md      # Dependency injection patterns
│   └── patterns.md          # Code organization and error handling
├── auth/                     # Authentication and security
│   ├── overview.md          # JWT authentication flows
│   ├── integration.md       # HTTP middleware integration
│   └── security.md          # Security model and threat mitigation
├── api/                      # API reference and examples
│   ├── endpoints.md         # REST API documentation
│   ├── headers.md           # HTTP headers and CORS
│   └── examples.md          # Client implementation examples
├── database/                 # Data models and migrations
│   ├── schema.md            # PostgreSQL schema design
│   ├── models.md            # Data models and validation
│   └── migrations.md        # Migration strategy and tooling
├── ui/                       # Frontend and design system
│   ├── overview.md          # Component architecture and tokens
│   └── patterns.md          # UI implementation patterns
├── seo/                      # Search engine optimization
│   └── overview.md          # Meta tags, structured data, performance
└── guidelines.md             # Development workflows and standards
```

Each domain contains modular Markdown files optimized for:
- **Human readability** with clear structure
- **AI consumption** with parseable formats
- **Version control** with Git integration
- **Extensibility** for project-specific needs

## Features

- ✅ **Modular Documentation**: Domain-organized for precise context
- ✅ **AI-Optimized Format**: Structured for LLM consumption
- ✅ **Decision History**: Captures rationale and trade-offs
- ✅ **Code Examples**: Real patterns for immediate use
- ✅ **Mermaid Diagrams**: Visual architecture documentation
- ✅ **Generic Template**: Adaptable to any tech stack
- ✅ **MIT Licensed**: Free for commercial use

## Usage Patterns

### For Development Teams
```bash
# Before implementing auth
cat .context/auth/*.md > context.txt
# Use context.txt with your preferred AI tool for implementation guidance
```

### For Onboarding
```bash
# New developer orientation
cat .context/substrate.md .context/architecture/overview.md
```

### For AI-Assisted Development
```bash
# Context-aware code generation
echo "Based on the following documentation, implement user registration:" && \
cat .context/auth/integration.md .context/database/models.md
```

## AI-Assisted Substrate Generation

**Recommended Approach**: Instead of manually writing documentation, use AI to generate comprehensive substrate documentation tailored to your specific project. This ensures consistency with the methodology while adapting to your unique architecture, tech stack, and business domain.

### Complete AI Generation Prompt

Copy and paste this prompt into your preferred AI tool (Claude, GPT-4, etc.) to generate substrate documentation for your project:

```
You are an expert technical writer specializing in the "Documentation as Code as Context" methodology. Create comprehensive substrate documentation following the exact structure and quality standards of this template: https://github.com/andrefigueira/.context

PROJECT CONTEXT:
- Project Name: [YOUR PROJECT NAME]
- Tech Stack: [YOUR TECH STACK - e.g., Node.js/Express, React, PostgreSQL, Redis]
- Architecture Pattern: [YOUR PATTERN - e.g., microservices, monolith, serverless]
- Authentication Method: [YOUR AUTH - e.g., OAuth2, JWT, session-based]
- Database Type: [YOUR DB - e.g., PostgreSQL, MongoDB, MySQL]
- Target Audience: [YOUR USERS - e.g., internal APIs, public SaaS, enterprise]

REQUIREMENTS:
1. Generate content for ALL substrate domains: architecture, auth, api, database, guidelines
2. Use my specific tech stack and adapt all code examples accordingly
3. Include actual implementation patterns, not generic advice
4. Add decision rationale sections explaining "why" choices were made
5. Include Mermaid diagrams for architecture flows and database schemas
6. Provide realistic error handling patterns
7. Include performance considerations and security measures
8. Add specific testing strategies for my tech stack
9. Include deployment procedures for my infrastructure

STRUCTURE TO FOLLOW:
Create these files with comprehensive, production-ready content:

substrate.md - Entry point with navigation and AI usage patterns
architecture/
├── overview.md - System architecture with Mermaid diagrams
├── dependencies.md - Dependency injection patterns for my stack
└── patterns.md - Code organization and error handling

auth/
├── overview.md - Authentication flow for my auth method
├── integration.md - Framework integration patterns
└── security.md - Security model and threat mitigation

api/
├── endpoints.md - API reference with my actual endpoints
├── headers.md - HTTP headers and middleware patterns
└── examples.md - Client implementations for my stack

database/
├── schema.md - Database schema with ERD for my data model
├── models.md - Data models and validation for my stack
└── migrations.md - Migration strategy for my database

ui/
├── overview.md - Component architecture, design tokens, specifications
└── patterns.md - UI implementation patterns, forms, modals, data display

seo/
└── overview.md - Meta tags, structured data schemas, Core Web Vitals

guidelines.md - Development workflow and deployment for my stack

QUALITY STANDARDS:
- Each file should be 400-800 words with practical examples
- Include 2-3 realistic code snippets per file using my tech stack
- Add "Decision History & Trade-offs" sections explaining architectural choices
- Use consistent technical terminology throughout
- Include specific performance benchmarks and security considerations
- Provide actionable implementation guidance, not theoretical concepts

EXAMPLE OUTPUT QUALITY:
Reference the template structure at https://github.com/andrefigueira/.context but adapt ALL content to my specific project. Don't copy generic examples - create realistic implementations for my exact tech stack and business domain.

START WITH: substrate.md as the entry point, then generate each domain systematically.
```

### Usage Instructions

1. **Replace the bracketed placeholders** with your specific project details
2. **Paste the prompt** into your AI tool of choice
3. **Generate each domain** systematically (start with substrate.md)
4. **Review and refine** the generated content for accuracy
5. **Iterate on specific sections** that need more detail

### Example Customization

For a Node.js/Express API project:
```
PROJECT CONTEXT:
- Project Name: TaskFlow API
- Tech Stack: Node.js, Express.js, TypeScript, PostgreSQL, Redis, Docker
- Architecture Pattern: Layered monolith with clear service boundaries
- Authentication Method: JWT with refresh tokens
- Database Type: PostgreSQL with Prisma ORM
- Target Audience: Internal microservice for task management SaaS
```

### AI Tool Recommendations

- **Claude (Anthropic)**: Excellent for following complex instructions and maintaining consistency
- **GPT-4 (OpenAI)**: Great for code examples and technical accuracy
- **Cursor/Continue**: IDE integration for iterative refinement

### Quality Assurance

After AI generation:
1. **Validate code examples** actually work with your stack
2. **Test database schemas** match your actual data model  
3. **Verify security patterns** align with your compliance requirements
4. **Update decision rationale** to reflect your specific constraints
5. **Add team-specific conventions** not captured in the template

This AI-assisted approach typically produces 80-90% complete documentation that requires minimal manual refinement, compared to weeks of manual documentation work.

## Contributing

We welcome contributions to improve the Substrate Methodology template:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-domain`)
3. **Document** your changes in the relevant `.context/` files
4. **Submit** a pull request

### Contribution Guidelines

- Follow the established documentation structure
- Include decision rationale for changes
- Add code examples for new patterns
- Update the main README if adding new domains
- Ensure Markdown follows consistent formatting

## Community

- 🐛 **Report issues**: [GitHub Issues](https://github.com/andrefigueira/.context/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/andrefigueira/.context/discussions)
- 📖 **Documentation**: [Substrate Methodology Guide](./.context/substrate.md)

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Adapting to Your Tech Stack

The template uses Go code examples but is designed to be adapted to any technology. Below are complete examples showing how patterns translate across languages.

### TypeScript/Node.js Example

**Domain Error Pattern**
```typescript
// types/errors.ts
export class DomainError extends Error {
  constructor(
    public code: string,
    public message: string,
    public statusCode: number = 500,
    public details?: Record<string, unknown>
  ) {
    super(message);
    this.name = 'DomainError';
  }
}

export const Errors = {
  UserNotFound: new DomainError('USER_NOT_FOUND', 'User not found', 404),
  InvalidEmail: new DomainError('INVALID_EMAIL', 'Invalid email format', 400),
  DuplicateEmail: new DomainError('DUPLICATE_EMAIL', 'Email already exists', 409),
} as const;
```

**Repository Pattern**
```typescript
// repositories/user.repository.ts
export interface UserRepository {
  create(user: CreateUserDTO): Promise<User>;
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  update(id: string, data: UpdateUserDTO): Promise<User>;
  delete(id: string): Promise<void>;
}

export class PrismaUserRepository implements UserRepository {
  constructor(private prisma: PrismaClient) {}

  async create(data: CreateUserDTO): Promise<User> {
    return this.prisma.user.create({ data });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }
}
```

**Service Layer**
```typescript
// services/user.service.ts
export class UserService {
  constructor(
    private userRepo: UserRepository,
    private validator: UserValidator,
    private cache: CacheService
  ) {}

  async createUser(req: CreateUserRequest): Promise<User> {
    await this.validator.validateCreateUser(req);

    const existing = await this.userRepo.findByEmail(req.email);
    if (existing) throw Errors.DuplicateEmail;

    const user = await this.userRepo.create({
      email: req.email,
      name: req.name,
      status: 'active',
    });

    await this.cache.set(`user:${user.id}`, user, 3600);
    return user;
  }
}
```

### Python Example

**Domain Error Pattern**
```python
# errors.py
from dataclasses import dataclass
from typing import Optional, Dict, Any

@dataclass
class DomainError(Exception):
    code: str
    message: str
    status_code: int = 500
    details: Optional[Dict[str, Any]] = None

class Errors:
    USER_NOT_FOUND = DomainError("USER_NOT_FOUND", "User not found", 404)
    INVALID_EMAIL = DomainError("INVALID_EMAIL", "Invalid email format", 400)
    DUPLICATE_EMAIL = DomainError("DUPLICATE_EMAIL", "Email already exists", 409)
```

**Repository Pattern**
```python
# repositories/user_repository.py
from abc import ABC, abstractmethod
from typing import Optional
from models import User, CreateUserDTO

class UserRepository(ABC):
    @abstractmethod
    async def create(self, user: CreateUserDTO) -> User: ...

    @abstractmethod
    async def find_by_id(self, id: str) -> Optional[User]: ...

    @abstractmethod
    async def find_by_email(self, email: str) -> Optional[User]: ...

class SQLAlchemyUserRepository(UserRepository):
    def __init__(self, session: AsyncSession):
        self.session = session

    async def create(self, data: CreateUserDTO) -> User:
        user = User(**data.dict())
        self.session.add(user)
        await self.session.commit()
        await self.session.refresh(user)
        return user

    async def find_by_email(self, email: str) -> Optional[User]:
        result = await self.session.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
```

**Service Layer**
```python
# services/user_service.py
class UserService:
    def __init__(
        self,
        user_repo: UserRepository,
        validator: UserValidator,
        cache: CacheService
    ):
        self.user_repo = user_repo
        self.validator = validator
        self.cache = cache

    async def create_user(self, req: CreateUserRequest) -> User:
        await self.validator.validate_create_user(req)

        existing = await self.user_repo.find_by_email(req.email)
        if existing:
            raise Errors.DUPLICATE_EMAIL

        user = await self.user_repo.create(CreateUserDTO(
            email=req.email,
            name=req.name,
            status="active"
        ))

        await self.cache.set(f"user:{user.id}", user, ttl=3600)
        return user
```

### Java/Spring Boot Example

**Domain Error Pattern**
```java
// exceptions/DomainError.java
@Getter
public class DomainError extends RuntimeException {
    private final String code;
    private final int statusCode;
    private final Map<String, Object> details;

    public DomainError(String code, String message, int statusCode) {
        super(message);
        this.code = code;
        this.statusCode = statusCode;
        this.details = new HashMap<>();
    }

    public static final DomainError USER_NOT_FOUND =
        new DomainError("USER_NOT_FOUND", "User not found", 404);
    public static final DomainError DUPLICATE_EMAIL =
        new DomainError("DUPLICATE_EMAIL", "Email already exists", 409);
}
```

**Repository Pattern**
```java
// repositories/UserRepository.java
@Repository
public interface UserRepository extends JpaRepository<User, UUID> {
    Optional<User> findByEmail(String email);
    List<User> findByStatus(UserStatus status);
}
```

**Service Layer**
```java
// services/UserService.java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final UserValidator validator;
    private final CacheService cache;

    @Transactional
    public User createUser(CreateUserRequest request) {
        validator.validateCreateUser(request);

        userRepository.findByEmail(request.getEmail())
            .ifPresent(u -> { throw DomainError.DUPLICATE_EMAIL; });

        User user = User.builder()
            .email(request.getEmail())
            .name(request.getName())
            .status(UserStatus.ACTIVE)
            .build();

        User saved = userRepository.save(user);
        cache.set("user:" + saved.getId(), saved, Duration.ofHours(1));
        return saved;
    }
}
```

### Framework-Specific Authentication

**Express.js JWT Middleware**
```typescript
// middleware/auth.ts
export const authMiddleware = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'No token provided' });

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload;
    req.user = await userService.findById(payload.sub);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

**FastAPI Dependency Injection**
```python
# dependencies/auth.py
async def get_current_user(
    token: str = Depends(oauth2_scheme),
    user_service: UserService = Depends(get_user_service)
) -> User:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        user = await user_service.find_by_id(payload["sub"])
        if not user:
            raise HTTPException(status_code=401, detail="User not found")
        return user
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

**Spring Security Filter**
```java
// security/JwtAuthFilter.java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {
    private final JwtService jwtService;
    private final UserService userService;

    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain chain
    ) throws ServletException, IOException {
        String token = extractToken(request);
        if (token != null && jwtService.isValid(token)) {
            String userId = jwtService.extractSubject(token);
            User user = userService.findById(UUID.fromString(userId));
            var auth = new UsernamePasswordAuthenticationToken(
                user, null, user.getAuthorities()
            );
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        chain.doFilter(request, response);
    }
}
```

## Inspiration

Based on the ["Documentation as Code as Context" methodology](https://buildingbetter.tech/p/documentation-as-code-as-context) by Building Better. This template transforms that philosophy into a practical, ready-to-use system for any software project.

### Related Projects
- 📖 [Original methodology article](https://buildingbetter.tech/p/documentation-as-code-as-context) - The foundational concept behind this template
- 🤖 [Your AI sucks because you suck at prompting](https://buildingbetter.tech/p/your-ai-sucks-because-you-suck-at) - Why context matters for AI effectiveness
- 🧠 [Information Substrate Convergence](https://github.com/andrefigueira/information-substrate-convergence) - Advanced implementation and research

---

**Ready to transform your documentation?** Start with `.context/substrate.md` and experience the difference of having your codebase document itself.
