---
name: spec-driven
description: "Spec-first development workflow. Write the specification BEFORE code. Enforces RFC 2119 requirements, Given/When/Then acceptance criteria, bounded autonomy rules, and test generation from specs."
version: "1.0.0"
tags: [specification, requirements, tdd, acceptance-criteria, planning, quality]
createdBy: "built-in"
status: "active"
---

# Spec-Driven Workflow

## Overview

Spec-driven workflow enforces a single, non-negotiable rule: **write the specification BEFORE you write any code.** Not alongside. Not after. Before.

This is not documentation. This is a contract. A spec defines what the system MUST do, what it SHOULD do, and what it explicitly WILL NOT do. Every line of code traces back to a requirement. Every test traces back to an acceptance criterion.

### Why Spec-First Matters

1. **Eliminates rework.** 60-80% of defects originate from requirements, not implementation.
2. **Forces clarity.** If you cannot write what the system should do in plain language, you do not understand the problem.
3. **Enables parallelism.** Once approved, frontend, backend, QA, and docs can start simultaneously.
4. **Creates accountability.** The spec is the definition of done.
5. **Feeds TDD directly.** Given/When/Then criteria translate 1:1 into test cases.

### The Iron Law

```
NO CODE WITHOUT AN APPROVED SPEC.
NO EXCEPTIONS. NO "QUICK PROTOTYPES." NO "I'LL DOCUMENT IT LATER."
```

## The Spec Format

Every spec follows this structure. No sections are optional.

| # | Section | Key Rules |
|---|---------|-----------|
| 1 | **Title and Metadata** | Author, date, status (Draft/In Review/Approved/Superseded), reviewers |
| 2 | **Context** | Why this feature exists. 2-4 paragraphs with evidence. |
| 3 | **Functional Requirements** | RFC 2119 keywords (MUST/SHOULD/MAY). Numbered FR-N. Atomic and testable. |
| 4 | **Non-Functional Requirements** | Performance, security, accessibility — all with measurable thresholds. |
| 5 | **Acceptance Criteria** | Given/When/Then format. Every AC references at least one FR or NFR. |
| 6 | **Edge Cases** | Numbered EC-N. Cover failure modes for every external dependency. |
| 7 | **API Contracts** | TypeScript-style interfaces. Cover success and error responses. |
| 8 | **Data Models** | Table format with field, type, constraints. |
| 9 | **Out of Scope** | Explicit exclusions with reasons. Prevents scope creep. |

### RFC 2119 Keywords

| Keyword | Meaning |
|---------|---------|
| **MUST** | Absolute requirement |
| **MUST NOT** | Absolute prohibition |
| **SHOULD** | Recommended. Omit only with documented justification |
| **MAY** | Optional. Implementer's discretion |

## Bounded Autonomy Rules

### STOP and Ask When:

1. **Scope creep detected.** Implementation requires something not in the spec.
2. **Ambiguity exceeds 30%.** Cannot determine correct behavior for >30% of a requirement.
3. **Breaking changes required.** Would change existing API, DB schema, or public interface.
4. **Security implications.** Touches auth, encryption, or PII handling.
5. **Performance characteristics unknown.** No way to measure or guarantee thresholds.
6. **Cross-team dependencies.** Requires coordination with another team/service.

### Continue Autonomously When:

1. Spec is clear and unambiguous for the current task.
2. All acceptance criteria have passing tests and you are refactoring internals.
3. Changes are non-breaking.
4. Implementation is a direct translation of a well-defined acceptance criterion.

### Escalation Protocol

When you must stop, provide:
- What requirement is blocking (e.g., FR-3)
- Specific, answerable question
- Options considered with pros/cons
- Your recommendation
- Impact of waiting

## Workflow — 6 Phases

### Phase 1: Gather Requirements
Understand what needs to be built and why. Interview the user, read existing code, identify constraints, list unknowns.

### Phase 2: Write Spec
Fill every section of the template. Number all requirements (FR-*, NFR-*, AC-*, EC-*, OS-*). Use RFC 2119 keywords precisely.

### Phase 3: Validate Spec
```bash
python spec_validator.py --file spec.md --strict
```
Score must reach 80+ and all manual checklist items pass.

### Phase 4: Generate Tests
```bash
python test_extractor.py --file spec.md --framework pytest --output tests/
```
Each AC becomes test cases. All tests MUST fail initially (red phase of TDD).

### Phase 5: Implement
Pick one AC at a time. Make its tests pass with minimal code. Run full suite. Commit. Repeat.

Rules:
- Do NOT implement anything not in the spec
- Do NOT optimize before all AC pass
- If you discover a missing requirement, STOP and update the spec first

### Phase 6: Self-Review
Verify implementation matches spec before marking done.

## Self-Review Checklist

- [ ] Every acceptance criterion has a passing test
- [ ] Every edge case has a test
- [ ] No scope creep — implementation matches spec exactly
- [ ] API contracts match implementation (field names, types, status codes)
- [ ] Error scenarios tested
- [ ] Non-functional requirements verified with evidence
- [ ] Data model matches spec
- [ ] Out-of-scope items not built

## Anti-Patterns

1. **Coding Before Spec Approval** — Review will surface changes; now you have code implementing a rejected design
2. **Vague Acceptance Criteria** — "should work well" is untestable; rewrite until a machine can verify it
3. **Missing Edge Cases** — Happy path only = developers invent error handling on the fly
4. **Spec as Post-Hoc Documentation** — If written after code, it describes what was built, not what should have been
5. **Gold-Plating Beyond Spec** — Untested, unreviewed design. If not in spec, don't build it
6. **Orphaned AC** — Every AC-* MUST reference at least one FR-* or NFR-*
7. **Skipping Validation** — Missing sections discovered during implementation cause blocking delays

## Tools

| Script | Purpose |
|--------|---------|
| `spec_generator.py` | Generate spec template from feature name/description |
| `spec_validator.py` | Validate spec completeness (0-100 score) |
| `test_extractor.py` | Extract test stubs from acceptance criteria |
