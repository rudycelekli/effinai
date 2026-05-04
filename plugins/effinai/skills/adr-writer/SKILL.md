---
name: adr-writer
description: "Architecture Decision Record writer following standard ADR format"
version: "1.0.0"
tags: [documentation, architecture, adr, decisions]
createdBy: "built-in"
status: "active"
---

# ADR Writer

## Activation
When the user asks to document an architecture decision, create an ADR, or record a technical decision.

## Workflow

### Step 1: Gather Decision Context
Ask the user (if not provided):
1. What decision was made (or needs to be made)?
2. What was the context/problem?
3. What options were considered?
4. Why was this option chosen?

### Step 2: Determine ADR Number
```bash
# Find existing ADRs
ls docs/adr/ 2>/dev/null || ls docs/decisions/ 2>/dev/null || echo "No ADR directory found"
```
Use the next sequential number, or start at 001.

### Step 3: Generate ADR

```markdown
# ADR-NNN: [Title of Decision]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-XXX]

## Date
[YYYY-MM-DD]

## Context
[What is the issue that we're seeing that is motivating this decision?
What forces are at play? Include technical, business, and social factors.
This section describes the problem without prescribing a solution.]

## Decision
[What is the change that we're proposing and/or doing?
State the decision clearly and concisely.]

## Options Considered

### Option 1: [Name]
- Pros: [list]
- Cons: [list]

### Option 2: [Name]
- Pros: [list]
- Cons: [list]

### Option 3: [Name]
- Pros: [list]
- Cons: [list]

## Consequences

### Positive
- [What becomes easier or better]

### Negative
- [What becomes harder or worse]

### Risks
- [What could go wrong and how we mitigate it]

## References
- [Links to relevant documents, RFCs, discussions]
```

### Step 4: Save the ADR
Save to `docs/adr/NNN-decision-title.md` (create directory if needed).

### Step 5: Update ADR Index
If an index file exists (docs/adr/README.md or INDEX.md), add the new entry.

## Rules
- Keep the title short and descriptive
- Context should explain WHY, not WHAT
- Always list alternatives that were considered
- Be honest about negative consequences
- Date every ADR
- Use sequential numbering
- Only create when the user explicitly asks for it
