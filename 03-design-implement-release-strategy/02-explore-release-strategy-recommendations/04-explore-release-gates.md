# Explore Release Gates

## Overview
**Release gates** in Azure DevOps provide enhanced control over deployment pipeline initiation and completion, integrating security and governance requirements into automated processes.

**Key Concept**: Gates define **mandatory conditions** for deployment continuation (pre-deployment) or success validation (post-deployment).

## What Are Release Gates?

```
Traditional Manual Process:
Build → Email to Team → Meeting → Manual Check → Manual Deploy

Modern Automated Gates:
Build → [Automated Checks] → Auto-Deploy
         ├─ Zero incidents?
         ├─ Tests passing?
         ├─ Security scan clear?
         └─ Monitoring healthy?
```

### Gates vs Approvals

| Aspect | Release Gates | Manual Approvals |
|--------|--------------|------------------|
| **Automation** | Fully automated | Requires human |
| **Speed** | Seconds to minutes | Hours to days |
| **Criteria** | Objective, measurable | Subjective judgment |
| **Scalability** | Unlimited | Limited by people |
| **Consistency** | Always same criteria | Can vary by approver |
| **Use Case** | Quality validation | Business decisions |

**Example**:
- **Gate**: Deploy only if code coverage > 80% ✅ (objective, automated)
- **Approval**: Product owner confirms feature is ready ✅ (subjective, manual)

---

## Pre-Deployment vs Post-Deployment Gates

### Pre-Deployment Gates

**Definition**: Conditions that must be met **before** deployment begins.

```
Stage Ready → [PRE-GATES] → All Pass? → ✅ Deploy
                   ↓
                   ❌ Fail → ⏸️ Wait/Retry or ❌ Abort
```

**Use Cases**:
- Ensure no active incidents before deploying
- Verify security scans completed
- Check work item status (no blockers)
- Validate infrastructure health

**Example**:
```yaml
preDeploymentGates:
  - task: QueryAzureMonitor
    inputs:
      alertState: "Active"
      severity: "1,2"  # P1 and P2 alerts
    # Gate passes only if no active P1/P2 alerts
```

### Post-Deployment Gates

**Definition**: Conditions that must be met **after** deployment completes.

```
Deploy Complete → [POST-GATES] → All Pass? → ✅ Mark Success
                        ↓
                        ❌ Fail → 🔄 Rollback
```

**Use Cases**:
- Validate infrastructure health after deployment
- Monitor application for errors
- Check resource utilization
- Verify compliance standards

**Example**:
```yaml
postDeploymentGates:
  - task: QueryAzureMonitor
    inputs:
      timeRange: "1h"  # Last hour
      errorThreshold: "5%"  # Max 5% error rate
```

---

## Common Release Gate Scenarios

### 1️⃣ Incident and Issues Management

**Goal**: Ensure no blocking issues exist before deployment.

**Traditional Manual Process**:
```
1. Developer checks Jira for bugs
2. Sends email to QA lead
3. QA lead reviews open bugs
4. QA lead approves via email
Time: 2-4 hours
```

**Automated Gate**:
```
1. Pipeline queries work item system
2. Checks for bugs with severity = "Blocker"
3. If count > 0: Deployment blocked
4. If count = 0: Deployment proceeds
Time: 5 seconds
```

#### Implementation Example

**Azure DevOps Query Work Items**:
```yaml
preDeploymentGates:
  - task: QueryWorkItems@0
    inputs:
      queryId: '12345'  # Saved query: "Blocker Bugs"
      maxThreshold: '0'  # Must be zero blocker bugs
```

**Example Query** (Azure DevOps):
```wiql
SELECT [System.Id], [System.Title]
FROM WorkItems
WHERE [System.WorkItemType] = 'Bug'
  AND [Microsoft.VSTS.Common.Severity] = '1 - Critical'
  AND [System.State] <> 'Closed'
  AND [System.AreaPath] = 'MyProject\Production'
```

