# Optimize Pipelines for Cost, Time, Performance, and Reliability

## Key Concepts
- **Multi-dimensional optimization**: Balance cost, time, performance, and reliability (trade-offs inevitable)
- **Data-driven approach**: Measure current state → Identify bottlenecks → Implement improvements → Validate results
- **Cost optimization**: Reduce agent minutes, eliminate waste, efficient caching
- **Time optimization**: Parallelization, dependency optimization, build improvements
- **Performance optimization**: Resource efficiency, task optimization, test speed
- **Reliability optimization**: Retry logic, stability patterns, quality gates

## Optimization Philosophy

**Trade-Offs**:
```yaml
Cost vs. Speed:
  - Faster agents cost more
  - Parallel jobs increase cost but reduce time

Speed vs. Reliability:
  - Shortcuts may speed pipelines but introduce failures

Performance vs. Cost:
  - High-performance infrastructure costs more

Strategy:
  - Don't optimize blindly
  - Measure → Identify → Implement → Validate
```

## Cost Optimization Strategies

### Reduce Agent Minutes

| Strategy | Implementation | Savings |
|----------|----------------|---------|
| **Self-Hosted Agents** | For high-volume pipelines, cheaper than Microsoft-hosted | Up to 70% at scale |
| **Right-Size Agents** | Match agent specs to workload | Avoid paying for unused capacity |
| **Scheduled Builds** | Run non-urgent builds off-peak | Use cheaper agents during low demand |

### Eliminate Waste

**Conditional Execution**:
```yaml
# Skip unnecessary steps
- task: PublishBuildArtifacts@1
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  # Only publish on main branch
```

**Branch Policies**:
```yaml
Full Build Triggers:
  - Main branch: Full build + all tests
  - Feature branches: Fast build + unit tests only
  - PR validation: Incremental builds

Cost Reduction:
  - Avoid expensive builds on every commit
  - Limit full builds to critical branches
```

**Incremental Builds**:
```yaml
# Only build changed components
- script: |
    git diff --name-only HEAD~1 HEAD > changed_files.txt
    if grep -q "^frontend/" changed_files.txt; then
      echo "Building frontend..."
      cd frontend && npm run build
    fi
```

### Efficient Caching

| Cache Type | Implementation | Benefit |
|-----------|----------------|---------|
| **Package Caching** | Cache NuGet, npm, Maven packages | Avoid re-downloads (save 2-5 min) |
| **Build Output Caching** | Cache compilation outputs | Incremental builds (save 5-10 min) |
| **Docker Layer Caching** | Reuse unchanged Docker layers | Faster container builds (save 3-7 min) |

**Example: npm Package Caching**:
```yaml
- task: Cache@2
  inputs:
    key: 'npm | "$(Agent.OS)" | package-lock.json'
    path: $(Pipeline.Workspace)/.npm
    restoreKeys: |
      npm | "$(Agent.OS)"
  displayName: 'Cache npm packages'

- script: npm ci --cache $(Pipeline.Workspace)/.npm
```

## Time Optimization Strategies

### Parallelization

**Parallel Jobs**:
```yaml
jobs:
  - job: Build
    # Runs independently

  - job: UnitTests
    dependsOn: Build
    # Runs after Build

  - job: IntegrationTests
    dependsOn: Build
    # Runs parallel to UnitTests

  - job: Deploy
    dependsOn: [UnitTests, IntegrationTests]
    # Waits for both tests
```

**Parallel Tests**:
```yaml
# Distribute tests across agents
strategy:
  matrix:
    Agent1:
      TestSlice: '1'
    Agent2:
      TestSlice: '2'
    Agent3:
      TestSlice: '3'

- script: pytest --test-slice=$(TestSlice)/3
```

**Matrix Builds**:
```yaml
# Test multiple configurations in parallel
strategy:
  matrix:
    Windows_Python38:
      vmImage: 'windows-latest'
      python.version: '3.8'
    Linux_Python39:
      vmImage: 'ubuntu-latest'
      python.version: '3.9'
    macOS_Python310:
      vmImage: 'macOS-latest'
      python.version: '3.10'
```

### Dependency Optimization

| Optimization | Command | Time Saved |
|--------------|---------|------------|
| **Shallow Clone** | `git clone --depth 1` | 50-80% faster checkout |
| **Selective Restore** | Restore only required packages | 30-50% faster restore |
| **Pre-Warmed Agents** | Keep agents with common dependencies | Eliminate setup time |

**Shallow Clone**:
```yaml
- checkout: self
  fetchDepth: 1  # Only fetch latest commit
  fetchTags: false
```

### Build Improvements

```yaml
Incremental Compilation:
  - Only rebuild changed code
  - Example (C#): <IncrementalBuild>true</IncrementalBuild>

Distributed Builds:
  - Use IncrediBuild, Incredibuild for C++
  - Distribute compilation across multiple machines

Remove Bottlenecks:
  - Profile build tasks
  - Identify slowest tasks
  - Optimize or parallelize
```

## Performance Optimization Strategies

### Resource Efficiency

| Resource | Strategy | Impact |
|----------|----------|--------|
| **VM Sizing** | Match agent capacity to workload | Avoid over/under-provisioning |
| **Memory** | Reduce memory consumption in builds/tests | Prevent OOM failures |
| **Disk I/O** | Use SSDs, optimize file operations | Faster read/write operations |

**Appropriate VM Sizes**:
```yaml
Small Workloads (2 vCPU, 4 GB):
  - API builds
  - Simple unit tests

Medium Workloads (4 vCPU, 16 GB):
  - Full-stack builds
  - Integration tests

Large Workloads (8+ vCPU, 32+ GB):
  - Mobile app builds
  - Large codebases
  - Docker builds
```

