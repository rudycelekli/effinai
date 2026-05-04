---
name: effinai-skills
description: >
  98 built-in reusable skills with progressive loading, curator lifecycle management, 
  and auto-creation from successful tasks. Use when listing, running, or creating skills.
argument-hint: "[list|run|create|info] [--name skill-name]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  Read
  mcp__effinai__effinai_skill_list
  mcp__effinai__effinai_skill_run
  mcp__effinai__effinai_skill_create
  mcp__effinai__effinai_skill_search
  mcp__effinai__effinai_skill_info
  mcp__effinai__effinai_curator_status
metadata:
  priority: 6
  promptSignals:
    phrases:
      - "skill"
      - "list skills"
      - "run skill"
      - "create skill"
      - "curator"
    anyOf:
      - "skill"
      - "curator"
    minScore: 4
---

# Effin.AI Skills (98 Built-In)

## Categories
- **Development**: code-review, tdd, lint-fix, refactor-safe, type-safety, quick-fix
- **Architecture**: architecture-review, adr-lifecycle, ddd-design, sparc-methodology
- **Testing**: test-generator, test-coverage, security-scan, performance-audit
- **AI & Agents**: agent-designer, self-improving-agent, rag-architect, karpathy-coder
- **Memory**: knowledge-graph, graph-rag, hybrid-search, holographic-memory
- **DevOps**: ci-pipeline, docker-compose, k8s-deploy, monitoring-setup
- **Productivity**: prd-generator, autopilot-loop, goal-loop, cron-scheduler
- **Integration**: composio-connect, multi-platform-gateway, webhook-gateway

## Progressive Loading
Only frontmatter (~100 tokens) loaded for listing. Full body loads on-demand when skill is executed.

## Curator Lifecycle
- **Active** — Recently used
- **Stale** (30+ days) — Marked for review
- **Archived** (90+ days) — Removed from rotation
- **Pinned** — Protected from archival
- **Auto-Created** — Generated from successful tasks

```bash
effin skill list
effin skill run --name code-review
effin curator status
effin curator pin auth
```
