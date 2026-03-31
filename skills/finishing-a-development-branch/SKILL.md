---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# Finishing a Development Branch

## Overview

Guide completion of development work by turning local `wip/*` work into a clean `pr/*` branch without leaking interim history or branch notes.

**Core principle:** Verify tests -> update the branch note -> prepare a clean `pr/*` branch locally -> push only when explicitly requested.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with local PR preparation or push until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Determine Base Branch

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Or ask: "This branch split from main - is that correct?"

### Step 3: Identify Current Branch Role

Determine whether the current branch is:

- `wip/*` - local work branch with interim commits and `.ai-notes`
- `pr/*` - clean public branch prepared for push
- a protected branch (`main`, `master`, `develop`, `release/*`) - **stop and fix this before continuing**

If you are still on a protected branch, stop and move the work onto a local `wip/*` branch first.

### Step 4: Present Options

Present exactly these 4 options:

```
Implementation complete. What would you like to do?

1. Prepare a clean pr/* branch locally
2. Push the current pr/* branch and create a Pull Request
3. Keep the current wip/* / pr/* branches as-is
4. Discard this work

Which option?
```

**Don't add explanation** - keep options concise.

### Step 5: Execute Choice

#### Option 1: Prepare a Clean `pr/*` Branch Locally

Use this when the work is reviewed and ready to be turned into publishable history, but push has not been explicitly requested.

```bash
# Review the local checkpoint history first
git log --oneline --decorate --max-count=20

# Switch to the base branch
git checkout <base-branch>

# Create the clean PR branch
git checkout -b <pr-branch>
```

Then:
- Re-apply only the reviewed code changes from `wip/*`
- Do **not** carry `.ai-notes/` into `pr/*`
- Rewrite the public history into **2-4 logical commits**
- Use **Japanese** commit messages in `チケット番号 + 要約` format

Typical ways to do this:
- `git cherry-pick -n <selected-wip-commits>` followed by curated commits
- patch-based replay
- a careful manual restaging pass when the changes are small

Before committing, ensure:

```bash
git status --short
git diff --stat <base-branch>..HEAD
```

If `.ai-notes/` appears in `git status`, remove it from the `pr/*` branch before committing.

Report:

```
Clean pr/* branch prepared locally at <branch-name>.
Public history rewritten into <N> logical commits.
Nothing has been pushed.
```

Keep the `wip/*` branch and note file until push or merge is complete.

#### Option 2: Push and Create PR

Only proceed if the user **explicitly** asked to push or create a PR.

If the current branch is still `wip/*`, complete Option 1 first.

```bash
# Push branch
git push -u origin <pr-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets in Japanese>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

After a successful push:
- You may clean up the local `wip/*` branch and `.ai-notes/`
- Keep the local `pr/*` branch unless the user asks to remove it

#### Option 3: Keep As-Is

Report: "Keeping the current branches as-is. Local worktree preserved at <path>."

Do **not** clean up `wip/*`, `pr/*`, or `.ai-notes/`.

#### Option 4: Discard

**Confirm first:**
```
This will permanently delete:
- Branches <wip-branch> and <pr-branch-if-present>
- Local note file(s) under .ai-notes/
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

Wait for exact confirmation.

If confirmed:
```bash
git checkout <base-branch>
git branch -D <wip-branch>
git branch -D <pr-branch> 2>/dev/null || true
```

Then remove the worktree and local note file(s).

### Step 6: Cleanup Worktree

**For Option 1:** Keep the worktree. The user still needs it for local review and possible fixes.

**For Option 2:** After a successful push, remove the local `wip/*` worktree if the user does not need it anymore.

**For Option 3:** Keep the worktree.

**For Option 4:** Remove the worktree after confirmation.

## Quick Reference

| Option | Prepare `pr/*` | Push | Keep `wip/*` | Carry `.ai-notes/` into `pr/*` |
|--------|------------------|------|--------------|-------------------------------|
| 1. Prepare locally | ✓ | - | ✓ | - |
| 2. Push and create PR | maybe (if needed first) | ✓ | - after push | - |
| 3. Keep as-is | - | - | ✓ | n/a |
| 4. Discard | - | - | - | - |

## Common Mistakes

**Skipping test verification**
- **Problem:** Prepare a public branch from broken code
- **Fix:** Always verify tests before offering options

**Open-ended questions**
- **Problem:** "What should I do next?" → ambiguous
- **Fix:** Present exactly 4 structured options

**Pushing without explicit request**
- **Problem:** Unexpected publication of work-in-progress history
- **Fix:** Treat push and PR creation as allowed only after a direct instruction

**Carrying `.ai-notes/` into `pr/*`**
- **Problem:** Internal manufacturing notes leak into public history
- **Fix:** Keep `.ai-notes/` on `wip/*` only and remove it before any `pr/*` commit

## Red Flags

**Never:**
- Proceed with failing tests
- Push or create a PR without an explicit request
- Publish `wip/*` history directly
- Carry `.ai-notes/` into `pr/*`
- Work directly on protected branches
- Delete work without confirmation
- Force-push without explicit request

**Always:**
- Verify tests before offering options
- Present exactly 4 options
- Get typed confirmation for Option 4
- Prepare a clean `pr/*` branch locally before push
- Use Japanese public commit messages in `チケット番号 + 要約` format

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - After all tasks complete
- **executing-plans** (Step 5) - After all batches complete

**Pairs with:**
- **using-git-worktrees** - Creates the `wip/*` workspace and initializes `.ai-notes/`
