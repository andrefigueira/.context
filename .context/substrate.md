# Substrate Methodology: Documentation as Code as Context

This is the entry point for [Your Project Name]'s comprehensive documentation system. The Substrate methodology transforms your codebase into a self-documenting system optimized for both human understanding and AI consumption.

## Navigation Guide

### AI-Specific Context
These files help AI tools work better with this codebase:
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
- **[Guidelines](./guidelines.md)** - Development workflow and cross-cutting concerns

### Quick Access
```bash
# Project overview and architecture
cat .context/substrate.md .context/architecture/overview.md

# Security and authentication context
cat .context/auth/overview.md .context/auth/security.md

# API reference and examples
cat .context/api/endpoints.md .context/api/examples.md

# Database schema and models
cat .context/database/schema.md .context/database/models.md

# UI components and design system
cat .context/ui/overview.md .context/ui/patterns.md

# SEO and structured data
cat .context/seo/overview.md
```

## Methodology Overview

The Substrate approach creates modular, version-controlled documentation that serves as executable context for development teams and AI tools. Each domain contains structured Markdown optimized for both human readability and machine parsing.

### Key Principles

**Living Documentation**: Documentation lives in Git alongside code, updating through normal PR workflows. No separate wiki or documentation site that becomes stale.

**Modular Structure**: Domain-organized files allow precise context selection. Need auth context? Pull the auth folder. Working on API changes? Reference the api domain.

**AI Optimization**: Structured format with consistent sections (overview, patterns, examples, rationale) enables reliable AI consumption and reduces hallucinations.

**Decision Capture**: Every file includes rationale sections documenting "why" decisions were made, preserving institutional knowledge.

## Using as AI Context

### For Code Generation
```bash
# Generate auth middleware
cat .context/auth/integration.md .context/auth/security.md | ai-tool

# Create new API endpoint
cat .context/api/endpoints.md .context/architecture/patterns.md | ai-tool

# Implement database model
cat .context/database/models.md .context/database/schema.md | ai-tool
```

### For Code Review
```bash
# Review against project standards
cat .context/architecture/patterns.md .context/guidelines.md | ai-tool --review [code-file]
```

### For Architecture Decisions
```bash
# Understand existing decisions before proposing changes
cat .context/architecture/*.md | ai-tool --analyze
```

### Sample AI Prompts

**Feature Implementation**:
```
Based on the following project documentation:
[Include relevant domain files]

Implement [feature description] following the established patterns, security guidelines, and architectural decisions documented above. Ensure the implementation aligns with the decision history and trade-offs already considered.
```

**Security Review**:
```
Review this code for security compliance:
[Code to review]

Security context:
[Include .context/auth/security.md]

Check against the documented security model, threat considerations, and established mitigation patterns.
```

## Documentation Standards

### File Structure
Each domain file follows this pattern:
```markdown
# Domain: Specific Topic
Brief 1-2 sentence overview of what this file covers.

## Overview
High-level description with context.

## Implementation Patterns  
Code examples and standard approaches.

## Decision History & Trade-offs
Why choices were made, alternatives considered.

## Examples
Concrete usage patterns and code snippets.
```

### Content Guidelines
- **Specific over Generic**: Document exact patterns used in this project, not general best practices
- **Code Examples**: Include real code snippets adapted for your stack
- **Decision Rationale**: Always explain why, not just what
- **Mermaid Diagrams**: Use for complex flows and relationships
- **Consistent Formatting**: Follow Markdown standards for parsing reliability

## Extending the System

### Adding New Domains
Create new folders for project-specific needs:
```bash
mkdir .context/frontend     # For UI/UX patterns
mkdir .context/testing      # For test strategies  
mkdir .context/deployment   # For CI/CD and ops
mkdir .context/monitoring   # For observability
```

### Domain Template
```markdown
# Domain: [New Domain Name]
Overview sentence explaining this domain's scope.

## Overview
Context and purpose.

## Patterns
Standard approaches and code examples.

## Decision History & Trade-offs  
Why this approach, alternatives considered.

## Integration Points
How this domain connects to others.

## Examples
Concrete usage patterns.
```

### Maintenance Workflow
1. **Update with Code Changes**: When architectural decisions change, update relevant documentation in the same PR
2. **Review Documentation**: Include documentation review in code review process
3. **Validate Examples**: Ensure code examples remain current and functional
4. **Expand Domains**: Add new domains as project complexity grows

## Benefits Realized

### For Development Teams
- **Faster Onboarding**: New developers understand system architecture quickly
- **Consistent Implementation**: Documented patterns ensure consistency across team members
- **Decision Context**: Understand why systems were built as they were
- **Reduced Cognitive Load**: Reference documentation instead of remembering details

### For AI Tools
- **Reduced Hallucinations**: Structured context prevents AI from making incorrect assumptions
- **Consistent Output**: AI generates code following established project patterns
- **Better Understanding**: Context-aware AI provides more relevant suggestions
- **Accurate Reviews**: AI can review code against documented standards

### For Project Maintenance
- **Knowledge Preservation**: Critical decisions and rationale are preserved
- **Architecture Evolution**: Clear documentation of system changes over time
- **Technical Debt Management**: Documented trade-offs inform future refactoring decisions
- **Team Scalability**: Documentation scales knowledge across growing teams

## Getting Started

1. **Read This File**: Understand the methodology and navigation
2. **Review Architecture**: Start with [architecture/overview.md](./architecture/overview.md) for system understanding
3. **Explore Domains**: Browse domain folders relevant to your current work
4. **Use with AI**: Try the provided prompt patterns with your preferred AI tool
5. **Contribute**: Update documentation as you make architectural decisions

---

**Next Steps**: Begin with [architecture/overview.md](./architecture/overview.md) for a comprehensive system overview, then explore domain-specific documentation based on your current needs.