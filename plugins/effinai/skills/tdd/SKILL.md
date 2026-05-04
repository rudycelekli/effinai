---
name: tdd
description: "Test-driven development workflow: red-green-refactor cycle"
version: "1.0.0"
tags: [engineering, testing, tdd, quality]
createdBy: "built-in"
status: "active"
---

# Test-Driven Development

## Activation
When the user asks to implement a feature using TDD, or says "test first".

## Workflow: Red-Green-Refactor

### Phase 1: RED — Write Failing Tests
1. Understand the requirement from the user
2. Identify the test file location (follow project conventions)
3. Write the minimal test that describes the desired behavior:
   - One test per behavior, not per function
   - Use descriptive test names: `it('should reject expired tokens')`
   - Include edge cases: null, empty, boundary values
   - Include error cases: what should throw/reject
4. Run the tests — confirm they FAIL (this validates the test is meaningful)

```bash
# Detect test runner
npm test        # or jest, vitest, pytest, go test, etc.
```

### Phase 2: GREEN — Write Minimal Implementation
1. Write the MINIMUM code to make the failing test pass
2. Do NOT add extra functionality beyond what the test requires
3. It's okay if the code is ugly — correctness first
4. Run the tests — confirm they PASS
5. If a test still fails, fix the implementation (not the test)

### Phase 3: REFACTOR — Clean Up
1. Now improve the code while keeping tests green:
   - Extract common logic into helper functions
   - Improve naming and readability
   - Remove duplication
   - Apply SOLID principles where appropriate
2. Run tests after each refactor step — they must stay GREEN
3. Do NOT add new behavior during refactor (that needs a new RED test)

### Cycle Repeat
For each new requirement:
1. Write the next failing test
2. Make it pass with minimal code
3. Refactor
4. Repeat

## Test Quality Checklist
- [ ] Tests are independent (no shared mutable state)
- [ ] Tests are deterministic (no random, no time-dependent)
- [ ] Tests are fast (mock external services)
- [ ] Test names describe behavior, not implementation
- [ ] Arrange-Act-Assert pattern is clear
- [ ] Edge cases covered (null, empty, max, min, duplicate)
- [ ] Error paths tested (invalid input, network failure)

## Rules
- NEVER skip the RED phase — if the test passes immediately, the test is wrong
- NEVER write implementation before tests
- Keep each cycle small — 5-15 minutes max
- Commit after each GREEN phase
- Tests are production code — apply same quality standards
