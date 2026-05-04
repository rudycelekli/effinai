---
name: test-coverage
description: "Analyze and improve test coverage with targeted test generation"
version: "1.0.0"
tags: [quality, testing, coverage, analysis]
createdBy: "built-in"
status: "active"
---

# Test Coverage

## Activation
When the user asks to improve test coverage, find untested code, or analyze testing gaps.

## Workflow

### Step 1: Measure Current Coverage
```bash
# Node.js (Jest/Vitest)
npx jest --coverage --coverageReporters=text 2>&1 | tail -30
# or
npx vitest run --coverage 2>&1 | tail -30

# Python
python -m pytest --cov=src --cov-report=term-missing 2>&1 | tail -30
```

### Step 2: Identify Gaps
From the coverage report, find:
1. **Uncovered files** (0% coverage) — highest priority
2. **Low coverage files** (<50%) — need attention
3. **Uncovered branches** — conditional logic not tested
4. **Untested error paths** — catch blocks, error handlers

Prioritize by risk:
| Priority | Code Type | Why |
|----------|-----------|-----|
| P0 | Authentication/authorization | Security-critical |
| P0 | Payment/financial logic | Business-critical |
| P1 | Data validation | Data integrity |
| P1 | API endpoints | User-facing |
| P2 | Business logic | Core functionality |
| P3 | Utility functions | Low risk |
| P4 | UI components | Visual, lower risk |

### Step 3: Generate Tests for Gaps

For each uncovered function/module:

1. **Read the source code** to understand behavior
2. **Identify test cases**:
   - Happy path (normal input, expected output)
   - Edge cases (empty, null, boundary values)
   - Error cases (invalid input, failures)
   - Integration points (external calls mocked)
3. **Write tests** following project conventions:
   - Same test framework already in use
   - Same assertion style
   - Same mocking approach
   - File naming convention (*.test.ts, *.spec.ts)

### Step 4: Test Quality Check
Generated tests should:
- [ ] Test behavior, not implementation details
- [ ] Be independent (no test depends on another)
- [ ] Be deterministic (no flaky tests)
- [ ] Use meaningful assertions (not just "doesn't throw")
- [ ] Mock external dependencies (DB, API, filesystem)
- [ ] Cover both success and failure paths

### Step 5: Re-Measure
Run coverage again after adding tests to confirm improvement:
```bash
npx jest --coverage --coverageReporters=text 2>&1 | tail -30
```

Report the delta:
```
## Coverage Improvement
| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| Statements | 45% | 72% | +27% |
| Branches | 30% | 65% | +35% |
| Functions | 50% | 80% | +30% |
| Lines | 45% | 71% | +26% |
```

## Rules
- Focus on meaningful tests, not just hitting coverage numbers
- Prioritize business-critical and security-critical code
- Don't test trivial getters/setters just for coverage
- Use the project's existing test patterns and conventions
- Place test files according to project convention (colocated or test/ dir)
- Mock external dependencies, don't test third-party code