### Task Optimization

```yaml
Efficient Scripts:
  - Optimize custom scripts
  - Remove unnecessary commands
  - Use native tools (e.g., 'az' CLI over PowerShell wrappers)

Minimal Artifacts:
  - Publish only necessary artifacts
  - Smaller artifacts = faster upload/download
  - Use artifact filters

Compression:
  - Compress artifacts before publishing
  - Reduce transfer time
```

### Test Optimization

| Strategy | Implementation | Benefit |
|----------|----------------|---------|
| **Test Selection** | Run only affected tests for PRs | 70-90% faster for PRs |
| **Fast Unit Tests** | Optimize slow unit tests | Faster feedback |
| **Parallel Execution** | Distribute tests across agents | N× speedup (N = # agents) |

**Example: Affected Tests Only**:
```yaml
- script: |
    # Identify changed files
    git diff --name-only origin/main...HEAD > changed.txt
    
    # Run tests for changed modules only
    pytest $(cat changed.txt | xargs)
  condition: ne(variables['Build.SourceBranch'], 'refs/heads/main')
```

## Reliability Optimization Strategies

### Retry Logic

```yaml
# Retry tasks that fail due to transient issues
- task: AzureCLI@2
  inputs:
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: 'az storage blob upload ...'
  retryCountOnTaskFailure: 3  # Retry up to 3 times
```

**Exponential Backoff**:
```yaml
- script: |
    for i in 1 2 3; do
      npm install && break
      echo "Retry $i failed, waiting..."
      sleep $((i * 10))  # 10s, 20s, 30s
    done
```

### Stability Patterns

| Pattern | Implementation | Benefit |
|---------|----------------|---------|
| **Timeout Configuration** | Set appropriate timeouts | Prevent hanging builds |
| **Health Checks** | Verify services before dependent tasks | Fail fast on unavailable dependencies |
| **Graceful Degradation** | Allow non-critical tasks to fail | Pipeline succeeds despite minor issues |

**Example: Health Check**:
```yaml
- script: |
    # Wait for service to be ready
    timeout 60 bash -c 'until curl -f http://localhost:8080/health; do sleep 2; done'
  displayName: 'Wait for service health'

- task: RunTests@1
  # Only runs if health check succeeds
```

### Quality Gates

```yaml
Code Coverage Threshold:
  - task: PublishCodeCoverageResults@1
    inputs:
      failOnCoverageDecrease: true
      minimumCoverage: 80

Security Scanning:
  - task: SecurityScan@1
    inputs:
      failOnCriticalVulnerabilities: true

Performance Baselines:
  - script: |
      # Fail if response time > baseline
      if [ $RESPONSE_TIME -gt 500 ]; then
        echo "Performance regression detected"
        exit 1
      fi
```

## Dashboard Metrics for Optimization

### Cost Dashboard
```yaml
Metrics:
  - Total agent minutes: Monthly consumption
  - Cost per pipeline: Identify most expensive
  - Cost trends: Month-over-month changes
  - Agent utilization: % of time agents busy

Visualizations:
  - Line chart: Agent minutes over time
  - Bar chart: Cost per pipeline
  - Gauge: Current month spend vs. budget
```

### Time Dashboard
```yaml
Metrics:
  - Duration percentiles: P50, P90, P95
  - Task breakdown: Time per task
  - Queue time: Agent availability
  - Trend: Duration week-over-week

Visualizations:
  - Line chart: P90 duration trend
  - Waterfall: Task duration breakdown
  - Gauge: P90 vs. target (e.g., < 10 min)
```

### Performance Dashboard
```yaml
Metrics:
  - Resource utilization: CPU, memory, disk
  - Throughput: Builds/deployments per day
  - Efficiency: Success rate × speed

Visualizations:
  - Line chart: Throughput over time
  - Heatmap: Resource utilization by agent
```

### Reliability Dashboard
```yaml
Metrics:
  - Failure rate: % of failed runs
  - MTTR: Mean time to recovery
  - Flaky test count: Number of unstable tests

Visualizations:
  - Line chart: Failure rate trend
  - Bar chart: Failure rate by pipeline
  - Table: Top flaky tests
```

## Critical Notes
- ⚠️ **Measure before optimizing**: Identify actual bottlenecks, not assumptions
- 💡 **Trade-offs exist**: Balance cost, time, performance, reliability
- 🎯 **Iterative approach**: Make small changes, measure impact, iterate
- 📊 **Dashboard-driven**: Use dashboards to track optimization impact

## Quick Commands

**Profile Pipeline Duration**:
```bash
# Analyze task durations
az pipelines runs show \
  --id <run-id> \
  --query "timeline.records[].{name:name, duration:duration}" \
  --output table
```

**Identify Expensive Pipelines**:
```kql
// Log Analytics query
PipelineRuns
| where CompletedDate > ago(30d)
| summarize 
    TotalRuns = count(),
    TotalMinutes = sum(DurationMinutes)
    by PipelineName
| extend CostPerRun = TotalMinutes / TotalRuns
| order by TotalMinutes desc
```

**Compare Before/After Optimization**:
```yaml
Baseline (Before):
  - P50 duration: 12 min
  - P90 duration: 18 min
  - Agent minutes/month: 10,000

After Optimization:
  - P50 duration: 8 min (33% improvement)
  - P90 duration: 12 min (33% improvement)
  - Agent minutes/month: 7,500 (25% cost reduction)
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/8-optimize-pipeline-cost-time-performance-reliability)
