# Knowledge Check

⏱️ **Duration**: ~8 minutes | 📚 **Type**: Assessment

## Overview

Test your understanding of **automated health inspection, events/notifications, service hooks, quality measurement, release notes documentation, and release management tool selection**. This knowledge check covers key concepts from Module 5.

---

## Questions

### Question 1: Automated Health Monitoring Capabilities

**Which of the following is NOT a core capability of automated release health monitoring?**

A) Real-time status awareness  
B) Manual deployment verification  
C) Automated intervention (rollback triggers)  
D) Release traceability across environments  

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Manual deployment verification**

**Explanation**: Automated health monitoring focuses on **automated** capabilities. The six core capabilities are:
1. Real-time status awareness (dashboards)
2. Success/failure detection (automated identification)
3. Quality assessment (metrics tracking)
4. Release traceability (audit trails)
5. Automated intervention (rollback triggers)
6. Visual analytics (trend analysis)

Manual verification is an **anti-pattern** that automated monitoring aims to eliminate (reduces 40 min/failure to 1 min with automation).

**References**: Unit 2 - Automate Inspection of Health
</details>

---

### Question 2: Alert Fatigue Problem

**Your team receives 200+ notifications per day from your CI/CD pipeline, and developers now ignore 80% of them. What is the PRIMARY cause of this alert fatigue?**

A) Too few notification channels (only email)  
B) Over-configured notifications without strategic filtering  
C) Lack of webhook integrations  
D) Insufficient approvers for production deployments  

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Over-configured notifications without strategic filtering**

**Explanation**: **Alert fatigue** occurs when teams receive too many low-value notifications, causing them to ignore even critical alerts. The problem is **over-configuration** (notifying on every event) rather than under-configuration.

**Solution strategies**:
- Filter by relevance (only actionable events)
- Filter by criticality (high-severity only)
- Filter by environment (production failures only, not dev)
- Target correct audiences (actionable notifications to developers, informational to stakeholders)

**Impact**: Studies show 80%+ notifications ignored when over-configured. Strategic filtering can save 39 min per failure, 26 hours/month per developer.

**References**: Unit 3 - Explore Events and Notifications
</details>

---

### Question 3: Service Hooks vs Polling

**Your team currently polls the Azure DevOps API every 60 seconds to check for build completions (3,600 API calls per hour). What improvement would service hooks provide?**

A) 50% reduction in API calls  
B) 90% reduction in API calls  
C) 98.3% reduction in API calls  
D) No significant reduction (similar call volume)  

<details>
<summary>Click to reveal answer</summary>

**Answer: C) 98.3% reduction in API calls**

**Explanation**: **Service hooks use event-driven architecture** (push model) vs polling (pull model).

**Comparison**:
| Approach | API Calls (1 hour) | Latency | Server Load |
|----------|-------------------|---------|-------------|
| Polling | 60 calls/hour (every 60 sec) | 0-60 seconds | Continuous |
| Service Hooks | 1 call/hour (event-only) | <1 second | Event-only |

**Reduction**: (60 - 1) / 60 = 98.3% reduction in API calls

**Additional benefits**:
- 99% reduction in latency (60 sec → 1 sec)
- 98%+ reduction in server load
- Real-time responsiveness

**References**: Unit 4 - Explore Service Hooks
</details>

---

### Question 4: Azure DevOps Notification Scopes

**You need to configure a notification that alerts ALL members of the "Security Team" when ANY pull request is created in ANY repository across the organization. Which notification scope should you use?**

A) Personal notifications  
B) Team notifications  
C) Project notifications  
D) Global notifications  

<details>
<summary>Click to reveal answer</summary>

**Answer: D) Global notifications**

**Explanation**: Azure DevOps has **4 hierarchical notification scopes**:

1. **Personal notifications**: Individual user subscriptions (user-specific)
2. **Team notifications**: Group-specific alerts (team members only)
3. **Project notifications**: Project-wide announcements (single project scope)
4. **Global notifications**: Organization-level alerts (**across ALL projects**)

**Scenario analysis**:
- Scope: "ANY repository across the organization" → **Organization-level** (not project-specific)
- Recipients: "ALL members of Security Team" → Team-based (not personal)
- Event: "ANY pull request created" → Organization-wide event

**Answer**: Global notifications with team filter (Security Team)

**Configuration path**: Organization Settings → Notifications → Global notifications → New subscription → Filter by team

**References**: Unit 5 - Configure Azure DevOps Notifications
</details>

---

### Question 5: GitHub Notification Management

**Your team watches 50 repositories in GitHub and receives 200+ notifications per day. What is the MOST effective strategy to reduce notification overload while staying informed about critical issues?**

A) Disable all notifications (manual repository checks)  
B) Use selective watching (only critical repositories) + email filters  
C) Switch to weekly digest emails only  
D) Unsubscribe from all repositories  

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Use selective watching (only critical repositories) + email filters**

