# Examine Release and Test Actions

Understanding how to access workflow logs, debug issues, manage action versions, and test workflows are essential skills for maintaining reliable GitHub Actions automation.

## Accessing Workflow Logs

### Viewing Workflow Runs

1. Navigate to repository → **Actions** tab
2. Select a workflow from the left sidebar
3. Click a specific workflow run
4. Click a job to expand steps
5. Click a step to view detailed logs

**UI Hierarchy**:
```
Repository → Actions Tab
  └─ Workflow Name (e.g., "CI")
      └─ Workflow Run (#123)
          └─ Job (e.g., "build")
              └─ Step (e.g., "Run tests")
                  └─ Step logs (stdout/stderr)
```

### Log Details

Each step shows:
- ✅ Success/failure status
- ⏱️ Execution time
- 📄 Full stdout/stderr output
- 🔍 Expandable sections (setup, execution, post-actions)

### Downloading Logs

```bash
# Download via UI:
# Actions → Workflow Run → ⋮ (three dots) → Download log archive

# Download via GitHub CLI:
gh run download <run-id>

# Download specific job logs:
gh run view <run-id> --log
```

## Enhanced Debugging

### Enable Debug Logging

Add repository secrets to enable verbose debugging:

#### Step 1: Create Debug Secrets

Repository Settings → Secrets and variables → Actions → New repository secret

| Secret Name | Value | Purpose |
|-------------|-------|---------|
| `ACTIONS_RUNNER_DEBUG` | `true` | Detailed runner diagnostics |
| `ACTIONS_STEP_DEBUG` | `true` | Verbose step output |

#### Step 2: Trigger Workflow

Debug logging activates automatically on subsequent runs.

### Debug Output Example

**Normal Output**:
```
Run actions/checkout@v4
  Syncing repository: owner/repo
  Getting Git version info
  Fetching the repository
```

**Debug Output** (with `ACTIONS_STEP_DEBUG=true`):
```
##[debug]Evaluating condition for step: 'Checkout code'
##[debug]Evaluating: success()
##[debug]Evaluating success:
##[debug]=> true
##[debug]Result: true
##[debug]Starting: Checkout code
Run actions/checkout@v4
##[debug]Loading inputs
##[debug]Evaluating: github.workspace
##[debug]=> '/home/runner/work/repo/repo'
##[debug]Result: '/home/runner/work/repo/repo'
  Syncing repository: owner/repo
##[debug]Getting Git version info
##[debug]Getting branch list
  Getting Git version info
  Fetching the repository
##[debug]Creating fetch options
##[debug]git fetch --depth=1 origin +refs/heads/main:refs/remotes/origin/main
```

### Manual Debug Output

Add debug output in your workflow:

```yaml
steps:
  - name: Debug information
    run: |
      echo "::debug::This is a debug message"
      echo "::notice::This is a notice message"
      echo "::warning::This is a warning message"
      echo "::error::This is an error message"
      
      echo "Event: ${{ github.event_name }}"
      echo "Ref: ${{ github.ref }}"
      echo "SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
  
  - name: Dump GitHub context
    env:
      GITHUB_CONTEXT: ${{ toJson(github) }}
    run: echo "$GITHUB_CONTEXT"
  
  - name: Dump environment variables
    run: env | sort
```

**Log Annotations**:
- `::debug::` - Debug message (only visible with `ACTIONS_STEP_DEBUG`)
- `::notice::` - Informational message (blue in UI)
- `::warning::` - Warning message (yellow in UI)
- `::error::` - Error message (red in UI)

### Debugging Failed Steps

```yaml
- name: Run tests
  id: test
  run: npm test
  continue-on-error: true

- name: Debug on failure
  if: steps.test.outcome == 'failure'
  run: |
    echo "::error::Tests failed!"
    cat test-results.xml
    echo "--- Environment ---"
    env | grep -i node
    echo "--- Disk space ---"
    df -h
```

## Action Version Management

Actions are versioned using Git tags and branches. Choosing the right version reference is critical for stability and security.

### Version Reference Strategies

| Strategy | Format | Updates | Use Case |
|----------|--------|---------|----------|
| **Semantic Tag** | `@v4` | Major version (breaking changes) | Recommended for most workflows |
| **Specific Tag** | `@v4.2.1` | Never (pinned) | Maximum stability |
| **SHA Commit** | `@abc123...` | Never (immutable) | Security-critical workflows |
| **Branch** | `@main` | Continuous (risky) | Development/testing only |

### Semantic Versioning (Recommended)

```yaml
steps:
  # ✅ Good: Major version tag (gets patches and features)
  - uses: actions/checkout@v4
  
  # Updates automatically to:
  # - v4.0.1 (patch: bug fix)
  # - v4.1.0 (minor: new feature)
  # But NOT to:
  # - v5.0.0 (major: breaking change)
```

