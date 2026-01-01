# Architecture Decision Records

This folder contains Architecture Decision Records (ADRs) documenting significant technical decisions made in this project.

## What is an ADR?

An ADR captures a single decision, its context, and consequences. ADRs are immutable once accepted - if a decision changes, create a new ADR that supersedes the old one.

## ADR Template

```markdown
# ADR-XXX: [Title]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-XXX]

## Context
What is the issue that we're seeing that is motivating this decision?

## Decision
What is the change that we're proposing and/or doing?

## Consequences
What becomes easier or more difficult to do because of this change?

### Positive
- Benefit 1
- Benefit 2

### Negative
- Drawback 1
- Drawback 2

### Neutral
- Side effect 1
```

## Index

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [001](./001-jwt-authentication.md) | Use JWT for Authentication | Accepted | 2024-01-15 |
| [002](./002-repository-pattern.md) | Implement Repository Pattern | Accepted | 2024-01-15 |
| [003](./003-postgresql-database.md) | Use PostgreSQL as Primary Database | Accepted | 2024-01-15 |

## Creating a New ADR

1. Copy the template above
2. Number sequentially (next: ADR-004)
3. Fill in all sections
4. Submit via PR for team review
5. Update the index above

## When to Write an ADR

Write an ADR when:
- Choosing between multiple valid technical approaches
- Adopting a new library, framework, or tool
- Changing an existing architectural pattern
- Making a decision that will be hard to reverse
- A decision needs to be explained to future developers

Don't write an ADR for:
- Obvious or trivial choices
- Decisions that can easily be changed later
- Style preferences covered by linting rules
