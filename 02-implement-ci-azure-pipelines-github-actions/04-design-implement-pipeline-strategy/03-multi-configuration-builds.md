# Explore Multi-Configuration and Multi-Agent Builds

Multi-configuration and multi-agent builds enable parallel execution across different configurations or multiple agents, significantly reducing pipeline duration and improving efficiency.

## Types of Job Execution

### 1. Single Agent Jobs (Default)

**Characteristics**:
- Tasks run sequentially on one agent
- Simple, straightforward workflow
- Best for basic builds

```yaml
jobs:
- job: Build
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - script: npm install
  - script: npm test
  - script: npm run build
```

### 2. Multi-Configuration Jobs

**Purpose**: Run same tasks across multiple configurations simultaneously

```yaml
strategy:
  matrix:
    Python38:
      python.version: '3.8'
    Python39:
      python.version: '3.9'
    Python310:
      python.version: '3.10'

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '$(python.version)'
- script: python -m pytest
```

**Each configuration runs on separate agent in parallel**

### 3. Multi-Agent Jobs

**Purpose**: Distribute same tasks across multiple agents to reduce execution time

```yaml
strategy:
  parallel: 4  # Split work across 4 agents

steps:
- script: |
    # Each agent runs 1/4 of tests
    pytest --test-group=$(System.JobPositionInPhase)
```

## Multi-Configuration Use Cases

### Testing Different Configurations

**Example: Cross-Platform Testing**

```yaml
trigger:
- main

strategy:
  matrix:
    Windows:
      imageName: 'windows-latest'
      osType: 'Windows'
    Linux:
      imageName: 'ubuntu-latest'
      osType: 'Linux'
    macOS:
      imageName: 'macos-latest'
      osType: 'macOS'

pool:
  vmImage: $(imageName)

steps:
- script: echo "Testing on $(osType)"
- script: npm install
- script: npm test
```

**Result**: 3 jobs run in parallel, one per OS

### Framework Variations

```yaml
strategy:
  matrix:
    DotNet6:
      framework: 'net6.0'
    DotNet7:
      framework: 'net7.0'
    DotNet8:
      framework: 'net8.0'

steps:
- script: dotnet build --framework $(framework)
- script: dotnet test --framework $(framework)
```

### Multi-Region Deployments

```yaml
strategy:
  matrix:
    EastUS:
      region: 'eastus'
      resourceGroup: 'rg-eastus'
    WestEurope:
      region: 'westeurope'
      resourceGroup: 'rg-westeurope'
    AsiaPacific:
      region: 'southeastasia'
      resourceGroup: 'rg-asia'

steps:
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Production'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az webapp create --name myapp-$(region)         --resource-group $(resourceGroup)         --location $(region)
```

## Multi-Agent Distribution

### Large Test Suite Example

**Problem**: 1,000 tests take 60 minutes on single agent

**Solution**: Distribute across 4 agents

```yaml
strategy:
  parallel: 4

steps:
- script: |
    # Each agent runs ~250 tests
    # Total time: ~15 minutes
    pytest --test-group=$(System.JobPositionInPhase)            --total-groups=$(System.TotalJobsInPhase)
```

**Variables Available**:
- `$(System.JobPositionInPhase)`: Current job number (1-based)
- `$(System.TotalJobsInPhase)`: Total parallel jobs

### Data Processing

```yaml
strategy:
  parallel: 8  # 8 agents process data

steps:
- script: |
    # Each agent processes 1/8 of dataset
    python process_data.py       --partition=$(System.JobPositionInPhase)       --total-partitions=$(System.TotalJobsInPhase)
```

## Planning Your Job Strategy

### Decision Matrix

| Factor | Single Agent | Multi-Configuration | Multi-Agent |
|--------|--------------|-------------------|-------------|
| **Test multiple platforms** | ❌ Sequential | ✅ Parallel | ❌ |
| **Reduce test suite time** | ❌ | ❌ | ✅ Parallel distribution |
| **Deploy to multiple regions** | ❌ Sequential | ✅ Parallel | ❌ |
| **Process large datasets** | ❌ Slow | ❌ | ✅ Partition data |
| **Simple build** | ✅ Easy | ❌ Overkill | ❌ Overkill |

