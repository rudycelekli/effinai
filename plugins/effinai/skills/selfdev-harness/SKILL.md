---
name: selfdev-harness
description: "Agent self-modification capabilities — hot reload, canary builds, runtime skill updates, and harness evolution"
version: "1.0.0"
tags: [self-modification, meta, harness, evolution, hot-reload]
---

# Self-Dev Harness

## Activation
When the agent needs to modify its own tools, skills, or configuration at runtime, or when building and testing improvements to the agent harness itself.

## Concept

The agent can modify its own harness: adding skills, updating tool configurations, building canary versions of itself, and hot-reloading changes without restart. This enables self-improvement loops where the agent observes deficiencies and patches them. Ported from jcode's selfdev system.

## Capabilities

### 1. Hot-Reload Skills
Add or modify skills without restarting the agent:
```
skill_manage:
  action: "create"
  name: "my-new-skill"
  content: |
    ---
    name: my-new-skill
    description: "A skill I just created"
    ---
    # My New Skill
    Instructions for this skill...
```

After creation, the skill is immediately available via `/my-new-skill`.

### 2. Dynamic Tool Registration
Register new tools at runtime:
```
- MCP servers can be connected/disconnected without restart
- New tool definitions take effect immediately
- Tool aliases allow compatibility across naming conventions
```

### 3. Canary Builds (Self-Dev Mode)
When working on the agent's own codebase:
```
selfdev:
  action: "build"
  target: "auto"        # auto, tui, desktop, or all
  reason: "Testing new memory compaction algorithm"
```

Build queue prevents conflicts:
- Only one build at a time
- Queued builds wait for active build to complete
- Agents notified when their build is ready

### 4. Hot Reload After Build
```
selfdev:
  action: "reload"
  context: "Testing memory compaction changes"
```

Process:
1. Save current context and task state
2. Build new version
3. Restart with saved context
4. Resume work where it left off

### 5. Runtime Testing
```
selfdev:
  action: "test"
  command: "cargo test memory_compaction"
```

Run tests against the agent's own code without leaving the session.

## Self-Improvement Loop

```
1. Agent notices a deficiency
   "I keep losing context during long sessions"

2. Agent designs a fix
   "I need smarter compaction that preserves decisions"

3. Agent implements the fix
   Edit compaction.rs, add semantic mode

4. Agent builds and tests
   selfdev build -> selfdev test

5. Agent hot-reloads
   selfdev reload with saved context

6. Agent validates the improvement
   "Context retention improved from 60% to 90%"

7. Agent stores the pattern
   memory store: "semantic compaction preserves decisions"
```

## Skill Lifecycle

### Creating Skills
```
1. Agent identifies a recurring pattern
2. Creates a skill with YAML frontmatter + markdown body
3. Skill stored in ~/.effin/skills/ or ./.effin/skills/
4. Immediately available for /slash-command invocation
```

### Updating Skills
```
1. Agent detects skill needs improvement
2. Reloads skill from disk after edit
3. Updated content takes effect immediately
```

### Skill Discovery
Skills are imported from multiple sources:
- Built-in skills (shipped with Effin.AI)
- User global skills (~/.effin/skills/)
- Project local skills (./.effin/skills/)
- External imports (from Claude Code, Codex CLI, etc.)

## Safety Constraints

- Builds are sandboxed (no access to production data)
- Reload preserves session state and context
- Destructive operations require confirmation
- Build failures don't affect running instance
- Context is saved before any self-modification

## MCP Server Self-Configuration

The agent can configure its own MCP integrations:
```
mcp connect:
  name: "my-api"
  command: "npx"
  args: ["-y", "@my/mcp-server"]
  env:
    API_KEY: "..."
```

After connection, all tools from the MCP server are available immediately.
