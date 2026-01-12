# Introduction

This module covers Azure Pipelines release management capabilities, exploring how to design and implement release pipelines that automate deployments across multiple environments with proper approvals, gates, and quality controls.

## Module Overview

**Duration**: Approximately 50 minutes  
**Level**: Advanced  
**Roles**: DevOps Engineer, Developer, Administrator, Solution Architect  

## Learning Objectives

After completing this module, you'll be able to:

**1. Explain Azure Pipelines Release Management Terminology**
- Understand release pipeline components (artifacts, stages, approvals, gates)
- Differentiate between build pipelines and release pipelines
- Know when to use classic release pipelines vs YAML multi-stage pipelines
- Understand continuous deployment vs continuous delivery

**2. Describe Build and Release Tasks**
- Identify common deployment tasks (Azure Web App, Kubernetes, IIS)
- Understand task capabilities and configuration options
- Work with custom build and release tasks
- Configure task conditions and dependencies

**3. Implement Release Jobs**
- Configure deployment jobs for various environments
- Set up multi-agent release jobs for parallel deployment
- Implement multi-configuration jobs for cross-platform releases
- Understand agent pools and deployment groups

**4. Configure Deployment Workflows**
- Design multi-stage release pipelines
- Configure deployment triggers (manual, scheduled, continuous)
- Implement approval workflows and release gates
- Set up environment-specific variables and secrets

## Prerequisites

To get the most from this module, you should have:

**Required Knowledge**:
- ✅ Understanding of DevOps concepts and practices
- ✅ Familiarity with Azure Pipelines basics (from LP2)
- ✅ Basic knowledge of CI/CD workflows
- ✅ Experience with version control (Git)

**Helpful Experience**:
- Experience deploying applications to cloud platforms
- Familiarity with Azure services (App Service, AKS, etc.)
- Understanding of infrastructure as code concepts
- Knowledge of application lifecycle management

