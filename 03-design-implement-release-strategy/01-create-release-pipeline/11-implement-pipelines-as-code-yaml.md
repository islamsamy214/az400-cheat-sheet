# Implement Pipelines as Code with YAML

YAML pipelines enable infrastructure-as-code for CI/CD, providing version control, code review, and reusability for release processes. Multi-stage YAML pipelines combine build and release into a single, unified pipeline definition.

## Classic vs YAML Pipelines

### Architecture Comparison

```
Classic Release Pipeline:
┌─────────────┐       ┌──────────────────┐
│Build        │──────→│Release (Separate)│
│Pipeline     │       │ - Dev Stage      │
│(YAML/GUI)   │       │ - QA Stage       │
└─────────────┘       │ - Prod Stage     │
                      └──────────────────┘
                      ⚠️ UI-configured, not in source control

Multi-Stage YAML Pipeline:
┌────────────────────────────────────────┐
│  Single YAML File (in source control)  │
│  ├── Build Stage                       │
│  ├── Deploy Dev Stage                  │
│  ├── Deploy QA Stage                   │
│  └── Deploy Prod Stage                 │
└────────────────────────────────────────┘
✅ Version controlled, code reviewable
```

### Key Differences

| Aspect | Classic Release | Multi-Stage YAML |
|--------|-----------------|------------------|
| **Configuration** | UI (web-based) | Code (YAML file) |
| **Version Control** | ❌ No (stored in Azure DevOps) | ✅ Yes (in repository) |
| **Code Review** | ❌ No | ✅ Yes (PRs, approvals) |
| **Branching** | Single definition | ✅ Per-branch definitions |
| **Reusability** | Task groups | Templates |
| **History** | Deployment history | Git history |
| **Testing** | Hard to test changes | ✅ Test in feature branches |
| **Portability** | Azure DevOps only | YAML (portable) |

## Multi-Stage YAML Pipeline Structure

### Basic Structure

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: BuildJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: echo "Building application..."
    - script: echo "Running tests..."
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

- stage: DeployDev
  displayName: 'Deploy to Development'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployWeb
    environment: 'Dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to Dev..."

- stage: DeployQA
  displayName: 'Deploy to QA'
  dependsOn: DeployDev
  jobs:
  - deployment: DeployWeb
    environment: 'QA'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to QA..."

- stage: DeployProd
  displayName: 'Deploy to Production'
  dependsOn: DeployQA
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployWeb
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to Production..."
```

### Stage Dependencies

**Control execution order**:

```yaml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: echo "Build"

# Sequential (one after another)
- stage: DeployDev
  dependsOn: Build  # Wait for Build
  jobs:
  - deployment: Deploy
    environment: Dev

- stage: DeployQA
  dependsOn: DeployDev  # Wait for DeployDev
  jobs:
  - deployment: Deploy
    environment: QA

# Parallel (both run after DeployDev)
- stage: DeployEastUS
  dependsOn: DeployDev
  jobs:
  - deployment: Deploy
    environment: Prod-EastUS

- stage: DeployWestUS
  dependsOn: DeployDev  # Both depend on DeployDev, so run in parallel
  jobs:
  - deployment: Deploy
    environment: Prod-WestUS

# Multiple dependencies (wait for ALL)
- stage: DeployProd
  dependsOn:
  - DeployEastUS
  - DeployWestUS
  condition: succeeded()
  jobs:
  - deployment: Deploy
    environment: Production
```

## Deployment Jobs

### Deployment Job Structure

**Deployment jobs track deployment history and support rollback**:

```yaml
jobs:
- deployment: DeploymentJobName
  displayName: 'Deploy to Environment'
  environment: 'EnvironmentName'  # Required for deployment jobs
  strategy:
    runOnce:  # or rolling, canary
      deploy:
        steps:
        - download: current
          artifact: drop
        
        - task: AzureWebApp@1
          inputs:
            azureSubscription: 'AzureConnection'
            appName: 'my-web-app'
            package: '$(Pipeline.Workspace)/drop/*.zip'
```

### Deployment Strategies

#### 1. RunOnce Strategy

**Deploy once to all targets**:

```yaml
- deployment: DeployApp
  environment: Production
  strategy:
    runOnce:
      preDeploy:
        steps:
        - script: echo "Pre-deployment validation"
      
      deploy:
        steps:
        - task: AzureWebApp@1
          inputs:
            appName: 'myapp'
            package: '$(Pipeline.Workspace)/drop/*.zip'
      
      routeTraffic:
        steps:
        - script: echo "Switch traffic to new version"
      
      postRouteTraffic:
        steps:
        - script: echo "Monitor new version"
      
      on:
        failure:
          steps:
          - script: echo "Rollback on failure"
        success:
          steps:
          - script: echo "Send success notification"
