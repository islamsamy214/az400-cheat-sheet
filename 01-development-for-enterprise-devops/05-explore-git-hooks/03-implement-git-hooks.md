# Implement Git Hooks

## Modern Git Hooks Setup

Git hooks implementation requires careful thinking about cross-platform compatibility, maintainability, and team deployment strategies. Modern approaches focus on **version-controlled hook management** and automatic distribution rather than manual setup.

### Cross-Platform Hook Development

Modern development environments need Git hooks that work the same way across Windows, macOS, and Linux development environments.

#### Universal Implementation

**Cross-Platform Compatible Shebang**:
```bash
#!/usr/bin/env bash
# Cross-platform compatible shebang that automatically finds bash interpreter
# Works consistently across Windows Git Bash, macOS, and Linux environments
```

This approach removes the platform-specific path problems that cause issues with traditional implementations and ensures consistent behavior across different development environments.

#### Environment Detection Strategy

```bash
#!/usr/bin/env bash
# Smart environment detection for platform-specific optimizations

detect_environment() {
    if [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "win32" ]]; then
        PLATFORM="windows"
        PYTHON_CMD="python"
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        PLATFORM="macos"
        PYTHON_CMD="python3"
    else
        PLATFORM="linux"
        PYTHON_CMD="python3"
    fi
}

# Use detected environment
detect_environment
echo "Running on $PLATFORM"
$PYTHON_CMD --version
```

## Enterprise Pre-Commit Hook Implementation

### Security-Focused Credential Detection

Implement sophisticated credential detection that goes beyond simple keyword matching:

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
    )

    for file in $staged_files; do
        if [ -f "$file" ]; then
            for pattern in "${patterns[@]}"; do
                if git show ":$file" | grep -qiE "$pattern"; then
                    violations+="Potential credential detected in $file\n"
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

