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