**Advantages**:
- ✅ Automatic security patches
- ✅ New features without breaking changes
- ✅ Easy to read and maintain

**Best Practice**: Use major version tags (`@v4`) for most actions.

### Specific Version Tag

```yaml
steps:
  # Pin to exact version
  - uses: actions/checkout@v4.2.1
  
  # Never updates - you control when to upgrade
```

**Advantages**:
- ✅ Maximum stability
- ✅ Reproducible builds
- ✅ Control upgrade timing

**Use When**:
- Critical production workflows
- Compatibility concerns with new versions
- Regulatory/compliance requirements

### SHA Commit Reference

```yaml
steps:
  # Use specific commit SHA (full 40-character SHA)
  - uses: actions/checkout@a12b3c4d5e6f7890abcdef1234567890abcdef12
  
  # Immutable - cannot be changed (even by action maintainer)
```

**Advantages**:
- ✅ Immutable (highest security)
- ✅ Cannot be modified after creation
- ✅ Cryptographically verified

**Use When**:
- Maximum security required
- Supply chain attack concerns
- Auditing and compliance needs

**Finding SHA**:
```bash
# Get SHA for a tag
git ls-remote https://github.com/actions/checkout v4.2.1
# Output: a12b3c4d5e6f7890abcdef1234567890abcdef12  refs/tags/v4.2.1
```

### Branch Reference (Not Recommended)

```yaml
steps:
  # ❌ Not recommended: Branch reference
  - uses: actions/checkout@main
  
  # Gets latest commit on 'main' - can break at any time
```

**Disadvantages**:
- ❌ Unpredictable behavior
- ❌ Breaking changes without warning
- ❌ Difficult to debug

**Only Use For**:
- Testing unreleased features
- Development workflows
- Temporary debugging

### Version Management Best Practices

#### 1. Update Actions Regularly

```yaml
# Create a schedule to review action versions
name: Update Dependencies
on:
  schedule:
    - cron: '0 0 * * MON'  # Every Monday

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    # Use Dependabot or Renovate to automate updates
```

#### 2. Use Dependabot for Actions

Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

**Result**: Automatic PRs for action updates.

#### 3. Review Release Notes

