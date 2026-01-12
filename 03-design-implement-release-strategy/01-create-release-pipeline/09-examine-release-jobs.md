# Examine Release Jobs

Release jobs are the execution units within release pipeline stages that define how deployment tasks run on target environments. Understanding job types, agent selection, and deployment patterns is crucial for designing effective release pipelines.

## Release Job Types

Azure Pipelines supports multiple job types for different deployment scenarios:

```
Release Stage
      ↓
┌─────────────────────────────────────┐
│  Job Type Selection                 │
│  ├── Agent Job                      │ ← Most common
│  ├── Deployment Group Job           │ ← Multi-server deployment
│  ├── Agentless Job                  │ ← Manual intervention, gates
│  └── Container Job                  │ ← Deploy in containers
└─────────────────────────────────────┘
```

### 1. Agent Job

**Standard deployment job** running on a single agent:

```yaml
# Classic Release Pipeline (conceptual)
Stage: Production
  Job: Deploy Web App
    Agent Pool: Azure Pipelines (Ubuntu-latest)
    Tasks:
      - Azure App Service Deploy
      - Run smoke tests
      - Send notification
```

**Characteristics**:
- Runs on Microsoft-hosted or self-hosted agent
- Single execution environment
- Sequential task execution
- Most common job type

**Use Cases**:
- Deploy to Azure services (App Service, AKS, Functions)
- Run deployment scripts
- Execute post-deployment tests
- Single-target deployments

**Agent Job Configuration**:

```yaml
# YAML equivalent
jobs:
- job: DeployWebApp
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - task: AzureWebApp@1
    inputs:
      azureSubscription: 'MyAzureConnection'
      appName: 'my-web-app'
      package: '$(Pipeline.Workspace)/drop/*.zip'
  
  - script: |
      curl https://my-web-app.azurewebsites.net/health
    displayName: 'Run smoke test'
```

### 2. Deployment Group Job

**Deploy to multiple servers simultaneously**:

```
Deployment Group: Web Servers
        ↓
┌───────┬───────┬───────┬───────┐
│Server1│Server2│Server3│Server4│
│Deploy │Deploy │Deploy │Deploy │
│Tasks  │Tasks  │Tasks  │Tasks  │
└───────┴───────┴───────┴───────┘
Parallel execution on all servers
```

**What is a Deployment Group?**

A collection of target machines with agents installed that receive deployment artifacts and execute tasks.

**Deployment Group Components**:

| Component | Purpose | Example |
|-----------|---------|---------|
| **Target Machines** | Servers to deploy to | IIS servers, Linux VMs |
| **Deployment Agent** | Runs deployment tasks | Windows/Linux agent |
| **Tags** | Organize servers by role | "web", "database", "api" |
| **Capabilities** | Define what servers can do | IIS installed, SQL tools |

**Creating a Deployment Group**:

```bash
# 1. Create deployment group in Azure DevOps
# Navigate to: Pipelines > Deployment Groups > New

# 2. Install agent on target servers (Windows)
$env:VSTS_AGENT_INPUT_URL = "https://dev.azure.com/myorg"
$env:VSTS_AGENT_INPUT_AUTH = "pat"
$env:VSTS_AGENT_INPUT_TOKEN = "<PAT_TOKEN>"
$env:VSTS_AGENT_INPUT_DEPLOYMENTGROUP = "WebServers"
$env:VSTS_AGENT_INPUT_DEPLOYMENTGROUPTAGS = "web,production"

.\config.cmd

# 3. Install agent on target servers (Linux)
sudo ./config.sh --deploymentgroup \
  --url https://dev.azure.com/myorg \
  --auth pat \
  --token <PAT_TOKEN> \
  --deploymentgroupname "WebServers" \
  --adddeploymentgrouptags --deploymentgrouptags "web,production"
```

**Deployment Group Job Configuration**:

```yaml
# Classic pipeline uses deployment group jobs
# YAML alternative: deployment job
jobs:
- deployment: DeployToWebServers
  displayName: Deploy to all web servers
  environment:
    name: Production
    resourceType: VirtualMachine
    tags: web  # Deploy only to servers tagged 'web'
  strategy:
    rolling:  # Or runOnce, canary
      maxParallel: 2
      deploy:
        steps:
        - task: IISWebAppDeploymentOnMachineGroup@0
          inputs:
            WebSiteName: 'MyWebSite'
            Package: '$(Pipeline.Workspace)/drop/*.zip'
            TakeAppOfflineFlag: true
        
        - powershell: |
            Invoke-WebRequest http://localhost/health
          displayName: 'Verify deployment'
```

