# Substrate Methodology: Documentation as Code as Context

Entry point for the project's documentation system. Modular, version-controlled documentation that serves as context for development teams and AI tools.

## Navigation Guide

### AI-Specific Context
- **[AI Rules](./ai-rules.md)** - Hard constraints for code generation
- **[Glossary](./glossary.md)** - Project-specific terminology
- **[Anti-Patterns](./anti-patterns.md)** - What NOT to do
- **[Boundaries](./boundaries.md)** - What AI should/shouldn't modify
- **[Technical Debt](./debt.md)** - Known debt to avoid compounding
- **[Decisions](./decisions/README.md)** - Architecture Decision Records

### Core Domains
- **[Architecture](./architecture/overview.md)** - System design, patterns, and architectural decisions
- **[Authentication](./auth/overview.md)** - Security model, auth flows, and integration patterns
- **[API](./api/endpoints.md)** - REST endpoints, headers, and usage examples
- **[Database](./database/schema.md)** - Data models, schema design, and migrations
- **[UI](./ui/overview.md)** - Component architecture, design tokens, and frontend patterns
- **[SEO](./seo/overview.md)** - Meta tags, structured data, and performance optimization

### Operational Context
- **[Workflows](./workflows.md)** - Step-by-step development guides
- **[Environment](./env.md)** - Environment variables documentation
- **[Errors](./errors.md)** - Error codes catalog
- **[Testing](./testing.md)** - Testing strategy and standards
- **[Performance](./performance.md)** - Performance budgets and guidelines
- **[Dependencies](./dependencies.md)** - Approved packages and libraries
- **[Guidelines](./guidelines.md)** - Git workflow, testing, deployment

### Pre-Built Prompts
Task-specific prompts in [prompts/](./prompts/): new-endpoint, new-feature, fix-bug, refactor, review, security-audit, performance, documentation.

## Key Principles

**Living Documentation**: Lives in Git alongside code, updates through normal PR workflows.

**Modular Structure**: Domain-organized files allow precise context selection. Load only what you need for the task.

**Specific over Generic**: Document exact patterns used in this project, not general best practices.

**Decision Capture**: Every file includes rationale sections documenting "why" decisions were made.

## Documentation Standards

Each domain file follows this structure:

```markdown
# Domain: Specific Topic
Brief 1-2 sentence overview.

## Overview
High-level description with context.

## Implementation Patterns
Code examples and standard approaches.

## Decision History & Trade-offs
Why choices were made, alternatives considered.

## Examples
Concrete usage patterns and code snippets.
```

### When Creating New Domain Files
- **Specific over Generic**: Document exact patterns, not general best practices
- **Code Examples**: Include real code snippets adapted for your stack
- **Decision Rationale**: Always explain why, not just what
- **Mermaid Diagrams**: Use for complex flows and relationships
- Keep individual files under 200 lines

## Extending the System

Create new domain folders as needed:
```bash
mkdir .context/[domain-name]
```

Use the domain template above. Update `CLAUDE.md` and `agents.md` when adding new domains.
