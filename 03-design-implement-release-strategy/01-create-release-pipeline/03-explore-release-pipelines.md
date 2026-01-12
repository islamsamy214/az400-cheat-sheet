# Explore Release Pipelines

A release pipeline is the automated workflow that takes build artifacts and deploys them through multiple stages (environments) until reaching production. Understanding release pipeline components is fundamental to designing effective continuous delivery strategies.

## Release Pipeline Overview

```
Complete Release Pipeline Workflow

Build Pipeline                    Release Pipeline
      ↓                                 ↓
┌─────────────┐         ┌───────────────────────────────────────┐
│   Build     │────────→│  Artifacts  (immutable packages)      │
│  Artifacts  │         └───────────────────────────────────────┘
└─────────────┘                         ↓
                           ┌────────────────────────────┐
                           │  Triggers                  │
                           │  - Manual                  │
                           │  - Scheduled               │
                           │  - Continuous Deployment   │
                           └────────────────────────────┘
                                        ↓
                   ┌────────────────────┴────────────────────┐
                   ↓                                         ↓
        ┌──────────────────────┐              ┌──────────────────────┐
        │  Stage: Development  │              │  Stage: QA           │
        │  - Auto-deploy       │──────────────→  - Manual approval   │
        │  - Run smoke tests   │              │  - Integration tests │
        └──────────────────────┘              │  - Quality gates     │
                                              └──────────────────────┘
                                                        ↓
                                              ┌──────────────────────┐
                                              │  Stage: Production   │
                                              │  - Manual approval   │
                                              │  - Release gates     │
                                              │  - Blue-green deploy │
                                              └──────────────────────┘
```

## Release Pipeline Components

### 1. Artifacts

**Definition**: Deployable components of your application—immutable packages produced by build pipelines.

**Artifact Types**:
- Compiled applications (DLLs, JARs, executables)
- Web application packages (ZIP, TAR)
- Container images (Docker images in ACR, Docker Hub)
- NuGet/npm/Maven packages
- Infrastructure as Code files (Terraform, ARM templates)
- Database migration scripts

**Immutability Principle** 🔒:

```
Build Once, Deploy Many

┌─────────────────────────────────────────────────────────┐
│  Build Pipeline (Once)                                  │
│  ├── Compile code                                       │
│  ├── Run tests                                          │
│  ├── Package application → myapp-1.2.3.zip              │
│  └── Publish to artifact store                          │
└─────────────────────────────────────────────────────────┘
                      ↓ (Same artifact!)
┌─────────────────────┬─────────────────┬────────────────┐
│  Deploy to Dev      │  Deploy to QA   │  Deploy to Prod│
│  myapp-1.2.3.zip    │  myapp-1.2.3.zip│  myapp-1.2.3.zip│
│  (Config: dev.json) │  (Config: qa)   │  (Config: prod) │
└─────────────────────┴─────────────────┴────────────────┘
```

**Key Principles**:
- ✅ **Never rebuild** between environments—same binary deployed everywhere
- ✅ **Configuration changes only**—environment-specific settings applied at deployment
- ✅ **Traceable**—artifact links back to specific build and source commit
- ✅ **Immutable**—artifact contents never change after creation

**Example Artifact Metadata**:
```
Artifact: WebApp-Build-1234
├── Version: 1.2.3
├── Build ID: 1234
├── Source Branch: refs/heads/main
├── Source Commit: abc123def456
├── Build Date: 2026-01-12T10:30:00Z
└── Files:
    ├── webapp.zip (15.2 MB)
    ├── database-migrations.sql
    └── deployment-scripts/
```

### 2. Triggers

**Definition**: Events that initiate a new release from the pipeline.

#### Trigger Types

**A. Manual Triggers**

User-initiated releases for controlled deployments:

```
Use Cases:
✅ Production deployments requiring explicit approval
✅ Hotfix releases outside normal schedule
✅ Deployments during maintenance windows
✅ Rollback scenarios

Example:
Release Manager clicks "Create Release" button
→ Selects specific artifact version
→ Chooses target stages
→ Provides release notes
→ Triggers deployment
```

**B. Scheduled Triggers**

Time-based automatic releases:

```
Configuration Examples:

Daily Off-Hours Deployment:
  Schedule: Every day at 2:00 AM UTC
  Days: Monday-Friday
  Stages: QA environment
  
Weekly Production Deployment:
  Schedule: Every Saturday at 8:00 PM UTC
  Days: Saturday
  Stages: Production (after approvals)
  
Sprint Deployment:
  Schedule: Last Friday of sprint at 5:00 PM
  Frequency: Every 2 weeks
  Stages: Staging → Production
```

