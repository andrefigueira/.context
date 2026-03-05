---
name: generate-tool-config
description: Generate tool-specific configuration files (Cursor rules, Copilot instructions, Windsurf rules) from the .context/ documentation. Use when setting up a project for a specific AI coding tool.
argument-hint: [cursor|copilot|windsurf|all]
disable-model-invocation: true
---

Generate AI tool configuration files from the existing `.context/` documentation.

## Target: $ARGUMENTS

### For "cursor"
Create `.cursor/rules/` directory with `.mdc` files:
- `general.mdc` - Always-apply rules from `@.context/ai-rules.md`
- One `.mdc` per domain with appropriate glob patterns
- Use `alwaysApply: true` only for `general.mdc`
- Reference `.context/` files with `@` syntax

### For "copilot"
Create `.github/copilot-instructions.md`:
- Extract key rules from `.context/ai-rules.md`
- Keep under 50 lines (Copilot works best with concise instructions)
- Reference `.context/` file paths for the agent to read

### For "windsurf"
Create `.windsurfrules`:
- Must stay under 6000 characters
- Prioritize rules from `.context/ai-rules.md` and `.context/architecture/patterns.md`
- Include file references for domain-specific context

### For "all"
Generate all three above.

## Rules
- Read `.context/ai-rules.md` and `.context/architecture/patterns.md` first
- Keep generated files minimal. Don't duplicate content, point to `.context/` files
- Follow each tool's format requirements exactly
- Test that file paths are correct relative to project root