**Explanation**: **Best practice for notification management** combines:

1. **Selective watching**: Watch 5 critical repositories instead of 50
   - Watching all: 50 repos × 50-100 notifications/day = 2,500-5,000 notifications/day
   - Selective: 5 repos × 50-100 notifications/day = 250-500 notifications/day
   - **Reduction**: 80-90% fewer notifications

2. **Email filters**: Route notifications by priority
   - High priority: Inbox (mentions, assignments, reviews)
   - Medium priority: Folder (PR comments, issue updates)
   - Low priority: Archive/skip (automated notifications)

3. **Participating notifications**: Auto-subscribe only when:
   - Assigned to issue/PR
   - @mentioned in comment
   - You commented (follow-up discussion)

**Wrong answers**:
- A) Disabling all: Misses critical issues (no awareness)
- C) Weekly digest: Too slow for critical issues (7-day delay)
- D) Unsubscribe: Complete loss of awareness

**References**: Unit 6 - Configure GitHub Notifications
</details>

---

### Question 6: Release Process Quality Metrics

**Your release pipeline has the following statistics over the last 30 days: 120 deployments, 115 succeeded, 5 failed, average recovery time 45 minutes. Which metric indicates a potential problem requiring investigation?**

A) Deployment frequency (4 per day)  
B) Success rate (96%)  
C) Mean time to recovery (MTTR: 45 minutes)  
D) All metrics are within acceptable ranges  

<details>
<summary>Click to reveal answer</summary>

**Answer: C) Mean time to recovery (MTTR: 45 minutes)**

**Explanation**: Compare against **target metrics**:

| Metric | Actual | Target | Status |
|--------|--------|--------|--------|
| **Deployment Frequency** | 4/day | Daily+ | ✅ Good (high maturity) |
| **Success Rate** | 96% | >95% | ✅ Good (above target) |
| **MTTR** | 45 min | <15 min | ⚠️ **Problem** (3× target) |
| **Change Failure Rate** | 4.2% | <5% | ✅ Good (below target) |

**MTTR Analysis**:
- Target: <15 minutes (automated rollback)
- Actual: 45 minutes (3× target)
- **Root causes**: Manual intervention, slow rollback process, insufficient automation

**Improvement strategies**:
- Implement automated rollback triggers
- Pre-warm blue-green deployments
- Improve health check response time
- Add automatic traffic shifting (canary → full deployment)

**References**: Unit 7 - Explore How to Measure Quality of Release Process
</details>

---

### Question 7: Release Documentation Storage Strategies

**Your organization requires release notes to be automatically generated from work items (User Stories, Bugs) and stored in a version-controlled wiki that developers can edit collaboratively. Which TWO strategies should you combine?**

A) Document storage systems (SharePoint)  
B) Wiki-based documentation (Azure DevOps Wiki)  
C) Repository-integrated documentation (CHANGELOG.md)  
D) Work item documentation (automated generation)  

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Wiki-based documentation + D) Work item documentation**

**Explanation**: Requirements analysis:

1. **"Automatically generated from work items"** → **Work item documentation** (Strategy D)
   - Query Azure Boards API
   - Aggregate User Stories, Bugs, Tasks
   - Generate release notes from work item data

2. **"Version-controlled wiki"** → **Wiki-based documentation** (Strategy B)
   - Azure DevOps Wiki is Git-backed (version control)
   - Collaborative editing (Markdown)
   - Cross-linking between pages

3. **"Developers can edit collaboratively"** → **Wiki-based** (Strategy B)
   - Lightweight editing (no complex tooling)
   - Real-time collaboration
   - Review and approval workflows

**Implementation**:
```yaml
# Azure Pipeline task
- task: WikiUpdaterTask@1
  inputs:
    repo: '$(System.TeamProject)/_wiki'
    filename: 'Releases/v2.5.0.md'
    sourceFile: '$(Build.ArtifactStagingDirectory)/ReleaseNotes.md'  # ← Generated from work items
```

**Wrong answers**:
- A) SharePoint: Not version-controlled with code, limited collaboration
- C) CHANGELOG.md: Doesn't auto-generate from work items (manual editing required)

**References**: Unit 8 - Examine Release Notes and Documentation
</details>

---

### Question 8: Release Management Tool Selection - Artifacts

**Your application deployment requires aggregating artifacts from 5 different sources: CI build artifacts, database scripts (Git repo), configuration files (Azure Key Vault), third-party libraries (NuGet), and infrastructure templates (separate Terraform repo). Which release management tool capability is MOST critical for this scenario?**

A) Continuous deployment triggers  
B) Multi-source artifact aggregation  
C) Manual approval workflows  
D) Schedule-based deployment triggers  

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Multi-source artifact aggregation**