**YAML Example**:
```yaml
schedules:
- cron: "0 2 * * 1-5"  # 2 AM UTC, Monday-Friday
  displayName: Daily QA Deployment
  branches:
    include:
    - main
  always: true  # Run even if no code changes
```

**C. Continuous Deployment Triggers**

Event-driven automatic releases:

```
Trigger Conditions:

1. Build Completion Trigger:
   Event: Build pipeline completes successfully
   Action: Automatically create and deploy release
   
2. Artifact Published Trigger:
   Event: New artifact version available
   Action: Deploy to first stage (e.g., Dev)
   
3. Branch Filter:
   Event: Build from main branch completes
   Action: Deploy to staging
   
   Event: Build from feature/* branches completes
   Action: Deploy to dev environment only
```

**Classic Release Pipeline Trigger Configuration**:
```
Continuous Deployment Trigger:
✅ Enabled
Artifact Source: MyApp-CI
Branch Filters:
  ✅ main → Deploy to Dev, QA, Production
  ✅ release/* → Deploy to Staging, Production
  ❌ feature/* → No automatic deployment
```

**YAML Multi-Stage Pipeline**:
```yaml
trigger:
  branches:
    include:
    - main
    - release/*
  paths:
    include:
    - src/*
    exclude:
    - docs/*  # Don't trigger on documentation changes
```

### 3. Stages (Environments)

**Definition**: Deployment target environments where artifacts are installed and validated.

#### Common Stage Patterns

```
Simple Pipeline:
Dev → QA → Production

Complex Pipeline:
Dev → Integration → QA → UAT → Staging → Production

Microservices Pipeline:
Dev → Test → Staging-A → Staging-B → Canary → Production
```

#### Stage Characteristics

| Stage | Purpose | Approval | Testing | Downtime Tolerance |
|-------|---------|----------|---------|-------------------|
| **Development** | Rapid iteration, developer testing | None | Unit, smoke tests | High |
| **Integration** | Component integration testing | None | Integration tests | High |
| **QA** | Quality assurance testing | QA Lead | Full test suite | Medium |
| **UAT** | User acceptance testing | Business owner | User scenarios | Medium |
| **Staging** | Production replica, final validation | Release manager | Performance, security | Low |
| **Production** | Live customer-facing environment | Change board | Smoke tests, monitoring | Very low |

#### Stage Configuration

**Environment-Specific Variables**:
```yaml
variables:
- group: Dev-Variables  # Contains: DbConnectionString, ApiKey, etc.

stages:
- stage: Dev
  variables:
    Environment: 'Development'
    AppServiceName: 'myapp-dev'
    DatabaseServer: 'dev-sql.database.windows.net'
    
- stage: Prod
  variables:
    Environment: 'Production'
    AppServiceName: 'myapp-prod'
    DatabaseServer: 'prod-sql.database.windows.net'
```

### 4. Approvals

**Definition**: Manual or automated checkpoints before proceeding to the next stage.

#### Approval Types

**Pre-Deployment Approvals**:
```
Before Stage Execution:
┌────────────────────────────────┐
│  Artifact Ready                │
│         ↓                      │
│  [Pre-Deployment Approval]     │ ← Must approve before deployment
│         ↓                      │
│  Deploy to Stage               │
│         ↓                      │
│  Run Tests                     │
└────────────────────────────────┘
```

**Post-Deployment Approvals**:
```
After Stage Execution:
┌────────────────────────────────┐
│  Deploy to Stage               │
│         ↓                      │
│  Run Tests                     │
│         ↓                      │
│  [Post-Deployment Approval]    │ ← Must approve before next stage
│         ↓                      │
│  Proceed to Next Stage         │
└────────────────────────────────┘
```

**Approval Configuration Example**:
```
Stage: Production
Pre-Deployment Approvals:
  ✅ Required Approvers:
     - Release Manager (required)
     - Operations Lead (required)
     - Security Team (1 of 3 members)
  ✅ Timeout: 7 days (auto-reject after timeout)
  ✅ Instructions: "Verify change ticket #XYZ approved by CAB"

Post-Deployment Approvals:
  ✅ Required Approvers:
     - Application Owner
  ✅ Timeout: 24 hours
  ✅ Instructions: "Confirm application health after production deployment"
```

**YAML Approval (via Environments)**:
```yaml
stages:
- stage: Production
  jobs:
  - deployment: DeployProd
    environment: production  # Environment has approval configured in UI
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
```

### 5. Release Gates

**Definition**: Automated quality checks that must pass before deployment proceeds.

