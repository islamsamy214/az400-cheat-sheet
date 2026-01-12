# Module Assessment - Knowledge Check

## Purpose
Test your understanding of collaborative pull request workflows in Azure Repos for enterprise environments.

## Assessment Format
- **Duration**: 5 minutes
- **Type**: Multiple-choice questions
- **Coverage**: Pull request workflows, branch policies, review practices, Azure DevOps integration
- **Passing**: Understanding verification (not graded)

## Key Topics Covered

### Enterprise Pull Request Architecture
**Assessment Areas**:
- Pull requests as infrastructure for collaborative development
- Azure Repos advantages: security, scalability, DevOps integration
- Multi-stakeholder review workflows
- Knowledge distribution and quality multiplier effects

### Branch Policies and Governance
**Assessment Areas**:
- Review governance policies (minimum reviewers, required reviewers)
- Quality assurance policies (build validation, test coverage, security scanning)
- Process compliance policies (work item linking, comment resolution, merge strategies)
- Integration validation policies (status checks, deployment validation)

### Advanced Collaboration Patterns
**Assessment Areas**:
- Strategic code review excellence (4 pillars: architecture, knowledge, quality, security)
- Effective review feedback framework (constructive specificity, educational focus)
- Priority classification (critical, high, medium, low)
- Adaptive reviewer assignment (code area expertise, team availability, skill development)

### Azure DevOps Integration
**Assessment Areas**:
- Work item integration (automatic linkage, context enrichment, progress tracking)
- Pipeline integration strategies (automated validation, progressive validation, deployment readiness)
- Measuring pull request effectiveness (cycle time, review quality, compliance tracking)

## Self-Assessment Checklist

Before taking the knowledge check, verify you can:

### Branch Policy Configuration
- [ ] **Configure minimum reviewers** - Set reviewer requirements with separation of duties
- [ ] **Set up automatic reviewer assignment** - Based on code areas using CODEOWNERS patterns
- [ ] **Enable comment resolution** - Require all review feedback addressed before merge
- [ ] **Require work item linking** - Ensure traceability for compliance and audit trails
- [ ] **Configure build validation** - Integrate Azure Pipelines for automated quality gates

### Pull Request Best Practices
- [ ] **Create comprehensive PRs** - Include description, context, testing details, checklists
- [ ] **Use PR templates** - Standardize information collection across team
- [ ] **Link work items properly** - Via branch naming, commit messages, or PR description
- [ ] **Provide effective feedback** - Constructive, specific, educational, solution-oriented
- [ ] **Manage review discussions** - Categorize by priority, facilitate resolution, track decisions

### Enterprise Collaboration Patterns
- [ ] **Understand 4 pillars of code review** - Architecture, knowledge, quality, security
- [ ] **Implement multi-stakeholder workflows** - Coordinate security, architecture, QA teams
- [ ] **Configure contextual policies** - Adapt requirements based on change scope and urgency
- [ ] **Leverage automatic assignment** - Route reviews to domain experts based on file paths

### Azure DevOps Integration
- [ ] **Integrate Azure Boards** - Automatic work item linking and status updates
- [ ] **Configure Azure Pipelines PR validation** - Build, test, security scanning triggers
- [ ] **Implement progressive validation** - Staged testing (fast feedback → comprehensive → staging)
- [ ] **Measure effectiveness** - Track cycle time, review quality, compliance metrics
- [ ] **Use Azure DevOps Analytics** - Query PR metrics for continuous improvement

## Exam Preparation Tips

### Core Concepts to Master

**Minimum Reviewer Policy**:
```
Configuration:
- Minimum number of reviewers: 2 (enterprise standard)
- Allow requestors to approve own changes: ❌ Disabled (separation of duties)
- Reset votes on new push: ✅ Enabled (security best practice)
- Allow completion with rejections: Governance decision (emergency override)

Rationale:
- 2 reviewers provide coverage and knowledge distribution
- Self-approval violates SOX compliance requirements
- Reset votes ensure reviewers see final code
```

