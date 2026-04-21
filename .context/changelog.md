# Substrate Changelog

Track significant changes to the `.context/` documentation itself.

## How to Use This File

Log changes when:
- Adding new domain files
- Changing methodology
- Updating patterns significantly
- Deprecating approaches

## Format

```markdown
## [Date] - Brief Title

### Added
- New file or section

### Changed
- Modified approach or pattern

### Deprecated
- Approach being phased out

### Removed
- Deleted content
```

---

## [2024-01-15] - Initial Substrate Creation

### Added
- Core methodology in `substrate.md`
- Architecture domain (`architecture/`)
- Authentication domain (`auth/`)
- API domain (`api/`)
- Database domain (`database/`)
- Guidelines (`guidelines.md`)

---

## [2024-01-20] - AI Enhancement

### Added
- `ai-rules.md` - Hard constraints for AI generation
- `glossary.md` - Project terminology
- `anti-patterns.md` - What not to do
- `boundaries.md` - What AI should not modify
- `debt.md` - Technical debt registry
- `decisions/` - Architecture Decision Records

### Changed
- Updated `substrate.md` to reference AI-specific files

---

## [2024-01-25] - Operational Files

### Added
- `prompts/` - Pre-built prompts for common tasks
- `workflows.md` - Step-by-step guides
- `env.md` - Environment variable documentation
- `errors.md` - Error code catalog
- `testing.md` - Testing strategy
- `performance.md` - Performance standards
- `dependencies.md` - Approved packages
- `code-review.md` - Review checklist
- `monitoring.md` - Observability standards
- `events.md` - Domain events catalog
- `feature-flags.md` - Feature flag patterns
- `versioning.md` - API versioning strategy

---

## [2026-04-21] - Research-Backed Pivot

### Added
- `research.md`: documents the four LLM-behavior findings that now constrain the methodology (context length degradation, instruction budget, lost-in-the-middle, path-scoped rules)
- `.claude/rules/` directory with `README.md` and six path-scoped rule files (`api.md`, `auth.md`, `database.md`, `ui.md`, `seo.md`, `testing.md`)
- `decisions/004-research-backed-pivot.md`: ADR capturing the reasoning and migration path

### Changed
- `CLAUDE.md`: rewritten as a lean path-to-rule-file table (~40 lines). Always-loaded content slimmed to project-wide constraints only.
- `AGENTS.md`: reshaped to match Tier 1 model; now documents cross-tool wiring for Cursor, Copilot, Windsurf, generic LLMs.
- `substrate.md`: rewritten to introduce Tier 1 / Tier 2 architecture explicitly; points at `research.md` as the first file to read.
- `README.md`: reframed around the research. New "Why this version is different" section, updated minimal starter, updated FAQ, updated structure diagram.

### Deprecated
- The "more files = better" framing. File count is no longer a methodology target.
- Loading the full domain folder set into AI tool entry points by default.

### Removed
- Nothing. All prior Tier 2 content (domain folders, operational files, prompts) is preserved. It is demoted from always-loaded to on-demand reference, not deleted.

---

## Future Changes

Log template for upcoming changes:

```markdown
## [YYYY-MM-DD] - Title

### Added
-

### Changed
-

### Deprecated
-

### Removed
-
```
