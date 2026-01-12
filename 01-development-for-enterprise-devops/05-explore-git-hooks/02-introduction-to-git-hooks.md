# Introduction to Git Hooks

## Why Automation Matters

Modern software development needs automation that works precisely at every stage. Git hooks provide the foundation for this automation. They let teams implement quality checks, security validations, and rule compliance that **run automatically without slowing down developers**.

### Core Automation Benefits

**Check Problems Early**:
Git hooks help teams use "shift-left" strategies that catch problems at the earliest possible point. This reduces the cost of fixing issues and improves overall software quality.

**Cost of Defects by Stage**:
| Detection Stage | Relative Cost | Time to Fix |
|----------------|---------------|-------------|
| **During commit** (hooks) | 1x | Minutes |
| **During code review** | 5x | Hours |
| **In CI/CD pipeline** | 10x | Hours to days |
| **In staging** | 50x | Days |
| **In production** | 100-1000x | Days to weeks |

**Security First Model**:
In business environments, Git hooks work as automatic security guards. They check every code change against security policies before allowing it to continue through the development process.

**Automatic Rule Checking**:
For companies that must follow strict regulations, Git hooks provide automatic compliance checking. This ensures every code change meets necessary standards without requiring manual review.

## Hooks on Developer Computers

Client-side hooks run on each developer's computer. They provide immediate feedback and prevent bad code from entering shared code repositories.

### Pre-Commit Hook Features

**Code Quality Checks**:
Automatically check code formatting, linting rules, and company coding standards before allowing commits.

**Example - JavaScript/TypeScript**:
```bash
#!/usr/bin/env bash
# Pre-commit hook for JavaScript/TypeScript projects

echo "Running code quality checks..."

# Get staged JavaScript/TypeScript files
staged_files=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(js|jsx|ts|tsx)$')

if [ -z "$staged_files" ]; then
    echo "No JavaScript/TypeScript files to check"
    exit 0
fi

# Run ESLint
echo "Linting with ESLint..."
npx eslint $staged_files
if [ $? -ne 0 ]; then
    echo "❌ ESLint failed. Please fix linting errors before committing"
    exit 1
fi

# Run Prettier
echo "Checking code formatting with Prettier..."
npx prettier --check $staged_files
if [ $? -ne 0 ]; then
    echo "❌ Code not formatted. Run 'npm run format' to fix"
    exit 1
fi

# Type checking for TypeScript
if echo "$staged_files" | grep -qE '\.(ts|tsx)$'; then
    echo "Type checking with TypeScript..."
    npx tsc --noEmit
    if [ $? -ne 0 ]; then
        echo "❌ TypeScript type errors detected"
        exit 1
    fi
fi

echo "✅ All quality checks passed"
exit 0
```

**Security Scanning**:
Run automatic security scans to find passwords, API keys, and vulnerable dependencies before they enter the code.

**Password and Secret Detection**:
```bash
#!/usr/bin/env bash
# Check for passwords and secrets in code changes

if git diff --cached --name-only | xargs grep -l -E "(password|secret|api[_-]?key|token|credential)" 2>/dev/null; then
    echo "🚨 Security Alert: Found potential passwords or secrets in your changes"
    echo "Please review and remove sensitive information before committing"
    exit 1
fi
```

**Checking for Vulnerable Dependencies**:
```bash
#!/usr/bin/env bash
# Check for security vulnerabilities in dependencies

if command -v npm &> /dev/null && [ -f package.json ]; then
    npm audit --audit-level=high
    if [ $? -ne 0 ]; then
        echo "🔒 Found security vulnerabilities in dependencies"
        echo "Please fix high-severity vulnerabilities before committing"
        exit 1
    fi
fi
```

**Test Running**:
Run specific test suites to make sure code changes don't break existing functionality.

**Smart Test Execution**:
```bash
#!/usr/bin/env bash
# Smart test running based on what changed

changed_files=$(git diff --cached --name-only)
if echo "$changed_files" | grep -q "src/"; then
    echo "Running unit tests for changed components..."
    npm test -- --findRelatedTests $changed_files
    if [ $? -ne 0 ]; then
        echo "❌ Tests failed. Please fix failing tests before committing"
        exit 1
    fi
fi
```

