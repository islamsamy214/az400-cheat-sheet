# Describe Azure Pipelines Capabilities

Azure DevOps provides two primary approaches for implementing release pipelines: **classic release pipelines** (UI-based) and **YAML multi-stage pipelines** (code-based). Understanding the capabilities and differences between these approaches is crucial for choosing the right solution for your organization.

## Pipeline Approaches

### Classic Release Pipelines

**UI-Based Release Management** introduced in Azure DevOps (formerly VSTS):

```
Classic Release Pipeline

Artifacts → [Visual Designer] → Stages → Approvals → Gates → Deployments
             ├── Drag-and-drop tasks
             ├── Visual stage configuration
             ├── Built-in approval UI
             └── Deployment group support
```

**Characteristics**:
- UI-based configuration (point-and-click)
- Separate from build pipelines
- Rich approval and gate features
- Deployment group support for on-premises servers
- Release dashboard with deployment history
- Not stored in source control (stored in Azure DevOps)

### YAML Multi-Stage Pipelines

**Pipelines as Code** approach with build and release in single YAML file:

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: npm run build

- stage: Deploy_Dev
  dependsOn: Build
  jobs:
  - deployment: DeployWeb
    environment: dev
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1

- stage: Deploy_Prod
  dependsOn: Deploy_Dev
  jobs:
  - deployment: DeployWeb
    environment: production
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
```

**Characteristics**:
- Code-based configuration (YAML)
- Build + release in single pipeline
- Stored in source control (Git)
- Version controlled with your code
- Templates for reusability
- Multi-stage support (build, test, deploy in one pipeline)

## Feature Comparison: YAML vs Classic

### Comprehensive Capabilities Table

| Capability | YAML Pipeline | Classic Build | Classic Release | Description |
|------------|---------------|---------------|-----------------|-------------|
| **Agents** | ✅ Yes | ✅ Yes | ✅ Yes | Specifies a required resource on which the pipeline runs |
| **Approvals** | ✅ Yes | ❌ No | ✅ Yes | Defines validations required before completing a deployment stage |
| **Artifacts** | ✅ Yes | ✅ Yes | ✅ Yes | Supports publishing or consuming different package types |
| **Caching** | ✅ Yes | ✅ Yes | ❌ No | Reduces build time by reusing outputs/dependencies from previous runs |
| **Conditions** | ✅ Yes | ✅ Yes | ✅ Yes | Specifies conditions to be met before running a job |
| **Container jobs** | ✅ Yes | ❌ No | ❌ No | Specifies jobs to run in a container |
| **Demands** | ✅ Yes | ✅ Yes | ✅ Yes | Ensures pipeline requirements are met (requires self-hosted agents) |
| **Dependencies** | ✅ Yes | ✅ Yes | ✅ Yes | Specifies requirement that must be met to run next job/stage |
| **Deployment groups** | ❌ No | ❌ No | ✅ Yes | Defines logical set of deployment target machines (on-prem) |
| **Deployment group jobs** | ❌ No | ❌ No | ✅ Yes | Specifies job to release to a deployment group |
| **Deployment jobs** | ✅ Yes | ❌ No | ❌ No | Defines deployment steps (multi-stage pipelines feature) |
| **Environment** | ✅ Yes | ❌ No | ❌ No | Represents collection of resources targeted for deployment |
| **Gates** | ❌ No | ❌ No | ✅ Yes | Automatic collection and evaluation of external health signals |
| **Jobs** | ✅ Yes | ✅ Yes | ✅ Yes | Defines execution sequence of a set of steps |
| **Service connections** | ✅ Yes | ✅ Yes | ✅ Yes | Enables connection to remote services (Azure, AWS, etc.) |
| **Service containers** | ✅ Yes | ❌ No | ❌ No | Manages lifecycle of containerized services |
| **Stages** | ✅ Yes | ❌ No | ✅ Yes | Organizes jobs within a pipeline |
| **Task groups** | ❌ No | ✅ Yes | ✅ Yes | Encapsulates sequence of tasks into single reusable task |
| **Tasks** | ✅ Yes | ✅ Yes | ✅ Yes | Defines building blocks that make up a pipeline |
| **Templates** | ✅ Yes | ❌ No | ❌ No | Defines reusable content, logic, and parameters |
| **Triggers** | ✅ Yes | ✅ Yes | ✅ Yes | Defines event that causes a pipeline to run |
| **Variables** | ✅ Yes | ✅ Yes | ✅ Yes | Represents value to be replaced by data passed to pipeline |
| **Variable groups** | ✅ Yes | ✅ Yes | ✅ Yes | Store values to control and make available across pipelines |

## When to Use Each Approach

### Use Classic Release Pipelines When:

**✅ Recommended Scenarios**:

1. **Complex Approval Workflows**
   - Multiple approval stages with different approvers
   - Pre-deployment and post-deployment approvals
   - Manual intervention requirements
   ```
   Example: Production deployment requires:
   - Change advisory board approval
   - Security team review
   - Database admin approval for schema changes
   ```

2. **Deployment Groups** (On-Premises Servers)
   - Deploying to multiple on-premises servers
   - Rolling deployments across server farms
   - IIS, Windows Service, or on-prem application deployments
   ```
   Example: Deploy web app to 20 on-prem IIS servers
   with 5-server rolling deployment strategy
   ```

3. **Rich Release Gates**
   - Azure Monitor alerts integration
   - Work item query gates
   - REST API health checks
   - SonarQube quality gate integration
   ```
   Example: Production gate checks:
   - No active incidents in ServiceNow
   - Azure Application Insights error rate < 1%
   - SonarQube quality gate passed
   ```

4. **Visual Release Tracking**
   - Team prefers visual pipeline designer
   - Need release dashboard with deployment history
   - Stakeholder visibility into releases

**Example Classic Release Pipeline**:
```
Artifacts: WebApp Build (Build #1234)
    ↓
Stage: Dev
    ├── Auto-deploy on artifact available
    ├── Deploy to Azure App Service (Dev)
    └── Run smoke tests
    ↓
Stage: QA  
    ├── Manual approval required (QA Lead)
    ├── Deploy to Azure App Service (QA)
    ├── Run integration tests
    └── Gate: Work item query (no active bugs)
    ↓
Stage: Production
    ├── Manual approval required (Release Manager)
    ├── Gate: Azure Monitor (no alerts in last 1 hour)
    ├── Deploy to Azure App Service (Prod) - Blue/Green
    ├── Run smoke tests
    └── Post-deployment approval (Operations Team)
```

### Use YAML Multi-Stage Pipelines When:

**✅ Recommended Scenarios**:

1. **Infrastructure as Code Philosophy**
   - Pipeline configuration version controlled with app code
   - Code review process for pipeline changes
   - Easy branching and merging of pipeline definitions
   ```yaml
   # Stored in repository: azure-pipelines.yml
   # Changes reviewed via pull requests
   # Branched with application code
   ```

2. **Simple Deployment Flows**
   - Straightforward dev → staging → production
   - Automated testing without complex gates
   - Container-based deployments (Docker, Kubernetes)

3. **Template Reusability**
   - Shared pipeline templates across multiple projects
   - Standardized deployment patterns
   - DRY (Don't Repeat Yourself) principle
   ```yaml
   # Template: deploy-template.yml
   parameters:
     environment: ''
   
   steps:
   - task: AzureWebApp@1
     inputs:
       azureSubscription: ${{ parameters.environment }}
   ```

4. **Modern Application Stacks**
   - Microservices architectures
   - Container-based applications
   - Cloud-native applications

**Example YAML Multi-Stage Pipeline**:
```yaml
trigger:
  branches:
    include:
    - main

stages:
- stage: Build
  jobs:
  - job: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: Docker@2
      inputs:
        command: buildAndPush
        repository: myapp
        tags: $(Build.BuildId)

- stage: Deploy_Dev
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployDev
    environment: dev
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            inputs:
              action: deploy
              manifests: k8s/deployment.yml

- stage: Deploy_Prod
  dependsOn: Deploy_Dev
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployProd
    environment: production  # Manual approval configured in environment
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            inputs:
              action: deploy
              manifests: k8s/deployment.yml
```

## Hybrid Approach

**Combine Both for Maximum Flexibility**:

```
Build: YAML Pipeline (in source control)
    ↓ Produces artifacts
Release: Classic Release Pipeline (for complex approvals/gates)
    ↓ Deploys artifacts through stages
```

**Example Hybrid Workflow**:
1. **YAML Build Pipeline**: Build, test, package, publish artifacts (version controlled)
2. **Classic Release Pipeline**: Deploy artifacts with complex approval workflows and gates
3. **Best of Both Worlds**: Code-based build + UI-based release management

## Migration Path: Classic to YAML

**Phased Migration Strategy**:

```
Phase 1: Keep Classic Release
├── Use YAML for build pipeline
├── Keep classic release for deployment
└── Team learns YAML syntax

Phase 2: Simple Environments to YAML
├── Migrate dev/QA stages to YAML
├── Keep production in classic release
└── Validate YAML deployment process

Phase 3: Full YAML Migration
├── Implement approvals in YAML environments
├── Replace gates with YAML checks
└── Retire classic release pipeline
```

## Feature Parity Progress

Microsoft is actively working on feature parity between YAML and classic pipelines:

**Recently Added to YAML**:
- ✅ Deployment jobs (environments with approval)
- ✅ Multi-stage pipelines
- ✅ Environment resources
- ✅ Deployment strategies (runOnce, rolling, canary)

**Still Classic-Only**:
- ⏳ Deployment groups (on-premises)
- ⏳ Rich gate UI with multiple gate types
- ⏳ Post-deployment approvals

**Workarounds in YAML**:
```yaml
# Manual approval via environments
- stage: Production
  jobs:
  - deployment: Deploy
    environment: production  # Configure manual approval in UI
    
# Quality gates via checks
# Configure checks in environment settings:
# - Invoke REST API
# - Query Azure Monitor
# - Query Work Items
```

## Decision Matrix

| Requirement | Recommendation |
|-------------|----------------|
| **Deployment groups** (on-prem servers) | Classic Release |
| **Complex approval workflows** (3+ approval stages) | Classic Release |
| **Rich gates** (Azure Monitor, work items, REST APIs) | Classic Release |
| **Infrastructure as code** | YAML Multi-Stage |
| **Container deployments** (Docker, K8s) | YAML Multi-Stage |
| **Template reusability** | YAML Multi-Stage |
| **New projects** | YAML Multi-Stage |
| **Legacy applications** (on-prem) | Classic Release |
| **Team prefers visual designer** | Classic Release |
| **Code review for pipeline changes** | YAML Multi-Stage |

## Critical Notes

🎯 **Feature Parity**: YAML pipelines are catching up but classic release still has unique features (deployment groups, rich gates).

💡 **Not Mutually Exclusive**: Can use YAML build + classic release for hybrid approach.

⚠️ **Migration Complexity**: Migrating complex classic releases to YAML requires careful planning for approvals and gates.

📊 **Future Direction**: Microsoft is investing heavily in YAML pipelines—expect feature parity eventually.

🔄 **Learning Curve**: YAML has steeper learning curve but provides better long-term maintainability.

✨ **Version Control**: YAML pipelines stored in Git provide audit trail and branching capabilities classic releases lack.

## Quick Reference

### Feature Availability Summary

```
YAML Advantages:
✅ Version controlled in Git
✅ Templates for reusability
✅ Container jobs
✅ Service containers
✅ Multi-stage in single file
✅ Deployment strategies (rolling, canary)

Classic Release Advantages:
✅ Deployment groups (on-prem)
✅ Rich gate UI
✅ Visual designer
✅ Post-deployment approvals
✅ Release dashboard
✅ Complex approval workflows
```

### Command Comparison

```bash
# Create YAML pipeline (stored in repo)
# File: azure-pipelines.yml
git add azure-pipelines.yml
git commit -m "Add YAML pipeline"
git push

# Create classic release pipeline (UI-based)
# Navigate to: Pipelines → Releases → New Pipeline
# Configure in Azure DevOps UI
# Not stored in source control
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline-devops/2-describe-azure-devops-capabilities)
