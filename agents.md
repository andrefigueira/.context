# AI Agents and Context Usage Guide

This document provides specific guidance for AI agents and developers on how to effectively use the Substrate documentation system for consistent, accurate code generation and project understanding.

## Context Consumption Patterns

### Primary Context Files
When working with this project, always reference these core files first:

```bash
# Essential project understanding
.substrate/substrate.md           # Methodology and navigation
.substrate/architecture/overview.md  # System architecture
.substrate/guidelines.md          # Development standards
```

### Domain-Specific Context
For specific implementation tasks, combine relevant domain files:

```bash
# Authentication tasks
.substrate/auth/overview.md + .substrate/auth/security.md + .substrate/auth/integration.md

# API development
.substrate/api/endpoints.md + .substrate/api/examples.md + .substrate/architecture/patterns.md

# Database work
.substrate/database/schema.md + .substrate/database/models.md + .substrate/database/migrations.md
```

## Agent Prompt Patterns

### Code Generation Prompts
```
Based on the following project documentation:
[Include relevant .substrate files]

Generate [specific component/feature] following the established patterns, security guidelines, and architectural decisions documented above.

Requirements:
- Follow the coding standards from .substrate/architecture/patterns.md
- Implement security measures from .substrate/auth/security.md
- Use the data models defined in .substrate/database/models.md
```

### Code Review Prompts
```
Review the following code against project standards:
[Include code to review]

Project context:
[Include .substrate/architecture/patterns.md and .substrate/guidelines.md]

Check for:
- Adherence to established patterns
- Security compliance
- Performance considerations
- Documentation completeness
```

### Debugging Prompts
```
Debug this issue:
[Describe the problem]

Relevant project context:
[Include domain-specific .substrate files]

Consider the architectural decisions and trade-offs documented in the context when suggesting solutions.
```

## Context Optimization Guidelines

### For AI Agents
1. **Always read substrate.md first** for project overview and navigation
2. **Combine multiple domain files** for complex tasks spanning multiple areas
3. **Reference decision history** sections for understanding trade-offs
4. **Use code examples** as templates for consistent implementation
5. **Follow security patterns** documented in auth/ domain

### For Human Developers
1. **Start with substrate.md** for project orientation
2. **Use domain files as implementation guides** rather than abstract documentation
3. **Update documentation** when making architectural decisions
4. **Reference patterns** before implementing new features
5. **Contribute back** improvements and new patterns

## Common Usage Scenarios

### New Feature Development
1. Read relevant domain documentation
2. Understand existing patterns and constraints
3. Generate initial implementation using AI with context
4. Review against project standards
5. Update documentation with new patterns

### Bug Fixing
1. Identify affected domains
2. Review relevant security and architecture constraints
3. Generate fix following established patterns
4. Validate against documented trade-offs

### Refactoring
1. Read architecture overview and patterns
2. Understand dependency injection and architectural layers
3. Plan refactoring respecting documented decisions
4. Update patterns documentation if introducing new approaches

## Agent Behavior Guidelines

### Do
- Always reference documentation before generating code
- Follow established naming conventions and patterns
- Respect security guidelines and architectural constraints
- Update documentation when making significant changes
- Ask for clarification when documentation is ambiguous

### Don't
- Generate code without understanding project context
- Ignore security patterns and constraints
- Create new architectural patterns without documentation
- Assume implementation details not explicitly documented
- Override established conventions without strong rationale

## Context Update Workflow

### When AI Agents Make Changes
1. Generate code following documented patterns
2. Identify any new patterns or decisions
3. Suggest documentation updates for new patterns
4. Flag any deviations from established guidelines

### When Humans Review AI Output
1. Validate against documented standards
2. Check for security compliance
3. Ensure architectural consistency
4. Update documentation for approved new patterns

## Integration with Development Tools

### CI/CD Integration
```bash
# Example validation commands (implement according to your CI/CD system)
# Lint documentation for consistency
markdownlint .context/

# Validate code examples compile/run
go build ./...
npm run build
```

### Editor Integration
Configure your editor to:
- Auto-suggest patterns from .substrate files
- Validate against documented standards
- Quick access to domain documentation

### AI Tool Configuration
Set up your AI development tools to:
- Automatically include relevant .substrate context
- Validate output against project patterns
- Suggest documentation updates for new patterns

---

This guide ensures consistent, context-aware development whether you're an AI agent or human developer working with the Substrate methodology.