**Result**:
- ✅ 0 blocker bugs → Gate passes → Deploy
- ❌ 1+ blocker bugs → Gate fails → Block deployment

---

### 2️⃣ Approval Integration with Collaboration Systems

**Goal**: Get stakeholder approval through chat platforms without manual pipeline intervention.

**Scenario**: Microsoft Teams Integration

```
1. Pipeline reaches pre-deployment gate
2. Adaptive Card sent to Teams channel
3. Stakeholders click "Approve" or "Reject"
4. API call updates pipeline
5. Pipeline proceeds or aborts
```

#### Implementation Example

**Azure Function + Teams Integration**:
```javascript
// Azure Function triggered by pipeline
module.exports = async function (context, req) {
    const teamWebhookUrl = process.env.TEAMS_WEBHOOK_URL;
    
    const card = {
        "@type": "MessageCard",
        "summary": "Production Deployment Approval",
        "sections": [{
            "activityTitle": "Deploy to Production",
            "facts": [
                { "name": "Release", "value": req.body.releaseName },
                { "name": "Build", "value": req.body.buildNumber }
            ]
        }],
        "potentialAction": [{
            "@type": "HttpPOST",
            "name": "Approve",
            "target": `${process.env.CALLBACK_URL}/approve`
        }, {
            "@type": "HttpPOST",
            "name": "Reject",
            "target": `${process.env.CALLBACK_URL}/reject`
        }]
    };
    
    await axios.post(teamWebhookUrl, card);
};
```

**Benefits**:
- Approvers notified in their workflow (Teams/Slack)
- No need to check email or Azure DevOps portal
- Faster response time
- Audit trail maintained

---

### 3️⃣ Quality Validation

**Goal**: Deploy only if quality metrics meet thresholds.

#### Test Pass Rate Gate

```yaml
preDeploymentGates:
  - task: PublishTestResults@2
  - task: ValidateTestResults@1
    inputs:
      testRunTitle: 'API Tests'
      minimumPassRate: '95'  # 95% tests must pass
      minimumExpectedTests: '100'  # At least 100 tests
```

**Logic**:
```
Test Results: 97 of 100 passed (97% pass rate)
Threshold: 95% required
Result: ✅ PASS - Deploy proceeds
```

#### Code Coverage Gate

```yaml
preDeploymentGates:
  - task: dotnetcore-coveragegatecheck@1
    inputs:
      coverageType: 'cobertura'
      coverageThreshold: '80'  # 80% code coverage required
      coverageVariance: '5'    # Allow 5% drop from baseline
```

**Example**:
```
Current Coverage: 82%
Baseline: 85%
Variance: 3% drop (within 5% allowed)
Threshold: 80% (met)
Result: ✅ PASS
```

---

### 4️⃣ Security Scan on Artifacts

**Goal**: Verify security scans complete before deployment.

#### Anti-Virus Check

```yaml
preDeploymentGates:
  - task: RunAntiVirusScan@1
    inputs:
      artifactPath: '$(Build.ArtifactStagingDirectory)'
      scanTimeout: '300'  # 5 minutes max
```

#### Dependency Vulnerability Scan

```yaml
preDeploymentGates:
  - task: WhiteSource@21
    inputs:
      projectName: 'MyApp'
      productName: 'Production'
      checkPolicies: true
      failOnPolicyViolation: true  # Block if high-severity vuln found
```

**Example Output**:
```
Scanning dependencies...
Found 0 critical vulnerabilities
Found 0 high vulnerabilities
Found 2 medium vulnerabilities (allowed)
Result: ✅ PASS - No blocking vulnerabilities
```

#### Code Signing Validation

```yaml
preDeploymentGates:
  - task: ValidateCodeSigning@1
    inputs:
      filePath: '$(Build.ArtifactStagingDirectory)/**/*.dll'
      certificateThumbprint: '$(SigningCertThumbprint)'
```

---

### 5️⃣ User Experience Monitoring

**Goal**: Prevent deployment if user experience regresses.

