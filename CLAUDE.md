# Claude Code Configuration

This repo teaches the **.context method**: documentation-as-code-as-context, tuned to measured LLM behavior. See `.context/research.md` for the evidence.

## Always loaded (Tier 1)

- `.context/ai-rules.md`: project-wide hard constraints
- `.context/glossary.md`: terminology. Do not invent synonyms

## Path-scoped rules (load when touching matching files)

This table is the source of truth. AGENTS.md mirrors it. Keep them in sync.

When a file matches multiple rows, load every matching rule file. Pages matching both `ui.md` (component rules) and `seo.md` (metadata, structured data) is intentional.

| Touching | Load |
|---|---|
| `src/api/**`, `src/routes/**`, `src/handlers/**`, `app/api/**` | `.claude/rules/api.md` |
| `src/auth/**`, `src/middleware/auth*`, `src/security/**` | `.claude/rules/auth.md` |
| `src/models/**`, `src/db/**`, `src/repositories/**`, `migrations/**`, `*.sql` | `.claude/rules/database.md` |
| `src/components/**`, `app/**` (non-page), `*.tsx`, `*.jsx`, `*.vue`, `*.svelte` | `.claude/rules/ui.md` |
| `src/app/**/page.*`, `src/pages/**`, `public/sitemap*`, `public/robots*`, `src/head/**` | `.claude/rules/seo.md` |
| `**/*.test.*`, `**/*.spec.*`, `tests/**`, `__tests__/**` | `.claude/rules/testing.md` |

## On-demand (Tier 2, load when the task asks for depth)

- Boundaries (what requires human review) → `.context/boundaries.md`
- Anti-patterns → `.context/anti-patterns.md`
- Tech debt register → `.context/debt.md`
- ADRs → `.context/decisions/README.md`
- Pre-built task prompts → `.context/prompts/`

## Non-negotiables

- When making an architectural decision, update the relevant file in the same change
- Do not compound items listed in `.context/debt.md`
- For files listed in `.context/boundaries.md`, propose a diff instead of editing

## Why it is shaped this way

Long prompts hurt accuracy (13-85%). Models reliably follow ~150-200 instructions, not more. Middle-of-document content is ignored. Path-scoped rules cut rule-token usage per turn sharply because most rules are inactive for most tasks. Details in `.context/research.md`.
