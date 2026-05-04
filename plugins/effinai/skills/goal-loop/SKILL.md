---
name: goal-loop
description: "Persistent autonomous goal pursuit with judge-based evaluation — set a standing goal, auto-continue until done or budget exhausted"
version: "1.0.0"
tags: [automation, autonomy, goals, persistence, self-improving]
createdBy: "built-in"
status: "active"
---

# Goal Loop (Autonomous Goal Pursuit)

## Activation
When the user sets a standing goal that requires multiple turns of autonomous work, or asks the agent to keep working toward an objective until it is complete.

## Concept

A goal is a free-form user objective that stays active across turns. After each turn, a judge evaluates: "Is this goal satisfied?" If not, a continuation prompt feeds back into the session. The agent keeps working until:
- The goal is done (judge confirms)
- Turn budget is exhausted (auto-pauses)
- The user pauses or clears it
- The user sends a new message (takes priority, pauses goal loop)

## Architecture

### GoalState
```
goal: str              # The objective text
status: active|paused|done|cleared
turns_used: int        # Turns consumed so far
max_turns: int         # Budget (default 20)
last_verdict: done|continue|skipped
last_reason: str       # One-sentence rationale from judge
```

### Judge System

The judge is a separate, lightweight LLM call that evaluates whether the goal is satisfied:

**Judge System Prompt:**
"You are a strict judge evaluating whether an autonomous agent has achieved a user's stated goal. Reply ONLY with: `{\"done\": true|false, \"reason\": \"rationale\"}`"

**A goal is DONE when:**
- Response explicitly confirms completion, OR
- Response shows final deliverable was produced, OR
- Response explains goal is unachievable/blocked/needs user input

**Fail-open design:** Any judge error returns "continue" — a broken judge must not wedge progress. The turn budget is the backstop.

### Continuation Prompt Template
```
[Continuing toward your standing goal]
Goal: {goal}

Continue working toward this goal. Take the next concrete step.
If you believe the goal is complete, state so explicitly and stop.
If you are blocked and need input from the user, say so clearly and stop.
```

## Workflow

### 1. Set a Goal
```
/goal Build a REST API with user auth, CRUD endpoints, and tests
```
Creates GoalState with status=active, turns_used=0, max_turns=20.

### 2. Autonomous Execution Loop
After each turn:
1. Increment turns_used
2. Call judge with goal + last response
3. If verdict="done" -> mark done, stop
4. If turns_used >= max_turns -> pause with reason "budget exhausted"
5. Otherwise -> generate continuation prompt, feed back as next user message

### 3. User Controls
- `/goal <text>` — Set new standing goal
- `/goal pause` — Pause the active goal
- `/goal resume` — Resume (optionally reset budget)
- `/goal clear` — Remove the active goal
- `/goal status` — Show current state

### 4. Decision After Each Turn
The evaluate_after_turn() method returns:
```json
{
  "status": "active|paused|done",
  "should_continue": true|false,
  "continuation_prompt": "string or null",
  "verdict": "done|continue|skipped|inactive",
  "reason": "one-sentence rationale",
  "message": "user-visible status line"
}
```

## Design Invariants

- Continuation prompt is a normal user message — no system prompt mutation, no toolset swap, prompt caching stays intact
- Judge failures are fail-OPEN: continue. A broken judge must not wedge progress
- Real user messages preempt continuation prompts and pause the goal loop
- State persists across sessions for resume capability
- Zero hard dependency on specific CLI or gateway — both wire the same GoalManager

## Status Display
```
Active:  ⊙ Goal (active, 3/20 turns): Build REST API with auth
Paused:  ⏸ Goal (paused, 20/20 — budget exhausted): Build REST API
Done:    ✓ Goal done (7/20 turns): Build REST API with auth
```

## Best Practices
- Set clear, measurable goals with concrete deliverables
- Use max_turns budget to control cost (default 20)
- Let the agent work autonomously — don't interrupt unless needed
- Review results when goal completes or pauses
- Resume with reset budget if more work needed
