---
name: performance-audit
description: "Performance profiling and optimization recommendations"
version: "1.0.0"
tags: [engineering, performance, optimization, profiling]
createdBy: "built-in"
status: "active"
---

# Performance Audit

## Activation
When the user asks to optimize performance, profile slow code, or investigate latency issues.

## Workflow

### Step 1: Identify the Target
Determine what to audit:
- Specific endpoint or function reported as slow
- Overall application startup time
- Build/compilation time
- Database query performance
- Frontend rendering performance

### Step 2: Measure Baseline
Before optimizing, establish measurable baselines:

**Node.js/Backend:**
```bash
# Check for obvious issues in package.json
cat package.json | grep -E '"start"|"build"'

# Look at bundle/build size
du -sh dist/ build/ .next/ 2>/dev/null

# Check dependency count
ls node_modules/ 2>/dev/null | wc -l
```

**Frontend:**
- Check bundle size (webpack-bundle-analyzer, source-map-explorer)
- Check for large dependencies
- Audit images and static assets

### Step 3: Static Analysis — Common Performance Issues

**Database:**
- N+1 queries (database calls inside loops)
- Missing indexes on frequently queried columns
- SELECT * instead of specific columns
- Unbounded queries without LIMIT
- Missing connection pooling

**Memory:**
- Large arrays growing without bounds
- Event listeners not cleaned up
- Closures capturing large objects unnecessarily
- Unbounded caches without eviction
- Large string concatenation in loops (use array + join)

**CPU:**
- Synchronous file I/O on hot paths
- JSON.parse/stringify on large objects in loops
- Regex with catastrophic backtracking
- Sorting without early termination
- Unnecessary deep cloning

**Network:**
- Sequential API calls that could be parallel (Promise.all)
- Missing response compression
- No request deduplication
- Large payloads without pagination
- Missing caching headers

**Frontend-Specific:**
- Re-renders caused by new object/array references
- Large component trees without virtualization
- Unoptimized images (missing WebP, no lazy loading)
- Blocking resources in <head>
- Layout thrashing (read-write-read-write DOM)

### Step 4: Recommend Fixes

Prioritize by impact:

| Priority | Category | Fix |
|----------|----------|-----|
| P0 | [area] | [fix with expected improvement] |
| P1 | [area] | [fix with expected improvement] |
| P2 | [area] | [fix with expected improvement] |

### Step 5: Implement Quick Wins
Offer to implement the top 3 easiest fixes immediately:
1. Add missing indexes
2. Parallelize sequential async calls
3. Add caching for repeated computations
4. Remove unused imports/dependencies
5. Add compression middleware

## Rules
- Always measure before and after — gut feeling is not optimization
- Focus on the hot path, not code that runs once at startup
- Prefer algorithmic improvements over micro-optimizations
- Document why each optimization matters (not just what)
- Consider readability trade-offs — 5% speedup is not worth unreadable code