**Technical Setup** (for hands-on exercises):
- Azure DevOps organization ([free account](https://dev.azure.com))
- Azure subscription ([free trial available](https://azure.microsoft.com/free/))
- Sample application repository (provided in exercises)

## What You'll Learn

### Release Pipeline Fundamentals

```
Release Pipeline Workflow

Build Pipeline                Release Pipeline
      ↓                             ↓
   Artifacts  →  [Dev Stage]  →  [QA Stage]  →  [Prod Stage]
                      ↓               ↓              ↓
                  Auto Deploy     Approval      Approval
                                  + Tests       + Gates
```

**Key Concepts**:
- **Artifacts**: Build outputs that are deployed (compiled apps, containers, packages)
- **Stages**: Deployment environments (dev, QA, staging, production)
- **Approvals**: Manual or automated sign-offs before deployment
- **Gates**: Automated quality checks (health monitoring, security scans)
- **Tasks**: Individual deployment steps (copy files, run scripts, deploy to Azure)
- **Jobs**: Collection of tasks that run on an agent

### Classic Release vs YAML Pipelines

| Feature | Classic Release | YAML Multi-Stage |
|---------|----------------|------------------|
| **Configuration** | UI-based | Code-based (YAML) |
| **Version Control** | Limited | Full Git integration |
| **Approval Gates** | ✅ Rich UI | ⚠️ Requires YAML |
| **Deployment Groups** | ✅ Supported | ❌ Not supported |
| **Stages** | ✅ Visual designer | ✅ YAML definition |
| **Templates** | Task groups | YAML templates |
| **Best For** | Complex approvals, deployment groups | Infrastructure as code, simple workflows |

### Build Tasks vs Release Tasks

**Build Tasks** (CI Pipeline):
- Compile source code
- Run unit tests
- Package artifacts
- Push to artifact repository
- Quality gates (code coverage, security scans)

**Release Tasks** (CD Pipeline):
- Download artifacts
- Deploy to environments (Azure, AWS, on-prem)
- Run integration/smoke tests
- Database migrations
- Configure application settings
- Blue-green or canary deployments

## Module Roadmap

This module consists of 13 units:

| Unit | Topic | What You'll Learn |
|------|-------|-------------------|
| **1** | Introduction | Module overview and learning objectives |
| **2** | Azure Pipelines capabilities | YAML vs classic pipelines, feature comparison |
| **3** | Explore release pipelines | Pipeline components, triggers, stages |
| **4** | Explore artifact sources | Build artifacts, repos, feeds, registries |
| **5** | Choose artifact source | When to use each artifact type |
| **6** | Deployment to stages | Environment considerations, strategies |
| **7** | Build and release tasks | Common tasks, configuration, usage |
| **8** | Custom tasks | Creating custom build/release tasks |
| **9** | Release jobs | Agent jobs, deployment jobs, server jobs |
| **10** | Database deployment | SQL deployment tasks and patterns |
| **11** | Pipelines as code (YAML) | Multi-stage YAML pipelines |
| **12** | Knowledge check | Assessment questions |
| **13** | Summary | Key takeaways and next steps |

## Real-World Scenarios

### Scenario 1: Web Application Deployment

**Goal**: Deploy ASP.NET Core web app to Azure App Service across dev, staging, and production environments.

**Pipeline Stages**:
1. **Dev**: Auto-deploy on every commit, no approval required
2. **Staging**: Auto-deploy after dev, manual approval for smoke tests
3. **Production**: Manual approval + automated quality gates (health checks, performance tests)

### Scenario 2: Microservices to Kubernetes

**Goal**: Deploy multiple containerized microservices to AKS with coordinated releases.

**Pipeline Approach**:
- Artifacts: Docker images from build pipeline
- Stages: Dev cluster → QA cluster → Prod cluster
- Deployment strategy: Blue-green deployment with traffic shifting
- Gates: Health endpoint checks, Prometheus metrics validation

### Scenario 3: Database + Application Release

**Goal**: Deploy application with database schema changes to on-premises servers.

**Pipeline Approach**:
- Artifacts: Application package + database migration scripts
- Deployment groups: Target multiple on-prem servers
- Tasks: Run database migrations, deploy application, run smoke tests
- Approvals: DBA approval for production database changes

## Why Release Pipelines Matter

**Business Benefits**:
- ✅ **Faster Time to Market**: Automate deployments, reduce manual work from hours to minutes
- ✅ **Reduced Risk**: Consistent deployment process reduces human errors
- ✅ **Quality Assurance**: Automated gates ensure only healthy releases reach production
- ✅ **Audit Trail**: Complete history of who approved what and when
- ✅ **Rollback Capability**: Quick rollback to previous versions if issues arise

**Technical Benefits**:
- ✅ **Standardization**: Same deployment process across all environments
- ✅ **Parallel Deployments**: Deploy to multiple servers simultaneously
- ✅ **Environment Parity**: Consistent configuration across dev/staging/prod
- ✅ **Integration**: Works with Azure, AWS, Kubernetes, VMs, on-prem servers
- ✅ **Extensibility**: Custom tasks for specific deployment needs

## Module Learning Path

```
Foundation (Units 1-3)
  ↓ Understand release management concepts
  ↓ Explore Azure Pipelines capabilities
  ↓ Learn release pipeline components

Artifacts & Sources (Units 4-5)
  ↓ Explore artifact types and sources
  ↓ Choose appropriate artifact source

Deployment (Units 6-9)
  ↓ Configure deployment to stages
  ↓ Work with build and release tasks
  ↓ Create custom tasks
  ↓ Implement release jobs

Advanced Topics (Units 10-11)
  ↓ Database deployment patterns
  ↓ YAML multi-stage pipelines

Assessment (Units 12-13)
  ↓ Knowledge check
  ↓ Module summary and next steps
```

## Success Criteria

By the end of this module, you should be able to:

- [ ] Create a release pipeline with multiple stages
- [ ] Configure artifact sources from build pipelines
- [ ] Set up approval workflows for production deployments
- [ ] Implement automated quality gates
- [ ] Deploy applications to Azure services using release tasks
- [ ] Configure deployment groups for on-premises servers
- [ ] Create custom build and release tasks
- [ ] Design multi-stage YAML pipelines
- [ ] Implement database deployment strategies
- [ ] Troubleshoot common release pipeline issues

## Key Terms

**Glossary**:

| Term | Definition |
|------|------------|
| **Artifact** | Output from build pipeline (compiled app, Docker image, package) |
| **Stage** | Deployment environment (dev, QA, prod) in release pipeline |
| **Approval** | Manual or automated check before proceeding to next stage |
| **Gate** | Automated quality check (e.g., health monitoring, security scan) |
| **Trigger** | Event that starts a release (manual, scheduled, continuous deployment) |
| **Deployment Group** | Collection of target servers for on-premises deployments |
| **Agent Pool** | Collection of build/release agents that execute tasks |
| **Task** | Individual step in deployment (e.g., deploy to Azure, run script) |
| **Job** | Collection of tasks that run sequentially on an agent |
| **Classic Release** | UI-based release pipeline designer in Azure DevOps |
| **YAML Pipeline** | Code-based multi-stage pipeline (build + release in single file) |

## What's Next

After this introduction, you'll explore:

1. **Azure Pipelines Capabilities** (Unit 2): Compare YAML vs classic pipelines, understand feature availability
2. **Release Pipeline Components** (Unit 3): Deep dive into artifacts, triggers, stages, approvals, gates
3. **Artifact Sources** (Units 4-5): Work with different artifact types and choose the right source

Let's get started by understanding Azure Pipelines capabilities!

---

[Continue to Unit 2: Describe Azure Pipelines Capabilities →](./02-describe-azure-pipelines-capabilities.md)

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline-devops/1-introduction)
