---
name: karpathy-coder
description: "Active coding discipline enforcing Karpathy's 4 principles: surface assumptions before coding, keep it simple, make surgical changes, define verifiable goals. Detects overcoding, drive-by refactors, and hidden assumptions."
version: "1.0.0"
tags: [code-quality, discipline, simplicity, review, anti-patterns, best-practices]
createdBy: "built-in"
status: "active"
---

# Karpathy Coder — Active Coding Discipline

Derived from Andrej Karpathy's observations on LLM coding pitfalls. Not just guidelines — a systematic review framework to detect violations.

> "The models make wrong assumptions on your behalf and just run along with them without checking. They don't manage their confusion, don't seek clarifications, don't surface inconsistencies, don't present tradeoffs, don't push back when they should."
>
> "They really like to overcomplicate code and APIs, bloat abstractions, don't clean up dead code... implement a bloated construction over 1000 lines when 100 would do."
>
> "LLMs are exceptionally good at looping until they meet specific goals... Don't tell it what to do, give it success criteria and watch it go."

## The Four Principles

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

**The test:** Would a senior engineer say this is overcomplicated? If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

**The test:** Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

| Instead of... | Transform to... |
|---|---|
| "Add validation" | "Write tests for invalid inputs, then make them pass" |
| "Fix the bug" | "Write a test that reproduces it, then make it pass" |
| "Refactor X" | "Ensure tests pass before and after" |

For multi-step tasks, state a brief plan:

```
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

## Slash Command

`/karpathy-check` — Run the full 4-principle review on your staged changes.

## Detection Tools

| Tool | What it detects |
|---|---|
| `complexity_checker` | Over-engineering: too many classes, deep nesting, high cyclomatic complexity, unused params, premature abstractions |
| `diff_surgeon` | Diff noise: lines that don't trace to the stated goal — comment changes, style drift, drive-by refactors |
| `assumption_linter` | Hidden assumptions in a plan: unasked features, missing clarifications, silent interpretation choices |
| `goal_verifier` | Weak success criteria: vague plans without verifiable checks, missing test assertions |

## When to Relax

These principles bias toward **caution over speed**. For trivial tasks (typo fixes, obvious one-liners), use judgment. The principles matter most on:

- Non-trivial implementations (>20 lines changed)
- Code you don't fully understand
- Multi-step tasks with unclear requirements
- Anything that will be reviewed by humans

## Anti-Pattern Examples

### Over-Engineering
```
BAD:  AbstractFactoryProvider<T> with 3 interfaces for a single use case
GOOD: A plain function that does the thing
```

### Drive-By Refactoring
```
BAD:  "While I was fixing the bug, I also reformatted the file and renamed 5 variables"
GOOD: Fix the bug. Nothing else. File a separate task for the cleanup.
```

### Hidden Assumptions
```
BAD:  Silently choosing JWT over session cookies without mentioning the tradeoff
GOOD: "Two approaches: JWT (stateless, harder to revoke) vs sessions (stateful, easy revoke). Which do you prefer?"
```

### Vague Goals
```
BAD:  "Improve the API"
GOOD: "Reduce p95 latency from 800ms to under 200ms, verified by load test with 100 concurrent users"
```
