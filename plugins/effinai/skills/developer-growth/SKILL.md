---
name: developer-growth
description: "Analyze coding patterns from chat history to identify skill gaps, generate personalized growth reports, and curate learning resources"
version: "1.0.0"
tags: [productivity, learning, growth, analysis, developer]
createdBy: "built-in"
status: "active"
---

# Developer Growth Analysis

## Activation
When the user asks to analyze their coding patterns, identify skill gaps, get feedback on their development work, or create a personal growth plan.

## Workflow

### 1. Access Chat History

Read conversation history from `~/.claude/history.jsonl`. Each line contains:
- `display`: User's message/request
- `project`: Project being worked on
- `timestamp`: Unix timestamp (milliseconds)
- `pastedContents`: Code or content pasted

Filter for entries from past 24-48 hours based on current timestamp.

### 2. Analyze Work Patterns

Extract and analyze:

**Projects and Domains:**
- Types of projects: backend, frontend, DevOps, data, etc.

**Technologies Used:**
- Languages, frameworks, and tools appearing in conversations

**Problem Types:**
- Performance optimization, debugging, feature implementation, refactoring, setup/configuration

**Challenges Encountered:**
- Repeated questions about similar topics
- Problems taking multiple attempts to solve
- Questions indicating knowledge gaps
- Complex architectural decisions

**Approach Patterns:**
- Methodical, exploratory, experimental problem-solving styles

### 3. Identify Improvement Areas (3-5)

Each area must be:
- **Specific**: Not "improve coding skills" but "Advanced TypeScript generics and type guards"
- **Evidence-based**: Grounded in actual chat history
- **Actionable**: Practical improvements that can be made
- **Prioritized**: Most impactful first

Good examples:
- "Advanced TypeScript patterns (generics, utility types, type guards) — you struggled with type safety in [project]"
- "Error handling and validation — I noticed multiple bugs from missing null checks"
- "Async/await patterns — race conditions and timing issues in recent work"
- "Database query optimization — the same query was rewritten multiple times"

### 4. Generate Report

```markdown
# Your Developer Growth Report

**Report Period**: [Date Range]
**Last Updated**: [Date and Time]

## Work Summary
[2-3 paragraphs: projects touched, technologies used, focus areas]

## Improvement Areas (Prioritized)

### 1. [Area Name]
**Why This Matters**: [Importance for their work]
**What I Observed**: [Specific evidence from chat history]
**Recommendation**: [Concrete steps to improve]
**Time to Skill Up**: [Effort estimate]

---
[Repeat for 2-4 additional areas]

## Strengths Observed
- [Things you're doing well — continue doing]

## Action Items (Priority Order)
1. [Derived from highest priority improvement area]
2. [Next area]
3. [Next area]

## Curated Learning Resources
### For: [Improvement Area]
1. **[Article Title]** - [Date]
   [Why it's relevant to your improvement area]
   [Link]
```

### 5. Curate Learning Resources

For each improvement area, search for high-quality resources:
- Target queries: "Learn [Technology] best practices", "[Tech] advanced patterns"
- Prioritize posts with high engagement
- Include 2-3 most relevant articles per area
- Provide article title, date, relevance description, and link

## Best Practices
- Run weekly to track improvement trajectory
- Pick one area at a time and focus for a few days
- Revisit report after a week to measure change
- Resources are curated for actual work, not generic topics
- Focus on actionable improvements, not vague feedback