# Execute security validation
check_credentials || exit 1
```

**Credential Pattern Explanation**:

| Pattern | Matches | Example |
|---------|---------|---------|
| `password\s*[=:]\s*['\"][^'\"]{8,}` | Password assignments | `password = "MySecret123"` |
| `api[_-]?key\s*[=:]\s*['\"][^'\"]{20,}` | API key assignments | `api_key: "abcd1234efgh5678ijkl"` |
| `-----BEGIN (RSA )?PRIVATE KEY-----` | Private keys | SSH/TLS private keys |
| `mysql://.*:.*@` | Database URLs | `mysql://user:pass@host` |
| `AKIA[0-9A-Z]{16}` | AWS Access Keys | `AKIAIOSFODNN7EXAMPLE` |

### Comprehensive Code Quality Validation

Implement multi-language code quality enforcement that adapts to your technology stack:

```bash
#!/usr/bin/env bash
# Enterprise code quality validation framework

validate_code_quality() {
    local staged_files=$(git diff --cached --name-only --diff-filter=ACM)
    local quality_violations=0

    echo "Performing code quality validation..."

    for file in $staged_files; do
        case "$file" in
            *.js|*.jsx|*.ts|*.tsx)
                if command -v npx >/dev/null 2>&1; then
                    echo "Linting JavaScript/TypeScript: $file"
                    npx eslint "$file" || ((quality_violations++))

                    if [[ "$file" =~ \.(ts|tsx)$ ]]; then
                        echo "Type checking: $file"
                        npx tsc --noEmit --skipLibCheck "$file" || ((quality_violations++))
                    fi
                fi
                ;;
            *.py)
                if command -v python3 >/dev/null 2>&1; then
                    echo "Linting Python: $file"
                    python3 -m flake8 "$file" || ((quality_violations++))

                    if command -v mypy >/dev/null 2>&1; then
                        echo "Type checking Python: $file"
                        python3 -m mypy "$file" || ((quality_violations++))
                    fi
                fi
                ;;
            *.cs)
                if command -v dotnet >/dev/null 2>&1; then
                    echo "Formatting C#: $file"
                    dotnet format --verify-no-changes --include "$file" || ((quality_violations++))
                fi
                ;;
            *.go)
                if command -v go >/dev/null 2>&1; then
                    echo "Formatting Go: $file"
                    if ! gofmt -l "$file" | grep -q .; then
                        gofmt -w "$file"
                        git add "$file"
                    fi

                    echo "Linting Go: $file"
                    golint "$file" || ((quality_violations++))
                fi
                ;;
        esac
    done

    if [ $quality_violations -gt 0 ]; then
        echo "Code quality validation failed with $quality_violations violations"
        echo "Please fix the issues above before committing"
        return 1
    fi

    echo "Code quality validation passed"
    return 0
}

# Execute quality validation
validate_code_quality || exit 1
```

**Language-Specific Tooling**:

| Language | Linting Tool | Formatting Tool | Type Checking | Required |
|----------|-------------|-----------------|---------------|----------|
| **JavaScript/TypeScript** | ESLint | Prettier | TypeScript | npm/npx |
| **Python** | Flake8, Pylint | Black | mypy | pip |
| **C#** | dotnet analyze | dotnet format | Built-in | .NET SDK |
| **Go** | golint, go vet | gofmt | Built-in | Go toolchain |
| **Java** | Checkstyle, PMD | google-java-format | javac | JDK |
| **Ruby** | RuboCop | RuboCop | Sorbet | gem |

### Intelligent Test Execution Strategy

Implement smart test execution that runs only relevant tests based on changed code:

```bash
#!/usr/bin/env bash
# Intelligent test execution based on change impact

execute_relevant_tests() {
    local changed_files=$(git diff --cached --name-only)
    local test_failures=0

    echo "Analyzing test requirements for changed files..."

    # Check if source code changes require testing
    if echo "$changed_files" | grep -qE "\.(js|jsx|ts|tsx|py|cs|go)$"; then
        echo "Source code changes detected, running relevant tests..."

        # JavaScript/TypeScript projects
        if [ -f "package.json" ] && command -v npm >/dev/null 2>&1; then
            if echo "$changed_files" | grep -qE "\.(js|jsx|ts|tsx)$"; then
                echo "Running JavaScript/TypeScript tests..."
                npm test -- --findRelatedTests $changed_files --passWithNoTests || ((test_failures++))
            fi
        fi

        # Python projects
        if [ -f "requirements.txt" ] || [ -f "pyproject.toml" ]; then
            if echo "$changed_files" | grep -qE "\.py$"; then
                echo "Running Python tests..."
                if command -v pytest >/dev/null 2>&1; then
                    pytest --tb=short || ((test_failures++))
                elif command -v python3 >/dev/null 2>&1; then
                    python3 -m unittest discover || ((test_failures++))
                fi
            fi
        fi

        # .NET projects
        if find . -name "*.csproj" -o -name "*.sln" | grep -q .; then
            if echo "$changed_files" | grep -qE "\.cs$"; then
                echo "Running .NET tests..."
                dotnet test --no-build --verbosity minimal || ((test_failures++))
            fi
        fi

        # Go projects
        if [ -f "go.mod" ]; then
            if echo "$changed_files" | grep -qE "\.go$"; then
                echo "Running Go tests..."
                go test ./... || ((test_failures++))
            fi
        fi
    fi

    if [ $test_failures -gt 0 ]; then
        echo "Test execution failed"
        echo "Please fix failing tests before committing"
        return 1
    fi

    echo "All relevant tests passed"
    return 0
}

# Execute intelligent testing
execute_relevant_tests || exit 1
```

**Test Strategy Matrix**:

| Project Size | Pre-Commit Tests | Pre-Push Tests | CI/CD Tests |
|--------------|------------------|----------------|-------------|
| **Small** (< 100 files) | All unit tests | All tests | All tests + integration |
| **Medium** (100-1000 files) | Changed files only | All unit tests | All tests + E2E |
| **Large** (> 1000 files) | Changed files only | Affected modules | Full suite + coverage |

## Advanced Commit Message Automation

### Prepare-Commit-Msg Hook Implementation

Automate commit message generation to ensure consistency and include necessary metadata:

```bash
#!/usr/bin/env bash
# Automated commit message enhancement with Azure DevOps integration

enhance_commit_message() {
    local commit_msg_file="$1"
    local commit_source="$2"
    local sha="$3"

    # Skip automation for merge commits, amend commits, etc.
    if [ -n "$commit_source" ]; then
        return 0
    fi

    # Get current branch information
    local current_branch=$(git branch --show-current)
    local branch_prefix=""

    # Extract work item ID from branch name if present
    if [[ "$current_branch" =~ ^(feature|bugfix|hotfix)/([0-9]+) ]]; then
        local work_item_id="${BASH_REMATCH[2]}"
        branch_prefix="AB#$work_item_id: "
    elif [[ "$current_branch" =~ ^(feature|bugfix|hotfix)/(.+)$ ]]; then
        local feature_name="${BASH_REMATCH[2]}"
        branch_prefix="[$feature_name] "
    fi

    # Read existing commit message
    local existing_message=$(cat "$commit_msg_file")

    # Only add prefix if not already present
    if [ ! -z "$branch_prefix" ] && ! echo "$existing_message" | grep -q "^$branch_prefix"; then
        # Create enhanced commit message
        echo "${branch_prefix}${existing_message}" > "$commit_msg_file"
    fi

    # Add commit template if message is empty
    if [ -z "$existing_message" ] || [ "$existing_message" = "" ]; then
        cat >> "$commit_msg_file" << EOF
${branch_prefix}Brief description of changes

# Detailed explanation of what and why changes were made
#
# Include:
# - What problem this solves
# - Why this approach was chosen
# - Any breaking changes or migration notes
#
# Link to work items: AB#1234
EOF
    fi
}

enhance_commit_message "$@"
```

**Branch Naming Patterns**:

| Branch Pattern | Extracted Info | Commit Prefix | Example |
|----------------|---------------|---------------|---------|
| `feature/12345-oauth` | Work item 12345 | `AB#12345:` | `AB#12345: implement OAuth` |
| `bugfix/98765-crash` | Work item 98765 | `AB#98765:` | `AB#98765: fix null pointer` |
| `hotfix/55555-security` | Work item 55555 | `AB#55555:` | `AB#55555: patch XSS vuln` |
| `feature/oauth-integration` | Feature name | `[oauth-integration]` | `[oauth-integration] add provider` |

### Commit Message Validation Hook

Ensure commit messages meet organizational standards:

```bash
#!/usr/bin/env bash
# Comprehensive commit message validation

validate_commit_message() {
    local commit_msg_file="$1"
    local commit_message=$(cat "$commit_msg_file")

    # Remove comment lines for validation
    local clean_message=$(echo "$commit_message" | grep -v '^#' | sed '/^$/d')

    # Check minimum length
    if [ ${#clean_message} -lt 10 ]; then
        echo "Commit message too short (minimum 10 characters)"
        return 1
    fi

    # Check maximum length for first line
    local first_line=$(echo "$clean_message" | head -n1)
    if [ ${#first_line} -gt 72 ]; then
        echo "Commit message first line too long (maximum 72 characters)"
        echo "Current length: ${#first_line}"
        return 1
    fi

    # Check for work item reference in enterprise environments
    if ! echo "$clean_message" | grep -qE "(AB#[0-9]+|#[0-9]+|closes #[0-9]+|fixes #[0-9]+)"; then
        echo "Commit message should reference a work item (e.g., AB#1234 or #1234)"
        echo "Continue anyway? (y/N)"
        read -r response
        if [[ ! "$response" =~ ^[Yy]$ ]]; then
            return 1
        fi
    fi

    # Check for conventional commit format (optional)
    if echo "$first_line" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: "; then
        echo "Conventional commit format detected"
    fi

    echo "Commit message validation passed"
    return 0
}

validate_commit_message "$@" || exit 1
```

**Commit Message Standards**:

| Element | Requirement | Example | Why |
|---------|-------------|---------|-----|
| **Length** | First line ≤ 72 chars | `feat: add OAuth authentication` | Readability in git log |
| **Work Item** | Reference ID | `AB#12345` or `#12345` | Traceability |
| **Format** | Conventional Commits | `feat(auth): add OAuth support` | Automation, changelog |
| **Body** | Wrap at 72 chars | Multi-line explanation | Readability |

## Azure DevOps Integration

### Server-Side Hook Integration

Use Azure DevOps service hooks for server-side automation:

```bash
#!/usr/bin/env bash
# Azure DevOps webhook integration for advanced workflows

trigger_azure_validation() {
    local branch_name=$(git branch --show-current)
    local commit_sha=$(git rev-parse HEAD)

    # Trigger Azure Pipelines validation build
    if command -v az >/dev/null 2>&1; then
        echo "Triggering Azure DevOps validation pipeline..."

        az pipelines build queue \
            --definition-name "PR-Validation" \
            --branch "$branch_name" \
            --commit-id "$commit_sha" \
            --output table
    fi
}

# Integration with Azure Boards
update_work_item() {
    local commit_message="$1"

    # Extract work item ID from commit message
    if [[ "$commit_message" =~ AB#([0-9]+) ]]; then
        local work_item_id="${BASH_REMATCH[1]}"

        echo "Updating Azure Boards work item #$work_item_id..."

        # Add commit information to work item
        az boards work-item update \
            --id "$work_item_id" \
            --discussion "Commit $(git rev-parse --short HEAD): $(echo "$commit_message" | head -n1)"
    fi
}
```

**Azure CLI Commands**:

| Command | Purpose | Trigger Point |
|---------|---------|---------------|
| `az pipelines build queue` | Trigger validation build | post-commit, pre-push |
| `az boards work-item update` | Add commit to work item discussion | post-commit |
| `az repos pr create` | Create pull request | custom script |
| `az repos policy evaluation list` | Check policy status | pre-push |

## Performance Optimization and Best Practices

### Hook Performance Guidelines

Implement performance optimizations to maintain developer productivity:

```bash
#!/usr/bin/env bash
# Performance-optimized hook implementation

optimize_hook_performance() {
    # Cache expensive operations
    local cache_dir=".git/hooks-cache"
    mkdir -p "$cache_dir"

    # Only run expensive checks on changed files
    local changed_files=$(git diff --cached --name-only)

    # Implement timeout protection
    timeout 30s expensive_operation || {
        echo "Hook operation timed out, skipping..."
        return 0
    }

    # Provide progress feedback for long operations
    echo "Running validation (this may take a moment)..."
}

# Implement graceful degradation
fallback_validation() {
    echo "Primary validation failed, running minimal checks..."
    # Implement basic validation as fallback
}
```

**Performance Targets**:

| Hook Type | Target Duration | User Experience | Action if Exceeded |
|-----------|----------------|-----------------|-------------------|
| **pre-commit** | < 10 seconds | Acceptable wait | Warn, offer skip |
| **prepare-commit-msg** | < 1 second | Imperceptible | Critical - must optimize |
| **commit-msg** | < 2 seconds | Barely noticeable | Acceptable |
| **post-commit** | < 5 seconds | Async, don't block | Run in background |
| **pre-push** | < 30 seconds | Expected wait | Show progress |

### Best Practices Summary

**DO's**:
- ✅ Make hooks cross-platform compatible (`#!/usr/bin/env bash`)
- ✅ Provide clear error messages with remediation steps
- ✅ Allow bypassing for emergencies (`--no-verify`)
- ✅ Version control hooks in repository
- ✅ Test hooks before deploying to team
- ✅ Optimize for performance (< 10s for pre-commit)
- ✅ Provide progress feedback for long operations

**DON'Ts**:
- ❌ Hardcode platform-specific paths
- ❌ Block commits for non-critical issues
- ❌ Run full test suite in pre-commit (use pre-push)
- ❌ Modify staged files without re-staging
- ❌ Fail silently without error messages
- ❌ Assume tools are installed (check with `command -v`)
- ❌ Make hooks run longer than 30 seconds

This comprehensive implementation guide provides the foundation for sophisticated enterprise Git hooks that enhance security, quality, and compliance while maintaining developer productivity and workflow efficiency.

## Critical Notes
- 🎯 **Cross-platform**: Use `#!/usr/bin/env bash` for universal compatibility
- 💡 **Security focus**: Advanced credential detection with comprehensive patterns
- ⚠️ **Multi-language support**: JavaScript, Python, C#, Go with appropriate tooling
- 📊 **Intelligent testing**: Run only tests relevant to changed code
- 🔄 **Azure DevOps integration**: Trigger pipelines, update work items automatically
- ✨ **Performance optimization**: < 10s for pre-commit, timeout protection, graceful degradation

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-git-hooks/3-implement)
