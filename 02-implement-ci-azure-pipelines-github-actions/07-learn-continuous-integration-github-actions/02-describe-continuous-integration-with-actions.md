# Describe Continuous Integration with Actions

Continuous Integration (CI) with GitHub Actions transforms how teams collaborate by automating build, test, and validation processes every time code changes. This catches issues early, improves quality, and accelerates delivery.

## Understanding CI with GitHub Actions

Continuous Integration is a development practice where team members integrate code changes **frequently**—ideally multiple times per day. Each integration triggers an automated build and test process that validates changes and provides immediate feedback.

### Key Benefits of CI with GitHub Actions

| Benefit | Description | Impact |
|---------|-------------|--------|
| **Immediate Feedback** | Tests run on every push/PR within minutes | Developers know instantly if changes break functionality |
| **Consistent Build Environment** | Every build runs in clean, reproducible runners | Eliminates "works on my machine" problems |
| **Automated Quality Checks** | Linting, scanning, testing happen automatically | Maintains standards without manual intervention |
| **Early Issue Detection** | Problems caught when they're easier to fix | Reduces cost and complexity of bug fixes |
| **Parallel Execution** | Multiple jobs run simultaneously | Faster feedback loop for developers |
| **Team Visibility** | Workflow status visible to entire team | Better collaboration and transparency |

### CI Principles

✅ **Integrate Frequently**: Commit small changes multiple times per day  
✅ **Automate the Build**: Every commit triggers automated build  
✅ **Test in Production-Like Environment**: Use containers or clean VMs  
✅ **Keep Build Fast**: Optimize for sub-10-minute feedback  
✅ **Test Automatically**: Unit, integration, and E2E tests in pipeline  
✅ **Everyone Sees Results**: Make build status visible to team  
✅ **Fix Broken Builds Immediately**: Treat failed builds as top priority

## Modern CI Workflow Example

Here's a comprehensive CI workflow demonstrating best practices:

