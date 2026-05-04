---
name: ci-pipeline
description: "Generate pragmatic CI/CD pipelines from detected project stack signals. Auto-detects language/runtime/tooling, recommends stages, generates GitHub Actions or GitLab CI YAML with caching and matrix strategies."
version: "1.0.0"
tags: [cicd, devops, github-actions, gitlab-ci, automation, pipeline]
createdBy: "built-in"
status: "active"
---

# CI/CD Pipeline Builder

## Overview

Generate pragmatic CI/CD pipelines from detected project stack signals, not guesswork. Focuses on fast baseline generation, repeatable checks, and environment-aware deployment stages.

## Activation

When the user asks to set up CI/CD, create pipelines, migrate between CI platforms, or audit whether pipeline steps match their actual stack.

## Core Capabilities

- Detect language/runtime/tooling from repository files
- Recommend CI stages (`lint`, `test`, `build`, `deploy`)
- Generate GitHub Actions or GitLab CI starter pipelines
- Include caching and matrix strategy based on detected stack
- Emit machine-readable detection output for automation
- Keep pipeline logic aligned with project lockfiles and build commands

## Key Workflows

### 1. Detect Stack

```bash
python3 scripts/stack_detector.py --repo . --format text
python3 scripts/stack_detector.py --repo . --format json > detected-stack.json
```

### 2. Generate Pipeline From Detection

```bash
python3 scripts/pipeline_generator.py \
  --input detected-stack.json \
  --platform github \
  --output .github/workflows/ci.yml

# Or end-to-end from repo:
python3 scripts/pipeline_generator.py --repo . --platform gitlab --output .gitlab-ci.yml
```

### 3. Validate Before Merge

1. Confirm commands exist in project (`test`, `lint`, `build`)
2. Run generated pipeline locally where possible
3. Ensure required secrets/env vars are documented
4. Keep deploy jobs gated by protected branches/environments

### 4. Add Deployment Stages Safely

- Start with CI-only (`lint/test/build`)
- Add staging deploy with explicit environment context
- Add production deploy with manual gate/approval
- Keep rollout/rollback commands explicit and auditable

## Detection Heuristics

The stack detector prioritizes deterministic file signals:
- Lockfiles determine package manager preference
- Language manifests determine runtime families
- Script commands drive lint/test/build commands
- Missing scripts trigger conservative placeholder commands

## Generation Strategy

Start with a minimal, reliable pipeline:

1. Checkout and setup runtime
2. Install dependencies with cache strategy
3. Run lint, test, build in separate steps
4. Publish artifacts only after passing checks

Then layer advanced behavior (matrix builds, security scans, deploy gates).

## Platform Decision Notes

- **GitHub Actions** for tight GitHub ecosystem integration
- **GitLab CI** for integrated SCM + CI in self-hosted environments
- Keep one canonical pipeline source per repo to reduce drift

## Common Pitfalls

1. Copying a Node pipeline into Python/Go repos
2. Enabling deploy jobs before stable tests
3. Forgetting dependency cache keys
4. Running expensive matrix builds for every trivial branch
5. Missing branch protections around prod deploy jobs
6. Hardcoding secrets in YAML instead of CI secret stores

## Best Practices

1. Detect stack first, then generate pipeline
2. Keep generated baseline under version control
3. Add one optimization at a time (cache, matrix, split jobs)
4. Require green CI before deployment jobs
5. Use protected environments for production credentials
6. Regenerate pipeline when stack changes significantly

## Validation Checklist

- [ ] Generated YAML parses successfully
- [ ] All referenced commands exist in the repo
- [ ] Cache strategy matches package manager
- [ ] Required secrets are documented, not embedded
- [ ] Branch/protected-environment rules match org policy

## Scaling Guidance

- Split long jobs by stage when runtime exceeds 10 minutes
- Introduce test matrix only when compatibility truly requires it
- Separate deploy jobs from CI jobs to keep feedback fast
- Track pipeline duration and flakiness as first-class metrics
