---
name: agenthub
description: "Multi-agent collaboration that spawns N parallel subagents competing on the same task via git worktree isolation. Agents work independently, results are evaluated by metric or LLM judge, and the best branch is merged."
version: "1.0.0"
tags: [agents, collaboration, competition, parallel, git-worktree, multi-agent]
createdBy: "built-in"
status: "active"
---

# AgentHub — Multi-Agent Collaboration

Spawn N parallel AI agents that compete on the same task. Each agent works in an isolated git worktree. The coordinator evaluates results and merges the winner.

## Slash Commands

| Command | Description |
|---------|-------------|
| `/hub:init` | Create a new collaboration session — task, agent count, eval criteria |
| `/hub:spawn` | Launch N parallel subagents in isolated worktrees |
| `/hub:status` | Show DAG state, agent progress, branch status |
| `/hub:eval` | Rank agent results by metric or LLM judge |
| `/hub:merge` | Merge winning branch, archive losers |
| `/hub:board` | Read/write the agent message board |
| `/hub:run` | One-shot lifecycle: init -> baseline -> spawn -> eval -> merge |

## Agent Templates

When spawning with `--template`, agents follow a predefined iteration pattern:

| Template | Pattern | Use Case |
|----------|---------|----------|
| `optimizer` | Edit -> eval -> keep/discard -> repeat x10 | Performance, latency, size |
| `refactorer` | Restructure -> test -> iterate until green | Code quality, tech debt |
| `test-writer` | Write tests -> measure coverage -> repeat | Test coverage gaps |
| `bug-fixer` | Reproduce -> diagnose -> fix -> verify | Bug fix approaches |

## Activation Triggers

- "try multiple approaches"
- "have agents compete"
- "parallel optimization"
- "spawn N agents"
- "compare different solutions"
- "fan-out" or "tournament"
- "generate content variations"
- "A/B test copy"
- "explore multiple strategies"

## Coordinator Protocol

The main session is the coordinator. It follows this lifecycle:

```
INIT -> DISPATCH -> MONITOR -> EVALUATE -> MERGE
```

### 1. Init

Run `/hub:init` to create a session. This generates:
- `.agenthub/sessions/{session-id}/config.yaml` — task config
- `.agenthub/sessions/{session-id}/state.json` — state machine
- `.agenthub/board/` — message board channels

### 2. Dispatch

Run `/hub:spawn` to launch agents. For each agent 1..N:
- Post task assignment to `.agenthub/board/dispatch/`
- Spawn via agent tool with worktree isolation
- All agents launched in a single message (parallel)

### 3. Monitor

Run `/hub:status` to check progress:
- Branch state and DAG visualization
- Board `progress/` channel has agent updates

### 4. Evaluate

Run `/hub:eval` to rank results:
- **Metric mode**: run eval command in each worktree, parse numeric result
- **Judge mode**: read diffs, coordinator ranks by quality
- **Hybrid**: metric first, LLM-judge for ties

### 5. Merge

Run `/hub:merge` to finalize:
- `git merge --no-ff` winner into base branch
- Tag losers: `git tag hub/archive/{session}/agent-{i}`
- Clean up worktrees
- Post merge summary to board

## Agent Protocol

Each subagent receives this prompt pattern:

```
You are agent-{i} in hub session {session-id}.
Your task: {task description}

Instructions:
1. Read your assignment at .agenthub/board/dispatch/{seq}-agent-{i}.md
2. Work in your worktree — make changes, run tests, iterate
3. Commit all changes with descriptive messages
4. Write your result summary to .agenthub/board/results/agent-{i}-result.md
5. Exit when done
```

Agents do NOT see each other's work. They do NOT communicate with each other. They only write to the board for the coordinator to read.

## DAG Model

### Branch Naming
```
hub/{session-id}/agent-{N}/attempt-{M}
```

### Immutability
The DAG is append-only:
- Never rebase or force-push agent branches
- Never delete commits (only branch refs after archival)
- Every approach preserved via git tags

## Message Board

Location: `.agenthub/board/`

| Channel | Writer | Reader | Purpose |
|---------|--------|--------|---------|
| `dispatch/` | Coordinator | Agents | Task assignments |
| `progress/` | Agents | Coordinator | Status updates |
| `results/` | Agents + Coordinator | All | Final results + merge summary |

### Board Rules
- Append-only: never edit or delete posts
- Unique filenames: `{seq:03d}-{author}-{timestamp}.md`
- YAML frontmatter required on all posts

## Evaluation Modes

### Metric-Based
Best for: benchmarks, test pass rates, file sizes, response times.

### LLM Judge
Best for: code quality, readability, architecture decisions.

The coordinator reads each agent's diff and ranks by:
1. Correctness (does it solve the task?)
2. Simplicity (fewer lines changed preferred)
3. Quality (clean execution, good structure)

### Hybrid
Run metric first. If top agents are within 10% of each other, use LLM judge to break ties.

## Session Lifecycle

```
init -> running -> evaluating -> merged
                              -> archived (if no winner)
```

## Proactive Triggers

| Signal | Action |
|--------|--------|
| All agents crashed | Post failure summary, suggest retry with different constraints |
| No improvement over baseline | Archive session, suggest different approaches |
| Orphan worktrees detected | Clean up |
| Session stuck in `running` | Check board for progress, consider timeout |