```

#### 2. Rolling Strategy

**Deploy gradually to multiple targets**:

```yaml
- deployment: DeployToWebServers
  environment:
    name: Production
    resourceType: VirtualMachine
    tags: web
  strategy:
    rolling:
      maxParallel: 2  # Deploy to 2 servers at a time
      preDeploy:
        steps:
        - script: echo "Take server out of load balancer"
      
      deploy:
        steps:
        - task: IISWebAppDeploymentOnMachineGroup@0
          inputs:
            WebSiteName: 'MyWebSite'
            Package: '$(Pipeline.Workspace)/drop/*.zip'
      
      postRouteTraffic:
        steps:
        - script: echo "Add server back to load balancer"
      
      on:
        failure:
          steps:
          - script: echo "Stop rolling deployment on failure"
```

**Rolling Deployment Flow**:

```
10 Web Servers, maxParallel: 2

Batch 1: Deploy to Server 1, 2
   ↓ (both succeed)
Batch 2: Deploy to Server 3, 4
   ↓ (both succeed)
Batch 3: Deploy to Server 5, 6
   ↓ (Server 5 fails)
Stop! Rollback servers 1-5
Servers 7-10 still on old version (safe)
```

#### 3. Canary Strategy

**Progressive rollout with traffic shifting**:

```yaml
- deployment: CanaryDeploy
  environment: Production
  strategy:
    canary:
      increments: [10, 25, 50, 100]  # Deploy to 10%, 25%, 50%, 100% of servers
      preDeploy:
        steps:
        - script: echo "Pre-deploy validation"
      
      deploy:
        steps:
        - script: echo "Deploy to $(strategy.increment)% of servers"
        - task: AzureWebApp@1
          inputs:
            appName: 'myapp'
            package: '$(Pipeline.Workspace)/drop/*.zip'
      
      postRouteTraffic:
        steps:
        - script: |
            # Monitor metrics for 10 minutes
            sleep 600
            # Check error rate
            if [ $(ERROR_RATE) -gt 5 ]; then
              echo "Error rate too high, failing deployment"
              exit 1
            fi
          displayName: 'Monitor Canary'
      
      on:
        failure:
          steps:
          - script: echo "Rollback canary deployment"
```

**Canary Flow**:

```
100 Servers

Increment 1 (10%): Deploy to 10 servers
   ↓ Monitor for 10 minutes
   ✅ Metrics good? Continue

Increment 2 (25%): Deploy to 25 servers total (15 more)
   ↓ Monitor for 10 minutes
   ✅ Metrics good? Continue

Increment 3 (50%): Deploy to 50 servers total (25 more)
   ↓ Monitor for 10 minutes
   ✅ Metrics good? Continue

Increment 4 (100%): Deploy to all 100 servers (50 more)
   ✅ Full deployment complete
```

## Environments and Approvals

### Creating Environments

**Define deployment targets**:

```yaml
# Environment defined in YAML
- deployment: DeployProd
  environment: Production  # References Azure DevOps environment
  strategy:
    runOnce:
      deploy:
        steps:
        - script: echo "Deploy"
```

**Create environment in Azure DevOps**:
```
Pipelines > Environments > New environment

Name: Production
Type: 
  - None (agentless approvals)
  - Virtual machines (deployment groups)
  - Kubernetes (AKS clusters)
```

### Approval Gates

**Require approval before deployment**:

```yaml
stages:
- stage: DeployProd
  jobs:
  - deployment: DeployApp
    environment: Production  # Production environment has approvals configured
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploy after approval"
```

**Configure approvals in Azure DevOps**:
```
Environments > Production > Approvals and checks > Add check > Approvals

Approvers: user@company.com, DevOps Team
Timeout: 30 days
Instructions: "Verify staging environment before approving production deployment"
```

**Approval Flow**:

```
Pipeline reaches DeployProd stage
        ↓
Environment "Production" requires approval
        ↓
Notification sent to approvers
        ↓
Approver reviews staging environment
        ↓
Approver clicks "Approve" or "Reject"
        ↓
If approved: Deployment continues
If rejected: Pipeline fails
```

### Environment Checks

**Automated checks before deployment**:

```yaml
# Configure in Azure DevOps UI
Environments > Production > Approvals and checks

