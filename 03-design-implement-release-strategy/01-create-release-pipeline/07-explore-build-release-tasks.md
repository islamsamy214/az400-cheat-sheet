# Explore Build and Release Tasks

Tasks are the fundamental building blocks of pipelines—individual executable units that perform specific actions like compiling code, running tests, deploying applications, or sending notifications. Understanding tasks is essential for creating effective build and release pipelines.

## What Are Tasks?

**Definition**: A task is an executable code unit that performs a specific action in a defined sequence within a pipeline.

```
Pipeline Structure

Pipeline
  └── Stage (e.g., Build, Deploy)
       └── Job (runs on an agent)
            └── Steps (sequence of tasks)
                 ├── Task 1: Checkout code
                 ├── Task 2: Restore dependencies
                 ├── Task 3: Build application
                 ├── Task 4: Run tests
                 └── Task 5: Publish artifacts
```

**Key Characteristics**:
- **Atomic**: Each task performs a single, well-defined action
- **Reusable**: Tasks can be used across multiple pipelines
- **Configurable**: Tasks accept parameters to customize behavior
- **Versioned**: Tasks have versions (e.g., v1, v2, v3)
- **Cross-platform**: Many tasks work on Windows, Linux, and macOS

## Task Categories

### 1. Build Tasks

**Purpose**: Compile source code, restore dependencies, and prepare artifacts.

```yaml
# .NET Core build tasks
- task: DotNetCoreCLI@2
  displayName: 'Restore NuGet Packages'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build Solution'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration Release'

# Node.js build tasks
- task: Npm@1
  displayName: 'npm install'
  inputs:
    command: 'install'

- task: Npm@1
  displayName: 'npm build'
  inputs:
    command: 'custom'
    customCommand: 'run build'

# Docker build tasks
- task: Docker@2
  displayName: 'Build Docker Image'
  inputs:
    command: 'build'
    dockerfile: 'Dockerfile'
    tags: '$(Build.BuildId)'
```

**Common Build Tasks**:

| Task | Purpose | Example Use Case |
|------|---------|------------------|
| **DotNetCoreCLI** | .NET Core operations | Build C# applications |
| **Maven** | Java build automation | Build Spring Boot apps |
| **Gradle** | Java/Android builds | Build Android apps |
| **Npm** | Node.js package manager | Build React/Angular apps |
| **Python** | Python builds | Build Python applications |
| **Go** | Go language builds | Build Go microservices |
| **Docker** | Container operations | Build Docker images |

### 2. Test Tasks

**Purpose**: Execute automated tests and publish test results.

```yaml
# Unit tests
- task: DotNetCoreCLI@2
  displayName: 'Run Unit Tests'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--configuration Release --collect:"XPlat Code Coverage"'

# Publish test results
- task: PublishTestResults@2
  displayName: 'Publish Test Results'
  inputs:
    testResultsFormat: 'VSTest'
    testResultsFiles: '**/*.trx'
    mergeTestResults: true
    failTaskOnFailedTests: true

# Code coverage
- task: PublishCodeCoverageResults@1
  displayName: 'Publish Code Coverage'
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
```

**Common Test Tasks**:

| Task | Purpose | Platforms |
|------|---------|-----------|
| **VSTest** | Visual Studio Test Platform | .NET (Windows) |
| **DotNetCoreCLI (test)** | .NET Core tests | Cross-platform |
| **PublishTestResults** | Publish test outcomes | All |
| **PublishCodeCoverageResults** | Publish coverage data | All |
| **Jest** | JavaScript testing | Node.js apps |
| **PyTest** | Python testing | Python apps |

### 3. Utility Tasks

**Purpose**: Perform helper operations like copying files, running scripts, or setting variables.

```yaml
# Copy files
- task: CopyFiles@2
  displayName: 'Copy Deployment Files'
  inputs:
    sourceFolder: '$(Build.SourcesDirectory)/deploy'
    contents: '**/*'
    targetFolder: '$(Build.ArtifactStagingDirectory)'

# PowerShell script
- task: PowerShell@2
  displayName: 'Run Deployment Script'
  inputs:
    targetType: 'filePath'
    filePath: '$(System.DefaultWorkingDirectory)/scripts/deploy.ps1'
    arguments: '-Environment $(Environment) -Version $(Build.BuildNumber)'

# Bash script
- task: Bash@3
  displayName: 'Run Setup Script'
  inputs:
    targetType: 'inline'
    script: |
      echo "Setting up environment..."
      chmod +x ./setup.sh
      ./setup.sh

# Replace tokens in config files
- task: replacetokens@5
  displayName: 'Replace Tokens in Configuration'
  inputs:
    targetFiles: '**/*.config'
    encoding: 'auto'
    tokenPattern: 'default'
    writeBOM: true
```