#### Application Insights Validation

```yaml
preDeploymentGates:
  - task: AzureMonitor@1
    inputs:
      resourceGroup: 'myapp-rg'
      resourceType: 'Microsoft.Insights/components'
      resourceName: 'myapp-appinsights'
      metricName: 'requests/duration'
      aggregation: 'average'
      threshold: '500'  # 500ms average response time
      timeRange: '1h'
```

**Logic**:
```
Baseline (last week): Avg response time = 300ms
Current (last hour): Avg response time = 280ms
Threshold: 500ms max
Result: ✅ PASS - Performance maintained
```

#### User Error Rate Gate

```yaml
postDeploymentGates:
  - task: QueryAppInsights@1
    inputs:
      query: |
        requests
        | where timestamp > ago(1h)
        | summarize 
            total = count(),
            failed = countif(success == false)
        | extend errorRate = (failed * 100.0) / total
      threshold: '5'  # Max 5% error rate
```

---

### 6️⃣ Change Management Integration

**Goal**: Wait for change management approval before deployment.

#### ServiceNow Integration

```yaml
preDeploymentGates:
  - task: ServiceNowChangeManagement@1
    inputs:
      serviceNowConnection: 'ServiceNow-Prod'
      changeRequestNumber: '$(ChangeRequestId)'
      expectedState: 'Implement'  # Change must be in "Implement" state
      pollingInterval: '5m'
      timeout: '24h'
```

**Workflow**:
```
1. Developer creates change request in ServiceNow
2. Change request ID added to pipeline variable
3. Pipeline reaches gate
4. Gate queries ServiceNow API
5. If state = "Implement": Proceed
6. If state = "Pending": Wait and retry
7. If state = "Rejected": Fail pipeline
```

---

### 7️⃣ Infrastructure Health Checks

**Goal**: Validate infrastructure health post-deployment.

#### Azure Monitor Alerts Gate

```yaml
postDeploymentGates:
  - task: QueryAzureMonitor@1
    inputs:
      connectedServiceNameARM: 'Azure-Connection'
      resourceGroupName: 'myapp-rg'
      filterType: 'severity'
      severity: '0,1,2'  # Sev 0, 1, 2 (Critical, Error, Warning)
      alertState: 'New,Acknowledged'
      monitorCondition: 'Fired'
```

**Checks**:
- No active critical alerts
- CPU utilization < 80%
- Memory utilization < 85%
- Disk space > 20% free

#### Resource Compliance Check

```yaml
postDeploymentGates:
  - task: AzurePolicyCompliance@1
    inputs:
      resourceGroup: 'myapp-rg'
      policyDefinitionId: '/providers/Microsoft.Authorization/policyDefinitions/...'
```

---

## Built-In Gate Types (Azure DevOps)

Azure DevOps includes **four default gate types**:

### 1. Invoke Azure Function ⚡

**Purpose**: Trigger custom logic via serverless function.

```yaml
gates:
  - task: AzureFunction@1
    inputs:
      function: 'https://myfunc.azurewebsites.net/api/ValidateDeployment'
      key: '$(FunctionKey)'
      method: 'POST'
      body: |
        {
          "releaseId": "$(Release.ReleaseId)",
          "environment": "Production"
        }
      successCriteria: 'eq(root.status, "approved")'
```

**Use Cases**:
- Custom business logic validation
- External system integration
- Complex approval workflows

### 2. Query Azure Monitor Alerts 📊

**Purpose**: Check for active alerts in Azure Monitor.

```yaml
gates:
  - task: AzureMonitorAlerts@0
    inputs:
      connectedServiceNameARM: 'Azure-Connection'
      resourceGroupName: 'myapp-rg'
      severity: '0,1,2'  # Critical, Error, Warning
      alertState: 'New,Acknowledged'
```

**Success Criteria**: No active alerts matching filter.

### 3. Invoke REST API 🌐

**Purpose**: Call any REST API and validate response.