**Documentation Checks**:
Verify that code changes include proper documentation updates and maintain documentation standards.

**Documentation Validation**:
```bash
#!/usr/bin/env bash
# Check if code changes include documentation updates

code_changes=$(git diff --cached --name-only | grep -E '\.(js|ts|py|cs)$')
doc_changes=$(git diff --cached --name-only | grep -E '\.(md|rst|txt)$')

if [ ! -z "$code_changes" ] && [ -z "$doc_changes" ]; then
    echo "⚠️  Warning: Code changes without documentation updates"
    echo "Consider updating README.md or relevant documentation"
    echo "Continue anyway? (y/N)"
    read -r response
    if [[ ! "$response" =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi
```

### Commit Message Hook Automation

**prepare-commit-msg**:
Automatically create or modify commit messages to ensure consistency with company standards and include necessary information.

**Auto-populate Commit Template**:
```bash
#!/usr/bin/env bash
# Automatically populate commit message template

commit_msg_file="$1"
commit_source="$2"

# Skip if this is a merge or amend
if [ -n "$commit_source" ]; then
    exit 0
fi

# Get current branch name
current_branch=$(git branch --show-current)

# Extract work item ID from branch name
if [[ "$current_branch" =~ ^(feature|bugfix|hotfix)/([0-9]+) ]]; then
    work_item_id="${BASH_REMATCH[2]}"
    
    # Add work item prefix to commit message
    cat > "$commit_msg_file" << EOF
AB#$work_item_id: 

# Brief description of changes (50 chars or less)
#
# Detailed explanation of what and why (wrap at 72 chars)
#
# - What problem this solves
# - Why this approach was chosen
# - Any breaking changes or migration notes
EOF
fi
```

**commit-msg**:
Check commit message format, enforce naming rules, and ensure proper links to work items or issue tracking systems.

**Commit Message Validation**:
```bash
#!/usr/bin/env bash
# Comprehensive commit message validation

commit_msg_file="$1"
commit_message=$(cat "$commit_msg_file")

# Remove comment lines
clean_message=$(echo "$commit_message" | grep -v '^#' | sed '/^$/d')

# Check minimum length
if [ ${#clean_message} -lt 10 ]; then
    echo "❌ Commit message too short (minimum 10 characters)"
    exit 1
fi

# Check maximum length for first line
first_line=$(echo "$clean_message" | head -n1)
if [ ${#first_line} -gt 72 ]; then
    echo "❌ Commit message first line too long (maximum 72 characters)"
    echo "Current length: ${#first_line}"
    exit 1
fi

# Check for work item reference
if ! echo "$clean_message" | grep -qE "(AB#[0-9]+|#[0-9]+|closes #[0-9]+|fixes #[0-9]+)"; then
    echo "⚠️  Commit message should reference a work item (e.g., AB#1234 or #1234)"
    echo "Continue anyway? (y/N)"
    read -r response
    if [[ ! "$response" =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi

# Check for conventional commit format (optional)
if echo "$first_line" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: "; then
    echo "✅ Conventional commit format detected"
fi

echo "✅ Commit message validation passed"
exit 0
```

### Post-Commit Integration Features

**Notification Automation**:
Send automatic notifications to team members, project management systems, or collaboration platforms.

**Slack Notification Example**:
```bash
#!/usr/bin/env bash
# Send commit notification to Slack

commit_sha=$(git rev-parse --short HEAD)
commit_message=$(git log -1 --pretty=%B)
commit_author=$(git log -1 --pretty=%an)
branch_name=$(git branch --show-current)

# Send to Slack webhook
curl -X POST "$SLACK_WEBHOOK_URL" \
  -H 'Content-Type: application/json' \
  -d "{
    \"text\": \"New commit by $commit_author on $branch_name\",
    \"attachments\": [{
      \"text\": \"$commit_message\",
      \"color\": \"good\",
      \"fields\": [{
        \"title\": \"Commit\",
        \"value\": \"$commit_sha\",
        \"short\": true
      }]
    }]
  }"
```

