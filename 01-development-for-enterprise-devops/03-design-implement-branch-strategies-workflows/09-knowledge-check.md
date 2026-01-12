# Module Assessment - Knowledge Check

## Purpose
Test your understanding of Git branch strategies and workflows for continuous delivery.

## Assessment Format
- **Duration**: 10 minutes
- **Type**: Multiple-choice questions
- **Coverage**: All module topics
- **Passing**: Understanding verification (not graded)

## Key Topics Covered

### Branching Workflow Types
**Assessment Areas**:
- GitHub Flow vs. Feature Branch Workflow vs. Release Branch Workflow
- Forking Workflow for distributed teams and open source
- Workflow selection criteria based on team size and release cadence
- Continuous delivery branching models

### Git Branch Management
**Assessment Areas**:
- Feature branch creation and naming conventions
- Pull request workflows and code review processes
- Merge strategies (merge commit, squash, rebase)
- Branch lifecycle management and cleanup

### Branch Protection and Policies
**Assessment Areas**:
- Azure DevOps branch policies configuration
- GitHub branch protection rules
- Minimum reviewer requirements and code owner reviews
- Build validation and status check integration

### Advanced Workflows
**Assessment Areas**:
- Fork workflow implementation for enterprise security
- Hotfix branch creation from release tags
- Merge conflict resolution strategies
- Git tags for release tracking

## Self-Assessment Checklist

Before taking the knowledge check, verify you can:

### Workflow Selection and Strategy
- [ ] **Compare branching workflows** - Understand differences between GitHub Flow, Feature Branch, Release Branch, and Forking workflows
- [ ] **Select appropriate workflow** - Choose workflow based on team size, release frequency, quality requirements
- [ ] **Evaluate tradeoffs** - Understand complexity, learning curve, and tool support for each workflow

### Feature Branch Workflow
- [ ] **Create feature branches** - Follow naming conventions ([type]/[ticket-id]-[description])
- [ ] **Make atomic commits** - Write clear commit messages with conventional format
- [ ] **Open pull requests** - Create PRs with comprehensive descriptions and context
- [ ] **Conduct code reviews** - Review code constructively and address feedback
- [ ] **Merge strategically** - Choose appropriate merge strategy (merge commit, squash, rebase)

### GitHub Flow
- [ ] **Implement 6-step process** - Create branch, develop, commit/push, open PR, review/merge, delete branch
- [ ] **Maintain deployable main** - Keep main branch always production-ready
- [ ] **Automate quality gates** - Integrate CI/CD, security scanning, status checks

### Fork Workflow
- [ ] **Understand dual-layer architecture** - Local repository + personal server-side fork
- [ ] **Configure upstream remote** - Sync with canonical repository
- [ ] **Create pull requests from forks** - Submit contributions without direct write access
- [ ] **Maintain security boundaries** - Contributors never have direct canonical repo access

### Branch Protection
- [ ] **Configure Azure DevOps policies** - Set minimum reviewers, work item linking, build validation
- [ ] **Set GitHub protection rules** - Require PR reviews, status checks, signed commits
- [ ] **Implement quality gates** - Enforce automated testing and code quality checks
- [ ] **Manage bypass permissions** - Control emergency override capabilities

### Continuous Delivery Model
- [ ] **Maintain always-ready main** - Main branch always deployable
- [ ] **Create hotfix branches** - Branch from release tags for production fixes
- [ ] **Resolve merge conflicts** - Update feature branch from main, resolve, commit
- [ ] **Tag releases** - Mark versions and milestones for traceability

## Exam Preparation Tips

### Core Concepts to Master

**GitHub Flow** (Recommended for Most Teams):
- Single main branch always deployable
- Feature branches for all changes
- Pull request workflow with reviews
- Continuous deployment or scheduled releases
- Simple, scalable, low learning curve

**Feature Branch Workflow**:
- Main branch + feature branches
- Formal review processes required
- Weekly to monthly release cycles
- Moderate complexity, good for teams transitioning

**Release Branch Workflow**:
- Dedicated release preparation branches
- Quarterly or seasonal release cycles
- Extensive testing and compliance validation
- High complexity, formal QA processes

**Forking Workflow**:
- Distributed ownership model
- Open source projects or external contributors
- Security isolation with access controls
- Maintainer-controlled integration

### Branch Protection Essentials

**Azure DevOps Policies**:
- Minimum reviewers (configurable threshold)
- Work item linking for traceability
- Build validation with Azure Pipelines
- Merge strategy control (basic, squash, rebase)
- Code owner notification (CODEOWNERS file)

**GitHub Protection Rules**:
- Required PR reviews (1-6 reviewers)
- Status check requirements (CI, tests, security)
- Signed commits for authenticity
- Linear history enforcement
- Administrator protection (no bypass)

### Workflow Decision Matrix

| Factor | GitHub Flow | Feature Branch | Release Branch | Forking |
|--------|-------------|----------------|----------------|---------|
| **Team Size** | Any | 5-25 | 10-50 | Any |
| **Release Frequency** | Continuous | Weekly-Monthly | Monthly-Quarterly | Variable |
| **Learning Curve** | Low | Moderate | High | Moderate |
| **Best For** | Speed, simplicity | Formal reviews | Compliance, QA | Open source, security |

## Study Resources

**Official Documentation**:
- [Git branching guidance - Azure Repos](https://learn.microsoft.com/en-us/azure/devops/repos/git/git-branching-guidance)
- [Create a new Git branch from the web - Azure Repos](https://learn.microsoft.com/en-us/azure/devops/repos/git/create-branch)
- [How Microsoft develops modern software with DevOps](https://learn.microsoft.com/en-us/devops/develop/how-microsoft-develops-devops)
- [Fork your repository - Azure Repos](https://learn.microsoft.com/en-us/azure/devops/repos/git/forks)

## Critical Notes
- 🎯 **Knowledge check**: 10-minute assessment of module understanding
- 💡 **Self-assessment**: Use checklist before attempting quiz
- ⚠️ **Key topics**: Workflow selection, feature branches, GitHub Flow, fork workflow, branch protection
- 📊 **Focus areas**: Strategy selection, implementation, policies, continuous delivery
- 🔄 **Not graded**: Designed for learning verification, not scoring
- ✨ **Preparation**: Review all 10 units before assessment

[Take Knowledge Check](https://learn.microsoft.com/en-us/training/modules/manage-git-branches-workflows/9-knowledge-check)
