---
name: new-domain
description: Create a new .context domain folder with overview and patterns files following the Substrate methodology. Use when adding a new documentation domain to the project.
argument-hint: [domain-name]
disable-model-invocation: true
---

Create a new `.context/` domain folder for `$ARGUMENTS`.

## Steps

1. Create the directory `.context/$ARGUMENTS/`
2. Create `overview.md` with this structure:

```markdown
# Domain: [Name]
Brief description of what this domain covers.

## Overview
High-level description with context.

## Implementation Patterns
Code examples and standard approaches.

## Decision History & Trade-offs
Why choices were made, alternatives considered.

## Integration Points
How this domain connects to others.
```

3. Create `patterns.md` if the domain has code patterns to document
4. Update `.context/substrate.md` to add the new domain under "Core Domains"
5. Add the domain to `CLAUDE.md` under "Task-Specific Context" if it needs task-specific file pointers
6. Update `AGENTS.md` to add the domain under "Context Files by Task"

## Rules
- Follow existing domain structure (see `.context/auth/` or `.context/api/` as examples)
- Use project-specific content, not generic best practices
- Include decision rationale in every section
- Keep files under 200 lines each
