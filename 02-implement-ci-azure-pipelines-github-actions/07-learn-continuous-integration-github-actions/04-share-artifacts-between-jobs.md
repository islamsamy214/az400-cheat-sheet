# Share Artifacts Between Jobs

Artifacts enable you to share data between jobs in a workflow and preserve important files after a workflow completes. They're essential for building robust CI/CD pipelines that can pass build outputs, test results, and other files between different stages of your automation.

## Understanding Artifacts in GitHub Actions

Artifacts are files or collections of files that you want to share between jobs or save from a workflow run.

### Common Use Cases

| Use Case | Example | Benefit |
|----------|---------|---------|
| **Build outputs** | Compiled apps, packages, dist files | Build once, deploy/test many times |
| **Test results** | Coverage reports, test logs | Share results across jobs, preserve for analysis |
| **Documentation** | Generated API docs, reports | Review documentation without rebuilding |
| **Debug information** | Log files, crash dumps | Troubleshoot failures after workflow completes |

### Artifact Lifecycle and Scope

**Storage duration**: 
- Default: 90 days (public repos), 400 days (private repos)
- Configurable: 1-400 days in Enterprise

**Accessibility**:
- Available to all jobs in the same workflow run
- Downloadable via GitHub UI, REST API, or CLI
- Automatically deleted after retention period

**Storage limits**:
- Free: 500 MB
- Pro: 1 GB
- Team: 2 GB
- Enterprise: 50 GB

## Uploading Artifacts

### Basic File Upload

```yaml
steps:
  - name: Build application
    run: npm run build

  - name: Upload build artifacts
    uses: actions/upload-artifact@v4
    with:
      name: build-output
      path: dist/
```

### Advanced Upload Patterns

**Multiple specific files**:

```yaml
- name: Upload test results and logs
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      test-results.xml
      coverage/lcov.info
      logs/test.log
    retention-days: 30
```

**Wildcard patterns**:

```yaml
- name: Upload all logs
  uses: actions/upload-artifact@v4
  with:
    name: application-logs
    path: |
      logs/**/*.log
      reports/**/coverage.xml
      build/output/**/*.map
```

**Conditional upload with error handling**:

```yaml
- name: Upload artifacts even on failure
  uses: actions/upload-artifact@v4
  if: always()  # Upload even if previous steps failed
  with:
    name: build-artifacts-${{ github.run_number }}
    path: |
      dist/
      !dist/**/*.map  # Exclude source maps
    retention-days: 7
    if-no-files-found: warn  # Don't fail if no files found
```

### Upload Options Reference

| Option | Description | Default |
|--------|-------------|---------|
| `name` | Artifact name (must be unique per workflow run) | Required |
| `path` | Files to upload (supports wildcards and exclusions) | Required |
| `retention-days` | How long to keep artifact (1-400 days) | 90 (public) / 400 (private) |
| `if-no-files-found` | Behavior when no files match: `error`, `warn`, `ignore` | `warn` |

## Downloading Artifacts

### Basic Artifact Download

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build application
        run: npm run build
      
      - name: Upload build
        uses: actions/upload-artifact@v4
        with:
          name: app-build
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: app-build
          path: ./deployment

      - name: Deploy application
        run: |
          ls -la ./deployment
          # Deploy files from ./deployment directory
```

### Advanced Download Scenarios

**Download all artifacts**:

```yaml
- name: Download all workflow artifacts
  uses: actions/download-artifact@v4
  # Downloads all artifacts to current directory
```

**Multiple artifact downloads with patterns**:

```yaml
- name: Download build and test artifacts
  uses: actions/download-artifact@v4
  with:
    pattern: build-*  # Downloads all artifacts matching pattern
    path: ./artifacts
    merge-multiple: true  # Merge into single directory
```

## Complete Workflow Example

Here's a comprehensive example showing artifact usage across multiple jobs:

```yaml
name: Build, Test, and Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Build Application
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: |
          npm run build
          npm run build:docs

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ github.sha }}
          path: |
            dist/
            docs/build/
          retention-days: 30

      - name: Upload source maps separately
        uses: actions/upload-artifact@v4
        with:
          name: sourcemaps-${{ github.sha }}
          path: dist/**/*.map
          retention-days: 7

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-${{ github.sha }}
          path: ./build-output

      - name: Run tests
        run: |
          npm test -- --coverage
          npm run test:e2e ./build-output/dist

      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results-${{ github.sha }}
          path: |
            coverage/
            test-results.xml
            screenshots/
          retention-days: 14

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-${{ github.sha }}
          path: ./scan-target

      - name: Run security scan
        run: npm audit --json > audit-results.json

      - name: Upload security reports
        uses: actions/upload-artifact@v4
        with:
          name: security-report-${{ github.sha }}
          path: |
            audit-results.json
            security-scan-report.html
          retention-days: 90

  deploy:
    name: Deploy Application
    runs-on: ubuntu-latest
    needs: [build, test, security-scan]
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-${{ github.sha }}
          path: ./deploy

      - name: Download test results
        uses: actions/download-artifact@v4
        with:
          name: test-results-${{ github.sha }}
          path: ./reports

      - name: Deploy to production
        run: |
          echo "Deploying files from ./deploy directory"
          ls -la ./deploy
          # Actual deployment commands here

      - name: Upload deployment logs
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: deployment-logs-${{ github.sha }}
          path: deployment.log
          retention-days: 30
