# AGENTS.md (For AI Tools Other Than Claude Code)

Research-backed entry point. Research: `.context/research.md`. Full methodology: `.context/substrate.md`.

Claude Code reads `CLAUDE.md` automatically. This file is for Cursor, GitHub Copilot, Windsurf, ChatGPT, and any other AI tool.

## Always loaded

Two files, every time:

- `.context/ai-rules.md`: project-wide hard constraints
- `.context/glossary.md`: terminology

## Path-scoped rules

Load only when touching matching files. Full rule files live in `.claude/rules/`.

This table mirrors `CLAUDE.md`. That file is the source of truth. Keep the two in sync.

When a file matches multiple rows, load every matching rule file.

| Touching | Rule file |
|---|---|
| `src/api/**`, `src/routes/**`, `src/handlers/**`, `app/api/**` | `.claude/rules/api.md` |
| `src/auth/**`, `src/middleware/auth*`, `src/security/**` | `.claude/rules/auth.md` |
| `src/models/**`, `src/db/**`, `src/repositories/**`, `migrations/**`, `*.sql` | `.claude/rules/database.md` |
| `src/components/**`, `app/**` (non-page), `*.tsx`, `*.jsx`, `*.vue`, `*.svelte` | `.claude/rules/ui.md` |
| `src/app/**/page.*`, `src/pages/**`, `public/sitemap*`, `public/robots*`, `src/head/**` | `.claude/rules/seo.md` |
| `**/*.test.*`, `**/*.spec.*`, `tests/**`, `__tests__/**` | `.claude/rules/testing.md` |

## Tool-specific wiring

### Cursor

`.cursor/rules/` with `globs:` frontmatter. Example:

```
# .cursor/rules/api.mdc
---
description: API development patterns
globs: src/api/**,src/routes/**,src/handlers/**,app/api/**
---
@.claude/rules/api.md
```

Repeat for each rule in the table above.

### GitHub Copilot

Create `.github/copilot-instructions.md` for the always-loaded set:

```markdown
Follow rules in .context/ai-rules.md.
Use terminology from .context/glossary.md.
```

For path-scoped rules, create `.github/instructions/<domain>.instructions.md` with `applyTo:` frontmatter pointing at the matching rule file.

### Windsurf

`.windsurfrules` is not path-aware. Keep it tight. Reference the always-loaded set and the most commonly used rule file for this project:

```
Follow .context/ai-rules.md.
Use terminology from .context/glossary.md.
For the domain you are touching, read the matching file under .claude/rules/.
```

### ChatGPT / generic LLM

Paste the always-loaded set, plus the single path-scoped rule file matching the task, plus any on-demand Tier 2 file the task explicitly needs.

Do not paste everything. The research in `.context/research.md` measures why that is worse.

## On-demand (Tier 2)

Load only when the task explicitly needs depth:

- `.context/api/`, `.context/auth/`, `.context/database/`, `.context/ui/`, `.context/seo/`: domain deep-dives
- `.context/anti-patterns.md`, `.context/boundaries.md`, `.context/debt.md`
- `.context/errors.md`, `.context/performance.md`, `.context/testing.md`, `.context/versioning.md`
- `.context/decisions/`: Architecture Decision Records
- `.context/prompts/`: pre-built task prompts

## Why this shape

Long prompts hurt accuracy (13-85%). Models reliably follow ~150-200 instructions, not more. Middle-of-document content gets ignored. Path-scoping cuts rule-token usage per turn sharply because most rules are inactive for most tasks. Evidence in `.context/research.md`.
