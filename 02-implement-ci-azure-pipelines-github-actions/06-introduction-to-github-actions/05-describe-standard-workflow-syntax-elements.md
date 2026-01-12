# Describe Standard Workflow Syntax Elements

GitHub Actions workflows use YAML syntax with specific elements that define when, where, and how your automation runs. Understanding these core syntax elements is essential for creating effective workflows.

## Essential Workflow Elements

### Top-Level Workflow Configuration

```yaml
name: CI/CD Pipeline  # Workflow name (optional but recommended)

on:  # Event triggers (required)
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: "0 2 * * 1"  # Weekly Monday 2 AM UTC

jobs:  # Job definitions (required)
  # Job configurations go here
```

### Core Syntax Elements Explained

| Element | Purpose | Required | Example |
|---------|---------|----------|---------|
| **name** | Workflow display name in GitHub UI | Optional | `name: "Build and Test"` |
| **on** | Event triggers for workflow execution | Required | `on: [push, pull_request]` |
| **jobs** | Collection of jobs to execute | Required | `jobs: build: ...` |
| **runs-on** | Specifies runner environment | Required | `runs-on: ubuntu-latest` |
| **steps** | Sequential actions within a job | Required | `steps: - name: ...` |
| **uses** | References pre-built actions | Optional | `uses: actions/checkout@v4` |
| **run** | Executes shell commands | Optional | `run: npm test` |

## Complete Workflow Example

```yaml
name: Node.js CI/CD Pipeline

# Event configuration
on:
  push:
    branches: [main, develop]
    paths-ignore: ["docs/**", "*.md"]
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

# Environment variables (workflow-level)
env:
  NODE_VERSION: "20"
  CI: true

# Job definitions
jobs:
  # Test job
  test:
    name: Run Tests
    runs-on: ubuntu-latest

    # Job-level environment variables
    env:
      DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}

    # Job steps
    steps:
      - name: Check out code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install dependencies
        run: |
          npm ci
          npm audit --audit-level=high

      - name: Run tests
        run: |
          npm run test:coverage
          npm run test:integration
        env:
          NODE_ENV: test

      - name: Upload coverage reports
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-reports
          path: coverage/
          retention-days: 30

  # Build job (depends on test)
  build:
    name: Build Application
    needs: test
    runs-on: ubuntu-latest

    outputs:
      build-version: ${{ steps.version.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install and build
        run: |
          npm ci --production
          npm run build

      - name: Generate version
        id: version
        run: |
          VERSION=$(date +%Y%m%d)-${GITHUB_SHA::8}
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - name: Save build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ steps.version.outputs.version }}
          path: |
            dist/
            package.json
```

## Advanced Syntax Elements

### Conditional Execution

```yaml
steps:
  - name: Deploy to production
    if: github.ref == 'refs/heads/main' && success()
    run: ./deploy.sh

  - name: Notify on failure
    if: failure()
    run: ./notify-failure.sh
```

**Common Conditions**:
- `success()` - Previous step succeeded (default)
- `failure()` - Previous step failed
- `always()` - Run regardless of status
- `cancelled()` - Workflow was cancelled

### Matrix Strategies

Run jobs across multiple configurations:

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
        include:
          - os: ubuntu-latest
            node-version: 22
            experimental: true
      fail-fast: false  # Don't cancel other jobs on first failure
    
    runs-on: ${{ matrix.os }}
    
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm test
```

**Result**: Creates 9 jobs (3 OS × 3 Node versions)

### Reusable Workflows

```yaml
jobs:
  call-reusable-workflow:
    uses: ./.github/workflows/reusable-tests.yml
    with:
      environment: production
      node-version: "20"
    secrets:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

## Syntax Reference

### Event Triggers

```yaml
# Single event
on: push

# Multiple events
on: [push, pull_request]

# Detailed configuration
on:
  push:
    branches:
      - main
      - 'releases/**'
    paths:
      - 'src/**'
    tags:
      - v1.*
  pull_request:
    types: [opened, synchronize, reopened]
  schedule:
    - cron: '0 8 * * MON-FRI'
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'
        required: true
        default: 'warning'
        type: choice
        options:
        - info
        - warning
        - debug
```

### Job Configuration

```yaml
jobs:
  my-job:
    name: My Job Name
    runs-on: ubuntu-latest
    needs: [previous-job]  # Dependencies
    if: success()  # Condition
    timeout-minutes: 30
    continue-on-error: false
    
    permissions:  # GitHub token permissions
      contents: read
      pull-requests: write
    
    env:  # Job-level environment variables
      JOB_VAR: value
    
    outputs:  # Job outputs for other jobs
      my-output: ${{ steps.step-id.outputs.value }}
    
    strategy:  # Matrix strategy
      matrix:
        os: [ubuntu-latest, windows-latest]
    
    steps:
      # Steps here
```

### Step Configuration

```yaml
steps:
  # Action step
  - name: Checkout code
    id: checkout
    uses: actions/checkout@v4
    with:
      fetch-depth: 1
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    if: success()
    continue-on-error: false
    timeout-minutes: 5
  
  # Command step
  - name: Run command
    run: npm test
    working-directory: ./src
    shell: bash
    env:
      NODE_ENV: test
```

### Context and Expressions

