---
name: effinai-domain-verticals
description: >
  Domain-specific AI intelligence — Financial Risk (VaR, stress testing), Healthcare 
  Clinical (protocols, pathways), Legal Contracts (clause analysis, risk scoring), 
  IoT Fleet (telemetry, firmware), Quantum Optimization, Neural Trading. Use when 
  working in specialized domains.
argument-hint: "[domain] [operation]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  mcp__effinai__effinai_market_analyze
  mcp__effinai__effinai_iot_status
  mcp__effinai__effinai_vector_search
  mcp__effinai__effinai_vector_store
  mcp__effinai__effinai_graph_query
  mcp__effinai__effinai_graph_add
  mcp__effinai__effinai_wasm_execute
  mcp__effinai__effinai_browser_snapshot
  mcp__effinai__effinai_browser_interact
metadata:
  priority: 5
  promptSignals:
    phrases:
      - "financial risk"
      - "healthcare"
      - "legal contract"
      - "IoT"
      - "quantum"
      - "trading"
      - "market analysis"
    anyOf:
      - "financial"
      - "healthcare"
      - "legal"
      - "IoT"
      - "quantum"
      - "trading"
    minScore: 4
---

# Effin.AI Domain Verticals

## Financial Risk Analysis
- Value-at-Risk calculations, stress testing, scenario analysis
- Portfolio risk assessment with Monte Carlo simulation
- 9 neural trading tools: backtest, signals, portfolio, risk

## Healthcare Clinical
- Clinical protocol analysis and pathway optimization
- Risk stratification and patient outcome prediction
- HIPAA-aware data handling

## Legal Contract Intelligence
- Contract clause analysis and risk scoring
- Comparison across document versions
- Auto-generation of standard clauses

## IoT Fleet Management
- Device telemetry monitoring (22 specialized tools)
- Fleet-wide firmware management and OTA updates
- Trust scoring and anomaly detection
- Edge computing orchestration

## Quantum Optimization
- Quantum annealing and variational algorithms
- Circuit optimization and benchmarking
- Hybrid quantum-classical workflows

## Neural Trading
- Market signal detection and backtesting
- Portfolio optimization with risk constraints
- Multi-timeframe analysis

## Vector Intelligence
- HNSW vector search (150x-12,500x faster)
- Knowledge graph operations
- Hyperbolic reasoning for hierarchical data
