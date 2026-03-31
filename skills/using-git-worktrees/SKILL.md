---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
---

# Using Git Worktrees

## Overview

Git worktrees create isolated workspaces sharing the same repository, allowing work on multiple branches simultaneously without switching.

**Core principle:** Systematic directory selection + safety verification = reliable isolation.

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Branch Rules

This skill prepares a local working branch, not a public PR branch.

- **Protected branches:** `main`, `master`, `develop`, `release/*`
- **Never** start implementation directly on a protected branch
- Default working branch format: `wip/<ticket>　<short-summary>`
- If the platform or tooling rejects the Japanese branch name, fall back to an ASCII slug automatically

### Branch Note

Initialize one tracked note file per working branch:

```text
.ai-notes/wip_<ticket>　<short-summary>.md
```

Use this file to preserve context and final reporting material. The note should contain these headings:

```markdown
# 作業メモ

## 目的・依頼内容
## 実施方針
## 判断理由
## 変更内容
## 変更ファイル
## 確認結果
## 見送り事項
## 未解決事項
## 最終報告用要約
```

## Directory Selection Process

Follow this priority order:

### 1. Check Existing Directories

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**If found:** Use that directory. If both exist, `.worktrees` wins.

### 2. Check CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**If preference specified:** Use it without asking.

### 3. Ask User

If no directory exists and no CLAUDE.md preference:

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## Safety Verification

### For Project-Local Directories (.worktrees or worktrees)

**MUST verify directory is ignored before creating worktree:**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:**

Per Jesse's rule "Fix broken things immediately":
1. Add appropriate line to .gitignore
2. Commit the change
3. Proceed with worktree creation

**Why critical:** Prevents accidentally committing worktree contents to repository.

### For Global Directory (~/.config/superpowers/worktrees)

No .gitignore verification needed - outside project entirely.

## Creation Steps

### 1. Detect Project Name

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. Determine Working Branch Name

Choose a local working branch name before creating the worktree.

- Prefer `wip/<ticket>　<short-summary>`
- If there is no ticket, use a concise summary
- If Japanese branch names fail on the current platform, fall back to `wip/<ticket>-<ascii-slug>`

### 3. Create Worktree

```bash
# Determine full path
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"

# Initialize branch note
mkdir -p .ai-notes
```

### 4. Run Project Setup

Auto-detect and run appropriate setup:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 5. Verify Clean Baseline

Run tests to ensure worktree starts clean:

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**If tests fail:** Report failures, ask whether to proceed or investigate.

**If tests pass:** Report ready.

### 6. Report Location

```
Worktree ready at <full-path>
Working branch: <branch-name>
Branch note: .ai-notes/<branch-note>.md
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check CLAUDE.md → Ask user |
| Protected branch checked out | Create a local `wip/*` branch in the new worktree |
| Directory not ignored | Add to .gitignore + commit |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

## Common Mistakes

### Skipping ignore verification

- **Problem:** Worktree contents get tracked, pollute git status
- **Fix:** Always use `git check-ignore` before creating project-local worktree

### Assuming directory location

- **Problem:** Creates inconsistency, violates project conventions
- **Fix:** Follow priority: existing > CLAUDE.md > ask

### Proceeding with failing tests

- **Problem:** Can't distinguish new bugs from pre-existing issues
- **Fix:** Report failures, get explicit permission to proceed

### Hardcoding setup commands

- **Problem:** Breaks on projects using different tools
- **Fix:** Auto-detect from project files (package.json, etc.)

### Reusing public branch names for local work

- **Problem:** Interim commits leak into the branch that will be pushed later
- **Fix:** Use a local `wip/*` branch first, then prepare a clean `pr/*` branch during finishing

## Example Workflow

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b "wip/ABC-123　認証条件整理"]
[Create .ai-notes/wip_ABC-123　認証条件整理.md]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Working branch: wip/ABC-123　認証条件整理
Branch note: .ai-notes/wip_ABC-123　認証条件整理.md
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## Red Flags

**Never:**
- Create worktree without verifying it's ignored (project-local)
- Skip baseline test verification
- Proceed with failing tests without asking
- Assume directory location when ambiguous
- Skip CLAUDE.md check
- Start implementation on a protected branch
- Skip initializing the branch note when this workflow expects one

**Always:**
- Follow directory priority: existing > CLAUDE.md > ask
- Verify directory is ignored for project-local
- Start from a local `wip/*` branch when public history will be prepared later
- Initialize the branch note before implementation begins
- Auto-detect and run project setup
- Verify clean test baseline

## Integration

**Called by:**
- **brainstorming** - REQUIRED: Ensures isolated workspace (creates one or verifies existing)
- **subagent-driven-development** - REQUIRED: Ensures isolated workspace (creates one or verifies existing)
- **executing-plans** - REQUIRED: Ensures isolated workspace (creates one or verifies existing)
- Any skill needing isolated workspace

**Pairs with:**
- **finishing-a-development-branch** - REQUIRED to convert local `wip/*` work into a clean `pr/*` branch
