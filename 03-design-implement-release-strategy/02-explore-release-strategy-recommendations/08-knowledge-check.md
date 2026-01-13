# Module Assessment

## Knowledge Check Questions

Test your understanding of release strategy recommendations with these assessment questions.

---

### Question 1: Trigger Types

**Which trigger type requires explicit human or system intervention to initiate a release?**

A) Continuous deployment trigger  
B) Scheduled trigger  
C) Manual trigger  
D) Automated trigger

<details>
<summary>Click to reveal answer</summary>

**Answer: C) Manual trigger**

**Explanation**:
- **Manual trigger** requires explicit initiation, either by a user clicking "Deploy" or through an API call
- **Continuous deployment triggers** fire automatically when artifacts are created
- **Scheduled triggers** run automatically at predetermined times
- **Automated trigger** is not a specific trigger type in Azure DevOps

**When to use manual triggers**:
- Production deployments requiring human oversight
- Customer demo environments
- Feature releases tied to marketing events
- Hotfix deployments requiring change management approval

</details>

---

### Question 2: Release Approvals

**What is the PRIMARY purpose of release approvals in a deployment pipeline?**

A) To execute deployment tasks faster  
B) To control WHETHER deployments proceed, not HOW they execute  
C) To automatically fix bugs in the application  
D) To compile source code into artifacts

<details>
<summary>Click to reveal answer</summary>

**Answer: B) To control WHETHER deployments proceed, not HOW they execute**

**Explanation**:
- **Release approvals** are governance mechanisms that determine if a deployment should proceed
- They control the "go/no-go" decision, not the technical execution
- **Deployment tasks** define HOW the deployment executes (copy files, restart services, etc.)
- Approvals address organizational trust, compliance, and coordination requirements

**Key distinction**:
- Approval: "Is it safe to deploy?" (business decision)
- Deployment task: "How do we deploy?" (technical implementation)

</details>

---

### Question 3: Pre-Deployment vs Post-Deployment Gates

**Which statement correctly describes the difference between pre-deployment and post-deployment gates?**

A) Pre-deployment gates run after deployment completes, post-deployment gates run before deployment starts  
B) Pre-deployment gates block deployment from starting, post-deployment gates validate deployment success  
C) Pre-deployment gates are manual, post-deployment gates are automated  
D) There is no difference; both terms refer to the same concept

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Pre-deployment gates block deployment from starting, post-deployment gates validate deployment success**

**Explanation**:

**Pre-Deployment Gates**:
- Run BEFORE deployment begins
- Block deployment if conditions not met
- Example: "Don't deploy if there are blocking bugs"

**Post-Deployment Gates**:
- Run AFTER deployment completes
- Validate deployment was successful
- Example: "Confirm no new alerts in Application Insights"

**Both can be automated or manual** (though gates are typically automated)

**Timeline**:
```
Stage Ready → [Pre-Gates] → Deploy → [Post-Gates] → Stage Complete
```

</details>

---

### Question 4: Release Gates vs Manual Approvals

**When should you use a release gate instead of a manual approval?**

A) When the decision requires subjective business judgment  
B) When the criteria is objective and measurable  
C) When only one person should make the decision  
D) When you want to slow down the deployment process

<details>
<summary>Click to reveal answer</summary>

**Answer: B) When the criteria is objective and measurable**

**Explanation**:

**Use Release Gates when**:
- Criteria is objective (code coverage > 80%, zero bugs)
- Can be automated (query API, check metrics)
- Measurable and repeatable
- Fast feedback required

**Use Manual Approvals when**:
- Requires subjective judgment (feature readiness)
- Business decision (marketing timing)
- Compliance requires human sign-off (SOX)
- Coordination with external stakeholders

**Comparison**:
| Scenario | Use Gate | Use Approval |
|----------|----------|--------------|
| Code coverage ≥80% | ✅ | ❌ |
| Product owner accepts feature | ❌ | ✅ |
| Zero critical vulnerabilities | ✅ | ❌ |
| CISO approves infrastructure change | ❌ | ✅ |
| No active monitoring alerts | ✅ | ❌ |

</details>

---

### Question 5: Quality Gates

**Which of the following is NOT a common quality gate implementation?**

