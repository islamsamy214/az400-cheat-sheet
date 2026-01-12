# Summary

Congratulations! You've completed the **Create a Release Pipeline** module. You now understand release pipeline fundamentals, artifact management, deployment strategies, release jobs, database deployments, and multi-stage YAML pipelines—essential skills for implementing continuous delivery in Azure DevOps.

## Module Recap

### What You Learned

This module covered comprehensive release pipeline strategies:

**Release Pipeline Fundamentals** (Units 1-3):
- ✅ Release pipeline purpose and architecture
- ✅ Azure Pipelines release capabilities
- ✅ Multi-stage release pipelines (Dev, QA, Production)
- ✅ Continuous deployment triggers
- ✅ Approval gates and manual interventions

**Artifact Management** (Units 4-5):
- ✅ Artifact sources (builds, containers, packages, repositories)
- ✅ Artifact filtering and selection
- ✅ Multiple artifact sources in single release
- ✅ Artifact versioning strategies

**Deployment Considerations** (Unit 6):
- ✅ Environment-specific configuration
- ✅ Deployment slots (staging/production)
- ✅ Blue-green deployments
- ✅ Canary releases
- ✅ Rolling updates

**Build and Release Tasks** (Units 7-8):
- ✅ Common deployment tasks (Azure App Service, AKS, VMs)
- ✅ Task configuration and variables
- ✅ Custom build and release tasks
- ✅ Task groups for reusability

**Release Jobs** (Unit 9):
- ✅ Agent jobs for single-target deployment
- ✅ Deployment group jobs for multi-server scenarios
- ✅ Agentless jobs for approvals and gates
- ✅ Job dependencies and parallel execution

**Database Deployment** (Unit 10):
- ✅ DACPAC deployments with schema comparison
- ✅ SQL script execution
- ✅ Expand-Contract pattern for zero-downtime
- ✅ Blue-green database deployments
- ✅ Database deployment best practices

**YAML Pipelines** (Unit 11):
- ✅ Multi-stage YAML pipeline structure
- ✅ Deployment jobs and strategies (runOnce, rolling, canary)
- ✅ Environments and approval configuration
- ✅ Templates for reusability
- ✅ Version control for deployment configuration

## Learning Objectives Achieved

Reviewing the module's learning objectives:

### ✅ Objective 1: Understand Release Pipelines
**Achieved**: You can now design multi-stage release pipelines with:
- Stage progression (Dev → QA → Production)
- Continuous deployment triggers
- Approval gates at each stage
- Environment-specific configuration
- Deployment tracking and history

### ✅ Objective 2: Manage Artifacts
**Achieved**: You master artifact management:
- Connecting multiple artifact sources (builds, containers, packages)
- Filtering artifacts for specific stages
- Versioning strategies (latest, specific, tags)
- Managing artifact dependencies

### ✅ Objective 3: Configure Deployment Tasks
**Achieved**: You understand deployment tasks:
- Azure service deployments (App Service, AKS, Functions)
- Database deployments (DACPAC, SQL scripts)
- Custom task creation
- Task groups for reusability

### ✅ Objective 4: Implement Deployment Strategies
**Achieved**: You can implement:
- Blue-green deployments (instant switch)
- Canary releases (progressive rollout)
- Rolling updates (gradual multi-server)
- Deployment slots (staging/production)

### ✅ Objective 5: Deploy with YAML Pipelines
**Achieved**: You can create multi-stage YAML pipelines:
- Combining build and release in single pipeline
- Deployment jobs with strategies
- Environment configuration and approvals
- Templates for reusable deployment patterns

## Key Concepts Mastered

### Release Pipeline Architecture

```
Release Pipeline Flow

Build Pipeline (CI)
       ↓
   Artifacts
       ↓
┌──────────────────────────┐
│  Release Pipeline (CD)   │
│                          │
│  Stage: Dev              │
│    ├── Pre-deployment    │
│    ├── Deploy tasks      │
│    └── Post-deployment   │
│         ↓                │
│  Stage: QA               │
│    ├── Manual approval   │
│    ├── Deploy tasks      │
│    └── Integration tests │
│         ↓                │
│  Stage: Production       │
│    ├── Manual approval   │
│    ├── Deploy tasks      │
│    └── Smoke tests       │
└──────────────────────────┘
```

### Deployment Strategies Comparison

