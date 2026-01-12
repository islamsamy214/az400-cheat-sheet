# Examine Considerations for Deployment to Stages

When designing a release strategy, a critical decision is **when** to deploy to each stage (environment). This unit explores deployment cadence, trigger selection, and considerations for timing deployments to different environments.

## Deployment Cadence Overview

**Deployment Cadence**: The frequency and timing pattern for deploying applications to environments.

```
Deployment Frequency Spectrum

Continuous Deployment (Multiple times per day)
        ↓
Scheduled Daily (Once per day at specific time)
        ↓
Weekly Releases (Once per week)
        ↓
Biweekly/Monthly (Planned releases)
        ↓
Manual/Ad-hoc (On-demand only)
```

### Typical Cadence by Environment

```
Environment Deployment Patterns

Development:
  Frequency: Continuous (every commit)
  Timing: Immediate after build
  Downtime: Acceptable anytime
  Users: Developers only

QA/Test:
  Frequency: Multiple times per day
  Timing: After dev deployment succeeds
  Downtime: Avoid during active testing
  Users: QA team, automated tests

Staging:
  Frequency: Daily or per sprint
  Timing: Off-hours (minimize user impact)
  Downtime: Minimal tolerance
  Users: Internal stakeholders, pre-production validation

Production:
  Frequency: Weekly, biweekly, or as needed
  Timing: Maintenance windows, low-traffic periods
  Downtime: Zero or near-zero tolerance
  Users: Customers, external users
```

## Key Considerations

### 1. Continuous Delivery vs. Continuous Deployment

**Continuous Delivery**:
```
Continuous Delivery (CD)

Code Commit → Build → Test → Ready to Deploy
                                     ↓
                            [Manual Approval]
                                     ↓
                               Deploy to Prod

Key: Always ready to deploy, but deployment is manual
```

**Continuous Deployment**:
```
Continuous Deployment

Code Commit → Build → Test → Auto-Deploy to Prod

Key: Fully automated deployment to production
```

**When to Use Each**:

| Scenario | Recommendation |
|----------|----------------|
| **High-maturity DevOps team** | Continuous Deployment ✅ |
| **Regulated industry (finance, healthcare)** | Continuous Delivery (manual approval) ✅ |
| **Consumer-facing app with high traffic** | Continuous Deployment with canary/blue-green ✅ |
| **Enterprise SaaS** | Continuous Delivery (scheduled releases) ✅ |
| **Internal tools** | Continuous Deployment ✅ |
| **Mission-critical systems** | Continuous Delivery (manual gates) ✅ |

### 2. Target Environment Availability

**Question**: Is the target environment available for deployment?

```
Environment Availability Patterns

Shared QA Environment:
  ┌─────────────────────────────────┐
  │  QA Team Schedule               │
  ├─────────────────────────────────┤
  │  9 AM - 5 PM: Active testing    │ ❌ Don't deploy
  │  5 PM - 9 AM: Available         │ ✅ Deploy OK
  │  Weekends: Available            │ ✅ Deploy OK
  └─────────────────────────────────┘

Dedicated Dev Environment:
  ┌─────────────────────────────────┐
  │  Always available for dev team  │ ✅ Deploy anytime
  └─────────────────────────────────┘

Production (24/7 Service):
  ┌─────────────────────────────────┐
  │  Mon-Fri: High traffic          │ ⚠️ Deploy carefully
  │  Sat-Sun: Lower traffic         │ ✅ Preferred window
  │  2-4 AM: Lowest traffic         │ ✅ Ideal window
  └─────────────────────────────────┘
```

**Configuration Example**:
```yaml
# Prevent QA deployments during business hours
stages:
- stage: Deploy_QA
  condition: |
    or(
      eq(variables['Build.Reason'], 'Manual'),
      and(
        eq(variables['Build.Reason'], 'Schedule'),
        eq(variables['Build.SourceBranch'], 'refs/heads/main')
      )
    )
  
  # Schedule: Deploy after business hours
  schedules:
  - cron: "0 18 * * 1-5"  # 6 PM Monday-Friday
    displayName: After Hours QA Deployment
    branches:
      include:
      - main
```

### 3. Multiple Daily Deployments

**Is multiple daily deployment appropriate?**

