# AI Agents Context Guide

This file is for AI tools that don't auto-discover project context. It explains how to use `.context/` files with your tool.

**Using Claude Code?** It reads `CLAUDE.md` automatically. You don't need this file.

## Tool-Specific Setup

### Cursor

Cursor uses `.cursor/rules/*.mdc` files. Create rules that reference `.context/` files:

```
# .cursor/rules/general.mdc
---
description: Project conventions and constraints
globs: **/*
alwaysApply: true
---

Read and follow the rules in @.context/ai-rules.md
Use terminology from @.context/glossary.md
```

```
# .cursor/rules/api.mdc
---
description: API development patterns
globs: src/api/**/*,src/routes/**/*
---

Follow patterns in @.context/api/endpoints.md
Use error formats from @.context/errors.md
```

```
# .cursor/rules/auth.mdc
---
description: Authentication and security
globs: src/auth/**/*,src/middleware/**/*
---

Follow security model in @.context/auth/security.md
Use auth patterns from @.context/auth/overview.md
```

```
# .cursor/rules/database.mdc
---
description: Database and data models
globs: src/models/**/*,src/db/**/*,migrations/**/*
---

Follow schema in @.context/database/schema.md
Use models from @.context/database/models.md
```

### GitHub Copilot

Create `.github/copilot-instructions.md`:

```markdown
Follow the rules in .context/ai-rules.md.
Use terminology defined in .context/glossary.md.
Follow architecture patterns in .context/architecture/patterns.md.
Avoid anti-patterns listed in .context/anti-patterns.md.
```

For path-specific instructions, create `.instructions.md` files with `applyTo` frontmatter in relevant directories.

### Windsurf

Create `.windsurfrules` in your project root. Windsurf limits rules to 6000 characters per file, 12000 total. Keep it tight:

```
# Project Rules

Follow rules in .context/ai-rules.md.
Use patterns from .context/architecture/patterns.md.
Check .context/anti-patterns.md before writing code.
Use correct terms from .context/glossary.md.

For auth work, read .context/auth/security.md.
For API work, read .context/api/endpoints.md.
For database work, read .context/database/schema.md.
```

### ChatGPT / Generic LLMs

Copy-paste the relevant `.context/` files into your prompt. Use the priority order below to decide what to include.

## What to Include (Priority Order)

When you need to trim context, keep files in this priority:

1. @.context/ai-rules.md - Hard constraints (always include)
2. @.context/glossary.md - Terminology consistency
3. Domain overview for your task (e.g., @.context/auth/overview.md)
4. @.context/architecture/patterns.md - Code patterns
5. @.context/anti-patterns.md - What not to do

## Context Files by Task

**Authentication:** @.context/auth/overview.md, @.context/auth/security.md, @.context/auth/integration.md

**API development:** @.context/api/endpoints.md, @.context/api/examples.md, @.context/architecture/patterns.md

**Database:** @.context/database/schema.md, @.context/database/models.md, @.context/database/migrations.md

**Frontend/UI:** @.context/ui/overview.md, @.context/ui/patterns.md

**SEO:** @.context/seo/overview.md

## Pre-Built Prompts

Ready-to-use prompts in @.context/prompts/:

| Task | File |
|------|------|
| New API endpoint | @.context/prompts/new-endpoint.md |
| New feature | @.context/prompts/new-feature.md |
| Bug fix | @.context/prompts/fix-bug.md |
| Refactor | @.context/prompts/refactor.md |
| Code review | @.context/prompts/review.md |
| Security audit | @.context/prompts/security-audit.md |
| Performance | @.context/prompts/performance.md |
| Documentation | @.context/prompts/documentation.md |

Copy the prompt content, fill in placeholders, and include the `.context/` files each prompt specifies.
