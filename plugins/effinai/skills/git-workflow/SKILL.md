---
name: git-workflow
description: "Advanced git workflow assistance: branching, rebasing, cherry-picking, conflict resolution"
version: "1.0.0"
tags: [productivity, git, workflow, version-control]
createdBy: "built-in"
status: "active"
---

# Git Workflow

## Activation
When the user asks for help with git operations, branching strategy, or resolving conflicts.

## Capabilities

### Branch Management
```bash
# Create feature branch from latest main
git checkout main && git pull && git checkout -b feature/description

# List branches with last commit date
git branch -a --sort=-committerdate | head -20

# Delete merged branches
git branch --merged main | grep -v main | xargs git branch -d
```

### Rebasing
```bash
# Rebase feature branch onto latest main
git checkout feature/my-feature
git fetch origin
git rebase origin/main

# If conflicts occur during rebase:
# 1. Fix conflicts in the files
# 2. git add <fixed-files>
# 3. git rebase --continue
# 4. Repeat until done

# Abort rebase if things go wrong
git rebase --abort
```

### Cherry-Picking
```bash
# Cherry-pick a specific commit to current branch
git cherry-pick <commit-hash>

# Cherry-pick without committing (stage only)
git cherry-pick --no-commit <commit-hash>

# Cherry-pick a range
git cherry-pick <start-hash>..<end-hash>
```

### Conflict Resolution
When the user has merge conflicts:
1. Identify conflicted files: `git status`
2. Read each conflicted file
3. Understand both sides of the conflict
4. Choose the correct resolution (ask user if ambiguous)
5. Edit the file to resolve
6. Stage the resolved file: `git add <file>`
7. Continue the merge/rebase: `git merge --continue` or `git rebase --continue`

### Stashing
```bash
# Stash current changes
git stash push -m "description of changes"

# List stashes
git stash list

# Apply most recent stash (keep in stash list)
git stash apply

# Pop most recent stash (remove from stash list)
git stash pop

# Apply specific stash
git stash apply stash@{N}
```

### History Investigation
```bash
# Find when a line was introduced
git log -S "search string" --oneline

# Find who last modified a line
git blame <file> -L <start>,<end>

# View file at a specific commit
git show <commit>:<filepath>

# Compare branches
git log main..feature/branch --oneline
```

### Undo Operations
```bash
# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Unstage a file
git restore --staged <file>

# Discard changes to a file (DESTRUCTIVE - only when user requests)
git restore <file>

# Create a revert commit (safe undo for pushed commits)
git revert <commit-hash>
```

## Branching Strategy Recommendations

**For small teams:** GitHub Flow (main + feature branches)
**For release cycles:** Git Flow (main + develop + feature + release + hotfix)
**For continuous deployment:** Trunk-based (main + short-lived feature branches)

## Rules
- NEVER run destructive commands (reset --hard, push --force, clean -f) without explicit user request
- Always suggest `git revert` over `git reset` for pushed commits
- Prefer rebase for local branches, merge for shared branches
- Always fetch before comparing branches
- Warn before force-pushing to shared branches
