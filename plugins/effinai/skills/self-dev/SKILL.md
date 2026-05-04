---
name: self-dev
description: "Self-modification workflows for agents to update their own configuration, skills, builds, and patterns — with safe reload and rollback"
version: "1.0.0"
tags: [self-modification, meta, development, canary, reload, skills]
---

# Self-Dev

## Activation
When the agent needs to modify its own configuration, create or update skills, build and test canary versions of itself, or update learned patterns and behaviors. Use for meta-development where the agent improves its own capabilities.

## Concept

A self-modification framework that allows agents to safely update their own code, configuration, skills, and learned patterns. Includes build queuing (so multiple agents don't conflict), context-preserving reload (the agent can restart itself and resume), and canary builds for testing changes before promotion. Ported from jcode's `SelfDevTool` which manages the full lifecycle of self-modification with safety controls.

## Actions

### Launch Self-Dev Session
```
action: "launch"
prompt: "Improve the error handling in the browser tool"
# Spawns a new session specifically for self-development work.
# The session inherits context from the requesting session.
# Returns: session_id, repo_dir, command preview
```

### Build
```
action: "build"
target: "auto"              # auto, tui, desktop, or all
reason: "Fixed null check in memory recall"
notify: true                # Notify requesting agent when done
wake: true                  # Wake requesting agent when done
# Queue a build of the agent's own binary.
# Uses a build queue to prevent conflicts between concurrent agents.
```

### Build Status
```
action: "status"
# Check the current state of self-dev builds.
# Returns: queued builds, active build, recent completions/failures
```

### Reload
```
action: "reload"
context: "Working on auth middleware refactoring, was about to write tests"
# Build the current code, swap the binary, and restart the agent.
# Context is preserved in a ReloadContext file and restored after restart.
# The agent resumes where it left off with a note about what changed.
```

### Test / Check
```
action: "test"
command: "cargo test --lib"
# Run tests against the current self-dev changes before committing.

action: "check"
command: "cargo clippy -- -D warnings"
# Run quality checks on self-dev changes.
```

### Cancel Build
```
action: "cancel-build"
request_id: "build_abc123"
# Cancel a queued or in-progress build.
```

## Build Queue System

When multiple agents are working on self-modifications concurrently:

### Build Request States
| State | Description |
|-------|-------------|
| `Queued` | Waiting for build slot |
| `Building` | Actively compiling |
| `Attached` | Piggybacking on another build with same changes |
| `Completed` | Build succeeded |
| `Superseded` | A newer build made this one obsolete |
| `Failed` | Build failed with error |
| `Cancelled` | Manually cancelled |

### Deduplication
The build queue deduplicates builds with the same source state (git hash + dirty files). If Agent A and Agent B both request builds from the same code state, Agent B's request is "attached" to Agent A's build rather than building twice.

### Build Request Tracking
Each build request records:
- `session_id` -- Which agent session requested it
- `reason` -- Why the build is needed
- `repo_dir` and `worktree_scope` -- Where the code lives
- `requested_source` -- Git state at request time
- `built_source` -- Git state when build actually ran
- `published_version` -- Version number if build succeeded

## Reload Recovery

When an agent reloads itself:

1. **Pre-reload**: Save `ReloadContext` with:
   - Current task context (what the agent was working on)
   - Version before reload
   - Target version
   - Session ID
   
2. **Build & swap**: Compile new binary, replace the running executable

3. **Post-reload**: New process starts and:
   - Detects the ReloadContext file
   - Restores session state
   - Injects a system message: "You just reloaded from vX to vY. You were working on: [context]"
   - Resumes the session

## Self-Modification Patterns

### Skill Creation
```
1. Identify a capability gap or repeated pattern
2. Design the skill interface (activation, parameters, workflow)
3. Write the skill markdown file
4. Test the skill in a sandboxed session
5. Commit to the skills directory
```

### Configuration Update
```
1. Read current configuration
2. Identify the setting to change
3. Validate the new value
4. Write the updated configuration
5. Reload if the change requires it (or apply hot)
```

### Pattern Learning
```
1. Complete a task successfully
2. Extract the approach as a reusable pattern
3. Store in ambient-memory with appropriate tags
4. Update skill definitions if the pattern is broadly applicable
```

### Agent Behavior Tuning
```
1. Observe suboptimal agent behavior (from logs or user feedback)
2. Identify the root cause (prompt, skill, or configuration)
3. Make targeted modification
4. Build canary version
5. Test with representative workloads
6. Promote if metrics improve, rollback if not
```

## Safety Controls

- Build queue prevents concurrent build conflicts
- ReloadContext preserves work across restarts
- Source state tracking detects if code changed between request and build
- Canary builds allow testing before promotion
- Build deduplication prevents wasted compute
- Session inheritance ensures spawned self-dev sessions have full context