```yaml
gates:
  - task: InvokeRESTAPI@1
    inputs:
      serviceConnection: 'MyAPI'
      method: 'GET'
      urlSuffix: '/api/health'
      headers: |
        {
          "Authorization": "Bearer $(ApiToken)"
        }
      successCriteria: 'eq(root.status, "healthy")'
      waitForCompletion: 'true'
```

**Use Cases**:
- Third-party service validation
- Custom health check endpoints
- External approval systems

### 4. Query Work Items 📝

**Purpose**: Ensure work item query results meet criteria.

```yaml
gates:
  - task: QueryWorkItems@0
    inputs:
      queryId: '12345'  # Saved query ID
      maxThreshold: '0'  # Must return 0 items
      minThreshold: '0'
```

**Common Queries**:
- Blocker bugs = 0
- Critical incidents = 0
- Approved work items > 0

---

## Gate Configuration Parameters

### Common Settings

```yaml
gates:
  - task: <TaskType>
    enabled: true
    continueOnError: false
    alwaysRun: false
    timeoutInMinutes: 60
    retryInterval: 5  # Minutes between retries
```

### Evaluation Options

| Parameter | Description | Default | Example |
|-----------|-------------|---------|---------|
| **Delay before evaluation** | Wait time before first check | 0 min | 5 min |
| **Time between re-evaluation** | Retry interval | 5 min | 10 min |
| **Timeout after which gates fail** | Max wait time | 1 day | 2 hours |
| **On successful gates** | Action after pass | Proceed | Require approval |
| **On failure** | Action after timeout | Reject | Manual intervention |

### Example Configuration

```yaml
preDeploymentGates:
  enabled: true
  timeout: 120  # 2 hours max
  minimumSuccessDuration: 5  # Must pass for 5 min consistently
  samplingInterval: 10  # Check every 10 minutes
  gates:
    - task: QueryAzureMonitor@1
      # ... gate configuration
```

**Behavior**:
```
T+0:  First evaluation (fails)
T+10: Retry (fails)
T+20: Retry (passes)
T+30: Retry (passes) → Consistent success achieved → Proceed
```

---

## Real-World Gate Workflow

### Example: Production Deployment with Multiple Gates

```yaml
stages:
  - stage: Production
    dependsOn: Staging
    jobs:
      - deployment: DeployProduction
        environment: production
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            preDeploy:
              steps:
                - script: echo "Pre-deployment validation"
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    appName: 'myapp-prod'
            postRouteTraffic:
              steps:
                - script: echo "Post-deployment health check"

# Pre-Deployment Gates (configured in Environment)
preDeploymentGates:
  - task: QueryWorkItems@0
    displayName: 'Check for blocker bugs'
    inputs:
      queryId: '$(BlockerBugsQuery)'
      maxThreshold: '0'
      
  - task: AzureMonitorAlerts@0
    displayName: 'Check Azure Monitor'
    inputs:
      resourceGroupName: 'myapp-rg'
      severity: '0,1'  # Critical and Error only
      
  - task: InvokeRESTAPI@1
    displayName: 'Validate staging health'
    inputs:
      serviceConnection: 'StagingAPI'
      urlSuffix: '/api/health'
      successCriteria: 'eq(root.status, "healthy")'

# Post-Deployment Gates (configured in Environment)
postDeploymentGates:
  - task: AzureMonitorAlerts@0
    displayName: 'Monitor for new alerts'
    inputs:
      resourceGroupName: 'myapp-rg'
      alertState: 'New'
      timeRange: '15m'  # Last 15 minutes
      
  - task: InvokeAzureFunction@1
    displayName: 'Run smoke tests'
    inputs:
      function: '$(SmokeTestsFunctionUrl)'
      successCriteria: 'eq(root.allTestsPassed, true)'
```

