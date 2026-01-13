# Explore Release Approvals

## Overview
**Continuous Delivery** enables on-demand software delivery through automated processes. However, technical deployment capabilities must integrate with **approval workflows** to establish comprehensive release governance.

**Key Distinction**: 
- **Deployment** = Technical implementation (installing software)
- **Release** = Organizational workflow (approval processes + deployment)

## What Are Release Approvals?

Release approvals control **WHETHER** deliveries proceed, not **HOW** deployments execute.

```
Build Complete → [APPROVAL GATE] → Deploy?
                      ↓
                 ✅ Approved → Deploy
                 ❌ Rejected → Stop
```

### Purpose of Approvals

| Purpose | Description | Example |
|---------|-------------|---------|
| **Governance** | Ensure proper oversight | Product owner signs off |
| **Compliance** | Meet regulatory requirements | SOX four-eyes principle |
| **Risk Management** | Validate changes before production | Security officer reviews |
| **Coordination** | Align with business processes | Marketing approves feature launch |
| **Dependency Management** | Wait for external prerequisites | Database migration completes first |

## Manual vs Automated Approvals

### Manual Approvals

**Definition**: Human validation before deployment proceeds.

#### Why Organizations Use Manual Approvals

1. **Trust Building**: Organizations beginning Continuous Delivery adoption often require human validation
2. **Regulatory Compliance**: Some industries mandate manual oversight
3. **High-Risk Changes**: Database migrations, infrastructure changes
4. **Business Coordination**: Feature releases tied to marketing campaigns

#### Evolution of Manual Approvals

```
Adoption Phase:
├── Early (Low Confidence)
│   └── Manual approvals at every stage
├── Intermediate (Building Trust)
│   └── Manual only for production
└── Advanced (High Maturity)
    └── Automated gates + manual for critical changes only
```

**Trend**: As confidence builds through successful deployments, manual approvals typically evolve into **automated quality gates** and validation checks.

### Automated Approvals (Quality Gates)

**Definition**: Policy-based validation without human intervention.

See **Unit 4: Explore Release Gates** for detailed coverage.

---

## Key Considerations for Release Approval Design

When designing approval workflows, address these critical questions:

### 1️⃣ Purpose: Why Do We Need Approval?

#### Compliance Requirements
**Example: SOX Four-Eyes Principle**
```
Scenario: Financial Services
- Regulation: Sarbanes-Oxley Act
- Requirement: Two different people must be involved
- Implementation:
  • Developer commits code (Person 1)
  • Different person approves production deploy (Person 2)
```

**Implementation**:
```yaml
approvals:
  - type: manual
    approvers:
      - ProductOwner@company.com
      - SecurityOfficer@company.com
    minApprovers: 2  # Both must approve
    segregationOfDuties: true  # Approver ≠ Committer
```

#### Dependency Management
**Example: Database Migration Coordination**
```
Scenario: Application with database changes
- Issue: App version 2.0 requires database schema v2
- Solution: Approval ensures database deploys first
  1. Deploy database schema update
  2. DBA approves after verification
  3. Application deployment proceeds
```

#### Authority Sign-off
**Example: Security Officer Approval**
```
Scenario: Security-sensitive changes
- Requirement: CISO must approve infrastructure changes
- Implementation: Manual approval before production deploy
```

---

### 2️⃣ Approvers: Who Approves?

| Role | Approval Responsibility | Impact on Velocity |
|------|------------------------|-------------------|
| **Product Owner** | Feature acceptance, business value | Medium (usually available) |
| **Security Officer** | Security compliance, vulnerability checks | High (limited availability) |
| **Code Reviewer** | Code quality, best practices | Low (already in PR process) |
| **Release Manager** | Deployment coordination, timing | Low (dedicated role) |
| **Compliance Officer** | Regulatory requirements | Very High (bottleneck risk) |

#### Approver Availability Impact