Available checks:
- Approvals (manual)
- Branch control (only deploy from main)
- Business hours (only deploy 9-5 weekdays)
- Invoke Azure Function (custom validation)
- Invoke REST API (health check)
- Query Azure Monitor alerts (no active alerts)
- Required template (enforce pipeline structure)
```

**Example: Require no active alerts**:

```
Query Azure Monitor Alerts Check:
  Resource Group: Production
  Alert Rule: Critical Errors
  Condition: No active alerts
  
If alerts exist → Deployment blocked
```

## Complete Multi-Stage YAML Example

### Real-World Pipeline

```yaml
# azure-pipelines.yml
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

pr:
  branches:
    include:
    - main
  paths:
    exclude:
    - docs/*

variables:
  - group: GlobalVariables  # Variable group
  - name: buildConfiguration
    value: 'Release'
  - name: vmImageName
    value: 'ubuntu-latest'

stages:
#-----------------------------------
# Build Stage
#-----------------------------------
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: BuildJob
    displayName: 'Build Application'
    pool:
      vmImage: $(vmImageName)
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        version: '8.x'
    
    - task: DotNetCoreCLI@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
    
    - task: DotNetCoreCLI@2
      displayName: 'Build application'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
        arguments: '--configuration $(buildConfiguration) --no-restore'
    
    - task: DotNetCoreCLI@2
      displayName: 'Run unit tests'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage"'
    
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish code coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
    
    - task: DotNetCoreCLI@2
      displayName: 'Publish application'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory) --no-build'
        zipAfterPublish: true
    
    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifacts'
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

#-----------------------------------
# Deploy to Development
#-----------------------------------
- stage: DeployDev
  displayName: 'Deploy to Development'
  dependsOn: Build
  condition: and(succeeded(), ne(variables['Build.Reason'], 'PullRequest'))
  variables:
    - group: Dev-Variables
  jobs:
  - deployment: DeployWeb
    displayName: 'Deploy Web App'
    environment: Development
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureWebApp@1
            displayName: 'Deploy to Azure App Service'
            inputs:
              azureSubscription: 'AzureDevConnection'
              appType: 'webAppLinux'
              appName: '$(appServiceName)'
              package: '$(Pipeline.Workspace)/drop/*.zip'
              runtimeStack: 'DOTNETCORE|8.0'
          
          - task: AzureAppServiceManage@0
            displayName: 'Restart App Service'
            inputs:
              azureSubscription: 'AzureDevConnection'
              action: 'Restart Azure App Service'
              webAppName: '$(appServiceName)'
          
          - script: |
              echo "Running smoke tests..."
              curl -f https://$(appServiceName).azurewebsites.net/health || exit 1
            displayName: 'Smoke Test'

#-----------------------------------
# Deploy to QA
#-----------------------------------
- stage: DeployQA
  displayName: 'Deploy to QA'
  dependsOn: DeployDev
  condition: succeeded()
  variables:
    - group: QA-Variables
  jobs:
  - deployment: DeployWeb
    displayName: 'Deploy Web App'
    environment: QA
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureRmWebAppDeployment@4
            displayName: 'Deploy to Staging Slot'
            inputs:
              azureSubscription: 'AzureQAConnection'
              appType: 'webAppLinux'
              WebAppName: '$(appServiceName)'
              deployToSlotOrASE: true
              slotName: 'staging'
              package: '$(Pipeline.Workspace)/drop/*.zip'
          
          - script: |
              echo "Running integration tests on staging slot..."
              curl -f https://$(appServiceName)-staging.azurewebsites.net/health || exit 1
            displayName: 'Integration Tests'
          
          - task: AzureAppServiceManage@0
            displayName: 'Swap Staging to Production'
            inputs:
              azureSubscription: 'AzureQAConnection'
              action: 'Swap Slots'
              WebAppName: '$(appServiceName)'
              sourceSlot: 'staging'
              targetSlot: 'production'

#-----------------------------------
# Deploy to Production
#-----------------------------------
- stage: DeployProd
  displayName: 'Deploy to Production'
  dependsOn: DeployQA
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  variables:
    - group: Prod-Variables
  jobs:
  - deployment: DeployWeb
    displayName: 'Deploy Web App'
    environment: Production  # Requires approval
    strategy:
      runOnce:
        preDeploy:
          steps:
          - script: |
              echo "Pre-deployment checks..."
              echo "Checking for active incidents..."
            displayName: 'Pre-Deployment Validation'
        
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureWebApp@1
            displayName: 'Deploy to Production'
            inputs:
              azureSubscription: 'AzureProdConnection'
              appType: 'webAppLinux'
              appName: '$(appServiceName)'
              package: '$(Pipeline.Workspace)/drop/*.zip'
              deploymentMethod: 'zipDeploy'
        
        postRouteTraffic:
          steps:
          - script: |
              echo "Post-deployment monitoring..."
              sleep 60
              curl -f https://$(appServiceName).azurewebsites.net/health || exit 1
            displayName: 'Health Check'
          
          - task: PublishToAzureServiceBus@1
            displayName: 'Send Deployment Notification'
            inputs:
              messageBody: 'Production deployment $(Build.BuildNumber) successful'
        
        on:
          failure:
            steps:
            - script: |
                echo "Deployment failed! Sending alert..."
              displayName: 'Send Failure Alert'
```

## Templates for Reusability

### Stage Template

**deployment-stage-template.yml**:

```yaml
parameters:
- name: environmentName
  type: string
- name: azureSubscription
  type: string
- name: appServiceName
  type: string
- name: requireApproval
  type: boolean
  default: false

stages:
- stage: Deploy${{ parameters.environmentName }}
  displayName: 'Deploy to ${{ parameters.environmentName }}'
  jobs:
  - deployment: DeployWeb
    environment: ${{ parameters.environmentName }}
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureWebApp@1
            displayName: 'Deploy to ${{ parameters.environmentName }}'
            inputs:
              azureSubscription: ${{ parameters.azureSubscription }}
              appName: ${{ parameters.appServiceName }}
              package: '$(Pipeline.Workspace)/drop/*.zip'
```

**Using the template**:

```yaml
# azure-pipelines.yml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: echo "Build"

- template: deployment-stage-template.yml
  parameters:
    environmentName: 'Dev'
    azureSubscription: 'AzureDevConnection'
    appServiceName: 'myapp-dev'

- template: deployment-stage-template.yml
  parameters:
    environmentName: 'QA'
    azureSubscription: 'AzureQAConnection'
    appServiceName: 'myapp-qa'

- template: deployment-stage-template.yml
  parameters:
    environmentName: 'Production'
    azureSubscription: 'AzureProdConnection'
    appServiceName: 'myapp-prod'
    requireApproval: true
```

## Benefits of YAML Pipelines

### 1. Version Control

```
Pipeline definition in source control:

azure-pipelines.yml (main branch)
  ├── Build stage
  ├── Deploy Dev stage
  ├── Deploy QA stage
  └── Deploy Prod stage

azure-pipelines.yml (feature/new-deployment)
  ├── Build stage
  ├── Deploy Dev stage
  ├── Deploy QA stage
  ├── Deploy Staging stage  ← New stage
  └── Deploy Prod stage

✅ Test new deployment strategy in feature branch
✅ Code review pipeline changes via Pull Request
✅ Rollback pipeline changes via Git revert
```

### 2. Branch-Specific Pipelines

```yaml
# Different pipeline behavior per branch
trigger:
  branches:
    include:
    - main
    - develop
    - feature/*

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: echo "Build"

# Only deploy from main branch
- stage: DeployProd
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
  jobs:
  - deployment: Deploy
    environment: Production

# Deploy to dev from develop branch
- stage: DeployDev
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/develop')
  jobs:
  - deployment: Deploy
    environment: Development
```

### 3. Code Review for Pipeline Changes

```
Developer modifies azure-pipelines.yml:
  - Adds new deployment stage
  - Changes approval requirements
  - Updates deployment strategy

Create Pull Request → Code Review
  ✅ Reviewer approves changes
  ✅ Merge to main
  ✅ New pipeline definition active
```

## Critical Notes

🎯 **Multi-Stage YAML**: Combines build and release into single pipeline—version controlled, code reviewable, testable in branches.

💡 **Deployment Jobs**: Use deployment jobs for environment deployments—track history, support rollback, integrate with environments.

⚠️ **Environments**: Create environments in Azure DevOps—enable approvals, branch policies, and deployment history tracking.

📊 **Strategies**: Choose deployment strategy based on risk—runOnce (simple), rolling (gradual), canary (progressive with monitoring).

🔄 **Templates**: Extract common deployment patterns into templates—promote reusability and consistency across stages.

✨ **Branch Protection**: Use conditions to protect production deployments—only deploy from `main` branch with approvals.

## Quick Reference

### Basic Multi-Stage Pipeline

```yaml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: echo "Build"

- stage: Deploy
  dependsOn: Build
  jobs:
  - deployment: DeployApp
    environment: Production
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploy"
```

### Deployment Strategy Selection

```
Simple single-target deployment → runOnce
Multiple servers, gradual rollout → rolling
Progressive with traffic shifting → canary
```

[Learn More](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/stages)
