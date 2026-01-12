# Knowledge Check

Test your understanding of release pipelines, deployment strategies, artifacts, release jobs, database deployments, and YAML pipelines.

## Questions

### Question 1: Release Pipeline Purpose

**What is the primary purpose of a release pipeline in Azure Pipelines?**

A) Build and compile source code  
B) Deploy artifacts to target environments with approvals and testing  
C) Version control source code  
D) Run unit tests  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: Release pipelines are specifically designed for **deployment orchestration**. They take artifacts produced by build pipelines and deploy them through multiple stages (Dev, QA, Production) with:
- **Deployment tasks**: Deploy to Azure, Kubernetes, VMs, etc.
- **Approvals and gates**: Manual/automated checks before deployment
- **Environment-specific configuration**: Different settings per stage
- **Deployment tracking**: History and rollback capabilities

**Build vs Release**:
- **Build Pipeline**: Compiles code, runs tests, produces artifacts
- **Release Pipeline**: Deploys artifacts to environments

**Review**: If you answered incorrectly, revisit [Unit 1: Introduction](#) for release pipeline fundamentals.

</details>

---

### Question 2: Artifact Sources

**Which of the following can be a valid artifact source for a release pipeline?**

A) Azure Pipelines build  
B) GitHub releases  
C) Azure Container Registry  
D) All of the above  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: D**

**Explanation**: Release pipelines support **multiple artifact sources**:

1. **Azure Pipelines**: Most common—build pipeline produces artifacts
2. **GitHub/TFVC/Bitbucket**: Source repositories
3. **Jenkins**: CI server builds
4. **TeamCity**: CI server builds
5. **Azure Container Registry**: Container images
6. **Docker Hub**: Container images
7. **NuGet/npm/Maven**: Package repositories

**Multi-Source Example**:
```yaml
Artifact 1: Build pipeline (application code)
Artifact 2: Azure Container Registry (Docker image)
Artifact 3: GitHub (configuration files)

Deploy: Combine all three artifacts in release
```

**Review**: If you answered incorrectly, revisit [Unit 4: Explore artifact sources](#) for artifact types.

</details>

---

### Question 3: Continuous Deployment Trigger

**What does enabling the continuous deployment trigger do in a release pipeline?**

A) Automatically runs tests on every commit  
B) Automatically creates a new release when a new build artifact is available  
C) Deploys to production without any approvals  
D) Prevents manual release creation  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: The **continuous deployment (CD) trigger** creates a new release automatically when:
- A new build completes successfully
- A new artifact version becomes available
- Trigger conditions are met (branch filters, tag filters)

**Workflow**:
```
Code commit → Build pipeline runs → Build succeeds → Artifact published
                                          ↓
                            CD Trigger activated
                                          ↓
                            Release pipeline starts automatically
                                          ↓
                            Deploy to Dev (auto)
                                          ↓
                            Deploy to QA (may require approval)
                                          ↓
                            Deploy to Prod (requires approval)
```

**Important**: CD trigger creates the release, but each stage still respects its own approval gates.

**Continuous Deployment ≠ No Approvals**:
- CD trigger: Automatic release creation
- Approvals: Still required per stage (if configured)

**Review**: If you answered incorrectly, revisit [Unit 3: Understand release pipelines](#) for trigger configuration.

</details>

---

### Question 4: Deployment Job Types

**Which job type should you use to deploy to multiple on-premises servers simultaneously?**

A) Agent Job  
B) Deployment Group Job  
C) Agentless Job  
D) Container Job  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: **Deployment Group Jobs** deploy to multiple target machines in parallel:

**Deployment Group Architecture**:
```
Deployment Group: Web Servers
        ↓
┌───────┬───────┬───────┬───────┐
│Server1│Server2│Server3│Server4│
│Agent  │Agent  │Agent  │Agent  │
│Deploy │Deploy │Deploy │Deploy │
└───────┴───────┴───────┴───────┘
Parallel execution on all servers
```

**Setup**:
1. Create deployment group in Azure DevOps
2. Install agent on each target server
3. Tag servers by role (web, api, database)
4. Deploy job targets deployment group with tag filter

**Job Type Comparison**:
| Job Type | Use Case |
|----------|----------|
| **Agent Job** | Single target, standard tasks |
| **Deployment Group Job** | Multiple servers, parallel deployment |
| **Agentless Job** | Manual approval, delays, gates |
| **Container Job** | Run tasks in container |

**Example**:
```yaml
- deployment: DeployToWebServers
  environment:
    name: Production
    resourceType: VirtualMachine
    tags: web  # Deploy to all servers tagged 'web'
  strategy:
    rolling:
      maxParallel: 2
```

