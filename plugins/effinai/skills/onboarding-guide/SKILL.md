---
name: onboarding-guide
description: "Create developer onboarding guides from project analysis"
version: "1.0.0"
tags: [documentation, onboarding, team, developer-experience]
createdBy: "built-in"
status: "active"
---

# Onboarding Guide Generator

## Activation
When the user asks to create an onboarding guide, developer setup guide, or getting-started documentation.

## Workflow

### Step 1: Analyze the Project
1. Read package.json for scripts, dependencies, engines
2. Check for .env.example or .env.template
3. Identify required external services (database, Redis, APIs)
4. Find existing docs/ directory content
5. Check for Docker/docker-compose setup
6. Identify the test framework and how to run tests

### Step 2: Generate the Guide

```markdown
# Developer Onboarding Guide

## Prerequisites
- [ ] Node.js [version] installed
- [ ] [Database] installed and running
- [ ] Access to [required services/accounts]
- [ ] Git configured with SSH key

## Environment Setup

### 1. Clone the Repository
[git clone command]

### 2. Install Dependencies
[package manager install command]

### 3. Environment Variables
Copy the example env file and fill in values:
[list each variable with description of where to get it]

### 4. Database Setup
[migration commands, seed data]

### 5. Start Development Server
[dev start command]

### 6. Verify Setup
[health check URL, expected output]

## Project Structure
[annotated directory tree of key folders]

## Key Concepts
- [Architecture pattern used and why]
- [Key domain terms and their meaning]
- [Important conventions to follow]

## Development Workflow
1. Create a branch from main
2. Make changes following [style guide]
3. Write tests for new code
4. Run [lint and test commands]
5. Create a PR with description

## Common Tasks
- **Add a new API endpoint**: [steps]
- **Add a database migration**: [steps]
- **Run specific tests**: [command]
- **Debug locally**: [setup steps]

## Troubleshooting
| Issue | Solution |
|-------|----------|
| [Common issue 1] | [Fix] |

## Resources
- [Architecture docs link]
- [API docs link]
- [Team communication channels]
```

### Step 3: Validate
- Verify all commands actually work
- Confirm env vars match .env.example
- Ensure prerequisites are complete

## Rules
- Only create when explicitly requested
- Base everything on actual project analysis
- Include actual working commands, not placeholders
- Cover the happy path first, then troubleshooting
- Keep it concise — link to detailed docs instead of duplicating
