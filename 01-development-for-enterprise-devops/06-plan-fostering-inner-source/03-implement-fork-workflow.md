# Implement the Fork Workflow

## Key Concepts
- **Fork**: Complete copy of a repository including all files, commits, and (optionally) branches
- **Upstream Repository**: The original repository you forked from
- **Origin Remote**: Your fork (where you push changes)
- **Fork Isolation**: Changes in fork don't affect upstream until PR is merged

## What's in a Fork?

| Included | Not Included |
|----------|--------------|
| All repository contents | Permissions |
| All branches (or just default) | Branch policies |
| Complete commit history | Build pipelines |
| Files and folders | Work items |

**Important**: After fork creation, new files/folders/branches aren't shared unless carried by pull request

## Choosing Between Branches and Forks

| Scenario | Recommendation | Reason |
|----------|----------------|--------|
| **Small Teams (2-5 devs)** | Branches | Single repository with protected main branch |
| **Larger Teams** | Forks | Better isolation and scalability |
| **Casual Contributors** | Forks | Open-source style collaboration |
| **Core Team Only** | Branches | Direct commit rights, simpler workflow |
| **Outside Contributors** | Forks | Non-core team members work from fork |
| **Review Before Integration** | Forks | Complete isolation until changes reviewed |

## The Forking Workflow

### Step 1: Create the Fork
```bash
# In Azure DevOps UI:
# 1. Navigate to repository and choose "Fork"
# 2. Specify name and target project
# 3. Choose to fork all branches or default branch only
# 4. Choose ellipsis → "Fork" to create
```

**Requirement**: Must have **Create Repository** permission in target project

### Step 2: Clone Your Fork Locally
```bash
# Clone your fork
git clone {your_fork_url}

# Add upstream repository as remote
git remote add upstream {upstream_url}

# Verify remotes
git remote -v
```

### Step 3: Make and Push Changes
```bash
# Create topic branch (recommended)
git checkout -b feature/my-changes

# Make changes and commit
git add .
git commit -m "Add new feature"

# Push to your fork (origin)
git push origin feature/my-changes
```

**Best Practice**: Use topic branches even in your fork for multiple independent workstreams

### Step 4: Create and Complete Pull Request
```bash
# In Azure DevOps UI:
# Open PR from your fork branch to upstream main
# All upstream policies, reviewers, and builds apply
# Complete PR once all policies satisfied
```

**Important**: Anyone with Read permission can open PR to upstream; build pipelines run against fork code

### Step 5: Sync Your Fork to Latest
```bash
# Fetch latest from upstream
git fetch upstream main

# Rebase your main on upstream
git rebase upstream/main

# Push updated main to your fork
git push origin main
```

## Sharing Code Between Forks

| Direction | Use Case |
|-----------|----------|
| **Fork → Upstream** | Most common: propose changes to original project |
| **Upstream → Fork** | Less common: sync updates from original project |

**Note**: Destination repository's permissions, policies, builds, and work items apply to pull request

## Benefits of Fork Workflow

| Benefit | Description |
|---------|-------------|
| **Safety** | Changes isolated until reviewed |
| **Collaboration** | Multiple people work on different features simultaneously |
| **Quality** | All changes go through same review process |
| **Flexibility** | Contributors don't need write access to main repository |

## Critical Notes
- 🎯 Forks enable contribution without write access to upstream repository
- 💡 Always work in topic branches even within your fork for better organization
- ⚠️ None of the permissions, policies, or pipelines are copied to fork
- 📊 Pull requests can flow in both directions (fork↔upstream)
- 🔄 Regularly sync your fork to avoid merge conflicts with upstream
- ✨ Fork workflow provides complete isolation until you're ready to integrate

[Learn More](https://learn.microsoft.com/en-us/training/modules/plan-fostering-inner-source/3-implement-fork-workflow)