**Review**: If you answered incorrectly, revisit [Unit 9: Examine release jobs](#) for job types.

</details>

---

### Question 5: Database Deployment Strategy

**What is the Expand-Contract pattern in database deployments?**

A) Expand storage capacity, then contract after migration  
B) Add new schema alongside old, migrate data, then remove old schema  
C) Deploy to expanded number of servers, then contract back  
D) Expand transaction logs, then contract them  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: **Expand-Contract** enables zero-downtime database deployments:

**Four Phases**:
```
Phase 1: Expand (Add new schema)
  Old Column: Email_Old ← App still uses
  New Column: Email_New ← Added (nullable)

Phase 2: Migrate (Copy data)
  UPDATE Users SET Email_New = Email_Old

Phase 3: Cutover (Deploy new app version)
  App now uses Email_New column

Phase 4: Contract (Remove old schema)
  ALTER TABLE Users DROP COLUMN Email_Old
```

**Benefits**:
- ✅ **Zero downtime**: Old and new app versions work simultaneously
- ✅ **Safe rollback**: Can revert app without database rollback
- ✅ **Gradual migration**: Data copied over time
- ✅ **Low risk**: Old schema remains until verified

**Alternative Strategies**:
- **Blue-Green Database**: Two complete database copies (high cost)
- **Direct ALTER**: Downtime required (locks table)
- **Shadow Database**: Test on copy first (good for validation)

**Review**: If you answered incorrectly, revisit [Unit 10: Explore database deployment tasks](#) for database strategies.

</details>

---

### Question 6: DACPAC vs SQL Scripts

**When should you use a DACPAC file instead of SQL scripts for database deployment?**

A) When you need to execute custom data migrations  
B) When you need automatic schema comparison and idempotent deployments  
C) When deploying stored procedures only  
D) When the database doesn't exist yet  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: **DACPAC (Data-Tier Application Package)** provides intelligent schema deployment:

**DACPAC Benefits**:
1. **Schema Comparison**: Compares source schema with target, generates only needed changes
2. **Idempotent**: Safe to run multiple times (only applies differences)
3. **Data Loss Protection**: Can block deployment if data loss detected
4. **Rollback Scripts**: Can generate scripts without executing

**DACPAC Workflow**:
```
1. Build database project → Produces .dacpac
2. Deploy .dacpac to target
3. SqlPackage.exe compares schemas
4. Generates ALTER scripts for differences only
5. Executes scripts
```

**DACPAC vs SQL Scripts**:
| Aspect | DACPAC | SQL Scripts |
|--------|--------|-------------|
| **Schema Comparison** | ✅ Automatic | ❌ Manual |
| **Idempotency** | ✅ Built-in | ⚠️ Must code (IF EXISTS) |
| **Data Loss Check** | ✅ Yes (BlockOnPossibleDataLoss) | ❌ No |
| **Data Migration** | ❌ Limited | ✅ Full SQL flexibility |
| **Hotfixes** | ⚠️ Hard (must rebuild project) | ✅ Easy (direct SQL) |

**Use DACPAC for**: Schema deployments, table structure changes  
**Use SQL Scripts for**: Data migrations, hotfixes, custom logic

**Review**: If you answered incorrectly, revisit [Unit 10: Explore database deployment tasks](#) for DACPAC details.

</details>

---

### Question 7: Multi-Stage YAML Pipelines

**What is a key advantage of multi-stage YAML pipelines over Classic release pipelines?**

A) Better UI for configuration  
B) Faster execution  
C) Version control and code review for deployment configuration  
D) More deployment tasks available  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: C**

**Explanation**: **Multi-stage YAML pipelines** treat deployment configuration as code:

**Key Advantages**:

1. **Version Control**:
   ```
   azure-pipelines.yml in Git
     ↓
   Change pipeline → Commit → Push
     ↓
   Full history of pipeline changes (who, when, what)
   ```

2. **Code Review**:
   ```
   Developer modifies deployment strategy
     ↓
   Create Pull Request with pipeline changes
     ↓
   Team reviews deployment logic changes
     ↓
   Approve → Merge → New pipeline active
   ```

3. **Branch-Specific Pipelines**:
   ```
   main branch: Deploy to Dev, QA, Prod
   develop branch: Deploy to Dev only
   feature/* branches: Build only, no deployment
   ```

4. **Testing Pipeline Changes**:
   ```
   Feature branch with new deployment stage
     ↓
   Test new deployment in feature branch
     ↓
   Verified working → Merge to main
   ```