```
Strategy Selection Guide

┌─────────────────────────────────────────┐
│  Blue-Green Deployment                  │
│  ┌──────┐    ┌──────┐                  │
│  │ Blue │    │Green │                  │
│  │(Old) │ →  │(New) │                  │
│  └──────┘    └──────┘                  │
│  Use: Instant switch, easy rollback    │
│  Cost: 2x infrastructure               │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  Canary Deployment                      │
│  10% → 25% → 50% → 100%                 │
│  Monitor between each increment         │
│  Use: High-risk production changes      │
│  Risk: Low (early detection)            │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  Rolling Deployment                     │
│  Deploy 2 servers at a time             │
│  Batch 1 → Batch 2 → Batch 3            │
│  Use: Multi-server gradual rollout      │
│  Downtime: Partial (some servers down)  │
└─────────────────────────────────────────┘
```

### Database Deployment - Expand-Contract Pattern

```
Zero-Downtime Database Changes

Phase 1: Expand
┌────────────────────────┐
│ Users Table            │
│ ├── Email_Old (active) │ ← App uses this
│ └── Email_New (added)  │ ← New column (nullable)
└────────────────────────┘

Phase 2: Migrate Data
UPDATE Users SET Email_New = Email_Old

Phase 3: Deploy New App Version
┌────────────────────────┐
│ Users Table            │
│ ├── Email_Old          │ ← App no longer uses
│ └── Email_New (active) │ ← App now uses this
└────────────────────────┘

Phase 4: Contract
┌────────────────────────┐
│ Users Table            │
│ └── Email_New          │ ← Old column removed
└────────────────────────┘
```

### Multi-Stage YAML Pipeline

```yaml
# Complete release workflow in single YAML file

trigger:
  branches:
    include:
    - main

stages:
# Build stage
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: dotnet build
    - task: PublishBuildArtifacts@1

# Deploy to Dev
- stage: DeployDev
  dependsOn: Build
  jobs:
  - deployment: DeployWeb
    environment: Development
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1

# Deploy to QA (requires approval)
- stage: DeployQA
  dependsOn: DeployDev
  jobs:
  - deployment: DeployWeb
    environment: QA  # Approval configured on environment
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1

# Deploy to Prod (canary strategy)
- stage: DeployProd
  dependsOn: DeployQA
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
  jobs:
  - deployment: DeployWeb
    environment: Production
    strategy:
      canary:
        increments: [10, 25, 50, 100]
        deploy:
          steps:
          - task: AzureWebApp@1
```

## Real-World Application

### Complete Release Pipeline Example

**Scenario**: Deploy ASP.NET Core web application to Azure App Service through Dev, QA, and Production environments.

**Pipeline Components**:

1. **Build Pipeline** (CI):
   ```yaml
   - Build application
   - Run unit tests
   - Publish artifact
   ```

2. **Release Pipeline** (CD):
   ```yaml
   Stages:
   - Dev: Auto-deploy on new artifact
   - QA: Auto-deploy after Dev success
   - Production: Manual approval required
   ```

3. **Deployment Strategy**:
   - Dev: RunOnce (simple deployment)
   - QA: Deploy to staging slot, then swap
   - Production: Canary (10% → 25% → 50% → 100%)

**Multi-Stage YAML Implementation**:

```yaml
trigger:
  branches:
    include:
    - main

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  jobs:
  - job: BuildApp
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Build'
      inputs:
        command: 'build'
        arguments: '--configuration $(buildConfiguration)'
    
    - task: DotNetCoreCLI@2
      displayName: 'Test'
      inputs:
        command: 'test'
        arguments: '--configuration $(buildConfiguration)'
    
    - task: DotNetCoreCLI@2
      displayName: 'Publish'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
        zipAfterPublish: true
    
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
    displayName: 'Deploy Web App'
    environment: Development
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureWebApp@1
            displayName: 'Deploy to Dev'
            inputs:
              azureSubscription: 'AzureDevConnection'
              appName: 'myapp-dev'
              package: '$(Pipeline.Workspace)/drop/*.zip'
          
          - script: curl -f https://myapp-dev.azurewebsites.net/health
            displayName: 'Smoke Test'

- stage: DeployQA
  displayName: 'Deploy to QA'
  dependsOn: DeployDev
  jobs:
  - deployment: DeployWeb
    environment: QA  # Approval required (configured in Azure DevOps)
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
              WebAppName: 'myapp-qa'
              deployToSlotOrASE: true
              slotName: 'staging'
              package: '$(Pipeline.Workspace)/drop/*.zip'
          
          - script: curl -f https://myapp-qa-staging.azurewebsites.net/health
            displayName: 'Validate Staging'
          
          - task: AzureAppServiceManage@0
            displayName: 'Swap Slots'
            inputs:
              azureSubscription: 'AzureQAConnection'
              action: 'Swap Slots'
              WebAppName: 'myapp-qa'
              sourceSlot: 'staging'
              targetSlot: 'production'

- stage: DeployProd
  displayName: 'Deploy to Production'
  dependsOn: DeployQA
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployWeb
    environment: Production  # Requires approval
    strategy:
      canary:
        increments: [10, 25, 50, 100]
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureWebApp@1
            displayName: 'Deploy to Production'
            inputs:
              azureSubscription: 'AzureProdConnection'
              appName: 'myapp-prod'
              package: '$(Pipeline.Workspace)/drop/*.zip'
        
        postRouteTraffic:
          steps:
          - script: |
              # Monitor for 5 minutes
              sleep 300
              # Check error rate from Application Insights
              ERROR_RATE=$(curl -s "https://api.applicationinsights.io/v1/apps/APP_ID/metrics/requests/failed?timespan=PT5M" | jq '.value.sum')
              if [ "$ERROR_RATE" -gt 10 ]; then
                echo "Error rate too high: $ERROR_RATE"
                exit 1
              fi
            displayName: 'Monitor Canary'
```

