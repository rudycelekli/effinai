---
name: exa-codesearch
description: "Search code examples, documentation, and technical content across the web using Exa's code-specialized search API"
version: "1.0.0"
tags: [search, code, web, documentation, examples, api]
---

# Exa Code Search

## Activation
When the agent needs to find code examples, library documentation, API usage patterns, or technical solutions from across the web — not from the local codebase.

## Concept

A web-based code search tool that queries Exa's code-specialized search API to find relevant code examples, documentation, and technical content. Unlike local grep/search, this searches the entire internet for code knowledge. Ported from jcode's codesearch tool.

## Usage

```
codesearch:
  query: "FastAPI dependency injection with async database sessions"
  max_tokens: 5000    # control response size (1000-50000)
```

## Parameters

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| `query` | required | - | Natural language search query |
| `max_tokens` | 5000 | 1000-50000 | Max tokens in response |

## Good Queries

### Find Implementation Patterns
```
"React Server Components with streaming and Suspense boundaries"
"Rust async trait with lifetime parameters"
"PostgreSQL recursive CTE for tree traversal"
```

### Find API Usage
```
"OpenAI function calling with structured outputs TypeScript"
"Stripe webhook signature verification Python"
"AWS S3 presigned URL with custom expiry Go"
```

### Find Solutions to Problems
```
"Fix React hydration mismatch with dynamic content"
"Handle WebSocket reconnection with exponential backoff"
"SQLAlchemy N+1 query detection and eager loading"
```

## Output Format

Results include:
- Source URLs with titles
- Relevant code snippets
- Documentation excerpts
- Context around the code

Results are truncated to max_tokens to prevent context overflow.

## When to Use

| Scenario | Use This | Not This |
|----------|----------|----------|
| Find how to use an API | exa-codesearch | local grep |
| Find code examples | exa-codesearch | web search |
| Understand a library | exa-codesearch | reading source |
| Debug a framework issue | exa-codesearch | stack overflow |
| Find local code | local grep | exa-codesearch |
| Search project files | agentgrep | exa-codesearch |

## Rate Limiting

- Timeout: 30 seconds per request
- Results are cached for the session duration
- Use specific queries to get better results in fewer calls
