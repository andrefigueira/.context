# Substrate: Research-Backed Documentation as Context

Entry point for the .context method. This file explains how the documentation system is structured and why.

**Before changing anything here, read [`research.md`](./research.md).** The structure is constrained by measured LLM behavior, not style preference.

## The Two Tiers

The methodology has two tiers. Each tier has a different job.

### Tier 1: Always-Loaded Substrate

Content loaded into every LLM conversation on this project. Optimized for accuracy under the constraints from `research.md`.

**Hard rules for Tier 1:**
- Small total footprint. `CLAUDE.md` <=50 lines, `AGENTS.md` <=90 lines (it carries additional cross-tool wiring)
- Short files. No mid-document drowning
- Path-scoped where possible. Use `.claude/rules/*.md` for rules that only apply to some files
- Instruction count stays inside the ~150-200 budget *in aggregate across loaded files*

**Tier 1 files:**
- `/CLAUDE.md`: entry point for Claude Code (auto-loaded)
- `/AGENTS.md`: entry point for other AI tools
- `/.claude/rules/*.md`: path-scoped rules, loaded only when matching files are touched
- `.context/ai-rules.md`: the minimum set of project-wide hard constraints
- `.context/glossary.md`: terminology, to prevent synonym drift

### Tier 2: Deep Reference

Opt-in content. Loaded *only* when a task explicitly needs it. Can be long, thorough, and cover edge cases.

**Tier 2 is for:**
- Domain deep-dives (full API reference, full schema docs)
- Decision records and rationale
- Historical context (changelog, deprecation notes)
- Onboarding material that a human will read
- Specifications that a task occasionally needs in full

**Tier 2 files in this template:**
- `.context/architecture/`, `.context/auth/`, `.context/api/`, `.context/database/`, `.context/ui/`, `.context/seo/`
- `.context/anti-patterns.md`, `.context/boundaries.md`, `.context/debt.md`
- `.context/workflows.md`, `.context/env.md`, `.context/errors.md`, `.context/testing.md`, `.context/performance.md`
- `.context/decisions/`: Architecture Decision Records
- `.context/prompts/`: pre-built task prompts
- `.context/guidelines.md`, `.context/monitoring.md`, `.context/events.md`, `.context/feature-flags.md`, `.context/versioning.md`, `.context/dependencies.md`, `.context/code-review.md`

Tier 2 files are not *bad* because they are long. They are Tier 2 *because* length is acceptable when loading is selective.

## Loading Model

Every Tier 1 file should answer, in order: which files is the task touching, which Tier 1 rule file covers those, and which Tier 2 files (if any) does this task need in full?

```
Turn begins
  │
  ├─► Always loaded: CLAUDE.md (or AGENTS.md) + ai-rules.md + glossary.md
  │
  ├─► Path match? Load .claude/rules/<domain>.md
  │
  └─► Task needs depth? Load specific Tier 2 file(s) on demand
```

Nothing else is loaded by default. That is the point.

## Research Anchors

The two-tier structure and the loading model above are constrained by four measured LLM failure modes. See [`research.md`](./research.md) for the findings, their implications, and citations. The summary table in the top-level [README](../README.md#why-this-version-is-different) is the short version.

## File Shape

Tier 1 files (short):

```markdown
# Title
One-line purpose.

## Hard rules
Short list. Each rule is one line.

## Path-scoped loads
Which rule file maps to which path.

## On-demand loads
Which Tier 2 file to load for which task.
```

Tier 2 files (long is fine):

```markdown
# Domain: Topic
Brief overview.

## Context
Why this exists.

## Patterns
Concrete implementation.

## Rationale
Why these choices, what was rejected.

## Examples
Real code.
```

## Adding to the Methodology

Before adding a file, answer:

1. **Which tier does it belong in?** If it is loaded every turn, it is Tier 1. Otherwise Tier 2.
2. **If Tier 1, can it be path-scoped?** If yes, it belongs in `.claude/rules/`, not in CLAUDE.md.
3. **If Tier 2, what loads it?** Either a task prompt references it, or a Tier 1 file instructs the agent to load it for specific tasks.
4. **What does it displace?** If adding 500 tokens to Tier 1, what 500 tokens are leaving?

If you cannot answer (4), the addition is probably a mistake.

## Updating Tier 1 Files

When a project convention evolves:
- A hard constraint (always applies) → update `.context/ai-rules.md`
- A path-specific rule → update the matching `.claude/rules/<domain>.md`
- A terminology change → update `.context/glossary.md`
- Deeper reference content → update the Tier 2 file, leave Tier 1 pointers unchanged

Do not add path-specific content to the always-loaded tier. Push it down to `.claude/rules/`.

## Further Reading Inside This Repo

- [`research.md`](./research.md): the evidence
- [`/CLAUDE.md`](../CLAUDE.md): Tier 1 entry for Claude Code
- [`/AGENTS.md`](../AGENTS.md): Tier 1 entry for other AI tools
- [`/.claude/rules/README.md`](../.claude/rules/README.md): how path-scoped rules work
- [`./decisions/004-research-backed-pivot.md`](./decisions/004-research-backed-pivot.md): why the methodology was restructured
