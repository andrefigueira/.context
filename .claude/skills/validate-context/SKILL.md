---
name: validate-context
description: Validate .context/ files for quality, staleness, and conformance to the Substrate methodology. Run after making changes to documentation.
context: fork
agent: Explore
allowed-tools: Read, Grep, Glob, Bash(wc *)
---

Validate the `.context/` documentation for quality and conformance.

## Checks to perform

1. **File size**: Flag any `.context/` file over 200 lines (context bloat risk)
2. **Required sections**: Each domain file should have Overview, Implementation Patterns, and Decision History sections
3. **Broken links**: Check that all markdown links to other `.context/` files point to files that exist
4. **Index sync**: Verify `CLAUDE.md` and `agents.md` reference all files that exist in `.context/`
5. **Stale content**: Flag files that reference patterns or technologies not present in the codebase
6. **ADR completeness**: Check all ADRs have Status, Context, Decision, and Consequences sections

## Output format

Report findings as:

### Passed
- List of checks that passed

### Issues Found
- File: issue description
- Suggested fix

### Recommendations
- Any improvements to documentation quality