\`\`\`yaml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DOTNET_VERSION: "8.0.x"
  NODE_VERSION: "20"

jobs:
  test:
    name: Test and Lint
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for better analysis

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: ~/.nuget/packages
          key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}

      - name: Restore dependencies
        run: dotnet restore

      - name: Build project
        run: dotnet build --no-restore --configuration Release

      - name: Run unit tests
        run: dotnet test --no-build --configuration Release --collect:"XPlat Code Coverage"

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.xml
          fail_ci_if_error: true

  security-scan:
    name: Security Analysis
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run security scan
        uses: github/codeql-action/init@v3
        with:
          languages: csharp

      - name: Build for analysis
        run: dotnet build --configuration Release

      - name: Perform CodeQL analysis
        uses: github/codeql-action/analyze@v3

  build-artifacts:
    name: Build Release Artifacts
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Build release package
        run: dotnet publish --configuration Release --output ./release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: release-package
          path: ./release
          retention-days: 30
\`\`\`

## Workflow Breakdown

### Event Triggers

\`\`\`yaml
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
\`\`\`

**Purpose**:
- **Push events**: Trigger CI on commits to main and develop branches
- **Pull request events**: Validate changes before they're merged
- **Branch filtering**: Only run for relevant branches, saving compute resources

### Environment Configuration

\`\`\`yaml
env:
  DOTNET_VERSION: "8.0.x"
  NODE_VERSION: "20"
\`\`\`

**Benefits**:
- **Centralized version management** for consistency across jobs
- **Easy updates**: Change version in one place
- **Reusability**: Same values accessible in all jobs

### Job Orchestration

The workflow demonstrates three job patterns:

#### 1. Parallel Execution (test + security-scan)

\`\`\`yaml
jobs:
  test:
    runs-on: ubuntu-latest
    # Steps...

  security-scan:
    runs-on: ubuntu-latest
    # Steps...
\`\`\`

**Execution**: Both jobs run **simultaneously** (parallel)  
**Time saved**: If each takes 5 minutes, total time = 5 minutes (not 10)  
**Use case**: Independent tasks that don't depend on each other

#### 2. Sequential Execution (build-artifacts after test + security-scan)

\`\`\`yaml
build-artifacts:
  needs: [test, security-scan]  # Wait for both to complete
  runs-on: ubuntu-latest
\`\`\`

**Execution**: Runs only after **both** test and security-scan succeed  
**Purpose**: Ensures artifacts are only built from validated code  
**Pattern**: Fan-in (multiple jobs → single dependent job)

#### 3. Conditional Execution

\`\`\`yaml
if: github.ref == 'refs/heads/main'
\`\`\`

**Purpose**: Only build artifacts from main branch  
**Cost optimization**: Saves compute minutes on feature branches  
**Pattern**: Environment-specific logic

### Best Practices Implemented

#### Security and Reliability

✅ **Pinned action versions** (`@v4`): Ensures reproducibility and prevents breaking changes  
✅ **Security scanning** with CodeQL: Catches vulnerabilities before merge  
✅ **Code coverage validation**: Fails CI on insufficient coverage  
✅ **Full checkout** (`fetch-depth: 0`): Enables better diff analysis

#### Performance Optimization

✅ **Dependency caching**: Reduces build times by caching NuGet packages  
✅ **Parallel job execution**: Minimizes total workflow duration  
✅ **Selective artifact building**: Only creates artifacts when needed  
✅ **Build once, use many**: Build in test job, reuse in later jobs

#### Developer Experience

✅ **Clear job names**: "Test and Lint" vs generic "job1"  
✅ **Descriptive step names**: Easy to identify what failed  
✅ **Test coverage reporting**: Visible quality metrics  
✅ **Fast feedback on PRs**: Results within minutes of pushing

## CI Pipeline Evolution

As projects grow, extend this foundation with:

### Multi-Environment Testing

\`\`\`yaml
strategy:
  matrix:
    dotnet-version: ['6.0.x', '7.0.x', '8.0.x']
    os: [ubuntu-latest, windows-latest, macos-latest]
runs-on: ${{ matrix.os }}
\`\`\`

**Result**: Tests across 9 combinations (3 versions × 3 OS)

### Integration Testing

\`\`\`yaml
services:
  postgres:
    image: postgres:15
    env:
      POSTGRES_PASSWORD: postgres
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
\`\`\`

**Purpose**: Test with real databases, caches, message queues

### Performance Testing

\`\`\`yaml
- name: Run load tests
  run: |
    artillery run load-test.yml
    artillery report results.json --output perf-report.html
\`\`\`

**Monitors**: Response times, throughput, error rates

### Deployment Automation (CI/CD)

\`\`\`yaml
deploy-production:
  needs: [test, security-scan, build-artifacts]
  if: github.ref == 'refs/heads/main'
  runs-on: ubuntu-latest
  environment: production
  steps:
  - uses: actions/download-artifact@v4
  - name: Deploy to Azure
    run: az webapp deploy --resource-group rg --name app --src-path ./release
\`\`\`

**Progression**: CI (this module) → CD (deployment automation)

## CI Workflow Patterns

### Pattern 1: Basic CI (Lint → Test → Build)

\`\`\`yaml
jobs:
  lint:
    steps:
    - run: npm run lint
  
  test:
    needs: lint
    steps:
    - run: npm test
  
  build:
    needs: test
    steps:
    - run: npm run build
\`\`\`

**Execution**: Sequential (lint → test → build)  
**When to use**: Simple projects with quick steps

### Pattern 2: Parallel Quality Gates

\`\`\`yaml
jobs:
  unit-test:
    steps:
    - run: npm run test:unit
  
  integration-test:
    steps:
    - run: npm run test:integration
  
  security-scan:
    steps:
    - run: npm audit
  
  lint:
    steps:
    - run: npm run lint
\`\`\`

**Execution**: All 4 jobs run in parallel  
**When to use**: Independent checks, need fast feedback

### Pattern 3: Matrix Testing

\`\`\`yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    os: [ubuntu-latest, windows-latest]
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node-version }}
\`\`\`

**Execution**: 6 jobs (3 versions × 2 OS) in parallel  
**When to use**: Cross-platform/cross-version compatibility

### Pattern 4: Fan-Out, Fan-In

\`\`\`yaml
jobs:
  build:
    # Single build job
  
  test-unit:
    needs: build  # Multiple jobs depend on build
  test-integration:
    needs: build
  test-e2e:
    needs: build
  
  deploy:
    needs: [test-unit, test-integration, test-e2e]  # Single job depends on multiple
\`\`\`

**Execution**: build → (3 test jobs parallel) → deploy  
**When to use**: Build once, test many ways, deploy after all pass

## Measuring CI Success

### Key Metrics

| Metric | Target | Purpose |
|--------|--------|---------|
| **Build Time** | < 10 minutes | Fast feedback loop |
| **Test Coverage** | > 80% | Code quality assurance |
| **Build Success Rate** | > 95% | Pipeline stability |
| **Mean Time to Recovery** | < 1 hour | How fast broken builds are fixed |
| **Deployment Frequency** | Multiple/day | Team velocity |

### Monitoring Workflow Health

\`\`\`yaml
- name: Report workflow metrics
  run: |
    echo "Build duration: ${{ steps.build.outputs.duration }}"
    echo "Test coverage: ${{ steps.coverage.outputs.percentage }}%"
    echo "Workflow status: ${{ job.status }}"
\`\`\`

## Critical Notes

�� **Start Simple**: Begin with basic lint → test → build, then add complexity.

💡 **Fail Fast**: Run fastest checks first (linting) to catch easy issues early.

⚠️ **Keep Builds Fast**: If CI takes > 10 minutes, developers stop trusting it.

📊 **Monitor Trends**: Track build times, failure rates, coverage over time.

🔄 **Iterate Continuously**: Review and improve CI pipeline regularly.

✨ **Make Visible**: Use badges, notifications, and dashboards to share status.

## Quick Reference

### CI Checklist

- ☑️ Runs on every push and pull request
- ☑️ Tests pass consistently
- ☑️ Builds complete in < 10 minutes
- ☑️ Code coverage tracked and enforced
- ☑️ Security scanning enabled
- ☑️ Artifacts preserved for deployment
- ☑️ Team notified of failures
- ☑️ Broken builds fixed immediately

### Common CI Commands

\`\`\`yaml
# Checkout code
- uses: actions/checkout@v4

# Setup language runtime
- uses: actions/setup-node@v4
  with:
    node-version: "20"

# Cache dependencies
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Run tests
- run: npm test

# Upload artifacts
- uses: actions/upload-artifact@v4
  with:
    name: coverage
    path: coverage/
\`\`\`

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/2-describe-continuous-integration-with-actions)
