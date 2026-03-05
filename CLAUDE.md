# Claude Code Configuration

This repository teaches the **.context method** - documentation-as-code-as-context. The documentation lives in `.context/`. This file tells Claude Code how to use it.

## Before Generating Code

1. Read `.context/ai-rules.md` first - hard constraints that override everything
2. Read domain-specific files relevant to the task (see below)
3. Follow patterns in `.context/architecture/patterns.md`

## Task-Specific Context

**Authentication work:**
Read `.context/auth/overview.md`, `.context/auth/security.md`, `.context/auth/integration.md`

**API development:**
Read `.context/api/endpoints.md`, `.context/api/examples.md`, `.context/architecture/patterns.md`

**Database work:**
Read `.context/database/schema.md`, `.context/database/models.md`

**Frontend/UI work:**
Read `.context/ui/overview.md`, `.context/ui/patterns.md`

**SEO implementation:**
Read `.context/seo/overview.md`

## Key Constraints

- Use terminology from `.context/glossary.md` - do not invent synonyms
- Check `.context/anti-patterns.md` before writing code
- Respect boundaries in `.context/boundaries.md` - some files require human review
- Don't compound debt listed in `.context/debt.md`
- When making architectural decisions, update relevant `.context/` files in the same change
- ADR template and index live in `.context/decisions/README.md`

## Pre-Built Prompts

Task-specific prompts in `.context/prompts/`: new-endpoint, new-feature, fix-bug, refactor, review, security-audit, performance, documentation.
