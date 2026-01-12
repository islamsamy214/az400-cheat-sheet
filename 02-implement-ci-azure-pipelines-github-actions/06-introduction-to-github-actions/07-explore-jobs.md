# Explore Jobs

Jobs are the building blocks of GitHub Actions workflows. Each job is a collection of steps that run sequentially on the same runner, sharing the filesystem and environment variables.

## Understanding Job Execution

### Key Characteristics of Jobs

✅ **Sequential Steps**: Steps within a job run one after another in order

✅ **Shared Environment**: All steps share the same runner and filesystem

✅ **Isolated Execution**: Each job gets a fresh virtual environment

✅ **Searchable Logs**: Job outputs are automatically captured and searchable

✅ **Artifact Support**: Jobs can save and share files between workflow runs

### Basic Job Structure

```yaml
jobs:
  build:
    name: Build Application  # Display name in UI
    runs-on: ubuntu-latest  # Runner type
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

## Parallel vs. Sequential Execution

### Parallel Execution (Default)

By default, multiple jobs run simultaneously to minimize workflow duration.

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit

  # All three jobs (lint, test, security) run simultaneously
```

**Execution Time**: If each job takes 2 minutes, total time = 2 minutes (not 6)

### Sequential Execution with Dependencies

Use the `needs` keyword to create job dependencies.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  test:
    needs: build  # Wait for build to complete
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: dist
      - run: npm test

  deploy:
    needs: test  # Wait for test to complete
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
      - run: ./deploy.sh
```

**Execution Flow**: build → test → deploy (sequential)

**Execution Time**: If each takes 2 minutes, total time = 6 minutes

### Multiple Dependencies

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - run: npm run build

  unit-test:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - run: npm run test:unit

  integration-test:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - run: npm run test:integration

  deploy:
    needs: [unit-test, integration-test]  # Wait for BOTH tests
    runs-on: ubuntu-latest
    steps:
    - run: ./deploy.sh
```

**Execution Flow**: build → (unit-test + integration-test) → deploy

## Advanced Job Patterns

### Matrix Strategy for Multiple Configurations

Run jobs across multiple environments simultaneously.

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
        # Creates 9 jobs: 3 OS × 3 Node versions
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      
      - run: npm ci
      - run: npm test
```

**Result**: Creates 9 parallel jobs testing all OS/Node combinations

### Matrix with Include/Exclude

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]
        include:
          # Add extra job: Ubuntu + Node 22
          - os: ubuntu-latest
            node-version: 22
            experimental: true
        exclude:
          # Remove job: Windows + Node 18
          - os: windows-latest
            node-version: 18
    
    runs-on: ${{ matrix.os }}
    continue-on-error: ${{ matrix.experimental || false }}
```

**Result**: Creates 4 jobs instead of default 4:
- ✅ ubuntu-latest + node 18
- ✅ ubuntu-latest + node 20
- ✅ ubuntu-latest + node 22 (experimental)
- ❌ windows-latest + node 18 (excluded)
- ✅ windows-latest + node 20

### Conditional Job Execution

Run jobs only when specific conditions are met.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - run: npm run build

  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
    - run: echo "Deploying to staging"

  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
    - run: echo "Deploying to production"

  notify-failure:
    needs: [build, deploy-production]
    if: failure()  # Only runs if previous jobs failed
    runs-on: ubuntu-latest
    steps:
    - run: echo "Deployment failed!"
```

**Conditional Functions**:
- `success()` - All previous jobs succeeded (default)
- `failure()` - Any previous job failed
- `always()` - Run regardless of status
- `cancelled()` - Workflow was cancelled

## Job Failure Handling

### Default Behavior

✅ If any step fails, the entire job fails

✅ Dependent jobs won't run if their prerequisites fail

✅ The workflow is marked as failed

### Controlling Failure Behavior

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # Step continues even on error
      - name: Run tests (non-blocking)
        run: npm test
        continue-on-error: true

      # Always upload test results, even on failure
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results.xml

      # Run only on failure
      - name: Debug on failure
        if: failure()
        run: cat logs/error.log
```

### Job-Level Continue on Error

```yaml
jobs:
  experimental-build:
    runs-on: ubuntu-latest
    continue-on-error: true  # Don't fail workflow if this job fails
    steps:
    - run: npm run experimental-build

  production-build:
    runs-on: ubuntu-latest
    steps:
    - run: npm run build  # This must succeed
```

## Job Outputs

Pass data from one job to another using outputs.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.set-version.outputs.version }}
      build-id: ${{ steps.set-build-id.outputs.id }}
    
    steps:
      - name: Set version
        id: set-version
        run: |
          VERSION=$(date +%Y%m%d)-${GITHUB_SHA::8}
          echo "version=$VERSION" >> $GITHUB_OUTPUT
      
      - name: Set build ID
        id: set-build-id
        run: |
          BUILD_ID=$(uuidgen)
          echo "id=$BUILD_ID" >> $GITHUB_OUTPUT

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: |
          echo "Deploying version: ${{ needs.build.outputs.version }}"
          echo "Build ID: ${{ needs.build.outputs.build-id }}"
