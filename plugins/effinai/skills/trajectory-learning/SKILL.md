---
name: trajectory-learning
description: "Record and learn from task execution trajectories to replay successful patterns"
version: "1.0.0"
tags: [learning, patterns, optimization, memory]
---

# Trajectory Learning

Record step-by-step task execution as trajectories. Extract patterns from successes, prune failures, and replay proven approaches on similar tasks.

## What is a Trajectory

A trajectory is an ordered sequence of steps an agent took to complete a task, capturing what was done, in what order, and whether it succeeded.

```typescript
interface Trajectory {
  id: string;
  taskType: string;           // e.g., "bug-fix", "feature", "refactor"
  taskDescription: string;
  agent: string;
  startedAt: string;
  completedAt: string;
  outcome: "success" | "failure" | "partial";
  steps: TrajectoryStep[];
  metrics: {
    totalDuration: number;    // ms
    tokenCost: number;        // USD
    filesModified: number;
    testsAdded: number;
    iterationCount: number;   // How many refinement cycles
  };
  tags: string[];
  similarity_hash: string;    // For finding similar trajectories
}

interface TrajectoryStep {
  order: number;
  action: "read_file" | "search" | "edit_file" | "create_file" |
          "run_command" | "run_test" | "review" | "plan" | "commit";
  target: string;             // File path, command, search query
  reasoning: string;          // Why this step was taken
  duration: number;           // ms
  outcome: "success" | "failure" | "skipped";
  artifacts?: string[];       // Files created or modified
}
```

## Recording Trajectories

### Automatic Recording

Hook into agent execution to capture steps automatically:

```bash
# Start recording before a task
npx @claude-flow/cli@latest hooks pre-task \
  --description "Fix authentication timeout bug" \
  --record-trajectory true

# Each agent action is logged as a step
# File reads, edits, test runs, commands are captured

# End recording after task completion
npx @claude-flow/cli@latest hooks post-task \
  --task-id "task-123" \
  --success true \
  --store-results true
```

### Step Capture Points

Record a step whenever an agent:

1. **Reads a file** - Which file, why (understanding context, checking tests, etc.)
2. **Searches codebase** - What query, what was found
3. **Edits a file** - What changed, reasoning for the change
4. **Runs a command** - What command, exit code, relevant output
5. **Runs tests** - Which tests, pass/fail counts
6. **Makes a decision** - Chose approach A over B, with reasoning
7. **Commits code** - What was committed, commit message

### What NOT to Record

- PII (apply REDACT policy to all trajectory data)
- Full file contents (record file paths and change summaries)
- API keys or secrets
- Raw LLM prompts (record intent summaries instead)

## Pattern Extraction

After accumulating trajectories, extract reusable patterns:

### Step 1: Cluster Similar Trajectories

Group trajectories by:
- Task type (bug-fix, feature, refactor)
- File patterns involved (same directories, same file types)
- Step sequence similarity (edit-test-commit vs. read-plan-edit-test-commit)

### Step 2: Identify Common Sequences

Find step sequences that appear in 3+ successful trajectories:

```
Pattern: "test-driven-bugfix"
Frequency: 12 occurrences
Success rate: 92%
Steps:
  1. Read failing test / reproduction steps
  2. Search for related source code
  3. Read source code around the bug
  4. Edit source to fix the bug
  5. Run failing test to verify fix
  6. Run full test suite
  7. Commit
Avg duration: 4.2 minutes
Avg cost: $0.85
```

### Step 3: Score Patterns

```typescript
interface Pattern {
  name: string;
  steps: PatternStep[];
  frequency: number;        // Times observed
  successRate: number;       // 0.0-1.0
  avgDuration: number;       // ms
  avgCost: number;           // USD
  applicableTaskTypes: string[];
  score: number;             // Composite quality score
}

// Score = successRate * 0.5 + frequency_normalized * 0.2 +
//         speed_normalized * 0.15 + cost_efficiency * 0.15
```

### Step 4: Store Patterns

```bash
npx @claude-flow/cli@latest memory store \
  --namespace trajectory-patterns \
  --key "test-driven-bugfix" \
  --value '{"steps":["read-test","search-source","read-source","edit-fix","run-test","run-suite","commit"],"successRate":0.92,"avgCost":0.85}'
```

## Replaying Patterns

When a new task arrives, find and suggest matching patterns:

```bash
# Search for relevant patterns
npx @claude-flow/cli@latest memory search \
  --query "fix authentication timeout" \
  --namespace trajectory-patterns
```

### Replay Protocol

1. **Match**: Find patterns with >70% similarity to current task
2. **Propose**: Show the agent the recommended step sequence
3. **Adapt**: Agent follows the pattern but adapts to specific context
4. **Deviate**: If the pattern doesn't fit, deviate and record the new trajectory
5. **Update**: After completion, update pattern statistics

```
Task: "Fix rate limiter returning 500 instead of 429"
Matched Pattern: "test-driven-bugfix" (similarity: 0.85)

Suggested approach:
  1. Find the failing test or write a reproduction test
  2. Search for rate limiter source code
  3. Read the error handling logic
  4. Fix the status code
  5. Run the test
  6. Run full suite
  7. Commit

Agent follows pattern with task-specific adaptations.
```

## Pruning Failed Trajectories

Not all trajectories are worth keeping:

### Prune When
- Outcome is "failure" AND no partial learnings
- Pattern has <30% success rate after 10+ observations
- Trajectory is a duplicate of an existing one
- Trajectory is older than 90 days with no replays

### Keep Failed Trajectories When
- They contain a valuable "what NOT to do" lesson
- The failure revealed a previously unknown constraint
- The trajectory eventually led to a successful approach (the failure path is instructive)

### Pruning Command

```bash
npx @claude-flow/cli@latest hooks intelligence trajectory-prune \
  --max-age 90d \
  --min-success-rate 0.3 \
  --dry-run true
```

## Metrics Dashboard

Track trajectory learning effectiveness:

```
=== Trajectory Learning Metrics ===
Total trajectories recorded:  248
Successful:                   198 (80%)
Patterns extracted:            32
Pattern replay attempts:       67
Replay success rate:           88%

Top Patterns:
  test-driven-bugfix       | 92% success | 12 replays | avg 4.2min
  read-plan-implement      | 85% success | 18 replays | avg 12.1min
  refactor-extract-method  | 78% success |  8 replays | avg 6.8min

Cost Savings from Replays:
  Without patterns: avg $2.40/task
  With patterns:    avg $1.10/task
  Savings:          54%
```

## Integration Points

- **SPARC Methodology**: Trajectories map to SPARC phases. Record which phase each step belongs to.
- **Cost Tracking**: Include token cost per trajectory for ROI analysis.
- **Autopilot Loop**: Autopilot uses trajectory patterns to choose approaches automatically.
- **PII Detection**: All trajectory data passes through PII REDACT before storage.

## Implementation Checklist

- [ ] Step capture hooks in agent execution pipeline
- [ ] Trajectory storage in memory with vector indexing
- [ ] Clustering algorithm for similar trajectories
- [ ] Pattern extraction from successful clusters
- [ ] Pattern scoring and ranking
- [ ] Similarity matching for new tasks
- [ ] Replay suggestion engine
- [ ] Pruning scheduler for stale/failed trajectories
- [ ] Metrics dashboard generation
- [ ] PII redaction in all trajectory data