**Explanation**: **Multi-source artifact aggregation** is the ability to pull artifacts from multiple different sources in a single release pipeline.

**Scenario requirements**:
```
Application Deployment:
├── Source 1: Application binaries (from CI build) → Build artifact
├── Source 2: Database scripts (from Git repository) → Git checkout
├── Source 3: Configuration files (from Azure Key Vault) → AzureKeyVault task
├── Source 4: Third-party libraries (from NuGet) → NuGet restore
└── Source 5: Infrastructure templates (Terraform from separate repo) → Git checkout (separate repo)
```

**Tool requirement**: Release pipeline must support:
- Consuming build artifacts (pipeline artifacts)
- Checking out multiple Git repositories
- Retrieving secrets from Azure Key Vault
- Downloading NuGet packages
- All in **single orchestrated deployment**

**Tools with strong multi-source support**:
- ✅ Azure Pipelines: Native multi-source support (resources section)
- ✅ GitLab: Multiple repositories, artifacts, external sources
- ⚠️ GitHub Actions: Possible but requires more manual scripting
- ⚠️ Jenkins: Plugin-dependent (requires configuration)

**Wrong answers**:
- A) Triggers: Doesn't address multi-source artifact challenge
- C) Approvals: Doesn't address artifact aggregation
- D) Scheduling: Doesn't address artifact aggregation

**References**: Unit 9 - Examine Considerations for Choosing Release Management Tools
</details>

---

### Question 9: Four-Eyes Principle in Release Management

**Your organization requires that NO single person can both develop code AND deploy it to production (four-eyes principle for compliance). How should you implement this in Azure DevOps?**

A) Create separate pipelines for build and release (different owners)  
B) Combine branch policies (minimum 2 reviewers) + environment approvals (different approvers than developers)  
C) Use manual approval gates only (any team member can approve)  
D) Disable pull requests and require direct commits with code review  

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Combine branch policies (minimum 2 reviewers) + environment approvals (different approvers than developers)**

**Explanation**: **Four-eyes principle** (separation of duties) requires:
1. Developer cannot merge own code
2. Different person approves deployment to production

**Implementation**:

**Layer 1: Branch Policies** (prevents self-approval of code)
```
Branch Policy (main branch):
├── Minimum 2 reviewers required
├── Requestor cannot approve own changes ← Prevents self-merge
├── Build validation required
└── No force push allowed
```

**Layer 2: Environment Approvals** (prevents developer-deployed production)
```
Environment Policy (Production):
├── Approvers: security-team, operations-team
├── Restrict: developers group CANNOT approve ← Prevents developer deployment
├── Timeout: 24 hours
└── Required approvals: 2 of 3 approvers
```

**Compliance verification**:
- ✅ Developer A writes code → Reviewer B approves → Automated merge
- ✅ Automated build + deploy to Dev/Test
- ✅ Production deployment → Security team approves (NOT Developer A or B)
- ❌ Developer A cannot approve own PR (branch policy blocks)
- ❌ Developer A cannot approve production deployment (environment policy blocks)

**Wrong answers**:
- A) Separate pipelines: Doesn't prevent developer from triggering release pipeline
- C) Any team member: Allows developers to approve production (violates principle)
- D) Disable PRs: Eliminates code review entirely (worse compliance)

**References**: Unit 9 - Examine Considerations for Choosing Release Management Tools (Traceability & Security)
</details>

---

### Question 10: Release Management Tool Selection - Decision Tree

**Your organization uses GitHub exclusively for source control, deploys containerized applications to Kubernetes, and has a small team (5 developers) with limited budget. Which release management tool is the BEST fit?**

A) Azure Pipelines (enterprise features)  
B) Jenkins (maximum flexibility)  
C) GitHub Actions (native GitHub integration)  
D) Bamboo (Atlassian ecosystem)  

<details>
<summary>Click to reveal answer</summary>

**Answer: C) GitHub Actions (native GitHub integration)**

**Explanation**: **Decision criteria analysis**:

| Requirement | GitHub Actions | Azure Pipelines | Jenkins | Bamboo |
|-------------|---------------|-----------------|---------|--------|
| **GitHub-exclusive** | ✅ Native | ⚠️ Supported | ⚠️ Plugin | ❌ Limited |
| **Container/K8s** | ✅ Excellent | ✅ Excellent | ✅ Plugin | ⚠️ Tasks |
| **Small team (5)** | ✅ Easy setup | ⚠️ Learning curve | ❌ Complex | ⚠️ Moderate |
| **Limited budget** | ✅ Free tier generous | ✅ Free tier good | ⚠️ Self-hosted costs | ❌ Commercial license |

**Scoring**:
- **GitHub Actions**: 4 / 4 requirements met (best fit)
- Azure Pipelines: 3 / 4 (good fit, but overkill for small team)
- Jenkins: 2 / 4 (too complex for small team)
- Bamboo: 1 / 4 (expensive, requires Atlassian ecosystem)