Before upgrading, check:
- [Action's releases page](https://github.com/actions/checkout/releases)
- Breaking changes
- New features
- Security fixes

#### 4. Test in Non-Production First

```yaml
# staging workflow uses latest versions
uses: actions/checkout@v4

# production workflow uses pinned versions
uses: actions/checkout@v4.2.1
```

### Version Migration Example

**Initial State** (outdated):
```yaml
- uses: actions/checkout@v2  # Old major version
- uses: actions/setup-node@v2  # Old major version
```

**Step 1**: Update to latest major versions
```yaml
- uses: actions/checkout@v4  # Latest major
- uses: actions/setup-node@v4  # Latest major
```

**Step 2**: Test workflow
```bash
# Trigger workflow manually
gh workflow run ci.yml
```

**Step 3**: Monitor for issues
```bash
# Check workflow runs
gh run list --workflow=ci.yml

# View specific run
gh run view <run-id>
```

**Step 4**: Pin if stable
```yaml
# Once confident, optionally pin to specific version
- uses: actions/checkout@v4.2.1
- uses: actions/setup-node@v4.1.0
```

## Testing Workflows

### 1. Manual Trigger for Testing

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:  # ✅ Add manual trigger for testing

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - run: echo "Testing workflow"
```

Trigger manually:
- UI: Actions → CI → Run workflow
- CLI: `gh workflow run ci.yml`

### 2. Local Testing with `act`

Test workflows locally before pushing:

```bash
# Install act
# macOS:
brew install act

# Linux:
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Run workflow locally
act push

# Run specific job
act -j build

# Run with secrets
act -s GITHUB_TOKEN=your_token

# List workflows
act -l
```

**Limitations of `act`**:
- May not support all GitHub Actions features
- Some actions require actual GitHub environment
- Docker required

### 3. Branch-Based Testing

```yaml
name: CI
on:
  push:
    branches:
      - main
      - develop
      - 'test/**'  # Allow testing on test/* branches

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm test
```

**Workflow**:
1. Create test branch: `git checkout -b test/new-feature`
2. Modify workflow: `.github/workflows/ci.yml`
3. Push and test: `git push origin test/new-feature`
4. Verify in Actions tab
5. Merge when stable

### 4. Testing with Different Inputs

```yaml
name: Test Workflow
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - development
          - staging
          - production
      dry-run:
        description: 'Dry run (no actual changes)'
        required: false
        type: boolean
        default: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Deploy
      run: |
        if [[ "${{ inputs.dry-run }}" == "true" ]]; then
          echo "DRY RUN: Would deploy to ${{ inputs.environment }}"
        else
          echo "Deploying to ${{ inputs.environment }}"
          # actual deployment commands
        fi
```

### 5. Integration Testing with Pull Requests

Test workflow changes in PRs before merging:

```yaml
name: Test Workflow Changes
on:
  pull_request:
    paths:
      - '.github/workflows/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Validate workflow syntax
      run: |
        # Check YAML syntax
        for file in .github/workflows/*.yml; do
          echo "Validating $file"
          python3 -c "import yaml; yaml.safe_load(open('$file'))"
        done
    
    - name: Test workflow
      run: echo "Workflow changes detected - triggering test run"
```

## Testing Checklist

Before deploying workflows to production:

- ☑️ **Syntax Valid**: YAML parses without errors
- ☑️ **Manually Triggered**: Test with `workflow_dispatch`
- ☑️ **Branch Tested**: Verify on test branch
- ☑️ **Secrets Available**: Ensure all secrets configured
- ☑️ **Permissions Correct**: Verify token permissions
- ☑️ **Dependencies Updated**: Actions on latest versions
- ☑️ **Logs Reviewed**: Check for warnings/errors
- ☑️ **Idempotent**: Can run multiple times safely
- ☑️ **Error Handling**: Fails gracefully
- ☑️ **Notifications Work**: Success/failure notifications sent

## Troubleshooting Common Issues

### Workflow Not Triggering

```yaml
# ❌ Problem: Workflow doesn't run on push
on:
  push:
    branches: [main]

# ✅ Check:
# 1. Workflow file in .github/workflows/ directory?
# 2. YAML syntax valid?
# 3. Branch name matches exactly? (main vs master)
# 4. Workflow file on the branch being pushed?
# 5. Actions enabled in repo settings?
```

### Action Not Found

```yaml
# ❌ Problem: Action not found
- uses: actions/checkout@v99

# ✅ Solution:
# 1. Check action exists: https://github.com/actions/checkout
# 2. Verify version/tag exists: https://github.com/actions/checkout/tags
# 3. Use valid version: @v4
```

### Permission Denied

```yaml
# ❌ Problem: Permission denied when writing to repo
steps:
  - run: git push origin main

# ✅ Solution: Add write permissions
permissions:
  contents: write

steps:
  - uses: actions/checkout@v4
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
  - run: git push origin main
```

### Timeout Issues

```yaml
# ❌ Problem: Workflow times out after 6 hours
jobs:
  long-job:
    runs-on: ubuntu-latest
    # times out at 360 minutes (default)

# ✅ Solution: Increase timeout or use self-hosted runner
jobs:
  long-job:
    runs-on: ubuntu-latest
    timeout-minutes: 120  # 2 hours (if sufficient)
    
  # Or use self-hosted for > 6 hour jobs
  very-long-job:
    runs-on: self-hosted
    # No timeout limit on self-hosted
```

## Critical Notes

🎯 **Version Strategy**: Use semantic version tags (`@v4`) for most actions - gets security patches without breaking changes.

💡 **Debug Secrets**: Enable `ACTIONS_RUNNER_DEBUG` and `ACTIONS_STEP_DEBUG` secrets for detailed debugging output.

⚠️ **SHA Pinning**: Use commit SHAs for security-critical workflows to prevent supply chain attacks.

📊 **Log Retention**: Workflow logs kept for 90 days (configurable from 1-400 days in Enterprise).

🔄 **Dependabot**: Use Dependabot to automatically create PRs for action updates.

✨ **Local Testing**: Use `act` to test workflows locally before pushing to GitHub.

## Quick Reference

### Version Selection Guide

| Scenario | Recommended Version | Example |
|----------|---------------------|---------|
| **Standard workflows** | Major version tag | `@v4` |
| **Production-critical** | Specific version tag | `@v4.2.1` |
| **Security-sensitive** | SHA commit | `@abc123...` |
| **Testing/development** | Branch (temporary) | `@main` |

### Debug Output Levels

```yaml
# Repository Secrets:
ACTIONS_RUNNER_DEBUG: true    # Runner-level diagnostics
ACTIONS_STEP_DEBUG: true      # Step-level verbose output

# In workflow:
- run: echo "::debug::Debug message"
- run: echo "::notice::Notice message"
- run: echo "::warning::Warning message"
- run: echo "::error::Error message"
```

### Testing Commands

```bash
# Manual trigger
gh workflow run ci.yml

# List workflows
gh workflow list

# View workflow runs
gh run list --workflow=ci.yml

# View specific run
gh run view <run-id>

# Local testing
act push
act -j build
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/9-examine-release-test-action)