```
Release Gate Workflow

Stage Ready to Deploy
        ↓
┌─────────────────────────────────────┐
│  Pre-Deployment Gates               │
│  ├── Check Azure Monitor Alerts     │ ← No active alerts?
│  ├── Query Work Items               │ ← No P0/P1 bugs?
│  ├── Invoke REST API                │ ← External system healthy?
│  └── Check Function Health          │ ← Health endpoint returns 200?
└─────────────────────────────────────┘
        ↓ (All gates passed)
┌─────────────────────────────────────┐
│  Deploy to Stage                    │
└─────────────────────────────────────┘
```

#### Common Gate Types

**1. Azure Monitor Alerts Gate**:
```
Configuration:
  Source: Azure Monitor
  Query: Check for active alerts in last 1 hour
  Condition: No alerts with severity Critical or Warning
  Timeout: 15 minutes (recheck every 5 minutes)
  
Example: Don't deploy if Application Insights shows errors
```

**2. Work Item Query Gate**:
```
Configuration:
  Source: Azure Boards
  Query: Active bugs with Priority 1 or 0
  Condition: Count = 0
  Timeout: 30 minutes
  
Example: Don't deploy if there are unresolved P0/P1 bugs
```

**3. REST API Gate**:
```
Configuration:
  URL: https://api.healthcheck.com/status
  Method: GET
  Headers: Authorization: Bearer ${token}
  Success Criteria: $.status == "healthy"
  Timeout: 10 minutes
  
Example: Check external dependency health before deployment
```

**4. Azure Function Gate (Custom Logic)**:
```
Configuration:
  Function App: quality-gates-functions
  Function: CheckDeploymentReadiness
  Parameters: { "environment": "production" }
  Success Criteria: Response code 200
  
Example: Custom business logic to validate deployment readiness
```

#### Gate Evaluation Process

```
Gate Evaluation Timeline

T=0:  Gates start evaluation
      ↓
T=5:  First evaluation (all gates checked)
      ↓ Some gates failed
T=10: Second evaluation
      ↓ Azure Monitor gate passed, Work Item gate still failing
T=15: Third evaluation
      ↓ All gates passed!
      
Proceed to Deployment ✅

If timeout reached (e.g., 30 minutes):
  → All gates must pass or deployment fails ❌
```

### 6. Tasks

**Definition**: Individual deployment steps that perform specific actions.

#### Common Deployment Tasks

**Azure Tasks**:
```yaml
# Deploy to Azure App Service
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Production'
    appName: 'myapp-prod'
    package: '$(Pipeline.Workspace)/drop/webapp.zip'

# Deploy to Azure Kubernetes Service
- task: KubernetesManifest@0
  inputs:
    action: 'deploy'
    kubernetesServiceConnection: 'AKS-Prod'
    manifests: 'k8s/deployment.yml'

# Azure SQL Database Deployment
- task: SqlAzureDacpacDeployment@1
  inputs:
    azureSubscription: 'Production'
    serverName: 'prod-sql.database.windows.net'
    databaseName: 'MyAppDB'
    sqlFile: 'migrations/schema-update.sql'
```

**Generic Tasks**:
```yaml
# Copy files
- task: CopyFiles@2
  inputs:
    sourceFolder: '$(Build.SourcesDirectory)'
    contents: '**/*.config'
    targetFolder: '$(Pipeline.Workspace)/config'

# PowerShell script
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      Write-Host "Deploying to $(Environment)"
      ./deploy-script.ps1 -Environment $(Environment)

# Run integration tests
- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    projects: '**/*IntegrationTests.csproj'
    arguments: '--configuration Release'
```

## Release Pipeline Best Practices

### 1. Immutable Artifacts

**✅ DO**:
```
Build Pipeline: myapp-1.2.3.zip (never changes)
  ↓
Dev:  Deploy myapp-1.2.3.zip + dev-config.json
QA:   Deploy myapp-1.2.3.zip + qa-config.json
Prod: Deploy myapp-1.2.3.zip + prod-config.json
```

**❌ DON'T**:
```
Dev:  Build and deploy (version 1.2.3)
QA:   Rebuild and deploy (version 1.2.3 but different bits!)
Prod: Rebuild again (version 1.2.3 but who knows what's inside?)
```

### 2. Progressive Exposure

```
Deployment Strategy:

1. Deploy to Dev (all developers)
2. Deploy to QA (QA team, limited users)
3. Deploy to Staging (production-like, internal users)
4. Deploy to Production (canary: 5% of users)
5. Deploy to Production (full: 100% of users)
```

### 3. Automated Quality Gates

