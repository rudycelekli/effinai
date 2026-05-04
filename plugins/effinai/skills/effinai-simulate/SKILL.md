---
name: effinai-simulate
description: >
  Multi-persona swarm debate simulation and AI-powered trend prediction. Use when 
  the user wants to explore scenarios, debate decisions, predict trends, or brainstorm 
  with multiple AI perspectives.
argument-hint: "[--scenario description] [--topic topic] [--horizon short|medium|long]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  mcp__effinai__effinai_simulate
  mcp__effinai__effinai_predict
  mcp__effinai__effinai_brainstorm
  mcp__effinai__effinai_compare
metadata:
  priority: 6
  promptSignals:
    phrases:
      - "what if"
      - "simulate"
      - "predict"
      - "debate"
      - "forecast"
      - "brainstorm"
    anyOf:
      - "simulate"
      - "predict"
      - "what if"
      - "debate"
      - "brainstorm"
      - "forecast"
    minScore: 4
---

# Effin.AI Simulation & Prediction

## Multi-Agent Debate

```bash
effin simulate --scenario "Should we migrate to microservices?"
```

Spawns multiple AI personas (Optimist, Skeptic, Pragmatist, etc.) to debate the scenario, synthesizing conclusions with confidence scores.

## Trend Prediction

```bash
effin predict --topic "AI agent adoption" --horizon long
```

Multi-lens analysis across short/medium/long horizons with confidence levels and key factors.

## Brainstorming

Multi-perspective brainstorming with diverse viewpoints and structured comparison.

## MCP Tools
- `effinai_simulate` — Multi-persona debate simulation
- `effinai_predict` — Trend prediction and forecasting
- `effinai_brainstorm` — Multi-perspective brainstorming
- `effinai_compare` — Structured option comparison