A) Zero new blocker issues  
B) Code coverage exceeding 80%  
C) Manager's personal opinion on code style  
D) No dependency vulnerabilities identified

<details>
<summary>Click to reveal answer</summary>

**Answer: C) Manager's personal opinion on code style**

**Explanation**:

**Quality gates must be**:
- **Objective**: Based on measurable criteria
- **Automated**: Can be checked without human intervention
- **Consistent**: Same criteria every time

**Valid quality gates**:
- ✅ Zero blocker bugs (measurable: count = 0)
- ✅ Code coverage >80% (measurable: percentage)
- ✅ No critical vulnerabilities (measurable: security scan results)
- ✅ Performance within SLA (measurable: response time)

**NOT valid quality gates**:
- ❌ "Manager thinks code looks good" (subjective)
- ❌ "Code follows team style guide" (use linter instead)
- ❌ "Feature is ready" (use manual approval)

**Note**: Code style CAN be enforced through automated linters (objective), but not through personal opinion (subjective).

</details>

---

### Question 6: GitOps Principles

**What is the primary principle of GitOps?**

A) Deploy applications using FTP  
B) Use Git repositories as the single source of truth for infrastructure and configuration  
C) Manually configure production servers via SSH  
D) Store configuration in a database

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Use Git repositories as the single source of truth for infrastructure and configuration**

**Explanation**:

**GitOps Core Principle**: "If it's not in Git, it doesn't exist in production"

**Key characteristics**:
- **Declarative**: Desired state defined in Git
- **Versioned**: All changes tracked in Git history
- **Immutable**: Never modify in place, always replace
- **Automated**: Changes automatically synced from Git
- **Auditable**: Full audit trail in Git log

**Traditional vs GitOps**:
```
Traditional:
Manual changes → Production → Hope Git is updated

GitOps:
Git commit → Automated sync → Production
```

**Benefits**:
- Reproducible deployments
- Fast rollback (git revert)
- Complete audit trail
- No configuration drift

</details>

---

### Question 7: Delivery Cadence

**A team wants to deploy to production during off-peak hours (2 AM) without requiring someone to be online at that time. Which trigger type should they use?**

A) Continuous deployment trigger  
B) Scheduled trigger  
C) Manual trigger  
D) On-demand trigger

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Scheduled trigger**

**Explanation**:

**Scheduled triggers** enable time-based automation without human intervention at deployment time.

**Configuration**:
```yaml
schedules:
  - cron: "0 2 * * *"  # Every day at 2:00 AM
    displayName: Off-hours production deploy
    branches:
      include:
        - main
```

**Workflow**:
1. Friday 3 PM: Code merged, build completes
2. Friday 4 PM: Product owner approves release
3. Saturday 2 AM: Scheduled deployment executes automatically
4. No one needs to be online at 2 AM

**Alternatives** (and why they don't work):
- ❌ **Continuous trigger**: Deploys immediately (not at 2 AM)
- ❌ **Manual trigger**: Requires someone to click "Deploy" at 2 AM
- ❌ **On-demand trigger**: Same as manual trigger

</details>

---

### Question 8: Release Gate Evaluation

**A pre-deployment gate is configured with a timeout of 2 hours and checks for zero blocker bugs. If a blocker bug exists, what happens?**

A) Deployment proceeds immediately  
B) Gate retries periodically; if bug resolved within 2 hours, deployment proceeds  
C) Gate fails permanently and never retries  
D) Gate sends an email and waits indefinitely

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Gate retries periodically; if bug resolved within 2 hours, deployment proceeds**

**Explanation**:

**Gate Evaluation Behavior**:
```
T+0:   Gate checks: 1 blocker bug ❌ FAIL
T+5:   Gate retries: 1 blocker bug ❌ FAIL
T+10:  Gate retries: 1 blocker bug ❌ FAIL
T+30:  Developer fixes bug
T+35:  Gate retries: 0 blocker bugs ✅ PASS → Deploy proceeds
```

**If bug not fixed within timeout**:
```
T+0:    Gate checks: 1 blocker bug ❌ FAIL
T+5-115: Gates retry every 5 minutes: ❌ FAIL
T+120:  TIMEOUT reached → Deployment REJECTED
```