```
Evaluation Factors:

Team Readiness:
  ✅ Automated tests cover critical paths
  ✅ Monitoring and alerting in place
  ✅ Rollback process tested
  ✅ Team comfortable with automation
  ❌ Manual testing required
  ❌ Deployment process is manual
  ❌ No automated rollback

Application Characteristics:
  ✅ Microservices (isolated deployments)
  ✅ Feature flags enabled
  ✅ Backward-compatible changes
  ✅ Database migrations automated
  ❌ Monolithic application
  ❌ Breaking API changes
  ❌ Manual database updates required

User Impact:
  ✅ Internal tools (users tolerate changes)
  ✅ B2B SaaS (users informed)
  ✅ Blue-green deployment (zero downtime)
  ❌ Consumer app (users expect stability)
  ❌ High-transaction system (downtime costly)
```

**Example - High-Frequency Deployment** (appropriate):
```yaml
# Microservice deployment (independent, low risk)
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - services/payment-api/**

stages:
- stage: Deploy_Dev
  jobs:
  - deployment: DeployPaymentAPI
    environment: dev
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            inputs:
              action: deploy
              manifests: services/payment-api/k8s/

# Deploys immediately after every commit to main
# Low risk: Isolated microservice
# Zero downtime: Kubernetes rolling update
```

**Example - Scheduled Deployment** (appropriate):
```yaml
# Monolithic application (higher risk, coordination needed)
trigger: none  # No automatic deployment

schedules:
- cron: "0 20 * * 2,4"  # 8 PM Tuesday and Thursday
  displayName: Biweekly Production Deployment
  branches:
    include:
    - release/*

stages:
- stage: Deploy_Prod
  jobs:
  - deployment: DeployMonolith
    environment: production  # Requires manual approval
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              appName: 'myapp-prod'
              deploymentMethod: 'auto'

# Scheduled deployments
# Manual approval required
# Coordinated release window
```

### 4. Deployment Duration and Downtime

**Consideration**: How long does deployment take? Is downtime acceptable?

```
Deployment Duration Impact

Fast Deployment (< 5 minutes):
  ✅ Can deploy during business hours
  ✅ Multiple deployments per day feasible
  ✅ Quick rollback possible
  Examples: Container updates, serverless functions

Medium Deployment (5-30 minutes):
  ⚠️ Deploy during low-traffic periods
  ⚠️ Limit to 1-2 deployments per day
  Examples: Web application updates, database migrations

Long Deployment (> 30 minutes):
  ❌ Require maintenance windows
  ❌ Scheduled deployments only
  ❌ Extensive coordination needed
  Examples: Large monoliths, complex database migrations
```

**Downtime Strategies**:

```
Zero-Downtime Patterns

Blue-Green Deployment:
  ┌────────┐  ┌────────┐
  │ Blue   │  │ Green  │
  │ (Old)  │  │ (New)  │
  └────────┘  └────────┘
       ↓           ↑
  [Load Balancer]
  
  1. Deploy to Green (Blue still serving traffic)
  2. Test Green
  3. Switch traffic to Green
  4. Zero downtime! ✅
  Duration: 10-15 minutes
  Downtime: 0 seconds

Rolling Update (Kubernetes):
  Pod 1 (Old) → Pod 1 (New)
  Pod 2 (Old) → Pod 2 (New)  (Gradual replacement)
  Pod 3 (Old) → Pod 3 (New)
  
  Duration: 5-10 minutes
  Downtime: 0 seconds
  Always have healthy pods serving traffic

Canary Deployment:
  Old: 95% traffic
  New: 5% traffic (canary)
  
  Monitor canary health
  Gradually shift: 90%, 70%, 50%, 0% old
  
  Duration: 1-4 hours
  Downtime: 0 seconds
  Gradual rollout minimizes risk
```

**Example - Zero-Downtime Deployment**:
```yaml
# Azure App Service with deployment slots
- stage: Deploy_Prod
  jobs:
  - job: DeployToSlot
    steps:
    # 1. Deploy to staging slot
    - task: AzureWebApp@1
      inputs:
        azureSubscription: 'Production'
        appName: 'myapp-prod'
        slotName: 'staging'
        package: '$(Pipeline.Workspace)/drop/webapp.zip'
    
    # 2. Warm up staging slot
    - task: PowerShell@2
      inputs:
        targetType: 'inline'
        script: |
          Start-Sleep -Seconds 30  # Allow app to start
          Invoke-WebRequest -Uri 'https://myapp-prod-staging.azurewebsites.net/health'
    
    # 3. Swap slots (near-instant swap)
    - task: AzureAppServiceManage@0
      inputs:
        azureSubscription: 'Production'
        action: 'Swap Slots'
        webAppName: 'myapp-prod'
        resourceGroupName: 'Production-RG'
        sourceSlot: 'staging'
        targetSlot: 'production'
    
    # Result: Zero downtime, instant rollback capability
```

