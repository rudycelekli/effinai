---
name: test-generator
description: "Auto-generate tests from code analysis — unit tests, integration tests, edge cases, and coverage gap detection using structural understanding"
version: "1.0.0"
tags: [testing, generation, coverage, tdd, unit-tests, quality]
---

# Test Generator

## Activation
When the agent needs to generate tests for existing code, identify untested paths, fill coverage gaps, or bootstrap a test suite for a module. Use after writing new code, during refactoring, or when coverage analysis reveals gaps.

## Concept

Analyzes source code structure (functions, classes, branches, error paths) to automatically generate meaningful tests. Goes beyond template-based generation by understanding the code's behavior: input types, return values, side effects, error conditions, and boundary cases. Generates tests in the project's existing test framework and style.

## Analysis Phase

### Code Structure Extraction
For each function/method under test, extract:

```
Function Profile:
  name: "authenticateUser"
  parameters:
    - name: "email", type: "string", nullable: false
    - name: "password", type: "string", nullable: false
  return_type: "Promise<AuthResult>"
  throws: ["InvalidCredentialsError", "AccountLockedError"]
  side_effects: ["database read", "session creation", "audit log write"]
  branches: 5
  complexity: 12
  dependencies: ["UserRepository", "PasswordHasher", "SessionManager"]
```

### Path Analysis
Identify all execution paths through the code:

| Path Type | Description | Test Priority |
|-----------|-------------|---------------|
| **Happy path** | Normal successful execution | High |
| **Error path** | Each throw/reject/error return | High |
| **Null/undefined** | Nullable parameter handling | High |
| **Boundary** | Edge values (empty string, 0, MAX_INT) | Medium |
| **Type coercion** | Unexpected input types | Medium |
| **Async** | Promise rejection, timeout, race conditions | High |
| **State** | Different initial states leading to different outcomes | Medium |

## Generation Strategies

### 1. London School (Mock-First)
```
For each function:
1. Mock all dependencies
2. Test behavior through assertions on:
   - Return values
   - Mock call expectations (what was called, with what args)
   - Side effect verification
3. One test per behavior, not per code path

Example output:
  describe("authenticateUser")
    it("returns auth token for valid credentials")
    it("calls UserRepository.findByEmail with the provided email")
    it("throws InvalidCredentialsError for wrong password")
    it("throws AccountLockedError after 5 failed attempts")
    it("creates a session via SessionManager")
    it("writes to audit log on successful login")
```

### 2. Chicago School (State-Based)
```
For each function:
1. Set up real dependencies (or lightweight fakes)
2. Execute the function
3. Assert on final state, not intermediate calls

Example output:
  describe("authenticateUser")
    it("creates a valid session in the database")
    it("increments failed_attempts count on wrong password")
    it("resets failed_attempts on successful login")
```

### 3. Property-Based
```
For functions with mathematical properties:
1. Identify invariants (e.g., "encrypt then decrypt yields original")
2. Generate random inputs
3. Assert properties hold for all inputs

Example output:
  property("hashPassword produces different hashes for different passwords")
  property("validateToken accepts tokens created by createToken")
```

## Coverage Gap Detection

### Analysis
```
1. Parse existing test files
2. Map tested functions/methods
3. Compare against all exported functions
4. Identify:
   - Completely untested functions
   - Partially tested functions (missing error paths)
   - Uncovered branches within tested functions
   - Missing edge case tests
```

### Gap Report
```
Coverage Gaps:
  src/auth/middleware.ts:
    authenticateRequest:
      MISSING: error path when token is expired
      MISSING: boundary case for empty Authorization header
      MISSING: test for concurrent session limit
    refreshToken:
      UNTESTED: entire function has no tests
    
  src/auth/service.ts:
    createUser:
      MISSING: duplicate email handling
      MISSING: password validation edge cases

Priority order:
  1. refreshToken (0% coverage, high complexity)
  2. authenticateRequest error paths (partial coverage)
  3. createUser edge cases (low risk)
```

## Test Quality Checks

### Generated Test Validation
Each generated test is checked for:

| Check | Description |
|-------|-------------|
| **Deterministic** | No random values without seeds, no time-dependent assertions |
| **Isolated** | Each test can run independently, no shared mutable state |
| **Fast** | No unnecessary I/O, network calls, or sleeps |
| **Readable** | Clear test name describes the behavior being tested |
| **Single assertion** | Each test verifies one behavior (may use multiple asserts) |
| **Arrange-Act-Assert** | Clear structure with setup, execution, and verification |

### Anti-Patterns Avoided
- Testing implementation details instead of behavior
- Brittle assertions on exact string matches when pattern would suffice
- Tests that pass but don't actually verify anything meaningful
- Over-mocking that makes tests meaningless

## Framework Detection

Auto-detects the project's test framework and generates matching code:

| Language | Frameworks Detected |
|----------|-------------------|
| TypeScript/JavaScript | Jest, Vitest, Mocha, Playwright, Cypress |
| Python | pytest, unittest |
| Rust | built-in #[test], proptest |
| Go | testing, testify |
| Java | JUnit, TestNG, Mockito |

## Workflow

### Generate Tests for a File
```
1. Read the source file
2. Extract function profiles and dependencies
3. Analyze execution paths
4. Detect existing test file and framework
5. Generate tests for uncovered paths
6. Write test file (or append to existing)
7. Run tests to verify they pass
8. Report coverage improvement
```

### Fill Coverage Gaps
```
1. Run coverage analysis (if available)
2. Parse coverage report
3. Identify gaps by priority
4. Generate tests for top-priority gaps
5. Run and verify
6. Report new coverage percentage
```

## Integration

- **Knowledge Graph**: Use dependency information to identify what to mock
- **Agent Grep**: Find existing test patterns in the codebase to match style
- **Self-Dev**: Generate tests for self-modifications before promoting changes
- **Code Review**: Suggest missing tests during PR review
