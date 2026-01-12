# Exercise: Azure Repos Collaborating with Pull Requests

## Scenario: Enterprise Development Team Setup

You're the lead DevOps engineer for a **financial services company** implementing a new trading platform. The development involves multiple teams with varying responsibilities:

| Team | Responsibility | Review Focus |
|------|---------------|--------------|
| **Core Development Team** | Primary application development and feature implementation | Code quality, functionality, maintainability |
| **Security Team** | Security review and compliance validation | Authentication, authorization, data protection, vulnerabilities |
| **Architecture Team** | Architectural consistency and technical debt management | Design patterns, scalability, performance |
| **QA Team** | Quality assurance and testing coordination | Test coverage, edge cases, defect prevention |

**Mission**: Establish a pull request workflow that ensures security compliance, maintains architectural integrity, and supports rapid development velocity while providing comprehensive audit trails required for financial industry regulations.

**Estimated Time**: 18 minutes

## Exercise Steps

### Step 1: Access Branch Policies Configuration

**Objective**: Navigate to branch policy configuration in Azure DevOps

**Instructions**:
1. Navigate to your Azure DevOps project
2. Access the **Repos** section from left navigation
3. Select your target repository
4. Open the **Branches** view
5. Locate the **main** branch (or your primary integration branch)
6. Select **Branch policies** from the context menu (three dots)

**Expected Outcome**: Branch policies configuration page opens showing available policy options

**Screenshot Reference**: Branch policies menu in Azure DevOps Branches view

---

### Step 2: Configure Reviewer Requirements

**Objective**: Set up minimum reviewer requirements with enterprise considerations

**Strategic Configuration Settings**:

| Setting | Recommended Value | Rationale |
|---------|------------------|-----------|
| **Minimum number of reviewers** | 2 | Comprehensive coverage, knowledge distribution |
| **Allow requestors to approve their own changes** | ❌ Disabled | Separation of duties, SOX compliance |
| **Allow completion even if some reviewers vote to wait or reject** | ✅ Enabled (with governance) | Emergency scenarios, executive override |
| **When new changes are pushed** | Reset all approval votes | Security: ensures reviewers see final code |

**Configuration Steps**:
1. Check **Require a minimum number of reviewers**
2. Set **Minimum number of reviewers** to `2`
3. **Uncheck** "Allow requestors to approve their own changes"
4. **Check** "Reset all approval votes when new changes are pushed"
5. Consider enabling "Allow completion even if some reviewers vote to wait" for emergency scenarios

**Enterprise Considerations**:
- **Financial services**: SOX compliance requires separation of duties (no self-approval)
- **Security**: Reset votes on new changes ensures reviewers validate final code
- **Emergency override**: Allow completion with governance for critical production fixes

**Expected Outcome**: Minimum 2 reviewers required, self-approval blocked, votes reset on new pushes

---

### Step 3: Set Up Automatic Reviewer Assignment

**Objective**: Implement intelligent reviewer routing based on expertise and code areas

**Enterprise Reviewer Strategy**:

**Configure automatic reviewers for**:

| Code Area | Reviewers | Policy | Justification |
|-----------|-----------|--------|---------------|
| **Security code** (`/src/auth/**`, `/src/crypto/**`) | @security-team | Required | Authentication, authorization, data handling validation |
| **Infrastructure** (`/infrastructure/**`) | @architecture-team, @sre-team | Required | Framework, design patterns, scalability |
| **API endpoints** (`/src/api/**`) | @backend-team, @api-specialists | Optional | Domain expertise, API design |
| **Database migrations** (`/migrations/**`) | @database-team | Required | Schema changes, data integrity |
| **All changes** | @senior-developer (rotation) | Optional | Cross-training, knowledge transfer |

**Configuration Steps**:
1. Click **Add automatic reviewers**
2. For each code area pattern:
   - **Path filter**: `/src/auth/**` (use wildcards)
   - **Reviewers**: Select security-team
   - **Policy**: Select "Required" for critical areas
   - **Message**: "Security review required for authentication code"
3. Repeat for other critical code areas

**CODEOWNERS Integration**:
Create `.azuredevops/CODEOWNERS` file in repository root:
```
# Default owners
* @devops-team

# Security-sensitive code
/src/auth/** @security-team
/src/crypto/** @security-team @security-architects

# Infrastructure
/infrastructure/** @platform-team @sre-team

# Database
/migrations/** @database-team @backend-architects
```

