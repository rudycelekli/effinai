---
name: codebase-onboarding
description: "Analyze a codebase and generate onboarding documentation for engineers, tech leads, and contractors. Architecture discovery, key file inventory, local setup guidance, and audience-aware documentation."
version: "1.0.0"
tags: [onboarding, documentation, developer-experience, architecture, setup]
createdBy: "built-in"
status: "active"
---

# Codebase Onboarding

## Overview

Analyze a codebase and generate onboarding documentation for engineers, tech leads, and contractors. Optimized for fast fact-gathering and repeatable onboarding outputs.

## Activation

When the user asks to onboard a new team member, rebuild stale project docs, prepare handoff documentation, or create a standardized onboarding packet.

## Core Capabilities

- Architecture and stack discovery from repository signals
- Key file and config inventory for new contributors
- Local setup and common-task guidance generation
- Audience-aware documentation framing
- Debugging and contribution checklist scaffolding

## Quick Start

```bash
# 1) Gather codebase facts
python3 scripts/codebase_analyzer.py /path/to/repo

# 2) Export machine-readable output
python3 scripts/codebase_analyzer.py /path/to/repo --json
```

## Recommended Workflow

1. Run `codebase_analyzer.py` against the target repository
2. Capture key signals: file counts, detected languages, config files, top-level structure
3. Fill the onboarding template
4. Tailor output depth by audience:
   - **Junior**: setup + guardrails + common tasks
   - **Senior**: architecture + operational concerns + design decisions
   - **Contractor**: scoped ownership + integration boundaries + delivery expectations

## Onboarding Document Structure

### 1. Project Overview
- What the project does (1-2 sentences)
- Tech stack (languages, frameworks, databases)
- Architecture style (monolith, microservices, monorepo)
- Key dependencies and external services

### 2. Getting Started
- Prerequisites (runtime versions, tools)
- Clone and setup commands
- Environment variables needed
- How to run locally
- How to run tests
- Verification steps ("you'll know it's working when...")

### 3. Architecture Overview
- Directory structure with purpose of each top-level folder
- Data flow diagram (request -> response path)
- Key abstractions and patterns used
- Where business logic lives vs. infrastructure

### 4. Common Tasks
- Adding a new API endpoint
- Adding a database migration
- Running/writing tests
- Creating a PR (branch naming, CI checks)
- Deploying to staging/production

### 5. Debugging Guide
- How to read logs
- Common error messages and their fixes
- How to connect to staging/production databases (safely)
- Performance profiling tools available

### 6. Key Files Reference
- Configuration files and what they control
- Entry points (main, index, app)
- Shared types/interfaces
- Database schema/migration files
- CI/CD pipeline files

### 7. Team and Process
- Code review expectations
- Branch strategy (trunk-based, git flow, etc.)
- Release process
- On-call rotation (if applicable)
- Where to ask for help

## Audience Adaptation

### Junior Engineers
Focus on: step-by-step setup, common tasks with examples, guardrails (what NOT to do), glossary of project-specific terms, who to ask for help.

### Senior Engineers
Focus on: architecture decisions and their rationale, performance characteristics, scaling considerations, technical debt and known issues, system boundaries and contracts.

### Contractors
Focus on: scoped area of ownership, integration points with other teams, definition of done, access and permissions needed, communication channels.

## Common Pitfalls

- Writing docs without validating setup commands on a clean environment
- Mixing architecture deep-dives into contractor-oriented docs
- Omitting troubleshooting and verification steps
- Letting onboarding docs drift from current repo state
- Assuming tools are installed (specify versions and install commands)

## Best Practices

1. **Keep setup instructions executable and time-bounded** — "you should be running locally within 30 minutes"
2. **Document the why for key architectural decisions** — not just what, but why this approach was chosen
3. **Update docs in the same PR as behavior changes** — treat docs as code
4. **Treat onboarding docs as living operational assets** — not one-time deliverables
5. **Include verification steps** — after each setup step, tell them how to confirm it worked
6. **Test the docs** — have someone follow them from scratch on a fresh machine