**Timeline**:
```
14:00 - Staging deployment completes
14:05 - Production stage triggered
14:05 - Pre-deployment gates start
        ├─ Check blocker bugs: ✅ PASS (0 found)
        ├─ Check Azure Monitor: ⏸️ WAIT (1 warning alert)
        └─ Validate staging: ✅ PASS
14:15 - Azure Monitor gate retries: ✅ PASS (alert resolved)
14:15 - All pre-gates passed → Deployment starts
14:20 - Deployment completes
14:20 - Post-deployment gates start
        ├─ Monitor for alerts: ✅ PASS (no new alerts)
        └─ Run smoke tests: ✅ PASS
14:25 - All post-gates passed → Deployment SUCCESS
```

---

## Best Practices

### 🎯 Gate Design

1. **Keep Gates Fast**
   - Target: < 5 minutes per gate evaluation
   - Use efficient queries
   - Cache results where possible

2. **Set Reasonable Timeouts**
   ```yaml
   timeout: 120  # 2 hours (not infinite)
   ```
   - Prevents indefinite waiting
   - Forces issue resolution

3. **Use Minimum Success Duration**
   ```yaml
   minimumSuccessDuration: 5  # Must pass for 5 min consistently
   ```
   - Prevents flaky gate passes
   - Ensures stable state

4. **Combine Multiple Gates**
   ```
   Gate 1: Work items (fast - 10 sec)
   Gate 2: Security scan (medium - 2 min)
   Gate 3: Azure Monitor (fast - 30 sec)
   ```

### 🛡️ Security Best Practices

1. **Store Secrets Securely**
   ```yaml
   inputs:
     apiKey: '$(SecretApiKey)'  # From Azure Key Vault
   ```

2. **Use Service Connections**
   - Don't hardcode credentials
   - Leverage Azure managed identities

3. **Audit Gate Evaluations**
   - Log all gate checks
   - Track failures and reasons

### ⚡ Performance Best Practices

1. **Parallel Gates** (when independent)
   ```yaml
   preDeploymentGates:
     - task: CheckWorkItems
     - task: CheckMonitor
   # Both run in parallel
   ```

2. **Sequential Gates** (when dependent)
   ```yaml
   stage1Gates:
     - task: SecurityScan  # Must complete first
   stage2Gates:
     - task: DeploymentValidation  # Runs after deploy
   ```

3. **Sampling Interval Optimization**
   ```yaml
   samplingInterval: 5  # Frequent checks (5 min)
   # Use for: Fast-changing conditions
   
   samplingInterval: 15  # Infrequent checks (15 min)
   # Use for: Stable conditions (change management)
   ```

---

## Critical Notes

⚠️ **Important Considerations**:

1. **Gates Don't Replace Testing**: Gates validate deployment readiness, not code quality
2. **Pre-Gates Block Deployment**: Failed pre-gate = no deployment
3. **Post-Gates Block Progression**: Failed post-gate = deployment doesn't advance to next stage
4. **Timeouts Cause Failures**: Plan for worst-case gate evaluation time
5. **Flaky Gates Hurt Velocity**: Ensure gates are reliable
6. **Gates Add Delay**: Even fast gates add minutes to deployment time
7. **Combine Gates and Approvals**: Use gates for automation, approvals for business decisions

## Quick Reference

| Gate Type | Use Case | Typical Duration |
|-----------|----------|------------------|
| **Query Work Items** | Check for blocker bugs | 5-30 seconds |
| **Azure Monitor Alerts** | Validate infrastructure health | 10-60 seconds |
| **Invoke REST API** | Custom validation logic | 30 seconds - 5 minutes |
| **Invoke Azure Function** | Complex business logic | 1-10 minutes |
| **Security Scan** | Vulnerability checks | 2-15 minutes |
| **Test Validation** | Quality metrics check | 30 seconds - 2 minutes |

---

**Learn More**:
- [Release Gates Overview](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/approvals/gates)
- [Invoke REST API Task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/invoke-rest-api-v1)
- [Azure Monitor Task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/azure-monitor-v1)

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/explore-release-strategy-recommendations/4-explore-release-gates)