**Deployment Strategies with Deployment Groups**:

```yaml
# 1. RunOnce (all servers at once)
strategy:
  runOnce:
    deploy:
      steps:
      - script: echo "Deploy to all servers"

# 2. Rolling (gradual rollout)
strategy:
  rolling:
    maxParallel: 2  # Deploy to 2 servers at a time
    deploy:
      steps:
      - script: echo "Deploy batch"

# 3. Canary (progressive deployment)
strategy:
  canary:
    increments: [10, 25, 50, 100]  # Deploy to 10%, then 25%, etc.
    deploy:
      steps:
      - script: echo "Deploy canary"
```

**Tags for Server Organization**:

```
Deployment Group: Production Servers

┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   Server 1      │  │   Server 2      │  │   Server 3      │
│  Tags:          │  │  Tags:          │  │  Tags:          │
│  - web          │  │  - api          │  │  - web          │
│  - frontend     │  │  - backend      │  │  - frontend     │
│  - production   │  │  - production   │  │  - production   │
└─────────────────┘  └─────────────────┘  └─────────────────┘

Deploy to "web" tag → Servers 1 & 3 only
Deploy to "api" tag → Server 2 only
Deploy to "production" tag → All servers
```

### 3. Agentless Job

**Execute tasks without an agent** (manual intervention, gates, delays):

```yaml
jobs:
- job: WaitForApproval
  pool: server  # Agentless
  steps:
  - task: ManualValidation@0
    inputs:
      notifyUsers: 'admin@company.com'
      instructions: 'Please validate deployment and approve'
      onTimeout: 'reject'

- job: DelayedExecution
  pool: server
  steps:
  - task: Delay@1
    inputs:
      delayForMinutes: '30'
```

**Agentless Tasks**:

| Task | Purpose | Use Case |
|------|---------|----------|
| **Manual Validation** | Wait for human approval | Production gate |
| **Delay** | Wait for time period | Cool-down after deployment |
| **Invoke Azure Function** | Call serverless function | Custom validation logic |
| **Invoke REST API** | Call external service | Check health endpoints |
| **Query Work Items** | Check Azure Boards | Verify no critical bugs |

**Use Cases**:
- Manual approval gates
- Timed delays (waiting for services to stabilize)
- Calling external APIs for validation
- Querying work items for blockers

### 4. Container Job

**Run deployment tasks inside a container**:

```yaml
jobs:
- job: DeployWithContainer
  pool:
    vmImage: 'ubuntu-latest'
  container:
    image: mcr.microsoft.com/azure-cli:latest
    options: --cpus 2 --memory 4g
  steps:
  - script: |
      az login --service-principal -u $(servicePrincipalId) -p $(servicePrincipalKey) --tenant $(tenantId)
      az webapp deployment source config-zip --resource-group myRG --name myApp --src $(Pipeline.Workspace)/app.zip
    displayName: 'Deploy using Azure CLI container'
```

**Benefits**:
- Consistent deployment environment
- Pre-installed tools (Azure CLI, kubectl, Terraform)
- Isolated execution
- Reproducible deployments

## Job Execution Flow

### Agent Job Execution

```
1. Agent Selection
   ↓
2. Workspace Preparation
   ├── Download artifacts
   ├── Clone repository (if needed)
   └── Prepare working directory
   ↓
3. Task Execution
   ├── Pre-task validation
   ├── Execute task (deploy, test, etc.)
   ├── Capture output/logs
   └── Handle errors
   ↓
4. Post-Job Cleanup
   ├── Upload logs
   ├── Publish test results
   └── Clean workspace
   ↓
5. Report Results
```

**Job Execution Example**:

```yaml
jobs:
- job: DeployToProduction
  pool:
    vmImage: 'ubuntu-latest'
  
  # Variables for this job
  variables:
    environmentName: 'Production'
    resourceGroup: 'prod-rg'
  
  # Job-level conditions
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  
  # Timeout
  timeoutInMinutes: 30
  
  # Cancel timeout
  cancelTimeoutInMinutes: 5
  
  steps:
  - download: current  # Download artifacts
    artifact: drop
  
  - task: AzureRmWebAppDeployment@4
    displayName: 'Deploy Azure App Service'
    inputs:
      azureSubscription: 'AzureConnection'
      appType: 'webAppLinux'
      WebAppName: 'myapp-$(environmentName)'
      packageForLinux: '$(Pipeline.Workspace)/drop/*.zip'
      RuntimeStack: 'NODE|18-lts'
  
  - script: |
      echo "Running post-deployment smoke tests..."
      curl -f https://myapp-production.azurewebsites.net/health || exit 1
    displayName: 'Smoke Test'
    continueOnError: false
  
  - task: PublishTestResults@2
    condition: always()  # Run even if previous steps failed
    inputs:
      testResultsFiles: '**/TEST-*.xml'
```

## Job Dependencies

**Control execution order** between jobs:

```yaml
jobs:
- job: BuildApp
  steps:
  - script: echo "Building application"

- job: RunTests
  dependsOn: BuildApp  # Wait for BuildApp to complete
  steps:
  - script: echo "Running tests"

- job: DeployDev
  dependsOn: RunTests  # Wait for RunTests
  condition: succeeded()  # Only if tests passed
  steps:
  - script: echo "Deploy to Dev"

- job: DeployProd
  dependsOn: DeployDev  # Wait for Dev deployment
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  steps:
  - script: echo "Deploy to Production"
```

**Multiple Dependencies**:

```yaml
jobs:
- job: UnitTests
  steps:
  - script: echo "Running unit tests"

- job: IntegrationTests
  steps:
  - script: echo "Running integration tests"

- job: SecurityScan
  steps:
  - script: echo "Running security scan"

- job: Deploy
  dependsOn:  # Wait for ALL to complete
  - UnitTests
  - IntegrationTests
  - SecurityScan
  condition: and(succeeded(), succeeded('UnitTests'), succeeded('IntegrationTests'), succeeded('SecurityScan'))
  steps:
  - script: echo "All checks passed, deploying..."
```

## Parallel vs Sequential Jobs

### Sequential Execution (Default)

```yaml
# Jobs run one after another
jobs:
- job: Job1
  steps:
  - script: echo "Job 1"  # Runs first

- job: Job2
  dependsOn: Job1
  steps:
  - script: echo "Job 2"  # Runs after Job1
```

### Parallel Execution

```yaml
# Jobs run simultaneously
jobs:
- job: DeployRegion1
  steps:
  - script: echo "Deploy to East US"

- job: DeployRegion2
  steps:
  - script: echo "Deploy to West US"

- job: DeployRegion3
  steps:
  - script: echo "Deploy to Europe"

# All three jobs run in parallel (if agents available)
```

**Parallel Jobs Limits**:
- **Free tier**: 1 parallel job (Microsoft-hosted), 1 parallel job (self-hosted)
- **Paid tier**: Purchase additional parallel jobs
- **Effect**: Jobs queue if limit exceeded

## Job Variables and Outputs

**Pass data between jobs**:

```yaml
jobs:
- job: BuildJob
  steps:
  - script: |
      echo "##vso[task.setvariable variable=buildNumber;isOutput=true]1.0.$(Build.BuildId)"
    name: setBuildNumber
  
  - script: |
      echo "Build number set to: $(setBuildNumber.buildNumber)"

- job: DeployJob
  dependsOn: BuildJob
  variables:
    # Reference output from BuildJob
    buildNumber: $[ dependencies.BuildJob.outputs['setBuildNumber.buildNumber'] ]
  steps:
  - script: |
      echo "Deploying build: $(buildNumber)"
```

## Release Job Best Practices

### 1. Use Deployment Jobs for Kubernetes/VMs

```yaml
jobs:
- deployment: DeployToK8s
  environment: production-k8s
  strategy:
    runOnce:
      deploy:
        steps:
        - task: KubernetesManifest@0
          inputs:
            action: 'deploy'
            manifests: 'k8s/*.yaml'
```

**Benefits**:
- Built-in deployment history
- Environment tracking
- Rollback support
- Approval gates

### 2. Implement Job Timeout