**Expected Outcome**: Reviewers automatically assigned based on files modified in PR

---

### Step 4: Enable Quality Assurance Policies

**Objective**: Establish comprehensive quality gates through comment resolution requirements

**Policy Configuration**:
1. Check **Require comment resolution**
2. This ensures all review feedback is addressed before merge

**Quality Benefits**:

| Benefit | Impact | Example |
|---------|--------|---------|
| **Accountability** | All feedback must be discussed/resolved | Security concern can't be ignored |
| **Prevents oversight** | Forces explicit decision on each comment | "Won't fix" requires justification |
| **Audit trail** | Complete record of decisions | Regulatory compliance documentation |
| **Knowledge transfer** | Discussion captured for future reference | Junior devs learn from senior feedback |

**Comment Resolution Workflow**:
```
1. Reviewer leaves comment: "This SQL query is vulnerable to injection"
2. Author responds: "Fixed by using parameterized queries in commit abc123"
3. Reviewer validates fix
4. Reviewer marks comment as "Resolved"
5. PR can now be merged (all comments resolved)
```

**Expected Outcome**: PRs cannot be completed until all comments are marked as resolved

---

### Step 5: Configure Traceability and Compliance

**Objective**: Configure work item linking for regulatory compliance and project tracking

**Policy Configuration**:
1. Check **Check for linked work items**
2. Select **Required** (not optional)

**Compliance Benefits**:

| Benefit | Regulatory Value | Business Value |
|---------|-----------------|----------------|
| **Full audit trail** | From requirement to deployment | Complete change documentation |
| **Regulatory compliance** | SOX, HIPAA, PCI-DSS documentation | Audit readiness |
| **Impact analysis** | Which features affected by change | Risk assessment |
| **Project tracking** | Progress visibility | Management reporting |

**Work Item Linking Methods**:

**Method 1: Branch Creation from Work Item**:
```
1. Open work item (e.g., User Story #12345)
2. Click "Create a branch" link
3. Branch automatically named: feature/12345-implement-oauth
4. PR created from this branch auto-links to work item
```

**Method 2: Commit Message Patterns**:
```bash
# Automatic linking via commit messages
git commit -m "feat: implement OAuth 2.0 (#12345)"
git commit -m "fix: resolve token expiry (Fixes AB#12345)"
git commit -m "refactor: improve auth flow (Related #12345)"
```

**Method 3: PR Description**:
```markdown
## Related Work Items
- Resolves AB#12345
- Related to AB#12346
```

**Expected Outcome**: PRs require linked work item before merge approval

---

### Step 6: Create Feature Branches with Traceability

**Objective**: Create feature branches directly from work items to establish automatic linkage

**Enterprise Branch Naming Strategy**:

| Pattern | Example | Use Case |
|---------|---------|----------|
| `feature/[id]-[description]` | `feature/12345-oauth-authentication` | New features |
| `hotfix/[id]-[description]` | `hotfix/12346-token-refresh-bug` | Production fixes |
| `release/[version]` | `release/2.5.0` | Release preparation |
| `bugfix/[id]-[description]` | `bugfix/12347-validation-error` | Non-critical fixes |

