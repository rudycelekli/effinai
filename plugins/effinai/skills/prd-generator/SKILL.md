---
name: prd-generator
description: "Generate Product Requirements Documents from conversations and ideas"
version: "1.0.0"
tags: [engineering, product, documentation, planning]
createdBy: "built-in"
status: "active"
---

# PRD Generator

## Activation
When the user asks to create a PRD, product spec, feature spec, or requirements document.

## Workflow

### Step 1: Gather Requirements
Ask the user (if not already provided):
1. What problem does this solve?
2. Who is the target user?
3. What does success look like?
4. Any constraints (timeline, tech stack, budget)?

### Step 2: Generate PRD

Use this structure:

```markdown
# PRD: [Feature/Product Name]

## Overview
**Author**: [name]
**Date**: [today]
**Status**: Draft
**Priority**: P0/P1/P2/P3

## Problem Statement
[1-2 paragraphs describing the user pain point and why it matters]

## Goals & Success Metrics
| Goal | Metric | Target |
|------|--------|--------|
| [Goal 1] | [How measured] | [Target value] |

## Non-Goals
- [What this project explicitly does NOT do]

## User Stories
- As a [persona], I want to [action] so that [benefit]
- ...

## Requirements

### Functional Requirements
| ID | Requirement | Priority | Notes |
|----|------------|----------|-------|
| FR-1 | [requirement] | Must | |

### Non-Functional Requirements
| ID | Requirement | Target |
|----|------------|--------|
| NFR-1 | Performance | [e.g., < 200ms response] |
| NFR-2 | Availability | [e.g., 99.9%] |

## Technical Design (High Level)
- Architecture approach
- Key components
- Data model changes
- API changes

## Dependencies
- [External systems, teams, services]

## Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|-----------|------------|
| [risk] | High/Med/Low | High/Med/Low | [plan] |

## Timeline & Milestones
| Phase | Deliverable | Target Date |
|-------|------------|-------------|
| Phase 1 | [MVP scope] | [date] |

## Open Questions
- [ ] [Question that needs answering]
```

### Step 3: Review with User
- Present the PRD
- Ask if any sections need expansion or correction
- Iterate until the user is satisfied

## Rules
- Keep language precise and unambiguous
- Every requirement must be testable/verifiable
- Separate must-haves from nice-to-haves clearly
- Include acceptance criteria for each user story when possible
- Save the PRD in the project's docs/ directory unless user specifies otherwise