**❌ Bottleneck Example**:
```
Scenario: Single Security Officer Approval
- Team: 50 developers
- Deployments: 20 per week
- Security Officer: 1 person, frequently in meetings
- Result: Deployments wait 24-48 hours for approval
- Velocity Impact: 75% slower time-to-production
```

**✅ Optimized Approach**:
```
Solution 1: Approval Groups
- Create approval group with 3 security officers
- Any one can approve
- Result: Average wait time reduced to 2-4 hours

Solution 2: Automated Security Gates
- Run automated security scans in pipeline
- Manual approval only for high-risk changes
- Result: 90% of deployments proceed automatically
```

#### Configuring Approvers (Azure DevOps)

**Single Approver**:
```yaml
environments:
  - name: production
    approvals:
      - approver: ProductOwner@company.com
        timeout: 4h
```

**Approval Group (Any One)**:
```yaml
environments:
  - name: production
    approvals:
      - approvers:
          - SecurityTeam@company.com  # Group with multiple members
        minApprovals: 1  # Any one member can approve
```

**Multiple Required Approvers**:
```yaml
environments:
  - name: production
    approvals:
      - approvers:
          - ProductOwner@company.com
          - SecurityOfficer@company.com
        minApprovals: 2  # Both must approve
```

---

### 3️⃣ Timing: When in the Pipeline?

#### Pre-Deployment Approvals

**Definition**: Approval required **before** deployment begins.

```
Build Complete → [PRE-APPROVAL] → Deploy → Success
```

**Use Cases**:
- Production deployments (final gate)
- High-risk environment changes
- Customer demo environments

**Example Configuration**:
```yaml
stages:
  - stage: Production
    dependsOn: Staging
    jobs:
      - deployment: DeployProduction
        environment: production  # Pre-approval configured in environment
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
```

#### Post-Deployment Approvals

**Definition**: Approval required **after** deployment completes.

```
Deploy → Smoke Tests → [POST-APPROVAL] → Mark Complete
```

**Use Cases**:
- Validation of deployment success
- User acceptance testing sign-off
- Staged rollout approval (proceed to next stage)

**Example**:
```
Scenario: Canary Deployment
1. Deploy to 5% of production servers
2. Monitor for 2 hours
3. Product owner approves based on metrics
4. Deploy to remaining 95%
```

#### Multiple Approval Points

```yaml
stages:
  - stage: Production
    jobs:
      - deployment: DeployProduction
        environment: production
        # Pre-deployment approval configured
        
      - job: PostDeploymentValidation
        dependsOn: DeployProduction
        steps:
          - task: RunSmokeTests
          
      - deployment: CompleteDeployment
        dependsOn: PostDeploymentValidation
        environment: production-post-deploy
        # Post-deployment approval configured
```

---

## Approval Timing Strategies

### Strategy 1: Block Entire Pipeline
```
Build → [APPROVAL] → Deploy Dev → Deploy QA → Deploy Prod
         ↑
         ⏸️ WAIT - Everything stops
```

**Pros**: Full control before any deployment  
**Cons**: High delay, blocks lower environments

### Strategy 2: Approve Per Stage
```
Build → Deploy Dev → [APPROVAL] → Deploy QA → [APPROVAL] → Deploy Prod
                     ↑                        ↑
                     Per-stage control
```

**Pros**: Lower environments proceed automatically  
**Cons**: Multiple approval points

### Strategy 3: Scheduled Deployments with Early Approval
```
Friday 10 AM: Approval given
  ↓
Saturday 2 AM: Scheduled deployment executes automatically
```

**Pros**: Decouples approval from deployment timing  
**Cons**: Requires discipline to not deploy immediately

**Implementation**:
```yaml
schedules:
  - cron: "0 2 * * 6"  # Saturday 2 AM
    branches:
      include:
        - release/*
    always: false  # Only if approved release exists
```

---

## Scheduled Deployments Resolve Timing Dependencies

### Problem: Real-Time Human Intervention Required

