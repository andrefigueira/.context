# Testing Strategy

Testing philosophy, standards, and guidelines for the project.

## Testing Philosophy

1. **Tests are documentation** - They show how code should behave
2. **Test behavior, not implementation** - Tests should survive refactoring
3. **Fast feedback** - Unit tests run in milliseconds
4. **Confidence over coverage** - Cover critical paths, not arbitrary %

## Test Pyramid

```
         /\
        /  \        E2E Tests (few)
       /----\       - Critical user journeys
      /      \      - Slow, expensive
     /--------\     Integration Tests (some)
    /          \    - API endpoints
   /------------\   - Database queries
  /              \  Unit Tests (many)
 /----------------\ - Business logic
                    - Fast, isolated
```

## What to Test

### Must Test
- Business logic and rules
- Edge cases and error conditions
- Security-sensitive code
- Data transformations
- API contracts

### Don't Test
- Framework code
- Third-party libraries
- Simple getters/setters
- Configuration files
- Generated code

## Test Organization

### File Structure
```
tests/
├── unit/           # Unit tests
│   ├── services/
│   └── utils/
├── integration/    # Integration tests
│   ├── api/
│   └── database/
├── e2e/           # End-to-end tests
├── fixtures/      # Test data
└── helpers/       # Test utilities
```

### Naming Conventions
- Test files: `*.test.ts`, `*_test.go`, `test_*.py`
- Test names: Describe the scenario
  - Good: `returns error when email already exists`
  - Bad: `test1`, `testCreate`

## Test Structure

Use **Arrange-Act-Assert** pattern:

```
Arrange: Set up test data and mocks
Act:     Execute the code under test
Assert:  Verify the results
```

## Mocking Guidelines

### When to Mock
- External services (APIs, email, etc.)
- Database (for unit tests)
- Time-dependent operations
- Randomness

### When NOT to Mock
- The code under test
- Simple value objects
- Standard library functions

## Test Data

### Principles
- Tests create their own data
- No shared mutable state
- Clean up after tests
- Use factories for complex objects

### Fixtures
- Use for read-only reference data
- Keep fixtures minimal
- Document fixture purpose

## Integration Tests

### Database Tests
- Use test database
- Run migrations before tests
- Truncate tables between tests
- Use transactions where possible

### API Tests
- Test full request/response cycle
- Include authentication tests
- Test error responses
- Verify response format

## Test Quality Checklist

- [ ] Tests are independent (can run in any order)
- [ ] Tests are deterministic (same result every time)
- [ ] Tests are fast (unit tests < 100ms)
- [ ] Tests have clear names
- [ ] Tests cover happy path and error cases
- [ ] Tests don't duplicate production code logic
- [ ] Mocks are minimal and appropriate

## Continuous Integration

- All tests run on every PR
- Tests must pass before merge
- Coverage reports generated
- Flaky tests are fixed or deleted

## When Tests Fail

1. Read the failure message
2. Reproduce locally
3. Determine if test or code is wrong
4. Fix the root cause
5. Never skip tests without fixing
