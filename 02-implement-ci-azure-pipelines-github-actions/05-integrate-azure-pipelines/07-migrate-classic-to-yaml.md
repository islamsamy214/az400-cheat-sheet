# Migrate from Classic Pipelines to YAML

## Key Concepts

Azure Pipelines supports two authoring experiences: Classic (visual designer) and YAML (code-based). YAML pipelines provide version control, code review, and infrastructure-as-code benefits. Microsoft recommends YAML for new pipelines and provides migration tools for existing classic pipelines.

### Classic vs YAML Comparison

| Feature | Classic Pipelines | YAML Pipelines |
|---------|-------------------|----------------|
| **Authoring** | Visual designer (GUI) | Code editor (YAML) |
| **Version Control** | Separate from code | Stored with code |
| **Code Review** | Not applicable | PR reviews apply |
| **Branching** | Single definition | Branch-specific configs |
| **Reusability** | Task groups | Templates |
| **History** | Pipeline history only | Git history |
| **Multi-Stage** | Release pipelines (separate) | Single YAML file |
| **Variables** | UI-defined | YAML + variable groups |
| **Approvals** | Release gates | Environment approvals |

### Why Migrate to YAML?

**Version Control Integration**:
- Pipeline changes tracked with code changes
- Branch-specific pipeline configurations
- Rollback pipeline changes via Git

**Code Review Process**:
- Pipeline changes reviewed via pull requests
- Prevents unauthorized pipeline modifications
- Team collaboration on pipeline improvements

**Modular Design**:
- Reusable templates across pipelines
- Centralized template repositories
- DRY principle for pipeline code

**Environment-Specific Configurations**:
- Different pipelines for dev/staging/prod branches
- Feature branch-specific builds
- Consistent testing across branches

## Migration Approaches

### 1. View YAML Feature

Azure DevOps provides "View YAML" feature to export classic pipelines to YAML format.

**Steps**:
1. Open classic build pipeline in Azure DevOps
2. Click on pipeline name → Edit
3. Click **"View YAML"** button (top right)
4. Copy generated YAML
5. Create new `azure-pipelines.yml` in repository
6. Paste and commit YAML

**Example**: Classic pipeline with 3 tasks

**Classic Pipeline** (Visual):
- Task: Use .NET Core 8.x
- Task: dotnet restore
- Task: dotnet build --configuration Release
- Task: dotnet test
- Task: Publish artifacts

**Generated YAML**:
```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  displayName: 'Use .NET Core sdk'
  inputs:
    packageType: 'sdk'
    version: '8.x'

- task: DotNetCoreCLI@2
  displayName: 'dotnet restore'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'dotnet build'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration) --no-restore'

- task: DotNetCoreCLI@2
  displayName: 'dotnet test'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--configuration $(buildConfiguration) --no-build'

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: drop'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
```

### 2. Export and Optimize

The "View YAML" feature generates verbose YAML. Optimize for readability and maintainability.

**Before Optimization** (Generated):
```yaml
steps:
- task: DotNetCoreCLI@2
  displayName: 'dotnet restore'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'dotnet build'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration Release --no-restore'
```

**After Optimization**:
```yaml
steps:
- script: |
    dotnet restore
    dotnet build --configuration Release --no-restore
  displayName: 'Build project'
```

**Benefits**:
- Fewer lines (2 vs 11)
- More readable
- Easier to maintain
- Faster execution (less task overhead)

### 3. Incremental Migration

For complex pipelines, migrate incrementally:

**Phase 1**: Build-only YAML pipeline
```yaml
# azure-pipelines-build.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- script: dotnet build --configuration Release
  displayName: 'Build'
```

**Phase 2**: Add testing
```yaml
steps:
- script: dotnet build --configuration Release
  displayName: 'Build'

- script: dotnet test --configuration Release --no-build
  displayName: 'Test'
```

**Phase 3**: Add multi-stage deployment
```yaml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: dotnet build --configuration Release

- stage: Deploy
  jobs:
  - deployment: DeployJob
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying"
```

## Migration Best Practices

### 1. Start with Simple Pipelines

Migrate simple build pipelines first to gain experience before tackling complex release pipelines.

**Simple Build Pipeline**:
```yaml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- script: npm install
  displayName: 'Install dependencies'

- script: npm run build
  displayName: 'Build application'

- script: npm test
  displayName: 'Run tests'
```

