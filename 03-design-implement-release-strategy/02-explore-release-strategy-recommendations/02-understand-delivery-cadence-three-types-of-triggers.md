# Understand the Delivery Cadence and Three Types of Triggers

## Overview
Release pipelines and stages utilize **triggers** to initiate deployment processes. Understanding trigger types is essential for designing effective release strategies that balance automation with control.

**Key Concept**: Triggers determine **WHEN** deployments occur, while release definitions determine **HOW** they execute.

## Three Primary Trigger Types

### 1️⃣ Continuous Deployment Trigger

**Definition**: Automatically initiates releases when build completion creates new artifacts.

#### How It Works
```
Source Commit → Build Pipeline → Artifacts Created → 🚀 Immediate Deployment
```

#### Characteristics

| Aspect | Details |
|--------|---------|
| **Automation Level** | Fully automated |
| **Speed** | Immediate (seconds to minutes) |
| **Human Involvement** | None required |
| **Use Case** | Dev/test environments, CI/CD pipelines |
| **Risk Level** | Higher (no manual validation) |

#### Example Configuration (Azure Pipelines)
```yaml
trigger:
  branches:
    include:
      - main
      - develop

resources:
  pipelines:
    - pipeline: buildPipeline
      source: MyBuildPipeline
      trigger:
        branches:
          include:
            - main
```

#### When to Use Continuous Triggers

✅ **Good For**:
- Development environments
- QA/Test environments
- Staging environments (for mature teams)
- Services with comprehensive automated testing
- Microservices with isolated impact

❌ **Avoid For**:
- Production (without safety gates)
- Regulated industries without compliance checks
- Systems requiring manual validation
- Monolithic apps with high blast radius

#### Real-World Example
```
E-commerce Platform - Development Environment:
1. Developer commits fix to 'develop' branch
2. Build pipeline runs tests and creates artifacts
3. CD trigger fires automatically
4. Deploy to dev environment within 5 minutes
5. Developers can test immediately
```

---

### 2️⃣ Scheduled Triggers

**Definition**: Enable time-based release automation at predetermined intervals.

#### How It Works
```
Cron Schedule Defined → Time Reached → 🚀 Scheduled Deployment
```

#### Characteristics

| Aspect | Details |
|--------|---------|
| **Automation Level** | Fully automated (time-based) |
| **Speed** | Predictable (runs at set times) |
| **Human Involvement** | None during execution |
| **Use Case** | Off-hours deployments, batch updates |
| **Risk Level** | Medium (planned windows) |

#### Configuration Examples

**Daily Deployment**:
```yaml
schedules:
  - cron: "0 3 * * *"  # Every day at 3:00 AM UTC
    displayName: Daily Production Deployment
    branches:
      include:
        - main
    always: false  # Only run if there are new changes
```

**Multiple Daily Schedules**:
```yaml
schedules:
  - cron: "0 12 * * *"  # Noon deployment
    displayName: Midday QA Deploy
    
  - cron: "0 0 * * *"   # Midnight deployment
    displayName: Overnight Production Deploy
```

#### Common Schedule Patterns

| Pattern | Cron Expression | Use Case |
|---------|-----------------|----------|
| **Every day at 3 AM** | `0 3 * * *` | Off-hours production deploys |
| **Every weekday at noon** | `0 12 * * 1-5` | Business hours QA deploys |
| **Every Sunday at 2 AM** | `0 2 * * 0` | Weekly maintenance window |
| **Every hour** | `0 * * * *` | Frequent staging updates |
| **Every 4 hours** | `0 */4 * * *` | Regular sync deployments |

#### When to Use Scheduled Triggers

✅ **Good For**:
- Production deployments during off-peak hours
- Batch processing systems
- Data migration jobs
- Environments with specific maintenance windows
- Deployments requiring downtime
- Coordinating with user notifications

❌ **Avoid For**:
- Critical hotfixes (need immediate deployment)
- Development environments (prefer continuous)
- Systems requiring on-demand deployments

#### Real-World Example
```
Banking Application - Production:
1. Schedule: Every Sunday at 2:00 AM (low traffic)
2. Maintenance window: 2:00 AM - 4:00 AM
3. Automated deployment of weekly updates
4. Rollback automated if health checks fail by 3:00 AM
5. System back online by 4:00 AM
6. Users notified via email on Friday
```

---

### 3️⃣ Manual Trigger

**Definition**: Requires explicit human or system intervention to initiate releases.

#### How It Works
```
User Decides → Clicks "Deploy" Button → 🚀 Manual Deployment Initiated
```

**OR**

```
External System → API Call → 🚀 Programmatic Deployment Triggered
```

#### Characteristics

| Aspect | Details |
|--------|---------|
| **Automation Level** | Manual initiation, automated execution |
| **Speed** | On-demand (immediate when triggered) |
| **Human Involvement** | Required to start |
| **Use Case** | Production, demo environments, hotfixes |
| **Risk Level** | Lower (human validation before deploy) |

#### Initiation Methods

**1. UI-Based Manual Trigger**
```
Azure DevOps Portal:
Pipelines → Releases → Select Release Definition → Create Release → Deploy
```

**2. API-Based Trigger**
```bash
# Azure DevOps REST API
curl -X POST \
  'https://dev.azure.com/{organization}/{project}/_apis/release/releases?api-version=7.0' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {PAT}' \
  -d '{
    "definitionId": 1,
    "description": "Manual hotfix deployment",
    "artifacts": [...]
  }'
```

