# Describe Inner Source with Forks

## Key Concepts
- **Fork Use Cases**: Contributing changes, experimenting with code, or using code as starting point
- **Azure DevOps Forks**: Built-in fork support in Azure Repos Git repositories
- **Fork Independence**: Work without affecting original repository
- **Upstream Contribution**: Pull requests merge changes back to original project

## Common Reasons to Fork

| Reason | Scenario |
|--------|----------|
| **Make a Change** | Found a bug or want to add feature to someone else's project |
| **Experiment Safely** | Test ideas without affecting original repository |
| **Use as Starting Point** | Build on existing work for your own project |
| **Learning** | Explore and learn from code in other teams' projects |
| **Inner Open Source** | Foster culture of contribution across all internal projects |

## Understanding Fork Structure

### What's Included
- All contents of upstream (original) repository
- All branches or just default branch (your choice)
- Complete commit history

### What's NOT Included
- Permissions from original repository
- Branch policies
- Build definitions
- Work items

**Key Point**: Fork acts as if someone cloned original, then pushed to new empty repository

### Post-Fork Behavior
- Newly created files, folders, and branches NOT shared automatically
- Sharing requires pull request to carry changes
- Pull requests supported in both directions (fork↔upstream)
- Most common: fork → upstream

## Step-by-Step Fork Workflow

### Step 1: Create the Fork
1. Choose **Fork** button
2. Select project for fork location
3. Give fork a name
4. Choose to include all branches or default only
5. Click **Fork** to create

### Step 2: Clone Your Fork
```bash
# Clone your fork (becomes "origin")
git clone {your_fork_url}

# Add upstream remote for syncing
git remote add upstream {upstream_url}
```

### Step 3: Make Your Changes
```bash
# Create topic branch (recommended)
git checkout -b feature/my-feature

# Make changes and commit
git add .
git commit -m "Implement new feature"

# Push to your fork
git push origin feature/my-feature
```

**Why Topic Branches?**
- Maintain multiple independent workstreams
- Reduces confusion when syncing fork with upstream
- Better organization even though it's your copy

### Step 4: Create Pull Request
```bash
# In Azure DevOps UI:
# Open PR from fork branch to upstream repository
# Upstream policies and reviewers apply
# Complete PR after all policies satisfied
```

### Step 5: Sync Your Fork
```bash
# Fetch latest from upstream
git fetch upstream main

# Rebase on upstream main
git rebase upstream/main

# Push updated fork
git push origin main
```

## Benefits of Fork Workflow

| Benefit | Description |
|---------|-------------|
| **Independence** | Work on changes without affecting original repository |
| **Collaboration** | Contribute to projects you don't normally work on |
| **Quality Control** | All changes go through same review process before merging |
| **Learning** | Explore and learn from code in other teams' projects |
| **Safety** | Original team retains control through PR review |

## Pull Request Flow
- Open PR from fork to upstream
- Upstream repository applies all policies:
  - Required reviewers
  - Build validations
  - Status checks
  - Comment resolution requirements
- Changes merged only after all policies satisfied

## Critical Notes
- 🎯 Forks enable contribution without write access to original repository
- 💡 Software teams encouraged to contribute to all internal projects, not just their own
- ⚠️ Fork doesn't copy permissions, policies, or build definitions
- 📊 Pull requests can flow in either direction: fork→upstream or upstream→fork
- 🔄 Regular syncing prevents your fork from diverging too far from upstream
- ✨ Forks foster culture of inner open source within your organization

[Learn More](https://learn.microsoft.com/en-us/training/modules/plan-fostering-inner-source/4-describe-forks)