**Common Utility Tasks**:

| Task | Purpose | Example |
|------|---------|---------|
| **CopyFiles** | Copy files between directories | Prepare artifacts |
| **DeleteFiles** | Delete files/folders | Cleanup old artifacts |
| **PowerShell** | Run PowerShell scripts | Custom deployment logic |
| **Bash** | Run Bash scripts | Linux/macOS operations |
| **DownloadBuildArtifacts** | Download artifacts from build | Multi-stage pipelines |
| **PublishBuildArtifacts** | Publish artifacts | Store build outputs |

### 4. Package Tasks

**Purpose**: Create and publish packages to feeds or registries.

```yaml
# NuGet packaging
- task: NuGetCommand@2
  displayName: 'Pack NuGet Package'
  inputs:
    command: 'pack'
    packagesToPack: 'src/**/*.csproj'
    versioningScheme: 'byBuildNumber'

- task: NuGetCommand@2
  displayName: 'Push to Azure Artifacts'
  inputs:
    command: 'push'
    packagesToPush: '$(Build.ArtifactStagingDirectory)/**/*.nupkg'
    nuGetFeedType: 'internal'
    publishVstsFeed: 'MyFeed'

# npm packaging
- task: Npm@1
  displayName: 'Publish npm Package'
  inputs:
    command: 'publish'
    publishRegistry: 'useFeed'
    publishFeed: 'MyFeed'

# Docker push
- task: Docker@2
  displayName: 'Push to Azure Container Registry'
  inputs:
    command: 'push'
    repository: 'myapp'
    containerRegistry: 'myACR'
    tags: |
      $(Build.BuildId)
      latest
```

**Common Package Tasks**:

| Task | Purpose | Package Type |
|------|---------|--------------|
| **NuGetCommand** | NuGet operations | .NET packages |
| **Npm** | npm operations | JavaScript packages |
| **Maven** | Maven publish | Java packages |
| **Docker** | Container push | Container images |
| **UniversalPackages** | Publish any file type | Universal packages |

### 5. Deploy Tasks

**Purpose**: Deploy applications to target environments.

```yaml
# Azure Web App deployment
- task: AzureWebApp@1
  displayName: 'Deploy to Azure App Service'
  inputs:
    azureSubscription: 'Production'
    appType: 'webApp'
    appName: 'myapp-prod'
    package: '$(Pipeline.Workspace)/drop/webapp.zip'
    deploymentMethod: 'zipDeploy'

# Azure Kubernetes Service deployment
- task: KubernetesManifest@0
  displayName: 'Deploy to AKS'
  inputs:
    action: 'deploy'
    kubernetesServiceConnection: 'AKS-Production'
    namespace: 'production'
    manifests: |
      k8s/deployment.yml
      k8s/service.yml

# Azure Functions deployment
- task: AzureFunctionApp@1
  displayName: 'Deploy Azure Function'
  inputs:
    azureSubscription: 'Production'
    appType: 'functionApp'
    appName: 'myfunctions-prod'
    package: '$(Pipeline.Workspace)/drop/functions.zip'

# IIS Web App deployment
- task: IISWebAppDeploymentOnMachineGroup@0
  displayName: 'Deploy to IIS'
  inputs:
    webSiteName: 'MyWebSite'
    package: '$(Pipeline.Workspace)/drop/webapp.zip'
    removeAdditionalFilesFlag: true
    takeAppOfflineFlag: true
```

**Common Deploy Tasks**:

| Task | Target Platform | Use Case |
|------|----------------|----------|
| **AzureWebApp** | Azure App Service | Web applications |
| **AzureFunctionApp** | Azure Functions | Serverless functions |
| **KubernetesManifest** | Kubernetes/AKS | Container orchestration |
| **AzureRmWebAppDeployment** | Azure App Service (ARM) | ARM-based deployments |
| **SqlAzureDacpacDeployment** | Azure SQL Database | Database deployments |
| **IISWebAppDeployment** | IIS on Windows | On-premises web apps |
| **SSH** | Linux/Unix servers | SSH-based deployments |

### 6. Test/Quality Tasks

**Purpose**: Security scanning, code quality, compliance checks.

