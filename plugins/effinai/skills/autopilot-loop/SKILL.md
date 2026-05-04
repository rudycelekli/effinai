---
name: autopilot-loop
description: "Autonomous coding loop that picks tasks, implements in branches, runs quality gates, and commits"
version: "1.0.0"
tags: [automation, autonomous, ci, coding-loop]
---

# Autopilot Loop

An autonomous execution loop that picks the highest priority task, implements it in an isolated branch, runs quality gates, and commits or retries until all tasks are complete.

## Loop Overview

```
[Start] --> [Pick Task] --> [Create Branch] --> [Implement] --> [Quality Gates]
                ^                                                    |
                |                                              Pass? |
                |                                            /       \
                |                                         Yes         No
                |                                          |           |
                |                                      [Commit]   [Retry?]
                |                                          |       /     \
                |                                          |     Yes      No
                |                                          |      |       |
                +------------------------------------------+------+   [Skip + Log]
                                                                      |
                                                               [Next Task]
```

## Task Selection

Pick the next task based on priority scoring:

```typescript
interface AutopilotTask {
  id: string;
  title: string;
  description: string;
  priority: "critical" | "high" | "medium" | "low";
  type: "bug-fix" | "feature" | "refactor" | "test" | "chore";
  estimatedComplexity: 1 | 2 | 3 | 5 | 8;  // Fibonacci
  dependencies: string[];    // Task IDs that must complete first
  status: "queued" | "in-progress" | "passed" | "failed" | "skipped";
  retryCount: number;
  maxRetries: number;        // Default: 2
}

function selectNextTask(tasks: AutopilotTask[]): AutopilotTask | null {
  return tasks
    .filter(t => t.status === "queued")
    .filter(t => t.dependencies.every(d => getTask(d).status === "passed"))
    .sort((a, b) => {
      const priorityOrder = { critical: 0, high: 1, medium: 2, low: 3 };
      if (priorityOrder[a.priority] !== priorityOrder[b.priority]) {
        return priorityOrder[a.priority] - priorityOrder[b.priority];
      }
      return a.estimatedComplexity - b.estimatedComplexity; // Smaller first
    })[0] ?? null;
}
```

**Selection rules**:
1. Critical before high before medium before low
2. Among same priority, smaller tasks first (quick wins build momentum)
3. Respect dependency ordering -- never start a task whose dependencies haven't passed
4. Skip tasks that have exceeded max retries

## Branch Strategy

Each task gets its own branch for isolation:

```bash
# Branch naming convention
git checkout -b autopilot/{task-type}/{task-id}

# Examples
git checkout -b autopilot/bug-fix/fix-auth-timeout
git checkout -b autopilot/feature/add-rate-limiter
git checkout -b autopilot/refactor/extract-payment-service
```

**Rules**:
- Always branch from the latest main/develop
- Never commit directly to main
- If the branch already exists (retry scenario), reset to latest main first
- Clean up merged branches after task completes

## Implementation Phase

The agent implements the task following trajectory patterns if available:

```bash
# Check for matching trajectory patterns
npx @claude-flow/cli@latest memory search \
  --query "{task description}" \
  --namespace trajectory-patterns

# If pattern found, follow its step sequence
# If no pattern, use default approach:
#   1. Analyze requirements
#   2. Read relevant code
#   3. Implement changes
#   4. Write/update tests
```

**Time budget**: Each task has a time limit based on complexity:

| Complexity | Time Budget | Token Budget |
|-----------|-------------|--------------|
| 1 | 5 minutes | $1.00 |
| 2 | 10 minutes | $2.00 |
| 3 | 20 minutes | $4.00 |
| 5 | 35 minutes | $7.00 |
| 8 | 60 minutes | $12.00 |

If the budget is exceeded, the task is marked for retry or skip.

## Quality Gates

Every implementation must pass ALL gates before committing:

### Gate 1: Type Check
```bash
npx tsc --noEmit
# OR for Python: mypy src/
# OR for Go: go vet ./...
```
Pass criteria: Zero type errors.

### Gate 2: Lint
```bash
npx eslint . --max-warnings 0
# OR: ruff check .
# OR: golangci-lint run
```
Pass criteria: Zero errors, zero warnings.

### Gate 3: Unit Tests
```bash
npx vitest run --coverage
# OR: pytest --cov
# OR: go test ./... -cover
```
Pass criteria: All tests pass. Coverage on changed files >= 80%.

