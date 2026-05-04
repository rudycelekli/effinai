---
name: quick-fix
description: "Rapid bug fix workflow with minimal, targeted changes"
version: "1.0.0"
tags: [productivity, bugfix, debugging, rapid]
createdBy: "built-in"
status: "active"
---

# Quick Fix

## Activation
When the user reports a bug, error, or unexpected behavior that needs a fast, focused fix.

## Workflow

### Step 1: Reproduce and Understand
1. Get the error message, stack trace, or behavior description
2. Identify the file and line from the stack trace
3. Read the relevant code section
4. Understand what the code SHOULD do vs what it DOES

### Step 2: Root Cause Analysis
Follow the error chain:
1. Read the exact line that throws/fails
2. Check the inputs to that line (are they null, wrong type, unexpected value?)
3. Trace back to where those inputs come from
4. Find the FIRST point where the data goes wrong — that's the root cause

Common root causes:
- **Null/undefined access**: Missing null check or optional chaining
- **Type mismatch**: String where number expected, or vice versa
- **Off-by-one**: Array index, loop boundary, pagination
- **Race condition**: Async operations without proper await
- **Missing import**: Module not imported or wrong path
- **Stale state**: Cached value not updated after change
- **Config mismatch**: Environment variable missing or wrong

### Step 3: Fix with Minimal Changes
1. Make the SMALLEST change that fixes the root cause
2. Do NOT refactor surrounding code (that's a separate task)
3. Do NOT fix other issues you notice (note them but stay focused)
4. Ensure the fix handles edge cases

### Step 4: Verify
1. If tests exist for this code, run them
2. If no test exists, write a minimal regression test
3. Manually verify the fix addresses the original error
4. Check that the fix doesn't break adjacent functionality

### Step 5: Document
Add a brief comment if the fix is non-obvious:
```typescript
// Fix: Handle case where user.profile is null for new accounts
const name = user.profile?.displayName ?? 'Anonymous';
```

## Output Format
```
## Bug Fix

**Root Cause**: [one-line explanation]
**Fix**: [what was changed and why]
**Files Changed**: [list]
**Risk**: Low/Medium — [brief justification]
**Regression Test**: [added/existing/needed]
```

## Rules
- Fix the bug, nothing else — scope discipline is critical
- Always identify root cause, don't just suppress symptoms
- Prefer the smallest change that fully resolves the issue
- Add a regression test to prevent recurrence
- Note other issues found but do NOT fix them in this pass
