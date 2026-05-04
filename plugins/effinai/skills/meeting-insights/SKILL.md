---
name: meeting-insights
description: "Analyze meeting transcripts for communication patterns, conflict avoidance, speaking ratios, and actionable feedback"
version: "1.0.0"
tags: [productivity, meetings, communication, leadership, analysis]
createdBy: "built-in"
status: "active"
---

# Meeting Insights Analyzer

## Activation
When the user asks to analyze meeting transcripts, communication patterns, speaking habits, or facilitation style.

## Workflow

### 1. Discover Available Data
- Scan folder for transcript files (.txt, .md, .vtt, .srt, .docx)
- Check for speaker labels and timestamps
- Confirm date range
- Identify user's name/identifier in transcripts

### 2. Analyze Patterns

**Conflict Avoidance:**
- Hedging language: "maybe", "kind of", "I think"
- Indirect phrasing instead of direct requests
- Changing subject when tension arises
- Agreeing without commitment: "yeah, but..."
- Not addressing obvious problems

**Speaking Ratios:**
- Percentage of meeting spent speaking
- Interruptions (by and of the user)
- Average speaking turn length
- Question vs. statement ratio

**Filler Words:**
- Count "um", "uh", "like", "you know", "actually"
- Frequency per minute or per speaking turn
- Situations where they increase (nervous, uncertain)

**Active Listening:**
- Questions referencing others' previous points
- Paraphrasing or summarizing others' ideas
- Building on others' contributions
- Clarifying questions

**Leadership & Facilitation:**
- Decision-making approach (directive vs. collaborative)
- Disagreement handling
- Inclusion of quieter participants
- Time management and agenda control
- Follow-up and action item clarity

### 3. Provide Specific Examples

For each pattern found:

```markdown
### [Pattern Name]

**Finding**: [One-sentence summary]
**Frequency**: [X times across Y meetings]

**Example: [Meeting Name/Date]** - [Timestamp]

**What Happened**:
> [Actual quote from transcript]

**Why This Matters**:
[Impact or missed opportunity]

**Better Approach**:
[Specific alternative phrasing or behavior]
```

### 4. Synthesize Insights

```markdown
# Meeting Insights Summary

**Analysis Period**: [Date range]
**Meetings Analyzed**: [X]
**Total Duration**: [X hours]

## Key Patterns
### 1. [Pattern] - Impact/Recommendation
### 2. [Pattern] - Impact/Recommendation

## Communication Strengths
1. [Strength with example]

## Growth Opportunities
1. [Area]: [Specific, actionable advice]

## Speaking Statistics
- Average speaking time: [X% of meeting]
- Questions asked: [X per meeting]
- Filler words: [X per minute]
- Interruptions: [X given / Y received]

## Next Steps
[3-5 concrete actions]
```

### 5. Trend Tracking
When analyzing multiple time periods, create comparative analysis:
- Changes in interruption frequency
- Improvement in clarifying questions
- Evolution of facilitation style
- Remaining growth areas

## Transcript Sources
- Zoom: Cloud recording transcription (.vtt/.srt)
- Google Meet: Auto-transcription docs
- Granola, Fireflies.ai, Otter.ai: Export transcripts
- Format: `YYYY-MM-DD - Meeting Name.txt`

## Common Analysis Requests
- "When do I avoid difficult conversations?"
- "How often do I interrupt others?"
- "What's my speaking vs. listening ratio?"
- "How do I handle disagreement?"
- "Am I inclusive of all voices?"
- "How has my communication changed over time?"