**3. CLI-Based Trigger**
```bash
# Azure CLI
az pipelines release create \
  --definition-id 1 \
  --description "Manual production deploy"
```

#### When to Use Manual Triggers

✅ **Good For**:
- Production environments (with high risk)
- Customer demo environments
- Hotfix deployments
- Feature releases tied to marketing events
- Systems requiring change management approval
- Initial deployments to new environments

❌ **Avoid For**:
- High-frequency deployments
- Development environments
- Situations requiring 24/7 deployment capability
- Teams practicing true continuous deployment

#### Real-World Example
```
SaaS Product Launch:
1. Marketing announces feature launch for 10:00 AM PST
2. Engineering prepares release in advance
3. Product manager reviews staging environment at 9:30 AM
4. At 10:00 AM, PM clicks "Deploy to Production"
5. Automated deployment executes
6. Marketing publishes announcement simultaneously
```

---

## Trigger Comparison Matrix

| Criteria | Continuous | Scheduled | Manual |
|----------|-----------|-----------|--------|
| **Automation** | Full | Full | Initiation only |
| **Speed to Deploy** | Fastest (minutes) | Predictable | On-demand |
| **Human Control** | Minimal | None during execution | Full control |
| **Best For** | Dev/QA | Off-hours production | Critical production |
| **Deployment Frequency** | Very high | Medium | Variable |
| **Suitable for Production** | With gates | Yes | Yes |
| **Risk Management** | Requires safety gates | Planned windows | Human oversight |
| **Rollback Speed** | Immediate | Next schedule | Immediate |

## Combining Trigger Types

### Multi-Environment Strategy

```
Development:     Continuous trigger (every commit)
    ↓
QA/Testing:      Continuous trigger (after dev success)
    ↓
Staging:         Scheduled trigger (daily at noon)
    ↓
Production:      Manual trigger (on-demand)
```

### Hybrid Approach with Gates

```yaml
# YAML Pipeline with multiple triggers
trigger:
  branches:
    include:
      - main  # Continuous for dev

schedules:
  - cron: "0 3 * * *"  # Scheduled for staging
    branches:
      include:
        - release/*

stages:
  - stage: Dev
    jobs:
      - deployment: DeployDev
        environment: dev
        # Automatic deployment
        
  - stage: Staging
    dependsOn: Dev
    jobs:
      - deployment: DeployStaging
        environment: staging
        # Runs on schedule
        
  - stage: Production
    dependsOn: Staging
    jobs:
      - deployment: DeployProduction
        environment: production
        # Manual approval required (configured in environment)
```

## Delivery Cadence Considerations

### High-Frequency Deployments (Multiple per Day)
- **Trigger**: Continuous
- **Requirements**: Excellent automated testing, feature flags, rollback capability
- **Risk**: Lower per deployment (smaller changes)

### Medium-Frequency Deployments (Daily/Weekly)
- **Trigger**: Scheduled or Manual
- **Requirements**: Comprehensive testing, planned maintenance windows
- **Risk**: Medium (accumulated changes)

### Low-Frequency Deployments (Monthly/Quarterly)
- **Trigger**: Manual
- **Requirements**: Extensive validation, change management approval
- **Risk**: Higher (large change batches)

## Critical Notes

⚠️ **Important Considerations**:

1. **Continuous ≠ Continuous Deployment**: 
   - Continuous *Integration*: Build and test on every commit
   - Continuous *Deployment*: Deploy to production on every commit
   
2. **Scheduled Triggers Don't Wait**: If build fails, scheduled deploy may fail or use old artifacts

3. **Manual Doesn't Mean Slow**: Execution is still automated, only initiation is manual

4. **Combine with Gates**: Any trigger type can use pre/post-deployment gates

5. **Time Zones Matter**: Cron schedules use UTC, convert to your local time

6. **Artifact Retention**: Ensure artifacts are retained long enough for scheduled deployments

## Best Practices

### 🎯 Choosing the Right Trigger

```
Decision Tree:
├── Is it Production? 
│   ├── Yes → Manual (with gates) or Scheduled (off-hours)
│   └── No → Continue
├── Do changes need immediate testing?
│   ├── Yes → Continuous
│   └── No → Continue
├── Is there a maintenance window?
│   ├── Yes → Scheduled
│   └── No → Manual or Continuous
```

### 🛡️ Safety Practices

1. **Always Use Gates with Continuous Triggers in Production**
   ```yaml
   - stage: Production
     dependsOn: Staging
     trigger: continuous
     preDeploymentGates:
       - task: QueryAzureMonitor
       - task: CheckWorkItems
   ```

2. **Schedule Deployments During Low Traffic**
   - Analyze user traffic patterns
   - Choose maintenance windows wisely
   - Communicate schedules to users

3. **Implement Circuit Breakers**
   ```
   If error rate > 5% → Halt deployments → Notify team
   ```

4. **Test Scheduled Triggers**
   - Set test schedule for near-future time
   - Verify execution as expected
   - Update to production schedule

## Quick Reference

| Trigger Type | Azure Pipelines YAML | Use Case |
|--------------|----------------------|----------|
| **Continuous** | `trigger: - main` | Auto-deploy after build |
| **Scheduled** | `schedules: - cron: "0 3 * * *"` | Off-hours deployment |
| **Manual** | (No trigger, manual release creation) | On-demand production |

---

**Learn More**:
- [Release and Stage Triggers](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/triggers)
- [YAML Schema: Triggers](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/trigger)
- [Schedule Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/scheduled-triggers)

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/explore-release-strategy-recommendations/2-understand-delivery-cadence-three-types-of-triggers)