**Classic vs YAML**:
| Aspect | Classic | YAML |
|--------|---------|------|
| **Configuration** | UI (web-based) | Code (YAML file) |
| **Version Control** | ❌ No | ✅ Yes (Git) |
| **Code Review** | ❌ No | ✅ Yes (PRs) |
| **Branching** | Single definition | Per-branch definitions |
| **Portability** | Azure DevOps only | YAML (portable) |

**Review**: If you answered incorrectly, revisit [Unit 11: Implement pipelines as code with YAML](#) for YAML benefits.

</details>

---

### Question 8: Deployment Strategies

**Which deployment strategy gradually rolls out to increasing percentages of servers with monitoring between increments?**

A) RunOnce  
B) Rolling  
C) Canary  
D) Blue-Green  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: C**

**Explanation**: **Canary deployment** progressively increases deployment scope:

**Canary Strategy**:
```yaml
strategy:
  canary:
    increments: [10, 25, 50, 100]
    deploy:
      steps:
      - script: echo "Deploy to $(strategy.increment)% of servers"
    postRouteTraffic:
      steps:
      - script: echo "Monitor metrics for 10 minutes"
```

**Canary Flow**:
```
100 Servers

Increment 1 (10%): Deploy to 10 servers
   ↓ Monitor: Error rate, response time, CPU
   ✅ Metrics healthy → Continue

Increment 2 (25%): Deploy to 25 total (15 more)
   ↓ Monitor for 10 minutes
   ✅ Metrics healthy → Continue

Increment 3 (50%): Deploy to 50 total (25 more)
   ↓ Monitor for 10 minutes
   ⚠️ Error rate spike detected → STOP & ROLLBACK

Result: Only 50% deployed, 50% still on safe version
```

**Strategy Comparison**:
| Strategy | Risk Level | Rollout Speed | Use Case |
|----------|-----------|---------------|----------|
| **RunOnce** | High | Fast | Low-risk deployments |
| **Rolling** | Medium | Medium | Multi-server, gradual |
| **Canary** | Low | Slow | High-risk, production |
| **Blue-Green** | Low | Fast (instant switch) | Zero-downtime |

**Canary Benefits**:
- ✅ **Early detection**: Issues caught with minimal impact
- ✅ **Progressive rollout**: Confidence builds with each increment
- ✅ **Easy rollback**: Most servers unaffected if failure
- ✅ **Monitoring integration**: Automated health checks

**Review**: If you answered incorrectly, revisit [Unit 11: Implement pipelines as code with YAML](#) for deployment strategies.

</details>

---

### Question 9: Environment Approvals

**In a multi-stage YAML pipeline, how do you configure manual approval before deploying to production?**

A) Add `approval: required` in YAML  
B) Create environment in Azure DevOps and configure approvals on it  
C) Use `waitForApproval` task  
D) Set `manualValidation: true` in pipeline  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: **Environments** in Azure DevOps control deployment approvals:

**Setup Process**:

1. **Create Environment** (Azure DevOps UI):
   ```
   Pipelines > Environments > New environment
   Name: Production
   Type: None (or VMs/Kubernetes)
   ```

2. **Configure Approvals** (Environment settings):
   ```
   Environments > Production > Approvals and checks
   Add check > Approvals
   Approvers: user@company.com, DevOps Team
   Timeout: 30 days
   ```

3. **Reference in YAML**:
   ```yaml
   - deployment: DeployProd
     environment: Production  # Links to environment with approvals
     strategy:
       runOnce:
         deploy:
           steps:
           - script: echo "Deploy after approval"
   ```

**Approval Flow**:
```
Pipeline reaches Production stage
        ↓
Environment "Production" requires approval
        ↓
Pipeline pauses, notification sent to approvers
        ↓
Approver reviews, clicks "Approve" or "Reject"
        ↓
If approved: Deployment continues
If rejected: Pipeline fails
If timeout: Pipeline fails
```

**Environment Checks Available**:
- ✅ **Approvals**: Manual approval from users/groups
- ✅ **Branch control**: Only deploy from specific branches
- ✅ **Business hours**: Only deploy during work hours
- ✅ **Invoke Azure Function**: Custom validation logic
- ✅ **Query Azure Monitor**: No active alerts required

**YAML doesn't contain approval logic** (separation of concerns):
- YAML: Deployment tasks (what to deploy)
- Environment: Deployment policy (when/who approves)

