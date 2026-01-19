# Test Coverage

Analyze test coverage and generate missing tests:

1. Detect project type and run tests with coverage:
   - Look for package.json, Cargo.toml, go.mod, pyproject.toml, etc.
   - Run the project's test command with coverage enabled

2. Analyze coverage report (format varies by language/tooling)

3. Identify files below 80% coverage threshold

4. For each under-covered file:
   - Analyze untested code paths
   - Generate unit tests for functions
   - Generate integration tests for APIs
   - Generate E2E tests for critical flows

5. Verify new tests pass

6. Show before/after coverage metrics

7. Ensure project reaches 80%+ overall coverage

Focus on:
- Happy path scenarios
- Error handling
- Edge cases (null, undefined, empty, zero values)
- Boundary conditions
