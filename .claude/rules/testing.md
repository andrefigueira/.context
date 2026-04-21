# Rule: Testing

Rules for tests and test infrastructure.

## Applies to
`**/*.test.*`, `**/*.spec.*`, `tests/**`, `__tests__/**`

## Hard rules
1. One assertion focus per test. A test that fails should name the exact thing that broke
2. Tests do not depend on each other's state. Order-independence is a non-negotiable
3. Mock external services (HTTP, cloud APIs, third-party SDKs). Do not hit real services in unit tests
4. Do *not* mock the database in integration tests. Run against a real local instance
5. Test error paths, not just happy paths. Every failure mode in the code has a test
6. No sleeping for timing. Use fake timers or deterministic waits
7. Fixtures are deterministic. Random data uses a seeded RNG
8. A flaky test is a bug, not a nuisance. Fix it or delete it

## Patterns

Unit test structure:

```ts
describe("UserService.create", () => {
  it("rejects duplicate emails", async () => {
    // arrange
    const repo = makeRepo({ existing: [{ email: "a@x" }] });
    const svc = new UserService(repo);
    // act
    await expect(svc.create({ email: "a@x" })).rejects.toThrow(/duplicate/i);
  });
});
```

## Load on demand
- Testing strategy overview → `.context/testing.md`
- Anti-patterns in tests → `.context/anti-patterns.md`
