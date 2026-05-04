---
name: agent-designer
description: "Design multi-agent systems with structured approaches to architecture patterns, tool design, communication strategies, guardrails, memory patterns, and evaluation frameworks."
version: "1.0.0"
tags: [agents, architecture, multi-agent, orchestration, system-design]
createdBy: "built-in"
status: "active"
---

# Agent Designer — Multi-Agent System Architecture

## Overview

Comprehensive toolkit for designing, architecting, and evaluating multi-agent systems. Covers agent architecture patterns, tool design principles, communication strategies, and performance evaluation frameworks.

## Activation

When the user asks to design multi-agent systems, create agent architectures, define agent communication patterns, or build autonomous agent workflows.

## 1. Architecture Patterns

### Single Agent
- Simple, focused tasks with clear boundaries
- Minimal complexity, easy debugging
- Direct user-agent interaction with comprehensive tool access

### Supervisor Pattern
- One supervisor agent coordinating multiple specialists
- Clear command structure, centralized decision making
- Risk: supervisor bottleneck

### Swarm Pattern
- Multiple autonomous agents with shared objectives
- High parallelism, fault tolerance, emergent intelligence
- Risk: complex coordination, potential conflicts

### Hierarchical Pattern
- Tree structure with managers and workers at different levels
- Natural organizational mapping, clear responsibilities
- Risk: communication overhead at each level

### Pipeline Pattern
- Agents arranged in sequential processing stages
- Clear data flow, specialized optimization per stage
- Risk: sequential bottlenecks, rigid processing order

## 2. Agent Role Definition

### Role Specification Framework
- **Identity:** Name, purpose statement, core competencies
- **Responsibilities:** Primary tasks, decision boundaries, success criteria
- **Capabilities:** Required tools, knowledge domains, processing limits
- **Interfaces:** Input/output formats, communication protocols
- **Constraints:** Security boundaries, resource limits, operational guidelines

### Common Archetypes

| Archetype | Role |
|-----------|------|
| **Coordinator** | Orchestrates workflows, makes high-level decisions, handles escalations |
| **Specialist** | Deep expertise in specific domain, high-quality output within narrow scope |
| **Interface** | Handles external interactions, protocol translation, auth management |
| **Monitor** | System health, performance metrics, anomaly detection, compliance |

## 3. Tool Design Principles

- **Input Validation:** Strong typing, required vs optional parameters
- **Output Consistency:** Standardized response formats, error handling
- **Idempotency:** Safe operations that can be repeated
- **Error Handling:** Graceful degradation, exponential backoff, circuit breakers
- **Versioning:** Backward compatibility, migration paths

## 4. Communication Patterns

### Message Passing
- Asynchronous messaging with structured payloads
- Delivery guarantees: at-least-once, exactly-once
- Routing: direct, publish-subscribe, broadcast

### Shared State
- Centralized data repositories with consistency models
- Conflict resolution: last-writer-wins, merge strategies

### Event-Driven
- Event sourcing with immutable event logs
- Real-time, batch, and stream processing
- Versioned event schemas

## 5. Guardrails and Safety

### Input
- Schema enforcement, content filtering, rate limiting, authentication

### Output
- Content moderation, consistency validation, audit logging

### Human-in-the-Loop
- Approval workflows for critical decisions
- Escalation triggers based on confidence thresholds
- Override mechanisms with feedback loops

## 6. Evaluation Frameworks

| Category | Metrics |
|----------|---------|
| **Task Completion** | Success rate, partial completion, failure analysis |
| **Quality** | Accuracy, relevance, completeness, consistency |
| **Cost** | Token usage, API costs, compute resources, cost per task |
| **Latency** | Response time, processing stages, queue times |

## 7. Orchestration Strategies

### Centralized
- Workflow engine manages all agents, comprehensive visibility
- Clear state management and routing rules

### Decentralized
- Peer-to-peer coordination, service discovery
- No single point of failure, consensus protocols

### Hybrid
- Centralized within domains, federated across
- Context-dependent strategy selection

## 8. Memory Patterns

### Short-Term
- Context windows, session state, cache management

### Long-Term
- Persistent storage, knowledge base, experience replay
- Memory consolidation from short to long-term

### Shared
- Collaborative knowledge across agents
- Synchronization, access control, partitioning

## 9. Failure Handling

- **Retry:** Exponential backoff with jitter, bounded attempts
- **Fallback:** Graceful degradation, alternative approaches, safe defaults
- **Circuit Breakers:** Failure detection, open/closed/half-open states, cascading failure prevention

## Architecture Decision Process

1. **Requirements Analysis:** Goals, constraints, scale
2. **Pattern Selection:** Choose architecture pattern
3. **Agent Design:** Define roles, responsibilities, interfaces
4. **Tool Architecture:** Design schemas and error handling
5. **Communication Design:** Select message patterns
6. **Safety Implementation:** Build guardrails and validation
7. **Evaluation Planning:** Define metrics and monitoring
8. **Deployment Strategy:** Plan scaling and failure handling
