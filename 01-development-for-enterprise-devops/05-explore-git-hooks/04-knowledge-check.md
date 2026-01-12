# Module Assessment - Knowledge Check

## Purpose
Test your understanding of Git hooks for automation, security, and quality enforcement in development workflows.

## Assessment Format
- **Duration**: 5 minutes
- **Type**: Multiple-choice questions
- **Coverage**: Hook types, implementation patterns, security focus, Azure DevOps integration
- **Passing**: Understanding verification (not graded)

## Key Topics Covered

### Git Hook Fundamentals
**Assessment Areas**:
- Client-side vs. server-side hooks
- Hook execution timing and order
- Hook types: pre-commit, commit-msg, prepare-commit-msg, post-commit, pre-push, pre-receive
- When to use each hook type
- Bypassing hooks with `--no-verify`

### Security Implementation
**Assessment Areas**:
- Credential detection patterns (passwords, API keys, tokens)
- Private key detection
- Database URL scanning
- AWS Access Key patterns
- Vulnerability scanning in dependencies

### Code Quality Automation
**Assessment Areas**:
- Multi-language linting (JavaScript, Python, C#, Go)
- Formatting enforcement
- Type checking integration
- Test execution strategies
- Performance optimization

### Commit Message Standards
**Assessment Areas**:
- prepare-commit-msg automation
- commit-msg validation
- Work item linking requirements
- Conventional commit format
- Message length limits

### Azure DevOps Integration
**Assessment Areas**:
- Triggering Azure Pipelines from hooks
- Updating Azure Boards work items
- Branch naming conventions
- Azure CLI command usage

## Self-Assessment Checklist

Before taking the knowledge check, verify you can:

### Hook Implementation Basics
- [ ] **Create cross-platform hooks** - Use `#!/usr/bin/env bash` for compatibility
- [ ] **Install hooks in repository** - Copy to `.git/hooks/` and make executable
- [ ] **Test hooks locally** - Verify behavior before team deployment
- [ ] **Handle missing dependencies** - Check tool availability with `command -v`
- [ ] **Provide clear error messages** - Guide developers on remediation steps

### Security-Focused Hooks
- [ ] **Implement credential detection** - Scan for passwords, API keys, tokens using regex
- [ ] **Check for private keys** - Detect `-----BEGIN PRIVATE KEY-----` patterns
- [ ] **Scan database URLs** - Find connection strings with credentials
- [ ] **Detect cloud provider keys** - AWS (`AKIA`), Azure, GCP patterns
- [ ] **Validate dependencies** - Run `npm audit`, `pip-audit`, `dotnet list package --vulnerable`

### Code Quality Hooks
- [ ] **Configure linting** - ESLint, Flake8, golint per language
- [ ] **Enforce formatting** - Prettier, Black, gofmt automatically
- [ ] **Run type checking** - TypeScript, mypy for static analysis
- [ ] **Execute tests intelligently** - Only for changed code in pre-commit
- [ ] **Optimize performance** - Keep pre-commit hooks under 10 seconds

### Commit Message Automation
- [ ] **Auto-populate templates** - prepare-commit-msg extracts branch info
- [ ] **Validate format** - commit-msg checks length, structure, work item references
- [ ] **Extract work item IDs** - Parse from branch names like `feature/12345-description`
- [ ] **Enforce conventional commits** - `feat:`, `fix:`, `docs:`, etc. (optional)
- [ ] **Handle exceptions** - Allow override for urgent situations

### Azure DevOps Integration
- [ ] **Trigger pipelines** - `az pipelines build queue` from post-commit/pre-push
- [ ] **Update work items** - `az boards work-item update` with commit details
- [ ] **Validate branch names** - Enforce `feature/`, `bugfix/`, `hotfix/` conventions
- [ ] **Check policies** - `az repos policy evaluation list` before push
- [ ] **Integrate with service hooks** - Server-side automation for team enforcement

## Exam Preparation Tips

### Core Concepts to Master

**Hook Execution Order**:
```
git commit workflow:
1. prepare-commit-msg (auto-populate template)
2. commit-msg (validate format)
3. pre-commit (quality checks, security scans)
4. [commit created]
5. post-commit (notifications, metrics)

git push workflow:
1. pre-push (integration tests, build validation)
2. [push to remote]
3. pre-receive (server-side policy enforcement)
4. update (per-branch validation)
5. post-receive (CI/CD triggers, notifications)
```

**Hook Type Decision Matrix**:

| Requirement | Hook Type | Rationale |
|-------------|-----------|-----------|
| **Code formatting** | pre-commit | Fast feedback, can auto-fix |
| **Linting errors** | pre-commit | Fast (< 10s), early detection |
| **Unit tests (changed files)** | pre-commit | Quick validation of modifications |
| **Commit message format** | commit-msg | Validates after user input |
| **Integration tests** | pre-push | Slower, run before sharing |
| **Security scanning** | pre-commit + pre-push | Multi-layer defense |
| **Branch policies** | pre-receive (server) | Centrally enforced, cannot bypass |

**Credential Detection Patterns**:

| Pattern | Regex | Example Match |
|---------|-------|---------------|
| **Generic password** | `password\s*[=:]\s*['\"][^'\"]{8,}` | `password = "MySecret123"` |
| **API key** | `api[_-]?key\s*[=:]\s*['\"][^'\"]{20,}` | `api_key: "abcdef1234567890"` |
| **Private key** | `-----BEGIN\s+(RSA\s+)?PRIVATE\s+KEY-----` | SSH/TLS private keys |
| **AWS Access Key** | `AKIA[0-9A-Z]{16}` | `AKIAIOSFODNN7EXAMPLE` |
| **Database URL** | `mysql://.*:.*@` | `mysql://user:pass@host` |
| **Generic secret** | `secret\s*[=:]\s*['\"][^'\"]{16,}` | `secret: "verysecretvalue"` |

**Multi-Language Quality Check Template**:
```bash
for file in $(git diff --cached --name-only --diff-filter=ACM); do
    case "$file" in
        *.js|*.jsx|*.ts|*.tsx)
            npx eslint "$file" || exit 1
            ;;
        *.py)
            python3 -m flake8 "$file" || exit 1
            ;;
        *.cs)
            dotnet format --verify-no-changes --include "$file" || exit 1
            ;;
        *.go)
            gofmt -l "$file" | grep -q . && exit 1
            ;;
    esac
done
```

**Commit Message Validation Standards**:

| Element | Validation | Example | Enforcement Level |
|---------|-----------|---------|-------------------|
| **First line length** | ≤ 72 characters | `feat: add OAuth authentication` | ❌ Block |
| **Minimum length** | ≥ 10 characters | Short description required | ❌ Block |
| **Work item reference** | AB#\d+ or #\d+ | `AB#12345` | ⚠️ Warn |
| **Conventional format** | `type(scope): description` | `fix(auth): resolve token expiry` | ℹ️ Optional |
| **Body wrapping** | ≤ 72 characters | Wrapped paragraphs | ℹ️ Recommended |

**Azure DevOps Integration Commands**:

```bash
# Trigger validation pipeline
az pipelines build queue \
  --definition-name "PR-Validation" \
  --branch $(git branch --show-current)

# Update work item with commit
az boards work-item update \
  --id $work_item_id \
  --discussion "Commit $commit_sha: $commit_message"

# Check branch policies
az repos policy evaluation list \
  --project MyProject \
  --repository MyRepo

# Create pull request
az repos pr create \
  --source-branch $(git branch --show-current) \
  --target-branch main \
  --title "$PR_TITLE"
```

### Performance Optimization Guidelines

| Check Type | Target Duration | Optimization Strategy |
|------------|----------------|----------------------|
| **Code formatting** | < 1 second | Only check staged files |
| **Linting** | < 5 seconds | Parallel execution per file |
| **Unit tests (subset)** | < 10 seconds | Only tests for changed files |
| **Security scan** | < 5 seconds | Pattern matching (fast) |
| **Type checking** | < 3 seconds | Incremental compilation |
| **Integration tests** | < 30 seconds | Move to pre-push hook |

**Timeout Protection**:
```bash
# Prevent hooks from running too long
timeout 30s expensive_operation || {
    echo "⚠️  Hook operation timed out, skipping..."
    return 0  # Don't block commit
}
```

### Common Pitfalls and Solutions

**Pitfall #1**: Hooks not executable
```bash
# Problem: Hook doesn't run
# Solution: Make executable
chmod +x .git/hooks/pre-commit
```

**Pitfall #2**: Platform-specific paths
```bash
# ❌ Bad: Hardcoded path
#!/bin/bash

# ✅ Good: Environment-based
#!/usr/bin/env bash
```

**Pitfall #3**: Missing tool dependencies
```bash
# ❌ Bad: Assumes tool exists
eslint "$file"

# ✅ Good: Check availability
if command -v npx >/dev/null 2>&1; then
    npx eslint "$file"
else
    echo "⚠️  ESLint not found, skipping lint check"
fi
```

**Pitfall #4**: Blocking for non-critical issues
```bash
# ❌ Bad: Block for warnings
npm audit || exit 1

# ✅ Good: Only block for high-severity
npm audit --audit-level=high || exit 1
```

**Pitfall #5**: No progress feedback
```bash
# ❌ Bad: Silent operation
npm test

# ✅ Good: Show progress
echo "Running tests (this may take a moment)..."
npm test
```

## Study Resources

**Official Documentation**:
- [Git Hooks Documentation](https://git-scm.com/docs/githooks) - Complete reference for all hook types
- [Azure DevOps Service Hooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/events) - Server-side automation
- [Azure Repos Branch Policies](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies) - Policy enforcement strategies

**Example Hook Repository**:
Create `.git/hooks/pre-commit`:
```bash
#!/usr/bin/env bash
# Example enterprise pre-commit hook

set -e  # Exit on error

echo "🔍 Running pre-commit checks..."

# 1. Security: Check for credentials
echo "  • Checking for credentials..."
git diff --cached --name-only | xargs grep -l -E "(password|api[_-]?key|secret)" && {
    echo "❌ Potential credentials detected"
    exit 1
} || true

# 2. Quality: Lint staged files
echo "  • Linting code..."
npx lint-staged

# 3. Tests: Run tests for changed files
echo "  • Running tests..."
changed_files=$(git diff --cached --name-only | grep -E '\.(js|ts)$' || true)
if [ ! -z "$changed_files" ]; then
    npm test -- --findRelatedTests $changed_files --passWithNoTests
fi

echo "✅ All pre-commit checks passed"
```

**Hook Installation Script**:
```bash
#!/usr/bin/env bash
# install-hooks.sh - Deploy hooks to team

# Copy hooks to .git/hooks
cp hooks/pre-commit .git/hooks/pre-commit
cp hooks/commit-msg .git/hooks/commit-msg

# Make executable
chmod +x .git/hooks/pre-commit
chmod +x .git/hooks/commit-msg

echo "✅ Git hooks installed successfully"
```

## Practice Scenarios

**Scenario 1**: A developer commits code with an AWS Access Key. Which hook should detect this?
**Answer**: `pre-commit` hook with pattern `AKIA[0-9A-Z]{16}`

**Scenario 2**: Your team requires all commits to reference Azure Boards work items. Which hook enforces this?
**Answer**: `commit-msg` hook checking for `AB#\d+` or `#\d+` pattern

**Scenario 3**: Integration tests take 2 minutes to run. Where should they execute?
**Answer**: `pre-push` hook (not pre-commit, too slow at 2 minutes)

**Scenario 4**: You need to automatically add branch name to commit messages. Which hook?
**Answer**: `prepare-commit-msg` hook extracting from `git branch --show-current`

**Scenario 5**: Prevent any push to main branch that doesn't pass build. Client-side or server-side?
**Answer**: Server-side `pre-receive` hook (client-side can be bypassed with `--no-verify`)

## Critical Notes
- 🎯 **Hook types**: pre-commit (quality), commit-msg (format), pre-push (testing), pre-receive (policy)
- 💡 **Security focus**: Credential detection with comprehensive regex patterns
- ⚠️ **Performance targets**: pre-commit < 10s, commit-msg < 2s, pre-push < 30s
- 📊 **Multi-language support**: JavaScript, Python, C#, Go with appropriate linting tools
- 🔄 **Azure DevOps integration**: Pipeline triggers, work item updates, policy validation
- ✨ **Best practices**: Cross-platform compatibility, clear errors, graceful degradation

[Take Knowledge Check](https://learn.microsoft.com/en-us/training/modules/explore-git-hooks/4-knowledge-check)
