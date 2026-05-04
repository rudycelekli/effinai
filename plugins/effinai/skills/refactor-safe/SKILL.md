---
name: refactor-safe
description: "Safe refactoring with test coverage verification before and after changes"
version: "1.0.0"
tags: [engineering, refactoring, safety, testing]
createdBy: "built-in"
status: "active"
---

# Safe Refactoring

## Activation
When the user asks to refactor code, restructure modules, or improve code quality.

## Workflow

### Step 1: Pre-Flight Checks
1. Identify the files/modules to refactor
2. Run existing tests to establish a baseline:
   ```bash
   npm test 2>&1 | tail -20
   ```
3. Record which tests pass and their count
4. If test coverage tool exists, record coverage percentage:
   ```bash
   npm run test:coverage 2>&1 | tail -30
   ```
5. If NO tests exist for the code being refactored — STOP and write tests first

### Step 2: Catalog Dependencies
1. Find all files that import/reference the code being refactored:
   - Search for imports of the module
   - Search for usage of the functions/classes being changed
2. List all public API surfaces that must remain stable
3. Note any external consumers (other packages, APIs, config files)

### Step 3: Plan the Refactoring
Present the plan to the user before executing:
```
## Refactoring Plan
- Target: [files/modules]
- Type: [rename | extract | inline | restructure | simplify]
- Breaking changes: [none | list them]
- Files affected: [count]
- Risk level: [low | medium | high]
```

### Step 4: Execute Incrementally
Apply refactoring in small, testable steps:

1. **Rename** — Change names one at a time, update all references
2. **Extract** — Pull logic into new function/module, keep old as wrapper first
3. **Move** — Create new location, re-export from old, then update imports
4. **Simplify** — Reduce complexity one function at a time
5. **Delete** — Remove dead code only after confirming zero references

After EACH step:
- Run the test suite
- Verify no regressions
- If tests fail, revert and try a smaller step

### Step 5: Post-Flight Verification
1. Run full test suite — all previously passing tests must still pass
2. Run coverage — coverage must not decrease
3. Run type checker if applicable: `npx tsc --noEmit`
4. Run linter: `npm run lint`
5. Create a summary of changes

## Refactoring Catalog

| Pattern | When to Use |
|---------|------------|
| Extract Function | Function > 20 lines, or reusable logic |
| Extract Module | File > 300 lines, or mixed concerns |
| Rename | Name doesn't match purpose |
| Inline | Wrapper adds no value |
| Replace Conditional with Polymorphism | Complex if/switch on type |
| Introduce Parameter Object | Function has 4+ parameters |
| Replace Magic Number | Literal values without context |

## Rules
- NEVER refactor without passing tests as a safety net
- NEVER change behavior during refactoring — that's a feature/bugfix
- Keep each commit focused on ONE refactoring type
- If you discover a bug during refactoring, note it but fix it in a separate step
- Prefer automated refactoring tools (IDE rename, etc.) when available
