---
name: self-improving-agent
description: "Curate auto-memory into durable project knowledge. Analyze memory for patterns, promote proven learnings to project rules, extract recurring solutions into reusable skills."
version: "1.0.0"
tags: [meta, memory, self-improvement, knowledge-management, automation]
createdBy: "built-in"
status: "active"
---

# Self-Improving Agent

> Auto-memory captures. This skill curates.

AI coding agents automatically record project patterns, debugging insights, and preferences in memory. This skill adds the intelligence layer: it analyzes what the agent has learned, promotes proven patterns into project rules, and extracts recurring solutions into reusable skills.

## Slash Commands

| Command | What it does |
|---------|-------------|
| `/si:review` | Analyze memory — find promotion candidates, stale entries, consolidation opportunities |
| `/si:promote` | Graduate a pattern from memory to CLAUDE.md or `.claude/rules/` |
| `/si:extract` | Turn a proven pattern into a standalone skill |
| `/si:status` | Memory health dashboard — line counts, topic files, recommendations |
| `/si:remember` | Explicitly save important knowledge to auto-memory |

## How It Fits Together

```
┌─────────────────────────────────────────────────────────┐
│                   Agent Memory Stack                     │
├─────────────┬──────────────────┬────────────────────────┤
│  CLAUDE.md  │   Auto Memory    │   Session Memory       │
│  (you write)│   (agent writes) │   (agent writes)       │
│  Rules &    │   MEMORY.md      │   Conversation logs    │
│  standards  │   + topic files  │   + continuity         │
│  Full load  │   First 200 lines│   Contextual load      │
├─────────────┴──────────────────┴────────────────────────┤
│              ↑ /si:promote        ↑ /si:review          │
│         Self-Improving Agent (this skill)                │
│              ↓ /si:extract    ↓ /si:remember            │
├─────────────────────────────────────────────────────────┤
│  .claude/rules/    │    New Skills    │   Error Logs     │
│  (scoped rules)    │    (extracted)   │   (auto-captured)│
└─────────────────────────────────────────────────────────┘
```

## Memory Architecture

### Where things live

| File | Who writes | Scope | Loaded |
|------|-----------|-------|--------|
| `./CLAUDE.md` | You (+ `/si:promote`) | Project rules | Full file, every session |
| `~/.claude/CLAUDE.md` | You | Global preferences | Full file, every session |
| `~/.claude/projects/<path>/memory/MEMORY.md` | Agent (auto) | Project learnings | First 200 lines |
| `~/.claude/projects/<path>/memory/*.md` | Agent (overflow) | Topic-specific notes | On demand |
| `.claude/rules/*.md` | You (+ `/si:promote`) | Scoped rules | When matching files open |

### The promotion lifecycle

```
1. Agent discovers pattern → auto-memory (MEMORY.md)
2. Pattern recurs 2-3x → /si:review flags it as promotion candidate
3. You approve → /si:promote graduates it to CLAUDE.md or rules/
4. Pattern becomes an enforced rule, not just a note
5. MEMORY.md entry removed → frees space for new learnings
```

## Core Concepts

### Auto-memory is capture, not curation

Auto-memory is excellent at recording what the agent learns. But it has no judgment about:
- Which learnings are temporary vs. permanent
- Which patterns should become enforced rules
- When the 200-line limit is wasting space on stale entries
- Which solutions are good enough to become reusable skills

That is what this skill does.

### Promotion = graduation

When you promote a learning, it moves from the agent's scratchpad (MEMORY.md) to your project's rule system (CLAUDE.md or `.claude/rules/`). The difference matters:

- **MEMORY.md**: "I noticed this project uses pnpm" (background context)
- **CLAUDE.md**: "Use pnpm, not npm" (enforced instruction)

Promoted rules have higher priority and load in full (not truncated at 200 lines).

### Rules directory for scoped knowledge

Not everything belongs in CLAUDE.md. Use `.claude/rules/` for patterns that only apply to specific file types:

```yaml
# .claude/rules/api-testing.md
---
paths:
  - "src/api/**/*.test.ts"
  - "tests/api/**/*"
---
- Use supertest for API endpoint testing
- Mock external services with msw
- Always test error responses, not just happy paths
```

This loads only when the agent works with API test files — zero overhead otherwise.

## Agents

### memory-analyst
Analyzes MEMORY.md and topic files to identify:
- Entries that recur across sessions (promotion candidates)
- Stale entries referencing deleted files or old patterns
- Related entries that should be consolidated
- Gaps between what MEMORY.md knows and what CLAUDE.md enforces

### skill-extractor
Takes a proven pattern and generates a complete skill:
- SKILL.md with proper frontmatter
- Reference documentation
- Examples and edge cases

## When to Use Each Command

### After completing a feature or debugging session
```
/si:review
```
Check if anything the agent learned should become a permanent rule.

### When a pattern keeps coming up
```
/si:promote "Always run migrations before tests in this project"
```
Moves it from MEMORY.md (background note) to CLAUDE.md (enforced rule).

### When you solved something non-obvious that could help other projects
```
/si:extract "Docker build fix for ARM64 platform mismatch"
```
Creates a standalone skill ready to install elsewhere.

### To check memory capacity
```
/si:status
```
Shows line counts, topic files, stale entries, and recommendations.

## Key Principle

**Don't fight auto-memory — orchestrate it.**

- Auto-memory is great at capturing patterns. Let it do its job.
- This skill adds judgment: what's worth keeping, what should be promoted, what's stale.
- Promoted rules in CLAUDE.md have higher priority than MEMORY.md entries.
- Removing promoted entries from MEMORY.md frees space for new learnings.
