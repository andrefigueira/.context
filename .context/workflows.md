# Development Workflows

Step-by-step guides for common development tasks. Follow these workflows to ensure consistency.

## Adding a New API Endpoint

1. **Define the contract**
   - Add endpoint to `.context/api/endpoints.md`
   - Specify request/response format
   - Document authentication requirements

2. **Create the route**
   - Add route to appropriate router file
   - Apply authentication middleware if needed
   - Apply validation middleware

3. **Implement the handler**
   - Parse and validate input
   - Call service layer
   - Format response
   - Handle errors consistently

4. **Add service logic**
   - Implement business rules
   - Use repository for data access
   - Emit events if needed

5. **Write tests**
   - Unit tests for service logic
   - Integration tests for full endpoint

6. **Update documentation**
   - API docs if external
   - Update `.context/` if patterns changed

## Adding a New Database Table

1. **Design the schema**
   - Add table design to `.context/database/schema.md`
   - Define relationships and constraints
   - Plan indexes for query patterns

2. **Create migration**
   - Create new migration file (never modify existing)
   - Include rollback procedure
   - Test migration locally

3. **Add model/entity**
   - Create model matching schema
   - Add validation rules
   - Document in `.context/database/models.md`

4. **Create repository**
   - Define repository interface
   - Implement data access methods
   - Add to dependency injection

5. **Test**
   - Test migration up/down
   - Test repository methods

## Adding a New Feature

1. **Understand requirements**
   - Clarify scope and acceptance criteria
   - Identify affected areas

2. **Check existing patterns**
   - Read relevant `.context/` files
   - Find similar implementations

3. **Create ADR if needed**
   - Document significant decisions
   - Add to `.context/decisions/`

4. **Implement incrementally**
   - Database changes first
   - Then service layer
   - Then API/UI layer
   - Tests alongside code

5. **Review and update docs**
   - Update affected `.context/` files
   - Add to changelog if significant

## Fixing a Bug

1. **Reproduce reliably**
   - Confirm the bug exists
   - Identify minimal reproduction steps

2. **Find root cause**
   - Don't just fix symptoms
   - Understand why it happens

3. **Write failing test**
   - Test that fails with bug present
   - Will pass when fixed

4. **Fix the bug**
   - Minimal change to fix the issue
   - Follow existing patterns

5. **Verify fix**
   - Test passes
   - Manual verification
   - Check for similar bugs elsewhere

6. **Document if needed**
   - Update docs if behavior unclear
   - Add to anti-patterns if common mistake

## Refactoring

1. **Ensure test coverage**
   - Cannot refactor safely without tests
   - Add tests first if missing

2. **Make changes incrementally**
   - Small commits that work
   - Easy to revert if needed

3. **Run tests frequently**
   - After every change
   - Catch regressions early

4. **Update documentation**
   - Especially if patterns change
   - Update `.context/` files

## Code Review Process

1. **Author prepares**
   - Self-review first
   - Add context in PR description
   - Link to related issues/docs

2. **Reviewer checks**
   - Use `.context/code-review.md` checklist
   - Focus on logic, not style
   - Check against project patterns

3. **Iterate**
   - Discuss, don't dictate
   - Explain reasoning
   - Learn from feedback

4. **Merge**
   - All checks passing
   - Approved by required reviewers
   - Squash if many small commits

## Deployment

1. **Pre-deployment**
   - All tests passing
   - Code reviewed and merged
   - Changelog updated

2. **Deploy to staging**
   - Verify in staging environment
   - Run smoke tests
   - Check logs for errors

3. **Deploy to production**
   - Follow deployment runbook
   - Monitor closely after deploy
   - Be ready to rollback

4. **Post-deployment**
   - Verify key functionality
   - Monitor error rates
   - Communicate completion
