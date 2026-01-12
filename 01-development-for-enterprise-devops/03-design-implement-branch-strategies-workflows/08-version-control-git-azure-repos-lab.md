# Version Control with Git in Azure Repos - Lab

## Lab Overview

**Duration**: 60 minutes

**Purpose**: Hands-on experience with Git version control in Azure Repos, including repository management, branching, and pull request workflows.

## Learning Objectives

After completing this lab, you'll be able to:
- **Clone an existing repository** from Azure Repos to local environment
- **Save work with commits** using proper Git commands and conventions
- **Review commit history** to understand codebase evolution
- **Work with branches** using Git command line and Azure Repos web interface
- **Manage repositories** through Azure Repos portal

## Lab Environment Requirements

**Software Prerequisites**:
- Git installed and configured on your local machine
- Azure DevOps account with organization access
- Code editor (Visual Studio Code recommended)
- Web browser for Azure DevOps portal access

**Azure DevOps Setup**:
- Create or access existing Azure DevOps organization
- Create new project or use existing project
- Initialize Git repository in Azure Repos

## Lab Exercises

### Exercise 1: Configure Git Environment

**Task 1.1: Install and Configure Git**
```bash
# Verify Git installation
git --version

# Configure user information
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Configure default editor
git config --global core.editor "code --wait"

# View configuration
git config --list
```

**Task 1.2: Generate SSH Keys (Optional)**
```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"

# Display public key
cat ~/.ssh/id_rsa.pub

# Add to Azure DevOps: User Settings → SSH public keys
```

### Exercise 2: Clone and Initialize Repository

**Task 2.1: Clone Repository from Azure Repos**
```bash
# Navigate to your projects directory
cd ~/projects

# Clone repository (use HTTPS or SSH URL from Azure Repos)
git clone https://dev.azure.com/organization/project/_git/repository-name
cd repository-name

# Verify remote configuration
git remote -v
# Output:
# origin  https://dev.azure.com/organization/project/_git/repository-name (fetch)
# origin  https://dev.azure.com/organization/project/_git/repository-name (push)
```

**Task 2.2: Explore Repository Structure**
```bash
# List all files including hidden
ls -la

# Check repository status
git status
# Output: On branch main
#         Your branch is up to date with 'origin/main'.
#         nothing to commit, working tree clean

# View commit history
git log --oneline --graph --all
```

### Exercise 3: Save Work with Commits

**Task 3.1: Create and Modify Files**
```bash
# Create new file
echo "# Project Documentation" > README.md

# Add content to file
echo "This project demonstrates Git workflows." >> README.md

# Check status
git status
# Output: Untracked files:
#         README.md
```

**Task 3.2: Stage and Commit Changes**
```bash
# Stage specific file
git add README.md

# Check status after staging
git status
# Output: Changes to be committed:
#         new file: README.md

# Commit with message
git commit -m "docs: add project README with initial documentation"

# View commit details
git show HEAD
```

**Task 3.3: Push Changes to Remote**
```bash
# Push to origin/main
git push origin main

# Verify in Azure DevOps web portal
# Navigate to Repos → Files to see updated README.md
```

### Exercise 4: Work with Branches

**Task 4.1: Create and Switch Branches**
```bash
# Create new branch
git branch feature/add-authentication

# Switch to new branch
git checkout feature/add-authentication
# Or use shorthand: git checkout -b feature/add-authentication

# Verify current branch
git branch
# Output:
#   main
# * feature/add-authentication
```

**Task 4.2: Make Changes on Feature Branch**
```bash
# Create new file
echo "// Authentication module" > auth.js

# Stage and commit
git add auth.js
git commit -m "feat: add authentication module skeleton"

# Push feature branch to remote
git push -u origin feature/add-authentication
```

**Task 4.3: View Branch in Azure Repos**
```
1. Navigate to Azure DevOps portal
2. Go to Repos → Branches
3. Observe feature/add-authentication branch listed
4. Note: "1 commit ahead of main"
```

### Exercise 5: Review History and Manage Branches

**Task 5.1: Review Commit History**
```bash
# View detailed history
git log --all --decorate --oneline --graph

# View history for specific file
git log --follow -- README.md

# View changes in specific commit
git show <commit-hash>

# Compare branches
git diff main feature/add-authentication
```

**Task 5.2: Merge Feature Branch (via Pull Request)**
```
1. In Azure DevOps, navigate to Repos → Pull requests
2. Click "New pull request"
3. Set source: feature/add-authentication, target: main
4. Add title: "Add authentication module"
5. Add description with details
6. Assign reviewers
7. Click "Create"
8. After approval, click "Complete" to merge
9. Optional: Delete source branch after merge
```

**Task 5.3: Sync Local Repository**
```bash
# Switch back to main branch
git checkout main

# Fetch updates from remote
git fetch origin

# Merge remote changes
git pull origin main

# Verify feature is merged
git log --oneline --graph
```

### Exercise 6: Advanced Git Operations (Optional)

**Task 6.1: Stash Uncommitted Changes**
```bash
# Make changes without committing
echo "// Work in progress" >> auth.js

# Stash changes
git stash save "WIP: authentication improvements"

# View stash list
git stash list

# Apply stashed changes later
git stash apply stash@{0}
```

**Task 6.2: Cherry-Pick Specific Commits**
```bash
# Cherry-pick commit from another branch
git cherry-pick <commit-hash>

# Resolve conflicts if necessary
# Then continue
git cherry-pick --continue
```

**Task 6.3: Revert Commits**
```bash
# Revert specific commit (creates new commit)
git revert <commit-hash>

# Push revert to remote
git push origin main
```

## Lab Verification Checklist

- [ ] Git configured with your name and email
- [ ] Repository cloned from Azure Repos successfully
- [ ] Created and committed files with proper messages
- [ ] Pushed commits to remote repository
- [ ] Created feature branch locally and remotely
- [ ] Viewed commit history with git log
- [ ] Created pull request in Azure Repos
- [ ] Merged pull request to main branch
- [ ] Synced local main branch with remote

## Common Troubleshooting

**Authentication Issues**:
```bash
# Use Git Credential Manager
git credential-manager configure

# Or use personal access token (PAT)
# Generate PAT in Azure DevOps: User Settings → Personal access tokens
```

**Merge Conflicts**:
```bash
# View conflicted files
git status

# Edit files to resolve conflicts
# Then stage resolved files
git add <conflicted-file>

# Complete merge
git commit -m "Resolve merge conflicts"
```

## Critical Notes
- 🎯 **Lab duration**: 60 minutes of hands-on Git and Azure Repos practice
- 💡 **Key skills**: Clone, commit, branch, pull request, merge workflows
- ⚠️ **Configuration**: Set up user.name and user.email before committing
- 📊 **Branching**: Feature branches isolate work from main branch
- 🔄 **Pull requests**: Structured code review before merging to main
- ✨ **Azure Repos**: Web interface complements Git command line for visualization

[Launch Lab](https://learn.microsoft.com/en-us/training/modules/manage-git-branches-workflows/8-version-control-git-azure-repos)
