---
name: background-tasks
description: "Launch, monitor, and manage long-running background tasks with output streaming, notifications, and lifecycle control"
version: "1.0.0"
tags: [background, async, tasks, monitoring, automation]
---

# Background Tasks

## Activation
When the agent needs to run long-running operations (builds, tests, deployments) without blocking the conversation, or needs to monitor and manage multiple concurrent operations.

## Concept

A background task system that lets agents launch operations, continue other work, and get notified when tasks complete. Supports output streaming, tail following, task waiting, and lifecycle management. Ported from jcode's bg (background) tool.

## Actions

| Action | Description |
|--------|-------------|
| `list` | List all background tasks with status |
| `status` | Get detailed status of a specific task |
| `output` | Read full output of a completed task |
| `tail` | Get last N lines of task output (default 80) |
| `cancel` | Cancel a running task |
| `cleanup` | Remove old completed/failed tasks |
| `watch` | Subscribe to task completion notifications |
| `wait` | Block until task reaches target state (with timeout) |
| `subscribe` | Get push notifications on task events |

## Task States

| State | Description |
|-------|-------------|
| **Running** | Currently executing |
| **Completed** | Finished successfully |
| **Failed** | Exited with error |
| **Superseded** | Replaced by a newer task |
| **Terminal** | Any final state (completed/failed/superseded) |

## Launching Background Tasks

### Via Bash
```bash
# Run a build in background
bash --background "cargo build --release"

# Run tests in background  
bash --background "npm test -- --coverage"

# Long-running operation
bash --background "python train_model.py --epochs 100"
```

### Via Subagent
```
subagent:
  description: "Run full test suite"
  prompt: "Execute all tests and report failures"
  subagent_type: "tester"
  background: true
```

## Monitoring

### List Tasks
```
bg list:
  status_filter: "running"     # or completed, failed, all
  session_only: true           # only tasks from this session
```

Output:
```
ID          | Status    | Started      | Command
bg-abc123   | running   | 2min ago     | cargo build --release
bg-def456   | completed | 5min ago     | npm test
bg-ghi789   | failed    | 10min ago    | deploy.sh production
```

### Wait for Completion
```
bg wait:
  task_id: "bg-abc123"
  max_wait_seconds: 300          # timeout (default 60, max 3600)
  return_on_progress: true       # return on intermediate progress events
```

### Tail Output
```
bg tail:
  task_id: "bg-abc123"
  lines: 40                      # last N lines
```

## Notifications

### Watch Mode
Subscribe to completion events:
```
bg watch:
  task_id: "bg-abc123"
  notify: true      # send notification on completion
  wake: true         # wake the agent on completion
```

### Delivery
When a watched task completes:
- Agent receives a soft interrupt with task result
- Can auto-resume work based on task outcome
- Notification includes exit code, runtime, and output preview

## Cleanup

Remove old tasks to free disk space:
```
bg cleanup:
  max_age_hours: 24      # remove tasks older than this
  dry_run: true           # preview what would be removed
  status_filter: ["completed", "failed"]
```

## Workflow Example

### Build + Test + Deploy Pipeline
```
1. Launch build in background
   -> bg-build-123

2. Continue working on other code while build runs

3. Wait for build
   bg wait: task_id: bg-build-123

4. If build succeeded, launch tests in background
   -> bg-test-456

5. Continue code review while tests run

6. Wait for tests
   bg wait: task_id: bg-test-456

7. If tests passed, deploy
```

### Parallel Test Suites
```
1. Launch unit tests:      bg-unit-123
2. Launch integration tests: bg-int-456
3. Launch e2e tests:        bg-e2e-789

4. Wait for all:
   bg wait: task_ids: [bg-unit-123, bg-int-456, bg-e2e-789]

5. Check results of each
```

## Output Management

| Parameter | Default | Description |
|-----------|---------|-------------|
| `tail_lines` | 80 | Default lines for tail action |
| `wait_preview_lines` | 40 | Lines shown when wait returns |
| `max_output_bytes` | 50,000 | Max output to store per task |

Large outputs are truncated with a notice. Use `tail` with specific line counts for precise output access.