**Automatic Reviewer Assignment**:
```
CODEOWNERS file patterns:
/src/auth/** @security-team (required)
/infrastructure/** @platform-team @sre-team (required)
/src/api/** @backend-team (optional for knowledge transfer)

Benefits:
- Domain experts review relevant changes automatically
- Workload distributed across team members
- Junior developers included for learning opportunities
```

**Four Pillars of Strategic Code Review**:

| Pillar | Focus | Example Review Comment |
|--------|-------|----------------------|
| **Architectural Consistency** | Design patterns, standards | "This violates SOLID principles - consider dependency injection pattern" |
| **Knowledge Amplification** | Learning opportunities | "Here's why we use async/await: prevents event loop blocking..." |
| **Quality Systematization** | Maintainability, testing | "Add unit tests for edge case: null user ID input" |
| **Security Integration** | Vulnerabilities, best practices | "🚨 SQL injection risk - use parameterized queries" |

**Effective Review Feedback Framework**:

| Element | Poor Example | Excellent Example |
|---------|--------------|-------------------|
| **Specificity** | "This is wrong" | "Line 42: SQL injection vulnerability - use parameterized query" |
| **Education** | "Fix this" | "This blocks event loop. Use async/await for DB calls (see docs/async-patterns.md)" |
| **Priority** | No classification | "🚨 Critical (blocking): Security vulnerability requires immediate fix" |
| **Solution** | Problem only | "Issue: N+1 query. Solution: Use eager loading with `.includes()`" |

**Work Item Integration Methods**:

| Method | Implementation | Auto-Linking |
|--------|---------------|--------------|
| **Branch from work item** | Click "Create branch" in work item | ✅ Automatic |
| **Commit message** | `git commit -m "feat: add feature (#12345)"` | ✅ Automatic |
| **PR description** | Add "Resolves AB#12345" in PR text | ✅ Automatic |
| **Manual linking** | Link work item in PR interface | ❌ Manual |

**Pipeline Integration Stages**:

| Stage | Duration | Tests | Trigger | Blocking |
|-------|----------|-------|---------|----------|
| **Fast Feedback** | < 5 min | Build, lint, unit tests | Every push | ✅ Yes |
| **Comprehensive** | 15-30 min | Integration, security scan | Review approval | ✅ Yes |
| **Staging Deploy** | Variable | Smoke tests, UAT | Pre-merge | ⚠️ Optional |

### Branch Policy Decision Matrix

| Scenario | Policy Configuration | Rationale |
|----------|---------------------|-----------|
| **Small team (< 5)** | 1 reviewer, build validation, work item optional | Balance velocity with minimal overhead |
| **Growing team (5-20)** | 2 reviewers, auto-assignment, build validation, work item required | Knowledge distribution, audit trail |
| **Enterprise (20+)** | 2-3 reviewers, required reviewers, comment resolution, full validation | Governance, compliance, quality |
| **Financial services** | 2 reviewers, no self-approval, security review, work item mandatory | SOX compliance, regulatory audit trail |

### Metrics for Continuous Improvement

| Metric | Target | Action if Below Target |
|--------|--------|----------------------|
| **PR cycle time** | < 24 hours | Analyze bottlenecks: slow reviews? Failed builds? |
| **Time to first review** | < 4 hours | Add more reviewers? Improve availability? |
| **Review participation** | > 80% | Rotate reviewers? Gamification? Recognition? |
| **Defects caught in review** | > 70% | Improve review training? Add checklists? |
| **Policy compliance** | 100% | Investigate bypass reasons? Adjust policies? |

## Study Resources

