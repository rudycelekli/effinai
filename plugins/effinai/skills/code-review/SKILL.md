---
name: code-review
description: "Thorough code review with security, performance, and maintainability checks"
version: "1.0.0"
tags: [engineering, review, security, performance]
createdBy: "built-in"
status: "active"
---

# Code Review

## Activation
When the user asks to review code, a PR, or specific files.

## Workflow

### 1. Gather Context
- Identify the files to review (from PR diff, staged changes, or specified paths)
- Read each file fully using the Read tool
- Understand the purpose of the changes from commit messages or user description

### 2. Security Review (OWASP-Aware)
Check for:
- **Injection**: SQL injection, command injection, XSS via unsanitized input
- **Authentication flaws**: Hardcoded secrets, weak token handling, missing auth checks
- **Sensitive data exposure**: Logging secrets, PII in responses, missing encryption
- **Access control**: Missing authorization checks, privilege escalation paths
- **Insecure dependencies**: Known vulnerable packages (check package.json/requirements.txt)
- **Path traversal**: User input in file paths without sanitization

### 3. Performance Review
Check for:
- **N+1 queries**: Database calls inside loops
- **Memory leaks**: Unclosed resources, growing arrays, event listener buildup
- **Unnecessary re-renders**: Missing memoization in React, redundant state updates
- **Algorithm complexity**: O(n^2) or worse where O(n log n) is possible
- **Bundle size**: Importing entire libraries when only a function is needed
- **Caching opportunities**: Repeated expensive computations without caching

### 4. Maintainability Review
Check for:
- **Code duplication**: Repeated logic that should be extracted
- **Naming clarity**: Ambiguous variable/function names
- **Function length**: Functions over 50 lines that should be split
- **Type safety**: Missing types, excessive `any`, unsafe casts
- **Error handling**: Swallowed errors, missing try/catch, unclear error messages
- **Dead code**: Unused imports, unreachable branches, commented-out code

### 5. Architecture Review
Check for:
- **Separation of concerns**: Business logic mixed with presentation/data access
- **Dependency direction**: Lower layers importing from higher layers
- **API contract changes**: Breaking changes to public interfaces
- **Test coverage**: New code without corresponding tests

### 6. Output Format
Present findings as:

```
## Code Review Summary

### Critical (Must Fix)
- [FILE:LINE] Description of issue
  Suggestion: How to fix

### Warnings (Should Fix)
- [FILE:LINE] Description of issue
  Suggestion: How to fix

### Suggestions (Nice to Have)
- [FILE:LINE] Description of improvement

### Positive Notes
- What was done well

### Verdict: APPROVE / REQUEST_CHANGES / NEEDS_DISCUSSION
```

## Rules
- Be specific — always reference file and line number
- Provide fix suggestions, not just complaints
- Acknowledge good patterns alongside issues
- Prioritize security and correctness over style
- If reviewing a PR, use `gh pr diff` to get the changes