**Configuration**:
```yaml
gates:
  - task: QueryWorkItems@0
    inputs:
      maxThreshold: '0'
    timeout: 120  # 2 hours in minutes
    retryInterval: 5  # Retry every 5 minutes
```

**Key points**:
- Gates **retry** automatically at intervals
- Gates **don't fail permanently** on first failure
- Gates **timeout** after configured duration
- Timeouts result in deployment rejection

</details>

---

### Question 9: Application Insights Integration

**Which built-in gate type in Azure DevOps is used to query Application Insights for active alerts?**

A) Invoke Azure Function  
B) Query Azure Monitor alerts  
C) Invoke REST API  
D) Query work items

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Query Azure Monitor alerts**

**Explanation**:

**Query Azure Monitor alerts** is one of the four built-in gate types in Azure DevOps.

**Four Built-In Gates**:
1. **Invoke Azure Function**: Trigger custom logic via serverless function
2. **Query Azure Monitor alerts**: Check for active alerts in Azure Monitor/Application Insights
3. **Invoke REST API**: Call any REST API and validate response
4. **Query work items**: Ensure work item query results meet criteria

**Query Azure Monitor Configuration**:
```yaml
gates:
  - task: AzureMonitorAlerts@0
    inputs:
      connectedServiceNameARM: 'Azure-Connection'
      resourceGroupName: 'myapp-rg'
      severity: '0,1,2'  # Critical, Error, Warning
      alertState: 'New,Acknowledged'
```

**Use case**:
```
Post-deployment gate for production:
- Wait 5 minutes after deployment
- Query Application Insights for alerts
- If 0 alerts: PASS → Deployment successful
- If alerts exist: FAIL → Investigate or rollback
```

</details>

---

### Question 10: Segregation of Duties

**Why is segregation of duties important in release approvals?**

A) To make deployments slower  
B) To ensure the person who committed code is different from the person who approves production deployment  
C) To require multiple people to write the same code  
D) To prevent developers from using Git

<details>
<summary>Click to reveal answer</summary>

**Answer: B) To ensure the person who committed code is different from the person who approves production deployment**

**Explanation**:

**Segregation of Duties (SoD)**: A compliance principle requiring different people to perform different steps in a critical process.

**Why it matters**:
- **Prevents fraud**: One person can't make unauthorized changes
- **Compliance**: Required by regulations (SOX, PCI-DSS)
- **Quality**: Independent review catches mistakes
- **Accountability**: Clear responsibility for each step

**Example**:
```
❌ Violates SoD:
- Alice commits code
- Alice approves production deployment
- No independent review

✅ Enforces SoD:
- Alice commits code
- Bob approves production deployment
- Independent validation
```

**Implementation**:
```yaml
environments:
  - name: production
    approvals:
      - approvers:
          - ReleaseManagers@company.com
        allowApproverToApproveOwnRuns: false  # SoD enforcement
```

**Regulations requiring SoD**:
- **SOX** (Sarbanes-Oxley): Financial reporting
- **PCI-DSS**: Payment card processing
- **HIPAA**: Healthcare data
- **GDPR**: Data privacy (in some contexts)

</details>

---

## Assessment Summary

After completing this knowledge check, you should be able to:

✅ Distinguish between trigger types (continuous, scheduled, manual)  
✅ Explain the purpose of release approvals vs release gates  
✅ Describe pre-deployment vs post-deployment gates  
✅ Identify when to use gates vs manual approvals  
✅ Define quality gate implementations  
✅ Explain GitOps principles  
✅ Configure appropriate delivery cadence strategies  
✅ Understand gate evaluation behavior  
✅ Use Application Insights integration in gates  
✅ Implement segregation of duties compliance

---

## Review Recommendations

If you struggled with any questions, review these units:

- **Questions 1, 7**: Unit 2 (Delivery Cadence and Triggers)
- **Questions 2, 10**: Unit 3 (Release Approvals)
- **Questions 3, 4, 8, 9**: Unit 4 (Release Gates)
- **Question 5**: Unit 5 (Quality Gates)
- **Question 6**: Unit 6 (GitOps Strategy)

---

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/explore-release-strategy-recommendations/8-knowledge-check)