```yaml
steps:
  # Access GitHub context
  - run: echo "Repository: ${{ github.repository }}"
  
  # Access job context
  - run: echo "Job status: ${{ job.status }}"
  
  # Access step outputs
  - id: build
    run: echo "version=1.0.0" >> $GITHUB_OUTPUT
  - run: echo "Version: ${{ steps.build.outputs.version }}"
  
  # Functions
  - if: contains(github.event.head_commit.message, '[skip ci]')
    run: echo "Skipping CI"
  
  # Boolean operations
  - if: github.ref == 'refs/heads/main' && success()
    run: ./deploy.sh
```

## Best Practices for Workflow Syntax

### 1. Structure and Organization

✅ **Use descriptive names** for workflows, jobs, and steps:
```yaml
name: CI/CD Pipeline  # Not: workflow.yml
jobs:
  test:
    name: Run Unit Tests  # Not: test
```

✅ **Group related steps** logically within jobs

✅ **Keep workflows focused** on specific purposes (CI, CD, maintenance)

### 2. Efficiency Optimization

```yaml
# Use paths to limit unnecessary runs
on:
  push:
    paths:
      - 'src/**'
      - 'package*.json'
    paths-ignore:
      - 'docs/**'
      - '**.md'

# Cache dependencies
steps:
  - uses: actions/setup-node@v4
    with:
      cache: npm  # Built-in caching
  
  # Or use actions/cache for custom caching
  - uses: actions/cache@v3
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

✅ **Run independent jobs in parallel** (default behavior)

✅ **Use artifacts** to share data between jobs instead of rebuilding

### 3. Security Considerations

```yaml
permissions:
  contents: read  # Minimum required permissions
  security-events: write
  pull-requests: write

env:
  # Use secrets for sensitive data
  API_KEY: ${{ secrets.API_KEY }}
  # Use variables for non-sensitive configuration
  ENVIRONMENT: ${{ vars.ENVIRONMENT }}

steps:
  - name: Security scan
    env:
      TOKEN: ${{ secrets.SECURITY_TOKEN }}  # Never log secrets
    run: ./security-scan.sh
```

⚠️ **Never echo or log secrets** in workflow output

⚠️ **Use least-privilege permissions** with `permissions:` key

⚠️ **Pin action versions** for security: `uses: actions/checkout@v4` or commit SHA

### 4. Error Handling and Debugging

```yaml
steps:
  # Enable debug logging with repository secrets
  - name: Debug information
    if: runner.debug == '1'
    run: |
      echo "Runner OS: $RUNNER_OS"
      echo "Workflow: $GITHUB_WORKFLOW"
      echo "Event: $GITHUB_EVENT_NAME"
  
  # Continue on error for non-critical steps
  - name: Optional step
    run: npm run optional-task
    continue-on-error: true
  
  # Always run cleanup steps
  - name: Cleanup
    if: always()
    run: ./cleanup.sh
```

**Debug Logging**: Set repository secrets:
- `ACTIONS_RUNNER_DEBUG: true` - Runner diagnostic logging
- `ACTIONS_STEP_DEBUG: true` - Step debug logging

## Common Patterns

### Multi-Environment Deployment

```yaml
jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    steps:
    - run: ./deploy-staging.sh
  
  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
    - run: ./deploy-production.sh
```

### Monorepo with Path Filters

```yaml
on:
  push:
    paths:
      - 'services/api/**'

jobs:
  build-api:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: |
        cd services/api
        npm ci
        npm run build
```

### Artifact Sharing Between Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - run: npm run build
    - uses: actions/upload-artifact@v4
      with:
        name: dist
        path: dist/
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/download-artifact@v4
      with:
        name: dist
    - run: ./deploy.sh
```

## Critical Notes

�� **YAML Indentation**: Use 2 spaces (not tabs). Incorrect indentation causes syntax errors.

💡 **Expression Syntax**: Use `${{ }}` for expressions, `$()` for environment variables in shell.

⚠️ **Context Availability**: Not all contexts are available in all places. Check [documentation](https://docs.github.com/actions/learn-github-actions/contexts).

📊 **Job Outputs**: Use `outputs:` to pass data between jobs. Limited to strings only.

🔄 **Workflow Re-run**: Workflows can be re-run from the Actions tab - useful for transient failures.

✨ **Secrets in Expressions**: Secrets are redacted in logs. Use `${{ secrets.NAME }}` syntax.

## Quick Reference

### Workflow File Structure

```yaml
name: Workflow Name
on: [events]
env: {variables}
permissions: {permissions}

jobs:
  job-id:
    name: Job Name
    runs-on: runner
    needs: [dependencies]
    if: condition
    env: {job-variables}
    strategy: {matrix}
    steps:
    - name: Step Name
      uses: action@version
      with: {inputs}
    - run: command
      env: {step-variables}
```

### Context Variables

| Context | Example | Description |
|---------|---------|-------------|
| `github` | `${{ github.repository }}` | GitHub event info |
| `env` | `${{ env.NODE_VERSION }}` | Environment variables |
| `job` | `${{ job.status }}` | Current job info |
| `steps` | `${{ steps.id.outputs.value }}` | Step outputs |
| `runner` | `${{ runner.os }}` | Runner environment |
| `secrets` | `${{ secrets.TOKEN }}` | Repository secrets |

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/5-describe-standard-workflow-syntax-elements)
