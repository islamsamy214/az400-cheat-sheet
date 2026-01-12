# Understand Workflows

Workflows are automated processes that build, test, package, release, or deploy your project. They consist of one or more jobs that run in response to specific events in your repository.

## Workflow Structure

Every workflow includes these key components organized hierarchically:

**Workflow Hierarchy**:
```
Workflow (YAML file in .github/workflows/)
└── Triggers/Events (when to run)
└── Jobs (what to run)
    └── Steps (how to run)
        └── Actions/Commands (specific tasks)
```

### Core Components

| Component | Purpose | Required | Example |
|-----------|---------|----------|---------|
| **name** | Workflow display name in UI | No | `name: CI Pipeline` |
| **on** | Event triggers | Yes | `on: [push, pull_request]` |
| **jobs** | Collection of jobs to execute | Yes | `jobs: build: ...` |
| **runs-on** | Runner environment | Yes | `runs-on: ubuntu-latest` |
| **steps** | Sequential actions within job | Yes | `steps: - name: Build` |

### Triggers (Events)

Define when your workflow should run:

```yaml
on:
  # Single event
  push

  # Multiple events (array)
  on: [push, pull_request]

  # Detailed configuration
  push:
    branches: [main, develop]
    paths:
      - 'src/**'
      - '!docs/**'
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * 0'  # Weekly Sunday 2 AM
  workflow_dispatch:  # Manual trigger
```

### Jobs

A set of steps that execute on the same runner.

**Characteristics**:
- Jobs run in parallel by default
- Can be configured to run sequentially using `needs`
- Each job gets a fresh virtual environment
- Share artifacts using upload/download actions

```yaml
jobs:
  build:  # Job ID
    name: Build Application  # Display name
    runs-on: ubuntu-latest  # Runner
    steps:  # Job steps
    - uses: actions/checkout@v4
    - run: npm run build
```

### Steps

Individual tasks within a job.

**Step Types**:
1. **Action steps**: Use pre-built actions (`uses:`)
2. **Command steps**: Run shell commands (`run:`)
3. **Script steps**: Execute multi-line scripts

```yaml
steps:
  # Action step
  - name: Checkout code
    uses: actions/checkout@v4
  
  # Command step
  - name: Install dependencies
    run: npm install
  
  # Multi-line script step
  - name: Build and test
    run: |
      npm run build
      npm test
```

### Runners

The compute environment where your jobs execute.

**GitHub-Hosted Runners**:
- Ubuntu, Windows, macOS
- Pre-configured with development tools
- 4-core CPU, 16 GB RAM, 14 GB SSD
- Managed and updated by GitHub

**Self-Hosted Runners**:
- Run on your infrastructure
- Custom hardware/software
- Private network access
- No per-minute billing

## Workflow Location and Naming

Workflows are stored in the `.github/workflows/` directory of your repository.

**File Requirements**:
- Must use `.yml` or `.yaml` extension
- Must be valid YAML syntax
- Stored in `.github/workflows/` directory
- Filename becomes workflow identifier in UI

**Directory Structure**:
```
repository-root/
└── .github/
    └── workflows/
        ├── ci.yml
        ├── deploy.yml
        └── codeql-analysis.yml
```

## Modern Workflow Example

Here's a comprehensive workflow demonstrating current best practices:

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: "0 2 * * 0"  # Weekly dependency check

env:
  NODE_VERSION: "20"

jobs:
  test:
    name: Test and Lint
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Run tests with coverage
        run: npm run test:coverage

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: test

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/
```

## Key Improvements in Modern Workflows

This example demonstrates current best practices:

✅ **Latest action versions**: Using `@v4` versions of popular actions

✅ **Dependency caching**: `cache: "npm"` speeds up workflow execution

✅ **Environment variables**: Centralized configuration with `env:`

✅ **Job dependencies**: `build` runs only after `test` succeeds

✅ **Artifact handling**: Proper storage and sharing of build outputs

✅ **Security**: Using secrets for sensitive data (`${{ secrets.CODECOV_TOKEN }}`)

✅ **Descriptive names**: Clear `name:` fields for jobs and steps

✅ **Path filters**: Optimize triggers with branch and path filters

## Workflow Best Practices

### 1. Structure and Organization

```yaml
# Good: Clear, organized workflow
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm run lint

  test:
    name: Unit Tests
    needs: lint
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm test
```

### 2. Efficiency Optimization

```yaml
on:
  push:
    branches: [main]
    paths:  # Run only when code changes
      - 'src/**'
      - 'package*.json'
    paths-ignore:  # Ignore documentation changes
      - 'docs/**'
      - '**.md'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    
    - run: npm ci  # Faster with cache
```

### 3. Security Considerations

```yaml
permissions:
  contents: read
  security-events: write
  pull-requests: write

env:
  # Use secrets for sensitive data
  API_KEY: ${{ secrets.API_KEY }}
  # Use variables for non-sensitive config
  ENVIRONMENT: ${{ vars.ENVIRONMENT }}

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Run security scan
      uses: github/codeql-action/analyze@v2
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
```

### 4. Error Handling and Debugging

```yaml
steps:
  - name: Debug information
    if: env.ACTIONS_STEP_DEBUG == 'true'
    run: |
      echo "Runner OS: $RUNNER_OS"
      echo "Workflow: $GITHUB_WORKFLOW"
      echo "Event: $GITHUB_EVENT_NAME"
  
  - name: Run tests
    run: npm test
    continue-on-error: false  # Fail job on error (default)
  
  - name: Upload logs
    if: always()  # Run even if previous steps fail
    uses: actions/upload-artifact@v4
    with:
      name: logs
      path: logs/
```

## Additional Resources

- [Starter Workflows](https://github.com/actions/starter-workflows) - Pre-built templates for common scenarios
- [Workflow Syntax Reference](https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions) - Complete syntax documentation
- [Marketplace Actions](https://github.com/marketplace?type=actions) - Community-contributed actions

## Critical Notes

🎯 **File Location**: Workflows MUST be in `.github/workflows/` directory to be recognized by GitHub Actions.

💡 **YAML Syntax**: Indentation matters! Use 2 spaces (not tabs). Validate YAML before committing.

⚠️ **Default Branch**: Workflows in feature branches won't run until merged unless triggered by events in that branch.

📊 **Job Limits**: Free accounts get 20 concurrent jobs. Enterprise gets 180. Plan complex workflows accordingly.

🔄 **Workflow Re-runs**: You can re-run failed workflows from the Actions tab - useful for transient failures.

✨ **Matrix Builds**: Use `strategy.matrix` to run jobs across multiple OS/language versions automatically.

## Quick Reference

### Minimal Workflow

```yaml
name: Minimal CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: echo "Hello, GitHub Actions!"
```

### Complete Workflow Template

```yaml
name: Complete CI/CD

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

env:
  NODE_VERSION: 20

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: npm
    - run: npm ci
    - run: npm run build
    - uses: actions/upload-artifact@v4
      with:
        name: dist
        path: dist/
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/4-understand-workflows)