### Performance Considerations

**Example: E-Commerce Application**

```yaml
# Before: Sequential (60 minutes total)
Job 1: Build (10 min)
Job 2: Unit Tests (15 min)
Job 3: Integration Tests (20 min)
Job 4: UI Tests (15 min)

# After: Multi-Configuration (20 minutes total)
strategy:
  matrix:
    UnitTests:
      testType: 'unit'
      duration: '15 min'
    IntegrationTests:
      testType: 'integration'
      duration: '20 min'
    UITests:
      testType: 'ui'
      duration: '15 min'

# All tests run in parallel
# Total time = longest job (20 min) + build (10 min) = 30 min
```

## Artifacts and Dependencies

### Sharing Artifacts Across Configurations

```yaml
jobs:
- job: Build
  steps:
  - script: npm run build
  - publish: $(System.DefaultWorkingDirectory)/dist
    artifact: WebApp

- job: Test
  dependsOn: Build
  strategy:
    matrix:
      Chrome:
        browser: 'chrome'
      Firefox:
        browser: 'firefox'
      Safari:
        browser: 'safari'
  steps:
  - download: current
    artifact: WebApp
  - script: npm test -- --browser=$(browser)
```

## Deployment Requirements

### Parallel Deployments Safe?

**Safe Scenarios**:
```yaml
# Independent targets - can deploy in parallel
strategy:
  matrix:
    Region1:
      target: 'server1.example.com'
    Region2:
      target: 'server2.example.com'
    Region3:
      target: 'server3.example.com'
```

**Unsafe Scenarios** (require sequential):
```yaml
# Shared database - must deploy sequentially
jobs:
- job: MigrateDatabase
  steps:
  - script: ./migrate-db.sh

- job: DeployApp
  dependsOn: MigrateDatabase  # Must wait
  steps:
  - script: ./deploy-app.sh
```

## Matrix Strategy Advanced

### MaxParallel Limit

```yaml
strategy:
  maxParallel: 2  # Run max 2 configurations at a time
  matrix:
    Config1: { version: '1' }
    Config2: { version: '2' }
    Config3: { version: '3' }
    Config4: { version: '4' }

# Execution:
# Wave 1: Config1 + Config2
# Wave 2: Config3 + Config4
```

### Conditional Matrix

```yaml
strategy:
  matrix:
    ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
      Production:
        environment: 'prod'
    ${{ if ne(variables['Build.SourceBranch'], 'refs/heads/main') }}:
      Development:
        environment: 'dev'
      Staging:
        environment: 'staging'
```

## Cost Implications

### Parallel Jobs Consumption

```yaml
# Consumes 3 parallel jobs
strategy:
  matrix:
    Windows: { os: 'windows' }
    Linux: { os: 'linux' }
    macOS: { os: 'macos' }

# If organization has 2 parallel jobs:
# - Windows + Linux run immediately
# - macOS queues until slot available
```

## Critical Notes

- 🎯 **Multi-configuration = same tasks, different configs** - Test app on Windows/Linux/macOS simultaneously; each runs on separate agent in parallel
- 💡 **Multi-agent = distribute workload** - Split 1,000 tests across 4 agents (250 each); reduces total time from 60 to 15 minutes
- ⚠️ **Parallel jobs limit applies** - Matrix with 5 configs needs 5 parallel job slots; jobs queue if insufficient capacity
- 📊 **Use for independent work** - Parallel deployments safe when targets are independent; sequential required for shared resources (databases)
- 🔄 **Matrix strategy for platforms** - Cross-platform testing, multi-region deployment, framework variations all benefit from matrix
- ✨ **Balance speed vs cost** - More parallelism = faster builds but consumes more parallel jobs; optimize based on bottlenecks

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-pipeline-strategy/3-explore-multi-configuration-multi-agent)