**Review**: If you answered incorrectly, revisit [Unit 11: Implement pipelines as code with YAML](#) for environment configuration.

</details>

---

### Question 10: Artifact Filtering

**You have a build pipeline that produces multiple artifacts, but your release pipeline only needs the web application artifact. How do you specify this?**

A) Delete unwanted artifacts after download  
B) Use artifact filters/alias in release pipeline configuration  
C) Modify build pipeline to produce only one artifact  
D) Download all artifacts and ignore unwanted ones  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: **Artifact filtering** optimizes release pipelines:

**Classic Release Pipeline**:
```
Artifacts section:
  Source: MyBuildPipeline
  Default version: Latest
  Artifact filter: webapp  ← Only download 'webapp' artifact
  
Build produces:
  - webapp (needed)
  - desktop-app (not needed)
  - mobile-app (not needed)

Release downloads: Only 'webapp' artifact
```

**YAML Pipeline**:
```yaml
- download: current
  artifact: webapp  # Download specific artifact only

# Or download multiple specific artifacts
- download: current
  artifact: webapp

- download: current
  artifact: api-service

# Or download all (default)
- download: current
  displayName: 'Download all artifacts'
```

**Benefits**:
- ✅ **Faster downloads**: Only download what's needed
- ✅ **Storage savings**: Don't waste pipeline storage
- ✅ **Cleaner workspace**: Fewer files to manage
- ✅ **Clear intent**: Explicit artifact dependencies

**Multi-Artifact Example**:
```yaml
# Build pipeline produces multiple artifacts
- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: 'webapp/dist'
    artifactName: 'webapp'

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: 'api/publish'
    artifactName: 'api'

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: 'database/dacpac'
    artifactName: 'database'

# Release pipeline selects specific artifacts
- download: current
  artifact: webapp  # Web deployment job needs only webapp

- download: current
  artifact: database  # Database deployment job needs only database
```

**Artifact Aliases** (Classic pipelines):
```
Primary artifact: _MyBuildPipeline (alias)
Secondary artifact: _DatabaseBuild (alias)

Reference in tasks:
  Package: $(System.DefaultWorkingDirectory)/_MyBuildPipeline/webapp/app.zip
  DACPAC: $(System.DefaultWorkingDirectory)/_DatabaseBuild/database/db.dacpac
```

**Review**: If you answered incorrectly, revisit [Unit 5: Examine artifact selection](#) for artifact filtering.

</details>

---

## Scoring Guide

- **9-10 correct**: Excellent! You have mastered release pipeline concepts.
- **7-8 correct**: Good understanding. Review missed topics for deeper knowledge.
- **5-6 correct**: Passing. Study the module content again, focusing on weak areas.
- **Below 5**: Review the entire module. Release pipeline concepts require solid understanding.

## Key Takeaways

If you struggled with certain questions, focus on these areas:

### Release Pipeline Fundamentals
- ✅ Purpose: Deploy artifacts through stages with approvals
- ✅ Continuous deployment: Automatic release creation on new build
- ✅ Artifacts: Multiple sources supported (builds, containers, packages)
- ✅ Stages: Dev, QA, Production with environment-specific config

### Deployment Jobs
- ✅ Agent Job: Single-target deployments
- ✅ Deployment Group Job: Multi-server parallel deployment
- ✅ Agentless Job: Manual approvals, gates, delays
- ✅ Container Job: Deploy using containerized tools

### Database Deployments
- ✅ DACPAC: Schema deployments with comparison and idempotency
- ✅ SQL Scripts: Data migrations and custom logic
- ✅ Expand-Contract: Zero-downtime schema changes
- ✅ Best practices: Idempotent scripts, transactions, versioning

### YAML Pipelines
- ✅ Multi-stage: Build and release in single pipeline
- ✅ Version control: Pipeline definition in Git
- ✅ Code review: PR approval for pipeline changes
- ✅ Environments: Approval gates and deployment tracking

### Deployment Strategies
- ✅ RunOnce: Simple single deployment
- ✅ Rolling: Gradual multi-server rollout
- ✅ Canary: Progressive with monitoring and rollback
- ✅ Blue-Green: Instant switch with duplicate environment

## Next Steps

Ready to continue? Proceed to the [Summary](#) unit for module recap and next learning steps.

Need more practice? Revisit specific units:
- [Unit 3: Understand release pipelines](#) - Pipeline structure
- [Unit 4-5: Artifacts](#) - Artifact sources and filtering
- [Unit 9: Release jobs](#) - Job types and deployment groups
- [Unit 10: Database deployment](#) - DACPAC, strategies, best practices
- [Unit 11: YAML pipelines](#) - Multi-stage, deployment strategies, environments

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline/)
