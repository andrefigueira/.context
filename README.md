# The .context method

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Stars](https://img.shields.io/github/stars/andrefigueira/.context?style=social)](https://github.com/andrefigueira/.context/stargazers)
[![Documentation](https://img.shields.io/badge/docs-substrate-blue.svg)](./.context/)
[![.context](https://img.shields.io/badge/.context-method-blue)](https://github.com/andrefigueira/.context)

> **TL;DR**: Give AI tools the *minimum* context they need, loaded at the *right moment*. Structured around measured LLM failure modes. A lean `CLAUDE.md` + path-scoped rules + an optional deep-reference tier = AI that actually performs.

A research-backed "Documentation as Code as Context" template. Transforms any software project into a self-documenting, AI-optimized codebase where structured documentation is loaded *selectively* rather than dumped into every turn.

## Why this version is different

Earlier iterations of documentation-as-context methodologies (including earlier versions of this template) followed the intuition that "more context is better." Recent research shows the opposite is measurably true once you cross a surprisingly small budget.

This version is engineered around four findings:

| Finding | Implication |
|---|---|
| **Context length alone hurts performance**: 13% to 85% accuracy degradation as input grows, even when added content is relevant. [[paper]](https://arxiv.org/html/2508.14850v1) | Keep always-loaded content small. |
| **Instruction budget is ~150-200**: frontier models reliably follow this many distinct rules. Claude Code's built-in prompt uses ~50. [[paper]](https://arxiv.org/html/2507.11538v1) | Distribute rules across path-scoped files. |
| **Lost in the middle**: LLM attention is U-shaped. Mid-document instructions get ignored even when most relevant. [[paper]](https://arxiv.org/abs/2307.03172) | Short files only. Critical rules at the top. |
| **Path-scoped rules cut rule-token usage sharply**: most rules are inactive for any single task. | Default to `.claude/rules/*.md`, not `CLAUDE.md` bloat. |

Full citations and detailed implications: [`.context/research.md`](./.context/research.md).

## The Two-Tier Architecture

```
Tier 1 (Always loaded)                    Tier 2 (Loaded on demand)
───────────────────────                   ─────────────────────────
CLAUDE.md (<=50 lines)                    .context/architecture/
AGENTS.md (<=90 lines)                    .context/auth/
.context/ai-rules.md                      .context/api/
.context/glossary.md                      .context/database/
.claude/rules/<domain>.md  ← path-scoped  .context/ui/
                                          .context/seo/
                                          .context/anti-patterns.md
                                          .context/boundaries.md
                                          .context/debt.md
                                          .context/errors.md
                                          .context/performance.md
                                          .context/testing.md
                                          .context/decisions/
                                          .context/prompts/
```

**Tier 1** is your LLM's working memory. Small, dense, path-aware. Loaded every turn.

**Tier 2** is your project's reference library. Long and thorough is fine here *because* loading is selective. Tier 2 files are pulled in only when a task explicitly needs them.

## Quick Start

```bash
git clone https://github.com/andrefigueira/.context.git your-project
cd your-project
rm -rf .git && git init
```

Open `CLAUDE.md`. It's under 50 lines. Edit the path globs in the rules table to match your repo layout (e.g., `src/api/**` vs `app/api/**`).

Open `.claude/rules/api.md`, `auth.md`, etc. Each is under 60 lines. Edit the rules to match your project's actual conventions. Delete ones you don't need. Add new rule files for domains you have that aren't covered.

Fill in `.context/ai-rules.md` (project-wide hard constraints) and `.context/glossary.md` (terminology). Aim for ~100 lines each.

That's Tier 1 done. You're operational. Tier 2 can be filled in over weeks as the project grows.

## Minimal Starter (Research-Aligned)

Don't want the full template? Start here:

| File | Purpose | Time |
|------|---------|------|
| `CLAUDE.md` | Entry point: path-to-rule table + pointers | 15 min |
| `.context/ai-rules.md` | Hard constraints that apply to all code | 20 min |
| `.context/glossary.md` | Your terminology | 10 min |
| `.claude/rules/<domain>.md` | One rule file for your biggest code area | 20 min |

That's it. 65 minutes. You have the research-backed structure with nothing you don't need yet.

## Adding to Existing Projects

1. **Create the minimal structure:**
   ```bash
   mkdir -p .claude/rules .context/decisions
   touch CLAUDE.md AGENTS.md .context/ai-rules.md .context/glossary.md
   ```

2. **Generate Tier 1 with AI, under research constraints:**
   ```
   Analyze this codebase and generate the Tier 1 files for the
   research-backed .context method (github.com/andrefigueira/.context).

   Produce:
   1. CLAUDE.md with a path-scoped rules table. Target <=50 lines.
   2. .context/ai-rules.md: only rules that apply project-wide. Target <100 lines.
   3. .context/glossary.md: actual project terms.
   4. One .claude/rules/<domain>.md per major code area. Each <80 lines.

   Budget: the aggregate active rule count for any one conversation must stay
   under ~150 distinct rules. If we exceed that, split the domain further
   rather than making a file longer.
   ```

3. **Review and cut.** The first pass will be too long. Cut aggressively. If a rule only sometimes applies, push it to a path-scoped file.

4. **Add Tier 2 when a task needs depth**, not up front.

## How AI Tools Discover the Context

| Tool | Entry point | Loading model |
|---|---|---|
| **Claude Code** | `CLAUDE.md` (auto-loaded) | Path-to-rule table in CLAUDE.md; agent loads matching `.claude/rules/<file>` when touching matching paths |
| **Cursor** | `.cursor/rules/*.mdc` with `globs:` | Native path scoping; each rule references a `.claude/rules/<file>` |
| **GitHub Copilot** | `.github/copilot-instructions.md` + `.github/instructions/*.instructions.md` | `applyTo:` frontmatter scopes to paths |
| **Windsurf** | `.windsurfrules` | No native path scoping; keep root rule small, reference the folder |
| **ChatGPT / generic** | Paste relevant files into prompt | Use the path-to-rule table in AGENTS.md to pick what to include |

Cross-tool wiring is documented in [`AGENTS.md`](./AGENTS.md).

## Before & After

**Without research-backed context:**
```
Prompt: "Add a password reset endpoint"
Context loaded: 5,900 lines of substrate docs in every turn
Output: Plausible implementation. Ignores half your auth rules
(lost in the middle). Invents an error code not in your catalog.
Uses slightly-wrong terminology from the glossary. You spend
25 minutes fixing it.
```

**With research-backed context:**
```
Prompt: "Add a password reset endpoint"
Context loaded: CLAUDE.md (<=50 lines) + ai-rules.md + glossary.md
               + .claude/rules/auth.md (path-scoped, <=80 lines)
               + .claude/rules/api.md (path-scoped, <=80 lines)
Output: Uses your JWT pattern, your error codes (AUTH_*),
your service layer, your rate-limit helper. Correct terminology.
Ready to review. Active instruction count: well under budget.
```

The difference is not "more context." It is *less context, loaded at the right moment*.

## Add the Badge

Using .context in your project? Add the badge:

```markdown
[![.context](https://img.shields.io/badge/.context-method-blue)](https://github.com/andrefigueira/.context)
```

## FAQ

### Didn't the older version of this template say "more files = better"?

Yes. It was wrong about the mechanism. The goal was always "give the AI enough to avoid guessing." The methodology had not yet caught up to the research showing that *how* you load context matters more than *how much* you have. This version fixes that. The [`.context/decisions/004-research-backed-pivot.md`](./.context/decisions/004-research-backed-pivot.md) ADR documents the reasoning.

### If less context is better, why keep Tier 2 at all?

Tier 2 is not loaded by default. It only enters the conversation when a specific task pulls it in. Having deep reference material available on demand is strictly additive: it costs nothing on the turns that do not need it. The mistake was putting Tier 2 content into the always-loaded set.

### What counts as a "rule" for the 150-200 budget?

A distinct instruction the model is expected to follow. Each numbered item in `ai-rules.md` counts as one. Each numbered rule in a path-scoped file counts as one. Code patterns in the rule files do not count individually (they are examples of one rule: "use this pattern"). A rough way to audit: open every file that would be loaded for a given task and count the imperative statements.

### What if a rule genuinely applies everywhere?

Put it in `.context/ai-rules.md`. The file exists for project-wide constraints. But be honest: most "universal" rules are actually "universal within a domain." A rule about database transactions only matters when writing database code.

### Won't path-scoped loading miss cross-cutting concerns?

Sometimes, yes. This is the tradeoff. Mitigations:
- True cross-cutting constraints (naming, security baseline, no-secrets-in-code) live in `.context/ai-rules.md` and load every turn.
- When a task spans domains (e.g., "add an authenticated endpoint that writes to the database"), multiple rule files load by path match.
- For complex multi-domain tasks, pre-built task prompts in `.context/prompts/` explicitly list the rule files to load.

### How do I migrate from the old version?

Existing `.context/` content stays and becomes Tier 2. Three migration steps:

1. **Slim CLAUDE.md** to the path-scoped-table shape (see this repo's `CLAUDE.md`).
2. **Extract path-specific rules** out of `.context/ai-rules.md` and into `.claude/rules/<domain>.md`.
3. **Leave the domain folders alone.** They are now Tier 2, loaded on demand via task prompts or explicit references.

Full migration walkthrough: [`.context/decisions/004-research-backed-pivot.md`](./.context/decisions/004-research-backed-pivot.md).

### Does this work for non-code domains (writing, design, research)?

The principles transfer. The specific rule files are code-focused in this template, but `.claude/rules/` can be anything path-scoped: `rules/marketing-copy.md`, `rules/brand-guidelines.md`, etc. The two-tier split and the research constraints are not code-specific.

### How do I validate the context file quality?

Run this prompt periodically:

```
Audit the .context/ and .claude/rules/ files against the actual codebase.

Tier 1 checks:
- Is CLAUDE.md under 50 lines? If not, what can be pushed to .claude/rules/?
- Does every path in the rules table still exist in the repo?
- Count distinct rules across ai-rules.md + all rule files. Flag if >200.

Tier 2 checks:
- Are any domain docs contradicted by current code?
- Are there undocumented patterns in the code that deserve a Tier 2 file?
- Any ADRs that describe decisions that have since been reversed?

Report a prioritized list. Flag any file where a rule has moved into
the "middle" of a longer document. Propose reordering or splitting.
```

## Structure Overview

```
README.md                          # This file
CLAUDE.md                          # Tier 1: Claude Code entry (path-scoped table)
AGENTS.md                          # Tier 1: other AI tools entry
LICENSE

.claude/
├── rules/                         # Tier 1: path-scoped rules
│   ├── README.md                  # How path-scoped loading works
│   ├── api.md                     # Loaded on src/api/**, src/routes/**, ...
│   ├── auth.md                    # Loaded on src/auth/**, ...
│   ├── database.md                # Loaded on src/models/**, migrations/**, ...
│   ├── ui.md                      # Loaded on src/components/**, *.tsx, ...
│   ├── seo.md                     # Loaded on src/app/**/page.*, ...
│   └── testing.md                 # Loaded on **/*.test.*, tests/**, ...
├── commands/                      # Slash commands
└── skills/                        # Skill definitions

.context/
├── substrate.md                   # Methodology entry point
├── research.md                    # The evidence (read this first)
│
├── # Tier 1 content
├── ai-rules.md                    # Project-wide hard constraints
├── glossary.md                    # Terminology
│
├── # Tier 2: always available, loaded on demand
├── anti-patterns.md
├── boundaries.md
├── debt.md
├── errors.md
├── workflows.md
├── env.md
├── testing.md
├── performance.md
├── dependencies.md
├── code-review.md
├── monitoring.md
├── events.md
├── feature-flags.md
├── versioning.md
├── changelog.md
│
├── # Decisions
├── decisions/                     # ADRs, including 004-research-backed-pivot.md
│
├── # Pre-built task prompts
├── prompts/                       # new-endpoint, new-feature, fix-bug, refactor, ...
│
└── # Domain deep-dives (Tier 2)
    ├── architecture/
    ├── auth/
    ├── api/
    ├── database/
    ├── ui/
    └── seo/
```

## Adapting to Your Tech Stack

The rule files and code examples below are templates. Adapt them to your stack. The *patterns* are what matter.

### TypeScript / Node.js

**Domain Error Pattern**
```typescript
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
  DuplicateEmail: new DomainError('DUPLICATE_EMAIL', 'Email already exists', 409),
} as const;
```

**Repository Pattern**
```typescript
export interface UserRepository {
  create(user: CreateUserDTO): Promise<User>;
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
}

export class PrismaUserRepository implements UserRepository {
  constructor(private prisma: PrismaClient) {}
  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }
}
```

**Express JWT Middleware**
```typescript
export const authMiddleware = async (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload;
    req.user = await userService.findById(payload.sub);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

### Python / FastAPI

**Domain Error Pattern**
```python
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
    DUPLICATE_EMAIL = DomainError("DUPLICATE_EMAIL", "Email already exists", 409)
```

**Repository Pattern**
```python
class SQLAlchemyUserRepository(UserRepository):
    def __init__(self, session: AsyncSession):
        self.session = session

    async def find_by_email(self, email: str) -> Optional[User]:
        result = await self.session.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
```

**FastAPI Auth Dependency**
```python
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

### Java / Spring Boot

**Domain Error Pattern**
```java
@Getter
public class DomainError extends RuntimeException {
    private final String code;
    private final int statusCode;

    public DomainError(String code, String message, int statusCode) {
        super(message);
        this.code = code;
        this.statusCode = statusCode;
    }

    public static final DomainError USER_NOT_FOUND =
        new DomainError("USER_NOT_FOUND", "User not found", 404);
    public static final DomainError DUPLICATE_EMAIL =
        new DomainError("DUPLICATE_EMAIL", "Email already exists", 409);
}
```

**Spring Security Filter**
```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {
    private final JwtService jwtService;
    private final UserService userService;

    @Override
    protected void doFilterInternal(
        HttpServletRequest request, HttpServletResponse response, FilterChain chain
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

## AI-Assisted Substrate Generation

Let AI do the first draft, but constrain it to the research budget. Use this prompt:

```
Generate the Tier 1 files for the research-backed .context method
(github.com/andrefigueira/.context) for this project.

PROJECT CONTEXT:
- Name: [YOUR PROJECT]
- Stack: [LANGUAGE / FRAMEWORK / DB]
- Architecture: [MONOLITH / MICROSERVICES / SERVERLESS]
- Auth: [JWT / SESSION / OAUTH]

PRODUCE:
1. CLAUDE.md: lean entry point with path-to-rule table. Target <=50 lines.
2. AGENTS.md: cross-tool variant. Target <=90 lines.
3. .context/ai-rules.md: project-wide constraints only. <100 lines.
4. .context/glossary.md: real project terms.
5. One .claude/rules/<domain>.md for each major code area. <80 lines each.

HARD CONSTRAINTS:
- Aggregate distinct instructions across all loaded-at-once files must stay
  under ~150 (the model instruction budget). Count them at the end.
- No file exceeds 100 lines. If it wants to, split it by path scope instead.
- Hard rules at the top of each file, not buried in the middle.
- Rules are one-line imperatives. Explanations belong in Tier 2 references.

DO NOT PRODUCE Tier 2 content (architecture/, auth/, api/, etc.) in this pass.
Tier 2 gets generated on demand, per task, not up front.
```

Then iterate: review each file, cut aggressively, push path-specific rules down.

## Related Projects

- 📖 [Original methodology article](https://buildingbetter.tech/p/documentation-as-code-as-context): the foundational concept
- 🧠 [Information Substrate Convergence](https://github.com/andrefigueira/information-substrate-convergence): advanced implementation and research
- 🎨 [.context-designs](https://github.com/andrefigueira/.context-designs): UI component library built with this method

## Research References

- [Context Length Alone Hurts LLM Performance](https://arxiv.org/html/2508.14850v1): 13-85% degradation as input grows
- [How Many Instructions Can LLMs Follow at Once?](https://arxiv.org/html/2507.11538v1): ~150-200 instruction budget
- [Lost in the Middle](https://arxiv.org/abs/2307.03172): U-shaped attention curve
- Claude Code community patterns on path-scoped rules: large reduction in rule-token usage per turn

All findings and their methodological implications are discussed in [`.context/research.md`](./.context/research.md).

## Contributing

1. Fork the repository
2. Create a feature branch
3. Changes to methodology require an ADR in `.context/decisions/` citing the reasoning
4. Changes to Tier 1 files must keep the aggregate instruction count inside the budget
5. Submit a pull request

## Community

- 🐛 **Report issues**: [GitHub Issues](https://github.com/andrefigueira/.context/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/andrefigueira/.context/discussions)

## License

MIT License. See [LICENSE](LICENSE).

---

**Ready?** Start with [`.context/research.md`](./.context/research.md) to understand why the method is shaped this way, then open [`CLAUDE.md`](./CLAUDE.md) and adapt it to your repo.
