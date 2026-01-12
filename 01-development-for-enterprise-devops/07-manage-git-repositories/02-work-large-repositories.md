# Work with Large Repositories

## Key Concepts
- **Large Repository Challenges**: Performance issues when repositories exceed certain sizes
- **Shallow Clone**: Download only recent history to reduce local repository size
- **VFS for Git**: Virtual File System enables working with extremely large repositories
- **Scalar**: .NET tool that optimizes Git performance for very large repositories

## Why Repositories Become Large

| Cause | Impact |
|-------|--------|
| **Long History** | Many years of commits and changes accumulate |
| **Large Binary Files** | Images, videos, or other big files tracked in repository |
| **Multiple Projects** | Monorepo containing many applications/services |
| **Build Artifacts** | Accidentally committed compiled binaries or dependencies |

**Example**: Microsoft moved a repository with over 300 GB of data to Git

## Shallow Clone

Reduces download size by limiting history depth:

```bash
# Clone only recent history
git clone --depth [depth] [clone-url]

# Clone only 10 most recent commits
git clone --depth 10 https://github.com/example/repo.git

# Clone only one branch
git clone --single-branch --branch main https://github.com/example/repo.git
```

**Benefits**:
- Saves local disk space
- Faster clone and sync operations
- Good for CI/CD systems that don't need full history

## VFS for Git

Virtual File System for Git enables enterprise-scale repositories:

| Feature | Description |
|---------|-------------|
| **On-Demand Downloads** | Files downloaded only when needed |
| **Git LFS Integration** | Requires Git LFS client to work |
| **Transparent Operation** | Normal Git commands work the same way |
| **REST Protocol** | Uses simple 4-endpoint REST-like protocol |

**Use Case**: Repositories too large for full local clones

## Scalar

Microsoft's .NET Core application for optimizing very large repositories:

### Automated Git Optimizations

| Feature | Benefit |
|---------|---------|
| **Partial Clone** | Faster initial clone by deferring object downloads |
| **Background Prefetch** | Downloads Git objects hourly, making fetch faster |
| **Sparse-Checkout** | Only includes files you need in working directory |
| **File System Monitor** | Tracks changes instead of scanning entire tree |
| **Commit-Graph** | Speeds up commit walks and reachability |
| **Multi-Pack-Index** | Fast object lookups across many pack files |
| **Incremental Repack** | Optimizes storage without disrupting work |

### Clone with Scalar
```bash
# Clone repository with Scalar optimizations
scalar clone <url>

# Repository automatically configured with:
# - Partial clone
# - Background maintenance
# - Sparse checkout
# - File system watcher
```

### GVFS Protocol Support
If Azure Repos hosts your repository, Scalar can use the GVFS protocol for even better performance.

**Microsoft Usage**: Used for Windows and Office repositories

## Decision Matrix

| Repository Size | Recommended Solution |
|-----------------|---------------------|
| **< 1 GB** | Standard Git clone |
| **1-10 GB** | Shallow clone or Git LFS |
| **10-50 GB** | Git LFS + Partial clone |
| **> 50 GB** | Scalar with VFS for Git |

## Critical Notes
- 🎯 Scalar automatically configures multiple Git optimizations for large repositories
- 💡 VFS for Git downloads files on-demand, dramatically reducing initial clone time
- ⚠️ Shallow clones don't include full history - may limit some Git operations
- 📊 Background prefetch keeps your repository up-to-date automatically
- 🔄 Scalar features updated with each Git version release
- ✨ Microsoft uses these techniques for repositories containing millions of files

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/2-work-large-repositories)