### Gate 4: Integration Tests (if applicable)
```bash
npx vitest run --config vitest.integration.config.ts
```
Pass criteria: All integration tests pass.

### Gate 5: Security Scan
```bash
npx @claude-flow/cli@latest security scan --depth quick
```
Pass criteria: No critical or high vulnerabilities introduced.

## Gate Results and Actions

```typescript
type GateResult = "pass" | "fail" | "skip";

interface QualityReport {
  typeCheck: GateResult;
  lint: GateResult;
  unitTests: GateResult;
  integrationTests: GateResult;
  securityScan: GateResult;
  overallPass: boolean;       // All required gates passed
  failures: string[];         // Details of what failed
}
```

### On All Gates Pass

```bash
# Stage and commit changes
git add -A
git commit -m "autopilot({task-type}): {task-title}

Automated implementation by autopilot loop.
Task: {task-id}
Quality gates: all passed"

# Update task status
# Record successful trajectory
```

### On Gate Failure (Retry)

```
Attempt 1 failed: lint errors in src/auth.ts (3 warnings)
  -> Auto-fix: run eslint --fix, re-check
  -> Re-run quality gates

Attempt 2 failed: test failure in auth.test.ts
  -> Analyze failure, adjust implementation
  -> Re-run quality gates

Attempt 3 (max retries): still failing
  -> Mark task as "failed"
  -> Log failure details for human review
  -> Move to next task
```

### On Skip

Tasks are skipped (not retried) when:
- The error is outside the task's scope (pre-existing test failures)
- The time/token budget is exceeded
- A dependency task failed

## Progress Tracking

```
=== Autopilot Progress ===
Session: autopilot-2025-03-15-001
Duration: 45 minutes
Budget used: $18.40 / $50.00

Tasks:
  [PASSED]  fix-auth-timeout       | 3min  | $0.85  | 1 attempt
  [PASSED]  add-rate-limiter       | 12min | $3.20  | 1 attempt
  [PASSED]  extract-payment-svc    | 18min | $5.40  | 2 attempts
  [FAILED]  migrate-user-schema    | 8min  | $4.10  | 3 attempts (max retries)
  [QUEUED]  add-webhook-support    | --    | --     | blocked by migrate-user-schema
  [QUEUED]  update-api-docs        | --    | --     | pending

Completed: 3/6 (50%)
Failed: 1/6 (blocked downstream: 1)
Remaining: 2/6
```

## Loop Termination

The autopilot loop stops when:

1. **All tasks completed** (passed or skipped) -- normal termination
2. **Budget exhausted** -- daily/session budget hit
3. **Time limit reached** -- configurable max session duration
4. **Critical failure** -- security vulnerability introduced, data loss risk
5. **Manual stop** -- user interrupts the loop
6. **No eligible tasks** -- all remaining tasks are blocked by failed dependencies

## Configuration

```yaml
autopilot:
  max_session_duration: 120m    # Stop after 2 hours
  max_budget: 50.00             # USD per session
  max_retries_per_task: 2
  branch_prefix: "autopilot"
  base_branch: "main"
  auto_merge: false             # Require human review before merge
  quality_gates:
    typecheck: required
    lint: required
    unit_tests: required
    integration_tests: optional
    security_scan: required
  on_all_complete: "notify"     # notify | merge | pr
```

## Safety Guards

- **Never force-push**: All work is additive
- **Never modify main directly**: Always use feature branches
- **Never skip security scan**: Even on retry
- **Rollback on critical failure**: `git checkout main` and discard branch
- **Human review before merge**: `auto_merge: false` by default
- **PII check**: Scan all committed content through PII detection before push

## Integration Points

- **Trajectory Learning**: Record every autopilot run as a trajectory for pattern extraction
- **Cost Tracking**: Report per-task and per-session cost to the cost tracker
- **SPARC Methodology**: Each task follows a compressed SPARC cycle (spec from task description, implement, refine through quality gates, complete with commit)

## Implementation Checklist

- [ ] Task queue with priority sorting and dependency resolution
- [ ] Branch creation and cleanup automation
- [ ] Quality gate runner for each supported language
- [ ] Retry logic with failure analysis
- [ ] Progress tracking and reporting
- [ ] Budget and time limit enforcement
- [ ] Safety guards (no force-push, no main commits)
- [ ] Trajectory recording for each autopilot session
- [ ] Termination condition detection