**Documentation Creation**:
Automatically update project documentation, API references, or change logs based on commit content.

**Metrics Collection**:
Gather development metrics and analytics to support continuous improvement efforts.

**Development Metrics Example**:
```bash
#!/usr/bin/env bash
# Collect development metrics

commit_sha=$(git rev-parse HEAD)
files_changed=$(git diff-tree --no-commit-id --name-only -r $commit_sha | wc -l)
insertions=$(git show --stat $commit_sha | grep 'insertion' | awk '{print $4}')
deletions=$(git show --stat $commit_sha | grep 'deletion' | awk '{print $6}')

# Log to metrics database or file
echo "$commit_sha,$files_changed,$insertions,$deletions,$(date -u +%Y-%m-%dT%H:%M:%SZ)" >> .git/metrics.csv
```

## Advanced Hook Implementation Patterns

### Security-Focused Hook Strategies

**Comprehensive Secret Pattern Detection**:
```bash
#!/usr/bin/env bash
# Advanced credential detection with pattern recognition

check_credentials() {
    local staged_files=$(git diff --cached --name-only)
    local violations=""

    # Define comprehensive credential patterns
    local patterns=(
        "password\s*[=:]\s*['\"][^'\"]{8,}"
        "api[_-]?key\s*[=:]\s*['\"][^'\"]{20,}"
        "secret\s*[=:]\s*['\"][^'\"]{16,}"
        "token\s*[=:]\s*['\"][^'\"]{24,}"
        "-----BEGIN\s+(RSA\s+)?PRIVATE\s+KEY-----"
        "mysql://.*:.*@"
        "postgresql://.*:.*@"
        "mongodb://.*:.*@"
        "AKIA[0-9A-Z]{16}"  # AWS Access Key
        "sk-[a-zA-Z0-9]{48}"  # OpenAI API Key
    )

    for file in $staged_files; do
        if [ -f "$file" ]; then
            for pattern in "${patterns[@]}"; do
                if git show ":$file" | grep -qiE "$pattern"; then
                    violations+="🚨 Potential credential detected in $file\n"
                fi
            done
        fi
    done

    if [ ! -z "$violations" ]; then
        echo -e "Security Alert: Credential Detection\n"
        echo -e "$violations"
        echo -e "Please remove sensitive information before committing\n"
        return 1
    fi

    return 0
}

check_credentials || exit 1
```

### Quality Checking Hook Automation

**Complete Multi-Language Code Quality**:
```bash
#!/usr/bin/env bash
# Check code quality for multiple programming languages

for file in $(git diff --cached --name-only --diff-filter=ACM); do
    case "$file" in
        *.js|*.ts)
            npx eslint "$file" || exit 1
            ;;
        *.py)
            python -m flake8 "$file" || exit 1
            python -m mypy "$file" || exit 1
            ;;
        *.cs)
            dotnet format --verify-no-changes --include "$file" || exit 1
            ;;
        *.go)
            gofmt -l "$file" | grep -q . && exit 1
            golint "$file" || exit 1
            ;;
        *.java)
            java -jar checkstyle.jar -c checkstyle.xml "$file" || exit 1
            ;;
    esac
done

echo "✅ Code quality checks passed"
exit 0
```

## Integration with Development Tools

### Azure DevOps Integration

**Work Item Link Checking**:
```bash
#!/usr/bin/env bash
# Make sure commit messages include Azure DevOps work item references

commit_message=$(cat "$1")
if ! echo "$commit_message" | grep -qE "#[0-9]+|AB#[0-9]+"; then
    echo "❌ Commit message must reference a work item (e.g., #1234 or AB#1234)"
    exit 1
fi
```

**Branch Name Checking**:
```bash
#!/usr/bin/env bash
# Check branch naming rules align with Azure DevOps policies

current_branch=$(git branch --show-current)
if ! echo "$current_branch" | grep -qE "^(feature|bugfix|hotfix)/[a-z0-9-]+$"; then
    echo "❌ Branch name must follow convention:"
    echo "  - feature/description"
    echo "  - bugfix/description"
    echo "  - hotfix/description"
    exit 1
fi
```