**Creation Steps**:
1. Navigate to Azure Boards → Work Items
2. Open target work item (e.g., User Story #12345 "Implement OAuth 2.0")
3. In work item details, click **Create a branch**
4. Configure branch:
   - **Name**: `feature/12345-implement-oauth-authentication`
   - **Base on**: `main`
   - **Link work item**: ✅ Checked (automatic)
5. Click **Create branch**

**Automation Benefits**:
- Automatic work item linking (no manual step required)
- Consistent naming supports analytics and automation
- Work item status automatically updates to "In Progress"

**Expected Outcome**: Feature branch created with automatic work item linkage

---

### Step 7: Implement Changes Using Best Practices

**Objective**: Implement changes using enterprise development practices

**Enterprise Commit Standards**:

**Commit Message Format** (Conventional Commits):
```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Examples**:
```bash
# Feature implementation
git commit -m "feat(auth): implement OAuth 2.0 authentication

- Add OAuth provider configuration
- Implement token exchange flow
- Add refresh token logic
- Add unit tests for auth service

Resolves AB#12345"

# Security fix
git commit -m "fix(security): prevent SQL injection in user query

- Replace string concatenation with parameterized queries
- Add input validation for user ID parameter
- Add security test cases

Security Advisory: CVE-2024-XXXX
Fixes AB#12346"

# Documentation update
git commit -m "docs: add OAuth configuration guide

- Document environment variables
- Add setup instructions
- Include troubleshooting section"
```

**Commit Best Practices**:

| Practice | Guideline | Rationale |
|----------|-----------|-----------|
| **Atomic commits** | One logical change per commit | Easier review, rollback, cherry-pick |
| **Descriptive messages** | Clear description of what and why | Audit trail, future understanding |
| **Link to work items** | Include work item IDs | Traceability, compliance |
| **Security-conscious** | No secrets, sensitive data in commits | Security, compliance |
| **Include tests** | Tests in same commit as code | Ensures functionality verified |

**Expected Outcome**: Commits follow enterprise standards with clear messages and work item links

---

### Step 8: Create Comprehensive Pull Requests

**Objective**: Create pull requests that facilitate comprehensive review and collaboration

**Enterprise Pull Request Template**:

Create `.azuredevops/pull_request_template.md`:
```markdown
## Summary
<!-- Brief description of changes (2-3 sentences) -->

## Related Work Items
- Resolves AB#XXXXX

## Changes Made
<!-- List of specific changes -->
- [ ] Change 1
- [ ] Change 2
- [ ] Change 3

## Testing Performed
<!-- Describe testing approach -->
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed
- [ ] Security testing (if applicable)

## Deployment Considerations
<!-- Any special deployment steps, migrations, configuration changes -->

## Screenshots/Videos
<!-- If UI changes, include visual evidence -->

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review performed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings or errors
- [ ] Tests pass locally
- [ ] Security considerations addressed

## Additional Context
<!-- Any other context reviewers should know -->
```

**Enterprise Pull Request Best Practices**:

| Element | Best Practice | Example |
|---------|--------------|---------|
| **Title** | `[WI #12345] feat: implement OAuth 2.0 authentication` | Clear, includes work item ID |
| **Description** | Comprehensive context (business + technical) | "Implements OAuth to meet security requirement..." |
| **Stakeholder Mentions** | @mention relevant teams/experts | "@security-team please review auth flow" |
| **Work Item Integration** | Link related work items, tests, dependencies | "Depends on #12344, Related to #12346" |
| **Visual Evidence** | Screenshots for UI changes, diagrams for architecture | Before/after screenshots, sequence diagrams |

**Creation Steps**:
1. Push feature branch to Azure Repos:
   ```bash
   git push origin feature/12345-implement-oauth-authentication
   ```
2. Navigate to Azure Repos → Pull Requests
3. Click **New pull request**
4. Configure:
   - **Source branch**: `feature/12345-implement-oauth-authentication`
   - **Target branch**: `main`
   - **Title**: `[AB#12345] feat: implement OAuth 2.0 authentication`
5. Fill out PR template with comprehensive information
6. Add reviewers (or rely on automatic assignment)
7. Click **Create**

**Expected Outcome**: PR created with rich context, automatic reviewer assignment, policy validation

---

### Step 9: Coordinate Multi-Stakeholder Reviews

**Objective**: Coordinate sophisticated review processes across multiple stakeholders

**Multi-Stakeholder Review Strategy**:

| Stakeholder | Review Focus | Timeline | Priority |
|-------------|-------------|----------|----------|
| **Security Team** | Authentication, authorization, data protection, vulnerabilities | < 4 hours | High |
| **Architecture Team** | Design patterns, technical debt, scalability, performance | < 8 hours | High |
| **Code Quality Team** | Maintainability, testing, documentation, standards | < 8 hours | Medium |
| **Domain Experts** | Business logic, edge cases, user experience | < 24 hours | Medium |

**Review Coordination Steps**:
1. **Author self-review**: Before requesting reviews, author performs thorough self-review
2. **Stakeholder notification**: Azure DevOps automatically notifies assigned reviewers
3. **Parallel reviews**: Multiple reviewers can review simultaneously
4. **Comment threads**: Each concern gets separate thread for focused discussion
5. **Resolution workflow**: Author addresses feedback, marks comments resolved
6. **Follow-up reviews**: Reviewers verify fixes, approve or request changes

**Example Review Timeline**:
```
Hour 0: PR created, automatic reviewers assigned
Hour 1: Security team reviews, leaves 3 comments (2 critical, 1 suggestion)
Hour 2: Author addresses critical security issues, pushes commit
Hour 2: Approval votes reset automatically (policy)
Hour 4: Security team re-reviews, approves
Hour 6: Architecture team reviews, leaves 2 comments (architectural suggestions)
Hour 8: Author discusses architectural approach, agrees to refactor
Hour 10: Author pushes refactored code
Hour 12: Architecture team approves
Hour 13: All comments resolved, build passes, PR merged
```

**Expected Outcome**: Coordinated reviews across teams with clear feedback and resolution

---

### Step 10: Facilitate Effective Review Discussions

**Objective**: Facilitate thorough review discussions that enhance code quality and knowledge transfer

**Review Facilitation Techniques**:

**Structured Feedback Categories**:
```
🚨 Critical (Blocking): Security vulnerability, data corruption risk
⚠️ High (Required): Performance issue, architectural violation
💡 Medium (Recommended): Code clarity improvement, missing test
✨ Low (Suggestion): Stylistic preference, minor optimization
❓ Question: Clarification needed, understanding check
📚 Learning: Educational content, "here's why..."
```

**Example Review Comments**:

**Poor Review Comment**:
> "This is wrong."

**Excellent Review Comment**:
> "🚨 Critical: This SQL query is vulnerable to SQL injection.
> 
> **Issue**: String concatenation allows malicious input to modify query structure.
> 
> **Current Code**:
> ```sql
> const query = `SELECT * FROM users WHERE id = '${userId}'`;
> ```
> 
> **Recommended Fix**:
> ```sql
> const query = 'SELECT * FROM users WHERE id = ?';
> db.execute(query, [userId]);
> ```
> 
> **Why**: Parameterized queries prevent injection by treating input as data, not executable code. This is a requirement for PCI-DSS compliance.
> 
> **References**: 
> - OWASP SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection
> - Company Security Standard: SEC-001 section 3.2
> 
> **Testing**: Add test case verifying injection attempts fail."

**Solution-Oriented Approach**:
- ✅ Identify problem clearly
- ✅ Explain reasoning (educational)
- ✅ Provide specific solution with code example
- ✅ Include references for learning
- ✅ Suggest testing approach

**Timeline Management**:

| Review Type | Response Time | Escalation |
|------------|---------------|------------|
| **Critical security issue** | < 1 hour | Immediate Slack notification |
| **Blocking comment** | < 4 hours | Daily standup mention |
| **Standard feedback** | < 24 hours | Weekly review metrics |
| **Suggestion** | Best effort | No escalation |

**Expected Outcome**: High-quality review discussions that educate and improve code

---

### Step 11: Leverage Azure DevOps Integration

**Objective**: Leverage Azure DevOps integration for comprehensive validation

**Pipeline Integration Benefits**:

| Integration | Capability | Business Value |
|-------------|-----------|----------------|
| **Automated Build Validation** | Compile verification, basic functionality | Prevent broken code from merging |
| **Test Suite Execution** | Unit, integration, E2E tests | Comprehensive quality validation |
| **Security Scanning** | SAST, dependency vulnerabilities, secrets | Automated security compliance |
| **Performance Testing** | Load tests, benchmarks | Prevent performance regressions |
| **Deployment Validation** | Deploy to staging, smoke tests | Production readiness verification |

**Azure Pipeline Configuration** (`azure-pipelines-pr.yml`):
```yaml
trigger: none  # Disable CI trigger

pr:
  branches:
    include:
      - main
      - release/*

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: FastFeedback
  displayName: 'Fast Feedback (< 5 min)'
  jobs:
  - job: Build
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
    
    - script: npm ci
      displayName: 'Install dependencies'
    
    - script: npm run build
      displayName: 'Build application'
    
    - script: npm run lint
      displayName: 'Lint code'
    
    - script: npm run test:unit
      displayName: 'Run unit tests'

- stage: ComprehensiveValidation
  displayName: 'Comprehensive Validation (15 min)'
  dependsOn: FastFeedback
  condition: succeeded()
  jobs:
  - job: IntegrationTests
    steps:
    - script: npm run test:integration
      displayName: 'Integration tests'
  
  - job: SecurityScan
    steps:
    - task: CredScan@3
      displayName: 'Scan for credentials'
    
    - task: Semmle@1
      inputs:
        sourceCodeDirectory: '$(Build.SourcesDirectory)'
      displayName: 'CodeQL security analysis'
    
    - task: dependency-check@6
      displayName: 'Dependency vulnerability scan'

- stage: StagingDeployment
  displayName: 'Deploy to Staging'
  dependsOn: ComprehensiveValidation
  condition: and(succeeded(), eq(variables['System.PullRequest.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployStaging
    environment: 'staging-pr-$(System.PullRequest.PullRequestId)'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'staging-subscription'
              appName: 'trading-platform-pr-$(System.PullRequest.PullRequestId)'
              package: '$(Pipeline.Workspace)/drop/*.zip'
          
          - script: npm run test:smoke -- --url https://trading-platform-pr-$(System.PullRequest.PullRequestId).azurewebsites.net
            displayName: 'Smoke tests on staging'
```

**Status Check Configuration**:
1. In Branch Policies, enable **Build validation**
2. Select pipeline: `azure-pipelines-pr.yml`
3. Set **Policy requirement**: Required
4. Set **Build expiration**: Immediately when main is updated
5. Set **Display name**: "PR Validation Pipeline"

**Expected Outcome**: PRs blocked until pipeline succeeds, providing automated quality gates

---

## Advanced Features and Management

### Branch Recovery and Management

Azure DevOps provides sophisticated branch management capabilities for enterprise environments:

**Deleted Branch Recovery**:
1. Navigate to Repos → Branches
2. Click **All** tab
3. Change filter to show "Deleted" branches
4. Search for deleted branch by exact name
5. View deletion history (who deleted, when)
6. Click **Restore** to recover branch with complete commit history
7. Re-establish policies and permissions as needed

**Enterprise Branch Management Best Practices**:

| Practice | Implementation | Benefit |
|----------|----------------|---------|
| **Consistent naming conventions** | Enforce via policy or automation | Searchability, analytics, automation |
| **Folder organization** | Use `/` delimiters: `feature/`, `release/` | Complex repository structure management |
| **Retention policies** | Auto-delete merged branches after 30 days | Reduce clutter, maintain hygiene |
| **Branch proliferation monitoring** | Dashboard tracking active branch count | Identify cleanup opportunities |

**Automation Script for Branch Cleanup**:
```bash
#!/bin/bash
# cleanup-merged-branches.sh

# Get all merged branches older than 30 days
az repos ref list \
  --organization https://dev.azure.com/yourorg \
  --project YourProject \
  --repository YourRepo \
  --filter heads/ \
  --query "[?objectId != '$(git rev-parse refs/remotes/origin/main)'].name" \
  -o tsv | while read branch; do
    
    # Check if branch is merged and old
    last_commit_date=$(git log -1 --format=%ct "$branch")
    current_date=$(date +%s)
    days_old=$(( (current_date - last_commit_date) / 86400 ))
    
    if [ $days_old -gt 30 ]; then
      echo "Deleting old merged branch: $branch (${days_old} days old)"
      az repos ref delete --name "$branch" --object-id "$(git rev-parse "$branch")"
    fi
done
```

---

## Measuring Success and Optimization

### Key Performance Indicators

**Velocity Metrics**:

| Metric | Target | Calculation | Dashboard |
|--------|--------|-------------|-----------|
| **PR cycle time** | < 24 hours | Merge time - creation time | Azure DevOps Analytics |
| **Time to first review** | < 4 hours | First review - creation time | Custom query |
| **Review participation rate** | > 80% | Reviewers / Total team * 100 | Team velocity report |
| **Policy compliance rate** | 100% | PRs with policies met / Total PRs | Branch policy report |

**Quality Metrics**:

| Metric | Target | Business Impact |
|--------|--------|-----------------|
| **Defects caught in review** | > 70% | Reduced production incidents |
| **Post-merge issue rate** | < 5% | Code quality, customer satisfaction |
| **Knowledge transfer score** | > 7/10 | Team capability growth |
| **Test coverage** | > 80% | Confidence in changes |

**Compliance Metrics**:

| Metric | Target | Regulatory Value |
|--------|--------|------------------|
| **Audit trail completeness** | 100% | SOX, HIPAA compliance |
| **Work item linkage** | 100% | Traceability, impact analysis |
| **Security review coverage** | 100% for auth/crypto | Risk mitigation |

**Azure DevOps Analytics Query**:
```kusto
// PR cycle time analysis
PullRequests
| where TargetBranch == "main"
| where ClosedDate >= ago(30d)
| extend CycleTimeHours = (ClosedDate - CreatedDate) / 1h
| summarize 
    AvgCycleTime = avg(CycleTimeHours),
    MedianCycleTime = percentile(CycleTimeHours, 50),
    P90CycleTime = percentile(CycleTimeHours, 90),
    Count = count()
  by Team
| order by AvgCycleTime desc
```

### Continuous Improvement

**Regular Assessment Framework**:

**Monthly Retrospectives**:
1. Review metrics dashboard (cycle time, quality, compliance)
2. Identify top 3 bottlenecks (e.g., "Security reviews take 8+ hours")
3. Brainstorm solutions (e.g., "Add security checklist to PR template")
4. Assign action items with owners and deadlines
5. Track improvements month-over-month

**Quarterly Review**:
- **Team feedback**: Anonymous survey on process satisfaction (1-10 scale)
- **Policy effectiveness**: Which policies add value vs. create friction?
- **Automation opportunities**: What manual steps can be automated?
- **Training needs**: What knowledge gaps cause review delays?

**Optimization Cycle**:
```
Measure → Analyze → Identify → Experiment → Validate → Standardize
   ↑                                                          ↓
   └────────────────────── Continuous Loop ──────────────────┘
```

**Evolution Based on Maturity**:

| Team Maturity | Policy Level | Review Focus |
|--------------|--------------|--------------|
| **Beginning** (0-6 months) | Minimal: 1 reviewer, build validation | Learning, knowledge transfer |
| **Developing** (6-12 months) | Moderate: 2 reviewers, work item linking | Quality, best practices |
| **Advanced** (12-24 months) | Comprehensive: Auto-assignment, security scanning | Architecture, optimization |
| **Expert** (24+ months) | Optimized: Context-aware policies, advanced automation | Innovation, mentorship |

---

## Exercise Results

This comprehensive exercise demonstrates how Azure Repos pull request capabilities support enterprise development requirements:

### Achievements

| Capability | Implementation | Enterprise Value |
|------------|---------------|------------------|
| **Scalable Governance** | Automated policy enforcement | Development velocity maintained with quality |
| **Comprehensive Compliance** | Full audit trails, work item linking | Regulatory adherence (SOX, HIPAA, PCI-DSS) |
| **Quality Amplification** | Systematic review processes | Defect reduction, knowledge transfer |
| **Knowledge Distribution** | Multi-stakeholder reviews, educational feedback | Team capability growth, reduced silos |
| **Security Integration** | Embedded security practices | Risk mitigation, security culture |

### Foundation for Success

The implemented workflow provides the foundation for **sustainable, scalable enterprise development** that balances governance requirements with development agility.

**Next Steps**:
1. Apply these practices to your team's repositories
2. Start with minimal policies, add incrementally
3. Measure effectiveness with Azure DevOps Analytics
4. Iterate based on team feedback and metrics
5. Scale across organization as maturity grows

## Critical Notes
- 🎯 **Scenario**: Financial services trading platform with multi-team collaboration
- 💡 **Key policies**: 2 reviewers, automatic assignment, comment resolution, work item linking
- ⚠️ **Branch naming**: Use folder prefixes (`feature/`, `hotfix/`) and work item IDs for traceability
- 📊 **Metrics**: Track cycle time (< 24h), review participation (> 80%), compliance (100%)
- 🔄 **Continuous improvement**: Monthly retrospectives, quarterly reviews, policy evolution
- ✨ **Integration**: Azure Pipelines for automated validation, Azure Boards for traceability

[Learn More](https://learn.microsoft.com/en-us/training/modules/collaborate-pull-requests-azure-repos/3-exercise-collaborate-pull-requests)
