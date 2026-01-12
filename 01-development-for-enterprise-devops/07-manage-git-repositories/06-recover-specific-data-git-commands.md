# Recover Specific Data by Using Git Commands

## Key Concepts
- **Data Recovery**: Restore commits, files, or code accidentally deleted or modified
- **Git Reflog**: Record of all commits, including deleted ones
- **Git Blame**: Track line-by-line code authorship and changes
- **Safety First**: Always backup before attempting recovery operations

## Git Data Recovery Commands

### Recovering Commits

| Command | Purpose |
|---------|---------|
| `git log` | Show commit history to find commits to recover |
| `git checkout <commit>` | Switch to specific commit, recovering that state |
| `git cherry-pick <commit>` | Apply changes from specific commit to current branch |

```bash
# View commit history
git log --oneline

# Recover state at specific commit
git checkout abc123def

# Apply specific commit to current branch
git cherry-pick abc123def
```

### Recovering Files

| Command | Purpose |
|---------|---------|
| `git checkout <commit> <file>` | Restore specific file from previous commit |
| `git restore <file>` | Discard local changes, restore to last commit |

```bash
# Restore file from specific commit
git checkout abc123 -- path/to/file.txt

# Discard uncommitted changes (Git 2.23+)
git restore path/to/file.txt

# Old syntax (before Git 2.23)
git checkout -- path/to/file.txt
```

### Recovering Specific Lines of Code

| Command | Purpose |
|---------|---------|
| `git blame <file>` | Show revision and author of each line |
| `git show <commit>:<file>` | Show file content at specific commit |

```bash
# See who changed each line and when
git blame src/app.js

# View file as it was at specific commit
git show abc123:src/app.js
```

### Recovering Deleted Commits or Branches

| Command | Purpose |
|---------|---------|
| `git reflog` | Show record of all commits, including deleted ones |
| `git fsck --lost-found` | Find commits not reachable from any branch/tag |

```bash
# View reflog to find deleted commit SHA
git reflog

# Recover deleted commit or branch
git checkout <commit-sha>
git branch recovered-branch <commit-sha>

# Find lost commits
git fsck --lost-found

# Recover lost commit
git show <commit-sha>
git cherry-pick <commit-sha>
```

## Sample Scenario: Recovering Deleted File

### Problem
File `example.txt` was accidentally deleted from repository. Need to recover it.

### Solution Steps

**Step 1: View Commit History**
```bash
# Find last commit where file existed
git log -- example.txt

# Or use --all to search all branches
git log --all --full-history -- example.txt
```

**Step 2: Identify Commit SHA**
```
commit abc123def456789...
Author: Developer Name
Date: Mon Jan 15 14:30:00 2024

    Added example feature

# File was present in this commit
```

**Step 3: Restore the Deleted File**
```bash
# Restore file from commit before deletion
# The ^ means "parent of that commit"
git checkout abc123def^ -- example.txt

# Alternative: specify exact commit where file existed
git checkout abc123def -- example.txt
```

**Step 4: Verify Changes**
```bash
# Check repository status
git status

# Should show:
# modified: example.txt
# or
# new file: example.txt
```

**Step 5: Stage and Commit**
```bash
# Stage restored file
git add example.txt

# Commit with clear message
git commit -m "Restored example.txt"

# Verify file is back
ls example.txt
```

## Advanced Recovery Techniques

### Recover Entire Branch
```bash
# Find branch in reflog
git reflog

# Output shows:
# abc123 HEAD@{0}: commit: latest change
# def456 HEAD@{1}: checkout: moving from deleted-branch
# ghi789 HEAD@{2}: commit: work on deleted-branch

# Recreate branch
git branch recovered-branch ghi789
```

### Recover from Destructive Rebase
```bash
# View reflog to find pre-rebase state
git reflog

# Find SHA before rebase (HEAD@{n})
# abc123 HEAD@{5}: rebase: before rebase operation

# Reset to pre-rebase state
git reset --hard abc123
```

### Recover Staged But Uncommitted Changes
```bash
# If you accidentally reset staged changes
git fsck --lost-found

# Git shows dangling blobs
# Use git show to view blob contents
git show <blob-sha>

# Restore if it's the right content
git show <blob-sha> > recovered-file.txt
```

## Safety Precautions

### Before Recovery Operations

```bash
# 1. Create backup branch
git branch backup-$(date +%Y%m%d)

# 2. Or stash current changes
git stash save "backup before recovery"

# 3. View what will change (--dry-run when available)
git merge --no-commit --no-ff <branch>

# 4. Check status before committing
git status
git diff --cached
```

### Recovery Checklist
- [ ] Identify exact commit or file to recover
- [ ] Backup current state (branch or stash)
- [ ] Use `git log` or `git reflog` to find SHA
- [ ] Test recovery in temporary branch first
- [ ] Verify recovered content before committing
- [ ] Document what was recovered and why

## Common Recovery Scenarios

| Situation | Command |
|-----------|---------|
| **Deleted file, not committed** | `git restore <file>` or `git checkout -- <file>` |
| **Deleted file, was committed** | `git checkout <commit>^ -- <file>` |
| **Deleted branch** | `git reflog` → `git branch <name> <SHA>` |
| **Wrong merge** | `git reflog` → `git reset --hard <SHA>` |
| **Lost commits after reset** | `git reflog` → `git cherry-pick <SHA>` |
| **Accidentally amended** | `git reflog` → `git reset HEAD@{1}` |

## Critical Notes
- 🎯 Git reflog is your safety net - keeps 90 days of commit history by default
- 💡 Always use `git stash` or create backup branch before recovery operations
- ⚠️ Be cautious with `git reset --hard` and history-rewriting commands
- 📊 `git fsck --lost-found` finds commits not reachable from any branch
- 🔄 Recovery operations should be tested in separate branch when possible
- ✨ Most "lost" data is actually still in Git - just need to find the SHA

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/6-recover-specific-data-git-commands)