### 2. Use Templates for Reusability

Extract common patterns into templates.

**Classic Approach**: Copy-paste tasks across multiple pipelines

**YAML Approach**: Shared templates

```yaml
# templates/dotnet-build.yml
parameters:
- name: buildConfiguration
  type: string
  default: 'Release'

steps:
- task: UseDotNet@2
  inputs:
    version: '8.x'

- script: |
    dotnet restore
    dotnet build --configuration ${{ parameters.buildConfiguration }}
    dotnet test --configuration ${{ parameters.buildConfiguration }} --no-build
  displayName: 'Build and test'
```

**Consumption**:
```yaml
# azure-pipelines.yml
steps:
- template: templates/dotnet-build.yml
  parameters:
    buildConfiguration: 'Release'
```

### 3. Migrate Variables

**Classic Variables** (UI-defined):
- Navigate to Pipeline → Edit → Variables tab
- Export variables to YAML

**YAML Variables**:
```yaml
variables:
  buildConfiguration: 'Release'
  vmImage: 'ubuntu-latest'
  azureSubscription: 'MyAzureConnection'

# Or use variable groups
variables:
- group: 'Production-Variables'
- name: buildConfiguration
  value: 'Release'
```

### 4. Handle Secrets

**Classic Approach**: Variables marked as "secret" in UI

**YAML Approach**: Use variable groups with Azure Key Vault integration

```yaml
variables:
- group: 'KeyVault-Secrets'  # Linked to Azure Key Vault

steps:
- script: |
    echo "Using secret: $(SecretValue)"
  displayName: 'Access secret'
  env:
    SECRET: $(SecretValue)  # Secret from Key Vault
```

### 5. Migrate Release Pipelines

Classic release pipelines become multi-stage YAML pipelines.

**Classic Release** (Separate):
- Build pipeline (classic)
- Release pipeline with stages: Dev → Staging → Production

**YAML Multi-Stage** (Single file):
```yaml
trigger:
- main

stages:
- stage: Build
  jobs:
  - job: BuildApp
    steps:
    - script: dotnet build --configuration Release
    - publish: $(Build.ArtifactStagingDirectory)
      artifact: drop

- stage: DeployDev
  dependsOn: Build
  jobs:
  - deployment: DeployToDev
    environment: 'Development'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          - script: echo "Deploy to Dev"

- stage: DeployStaging
  dependsOn: DeployDev
  jobs:
  - deployment: DeployToStaging
    environment: 'Staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          - script: echo "Deploy to Staging"

- stage: DeployProduction
  dependsOn: DeployStaging
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployToProduction
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          - script: echo "Deploy to Production"
```

## Task Assistant

The Task Assistant helps discover and configure tasks in YAML pipelines.

### Using Task Assistant

**In Azure DevOps**:
1. Edit YAML pipeline in Azure DevOps
2. Click **"Show assistant"** (right panel)
3. Search for task (e.g., "Azure Web App")
4. Configure task parameters in UI
5. Click **"Add"** to insert YAML snippet

**Example**: Adding Azure Web App Deployment

