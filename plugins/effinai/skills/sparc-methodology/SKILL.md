---
name: sparc-methodology
description: "SPARC 5-phase development methodology with quality gates and agent assignments"
version: "1.0.0"
tags: [methodology, development, planning, quality]
---

# SPARC Methodology

A structured 5-phase approach to software development: Specification, Pseudocode, Architecture, Refinement, Completion.

## Phase 1: Specification

**Goal**: Define what we're building with zero ambiguity.

**Steps**:
1. Gather requirements from user input, issues, or PRDs
2. Write acceptance criteria in Given/When/Then format
3. Identify constraints (performance, security, compatibility)
4. Define scope boundaries - what is explicitly NOT included
5. List dependencies and integration points

**Quality Gate**:
- [ ] All acceptance criteria are testable
- [ ] Constraints have measurable thresholds
- [ ] Scope boundaries documented
- [ ] Stakeholder sign-off obtained

**Agent Assignment**: `researcher` + `planner`

**Output**: `specs/{feature-name}.spec.md` containing requirements, acceptance criteria, constraints, and scope.

## Phase 2: Pseudocode

**Goal**: Design the solution logic before writing real code.

**Steps**:
1. Break the feature into discrete functions/modules
2. Write pseudocode for each function with clear inputs/outputs
3. Identify edge cases and error paths
4. Map data flow between components
5. Annotate complexity (O(n), O(log n), etc.) for critical paths

**Quality Gate**:
- [ ] Every acceptance criterion maps to pseudocode
- [ ] Error handling paths are explicit
- [ ] No ambiguous logic ("handle appropriately" is banned)
- [ ] Peer review of pseudocode completed

**Agent Assignment**: `planner` + `coder`

**Output**: Pseudocode document with function signatures, logic flow, and edge case handling.

## Phase 3: Architecture

**Goal**: Design the system structure, contracts, and data models.

**Steps**:
1. Define component boundaries and responsibilities
2. Design API contracts (request/response schemas, error codes)
3. Create data models with field types, constraints, and relationships
4. Choose patterns (repository, factory, strategy, observer, etc.)
5. Plan infrastructure (databases, queues, caches, external services)
6. Document security considerations (auth, input validation, rate limiting)

**Quality Gate**:
- [ ] API contracts have versioning strategy
- [ ] Data models handle migration paths
- [ ] Security threat model documented
- [ ] Performance budget defined per endpoint/operation
- [ ] No circular dependencies between components

**Agent Assignment**: `system-architect` + `security-architect`

**Output**: Architecture Decision Record (ADR), API contract specs, data model diagrams, component dependency map.

## Phase 4: Refinement

**Goal**: Iteratively implement and improve until quality gates pass.

**Steps**:
1. Implement code following pseudocode and architecture
2. Write tests alongside implementation (TDD preferred)
3. Run quality gates after each iteration:
   - Type checking passes
   - Linter passes with zero warnings
   - Unit tests pass with >80% coverage on new code
   - Integration tests pass
   - No security vulnerabilities (dependency scan)
4. Refactor based on review feedback
5. Optimize hot paths identified in architecture phase

**Quality Gate**:
- [ ] All tests pass
- [ ] Coverage threshold met
- [ ] No lint warnings
- [ ] Type-safe (no `any` escape hatches without justification)
- [ ] Code review approved
- [ ] Performance benchmarks within budget

**Agent Assignment**: `coder` + `tester` + `reviewer`

**Iteration Protocol**: Max 3 refinement cycles. If quality gates don't pass after 3 cycles, escalate to architect for redesign.

## Phase 5: Completion

**Goal**: Ship with confidence.

**Steps**:
1. Run full test suite (unit, integration, e2e)
2. Generate/update API documentation
3. Write migration scripts if needed
4. Create deployment plan (rollback strategy, feature flags)
5. Update changelog with user-facing descriptions
6. Final security scan
7. Deploy to staging, verify, then production

**Quality Gate**:
- [ ] All tests green in CI
- [ ] Documentation matches implementation
- [ ] Rollback procedure tested
- [ ] Monitoring/alerting configured
- [ ] Stakeholder acceptance testing passed

**Agent Assignment**: `reviewer` + `tester` + deployment team

**Output**: Deployed feature, updated docs, changelog entry, monitoring dashboards.

## Phase Transitions

Each phase transition requires the previous phase's quality gate to pass:

```
Specification --[gate]--> Pseudocode --[gate]--> Architecture --[gate]--> Refinement --[gate]--> Completion
      |                       |                      |                       |
      v                       v                      v                       v
  Reject & revise        Reject & revise        Reject & revise        Max 3 cycles
```

## Quick Reference

| Phase | Duration Target | Key Artifact | Primary Agent |
|-------|----------------|--------------|---------------|
| Specification | 10-15% of total | Requirements doc | researcher |
| Pseudocode | 10-15% of total | Logic document | planner |
| Architecture | 15-20% of total | ADR + contracts | architect |
| Refinement | 40-50% of total | Working code + tests | coder |
| Completion | 10-15% of total | Deployed feature | reviewer |

## Anti-Patterns to Avoid

- Skipping Pseudocode ("just start coding") leads to rework
- Architecture without constraints leads to over-engineering
- Refinement without quality gates leads to technical debt
- Completion without rollback strategy leads to incidents
