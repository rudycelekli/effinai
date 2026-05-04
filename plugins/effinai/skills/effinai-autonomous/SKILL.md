---
name: effinai-autonomous
description: >
  Autonomous PRD-to-code execution loop — reads user stories from a PRD file, 
  automatically spawns agents, implements features, runs quality gates, commits, 
  and moves to the next story. Use when the user wants autonomous development.
argument-hint: "[--prd file.json] [--max-iterations N]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  Read
  Write
  Edit
  Glob
  Grep
  Agent
  mcp__effinai__effinai_goal_plan
  mcp__effinai__effinai_loop_start
  mcp__effinai__effinai_loop_status
metadata:
  priority: 7
  promptSignals:
    phrases:
      - "autonomous"
      - "prd to code"
      - "run prd"
      - "auto implement"
      - "autopilot"
    anyOf:
      - "autonomous"
      - "prd"
      - "autopilot"
    minScore: 4
---

# Effin.AI Autonomous Mode (PRD-to-Code)

## The Loop

1. Read `prd.json` with user stories and acceptance criteria
2. Pick highest-priority pending story
3. Search memory for similar past patterns
4. Spawn the right agents (auto-selected by task type)
5. Implement the story with anti-drift monitoring
6. Run quality gates (typecheck, lint, test)
7. Commit on success: `feat: [S1] Add login endpoint`
8. Store the pattern for future learning
9. Move to next story
10. Repeat until all stories pass or max iterations reached

## Usage

```bash
# Generate example PRD
effin run --example

# Run autonomous loop
effin run --prd prd.json --max-iterations 20
```

## Quality Gates
- TypeScript type checking
- ESLint / linting
- Test suite execution
- All must pass before commit

## MCP Tools
- `effinai_goal_plan` — GOAP (Goal-Oriented Action Planning)
- `effinai_loop_start` — Start autonomous loop
- `effinai_loop_status` — Check loop progress
