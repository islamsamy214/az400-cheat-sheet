# Optimize Pipeline Concurrency for Performance

## Key Concepts
- **Concurrency optimization** enables multiple jobs to run simultaneously, reducing total execution time
- **Parallel jobs** run independent jobs at the same time (build + unit tests + integration tests)
- **Cost-benefit analysis**: 3 parallel jobs at 10 min = same cost as 1 job at 30 min, but 3× faster feedback
- **Job dependencies** model execution order while maximizing parallel opportunities
- **Agent sizing** matches compute resources to workload requirements
- **Dynamic scaling** adjusts capacity based on demand patterns (5 agents overnight, 20 during business hours)

## Parallel Jobs and Stages

### Job-Level Parallelism

| Pattern | Description | Time Saved |
|---------|-------------|------------|
| **Independent Jobs** | Run non-dependent jobs simultaneously | N× faster (N = # parallel jobs) |
| **Matrix Builds** | Test multiple configs (OS, runtime) in parallel | Test all configs in one pass |
| **Test Parallelization** | Split test suite across agents | 10 agents = 10× faster |

**Example: Independent Jobs**:
```yaml
jobs:
  - job: Build
    # No dependencies—starts immediately

  - job: UnitTests
    dependsOn: Build
    # Waits for Build

  - job: IntegrationTests
    dependsOn: Build
    # Waits for Build (runs parallel to UnitTests)

  - job: Deploy
    dependsOn: [UnitTests, IntegrationTests]
    # Waits for both tests
```

**Example: Matrix Builds**:
```yaml
strategy:
  matrix:
    Windows_Python38:
      vmImage: 'windows-latest'
      python: '3.8'
    Linux_Python39:
      vmImage: 'ubuntu-latest'
      python: '3.9'
    macOS_Python310:
      vmImage: 'macOS-latest'
      python: '3.10'

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: $(python)
  - script: pytest
```

### Stage-Level Parallelism

**Multi-Region Deployments**:
```yaml
stages:
  - stage: DeployProduction
    jobs:
      - deployment: DeployEastUS
        environment: prod-eastus

      - deployment: DeployWestUS
        environment: prod-westus

      - deployment: DeployEurope
        environment: prod-europe

# All three deployments run in parallel
```

### Cost-Benefit Analysis

**Scenario 1: No Parallelism**:
```yaml
Duration: 30 minutes (serial)
Cost: 1 agent × 30 min = 30 agent-minutes
Feedback time: 30 minutes
```

**Scenario 2: 3 Parallel Jobs**:
```yaml
Duration: 10 minutes (simultaneous)
Cost: 3 agents × 10 min = 30 agent-minutes
Feedback time: 10 minutes
Value: 3× faster feedback, same cost ✅
```

**Finding the Balance**:
- ❌ **Too few parallel jobs**: Long wait times, frustrated developers
- ❌ **Too many parallel jobs**: Costs increase with diminishing returns
- ✅ **Sweet spot**: Enough parallelism for acceptable speed at reasonable cost

## Job Dependencies and Execution Order

### Dependency Modeling

**Critical Path**:
```yaml
# Identify longest dependency chain
Build (5 min)
 └─> IntegrationTests (8 min)
      └─> Deploy (3 min)

Total: 16 minutes (critical path)

# Optimize jobs on critical path first
```

**Conditional Execution**:
```yaml
# Skip expensive tests on draft PRs
- job: ExpensiveIntegrationTests
  condition: and(succeeded(), ne(variables['System.PullRequest.IsDraft'], 'true'))
```

**Branch-Specific Logic**:
```yaml
# Different concurrency for main vs. feature branches
- job: FullTestSuite
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')

- job: FastTestSuite
  condition: ne(variables['Build.SourceBranch'], 'refs/heads/main')
```

### Resource Contention Prevention

| Strategy | Implementation | Benefit |
|----------|----------------|---------|
| **Separate Agent Pools** | Compute-intensive vs. lightweight jobs | Prevent resource starvation |
| **Demands** | Ensure jobs get appropriate agents | Match job to agent capabilities |
| **Locks** | Prevent concurrent access to shared resources | Avoid conflicts (e.g., shared DB) |

**Example: Separate Pools**:
```yaml
jobs:
  - job: LightweightBuild
    pool:
      name: Default
      demands:
        - Agent.OS -equals Linux

  - job: HeavyMobileBuild
    pool:
      name: HighPerformance
      demands:
        - Agent.OS -equals macOS
        - Xcode
```

## Agent Sizing and Pool Management

### Agent Size Selection

**Microsoft-Hosted Agents**:
```yaml
Standard:
  Specs: 2 vCPU, 7 GB RAM
  Best for: Most workloads
  Cost: Pay per minute
  Pros: Zero maintenance, auto-scaling
  Cons: Limited customization, queue times
```

**Self-Hosted Agents**:

| Size | Specs | Best For | Cost Model |
|------|-------|----------|------------|
| **Small** | 2 vCPU, 4 GB | Lightweight builds | Low cost |
| **Medium** | 4 vCPU, 16 GB | Typical builds | Standard cost |
| **Large** | 8+ vCPU, 32+ GB | Compute-intensive | High cost, cheaper at scale |

**Sizing Strategy**:
```yaml
Profile Workloads:
  - Monitor CPU, memory, disk during builds
  - Identify resource bottlenecks

Match to Needs:
  - Don't overprovision (wastes money)
  - Don't underprovision (slows builds)

Specialized Pools:
  - Default pool: 4 vCPU for typical builds
  - Heavy pool: 8+ vCPU for large builds
  - Fast pool: NVMe SSD for I/O-intensive
```

### Multiple Agent Pools

**Pool Strategy**:
```yaml
Pool Assignment:
  Default Pool (4 vCPU):
    - API builds
    - Small web apps
    - Unit tests

  Heavy Pool (8+ vCPU):
    - Mobile app builds
    - Large monolith builds
    - Docker image builds

  Fast Pool (NVMe SSD):
    - npm install heavy projects
    - Large artifact uploads
    - Database restore operations
```

**Example Assignment**:
```yaml
jobs:
  - job: APIBuild
    pool:
      name: Default

  - job: MobileAppBuild
    pool:
      name: Heavy

  - job: NodeJSBuild
    pool:
      name: Fast
```

### Dynamic Scaling (Self-Hosted)

**Azure Virtual Machine Scale Sets**:
```yaml
Auto-Scale Configuration:
  Scale Up:
    - When: Queue depth > 5
    - Action: Add 2 agents
    - Max: 20 agents

  Scale Down:
    - When: Agents idle > 30 min
    - Action: Remove 1 agent
    - Min: 5 agents

Schedule:
  Business Hours (8 AM - 6 PM):
    - Min: 10 agents
    - Max: 20 agents

  Off-Peak (6 PM - 8 AM, weekends):
    - Min: 5 agents
    - Max: 10 agents

Cost Savings:
  - Scale from 5 (overnight) to 20 (business hours)
  - Average: 12 agents vs. static 20 agents
  - Savings: 40% cost reduction
```

## Monitoring and Tuning Concurrency

### Key Metrics

| Metric | Target | Action if High | Action if Low |
|--------|--------|----------------|---------------|
| **Queue Time** | < 1 min | Add more agents or licenses | - |
| **Agent Utilization** | 60-80% | - | Reduce agents or consolidate pools |
| **Concurrency Efficiency** | > 80% | - | Improve parallelization |

**Dashboard Metrics**:
```yaml
Real-Time:
  - Current queue depth: Jobs waiting now
  - Agent busy %: Current utilization
  - Parallelism factor: Concurrent jobs / available agents

Historical:
  - Average queue time: Last 24 hours
  - P90 queue time: 90th percentile wait
  - Peak usage hours: When is demand highest?
```

### Tuning Approach

**Step 1: Establish Baseline**
```yaml
Measure:
  - Current queue times: 3 minutes average
  - Duration: P90 = 15 minutes
  - Agent utilization: 45% (too low)

Identify Patterns:
  - Peak hours: 9 AM - 11 AM, 2 PM - 4 PM
  - Low usage: 6 PM - 8 AM, weekends
```

**Step 2: Identify Bottlenecks**
```yaml
Analysis:
  Long queue times → Need more agents
  Low utilization → Too many agents
  Long durations → Need parallelization or faster agents
```

**Step 3: Implement Changes**
```yaml
Actions:
  - Add 5 agents for peak hours
  - Remove 3 agents for off-peak
  - Adjust parallel job limits from 3 to 5
  - Optimize pipeline for better parallelism
```

**Step 4: Measure Impact**
```yaml
Before:
  - Average queue: 3 min
  - P90 duration: 15 min
  - Utilization: 45%

After:
  - Average queue: 0.5 min (83% improvement)
  - P90 duration: 12 min (20% improvement)
  - Utilization: 65% (optimal range)

Cost:
  - Added 5 agents during peak (8 hours/day)
  - Removed 3 agents off-peak (16 hours/day)
  - Net cost: +10% for 3× faster queue times ✅
```

**Step 5: Iterate**
```yaml
Continuous Monitoring:
  - Weekly review of metrics
  - Adjust for workload changes
  - Seasonal patterns (e.g., end-of-sprint rushes)
```

## Cost Optimization Strategies

### Right-Size for Workload

**Automated Scaling**:
```yaml
Morning Rush (7 AM - 10 AM):
  - Scale to 20 agents
  - Handle morning commit surge

Business Hours (10 AM - 6 PM):
  - Scale to 15 agents
  - Steady-state workload

Evening/Overnight (6 PM - 7 AM):
  - Scale to 5 agents
  - Minimal activity

Weekends:
  - Scale to 3 agents
  - Very low activity

Cost Comparison:
  Static 20 agents: 20 × 24 × 7 = 3,360 agent-hours/week
  Dynamic scaling: ~12 avg × 24 × 7 = 2,016 agent-hours/week
  Savings: 40% reduction ✅
```

### Parallel Job Licenses

**Microsoft-Hosted**:
```yaml
Cost Model:
  - Pay per parallel job license
  - Buy just enough for peak concurrent demand

Analysis:
  - Peak concurrent jobs: 5
  - Buy 5 parallel job licenses
  - Cost: $40/month per license = $200/month
```

**Self-Hosted**:
```yaml
Cost Model:
  - Unlimited parallelism included
  - No incremental cost per parallel job

Advantage:
  - Flat cost regardless of concurrency
  - Better for high-parallelism scenarios
```

## Critical Notes
- ⚠️ **Balance speed and cost**: More parallelism = faster, but also costs more
- 💡 **Profile first**: Understand workload before sizing agents
- 🎯 **Dynamic scaling**: Adjust capacity for demand patterns (save 30-50%)
- 📊 **Monitor queue times**: < 1 minute queue time for good developer experience

## Quick Commands

**Check Queue Depth**:
```bash
# Azure CLI
az pipelines agent list \
  --pool-id <pool-id> \
  --query "[?status=='offline' || status=='busy'].{name:name, status:status}" \
  --output table
```

**Monitor Agent Utilization**:
```kql
// Log Analytics query
PipelineAgentMetrics
| where TimeGenerated > ago(7d)
| summarize 
    TotalHours = count() / 60.0,
    BusyHours = countif(Status == "Busy") / 60.0
    by AgentName
| extend Utilization = round(BusyHours * 100.0 / TotalHours, 2)
| order by Utilization desc
```

**Calculate Parallelism Factor**:
```yaml
Parallelism Factor = Average Concurrent Jobs / Available Agents

Example:
  - Available agents: 10
  - Average concurrent jobs: 7
  - Parallelism factor: 70%

Interpretation:
  - 70% = Good utilization
  - < 50% = Underutilized (reduce agents)
  - > 90% = Overutilized (add agents or optimize dependencies)
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/9-optimize-pipeline-concurrency-performance)
