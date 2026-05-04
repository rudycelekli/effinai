---
name: readme-generator
description: "Generate comprehensive README.md files from project analysis"
version: "1.0.0"
tags: [documentation, readme, project]
createdBy: "built-in"
status: "active"
---

# README Generator

## Activation
When the user explicitly asks to create or update a README file.

## Workflow

### Step 1: Analyze the Project
1. Read package.json / pyproject.toml / Cargo.toml for metadata
2. Scan directory structure for architecture understanding
3. Check for existing README to update vs create
4. Identify key features from source code
5. Check for existing docs, examples, tests

### Step 2: Generate README Structure

```markdown
# Project Name

One-line description of what the project does.

## Features
- Key feature 1
- Key feature 2

## Quick Start

### Prerequisites
- Node.js >= 20
- etc.

### Installation
[package manager install commands]

### Usage
[minimal working example]

## Configuration
[environment variables, config files]

## API Reference
[if applicable — key endpoints or functions]

## Development

### Setup
[dev environment setup steps]

### Testing
[how to run tests]

### Building
[how to build for production]

## Architecture
[brief overview of project structure]

## Contributing
[contribution guidelines or link to CONTRIBUTING.md]

## License
[license type and link]
```

### Step 3: Populate Each Section
- **Features**: Derive from source code analysis, not guesswork
- **Installation**: Use actual package name and manager
- **Usage**: Provide a real, working code example
- **Configuration**: List actual env vars from .env.example or code
- **API**: Document actual endpoints/functions found in code
- **Development**: Use actual scripts from package.json

### Step 4: Add Badges (if applicable)
```markdown
[![CI](badge-url)](link)
[![npm version](badge-url)](link)
[![License](badge-url)](link)
```

## Rules
- Only create README.md when the user explicitly requests it
- Base all content on actual project analysis, not templates
- Keep examples minimal but functional
- Include actual CLI commands that work
- Don't include sections that don't apply to the project
- Use the project's actual package name and paths
