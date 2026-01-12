# Describe Anatomy of Pipeline

## Key Concepts

Azure Pipelines are defined using YAML schema with specific structural elements that control execution behavior. Understanding pipeline anatomy enables you to create complex automation workflows with proper triggers, dependencies, and resource management.

### Core Pipeline Elements

| Element | Purpose | Scope | Required |
|---------|---------|-------|----------|
| **name** | Build number format | Pipeline | No |
| **trigger** | Automated run conditions | Pipeline | No (implicit) |
| **pr** | Pull request triggers | Pipeline | No |
| **schedules** | Scheduled runs | Pipeline | No |
| **variables** | Reusable values | Pipeline/Stage/Job | No |
| **stages** | Logical boundaries | Pipeline | No (implicit) |
| **jobs** | Units of work | Stage | Yes (implicit) |
| **steps** | Individual tasks | Job | Yes |
| **pool** | Agent selection | Pipeline/Job | No (default) |

### Hello World Pipeline

```yaml
# Simple pipeline demonstrating core elements
name: $(Date:yyyyMMdd)$(Rev:.r)

trigger:
  branches:
    include:
    - main
    - develop
  paths:
    exclude:
    - docs/*
    - README.md

variables:
  buildConfiguration: 'Release'
  vmImage: 'ubuntu-latest'

pool:
  vmImage: $(vmImage)

jobs:
- job: Build
  displayName: 'Build Application'
  steps:
  - script: echo Hello, world!
    displayName: 'Run a one-line script'
  
  - script: |
      echo Add other tasks to build, test, and deploy your project.
      echo See https://aka.ms/yaml
    displayName: 'Run a multi-line script'
```

## Pipeline Anatomy Deep Dive

### 1. Name Format

The `name` property defines the build number format using tokens and functions:

```yaml
# Semantic versioning with counter
name: $(Major).$(Minor).$(Patch)$(Rev:.r)

# Date-based versioning
name: $(Date:yyyyMMdd)$(Rev:.r)

# Branch and date combination
name: $(SourceBranchName)_$(Date:yyyyMMdd)$(Rev:.r)
```

**Common Tokens**:
- `$(Date:FORMAT)` - Current date with custom format
- `$(Rev:r)` - Auto-incremented revision counter
- `$(SourceBranchName)` - Source branch name
- `$(Build.DefinitionName)` - Pipeline name

### 2. Triggers

#### Branch Triggers

```yaml
trigger:
  branches:
    include:
    - main
    - releases/*
    exclude:
    - releases/archive/*
  paths:
    include:
    - src/*
    exclude:
    - docs/*
  tags:
    include:
    - v2.*
```

#### Pull Request Triggers

```yaml
pr:
  branches:
    include:
    - main
    - develop
  paths:
    exclude:
    - docs/*
    - '*.md'
  autoCancel: true  # Cancel outdated builds
```

#### Scheduled Triggers

```yaml
schedules:
- cron: "0 0 * * *"  # Daily at midnight
  displayName: Daily midnight build
  branches:
    include:
    - main
  always: true  # Run even without changes
```

### 3. Jobs and Dependencies

#### Sequential Jobs (Fan-In)

```yaml
jobs:
- job: BuildJob
  steps:
  - script: echo Building...

- job: TestJob
  dependsOn: BuildJob  # Wait for BuildJob
  steps:
  - script: echo Testing...

- job: DeployJob
  dependsOn: TestJob  # Wait for TestJob
  steps:
  - script: echo Deploying...
```

**Flow**: BuildJob → TestJob → DeployJob

#### Parallel Jobs (Fan-Out)

```yaml
jobs:
- job: BuildJob
  steps:
  - script: echo Building...

- job: UnitTest
  dependsOn: BuildJob
  steps:
  - script: echo Unit testing...

- job: IntegrationTest
  dependsOn: BuildJob
  steps:
  - script: echo Integration testing...

- job: SecurityScan
  dependsOn: BuildJob
  steps:
  - script: echo Security scanning...

- job: Deploy
  dependsOn:  # Fan-in: wait for all tests
  - UnitTest
  - IntegrationTest
  - SecurityScan
  steps:
  - script: echo Deploying...
```

**Flow**: BuildJob → (UnitTest + IntegrationTest + SecurityScan) → Deploy

### 4. Checkout and Resources

#### Checkout Control

```yaml
steps:
- checkout: self  # Default: checkout triggering repo
  clean: true  # Clean workspace before checkout
  fetchDepth: 1  # Shallow clone for performance
  lfs: false  # Git LFS support
  submodules: false  # Git submodules
  persistCredentials: true  # Persist credentials for later steps
```

#### Skip Checkout

```yaml
steps:
- checkout: none  # Skip checkout (e.g., for pure deployment jobs)
```

### 5. Download Artifacts

```yaml
steps:
- download: current  # Download artifacts from current pipeline
  artifact: drop
  
- download: MyOtherPipeline  # Download from another pipeline
  artifact: release
```

### 6. Resources

```yaml
resources:
  pipelines:
  - pipeline: SecurityScan  # Consume artifacts from other pipeline
    source: SecurityPipeline
    trigger: true
    
  repositories:
  - repository: templates  # External template repo
    type: github
    name: myorg/pipeline-templates
    endpoint: GitHubConnection
    
  containers:
  - container: linux  # Container for jobs
    image: ubuntu:20.04
```

