# Control Deployments Using Release Gates

## Lab Overview
This hands-on lab covers the configuration of deployment gates and demonstrates how to use them to control Azure Pipelines execution.

**Estimated Time**: 75 minutes  
**Difficulty**: Intermediate

## Scenario
Configure a release definition with two environments for an Azure Web App:
1. **Canary Environment**: Deploy only when no blocking bugs exist
2. **Production Environment**: Promote only when no active Application Insights alerts

## Learning Objectives
After completing this lab, you'll be able to:
- ✅ Configure release pipelines
- ✅ Configure release gates  
- ✅ Test release gates in real-world scenarios

## Prerequisites

| Requirement | Details |
|------------|---------|
| **Browser** | Microsoft Edge or Azure DevOps-supported browser |
| **Azure DevOps Organization** | [Create an organization](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization) |
| **Azure Subscription** | Existing or new subscription |
| **Azure Role** | Contributor or Owner role in subscription |

---

## Lab Architecture

```
Build Pipeline → Artifact → Release Pipeline
                               ↓
                    ┌──────────┴──────────┐
                    ↓                     ↓
            Canary Environment    Production Environment
            [Pre-Gate: No Bugs]   [Pre-Gate: No Alerts]
                    ↓                     ↓
              Azure Web App         Azure Web App
                    ↓                     ↓
            [Post-Gate: Monitor]  [Post-Gate: Monitor]
```

### Key Concepts Demonstrated

**Phased Rollout**:
```
Updates → Subset of Users (Canary) → Monitor → All Users (Production)
```

**Benefits**:
- Minimize risk of widespread issues
- Validate in production-like environment
- Quick rollback if problems detected
- User experience monitoring

---

## Lab Exercises

### Exercise 0: Configure Lab Prerequisites

**Tasks**:
1. Create Azure DevOps project
2. Import sample repository
3. Create service connection to Azure
4. Provision Azure resources

**Azure Resources Created**:
```
Resource Group: az400-lab-RG
├── App Service Plan: az400-lab-ASP
├── Web App (Canary): az400-lab-canary
├── Web App (Production): az400-lab-production
└── Application Insights: az400-lab-appinsights
```

---

### Exercise 1: Configure the Build Pipeline

**Objective**: Create CI pipeline that builds application and publishes artifacts.

#### Build Pipeline YAML

```yaml
# azure-pipelines-build.yml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

steps:
  - task: NuGetToolInstaller@1
  
  - task: NuGetCommand@2
    displayName: 'Restore NuGet packages'
    inputs:
      restoreSolution: '$(solution)'
  
  - task: VSBuild@1
    displayName: 'Build solution'
    inputs:
      solution: '$(solution)'
      msbuildArgs: '/p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.artifactStagingDirectory)"'
      platform: '$(buildPlatform)'
      configuration: '$(buildConfiguration)'
  
  - task: VSTest@2
    displayName: 'Run unit tests'
    inputs:
      platform: '$(buildPlatform)'
      configuration: '$(buildConfiguration)'
  
  - task: PublishBuildArtifacts@1
    displayName: 'Publish artifact'
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)'
      ArtifactName: 'drop'
```

**Outcome**: Artifact published for release pipeline.

---

### Exercise 2: Configure the Release Pipeline

**Objective**: Create release pipeline with Canary and Production stages.

#### Release Pipeline Structure

```yaml
# Conceptual structure (Classic Release Pipeline)
Release Pipeline:
  Artifacts:
    - Source: Build Pipeline
      Trigger: Continuous deployment
  
  Stages:
    - Canary:
        Tasks:
          - Deploy Azure App Service
        Post-Deployment Conditions:
          - Manual approval (optional)
    
    - Production:
        Depends On: Canary
        Tasks:
          - Deploy Azure App Service
        Pre-Deployment Conditions:
          - Manual approval (optional)
```

#### Canary Stage Configuration

```yaml
- stage: Canary
  displayName: 'Deploy to Canary'
  jobs:
    - deployment: DeployCanary
      environment: canary
      pool:
        vmImage: 'windows-latest'
      strategy:
        runOnce:
          deploy:
            steps:
              - task: AzureWebApp@1
                displayName: 'Deploy to Canary Web App'
                inputs:
                  azureSubscription: 'Azure-Connection'
                  appType: 'webApp'
                  appName: 'az400-lab-canary'
                  package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

#### Production Stage Configuration

```yaml
- stage: Production
  displayName: 'Deploy to Production'
  dependsOn: Canary
  condition: succeeded()
  jobs:
    - deployment: DeployProduction
      environment: production
      pool:
        vmImage: 'windows-latest'
      strategy:
        runOnce:
          deploy:
            steps:
              - task: AzureWebApp@1
                displayName: 'Deploy to Production Web App'
                inputs:
                  azureSubscription: 'Azure-Connection'
                  appType: 'webApp'
                  appName: 'az400-lab-production'
                  package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