**Traditional Approach**:
```
4:00 PM: Build completes
4:05 PM: Approval request sent
??? : Waiting for approver (could be in meeting, off-site, asleep)
6:30 PM: Approval received (2.5 hours later)
6:31 PM: Deployment proceeds
```

### Solution: Decouple Approval from Execution

**Scheduled Approach**:
```
Friday 2:00 PM: Build completes
Friday 2:15 PM: Approval request sent
Friday 3:00 PM: Product owner approves
Saturday 2:00 AM: Deployment executes automatically (off-peak hours)
```

**Benefits**:
- ✅ Deployments during low-traffic periods
- ✅ No need for approver to be available during deployment window
- ✅ Time for proper validation before deployment
- ✅ Predictable deployment schedule

---

## Approval Workflow Examples

### Example 1: Basic Production Approval

```yaml
# .azure-pipelines.yml
stages:
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - script: dotnet build
          
  - stage: Deploy_Dev
    dependsOn: Build
    jobs:
      - deployment: DeployDev
        environment: dev  # No approval required
        
  - stage: Deploy_Production
    dependsOn: Deploy_Dev
    jobs:
      - deployment: DeployProduction
        environment: production  # Manual approval required
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
```

**Environment Configuration** (in Azure DevOps Portal):
```
Environments → production → Approvals and checks
├── Add Check: Approvals
│   └── Approvers: ProductOwner@company.com
│   └── Timeout: 24 hours
│   └── Instructions: "Verify staging environment before approving"
```

### Example 2: Multi-Stage with Post-Deployment Approval

```yaml
stages:
  - stage: Deploy_Staging
    jobs:
      - deployment: DeployStaging
        environment: staging
        
  - stage: Validate_Staging
    dependsOn: Deploy_Staging
    jobs:
      - job: RunTests
        steps:
          - task: RunUITests
          - task: RunPerformanceTests
          
  - stage: Approval_To_Production
    dependsOn: Validate_Staging
    jobs:
      - deployment: ApprovalGate
        environment: approval-gate  # Manual approval environment
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Approved for production"
                
  - stage: Deploy_Production
    dependsOn: Approval_To_Production
    jobs:
      - deployment: DeployProduction
        environment: production
```

### Example 3: Segregation of Duties

**Requirement**: Approver must be different from committer.

```yaml
environments:
  - name: production
    approvals:
      - approvers:
          - ReleaseManagers@company.com
        minApprovals: 1
        requiredReviewers: 1
        allowApproverToApproveOwnRuns: false  # Segregation of duties
```

**Enforcement**:
- If user "Alice" triggered the pipeline, Alice cannot approve
- Another member of ReleaseManagers group must approve

---

## Best Practices

### 🎯 Efficiency Tips

1. **Push Approvals Earlier in Pipeline**
   ```
   ❌ Bad: Approve just before production deploy
   ✅ Good: Approve at pull request merge
   
   Rationale: 
   - PR approval validates code quality
   - Deployment becomes automated
   - Faster feedback to developers
   ```

2. **Use Approval Groups, Not Individuals**
   ```
   ❌ Bad: SinglePerson@company.com (bottleneck)
   ✅ Good: ApprovalGroup@company.com (3+ people)
   ```

3. **Set Reasonable Timeouts**
   ```yaml
   approvals:
     - timeout: 4h  # Not 24h or infinite
   ```
   - Prevents stale approvals
   - Forces decision-making
   - Fails fast if approver unavailable

4. **Provide Context to Approvers**
   ```yaml
   approvals:
     - instructions: |
         Review staging environment: https://staging.app.com
         Check release notes: $(Release.Notes.Url)
         Verify no critical bugs in backlog
   ```

### 🛡️ Governance Best Practices

1. **Document Approval Criteria**
   ```markdown
   ## Production Approval Checklist
   - [ ] All tests passing in staging
   - [ ] No P0/P1 bugs in backlog
   - [ ] Performance metrics within SLA
   - [ ] Security scan passed
   - [ ] Release notes reviewed
   - [ ] Rollback plan documented
   ```

