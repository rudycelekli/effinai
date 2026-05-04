---
name: pr-review
description: "Structured, systematic code review for GitHub PRs and GitLab MRs. Performs blast radius analysis, security scanning, breaking change detection, test coverage delta, and produces a reviewer-ready report with a 30+ item checklist."
version: "1.0.0"
tags: [code-review, pull-request, security, testing, quality-assurance]
createdBy: "built-in"
status: "active"
---

# PR Review Expert

## Overview

Structured, systematic code review for GitHub PRs and GitLab MRs. Goes beyond style nits — performs blast radius analysis, security scanning, breaking change detection, and test coverage delta calculation. Produces a reviewer-ready report with prioritized findings.

## Activation

When the user asks to review pull requests, analyze code changes, check for security issues in PRs, or assess code quality of diffs.

## Workflow

### Step 1 — Fetch Context

```bash
PR=123
gh pr view $PR --json title,body,labels,milestone,assignees | jq .
gh pr diff $PR --name-only
gh pr diff $PR > /tmp/pr-$PR.diff
```

### Step 2 — Blast Radius Analysis

For each changed file, identify:

1. **Direct dependents** — who imports this file?
2. **Service boundaries** — does this change cross a service?
3. **Shared contracts** — types, interfaces, schemas

**Blast radius severity:**
- CRITICAL — shared library, DB model, auth middleware, API contract
- HIGH — service used by >3 others, shared config, env vars
- MEDIUM — single service internal change, utility function
- LOW — UI component, test file, docs

### Step 3 — Security Scan

```bash
# SQL Injection — raw query string interpolation
grep -n "query\|execute\|raw(" $DIFF | grep -E '\$\{|f"|%s|format\('

# Hardcoded secrets
grep -nE "(password|secret|api_key|token)\s*=\s*['\"][^'\"]{8,}" $DIFF

# XSS vectors
grep -n "dangerouslySetInnerHTML\|innerHTML\s*=" $DIFF

# eval / exec
grep -nE "\beval\(|\bexec\(" $DIFF

# Path traversal
grep -nE "path\.join\(.*req\.|readFile\(.*req\." $DIFF
```

### Step 4 — Test Coverage Delta

```bash
CHANGED_SRC=$(gh pr diff $PR --name-only | grep -vE "\.test\.|\.spec\.|__tests__")
CHANGED_TESTS=$(gh pr diff $PR --name-only | grep -E "\.test\.|\.spec\.|__tests__")
echo "Source files: $(echo "$CHANGED_SRC" | wc -w)"
echo "Test files:   $(echo "$CHANGED_TESTS" | wc -w)"
```

**Coverage rules:**
- New function without tests -> flag
- Deleted tests without deleted code -> flag
- Coverage drop >5% -> block merge
- Auth/payments paths -> require 100% coverage

### Step 5 — Breaking Change Detection

- **API**: REST route removals, GraphQL schema removals, TypeScript interface removals
- **DB**: Destructive operations (DROP TABLE/COLUMN, ALTER NOT NULL), index removals
- **Config**: New env vars in code (might be missing in prod), removed env vars

### Step 6 — Performance Impact

- N+1 query patterns (DB calls inside loops)
- Heavy new dependencies
- Unbounded loops, missing await, large in-memory allocations

## Complete Review Checklist (30+ Items)

### Scope & Context
- [ ] PR title accurately describes the change
- [ ] PR description explains WHY, not just WHAT
- [ ] Linked ticket exists and matches scope
- [ ] No unrelated changes (scope creep)
- [ ] Breaking changes documented

### Blast Radius
- [ ] All files importing changed modules identified
- [ ] Cross-service dependencies checked
- [ ] Shared types/interfaces reviewed for breakage
- [ ] New env vars documented in .env.example
- [ ] DB migrations are reversible

### Security
- [ ] No hardcoded secrets or API keys
- [ ] SQL queries use parameterized inputs
- [ ] User inputs validated/sanitized
- [ ] Auth/authorization checks on new endpoints
- [ ] No XSS vectors
- [ ] New dependencies checked for CVEs
- [ ] No sensitive data in logs
- [ ] CORS configured correctly

### Testing
- [ ] New public functions have unit tests
- [ ] Edge cases covered (empty, null, max values)
- [ ] Error paths tested
- [ ] Integration tests for API changes
- [ ] Test names clearly describe what they verify

### Breaking Changes
- [ ] No API endpoints removed without deprecation
- [ ] No required fields added to existing responses
- [ ] No DB columns removed without two-phase migration
- [ ] Backward-compatible for external consumers

### Performance
- [ ] No N+1 query patterns
- [ ] DB indexes added for new query patterns
- [ ] No unbounded loops on large datasets
- [ ] Async operations correctly awaited
- [ ] Caching considered for expensive operations

### Code Quality
- [ ] No dead code or unused imports
- [ ] Error handling present (no empty catch blocks)
- [ ] Consistent with existing patterns
- [ ] Complex logic has comments

## Output Format

```
## PR Review: [PR Title] (#NUMBER)

Blast Radius: HIGH — changes lib/auth used by 5 services
Security: 1 finding (medium severity)
Tests: Coverage delta +2%
Breaking Changes: None detected

--- MUST FIX (Blocking) ---
1. [Finding with file:line and fix suggestion]

--- SHOULD FIX (Non-blocking) ---
2. [Finding]

--- SUGGESTIONS ---
3. [Improvement idea]

--- LOOKS GOOD ---
- [Positive observations]
```

## Best Practices

1. Read the linked ticket before looking at code — context prevents false positives
2. Check CI status before reviewing — don't review code that fails to build
3. Prioritize blast radius and security over style
4. Label each comment: "nit:", "must:", "question:", "suggestion:"
5. Batch all comments in one review round
6. Acknowledge good patterns, not just problems
