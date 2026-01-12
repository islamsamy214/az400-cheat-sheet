# Explore Git Branch Model for Continuous Delivery

## Strategic Balance: Speed vs. Quality

An effective branching model must strike the right balance:
- **Minimize process overhead** to accelerate time-to-market
- **Maintain quality gates** to prevent defects from reaching production
- **Enable parallel development** for team scalability
- **Support rapid hotfix deployment** for critical issues

**Core Principle**: Always-Ready Main Branch - maintain a consistently deployable main branch using feature branches and pull requests.

## Enterprise Branching Strategy Framework

### Main Branch Stability
- **Single source of truth**: The main branch is the exclusive path for production releases
- **Production readiness**: Main branch must always be in a deployable state
- **Branch protection**: Implement comprehensive branch policies to prevent direct commits
- **Gated changes**: All modifications flow through pull requests exclusively
- **Release tracking**: Tag all production releases with semantic Git tags

### Feature Branch Discipline
- **Isolated development**: Create dedicated branches for all features and bug fixes
- **Feature flag integration**: Manage long-running features with feature toggles to reduce branch lifetime
- **Strategic naming**: Use descriptive naming conventions that reflect business value
- **Pull request workflow**: Merge to main exclusively through reviewed pull requests

### Release Branch Strategy
- **Dedicated preparation**: Create release branches from stable feature integration points
- **Quality assurance**: Conduct thorough testing and stabilization on release branches
- **Production hardening**: Apply final bug fixes and performance optimizations
- **Milestone tracking**: Tag significant release milestones for traceability

### Naming Conventions for Scale

```bash
# Bug fixes
bugfix/[ticket-id]-description
bugfix/AUTH-123-login-timeout

# Feature development
features/[area]/[feature-name]
features/authentication/oauth-integration
features/api/rate-limiting

# Hotfixes
hotfix/[severity]-description
hotfix/critical-payment-gateway

# Personal branches
users/[username]/[description]
users/john-doe/experimental-caching
```

### Pull Request Management
- **Mandatory code review**: No exceptions for direct merges to main
- **Automated validation**: Integrate CI/CD pipelines for quality gates
- **Performance metrics**: Track and optimize pull request completion time
- **Knowledge sharing**: Use reviews for team learning and standards enforcement

## Hands-On Implementation: Enterprise Git Workflow

### Step 1: Feature Branch Creation and Development

```bash
# Update main branch
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/myFeature-1

# Verify branch context
git branch
# Output: * feature/myFeature-1
#         main
```

### Step 2: Feature Implementation and Version Control

```bash
# Edit source files
code Program.cs  # Or your preferred editor

# Check status
git status
# Output: On branch feature/myFeature-1
#         Changes not staged for commit:
#         modified: Program.cs

# Stage and commit
git add .
git commit -m "feat: implement myFeature-1 with enhanced error handling"

# Push to remote
git push -u origin feature/myFeature-1
```

### Step 3: Azure DevOps CLI Configuration and Pull Request Creation

```cmd
# Configure Azure DevOps CLI (replace organization and project name)
az devops configure --defaults organization=https://dev.azure.com/organization project="project name"

# Create pull request
az repos pr create --title "Review Feature-1 before merging to main" \
  --work-items 38 39 \
  --description "#Merge feature-1 to main" \
  --source-branch feature/myFeature-1 \
  --target-branch main \
  --repository myWebApp \
  --open
```

**Useful switches**:
- `--open`: Open PR in web browser after creation
- `--delete-source-branch`: Delete branch after PR completion
- `--auto-complete`: Complete automatically when all policies pass

**After team review and approval**, tag the main branch:
```bash
git tag -a v1.0 -m "Release version 1.0"
git push origin v1.0
```

### Step 4: Parallel Feature Development

```cmd
# Create Feature 2 branch on remote from main
git push origin main:refs/heads/feature/myFeature-2

# Checkout locally
git checkout feature/myFeature-2
# Output: Switched to a new branch 'feature/myFeature-2'
#         Branch feature/myFeature-2 set up to track remote branch feature/myFeature-2 from origin

# Modify Program.cs (same line as feature-1 for conflict demonstration)
# Commit and push changes
git add .
git commit -m "feat: implement myFeature-2 with alternative approach"
git push
```

### Step 5: Feature 2 Pull Request and Hotfix Scenario

```cmd
# Create PR for Feature 2
az repos pr create --title "Review Feature-2 before merging to main" \
  --work-items 40 42 \
  --description "#Merge feature-2 to main" \
  --source-branch feature/myFeature-2 \
  --target-branch main \
  --repository myWebApp \
  --open
```

**Critical bug reported in production!** Create hotfix branch from release tag:
```cmd
git checkout -b fof/bug-1 release_feature1
# Output: Switched to a new branch, 'fof/bug-1'
```

### Step 6: Critical Hotfix Implementation

```bash
# Fix the bug in Program.cs
# Stage and commit changes
git add .
git commit -m "fix: critical bug in production - apply emergency patch"

# Push to remote
git push -u origin fof/bug-1
```

### Step 7: Hotfix Integration and Conflict Resolution

```cmd
# Tag hotfix immediately after production deployment
git tag -a release_bug-1 -m "Emergency hotfix release"
git push origin release_bug-1

# Create PR to merge hotfix back to main
az repos pr create --title "Review Bug-1 before merging to main" \
  --work-items 100 \
  --description "#Merge Bug-1 to main" \
  --source-branch fof/Bug-1 \
  --target-branch main \
  --repository myWebApp \
  --open
```

**Feature-2 branch status**: 1 change ahead, 2 changes behind main (conflict detected!)

## Resolve Merge Conflicts

### Local Resolution Approach

```cmd
# Update feature branch with latest changes from main
git checkout feature/myFeature-2
git fetch origin
git merge origin/main
# Conflicts will be reported

# Resolve conflicts in your preferred editor
# Then complete the merge
git add .
git commit -m "Resolve merge conflicts between feature-2 and main"
git push origin feature/myFeature-2
```

### Create Release Branch from Main

```cmd
# Create release branch from main
git checkout -b release/v1.1 main

# Tag release milestones
git tag -a v1.1 -m "Release version 1.1"
git push origin v1.1
```

## How It Works

**Key Takeaways**:
- **Git branching model** provides flexibility for parallel feature development
- **Pull request workflow** enables systematic code review using branch policies
- **Git tags** record milestones (version numbers, releases) and enable branch creation from tags
- **Hotfix branches** can be created from previous release tags for critical bug fixes
- **Branches view** in web portal identifies branches ahead/behind main and detects merge conflicts
- **Lean branching model** enables short-lived branches and rapid quality pushes to production

## Critical Notes
- 🎯 **Core principle**: Always-ready main branch for continuous delivery
- 💡 **Naming conventions**: bugfix/, features/, hotfix/, users/ prefixes with descriptive names
- ⚠️ **Pull request mandatory**: No direct commits to main branch allowed
- 📊 **Azure DevOps CLI**: Powerful automation for PR creation and management
- 🔄 **Merge conflict resolution**: Update feature branch from main, resolve, commit, push
- ✨ **Git tags**: Mark releases and milestones for traceability and hotfix branching

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-branches-workflows/4-explore-git-branch-model-for-continuous-delivery)