```yaml
jobs:
- job: DeployWithTimeout
  timeoutInMinutes: 20  # Fail if exceeds 20 minutes
  cancelTimeoutInMinutes: 2  # Time to gracefully cancel
  steps:
  - script: echo "Deploy"
```

### 3. Use Conditions Wisely

```yaml
jobs:
- job: DeployProduction
  condition: |
    and(
      succeeded(),
      eq(variables['Build.SourceBranch'], 'refs/heads/main'),
      eq(variables['Build.Reason'], 'Manual')
    )
  steps:
  - script: echo "Deploy to production"
```

### 4. Tag Deployment Group Servers

```bash
# Organize servers by role for targeted deployment
Tags: web, frontend, production
Tags: api, backend, production
Tags: database, production
```

## Common Job Scenarios

### Scenario 1: Blue-Green Deployment

```yaml
jobs:
- job: DeployGreen
  steps:
  - task: AzureWebApp@1
    inputs:
      appName: 'myapp-green'
      package: '$(Pipeline.Workspace)/drop/*.zip'
  
  - script: |
      # Run smoke tests on green environment
      curl https://myapp-green.azurewebsites.net/health

- job: SwapSlots
  dependsOn: DeployGreen
  condition: succeeded()
  pool: server  # Agentless
  steps:
  - task: ManualValidation@0
    inputs:
      instructions: 'Validate green environment, then approve to swap'
  
- job: SwapToProduction
  dependsOn: SwapSlots
  steps:
  - task: AzureAppServiceManage@0
    inputs:
      action: 'Swap Slots'
      sourceSlot: 'green'
      targetSlot: 'production'
```

### Scenario 2: Multi-Region Deployment

```yaml
jobs:
- job: DeployEastUS
  steps:
  - script: echo "Deploy to East US"

- job: DeployWestUS
  steps:
  - script: echo "Deploy to West US"

- job: DeployEurope
  steps:
  - script: echo "Deploy to Europe"

- job: UpdateTrafficManager
  dependsOn:
  - DeployEastUS
  - DeployWestUS
  - DeployEurope
  condition: succeeded()
  steps:
  - script: echo "Update Traffic Manager with new endpoints"
```

### Scenario 3: Database + Application Deployment

```yaml
jobs:
- job: DeployDatabase
  steps:
  - task: SqlAzureDacpacDeployment@1
    inputs:
      azureSubscription: 'AzureConnection'
      ServerName: 'myserver.database.windows.net'
      DatabaseName: 'mydb'
      SqlUsername: '$(dbUser)'
      SqlPassword: '$(dbPassword)'
      DacpacFile: '$(Pipeline.Workspace)/database/*.dacpac'

- job: DeployApplication
  dependsOn: DeployDatabase
  condition: succeeded()
  steps:
  - task: AzureWebApp@1
    inputs:
      appName: 'myapp'
      package: '$(Pipeline.Workspace)/drop/*.zip'
```

## Critical Notes

🎯 **Job Types**: Use Agent jobs for standard deployments, Deployment Group jobs for multi-server scenarios, Agentless for gates/approvals.

💡 **Deployment Groups**: Install agents on target servers and organize with tags for flexible, targeted deployments.

⚠️ **Dependencies**: Control execution order with `dependsOn`—jobs run in parallel by default unless dependencies specified.

📊 **Conditions**: Use conditions to control when jobs run (branch filters, previous job status, manual triggers).

🔄 **Parallel Limits**: Free tier has 1 parallel job—purchase additional for concurrent deployments.

✨ **Deployment Jobs**: Prefer deployment jobs over agent jobs for Kubernetes/VM deployments—better history and rollback support.

## Quick Reference

### Job Type Selection

```
Use Case → Job Type

Single server deployment → Agent Job
Multi-server deployment → Deployment Group Job
Manual approval needed → Agentless Job
Containerized deployment tools → Container Job
Kubernetes/VM with history → Deployment Job
```

### Common Job Patterns

```yaml
# Standard deployment
- job: Deploy
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - task: AzureWebApp@1

# With dependencies
- job: Deploy
  dependsOn: Test
  condition: succeeded()

# With timeout
- job: Deploy
  timeoutInMinutes: 30

# Parallel execution
- job: DeployRegion1
- job: DeployRegion2
# No dependsOn = parallel
```

[Learn More](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/deployment-jobs)