```

## Best Practices for Artifact Management

### 1. Naming Conventions

```yaml
# ✅ Good: Descriptive, unique names with context
name: build-${{ matrix.os }}-${{ github.sha }}
name: test-results-unit-${{ github.run_number }}
name: security-report-${{ github.ref_name }}

# ❌ Bad: Generic names that may conflict
name: build
name: output
name: results
```

### 2. Retention Optimization

```yaml
# Match retention to artifact importance
- uses: actions/upload-artifact@v4
  with:
    name: critical-logs
    path: logs/
    retention-days: 90  # Long retention for important data

- uses: actions/upload-artifact@v4
  with:
    name: temp-build-cache
    path: .cache/
    retention-days: 1  # Short retention for temporary files
```

### 3. Security Considerations

```yaml
# Exclude sensitive files from artifacts
- uses: actions/upload-artifact@v4
  with:
    name: safe-build-output
    path: |
      dist/
      !dist/**/*.env      # Exclude environment files
      !dist/**/*secret*   # Exclude files with 'secret' in name
      !**/.env*           # Exclude all .env files
```

### 4. Performance Optimization

```yaml
# Compress large artifacts
- name: Compress large artifacts
  run: tar -czf logs.tar.gz logs/

- uses: actions/upload-artifact@v4
  with:
    name: compressed-logs
    path: logs.tar.gz
```

### 5. Error Handling

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
    if-no-files-found: error  # Fail if no files found
    # Options: error, warn, ignore
```

## Artifact Management Patterns

### Pattern 1: Build Once, Test Many Times

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

  test-unit:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/download-artifact@v4
      with:
        name: dist
    - run: npm run test:unit

  test-integration:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/download-artifact@v4
      with:
        name: dist
    - run: npm run test:integration

  test-e2e:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/download-artifact@v4
      with:
        name: dist
    - run: npm run test:e2e
```

### Pattern 2: Aggregate Test Results

```yaml
jobs:
  test-unit:
    steps:
    - run: npm run test:unit
    - uses: actions/upload-artifact@v4
      with:
        name: test-unit-results
        path: test-results/

  test-integration:
    steps:
    - run: npm run test:integration
    - uses: actions/upload-artifact@v4
      with:
        name: test-integration-results
        path: test-results/

  report:
    needs: [test-unit, test-integration]
    steps:
    - uses: actions/download-artifact@v4
      with:
        pattern: test-*-results
        path: all-results
        merge-multiple: true
    
    - name: Generate combined report
      run: ./generate-report.sh all-results/
```

## Critical Notes

🎯 **Build Once Principle**: Build artifacts once, reuse in multiple jobs to save time and ensure consistency.

💡 **Unique Names**: Use `${{ github.sha }}` or `${{ github.run_number }}` in artifact names to avoid conflicts.

⚠️ **Storage Costs**: Monitor artifact storage usage—it counts against repository storage limits.

📊 **Retention Strategy**: Keep critical artifacts longer (90 days), temporary artifacts shorter (1-7 days).

🔄 **Cleanup**: Artifacts are automatically deleted after retention period expires.

✨ **Compression**: For large artifacts (>100 MB), compress before uploading to save storage and transfer time.

## Quick Reference

### Upload Artifact

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: path/to/files
    retention-days: 30
    if-no-files-found: warn
```

### Download Artifact

```yaml
- uses: actions/download-artifact@v4
  with:
    name: my-artifact
    path: ./download-location
```

### Common Patterns

```yaml
# Upload multiple paths
path: |
  dist/
  docs/
  !dist/**/*.map

# Download all artifacts
- uses: actions/download-artifact@v4

# Download with pattern
- uses: actions/download-artifact@v4
  with:
    pattern: build-*
    merge-multiple: true
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/4-share-artifacts-between-jobs)