## Best Practices Checklist

Use this checklist for every release pipeline you create:

### Pipeline Design
- [ ] Multi-stage pipeline (Dev, QA, Production)
- [ ] Continuous deployment trigger enabled
- [ ] Environment-specific configuration (variable groups)
- [ ] Approval gates for production
- [ ] Post-deployment validation (smoke tests)

### Artifact Management
- [ ] Artifact filtering (download only needed artifacts)
- [ ] Version strategy defined (latest, specific, tags)
- [ ] Multiple artifact sources if needed (builds, containers, packages)
- [ ] Artifact retention policy configured

### Deployment Strategy
- [ ] Appropriate strategy per environment (runOnce, rolling, canary)
- [ ] Deployment slots for zero-downtime (staging/production)
- [ ] Rollback plan documented
- [ ] Health checks after deployment

### Database Deployment
- [ ] DACPAC for schema deployments
- [ ] SQL scripts for data migrations
- [ ] Expand-Contract pattern for breaking changes
- [ ] Backup before production deployment
- [ ] Idempotent scripts (safe to run multiple times)

### Security and Compliance
- [ ] Service principals for Azure authentication
- [ ] Secrets stored in Azure Key Vault
- [ ] Variable groups with sensitive data marked as secret
- [ ] Approval required for production
- [ ] Audit log review process

### Monitoring and Alerting
- [ ] Application Insights integration
- [ ] Deployment notifications (email, Slack, Teams)
- [ ] Error rate monitoring in canary deployments
- [ ] Post-deployment health checks
- [ ] Rollback trigger conditions defined

## Common Pitfalls to Avoid

### ❌ Mistake 1: No Approval Gates for Production
**Problem**: Automatic deployment to production without review.
**Solution**: Configure manual approval on Production environment:
```
Environments > Production > Approvals and checks > Add Approvals
```

### ❌ Mistake 2: Deploying Application Before Database
**Problem**: New application version fails because database schema not updated.
**Solution**: Always deploy database first:
```yaml
- stage: DeployDatabase
  jobs:
  - deployment: DeployDB

- stage: DeployApplication
  dependsOn: DeployDatabase  # Wait for database
```

### ❌ Mistake 3: No Rollback Strategy
**Problem**: Deployment fails, no plan to revert.
**Solution**: Use deployment slots or maintain previous version:
```yaml
# Blue-Green with slots
- Deploy to staging slot
- Validate staging
- Swap to production
# Rollback: Swap back to staging
```

### ❌ Mistake 4: Hardcoded Environment Values
**Problem**: Connection strings hardcoded in tasks.
**Solution**: Use variable groups per environment:
```
Dev variable group: connectionString = "dev-db"
QA variable group: connectionString = "qa-db"
Prod variable group: connectionString = "prod-db"
```

### ❌ Mistake 5: No Post-Deployment Validation
**Problem**: Deployment succeeds but application broken.
**Solution**: Add smoke tests after deployment:
```yaml
- script: curl -f https://myapp.azurewebsites.net/health || exit 1
  displayName: 'Health Check'
```

### ❌ Mistake 6: Not Testing Database Migrations
**Problem**: Schema changes fail in production.
**Solution**: Test on shadow database first:
```yaml
- Deploy DACPAC to shadow database (copy of prod)
- Validate migration success
- Run integration tests on shadow
- If success: Deploy to production
```

## Exam Tips (AZ-400)

For the AZ-400 DevOps Engineer Expert exam:

**Release Pipeline Concepts**:
- Understand multi-stage pipelines (Dev, QA, Prod)
- Know continuous deployment triggers
- Approval gates and manual interventions
- Environment-specific configuration

**Artifact Management**:
- Multiple artifact sources (builds, containers, packages)
- Artifact filtering and versioning strategies
- Default version options (latest, specific, tags)

**Deployment Strategies**:
- **Blue-Green**: Instant switch, 2x infrastructure
- **Canary**: Progressive rollout with monitoring
- **Rolling**: Gradual multi-server deployment
- **RunOnce**: Simple single deployment

**Release Jobs**:
- Agent job: Single target
- Deployment group job: Multi-server
- Agentless job: Approvals, gates
- Container job: Containerized tools

**Database Deployment**:
- DACPAC: Schema deployments with comparison
- SQL scripts: Data migrations
- Expand-Contract: Zero-downtime pattern
- Idempotent scripts: Safe to run multiple times

**YAML Pipelines**:
- Multi-stage structure (stages, jobs, steps)
- Deployment jobs with strategies
- Environments for approval configuration
- Templates for reusability

## Tools and Resources

### Essential Tools
- **Azure DevOps**: Release pipeline platform
- **Azure CLI**: Command-line Azure management
- **Azure Portal**: Service configuration
- **Git**: Version control for YAML pipelines
- **Visual Studio Code**: YAML editing with Azure Pipelines extension

### Documentation Links
- [Azure Pipelines Release Documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/)
- [Multi-Stage YAML Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/stages)
- [Deployment Jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/deployment-jobs)
- [Deployment Strategies](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/deployment-groups/)
- [Database Deployment](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/sql-azure-dacpac-deployment)

## Next Steps

### Continue Learning Path 3

**Within Learning Path 3** (Design and implement a release strategy):
- ✅ Module 1: Create a release pipeline (Complete) ← **YOU ARE HERE!**
- ⏳ Module 2: Use release strategy recommendations
- ⏳ Module 3: Configure and provision environments
- ⏳ Module 4: Manage and modularize tasks and templates
- ⏳ Module 5: Automate inspection of health

**Next Module**: Module 2 - Use release strategy recommendations
- Release notes and documentation
- Release patterns (blue-green, canary, progressive exposure)
- Feature flags for controlled rollout
- Testing strategy in production

### Deepen Your Knowledge

**Practice Hands-On**:
- Create multi-stage release pipeline
- Deploy to Azure App Service with deployment slots
- Implement canary deployment with monitoring
- Deploy database with DACPAC
- Convert Classic release to YAML pipeline

**Advanced Topics**:
- Infrastructure as Code (Terraform, Bicep) in pipelines
- Kubernetes deployments with Helm
- Service mesh integration (Istio, Linkerd)
- Advanced approval gates (Azure Functions, REST APIs)

## Module Completion Badge

```
╔══════════════════════════════════════════╗
║                                          ║
║   🎓 MODULE 1 COMPLETE 🎓                ║
║                                          ║
║   Create a Release Pipeline              ║
║                                          ║
║   ✅ Release pipeline fundamentals       ║
║   ✅ Artifact management                 ║
║   ✅ Deployment strategies               ║
║   ✅ Release jobs and deployment groups  ║
║   ✅ Database deployment patterns        ║
║   ✅ Multi-stage YAML pipelines          ║
║                                          ║
║   Ready for Module 2! 🚀                 ║
║                                          ║
╚══════════════════════════════════════════╝
```

**Thank you for completing this module!** Continue your DevOps journey with Module 2: Use release strategy recommendations.

---

## Quick Reference Card

### Most Important Concepts

```yaml
# Multi-Stage Release Pipeline
stages:
- stage: Build
  jobs:
  - job: BuildJob

- stage: DeployDev
  dependsOn: Build
  jobs:
  - deployment: DeployWeb
    environment: Development
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1

- stage: DeployProd
  dependsOn: DeployDev
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
  jobs:
  - deployment: DeployWeb
    environment: Production  # Approval required
    strategy:
      canary:
        increments: [10, 25, 50, 100]
```

### Deployment Strategy Selection

```
Low risk, single target → RunOnce
Multiple servers, gradual → Rolling
High risk, progressive monitoring → Canary
Instant switch, easy rollback → Blue-Green
```

### Database Deployment

```yaml
# DACPAC deployment
- task: SqlAzureDacpacDeployment@1
  inputs:
    ServerName: 'myserver.database.windows.net'
    DatabaseName: 'MyDatabase'
    DacpacFile: '$(Pipeline.Workspace)/database/db.dacpac'
    AdditionalArguments: '/p:BlockOnPossibleDataLoss=true'
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline/)