2. **Audit Approval History**
   ```
   Track:
   - Who approved
   - When approved
   - Approval duration
   - Rejection reasons
   ```

3. **Implement Approval Bypass for Hotfixes**
   ```yaml
   # Emergency hotfix pipeline (restricted access)
   trigger:
     branches:
       include:
         - hotfix/*
   
   # Expedited approval process (1 approver instead of 2)
   ```

### ⚠️ Anti-Patterns to Avoid

1. **❌ Approval for Every Environment**
   ```
   Build → [Approval] → Dev → [Approval] → QA → [Approval] → Prod
   
   Problem: Excessive bottlenecks, slow feedback
   Better: Auto-deploy to Dev/QA, approve only Prod
   ```

2. **❌ Infinite Approval Timeout**
   ```yaml
   timeout: 0  # Never expires
   
   Problem: Forgotten approvals, stale deployments
   Better: 24-hour timeout, fail and notify
   ```

3. **❌ Approving Without Context**
   ```
   Email: "Please approve Release-123"
   
   Problem: Approver doesn't know what they're approving
   Better: Include release notes, test results, staging link
   ```

4. **❌ Same Person Commits and Approves**
   ```
   Problem: No segregation of duties
   Better: Enforce different approver from committer
   ```

---

## Approval vs Gates: When to Use Each

| Scenario | Use Approval | Use Gate |
|----------|-------------|----------|
| Regulatory compliance requires human sign-off | ✅ | ❌ |
| Validate test results meet threshold | ❌ | ✅ |
| Coordinate with marketing launch | ✅ | ❌ |
| Check for active incidents | ❌ | ✅ |
| Product owner accepts feature | ✅ | ❌ |
| Verify code coverage > 80% | ❌ | ✅ |
| Business stakeholder decision | ✅ | ❌ |
| Automated quality validation | ❌ | ✅ |

**Rule of Thumb**: If criteria is **objective and measurable**, use a gate. If it requires **subjective judgment**, use approval.

---

## Critical Notes

⚠️ **Important Considerations**:

1. **Approvals Add Delay**: Each approval step impacts deployment velocity
   - Measure: Time from build complete to production
   - Optimize: Minimize approval points

2. **Approver Availability = Bottleneck**: Limited approvers slow deployments
   - Solution: Use approval groups with multiple members
   - Fallback: Automated gates where possible

3. **Scheduled Deployments Decouple Timing**: Approval can happen hours/days before deployment
   - Benefit: Deploy during off-peak hours
   - Benefit: No need for real-time approver availability

4. **Shift Approvals Left**: Earlier validation = faster deployments
   - Pull request reviews
   - Branch policies
   - Pre-merge quality checks

5. **Manual Approvals Are Transitional**: Organizations mature toward automated gates
   - Start: Heavy manual approvals
   - Mature: Automated gates + manual for critical only

6. **Compliance May Require Manual**: Some regulations mandate human oversight
   - SOX (Sarbanes-Oxley)
   - HIPAA (Healthcare)
   - PCI-DSS (Payment Card Industry)

## Quick Reference

| Concept | Definition | Example |
|---------|-----------|---------|
| **Pre-Deployment Approval** | Approval before deployment starts | Approve before production deploy |
| **Post-Deployment Approval** | Approval after deployment completes | Sign off after canary validation |
| **Approval Group** | Multiple people who can approve | Security team (any member) |
| **Approval Timeout** | Max time to wait for approval | 4 hours, then fail |
| **Segregation of Duties** | Approver ≠ Committer | Different person must approve |
| **Scheduled Deployment** | Deploy at specific time after approval | Approve Friday, deploy Saturday 2 AM |

---

**Learn More**:
- [Release Approvals and Gates Overview](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/approvals/approvals)
- [Environment Approvals](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals)
- [Segregation of Duties](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals#segregation-of-duties)

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/explore-release-strategy-recommendations/3-explore-release-approvals)
