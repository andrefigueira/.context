# Prompt: Code Review

## Context Files to Include
```
.context/ai-rules.md
.context/anti-patterns.md
.context/architecture/patterns.md
.context/code-review.md
```

## Prompt

```
Review the following code changes:

**Files Changed**:
[List files or paste diff]

**Purpose of Changes**:
[What these changes accomplish]

Review for:

1. **Correctness**
   - Does it do what it's supposed to?
   - Are there logic errors?
   - Edge cases handled?

2. **Architecture**
   - Follows project patterns?
   - Right layer for the logic?
   - Proper separation of concerns?

3. **Security**
   - Input validation?
   - Authorization checks?
   - Data exposure risks?

4. **Performance**
   - Obvious inefficiencies?
   - N+1 queries?
   - Memory issues?

5. **Maintainability**
   - Clear and readable?
   - Well-named?
   - Appropriate complexity?

6. **Testing**
   - Testable design?
   - Tests included?
   - Good coverage?

Provide:
- List of issues (critical/major/minor)
- Specific suggestions for each
- Any positive observations
```