### Pipeline Integration Automation

**Build Validation Triggers**:
```bash
#!/usr/bin/env bash
# Trigger Azure Pipelines validation builds for significant changes

if git diff --cached --name-only | grep -qE "\.(cs|js|ts|py)$"; then
    echo "Triggering validation build for code changes..."
    az pipelines build queue \
      --definition-name "PR-Validation" \
      --branch $(git branch --show-current)
fi
```

**Note**: You need to install the Azure DevOps CLI extension to use these commands.

## Hook Categories and Use Cases

### Quality Gate Automation

**Pre-Commit Quality Gates**:
- Code formatting and style checking
- Static analysis and linting
- Unit test running for changed code
- Documentation completeness checking
- Performance impact checking

**Pre-Push Validation**:
- Integration test running
- Security vulnerability scanning
- Dependency license compliance checking
- Build validation and artifact creation
- Deployment readiness checking

**Quality Gate Comparison**:

| Check Type | Pre-Commit | Pre-Push | CI/CD Pipeline |
|------------|-----------|----------|----------------|
| **Formatting** | ✅ Fast (< 10s) | ❌ Too late | ❌ Too late |
| **Linting** | ✅ Fast (< 30s) | ⚠️ Acceptable | ❌ Too late |
| **Unit Tests** | ✅ For changed code | ✅ Full suite | ✅ Full suite + coverage |
| **Integration Tests** | ❌ Too slow | ✅ Ideal location | ✅ Full suite |
| **Security Scan** | ✅ Secret detection | ✅ Dependency scan | ✅ Full SAST/DAST |
| **Build** | ❌ Too slow | ⚠️ If quick (<2 min) | ✅ Full build |

### Security and Compliance Automation

**Security Hook Implementation**:
- Password and secret detection
- Dependency vulnerability checking
- Code security pattern validation
- Compliance rule checking
- Audit trail creation

**Compliance Validation Hooks**:
- Regulatory requirement checking
- Code signing and verification
- Change approval validation
- Documentation requirement checking
- Audit log creation

**Compliance Matrix**:

| Industry | Requirement | Hook Implementation | Frequency |
|----------|-------------|---------------------|-----------|
| **Financial (SOX)** | Audit trail for all changes | commit-msg: require work item ID | Every commit |
| **Healthcare (HIPAA)** | PHI data handling validation | pre-commit: scan for PII patterns | Every commit |
| **Payment (PCI-DSS)** | Card data protection | pre-commit: detect card numbers | Every commit |
| **Government (FedRAMP)** | Security scanning | pre-push: vulnerability scan | Every push |

### Development Workflow Enhancement

**Developer Experience Optimization**:
- Automatic commit message creation
- Branch naming rule checking
- Work item linking automation
- Code review assignment automation
- Progress tracking and reporting

**Team Collaboration Enhancement**:
- Notification and communication automation
- Knowledge sharing help
- Metrics collection and reporting
- Process compliance monitoring
- Continuous improvement data gathering

**Developer Experience Impact**:

| Metric | Without Hooks | With Well-Designed Hooks | Improvement |
|--------|--------------|-------------------------|-------------|
| **Failed builds** | 30% of commits | 5% of commits | 83% reduction |
| **PR cycle time** | 24 hours | 14 hours | 42% faster |
| **Security incidents** | 8 per year | 0 per year | 100% reduction |
| **Onboarding time** | 2 weeks | 3 days | 85% faster |

## Critical Notes
- 🎯 **Automation foundation**: Git hooks provide essential automation at earliest possible point
- 💡 **Client-side hooks**: Pre-commit (quality), commit-msg (format), post-commit (notifications)
- ⚠️ **Security first**: Password detection, vulnerability scanning, security pattern validation
- 📊 **Quality gates**: Pre-commit for fast checks, pre-push for comprehensive validation
- 🔄 **Azure DevOps integration**: Work item linking, branch naming, pipeline triggers
- ✨ **Developer experience**: Fast feedback, immediate fixes, reduced failed builds

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-git-hooks/2-introduction-to)
