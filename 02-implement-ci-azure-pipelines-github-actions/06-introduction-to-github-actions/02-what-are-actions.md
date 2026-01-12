# What are Actions?

GitHub Actions is a platform for automating developer workflows directly within GitHub repositories. It enables continuous integration, continuous deployment, and general-purpose automation triggered by repository events.

## Core Concepts

### Actions

**Actions** are individual, reusable units of code that perform specific tasks. Think of them as building blocks you combine to create automation workflows.

**Characteristics**:
- Encapsulate single responsibility (checkout code, setup environment, run tests)
- Published to GitHub Marketplace or stored in repositories
- Versioned using tags, branches, or commit SHAs
- Can be created using Docker containers, JavaScript, or composite actions

**Example Actions**:
```yaml
- uses: actions/checkout@v4  # Action to checkout code
- uses: actions/setup-node@v4  # Action to install Node.js
- uses: actions/upload-artifact@v4  # Action to upload build artifacts
```

### Workflows

**Workflows** are automated processes composed of one or more jobs. They define WHEN (triggers) and WHAT (jobs/steps) to automate.

**Characteristics**:
- Defined in YAML files stored in `.github/workflows/` directory
- Triggered by repository events (push, pull_request), schedules, or manual dispatch
- Can run multiple jobs in parallel or sequence
- Provide status checks, logs, and artifact storage

## Common Use Cases

### 1. CI/CD Pipelines

**Continuous Integration**:
```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm install
    - run: npm test
```

**Benefits**:
- Automatic build and test on every commit
- Fast feedback on code quality
- Prevents broken code from reaching main branch

**Continuous Deployment**:
```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Deploy to Azure
      uses: azure/webapps-deploy@v2
      with:
        app-name: myapp
        publish-profile: ${{ secrets.AZURE_PUBLISH_PROFILE }}
```

**Benefits**:
- Automatic deployment on main branch push
- Consistent deployment process
- Reduced manual errors

### 2. Code Quality and Security

**Automated Testing**:
```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
    runs-on: ${{ matrix.os }}
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm test
```

**Code Analysis**:
```yaml
jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run CodeQL Analysis
      uses: github/codeql-action/analyze@v2
    - name: Run ESLint
      run: npm run lint
```

**Dependency Management**:
```yaml
jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run Dependabot
      run: npm audit --audit-level=high
    - name: Check for vulnerabilities
      uses: snyk/actions/node@master
```

### 3. Project Management Automation

**Issue Triage**:
```yaml
name: Label Issues
on:
  issues:
    types: [opened]

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/labeler@v4
      with:
        repo-token: ${{ secrets.GITHUB_TOKEN }}
```

**Auto-Merge Dependabot PRs**:
```yaml
name: Auto-Merge Dependabot
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  auto-merge:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: gh pr merge --auto --squash "$PR_URL"
      env:
        PR_URL: ${{ github.event.pull_request.html_url }}
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Release Automation**:
```yaml
name: Create Release
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Generate changelog
      run: ./scripts/generate-changelog.sh
    - name: Create GitHub Release
      uses: softprops/action-gh-release@v1
      with:
        body_path: CHANGELOG.md
        files: dist/*
```

## How Actions Work

### Workflow Execution Flow

```
Repository Event (push, PR, schedule)
         ↓
Workflow File (.github/workflows/ci.yml)
         ↓
Runner Allocation (GitHub-hosted or self-hosted)
         ↓
Job Execution (parallel or sequential)
         ↓
Step Execution (commands, scripts, actions)
         ↓
Results & Artifacts (logs, artifacts, status checks)
```

### Workflow File Structure

```yaml
# .github/workflows/ci.yml
name: CI Pipeline              # Workflow name
on: [push, pull_request]       # Triggers (events)

jobs:                          # Job definitions
  build:                       # Job ID
    runs-on: ubuntu-latest     # Runner type
    steps:                     # Steps within job
    - uses: actions/checkout@v4  # Pre-built action
    - run: npm install            # Shell command
    - run: npm test               # Another command
```

### Runners

**Runners** are virtual machines that execute your workflows:

**GitHub-Hosted Runners**:
- Managed by GitHub
- Ubuntu, Windows, macOS available
- Pre-installed with common development tools
- 4-core CPU, 16 GB RAM, 14 GB SSD
- Usage-based billing

**Self-Hosted Runners**:
- Managed by you
- Custom hardware/software configurations
- Private network access
- No per-minute charges
- Full control over environment

## Action Ecosystem

### GitHub Marketplace

The [GitHub Marketplace](https://github.com/marketplace?type=actions) hosts 10,000+ pre-built actions:

**Popular Categories**:

| Category | Examples | Use Cases |
|----------|----------|-----------|
| **Deployment** | `azure/webapps-deploy`, `aws-actions/configure-aws-credentials` | Deploy to cloud platforms |
| **Testing** | `codecov/codecov-action`, `cypress-io/github-action` | Code coverage, E2E tests |
| **Security** | `github/codeql-action`, `snyk/actions` | Vulnerability scanning |
| **Notifications** | `slackapi/slack-github-action`, `microsoft/teams-action` | Team notifications |
| **Utilities** | `actions/cache`, `actions/upload-artifact` | Performance, artifact management |

### Action Types

**1. Docker Container Actions**:
```yaml
- uses: docker://alpine:3.8
  with:
    entrypoint: /usr/local/bin/my-script.sh
```

**2. JavaScript Actions**:
```yaml
- uses: actions/github-script@v7
  with:
    script: |
      console.log('Hello from JavaScript!');
```

**3. Composite Actions**:
```yaml
# action.yml
name: 'Composite Action'
runs:
  using: 'composite'
  steps:
  - run: echo "Step 1"
  - run: echo "Step 2"
```

## Critical Notes

🎯 **Actions are Reusable**: Write once, use across multiple workflows. Publish to Marketplace for community reuse.

💡 **Marketplace Verification**: Look for "Verified creator" badge on actions. GitHub verifies publishers' identities.

⚠️ **Version Pinning**: Use semantic versioning (`@v4`) for stability or commit SHAs (`@abc123`) for maximum security.

📊 **Workflow Minutes**: Free accounts get 2,000 minutes/month for private repos. Public repos have unlimited minutes.

🔄 **Event-Driven**: Workflows only run when triggered by events. No polling or scheduled checks consuming resources unnecessarily.

✨ **Native Integration**: GitHub Actions integrates with GitHub features: status checks on PRs, protected branches, required reviews.

## Quick Reference

### Basic Workflow Example

```yaml
name: Basic CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
```

### Common Actions

| Action | Purpose | Usage |
|--------|---------|-------|
| `actions/checkout@v4` | Clone repository | `uses: actions/checkout@v4` |
| `actions/setup-node@v4` | Install Node.js | `uses: actions/setup-node@v4` |
| `actions/setup-python@v4` | Install Python | `uses: actions/setup-python@v4` |
| `actions/upload-artifact@v4` | Upload artifacts | `uses: actions/upload-artifact@v4` |
| `actions/download-artifact@v4` | Download artifacts | `uses: actions/download-artifact@v4` |
| `actions/cache@v3` | Cache dependencies | `uses: actions/cache@v3` |

### Workflow Triggers

```yaml
# Push to branches
on:
  push:
    branches: [main, develop]

# Pull requests
on:
  pull_request:
    branches: [main]

# Schedule (cron)
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

# Manual trigger
on:
  workflow_dispatch:
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/2-what-are-actions)