**Official Documentation**:
- [Advanced Branch Policies](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies)
- [Azure DevOps Integration](https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-requests)
- [Code Review Excellence](https://learn.microsoft.com/en-us/azure/devops/repos/git/review-pull-requests)
- [Enterprise Git Workflows](https://learn.microsoft.com/en-us/azure/devops/repos/git/about-pull-requests)

**Key Commands Reference**:
```bash
# Create feature branch from work item
git checkout -b feature/12345-implement-oauth

# Commit with work item linking
git commit -m "feat(auth): implement OAuth 2.0 (#12345)"

# Push and create PR
git push -u origin feature/12345-implement-oauth

# Azure CLI: Get PR metrics
az repos pr list --status completed \
  --query "[].{ID:pullRequestId, Title:title, Created:creationDate, Closed:closedDate}"
```

**CODEOWNERS Template**:
```
# Repository default owners
* @devops-team

# Security-sensitive code
/src/auth/** @security-team
/src/crypto/** @security-team @senior-developers

# Infrastructure as code
/infrastructure/** @platform-team @sre-team

# Database migrations
/migrations/** @database-team

# Frontend code
/src/frontend/** @frontend-team

# API endpoints
/src/api/** @backend-team @api-architects
```

## Common Misconceptions

**Misconception #1**: "More reviewers = better quality"
- **Reality**: 2-3 reviewers is optimal. Too many causes diffusion of responsibility.

**Misconception #2**: "All comments must be fixed immediately"
- **Reality**: Priority classification allows non-blocking suggestions for future improvement.

**Misconception #3**: "Build validation slows down development"
- **Reality**: Fast feedback (<5 min) catches issues early, saving time overall.

**Misconception #4**: "Work item linking is just bureaucracy"
- **Reality**: Essential for compliance, audit trails, impact analysis, and project tracking.

**Misconception #5**: "Senior developers don't need reviewers"
- **Reality**: Everyone benefits from review - knowledge distribution, catching blind spots.

## Practice Scenarios

**Scenario 1**: You're configuring branch policies for a financial services application requiring SOX compliance.

**Question**: Which policies are mandatory?
**Answer**: 
- ✅ Minimum 2 reviewers with no self-approval (separation of duties)
- ✅ Work item linking (audit trail requirement)
- ✅ Comment resolution (documented decision-making)
- ✅ Build validation with security scanning (quality assurance)

**Scenario 2**: Your team's average PR cycle time is 72 hours, causing deployment delays.

**Question**: How do you identify and fix bottlenecks?
**Answer**:
1. Use Azure DevOps Analytics to break down cycle time:
   - Time to first review: 12 hours (too slow)
   - Review discussion time: 48 hours (too long)
   - Build validation time: 5 minutes (acceptable)
   - Time from approval to merge: 7 hours (too slow)
2. Solutions:
   - Add more reviewers to reduce first review time
   - Implement review checklists to focus discussions
   - Automate merge after approval (remove manual step)

**Scenario 3**: A developer needs to merge a critical hotfix but doesn't have a linked work item.

**Question**: How should the team handle this?
**Answer**:
1. **Short-term**: Use policy bypass capability (if configured) with:
   - Manager approval
   - Documentation of emergency reason
   - Commitment to create work item post-merge
2. **Long-term**: Process improvement:
   - Create "Hotfix" work item type with expedited workflow
   - Template for rapid work item creation
   - Policy adaptation for emergency scenarios

## Critical Notes
- 🎯 **Assessment focus**: Branch policies, collaboration patterns, Azure DevOps integration
- 💡 **Key policies**: Minimum 2 reviewers, automatic assignment, comment resolution, work item linking
- ⚠️ **Four pillars**: Architectural consistency, knowledge amplification, quality systematization, security integration
- 📊 **Metrics**: Cycle time (< 24h), first review (< 4h), participation (> 80%), compliance (100%)
- 🔄 **Integration**: Azure Boards for work items, Azure Pipelines for validation
- ✨ **Continuous improvement**: Use analytics to identify bottlenecks, iterate on policies

[Take Knowledge Check](https://learn.microsoft.com/en-us/training/modules/collaborate-pull-requests-azure-repos/4-knowledge-check)
