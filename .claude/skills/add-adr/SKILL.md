---
name: add-adr
description: Create a new Architecture Decision Record in .context/decisions/. Use when documenting an architectural decision.
argument-hint: [title]
disable-model-invocation: true
---

Create a new ADR in `.context/decisions/`.

## Steps

1. Read `.context/decisions/README.md` for the template and existing ADR numbering
2. Determine the next ADR number (check existing files in `.context/decisions/`)
3. Create `.context/decisions/[NNN]-[kebab-case-title].md` using the template from README.md
4. Fill in: Title, Status (Proposed), Context, Decision, Consequences, Alternatives Considered
5. Update `.context/decisions/README.md` index with the new ADR

## Content from user

The user's input describing the decision: $ARGUMENTS

## Rules
- Status should be "Proposed" for new ADRs
- Always document alternatives considered and why they were rejected
- Include concrete consequences (both positive and negative)
- Link to related ADRs if they exist
- Keep the context section focused on the problem, not the solution