```

## Job Configuration Options

### Complete Job Configuration

```yaml
jobs:
  my-job:
    name: My Job Display Name
    runs-on: ubuntu-latest
    needs: [previous-job]
    if: success()
    timeout-minutes: 30
    continue-on-error: false
    
    permissions:
      contents: read
      pull-requests: write
    
    env:
      JOB_VAR: value
    
    outputs:
      my-output: ${{ steps.step-id.outputs.value }}
    
    strategy:
      matrix:
        version: [18, 20, 22]
      fail-fast: false
    
    steps:
      # Steps here
```

### Job Configuration Reference

| Property | Purpose | Example |
|----------|---------|---------|
| `name` | Display name in UI | `name: Build Application` |
| `runs-on` | Runner type | `runs-on: ubuntu-latest` |
| `needs` | Job dependencies | `needs: [build, test]` |
| `if` | Conditional execution | `if: github.ref == 'refs/heads/main'` |
| `timeout-minutes` | Max execution time | `timeout-minutes: 30` |
| `continue-on-error` | Don't fail workflow | `continue-on-error: true` |
| `permissions` | GitHub token permissions | `permissions: { contents: read }` |
| `env` | Environment variables | `env: { NODE_VERSION: 20 }` |
| `outputs` | Job outputs | `outputs: { version: ... }` |
| `strategy` | Matrix configuration | `strategy: { matrix: { os: [...] } }` |

## Best Practices for Jobs

### 1. Keep Jobs Focused

✅ **Good**: Each job has a single responsibility
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
    - run: npm test

  build:
    runs-on: ubuntu-latest
    steps:
    - run: npm run build
```

❌ **Bad**: One job doing everything
```yaml
jobs:
  everything:
    runs-on: ubuntu-latest
    steps:
    - run: npm run lint
    - run: npm test
    - run: npm run build
    - run: ./deploy.sh
```

### 2. Use Descriptive Names

✅ **Good**: Clear purpose
```yaml
jobs:
  build-frontend:
    name: Build Frontend Application
  
  test-api:
    name: Run API Integration Tests
```

❌ **Bad**: Generic names
```yaml
jobs:
  job1:
    name: Job 1
  
  test:
    name: Test
```

### 3. Optimize Dependencies

Only create dependencies when truly necessary:

```yaml
jobs:
  # These can run in parallel (no dependencies)
  lint:
    runs-on: ubuntu-latest
  
  unit-test:
    runs-on: ubuntu-latest
  
  # Deploy needs both to complete
  deploy:
    needs: [lint, unit-test]
    runs-on: ubuntu-latest
```

### 4. Choose Appropriate Runners

```yaml
jobs:
  # Linux for most tasks (fastest, cheapest)
  build:
    runs-on: ubuntu-latest
  
  # macOS for iOS/macOS builds
  build-ios:
    runs-on: macos-latest
  
  # Windows for .NET Framework
  build-windows:
    runs-on: windows-latest
  
  # Self-hosted for special requirements
  deploy:
    runs-on: [self-hosted, linux, gpu]
```

### 5. Handle Failures Gracefully

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - run: npm test
    
    # Always upload logs, even on failure
    - if: always()
      uses: actions/upload-artifact@v4
      with:
        name: test-logs
        path: logs/

  notify:
    needs: test
    if: failure()  # Only notify on failure
    runs-on: ubuntu-latest
    steps:
    - run: echo "Tests failed!"
```

### 6. Share Data Efficiently

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - run: npm run build
    
    # Upload artifacts once
    - uses: actions/upload-artifact@v4
      with:
        name: dist
        path: dist/

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
    # Download pre-built artifacts
    - uses: actions/download-artifact@v4
      with:
        name: dist
    - run: npm test

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    # Reuse same artifacts
    - uses: actions/download-artifact@v4
      with:
        name: dist
    - run: ./deploy.sh
```

## Critical Notes

🎯 **Parallel by Default**: Jobs run in parallel unless you specify `needs`. Use this to save time.

💡 **Matrix Explosion**: Be careful with matrix strategies - `3 OS × 4 Node versions × 2 databases = 24 jobs` can consume your concurrent job limit quickly.

⚠️ **Timeout Defaults**: Jobs have a default 360-minute (6-hour) timeout. Set lower values for faster failure detection.

📊 **Concurrent Job Limits**: Free accounts: 20 concurrent jobs, Team: 60, Enterprise: 180. Plan accordingly.

🔄 **Job Outputs**: Limited to strings only. For complex data, use artifacts instead.

✨ **Artifact Sharing**: Build once, test/deploy multiple times. Upload artifacts from build job, download in dependent jobs.

## Quick Reference

### Job Dependency Patterns

```yaml
# Independent (parallel)
jobs:
  job1: ...
  job2: ...

# Sequential
jobs:
  job1: ...
  job2:
    needs: job1

# Fan-out then fan-in
jobs:
  build: ...
  test1:
    needs: build
  test2:
    needs: build
  deploy:
    needs: [test1, test2]
```

### Conditional Patterns

```yaml
# Run on success (default)
if: success()

# Run on failure
if: failure()

# Always run
if: always()

# Run on main branch only
if: github.ref == 'refs/heads/main'

# Run on push events only
if: github.event_name == 'push'
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/7-explore-jobs)
