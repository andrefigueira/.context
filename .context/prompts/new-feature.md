# Prompt: New Feature Implementation

## Context Files to Include
```
.context/substrate.md
.context/architecture/overview.md
.context/architecture/patterns.md
.context/ai-rules.md
.context/anti-patterns.md
[Domain-specific files as needed]
```

## Prompt

```
Implement the following feature:

**Feature**: [Name]
**Description**: [What it does and why]

**Requirements**:
1. [Requirement 1]
2. [Requirement 2]
3. [...]

**Affected Areas**:
- [ ] API endpoints
- [ ] Database changes
- [ ] UI components
- [ ] Background jobs
- [ ] Other: [specify]

**Constraints**:
- [Any technical constraints]
- [Any business constraints]

**Out of Scope**:
- [What this does NOT include]

Follow the architecture and patterns in the included context. Before implementing:
1. Identify which layers are affected
2. Plan the changes needed in each layer
3. Consider error handling and edge cases
4. Think about testing approach

Implement incrementally, explaining decisions as you go.
```
