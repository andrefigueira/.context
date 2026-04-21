# Path-Scoped Rules

This directory holds project rules that apply only to specific paths, not to every turn of conversation. The loading pattern is:

- CLAUDE.md (always loaded) names which rule file covers which path
- When the agent reads or edits a file under a matching path, it loads the matching rule file
- Rules that do not match the current task's paths are never loaded

## Why

Most rules are inactive for any given task. A frontend change does not need database migration rules. A docs change does not need any of them. Loading every rule on every turn burns tokens and displaces content that would have helped. Path-scoping reclaims those tokens. See [`.context/research.md`](../../.context/research.md) for the underlying findings.

## How to Use

The authoritative path-to-rule mapping lives in [`CLAUDE.md`](../../CLAUDE.md) at the root of the repo. That table is the source of truth. AGENTS.md mirrors it for non-Claude Code tooling.

Do not duplicate the table here. Edit CLAUDE.md and AGENTS.md instead. Then adjust the `## Applies to` line inside each individual rule file so that the file self-reports its own scope.

## Overlap

Paths can legitimately match more than one rule file. Pages in Next.js Pages Router and App Router match both `seo.md` (metadata, structured data) and `ui.md` (component rules). When multiple files match, load all of them. Each rule file's `## Applies to` section notes any intentional overlap with another file.

## Rule File Shape

Each rule file is short. Target under 80 lines. Structure:

```markdown
# Rule: <domain>
One line purpose.

## Applies to
Path patterns this rule covers.

## Hard rules
Numbered list of one-line constraints.

## Patterns
Code snippets showing the expected shape.

## Load on demand
Tier 2 files to pull in if the task needs depth.
```

## What Does *Not* Belong Here

- Project-wide constraints that apply to every file → `.context/ai-rules.md`
- Terminology → `.context/glossary.md`
- Long-form explanation or rationale → Tier 2 file, referenced from here
- Anti-patterns catalog → `.context/anti-patterns.md`, referenced from here

## Adding a New Rule File

1. Pick a path scope that does not overlap heavily with existing files
2. Write rules in imperative one-liners
3. Add the mapping to CLAUDE.md so the agent knows to load it
4. Add the mapping to AGENTS.md for tool-specific configurations (Cursor globs, Copilot `applyTo`, etc.)

## For Non-Claude Code Tools

Path-scoped rules are not Claude Code specific. Equivalent mechanisms:

- **Cursor**: `.cursor/rules/*.mdc` with `globs:` frontmatter
- **GitHub Copilot**: `.instructions.md` with `applyTo:` frontmatter
- **Windsurf**: `.windsurfrules` (less granular; falls back to task-based loading)

AGENTS.md documents the cross-tool mapping.