**Replace manual checks with automated gates**:
```
Manual (Slow):
  Release Manager checks monitoring dashboard
  Release Manager asks team "Any issues?"
  Release Manager manually approves
  
Automated (Fast):
  Azure Monitor gate: No alerts ✅
  Work Item gate: No P0/P1 bugs ✅
  Health Check gate: All services healthy ✅
  Automatic approval when gates pass ✅
```

### 4. Rollback Strategy

```
Deployment Strategies for Easy Rollback:

Blue-Green Deployment:
  Blue (current): v1.2.2 (serving traffic)
  Green (new): v1.2.3 (deployed, tested, ready)
  Switch: Route traffic Blue → Green
  Rollback: Route traffic Green → Blue (instant)

Canary Deployment:
  v1.2.2: 95% of traffic
  v1.2.3: 5% of traffic (canary)
  Monitor canary health
  If healthy: Gradually increase to 100%
  If unhealthy: Route 100% back to v1.2.2
```

## Complete Release Pipeline Example

**Scenario**: Deploy ASP.NET Core web application to Azure App Service

```yaml
# Classic Release Pipeline (conceptual YAML representation)

trigger: none  # Triggered by build pipeline completion

resources:
  pipelines:
  - pipeline: buildPipeline
    source: MyApp-CI
    trigger:
      branches:
        include:
        - main

stages:
- stage: Deploy_Dev
  displayName: 'Deploy to Development'
  jobs:
  - deployment: DeployWeb
    environment: dev
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Dev'
              appName: 'myapp-dev'
              package: '$(Pipeline.Workspace)/buildPipeline/drop/webapp.zip'
          
          - task: PowerShell@2
            displayName: 'Run Smoke Tests'
            inputs:
              targetType: 'inline'
              script: |
                Invoke-WebRequest -Uri https://myapp-dev.azurewebsites.net/health

- stage: Deploy_QA
  displayName: 'Deploy to QA'
  dependsOn: Deploy_Dev
  condition: succeeded()
  jobs:
  - deployment: DeployWeb
    environment: qa  # Manual approval configured in environment
    strategy:
      runOnce:
        preDeploy:
          steps:
          - script: echo "Pre-deployment validation"
        
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'QA'
              appName: 'myapp-qa'
              package: '$(Pipeline.Workspace)/buildPipeline/drop/webapp.zip'
        
        postDeploy:
          steps:
          - task: DotNetCoreCLI@2
            displayName: 'Run Integration Tests'
            inputs:
              command: 'test'
              projects: '**/*IntegrationTests.csproj'

- stage: Deploy_Production
  displayName: 'Deploy to Production'
  dependsOn: Deploy_QA
  condition: succeeded()
  jobs:
  - deployment: DeployWeb
    environment: production  # Manual approval + gates configured
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Production'
              appName: 'myapp-prod'
              package: '$(Pipeline.Workspace)/buildPipeline/drop/webapp.zip'
              deploymentMethod: 'zipDeploy'
          
          - task: PowerShell@2
            displayName: 'Verify Deployment'
            inputs:
              targetType: 'inline'
              script: |
                $response = Invoke-WebRequest -Uri https://myapp-prod.azurewebsites.net/health
                if ($response.StatusCode -ne 200) {
                  throw "Health check failed!"
                }
```

## Critical Notes

🎯 **Immutable Artifacts**: Build once, deploy many—same binary in all environments with configuration-only changes.

💡 **Progressive Deployment**: Deploy to least critical environments first (dev → QA → staging → production).

⚠️ **Approval Workflows**: Balance speed and safety—automate where possible, require manual approval for production.

📊 **Quality Gates**: Automated gates catch issues before production—no alerts, no bugs, healthy dependencies.

🔄 **Rollback Strategy**: Always have a rollback plan—blue-green, canary, or versioned deployments.

✨ **Traceability**: Every release must link back to source commit and build—full audit trail required.

## Quick Reference

### Release Pipeline Components

```
Essential Elements:
┌─────────────────────────────────────┐
│  1. Artifacts (immutable packages)  │
│  2. Triggers (manual/scheduled/CD)  │
│  3. Stages (dev/QA/prod)            │
│  4. Approvals (manual checkpoints)  │
│  5. Gates (automated checks)        │
│  6. Tasks (deployment steps)        │
└─────────────────────────────────────┘
```

### Deployment Decision Tree

```
How should I deploy?

Manual control needed? → Manual trigger
Deploy at specific time? → Scheduled trigger
Deploy immediately after build? → Continuous deployment trigger

Requires approval? → Pre-deployment approval
Need validation after deploy? → Post-deployment approval

Automated quality checks? → Release gates
- No active alerts? → Azure Monitor gate
- No critical bugs? → Work Item gate
- External service healthy? → REST API gate
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline-devops/3-explore-release-pipelines)
