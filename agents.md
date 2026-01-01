# AI Agents and Context Usage Guide

This file is for AI tools that don't auto-discover project context (ChatGPT, Cursor, Copilot, generic Claude interfaces). It explains how to manually include `.context/` files in your prompts.

**Using Claude Code?** See `CLAUDE.md` instead. Claude Code automatically reads that file at session start, so you don't need to manually include context.

## How the .context Method Works

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
    │         │                    │(you are   │
    │ Claude  │                    │  here)    │
    │  Code   │                    │           │
    │(auto)   │                    │ (manual)  │
    └─────────┘                    └───────────┘
```

**CLAUDE.md** is for Claude Code. It's auto-loaded at session start.

**agents.md** (this file) is for other AI tools. You manually copy `.context/` files into your prompts following the patterns below.

Both point to the same `.context/` documentation.

---

## Context Consumption Patterns

### Primary Context Files
When working with this project, always reference these core files first:

```bash
# Essential project understanding
.context/substrate.md           # Methodology and navigation
.context/architecture/overview.md  # System architecture
.context/guidelines.md          # Development standards
```

### Domain-Specific Context
For specific implementation tasks, combine relevant domain files:

```bash
# Authentication tasks
.context/auth/overview.md + .context/auth/security.md + .context/auth/integration.md

# API development
.context/api/endpoints.md + .context/api/examples.md + .context/architecture/patterns.md

# Database work
.context/database/schema.md + .context/database/models.md + .context/database/migrations.md
```

## Agent Prompt Patterns

### Code Generation Prompts
```
Based on the following project documentation:
[Include relevant .context files]

Generate [specific component/feature] following the established patterns, security guidelines, and architectural decisions documented above.

Requirements:
- Follow the coding standards from .context/architecture/patterns.md
- Implement security measures from .context/auth/security.md
- Use the data models defined in .context/database/models.md
```

### Code Review Prompts
```
Review the following code against project standards:
[Include code to review]

Project context:
[Include .context/architecture/patterns.md and .context/guidelines.md]

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
[Include domain-specific .context files]

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
- Auto-suggest patterns from .context files
- Validate against documented standards
- Quick access to domain documentation

### AI Tool Configuration
Set up your AI development tools to:
- Automatically include relevant .context context
- Validate output against project patterns
- Suggest documentation updates for new patterns

---

This guide ensures consistent, context-aware development whether you're an AI agent or human developer working with the Substrate methodology.