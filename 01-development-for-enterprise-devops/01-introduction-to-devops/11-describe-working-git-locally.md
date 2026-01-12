# Describe Working with Git Locally

## Prerequisites
- .NET 8 SDK or later
- Visual Studio Code
- C# VS Code extension
- Git (Git for Windows if on Windows)

## Recommended VS Code Extensions
- **Git Lens**: Code history visualization with Git blame annotations, code lens, file/branch history exploration
- **Git History**: Visualize and interact with Git log, file history, compare branches/commits

## Essential Git Commands

### Initial Setup
```bash
# Create working folder
mkdir myWebApp
cd myWebApp

# Initialize Git repository
git init

# Configure user identity
git config --global user.name "John Doe"
git config --global user.email "john.doe@contoso.com"

# Configure proxy (if behind enterprise proxy)
git config --global http.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
```

### Basic Workflow
```bash
# Check repository status
git status

# Stage all changes
git add .

# Commit changes
git commit -m "commit message"

# View commit history
git log -v         # Verbose history
git log -p         # History with file changes
```

### Branching Workflow
```bash
# List branches
git branch --list

# Create new branch
git branch feature-branch-name

# Switch to branch
git checkout feature-branch-name

# Merge branch into main
git checkout main
git merge feature-branch-name

# Delete branch
git branch --delete feature-branch-name
```

### Undoing Changes
```bash
# Hard reset to previous commit (DESTRUCTIVE)
git reset --hard <commit-hash>
```

## Two-Step Commit Process
1. **Stage** changes: `git add .` (changes staged but not committed)
2. **Commit** staged changes: `git commit -m "message"` (promotes staged changes to repository)

## .gitignore Best Practices
- Exclude build artifacts, IDE folders (.vscode/), sensitive files
- Use folder patterns like `.vscode/` instead of specific files
- Add patterns before first commit to avoid tracking unwanted files

## VS Code Git Integration
- **Git panel**: View changes, stage, and commit
- **GitLens decorations**: Shows commit history inline with code
- **Commit from UI**: Add message and click checkmark (stages + commits in one action)

## Critical Notes
- 🎯 **Git + Continuous Delivery**: Perfect combination for quality codebases
- ⚠️ `git reset --hard` is **destructive**—use carefully
- 💡 Visual Studio Code provides excellent Git native support
- 📊 Staging area allows reviewing changes before committing
- 🔥 Git enables automation of quality checks before committing

## Workflow Example
1. Initialize repository: `git init`
2. Create feature branch: `git branch feature-x`
3. Checkout branch: `git checkout feature-x`
4. Make changes
5. Stage changes: `git add .`
6. Commit: `git commit -m "description"`
7. Switch to main: `git checkout main`
8. Merge feature: `git merge feature-x`
9. Delete feature branch: `git branch --delete feature-x`

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-devops/11-describe-working-git-locally)