### 5. Team Usage Patterns

**Consideration**: Who uses the environment and when?

```
Environment Usage Analysis

Single Team Environment (Dev):
  Users: 5 developers
  Usage: 9 AM - 6 PM Monday-Friday
  Deployment Strategy:
    ✅ Continuous deployment
    ✅ Automatic after every commit
    ✅ Developers expect frequent updates

Shared Environment (QA):
  Users: QA team (10 people) + Automated tests
  Usage: 8 AM - 8 PM Monday-Friday
  Deployment Strategy:
    ⚠️ Avoid deployments during active testing
    ✅ Schedule: Daily at 6 AM (before testers arrive)
    ✅ Schedule: Nightly at 8 PM (after testers leave)
    ❌ Don't deploy during working hours

Production (24/7 Global Service):
  Users: Customers worldwide
  Usage: 24/7, varying traffic by region
  Deployment Strategy:
    ⚠️ Identify low-traffic windows per region
    ✅ Deploy during lowest traffic (e.g., 2-4 AM local time)
    ✅ Use rolling updates / canary deployments
    ✅ Monitor traffic patterns for optimal timing
```

**Traffic Pattern Analysis**:
```
Production Traffic (Typical Web App)

Hour:  00 02 04 06 08 10 12 14 16 18 20 22
       ────────────────────────────────────
Mon    ██░░░░░░████████████████████████░░░░
Tue    ██░░░░░░████████████████████████░░░░
Wed    ██░░░░░░████████████████████████░░░░
Thu    ██░░░░░░████████████████████████░░░░
Fri    ██░░░░░░████████████████████████████
Sat    ████░░░░░░░░░░░░██████████████░░░░░░
Sun    ████░░░░░░░░░░░░░░░░██████░░░░░░░░░░

Legend:
░░ Low traffic (ideal deployment window)
██ Medium traffic (acceptable with zero-downtime)
████ High traffic (avoid deployments)

Recommendation:
✅ Best: 2-6 AM weekdays (░░)
✅ Good: Weekends 2-8 AM (░░)
⚠️ Acceptable: Weekdays 10 AM - 2 PM with blue-green
❌ Avoid: Friday 6 PM - Sunday 12 PM (peak usage)
```

### 6. User Expectations and Tolerance

**Consideration**: How do users react to frequent updates?

```
User Tolerance by Application Type

Internal Tools:
  Tolerance: High ✅
  Updates: Users expect frequent improvements
  Deployment: Continuous deployment OK
  Communication: Slack/email notification sufficient
  Example: HR portal, expense system

B2B SaaS:
  Tolerance: Medium ⚠️
  Updates: Users appreciate features but want stability
  Deployment: Weekly/biweekly releases
  Communication: Release notes, in-app notifications
  Example: CRM, project management tools

Consumer Apps:
  Tolerance: Low ❌
  Updates: Users expect seamless experience
  Deployment: Gradual rollouts, A/B testing
  Communication: Optional (transparent updates)
  Example: Social media, e-commerce

Mission-Critical Systems:
  Tolerance: Very Low ❌
  Updates: Users demand zero disruption
  Deployment: Planned maintenance windows only
  Communication: Advance notice, CAB approval
  Example: Banking, healthcare, emergency services
```

**Communication Strategy**:
```yaml
# Release notification pipeline
- stage: Notify_Stakeholders
  dependsOn: Deploy_Prod
  condition: succeeded()
  jobs:
  - job: SendNotifications
    steps:
    # Internal team notification
    - task: PowerShell@2
      displayName: 'Notify Slack'
      inputs:
        targetType: 'inline'
        script: |
          $payload = @{
            text = "🚀 Production deployment complete: v$(Build.BuildNumber)"
            channel = "#releases"
          }
          Invoke-RestMethod -Uri $env:SLACK_WEBHOOK -Method Post -Body ($payload | ConvertTo-Json)
    
    # Customer notification (for major releases)
    - task: PowerShell@2
      displayName: 'Update Status Page'
      inputs:
        targetType: 'inline'
        script: |
          # Update status page with release notes
          Invoke-RestMethod -Uri 'https://status.myapp.com/api/incident' -Method Post -Body @{
            status = "completed"
            message = "Version $(Build.BuildNumber) deployed successfully"
          }
```

## Deployment Trigger Patterns

### 1. Manual Triggers

**When to Use**:
- Production deployments requiring explicit approval
- Critical hotfixes
- Coordinated releases across multiple services
- Maintenance window deployments

