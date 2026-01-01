# Prompt: Code Refactoring

## Context Files to Include
```
.context/architecture/overview.md
.context/architecture/patterns.md
.context/anti-patterns.md
.context/ai-rules.md
```

## Prompt

```
Refactor the following code:

**Code Location**: [File path or paste code]

**Current Problems**:
- [Problem 1]
- [Problem 2]

**Refactoring Goals**:
- [ ] Improve readability
- [ ] Reduce complexity
- [ ] Better separation of concerns
- [ ] Improve testability
- [ ] Fix anti-patterns
- [ ] Other: [specify]

**Constraints**:
- [ ] Must maintain backward compatibility
- [ ] Cannot change public API
- [ ] Must keep existing tests passing
- [Other constraints]

Please:
1. Identify specific issues in the current code
2. Propose refactored version following project patterns
3. Explain the improvements made
4. Note any risks or breaking changes
5. Suggest follow-up improvements if needed
```
