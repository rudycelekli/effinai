---
name: architecture-review
description: "System architecture review with recommendations for scalability and maintainability"
version: "1.0.0"
tags: [engineering, architecture, review, scalability]
createdBy: "built-in"
status: "active"
---

# Architecture Review

## Activation
When the user asks to review system architecture, evaluate design decisions, or assess scalability.

## Workflow

### Step 1: Map the System
1. Read the project structure (package.json, directory layout, config files)
2. Identify the tech stack (language, framework, database, infrastructure)
3. Map the module dependency graph:
   - Entry points (main, index, app)
   - Core modules and their responsibilities
   - External integrations (APIs, databases, message queues)
4. Identify architectural pattern in use (MVC, hexagonal, microservices, monolith, etc.)

### Step 2: Evaluate Against Principles

**Separation of Concerns**
- Are layers clearly defined? (presentation, business logic, data access)
- Does each module have a single responsibility?
- Are cross-cutting concerns (logging, auth, error handling) handled consistently?

**Dependency Management**
- Do dependencies flow in one direction? (outer layers depend on inner)
- Are external services abstracted behind interfaces?
- Is dependency injection used where appropriate?

**Scalability**
- Can the system scale horizontally?
- Are there single points of failure?
- Is state management distributed or centralized?
- Are there bottlenecks (shared mutable state, synchronous chains)?

**Data Architecture**
- Is the data model normalized appropriately?
- Are read and write paths separated where beneficial?
- Is caching used effectively?
- Are database queries optimized (indexes, query plans)?

**Resilience**
- How does the system handle partial failures?
- Are there circuit breakers, retries, timeouts?
- Is there graceful degradation?
- What happens when a dependency is unavailable?

**Observability**
- Is structured logging in place?
- Are metrics collected (latency, throughput, error rates)?
- Is distributed tracing enabled?
- Are health checks implemented?

### Step 3: Output Format

```
## Architecture Review: [Project Name]

### System Overview
[Diagram or description of the architecture]

### Strengths
- [What the architecture does well]

### Concerns
| Area | Issue | Severity | Recommendation |
|------|-------|----------|----------------|
| [area] | [problem] | Critical/High/Medium/Low | [fix] |

### Technical Debt
- [Identified areas of accumulated debt]

### Recommendations (Prioritized)
1. [Immediate] — [action]
2. [Short-term] — [action]
3. [Long-term] — [action]

### Architecture Decision Records Needed
- ADR-NNN: [Decision that should be documented]
```

## Rules
- Base recommendations on evidence from the codebase, not assumptions
- Consider the team size and velocity when making recommendations
- Avoid recommending rewrites unless truly necessary — prefer incremental improvement
- Acknowledge trade-offs: every architecture decision has costs and benefits