```yaml
# SonarQube code quality
- task: SonarQubePrepare@5
  displayName: 'Prepare SonarQube Analysis'
  inputs:
    SonarQube: 'SonarQube'
    projectKey: 'myapp'
    projectName: 'MyApp'

- task: SonarQubeAnalyze@5
  displayName: 'Run SonarQube Analysis'

- task: SonarQubePublish@5
  displayName: 'Publish Quality Gate Result'

# Security scanning
- task: WhiteSource@21
  displayName: 'Security Vulnerability Scan'
  inputs:
    cwd: '$(Build.SourcesDirectory)'

# OWASP Dependency Check
- task: dependency-check-build-task@6
  displayName: 'OWASP Dependency Check'
  inputs:
    projectName: 'MyApp'
    scanPath: '$(Build.SourcesDirectory)'
    format: 'HTML'
```

## Task Versions

Tasks are versioned to maintain backward compatibility and introduce new features.

```yaml
# Task versioning examples

# Version 1 (deprecated)
- task: AzureWebApp@1
  inputs:
    # Old parameters

# Version 2 (current)
- task: AzureWebApp@2
  inputs:
    # New parameters with more features
    
# Always specify version to avoid breaking changes
# Use latest stable version (not preview)
```

**Version Best Practices**:
- ✅ Always specify task version explicitly (`@1`, `@2`, etc.)
- ✅ Use latest stable version (avoid preview versions in production)
- ✅ Test pipeline before upgrading task versions
- ⚠️ Review breaking changes in task release notes

## Task Conditions

Control task execution with conditions:

```yaml
# Run task only on specific branch
- task: AzureWebApp@1
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
  inputs:
    # Deployment tasks

# Run task only if previous task succeeded
- task: PublishTestResults@2
  condition: succeeded()
  inputs:
    # Publish tasks

# Run task even if previous tasks failed
- task: PublishBuildArtifacts@1
  condition: always()
  inputs:
    # Always publish artifacts for debugging

# Run task only if tests failed
- task: Slack@1
  condition: failed()
  inputs:
    # Notify team of failure

# Complex condition
- task: AzureWebApp@1
  condition: |
    and(
      succeeded(),
      eq(variables['Build.SourceBranch'], 'refs/heads/main'),
      eq(variables['BuildConfiguration'], 'Release')
    )
```

**Common Conditions**:

| Condition | When Task Runs |
|-----------|----------------|
| `succeeded()` | Previous tasks succeeded (default) |
| `failed()` | Any previous task failed |
| `always()` | Regardless of previous task status |
| `succeededOrFailed()` | Previous tasks completed (not canceled) |
| `eq(var1, var2)` | Variables are equal |
| `ne(var1, var2)` | Variables are not equal |
| `and(cond1, cond2)` | Both conditions true |
| `or(cond1, cond2)` | Either condition true |

## Task Marketplace

When built-in tasks don't meet requirements, use community-contributed tasks from marketplaces.

```
Task Marketplaces

Azure DevOps Marketplace:
  URL: https://marketplace.visualstudio.com
  Categories:
    - Azure services
    - Testing tools
    - Deployment tools
    - Security scanners
    - Notification services
  
Examples:
  - WhiteSource (security scanning)
  - Slack (notifications)
  - Octopus Deploy (deployment)
  - ServiceNow (change management)
  - Terraform (infrastructure as code)
```

**Installing Marketplace Tasks**:

1. **Browse Marketplace**: Visit [marketplace.visualstudio.com](https://marketplace.visualstudio.com)
2. **Select Extension**: Find task extension (e.g., "Slack Notification")
3. **Install**: Click "Get it free" → Select organization → Install
4. **Use in Pipeline**: Task appears in task list

```yaml
# Example: Using Slack marketplace task
- task: SlackNotification@1
  inputs:
    SlackApiToken: '$(SlackToken)'
    ChannelId: 'C12345'
    Message: 'Build $(Build.BuildNumber) deployed successfully!'
```

**Popular Marketplace Extensions**:

| Extension | Purpose | Publisher |
|-----------|---------|-----------|
| **WhiteSource** | Security/license scanning | WhiteSource |
| **SonarQube** | Code quality analysis | SonarSource |
| **Slack** | Team notifications | Microsoft DevLabs |
| **Terraform** | Infrastructure as code | Microsoft DevLabs |
| **Octopus Deploy** | Deployment automation | Octopus Deploy |
| **Replace Tokens** | Configuration management | Guillaume Rouchon |
| **ARM Outputs** | Extract ARM template outputs | Keesschollaart |

## Task Groups (Classic Pipelines)

**Task groups** encapsulate multiple tasks into a single reusable unit (classic pipelines only).

```
Task Group Benefits:

Reusability:
  Create once → Use in multiple pipelines
  
Consistency:
  Same deployment steps across environments
  
Maintainability:
  Update once → Applied to all pipelines using the group
  
Abstraction:
  Hide complexity from pipeline authors
```

**Example Task Group**:
```
Task Group: "Deploy Web Application"

Tasks:
1. Download artifact
2. Replace configuration tokens
3. Deploy to Azure App Service
4. Run smoke tests
5. Send Slack notification

Used in:
  - Dev environment pipeline
  - QA environment pipeline
  - Production environment pipeline
```

**YAML Alternative**: Use templates instead of task groups:
```yaml
# Template: deploy-template.yml
parameters:
  environment: ''
  appServiceName: ''

steps:
- task: DownloadBuildArtifacts@0
  inputs:
    artifactName: 'drop'

- task: AzureWebApp@1
  inputs:
    azureSubscription: '${{ parameters.environment }}'
    appName: '${{ parameters.appServiceName }}'
    package: '$(System.ArtifactsDirectory)/drop/webapp.zip'

- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      Invoke-WebRequest -Uri 'https://${{ parameters.appServiceName }}.azurewebsites.net/health'

# Use template in pipeline
steps:
- template: deploy-template.yml
  parameters:
    environment: 'Production'
    appServiceName: 'myapp-prod'
```

## Best Practices

### 1. Use Appropriate Task Versions

```yaml
# ✅ Good: Specific version
- task: AzureWebApp@1

# ❌ Bad: No version (uses latest, may break)
- task: AzureWebApp
```

### 2. Meaningful Display Names

```yaml
# ✅ Good: Descriptive names
- task: DotNetCoreCLI@2
  displayName: 'Build API Project'
  inputs:
    command: 'build'
    projects: 'src/Api/**/*.csproj'

# ❌ Bad: Generic name
- task: DotNetCoreCLI@2
  displayName: 'Build'
```

### 3. Fail Fast with Conditions

```yaml
# Stop pipeline if critical task fails
- task: DotNetCoreCLI@2
  displayName: 'Run Unit Tests'
  inputs:
    command: 'test'
  continueOnError: false  # Fail pipeline if tests fail

# Always publish results for debugging
- task: PublishTestResults@2
  condition: always()  # Run even if tests failed
```

### 4. Secret Management

```yaml
# ✅ Good: Use Azure Key Vault
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production'
    keyVaultName: 'mykeyvault'
    secretsFilter: '*'

- task: PowerShell@2
  inputs:
    script: |
      # Secrets available as environment variables
      Write-Host "Deploying with API key $(ApiKey)"

# ❌ Bad: Hardcoded secrets
- task: PowerShell@2
  inputs:
    script: |
      $apiKey = "abc123xyz"  # NEVER DO THIS!
```

## Critical Notes

🎯 **Task Versioning**: Always specify task version to avoid breaking changes from automatic updates.

💡 **Conditions**: Use conditions to control task execution—run cleanup tasks `always()`, deploy tasks on `succeeded()`.

⚠️ **Marketplace Extensions**: Review marketplace extensions for security and reliability before installing.

📊 **Task Groups/Templates**: Create reusable task groups (classic) or templates (YAML) for consistent deployments.

🔄 **Display Names**: Use descriptive display names to make pipeline logs readable and troubleshooting easier.

✨ **Fail Fast**: Configure critical tasks to fail pipeline immediately—don't deploy if tests fail.

## Quick Reference

### Common Task Pattern

```yaml
# Typical deployment task structure
- task: TaskName@Version
  displayName: 'Human-Readable Description'
  condition: succeeded()  # When to run
  continueOnError: false  # Fail pipeline on error
  inputs:
    parameter1: 'value1'
    parameter2: '$(Variable)'
  env:
    SECRET_KEY: $(SecretVariable)
```

### Task Categories Summary

```
Build:     DotNetCoreCLI, Maven, Npm, Docker
Test:      VSTest, PublishTestResults, CodeCoverage
Utility:   CopyFiles, PowerShell, Bash, DownloadArtifacts
Package:   NuGet, npm, Docker, UniversalPackages
Deploy:    AzureWebApp, Kubernetes, Functions, SQL
Quality:   SonarQube, WhiteSource, OWASP
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline-devops/7-explore-build-release-tasks)
