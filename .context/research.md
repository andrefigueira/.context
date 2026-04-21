# Research: Why the .context Method Works the Way It Does

This file documents the research that informs the .context methodology. The patterns here are not aesthetic preferences. They are engineered responses to measured LLM failure modes.

If you are adding to the methodology, changing a default, or making a tradeoff between "more context" and "less context," read this file first.

## The Core Tension

The intuitive move when working with AI tools is to give them everything. Full docs, full rules, full history. If the AI is hallucinating, surely more context will help.

The research says the opposite. Past a surprisingly small budget, added context makes output *worse*, not better. The methodology is designed around that constraint.

## Finding 1: Context Length Alone Hurts Performance

**Claim:** Even when the added tokens are on-topic and relevant, longer inputs degrade task accuracy by 13% to 85%.

**Why it matters for .context:** A 5,900-line substrate loaded into every turn is not "thorough." It is a direct hit on accuracy. The research measures this even when the extra content is *useful in principle*. The problem is not noise, it is length.

**How the methodology responds:**
- Tier 1 (always-loaded) targets a small total footprint
- Tier 2 (deep reference) is opt-in, loaded only for tasks that need it
- Path-scoped rules load only when relevant files are touched

**Reference:** [Context Length Alone Hurts LLM Performance](https://arxiv.org/html/2508.14850v1)

## Finding 2: Instruction Budget is ~150-200

**Claim:** Frontier models reliably follow around 150-200 distinct instructions in a single system prompt. Claude Code's own built-in system prompt uses roughly 50. Beyond the budget, instruction-following degrades non-linearly.

**Why it matters for .context:** A dense `ai-rules.md` with 300+ rules is not a specification, it is a wish list. Most of the rules past the budget will be silently ignored, and you will not see which ones.

**How the methodology responds:**
- CLAUDE.md and AGENTS.md are capped at the lean end of the budget
- Rules are distributed across path-scoped files so any single conversation only sees the subset relevant to the files being touched
- Aggregate rule count is never the target; *active* rule count per task is

**Reference:** [How Many Instructions Can LLMs Follow at Once?](https://arxiv.org/html/2507.11538v1)

## Finding 3: Lost in the Middle

**Claim:** LLM attention follows a U-shaped curve. Information at the start and end of the context window gets attended to. Information in the middle is disproportionately ignored, even when it is the most relevant content in the prompt.

**Why it matters for .context:** A 200-line CLAUDE.md with the most important constraint on line 97 is a trap. The constraint is in the worst possible position. It is present, the model has seen it, and yet it will be ignored more often than the less important rule on line 3 or line 198.

**How the methodology responds:**
- Short files only. Individual always-loaded files kept small enough that there is no "middle"
- Ordering matters: hard constraints go at the top; task-specific loading goes at the bottom; references go where they can be skimmed
- Cross-referencing instead of embedding. Link to deeper content rather than inlining it inside a larger file

**Reference:** [Lost in the Middle](https://arxiv.org/abs/2307.03172)

## Finding 4: Path-Scoped Rules Cut Rule-Token Usage Sharply

**Claim:** Moving rules from a single always-loaded file into path-scoped rule files loaded only when matching files are read or edited reclaims the majority of the tokens that would otherwise be spent on rules the current task does not need. The exact savings depend on project shape (number of domains, overlap between rules), but the direction and scale are consistent across real-world .claude/rules/ setups reported in the Claude Code ecosystem.

Unlike Findings 1-3, this one does not have a single canonical measurement paper. It is a direct consequence of Findings 1 and 2 applied to a partitionable rule set: if most rules are inactive for a given task, and loaded context degrades performance, then not loading the inactive rules is a strict improvement. Community reports of 80%+ token reductions on individual projects are consistent with this, but specific percentages are project-dependent.

**Why it matters for .context:** Most rules are inactive most of the time. A frontend change does not need the database migration rules. A docs change does not need any of them. Loading everything on every turn is the default and it is wasteful. It also compounds Finding 1 and Finding 3: more tokens loaded, more ignored middle.

**How the methodology responds:**
- `.claude/rules/*.md` files with path-scoped applicability
- CLAUDE.md tells the agent which rule file to load based on the files being touched
- Cursor, Copilot, Windsurf, and other tools have equivalent path-scoped mechanisms, documented in AGENTS.md

**Reference:** Consequence of Findings 1 and 2 when applied to a partitionable rule set. Tooling reference: [Claude Code sub-agent and rule patterns](https://docs.claude.com/en/docs/claude-code/sub-agents).

## Design Principles That Fall Out of the Research

1. **Lean defaults, deep on demand.** Nothing in the always-loaded tier that is not needed on every turn.
2. **Short files.** Avoid creating a middle for the model to lose things in.
3. **Path-scope by default.** If a rule only applies to some files, do not make every turn pay for it.
4. **Budget the instructions, not just the tokens.** Count distinct rules. Stay under 150-200 active in any one conversation.
5. **Measure changes.** When adding context, ask what it displaces and whether the task actually needed it.

## When the Research Does Not Apply

- **Research tasks** where completeness matters more than accuracy (one-shot questions, no generation)
- **Human onboarding** where a human will skim and skip; the U-shaped attention curve is an LLM phenomenon
- **Archival / reference** content stored in the repo but not loaded into LLM context

Tier 2 of the methodology serves these cases. Its files can be long and thorough because they are not loaded by default.