---

### Exercise 3: Configure Release Gates

**Objective**: Add automated gates to control deployment flow.

#### Gate 1: Query Work Items (Pre-Deployment for Canary)

**Purpose**: Block deployment if blocking bugs exist.

**Configuration**:
```
Gate Type: Query Work Items
Query: Blocking Bugs
Maximum threshold: 0
Evaluation options:
  - Delay before evaluation: 0 minutes
  - Time between re-evaluation: 5 minutes
  - Timeout: 12 hours
```

**Query Definition** (WIQL):
```wiql
SELECT [System.Id], [System.Title], [System.State]
FROM WorkItems
WHERE [System.WorkItemType] = 'Bug'
  AND [Microsoft.VSTS.Common.Priority] = 1
  AND [System.State] <> 'Closed'
  AND [System.Tags] CONTAINS 'Blocking'
```

**Expected Behavior**:
```
Scenario 1: No blocking bugs
- Gate checks: 0 bugs found ✅
- Result: PASS → Deploy to Canary

Scenario 2: 2 blocking bugs exist
- Gate checks: 2 bugs found ❌
- Result: FAIL → Retry in 5 minutes
- After 12 hours: TIMEOUT → Deployment rejected
```

#### Gate 2: Query Azure Monitor Alerts (Post-Deployment for Canary)

**Purpose**: Ensure no Application Insights alerts after Canary deployment.

**Configuration**:
```
Gate Type: Query Azure Monitor Alerts
Azure subscription: Azure-Connection
Resource group: az400-lab-RG
Resource type: microsoft.insights/components
Resource name: az400-lab-appinsights
Alert state: New, Acknowledged
Severity: Sev0, Sev1, Sev2
Evaluation options:
  - Delay before evaluation: 5 minutes (allow traffic)
  - Time between re-evaluation: 5 minutes
  - Timeout: 2 hours
```

**Expected Behavior**:
```
Deployment to Canary completes at 14:00

14:05 (T+5min): First gate evaluation
- Query Application Insights
- Check for alerts in last 5 minutes
- 0 alerts found ✅ → PASS

Canary validation successful → Proceed to Production
```

#### Gate 3: Query Azure Monitor Alerts (Post-Deployment for Production)

**Purpose**: Validate production deployment health.

**Configuration**: Same as Gate 2, but for production Web App.

```
Delay: 5 minutes (allow production traffic to generate telemetry)
Check: No Sev0/Sev1/Sev2 alerts in last 5 minutes
Timeout: 2 hours
```

---

### Exercise 4: Test Release Gates

**Objective**: Trigger deployments and observe gate behavior.

#### Test Case 1: Successful Deployment (No Blockers)

```
Timeline:
10:00 - Code commit pushed
10:05 - Build pipeline completes
10:06 - Release triggered automatically
10:06 - Canary pre-deployment gate: Check work items
        Query result: 0 blocking bugs ✅ PASS
10:06 - Deploy to Canary starts
10:10 - Deploy to Canary completes
10:10 - Canary post-deployment gate: Wait 5 minutes
10:15 - Canary post-deployment gate: Query App Insights
        Query result: 0 alerts ✅ PASS
10:15 - Canary stage COMPLETE
10:15 - Production pre-deployment gate: (if configured)
10:15 - Deploy to Production starts
10:20 - Deploy to Production completes
10:20 - Production post-deployment gate: Wait 5 minutes
10:25 - Production post-deployment gate: Query App Insights
        Query result: 0 alerts ✅ PASS
10:25 - Production stage COMPLETE ✅
```

#### Test Case 2: Blocked by Work Items

```
Timeline:
11:00 - Code commit pushed
11:05 - Build pipeline completes
11:06 - Release triggered
11:06 - Canary pre-deployment gate: Check work items
        Query result: 2 blocking bugs ❌ FAIL
11:11 - Gate retry: 2 blocking bugs ❌ FAIL
11:16 - Gate retry: 2 blocking bugs ❌ FAIL
...
11:45 - Developer resolves 2 blocking bugs
11:46 - Gate retry: 0 blocking bugs ✅ PASS
11:46 - Deploy to Canary proceeds
```

#### Test Case 3: Failed by Application Insights Alerts