**Example**:
```yaml
# Manual-only production deployment
trigger: none  # Disable automatic triggers

stages:
- stage: Deploy_Prod
  jobs:
  - deployment: ManualDeploy
    environment: production
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Manual deployment initiated by $(Build.RequestedFor)"
```

### 2. Scheduled Triggers

**When to Use**:
- Off-hours deployments to minimize user impact
- Regular release cadence (weekly, biweekly)
- Batch deployments of multiple services

**Example**:
```yaml
# Scheduled nightly deployment to QA
schedules:
- cron: "0 2 * * 1-5"  # 2 AM Monday-Friday
  displayName: Nightly QA Deployment
  branches:
    include:
    - develop
  always: false  # Only run if code changed

- cron: "0 20 * * 5"  # 8 PM Friday
  displayName: Weekly Production Deployment
  branches:
    include:
    - main
  always: false
```

### 3. Continuous Deployment Triggers

**When to Use**:
- Development environments
- Internal tools
- High-maturity DevOps teams
- Microservices with isolated deployments

**Example**:
```yaml
# Continuous deployment to dev
trigger:
  branches:
    include:
    - main
    - develop
  paths:
    include:
    - src/**
    exclude:
    - docs/**

stages:
- stage: Deploy_Dev
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/develop')
  jobs:
  - deployment: AutoDeploy
    environment: dev
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
```

## Best Practices

### 1. Progressive Deployment Strategy

```
Gradual Rollout Pattern

Hour 0: Deploy to Dev (immediate, continuous)
        ↓
Hour 2: Deploy to QA (after dev validation)
        ↓
Day 1:  Deploy to Staging (after QA tests pass)
        ↓
Day 2:  Deploy to Prod Canary (5% traffic)
        ↓
Day 3:  Deploy to Prod (100% traffic)
```

### 2. Environment-Specific Timing

```yaml
# Different timing for different environments
stages:
- stage: Deploy_Dev
  # Immediate deployment
  condition: succeeded()
  
- stage: Deploy_QA
  # Deploy after 2-hour delay (smoke tests complete)
  dependsOn: Deploy_Dev
  condition: succeeded()
  jobs:
  - deployment: DeployQA
    pool: server  # Delay job
    steps:
    - task: Delay@1
      inputs:
        delayForMinutes: 120

- stage: Deploy_Prod
  # Manual approval + scheduled window
  dependsOn: Deploy_QA
  condition: |
    and(
      succeeded(),
      or(
        eq(variables['Build.Reason'], 'Manual'),
        eq(variables['IsMaintenanceWindow'], 'true')
      )
    )
```

### 3. Health Check Gates

```yaml
# Don't deploy if environment is unhealthy
- stage: Deploy_Prod
  jobs:
  - deployment: Deploy
    environment: production
    strategy:
      runOnce:
        preDeploy:
          steps:
          # Check current production health
          - task: PowerShell@2
            displayName: 'Pre-Deployment Health Check'
            inputs:
              targetType: 'inline'
              script: |
                $health = Invoke-RestMethod -Uri 'https://myapp-prod/health'
                if ($health.status -ne 'healthy') {
                  throw "Production is unhealthy! Aborting deployment."
                }
```

## Critical Notes

🎯 **Know Your Users**: Deployment timing should match user patterns—deploy when usage is lowest.

💡 **Progressive Rollout**: Start with dev, progress through QA/staging, finally production—catch issues early.

⚠️ **Communication**: Inform stakeholders of deployment schedules—especially for user-facing applications.

📊 **Monitor Traffic**: Use analytics to identify low-traffic windows for production deployments.

🔄 **Zero Downtime**: For 24/7 services, use blue-green, rolling updates, or canary deployments.

✨ **Automate Where Safe**: Dev and QA can have continuous deployment; production may need approval.

## Quick Reference

### Deployment Timing Decision Matrix

```
Environment Type → Deployment Strategy

Development:
  ✅ Continuous deployment (immediate)
  ✅ Any time of day
  ✅ No approval needed

QA:
  ✅ Scheduled (off-hours)
  ✅ Multiple times per day OK
  ⚠️ Avoid during active testing

Staging:
  ✅ Scheduled (daily or weekly)
  ✅ Manual approval
  ⚠️ Coordinate with stakeholders

Production:
  ✅ Scheduled maintenance windows
  ✅ Manual approval required
  ✅ Low-traffic periods
  ✅ Communication plan
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline-devops/6-examine-considerations-for-deployment-to-stages)