**GitHub Actions advantages for this scenario**:
- ✅ **Zero setup**: YAML file in `.github/workflows/`, no infrastructure
- ✅ **Native GitHub**: Tight PR integration, code scanning, packages
- ✅ **Container-first**: Built-in Docker support, Kubernetes deployment actions
- ✅ **Free tier**: 2,000 minutes/month for private repos, unlimited for public
- ✅ **Marketplace**: 15,000+ actions (Kubernetes deploy, Helm, kubectl)

**Sample workflow**:
```yaml
name: Deploy to Kubernetes
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: azure/setup-kubectl@v3
      - run: |
          kubectl apply -f k8s/deployment.yaml
```

**Wrong answers**:
- A) Azure Pipelines: Enterprise features overkill for 5-person team
- B) Jenkins: Too complex, requires DevOps engineer for maintenance
- D) Bamboo: Expensive license, requires Atlassian ecosystem (not mentioned)

**References**: Unit 10 - Explore Common Release Management Tools
</details>

---

## Scoring Guide

**10 questions × 10 points each = 100 points total**

- **90-100 points (9-10 correct)**: ✅ **Excellent** - Strong understanding of automated health inspection, notifications, service hooks, and release management tool selection
- **70-89 points (7-8 correct)**: ✅ **Good** - Solid grasp of core concepts, review specific topics with lower scores
- **50-69 points (5-6 correct)**: ⚠️ **Fair** - Basic understanding, recommend reviewing Units 2-10 and additional resources
- **Below 50 points (<5 correct)**: ❌ **Needs Improvement** - Review all module content, focus on hands-on practice

---

## Key Concepts Covered

### Unit 2: Automate Inspection of Health
- ✅ 6 automated monitoring capabilities
- ✅ Release gates, events/notifications, service hooks, reporting
- ✅ Real-world impact (40 min → 1 min per failure)

### Unit 3: Events and Notifications
- ✅ Alert fatigue problem (80% ignored when over-configured)
- ✅ Strategic filtering (relevance, criticality, environment)
- ✅ Delivery channels (Email, Messaging, Mobile, Webhooks)

### Unit 4: Service Hooks
- ✅ Polling vs event-driven (98.3% API reduction)
- ✅ External service integrations (40+ services)
- ✅ Webhook payload examples

### Unit 5: Azure DevOps Notifications
- ✅ 4 hierarchical scopes (personal, team, project, global)
- ✅ Event-specific triggers (builds, PRs, work items)
- ✅ Configuration best practices

### Unit 6: GitHub Notifications
- ✅ Selective watching strategy (5 critical vs 50 repos)
- ✅ Email filters and routing
- ✅ Participating notifications

### Unit 7: Quality Measurement
- ✅ Key metrics (frequency, success rate, MTTR, failure rate)
- ✅ Degradation indicators (frequent changes, persistent failures)
- ✅ Quality gates (infrastructure, requirements, security)

### Unit 8: Release Notes Documentation
- ✅ 4 storage strategies (document, wiki, repository, work items)
- ✅ Automated generation (Generate Release Notes Task)
- ✅ Functional vs technical documentation

### Unit 9: Tool Selection Considerations
- ✅ 6 critical capabilities (artifacts, triggers, approvals, stages, tasks, traceability)
- ✅ Multi-source artifact aggregation
- ✅ Four-eyes principle (separation of duties)

### Unit 10: Common Release Management Tools
- ✅ GitHub Actions (GitHub-native, 15,000+ actions)
- ✅ Azure Pipelines (enterprise, multi-cloud)
- ✅ Jenkins (1,800+ plugins, flexibility)
- ✅ CircleCI (Docker-first, fast)
- ✅ GitLab Pipelines (all-in-one DevOps)
- ✅ Bamboo (Atlassian ecosystem)

---

## Next Steps

✅ **Completed**: Module 5 knowledge check

**Continue to**: Unit 12 - Summary (module recap, key takeaways, next learning paths)

---

## Additional Resources

- [Azure DevOps Notifications Documentation](https://learn.microsoft.com/en-us/azure/devops/notifications/)
- [GitHub Notifications Documentation](https://docs.github.com/en/account-and-profile/managing-subscriptions-and-notifications-on-github)
- [Service Hooks Integration Guide](https://learn.microsoft.com/en-us/azure/devops/service-hooks/)
- [DORA Metrics (DevOps Research and Assessment)](https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance)
- [Release Management Tools Comparison](https://learn.microsoft.com/en-us/devops/deliver/what-is-release-management)

[↩️ Back to Module Overview](01-introduction.md) | [⬅️ Previous: Common Tools](10-explore-common-release-management-tools.md) | [➡️ Next: Summary](12-summary.md)
