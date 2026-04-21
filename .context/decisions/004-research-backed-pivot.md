# ADR-004: Pivot to Research-Backed Two-Tier Methodology

## Status
Accepted, 2026-04-21

## Context

The original .context methodology treated documentation volume as the primary lever. The assumption was that richer, more structured context would produce better AI output. The template shipped with ~47 files and encouraged always-loading substrate content into AI tool entry points (CLAUDE.md, AGENTS.md).

Subsequent research measures this assumption as wrong past a small budget:

1. **Context Length Alone Hurts LLM Performance**: longer inputs degrade task accuracy by 13-85%, even when the added content is on-topic and relevant ([arxiv.org/html/2508.14850v1](https://arxiv.org/html/2508.14850v1)).
2. **How Many Instructions Can LLMs Follow at Once?**: frontier models reliably follow around 150-200 distinct instructions. Beyond that, instruction-following degrades non-linearly ([arxiv.org/html/2507.11538v1](https://arxiv.org/html/2507.11538v1)).
3. **Lost in the Middle**: LLM attention follows a U-shape. Mid-document content is disproportionately ignored, independent of relevance ([arxiv.org/abs/2307.03172](https://arxiv.org/abs/2307.03172)).
4. **Path-scoped rules recover a large fraction of rule tokens**: most rules are inactive for any single task. Scoping them to paths that the task is actually touching reclaims the majority of the tokens a monolithic rule file would spend.

The original methodology is contradicted on all four findings simultaneously. Keeping it as the default would mean shipping a practice that its own cited research refutes.

## Decision

Restructure the methodology as two explicit tiers:

**Tier 1 (Always Loaded)**
- `CLAUDE.md` (<=50 lines) and `AGENTS.md` (<=90 lines): lean, path-scoped rule tables
- `.context/ai-rules.md`: project-wide hard constraints only
- `.context/glossary.md`: terminology
- `.claude/rules/<domain>.md`: path-scoped rules, loaded only when matching files are touched

**Tier 2 (Loaded On Demand)**
- `.context/<domain>/` deep-dive docs (architecture, auth, api, database, ui, seo)
- `.context/anti-patterns.md`, `.context/boundaries.md`, `.context/debt.md`
- `.context/errors.md`, `.context/performance.md`, `.context/testing.md`, etc.
- `.context/decisions/` (ADRs)
- `.context/prompts/` (pre-built task prompts)

Tier 2 content remains in the template. It is not deleted. It is demoted from the always-loaded default to on-demand reference, pulled into conversations only when a task explicitly requires it.

Entry-point files (`CLAUDE.md`, `AGENTS.md`) now carry a path-to-rule-file mapping table. The agent is expected to load the matching rule file when editing files under the matching path, rather than loading everything up front.

## Consequences

### Positive
- Measurable alignment with the research: smaller always-loaded footprint, tighter instruction budget, shorter files (avoiding lost-in-the-middle), path-scoped loading
- Tier 2 reference remains available. No loss of depth, only a change in when it loads
- Cross-tool consistency: Cursor, Copilot, and Windsurf all have native path-scoped equivalents that can reference the same `.claude/rules/` files
- Reduced token cost per turn for the same effective coverage
- Clearer mental model for maintainers: when adding content, the question "which tier?" has a definite answer

### Negative
- Migration cost for existing users: existing CLAUDE.md files typically need to be slimmed and path-scoped rules extracted
- Risk that users misinterpret the pivot as deprecating the domain folders; they are not deprecated, only demoted to Tier 2
- Increased coordination: path globs in CLAUDE.md, AGENTS.md, Cursor config, Copilot config, and the rule files themselves must stay in sync
- Contradicts earlier public framing of the methodology; the README and substrate files carry explanatory weight for returning users

### Neutral
- Existing `.context/` domain documentation is preserved verbatim but relabeled as Tier 2
- The "47 files" framing is retired; file count is no longer a methodology target
- The pre-built prompts in `.context/prompts/` take on more importance as the canonical way to pull in Tier 2 content for complex multi-domain tasks

## Migration

For projects using the older .context methodology:

1. Rewrite `CLAUDE.md` to the research-backed shape (see this repo's `CLAUDE.md` for the template): a path-to-rule-file table plus a short "always loaded" and "on-demand" list.
2. Create `.claude/rules/<domain>.md` files for each domain. Extract path-specific rules from the old monolithic `ai-rules.md` into these files.
3. Leave `.context/` domain folders in place. They are Tier 2 now, loaded on demand via task prompts or explicit references.
4. Audit aggregate rule count across Tier 1: if any single path-match loads >150 distinct rules, split the domain further.
5. Update AGENTS.md and any Cursor / Copilot / Windsurf config to reference the new rule files.

## References

- `.context/research.md`: full discussion of the four findings and their methodological implications
- `.context/substrate.md`: updated entry point explaining the two-tier architecture
- `/.claude/rules/README.md`: path-scoped rules spec and file shape