### 7. Steps and Tasks

#### Script Steps

```yaml
steps:
- script: |
    echo "Multi-line script"
    npm install
    npm run build
  displayName: 'Build Application'
  workingDirectory: $(Build.SourcesDirectory)/src
  env:
    NODE_ENV: production
```

#### Task Steps

```yaml
steps:
- task: DotNetCoreCLI@2
  displayName: 'Build project'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration)'
```

#### PowerShell Steps

```yaml
steps:
- powershell: |
    Write-Host "Running PowerShell"
    Get-ChildItem -Path $(Build.SourcesDirectory)
  displayName: 'List files'
```

### 8. Variables

#### Pipeline Variables

```yaml
variables:
  buildConfiguration: 'Release'
  vmImage: 'ubuntu-latest'
  Major: 1
  Minor: 0
  Patch: 0
```

#### Variable Groups

```yaml
variables:
- group: 'Production-Variables'  # Variable group from Azure DevOps
- name: environment
  value: 'production'
```

#### Runtime Variables

```yaml
steps:
- script: echo "##vso[task.setvariable variable=myOutputVar]Hello World"
  displayName: 'Set output variable'

- script: echo $(myOutputVar)
  displayName: 'Use output variable'
```

## Complete Example Pipeline

```yaml
name: $(Major).$(Minor).$(Patch)$(Rev:.r)

trigger:
  branches:
    include:
    - main
    - develop
  paths:
    exclude:
    - docs/*
    - README.md

pr:
  branches:
    include:
    - main
  autoCancel: true

schedules:
- cron: "0 2 * * 0"  # Weekly Sunday 2 AM
  displayName: Weekly build
  branches:
    include:
    - main
  always: false

variables:
  Major: 1
  Minor: 0
  Patch: 0
  buildConfiguration: 'Release'

resources:
  repositories:
  - repository: templates
    type: git
    name: SharedTemplates
    ref: refs/heads/main

pool:
  vmImage: 'ubuntu-latest'

jobs:
- job: Build
  displayName: 'Build Application'
  steps:
  - checkout: self
    clean: true
    fetchDepth: 1
    
  - script: |
      echo "Building version $(Build.BuildNumber)"
      dotnet build --configuration $(buildConfiguration)
    displayName: 'Build project'
    
  - script: echo "##vso[task.setvariable variable=artifactPath;isOutput=true]$(Build.ArtifactStagingDirectory)"
    name: outputVars
    displayName: 'Set artifact path'
    
  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)'
      ArtifactName: 'drop'

- job: Test
  dependsOn: Build
  displayName: 'Run Tests'
  steps:
  - checkout: self
  - script: dotnet test --configuration $(buildConfiguration) --no-build
    displayName: 'Run unit tests'

- job: Deploy
  dependsOn: Test
  displayName: 'Deploy Application'
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  steps:
  - download: current
    artifact: drop
  - script: echo "Deploying from $(Pipeline.Workspace)/drop"
    displayName: 'Deploy application'
```

## Critical Notes

🎯 **Implicit vs Explicit**: Without explicit `trigger:`, pipeline triggers on all branches. Use `trigger: none` to disable automatic triggers.

💡 **Checkout Optimization**: Use `fetchDepth: 1` for shallow clones to improve performance (reduces clone time by 50-90%).

⚠️ **Dependency Chains**: Jobs without `dependsOn` run in parallel. Use dependencies strategically to balance speed and correctness.

📊 **Variable Scope**: Variables defined at pipeline level are available to all stages/jobs. Use stage/job-specific variables for isolation.

🔄 **Fan-Out Pattern**: Build once, then run multiple test jobs in parallel (unit, integration, security) before fan-in to deployment.

✨ **Resource Triggers**: When consuming artifacts from another pipeline via `resources.pipelines`, use `trigger: true` to automatically trigger when source pipeline completes.

## Quick Reference

### Trigger Patterns

```yaml
# Continuous Integration (CI)
trigger:
  branches:
    include: ['main']

# No automatic triggers (manual only)
trigger: none

# Pull Request validation
pr: ['main', 'develop']

# Scheduled (cron format: minute hour day month weekday)
schedules:
- cron: "0 0 * * *"  # Daily midnight
```

### Job Dependency Patterns

```yaml
# Sequential: A → B → C
jobs:
- job: A
- job: B
  dependsOn: A
- job: C
  dependsOn: B

# Parallel: A → (B + C)
jobs:
- job: A
- job: B
  dependsOn: A
- job: C
  dependsOn: A

# Fan-in: (A + B) → C
jobs:
- job: A
- job: B
- job: C
  dependsOn: [A, B]
```

### Checkout Options

| Option | Purpose | Default | Performance Impact |
|--------|---------|---------|-------------------|
| `clean` | Clean workspace | `false` | +5-10s |
| `fetchDepth` | Clone depth | `0` (full) | Shallow saves 50-90% |
| `lfs` | Git LFS | `false` | Depends on LFS size |
| `submodules` | Git submodules | `false` | +time per submodule |
| `persistCredentials` | Keep credentials | `true` | Minimal |

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-azure-pipelines/describe-anatomy-of-pipeline/)
