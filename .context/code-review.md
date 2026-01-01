# Code Review Checklist

Systematic checklist for reviewing code changes. Not every item applies to every change.

## Before Reviewing

- [ ] Understand the purpose (read PR description)
- [ ] Check linked issues/tickets
- [ ] Review at a time you can focus

## Correctness

- [ ] Does the code do what it claims?
- [ ] Are edge cases handled?
- [ ] Are error cases handled?
- [ ] Is the logic correct?
- [ ] Are there off-by-one errors?
- [ ] Are race conditions possible?

## Architecture

- [ ] Is code in the right layer? (handler/service/repository)
- [ ] Does it follow existing patterns?
- [ ] Is responsibility properly separated?
- [ ] Are dependencies injected correctly?
- [ ] Is there unnecessary coupling?

## Security

- [ ] Is input validated?
- [ ] Are authorization checks present?
- [ ] Is sensitive data protected?
- [ ] Are there injection vulnerabilities?
- [ ] Are secrets hardcoded?
- [ ] Is error information leaked?

## Performance

- [ ] Are there N+1 queries?
- [ ] Are lists paginated?
- [ ] Is caching appropriate?
- [ ] Are there expensive operations in hot paths?
- [ ] Could operations be async?

## Testing

- [ ] Are tests included?
- [ ] Do tests cover the change?
- [ ] Are edge cases tested?
- [ ] Are error cases tested?
- [ ] Are tests readable?
- [ ] Could tests be more focused?

## Readability

- [ ] Is the code clear without comments?
- [ ] Are names descriptive?
- [ ] Is complexity justified?
- [ ] Are functions focused (single responsibility)?
- [ ] Is there dead code?

## Documentation

- [ ] Is public API documented?
- [ ] Are complex algorithms explained?
- [ ] Is `.context/` updated if needed?
- [ ] Are breaking changes documented?

## Operations

- [ ] Are new configs documented?
- [ ] Is logging appropriate?
- [ ] Are metrics added if needed?
- [ ] Is monitoring/alerting considered?
- [ ] Is rollback possible?

## How to Give Feedback

### Be Specific
- Bad: "This is confusing"
- Good: "The variable name `x` doesn't indicate it's a user count"

### Explain Why
- Bad: "Use early return"
- Good: "Early return here would reduce nesting and improve readability"

### Suggest, Don't Demand
- Bad: "Change this to X"
- Good: "Consider using X because..."

### Ask Questions
- "What happens if this is null?"
- "Is this intentional?"
- "Could you explain this logic?"

### Acknowledge Good Work
- "Nice refactor here"
- "Good catch on this edge case"
- "Clear and readable"

## Severity Levels

Use these prefixes for clarity:

- `[blocker]` - Must fix before merge
- `[suggestion]` - Would improve but optional
- `[question]` - Need clarification
- `[nit]` - Trivial, don't block

## How to Receive Feedback

- Assume good intent
- Ask for clarification if unclear
- Explain your reasoning
- Be open to alternatives
- Thank reviewers for their time