**Task Assistant UI**:
- Search: "Azure Web App"
- Azure subscription: Select from dropdown
- App type: Web App on Linux
- App name: my-webapp
- Package: $(Pipeline.Workspace)/drop/*.zip

**Generated YAML**:
```yaml
- task: AzureWebApp@1
  displayName: 'Azure Web App Deploy'
  inputs:
    azureSubscription: 'MyAzureConnection'
    appType: 'webAppLinux'
    appName: 'my-webapp'
    package: '$(Pipeline.Workspace)/drop/*.zip'
```

### Task Assistant Benefits

- Discover available tasks
- Understand task parameters
- Get correct YAML syntax
- Avoid manual documentation lookup
- See parameter validation

## Configuration Differences

### Time Zone Handling

**Classic Pipelines**: Use local time zone of agent

**YAML Pipelines**: Always use UTC for scheduled triggers

```yaml
schedules:
- cron: "0 8 * * *"  # 8:00 AM UTC (not local time)
  displayName: Daily morning build
  branches:
    include:
    - main
  always: true
```

**Converting Local to UTC**:
- Pacific Standard Time (PST) is UTC-8
- To run at 8 AM PST, schedule at 16:00 UTC (4 PM UTC)

```yaml
schedules:
- cron: "0 16 * * *"  # 8:00 AM PST (16:00 UTC)
  displayName: Daily morning build (8 AM PST)
  branches:
    include:
    - main
```

### Variable Expansion

**Classic**: Variables expanded at queue time

**YAML**: Variables expanded at compile time or runtime depending on syntax

```yaml
# Compile-time (template expression)
${{ variables.buildConfiguration }}  # Evaluated when pipeline is compiled

# Runtime (macro syntax)
$(buildConfiguration)  # Evaluated during pipeline execution

# Runtime with dependencies (runtime expression)
$[variables.buildConfiguration]  # Evaluated with dependency support
```

## Complete Migration Example

### Classic Build Pipeline

**Classic Configuration**:
- Trigger: CI on main branch
- Pool: Ubuntu Latest
- Variables: buildConfiguration = Release
- Tasks:
  1. Use .NET 8.x
  2. dotnet restore
  3. dotnet build
  4. dotnet test
  5. Publish test results
  6. Publish code coverage
  7. Publish artifacts

### Migrated YAML Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '8.x'

stages:
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: BuildJob
    displayName: 'Build Application'
    steps:
    # Install .NET SDK
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        packageType: 'sdk'
        version: $(dotnetVersion)
    
    # Restore, build, and test
    - script: |
        dotnet restore
        dotnet build --configuration $(buildConfiguration) --no-restore
        dotnet test --configuration $(buildConfiguration) --no-build \
                    --logger trx \
                    --collect "Code coverage"
      displayName: 'Build and test'
    
    # Publish test results
    - task: PublishTestResults@2
      displayName: 'Publish test results'
      condition: succeededOrFailed()
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/*.trx'
        failTaskOnFailedTests: true
    
    # Publish code coverage
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish code coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
    
    # Publish build artifacts
    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'

- stage: Deploy
  displayName: 'Deploy to Azure'
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployWeb
    displayName: 'Deploy Web App'
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureWebApp@1
            displayName: 'Deploy to Azure Web App'
            inputs:
              azureSubscription: 'AzureConnection'
              appType: 'webAppLinux'
              appName: 'my-webapp-prod'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

## Critical Notes

🎯 **View YAML Feature**: Best starting point for migration. Generates verbose but functional YAML that you can optimize iteratively.

💡 **Templates First**: Before migrating all pipelines, create template library for common patterns (build, test, deploy). Reduces duplication.

⚠️ **Time Zones**: YAML scheduled triggers always use UTC. Convert local times to UTC to avoid scheduling errors.

📊 **Incremental Approach**: Don't migrate everything at once. Start with build, then add tests, then multi-stage deployment. Reduces risk.

🔄 **Variable Groups**: Migrate UI variables to variable groups with Azure Key Vault integration for secrets. Maintains security while enabling YAML.

✨ **Task Assistant**: Use Task Assistant in Azure DevOps editor to discover tasks and generate correct YAML syntax. Faster than reading documentation.

## Quick Reference

### Migration Checklist

```markdown
☐ Identify classic pipeline to migrate
☐ Use "View YAML" to export configuration
☐ Create azure-pipelines.yml in repository
☐ Optimize generated YAML (remove verbosity)
☐ Migrate variables to YAML or variable groups
☐ Migrate secrets to Key Vault-backed variable groups
☐ Test YAML pipeline in feature branch
☐ Convert release pipelines to multi-stage YAML
☐ Add environment approvals/checks
☐ Update documentation
☐ Disable/delete classic pipeline
```

### Common Task Conversions

| Classic Task | YAML Script Alternative |
|--------------|-------------------------|
| DotNetCoreCLI@2 (restore) | `dotnet restore` |
| DotNetCoreCLI@2 (build) | `dotnet build` |
| DotNetCoreCLI@2 (test) | `dotnet test` |
| Npm@1 (install) | `npm install` |
| Npm@1 (run build) | `npm run build` |
| CopyFiles@2 | `cp -r source dest` |
| PublishBuildArtifacts@1 | `publish: path` |

### YAML Advantages Summary

| Benefit | Impact |
|---------|--------|
| Version control | Pipeline history in Git |
| Code review | PR approval for pipeline changes |
| Branching | Different pipelines per branch |
| Templates | DRY principle, reusability |
| Multi-stage | Single file for build + deploy |
| Consistency | Infrastructure as code |

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-azure-pipelines/migrate-pipeline-classic-to-yaml/)
