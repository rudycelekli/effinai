---
name: code-search
description: "Intelligent codebase search, navigation, and understanding"
version: "1.0.0"
tags: [productivity, search, navigation, codebase]
createdBy: "built-in"
status: "active"
---

# Code Search

## Activation
When the user asks to find code, locate a function, trace a feature, or understand how something works.

## Strategies

### Find a Function/Class/Type
1. Search by name across all files:
   - Use Grep to search for the definition
   - Check common patterns: `function name`, `class Name`, `const name`, `export`, `def name`
2. If not found by name, search by behavior description in comments/docs

### Trace a Feature
1. Start from the user-facing entry point (route, UI component, CLI command)
2. Follow the call chain inward:
   - Read the handler/component
   - Find the service/logic it calls
   - Find the data access layer
   - Identify external calls
3. Document the flow as a chain: `Route -> Controller -> Service -> Repository -> DB`

### Find All Usages
1. Search for imports of the module/function
2. Search for direct references to the name
3. Check test files for usage patterns
4. Check config files for references

### Understand a Module
1. Read the main file (index.ts, mod.rs, __init__.py)
2. List all exports — these are the public API
3. Read the types/interfaces file
4. Check tests for usage examples
5. Summarize: purpose, public API, dependencies, dependents

### Search Patterns

| Goal | Search Strategy |
|------|----------------|
| Find definition | `function\s+name\|const name\|class Name` |
| Find usage | `import.*name\|require.*name` |
| Find tests | Search in `**/*.test.*` or `**/*.spec.*` |
| Find config | Search in `*.config.*`, `*.json`, `*.yaml` |
| Find TODO/FIXME | `TODO\|FIXME\|HACK\|XXX` |
| Find error handling | `catch\|throw\|Error\|reject` |
| Find API calls | `fetch\|axios\|http\.\|request\(` |
| Find DB queries | `query\|findOne\|findMany\|SELECT\|INSERT` |

### Output Format
When presenting search results:
```
## Search Results: "[query]"

### Definitions (N found)
- [file:line] — [context snippet]

### Usages (N found)
- [file:line] — [context snippet]

### Related
- [file] — [why it's related]
```

## Rules
- Always show file path and line number
- Show enough context (2-3 lines) around matches
- Distinguish between definitions and usages
- Search broadly first, then narrow down
- Check both source and test files
- Use Glob for file pattern matching, Grep for content search