```
Timeline:
12:00 - Deploy to Canary completes
12:00 - Post-deployment gate: Wait 5 minutes
12:05 - Post-deployment gate: Query App Insights
        Error rate spike detected: 3 Sev1 alerts ❌ FAIL
12:10 - Gate retry: 2 Sev1 alerts ❌ FAIL
12:15 - Ops team investigates, finds database connectivity issue
12:20 - Database issue resolved
12:20 - Gate retry: 0 alerts ✅ PASS
12:20 - Canary validation complete → Proceed to Production
```

---

## Hands-On Steps

### Step 1: Create Work Item Query

1. Navigate to Azure DevOps → Boards → Queries
2. Create new query: "Blocking Bugs"
3. Query definition:
   ```wiql
   Work Item Type = Bug
   Priority = 1
   State != Closed
   Tags Contains "Blocking"
   ```
4. Save query and note Query ID

### Step 2: Configure Application Insights Alerts

1. Navigate to Azure Portal → Application Insights
2. Create alert rule:
   ```
   Name: High Error Rate
   Condition: Exceptions > 10 in 5 minutes
   Severity: Sev1
   Action: Email ops team
   ```

### Step 3: Set Up Release Gates in Azure DevOps

**For Canary Stage**:
```
1. Edit Release Pipeline
2. Select Canary stage
3. Pre-deployment conditions → Gates → Enable
4. Add gate: Query Work Items
   - Query ID: <your-query-id>
   - Max threshold: 0
5. Post-deployment conditions → Gates → Enable
6. Add gate: Query Azure Monitor alerts
   - Resource group: az400-lab-RG
   - Resource name: az400-lab-appinsights
   - Alert state: New, Acknowledged
   - Severity: 0,1,2
7. Save pipeline
```

### Step 4: Trigger Test Deployment

```bash
# Make code change
echo "test" >> README.md
git add .
git commit -m "Test release gates"
git push origin main

# Monitor in Azure DevOps
# Pipelines → Releases → Watch gates evaluate
```

### Step 5: Simulate Failure Scenarios

**Scenario A: Create Blocking Bug**
```
1. Boards → Work Items → New Bug
2. Title: "Critical production issue"
3. Priority: 1
4. Tags: Blocking
5. Save
6. Trigger release → Observe pre-deployment gate FAIL
7. Close bug → Gate retries → PASS
```

**Scenario B: Trigger Application Insights Alert**
```
1. Deploy to Canary
2. Generate exceptions in application:
   curl https://az400-lab-canary.azurewebsites.net/api/error -X POST
3. Observe post-deployment gate FAIL (alerts detected)
4. Fix issue
5. Wait for alerts to resolve
6. Gate retries → PASS
```

---

### Exercise 5: Remove Azure Lab Resources

**Cleanup** (to avoid Azure charges):

```bash
# Delete resource group (removes all resources)
az group delete --name az400-lab-RG --yes --no-wait

# Delete Azure DevOps project (optional)
# Via Azure DevOps portal: Project Settings → Delete
```

---

## Lab Results

After completing this lab, you will have:

✅ **Configured Release Pipeline** with multiple stages  
✅ **Implemented Pre-Deployment Gates** (work item query)  
✅ **Implemented Post-Deployment Gates** (Azure Monitor)  
✅ **Tested Gate Behavior** (pass and fail scenarios)  
✅ **Practiced Phased Rollout** (Canary → Production)

## Key Takeaways

### 🎯 Release Gate Benefits

| Benefit | Description |
|---------|-------------|
| **Automated Quality Checks** | No manual validation needed |
| **Reduced Risk** | Catch issues before full rollout |
| **Audit Trail** | Gate evaluation history logged |
| **Consistent Criteria** | Same checks every deployment |

### 🛡️ Gate Best Practices

1. **Set Reasonable Timeouts**: Not infinite, but enough time to resolve issues
2. **Use Multiple Gates**: Layer different types of validations
3. **Monitor Gate Performance**: Track how often gates fail
4. **Plan for Failures**: Have runbooks for common gate failures
5. **Test Gates Regularly**: Ensure queries and integrations work

### ⚠️ Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **Gate Always Fails** | Incorrect query/threshold | Validate query returns expected results |
| **Gate Times Out** | Timeout too short | Increase timeout or fix underlying issue |
| **Alert Not Detected** | Wrong alert query | Verify alert severity/state filters |
| **Slow Gate Evaluation** | Long sampling interval | Reduce retry interval (5min minimum) |

---

## Additional Resources

**Lab Files**:
- [Launch Lab Environment](https://go.microsoft.com/fwlink/?linkid=2269097)

**Documentation**:
- [Release Gates Overview](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/approvals/gates)
- [Query Work Items Task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/query-work-items-v0)
- [Azure Monitor Alerts Task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/azure-monitor-v1)
- [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)

---

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/explore-release-strategy-recommendations/7-control-deployments-using-release-gates)
