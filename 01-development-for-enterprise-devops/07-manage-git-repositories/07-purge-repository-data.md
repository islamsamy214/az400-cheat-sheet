# Clean Up Repository Data

## Key Concepts
- **Repository Cleanup**: Remove unwanted history, large files, or sensitive data
- **History Rewriting**: Change Git history to remove data permanently
- **git filter-repo**: Modern tool for rewriting repository history
- **BFG Repo-Cleaner**: User-friendly tool for removing sensitive content

## Common Cleanup Situations

| Scenario | Reason |
|----------|--------|
| **Make repository smaller** | Remove extensive history to reduce size |
| **Remove large file** | Accidentally committed large binary file |
| **Remove sensitive data** | Passwords, keys, or credentials committed |
| **Clean up old branches** | Remove merged or abandoned branches |
| **Purge vendor dependencies** | Accidentally tracked node_modules or similar |

## git filter-repo Tool

### Overview
Modern, powerful tool for rewriting Git history:
- **Core Library**: Foundation for creating history rewriting tools
- **Specialized Needs**: Quickly create custom history rewriting tools
- **Faster Than filter-branch**: More performant than older Git commands

### Installation
```bash
# Install via pip
pip install git-filter-repo

# Or via package manager
# Ubuntu/Debian
apt-get install git-filter-repo

# macOS
brew install git-filter-repo
```

### Common Operations

**Remove Specific File from History**:
```bash
# Remove file from entire history
git filter-repo --path secrets.txt --invert-paths

# Remove directory from history
git filter-repo --path old-vendor/ --invert-paths
```

**Remove Large Files**:
```bash
# Find large files first
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  awk '/^blob/ {print substr($0,6)}' | \
  sort --numeric-sort --key=2 --reverse | \
  head -20

# Remove them
git filter-repo --strip-blobs-bigger-than 10M
```

**Replace Text in All Files**:
```bash
# Create replacements file
echo "PASSWORD==>***REMOVED***" > replacements.txt

# Replace throughout history
git filter-repo --replace-text replacements.txt
```

### More Information
See [git-filter-repo](https://github.com/newren/git-filter-repo) repository for details

## BFG Repo-Cleaner

### Overview
Open-source tool designed to be easier than `git filter-branch`:
- **User-Friendly**: Simpler syntax and faster execution
- **Common Operations**: Optimized for most cleanup scenarios
- **Safety**: Creates backup refs before modification

### Installation
```bash
# Download jar file
wget https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar

# Create alias
alias bfg='java -jar /path/to/bfg-1.14.0.jar'
```

### Delete Specific Files

```bash
# Delete single file from history
bfg --delete-files file_I_should_not_have_committed

# Delete files matching pattern
bfg --delete-files '*.{mp4,zip,tar.gz}'

# Delete folders
bfg --delete-folders vendor
```

### Replace Sensitive Text

```bash
# Create passwords.txt file with sensitive strings
# File content:
# MySecretPassword
# api_key_12345

# Replace all occurrences throughout history
bfg --replace-text passwords.txt
```

**How It Works**:
- Finds all places sensitive text appears
- Replaces with `***REMOVED***` (default)
- Works across entire repository history

### Remove Large Files

```bash
# Remove all files larger than 100MB
bfg --strip-blobs-bigger-than 100M

# Remove specific large files
bfg --delete-files large-dataset.csv
```

### Complete Cleanup Process

```bash
# 1. Clone fresh copy (mirror clone for full cleanup)
git clone --mirror git://example.com/repo.git

# 2. Run BFG cleanup
cd repo.git
bfg --delete-files sensitive-file.txt

# 3. Expire reflog and garbage collect
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 4. Push changes (CAUTION: rewrites history)
git push --force
```

## Safety Considerations

### Before Cleanup Operations

**⚠️ WARNING**: History rewriting is destructive. Follow these steps:

1. **Notify Team**: Coordinate with all team members
2. **Backup Repository**: Create complete backup
3. **Test First**: Run on copy, verify results
4. **Coordinate Timing**: Choose low-activity period
5. **Document Changes**: Record what was removed and why

```bash
# Create backup
git clone --mirror original-repo.git backup-repo.git

# Or create bundle
git bundle create repo-backup.bundle --all
```

### After Cleanup

**Everyone must re-clone**:
```bash
# Old clones are incompatible with rewritten history
# Team members must:
git clone <new-repo-url>

# DO NOT try to pull into old clone
# Will cause merge conflicts and confusion
```

## Comparison: filter-repo vs BFG

| Feature | git filter-repo | BFG Repo-Cleaner |
|---------|-----------------|------------------|
| **Speed** | Fast | Very Fast |
| **Ease of Use** | Moderate | Easy |
| **Flexibility** | Very High | Moderate |
| **Large Files** | ✅ Yes | ✅ Yes (optimized) |
| **Text Replacement** | ✅ Yes | ✅ Yes |
| **Complex Operations** | ✅ Excellent | ❌ Limited |
| **Active Development** | ✅ Yes | ⚠️ Maintenance mode |

## Complete Removal Process

To truly remove data from Git:

```bash
# 1. Rewrite history (filter-repo or BFG)
git filter-repo --path secrets.txt --invert-paths

# 2. Expire reflog references
git reflog expire --expire=now --all

# 3. Garbage collect with pruning
git gc --prune=now --aggressive

# 4. Verify data is gone
git log --all --oneline --source --remotes -- secrets.txt
# Should return nothing

# 5. Force push to remote
git push origin --force --all
git push origin --force --tags
```

## Additional Resources

- [Quickly rewrite git repository history](https://github.com/newren/git-filter-repo/)
- [Removing files from Git Large File Storage](https://docs.github.com/repositories/working-with-files/managing-large-files/removing-files-from-git-large-file-storage)
- [Removing sensitive data from a repository](https://docs.github.com/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
- [BFG Repo Cleaner](https://rtyley.github.io/bfg-repo-cleaner)

## Critical Notes
- 🎯 History rewriting is destructive - always create backups first
- 💡 BFG Repo-Cleaner is faster and easier for common cleanup tasks
- ⚠️ All team members must re-clone after history rewrite
- 📊 Removing sensitive data requires rewriting history AND force-pushing
- 🔄 git gc --aggressive reclaims disk space after cleanup
- ✨ Test cleanup operations on repository copy before applying to production

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/7-purge-repository-data)
