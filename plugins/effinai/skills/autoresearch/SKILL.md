---
name: autoresearch
description: "Autonomous experiment loop that optimizes any file by a measurable metric. The agent edits a target file, runs a fixed evaluation, keeps improvements (git commit), discards failures (git reset), and loops indefinitely."
version: "1.0.0"
tags: [optimization, experiments, automation, benchmarking, autonomous]
createdBy: "built-in"
status: "active"
---

# Autoresearch Agent

> You sleep. The agent experiments. You wake up to results.

Autonomous experiment loop inspired by Karpathy's autoresearch. The agent edits one file, runs a fixed evaluation, keeps improvements, discards failures, and loops indefinitely.

Not one guess — fifty measured attempts, compounding.

## Slash Commands

| Command | What it does |
|---------|-------------|
| `/ar:setup` | Set up a new experiment interactively |
| `/ar:run` | Run a single experiment iteration |
| `/ar:loop` | Start autonomous loop with configurable interval (10m, 1h, daily, weekly, monthly) |
| `/ar:status` | Show dashboard and results |
| `/ar:resume` | Resume a paused experiment |

## Activation Triggers

- "Make this faster / smaller / better"
- "Optimize [file] for [metric]"
- "Improve my [headlines / copy / prompts]"
- "Run experiments overnight"
- "I want to get [metric] from X to Y"
- Any request involving: optimize, benchmark, improve, experiment loop

If the user describes a target file + a way to measure success, this skill applies.

## Setup

### Create the Experiment

```bash
# Project-level (inside repo, git-tracked)
python scripts/setup_experiment.py \
  --domain engineering \
  --name api-speed \
  --target src/api/search.py \
  --eval "pytest bench.py --tb=no -q" \
  --metric p50_ms \
  --direction lower \
  --scope project

# User-level (personal, in ~/.autoresearch/)
python scripts/setup_experiment.py \
  --domain marketing \
  --name medium-ctr \
  --target content/titles.md \
  --eval "python evaluate.py" \
  --metric ctr_score \
  --direction higher \
  --evaluator llm_judge_content \
  --scope user
```

### What Setup Creates

```
.autoresearch/
├── config.yaml                        # Global settings
├── .gitignore                         # Ignores results.tsv, *.log
└── {domain}/{experiment-name}/
    ├── program.md                     # Objectives, constraints, strategy
    ├── config.cfg                     # Target, eval cmd, metric, direction
    ├── results.tsv                    # Experiment log (gitignored)
    └── evaluate.py                    # Evaluation script (if --evaluator used)
```

**results.tsv columns:** `commit | metric | status | description`

### Domains

| Domain | Use Cases |
|--------|-----------|
| `engineering` | Code speed, memory, bundle size, test pass rate, build time |
| `marketing` | Headlines, social copy, email subjects, ad copy, engagement |
| `content` | Article structure, SEO descriptions, readability, CTR |
| `prompts` | System prompts, chatbot tone, agent instructions |
| `custom` | Anything else with a measurable metric |

## Agent Protocol

You are the loop. The scripts handle setup and evaluation — you handle the creative work.

### Before Starting
1. Read config.cfg to get: target, evaluate_cmd, metric, metric_direction, time_budget_minutes
2. Read program.md for strategy, constraints, and what you can/cannot change
3. Read results.tsv for experiment history
4. Checkout the experiment branch: `git checkout autoresearch/{domain}/{name}`

### Each Iteration
1. Review results.tsv — what worked? What failed? What hasn't been tried?
2. Decide ONE change to the target file. One variable per experiment.
3. Edit the target file
4. Commit: `git add {target} && git commit -m "experiment: {description}"`
5. Evaluate: `python scripts/run_experiment.py --experiment {domain}/{name} --single`
6. Read the output — it prints KEEP, DISCARD, or CRASH with the metric value
7. Go to step 1

### Strategy Escalation
- Runs 1-5: Low-hanging fruit (obvious improvements, simple optimizations)
- Runs 6-15: Systematic exploration (vary one parameter at a time)
- Runs 16-30: Structural changes (algorithm swaps, architecture shifts)
- Runs 30+: Radical experiments (completely different approaches)
- If no improvement in 20+ runs: update program.md Strategy section

### Self-Improvement
After every 10 experiments, review results.tsv for patterns. Update the Strategy section of program.md with what you learned. Future iterations benefit from this accumulated knowledge.

### Rules

- **One change per experiment.** Don't change 5 things at once. You won't know what worked.
- **Simplicity criterion.** A small improvement that adds ugly complexity is not worth it. Equal performance with simpler code is a win. Removing code that gets same results is the best outcome.
- **Never modify the evaluator.** evaluate.py is the ground truth. Modifying it invalidates all comparisons.
- **Timeout.** If a run exceeds 2.5x the time budget, kill it and treat as crash.
- **Crash handling.** If it's a typo or missing import, fix and re-run. If fundamentally broken, revert, log "crash", move on. 5 consecutive crashes -> pause and alert.
- **No new dependencies.** Only use what's already available in the project.

## Evaluators

### Free Evaluators (no API cost)

| Evaluator | Metric | Use Case |
|-----------|--------|----------|
| `benchmark_speed` | `p50_ms` (lower) | Function/API execution time |
| `benchmark_size` | `size_bytes` (lower) | File, bundle, Docker image size |
| `test_pass_rate` | `pass_rate` (higher) | Test suite pass percentage |
| `build_speed` | `build_seconds` (lower) | Build/compile/Docker build time |
| `memory_usage` | `peak_mb` (lower) | Peak memory during execution |

### LLM Judge Evaluators (uses your subscription)

| Evaluator | Metric | Use Case |
|-----------|--------|----------|
| `llm_judge_content` | `ctr_score` 0-10 (higher) | Headlines, titles, descriptions |
| `llm_judge_prompt` | `quality_score` 0-100 (higher) | System prompts, agent instructions |
| `llm_judge_copy` | `engagement_score` 0-10 (higher) | Social posts, ad copy, emails |

### Custom Evaluators

The user writes their own evaluate.py. Only requirement: it must print `metric_name: value` to stdout.

## Viewing Results

```bash
# Single experiment
python scripts/log_results.py --experiment engineering/api-speed

# All experiments in a domain
python scripts/log_results.py --domain engineering

# Cross-experiment dashboard
python scripts/log_results.py --dashboard
```

### Dashboard Output

```
DOMAIN          EXPERIMENT          RUNS  KEPT  BEST         D FROM START  STATUS
engineering     api-speed            47    14   185ms        -76.9%        active
engineering     bundle-size          23     8   412KB        -58.3%        paused
marketing       medium-ctr           31    11   8.4/10       +68.0%        active
prompts         support-tone         15     6   82/100       +46.4%        done
```

## Proactive Triggers

- **No evaluation command works** — Test it before starting the loop
- **Target file not in git** — Initialize git first
- **Metric direction unclear** — Ask: is lower or higher better?
- **Agent modifying evaluate.py** — Hard stop. This invalidates all comparisons
- **5 consecutive crashes** — Pause the loop. Alert the user
- **No improvement in 20+ runs** — Suggest changing strategy or different approach
