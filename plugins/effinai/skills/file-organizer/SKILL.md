---
name: file-organizer
description: "Intelligently organize files and folders — find duplicates, suggest structures, automate cleanup"
version: "1.0.0"
tags: [productivity, files, organization, cleanup, duplicates]
createdBy: "built-in"
status: "active"
---

# File Organizer

## Activation
When the user asks to organize files, clean up folders, find duplicates, or restructure directories.

## Workflow

### 1. Analyze Current State

```bash
# Overview of structure
ls -la [target_directory]

# File type breakdown
find [target_directory] -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Largest files
du -sh [target_directory]/* | sort -rh | head -20

# Count total files
find [target_directory] -type f | wc -l
```

Summarize: total files, type breakdown, size distribution, date ranges, organization issues.

### 2. Identify Organization Patterns

**By Type:** Documents, Images, Videos, Archives, Code/Projects, Spreadsheets, Presentations

**By Purpose:** Work vs Personal, Active vs Archive, Project-specific, Reference, Temporary

**By Date:** Current year/month, Previous years, Archive candidates

### 3. Find Duplicates

```bash
# Exact duplicates by hash
find [directory] -type f -exec md5 {} \; | sort | uniq -d

# Same-name files
find [directory] -type f -printf '%f\n' | sort | uniq -d
```

For each duplicate set: show paths, sizes, dates. Recommend which to keep (newest or best-named). ALWAYS ask for confirmation before deleting.

### 4. Propose Organization Plan

Present the plan before making changes:

```markdown
# Organization Plan for [Directory]

## Current State
- X files across Y folders
- [Size] total
- Issues: [list]

## Proposed Structure
[directory tree]

## Changes
1. Create new folders: [list]
2. Move files: [breakdown by type]
3. Rename files: [patterns]
4. Delete: [duplicates/trash]

## Files Needing Your Decision
- [uncertain files]

Ready to proceed? (yes/no/modify)
```

### 5. Execute Organization

After approval:
```bash
mkdir -p "path/to/new/folders"
cp "old/path/file.pdf" "new/path/file.pdf"  # Copy to preserve originals
```

**Rules:**
- Always confirm before deleting
- Log all moves for potential undo
- Preserve original modification dates
- Handle filename conflicts gracefully
- Stop and ask if encountering unexpected situations

### 6. Provide Summary

```markdown
# Organization Complete

## What Changed
- Created [X] new folders
- Organized [Y] files
- Freed [Z] GB by removing duplicates
- Archived [W] old files

## Maintenance Tips
1. Weekly: Sort new downloads
2. Monthly: Review and archive completed projects
3. Quarterly: Check for new duplicates
4. Yearly: Archive old files
```

## File Naming Best Practices
- Include dates: "2024-10-17-meeting-notes.md"
- Be descriptive: "q3-financial-report.xlsx"
- Use hyphens, avoid spaces
- Remove download artifacts: "document-final-v2 (1).pdf" -> "document.pdf"

## Common Tasks
- Downloads cleanup: organize by type, archive old files
- Project organization: separate active from archive
- Desktop cleanup: move everything to Documents properly
- Photo organization: sort by EXIF date into year/month folders
- Duplicate removal: find and eliminate with confirmation
