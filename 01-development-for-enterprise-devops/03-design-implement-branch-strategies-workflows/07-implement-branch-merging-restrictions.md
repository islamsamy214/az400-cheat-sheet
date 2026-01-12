# Implement Branch Merging Restrictions

## Enterprise Value Proposition

Modern branch protection policies deliver measurable business value through:
- **Automated quality gates** reducing defect leakage to production
- **Compliance automation** satisfying regulatory requirements (SOX, HIPAA)
- **Risk mitigation** preventing unauthorized or untested code deployment
- **Process standardization** ensuring consistent practices across development teams
- **Audit trail creation** supporting security reviews and compliance audits

## Platform-Agnostic Strategic Principles

While implementation details vary between Azure DevOps and GitHub, both platforms provide feature parity for enterprise-grade branch protection, enabling organizations to maintain consistent governance regardless of platform choice.

## Azure DevOps: Enterprise Branch Protection Implementation

### Configuration Strategy

Navigate to your repository in the Azure DevOps portal and select target branches for protection. Leverage pattern-based protection to apply consistent policies across current and future branches matching specified criteria.

**Access Path**: Repository → Branches → Branch policies → Select branch → Configure policies

### Core Protection Policies

#### Review and Approval Framework
- **Minimum reviewer requirements**: Enforce peer review through configurable approval thresholds
- **Work item integration**: Ensure traceability by requiring linked work items for all changes
- **Comment resolution**: Mandate resolution of all review feedback before merge completion

**Configuration Example**:
- Minimum 2 reviewers required
- At least 1 reviewer must be from CODEOWNERS file
- All comments must be resolved
- Original author cannot approve their own changes

#### Merge Strategy Control

Control repository history through selective merge type enablement:

| Merge Type | History Impact | Use Case |
|------------|----------------|----------|
| **Basic merge (no fast forward)** | Preserves complete development history | Audit trails, detailed history |
| **Rebase and fast-forward** | Creates linear history by replaying commits | Clean history without merge commits |
| **Squash merge** | Condenses feature development into single commits | Simplified history, atomic changes |
| **Rebase with merge commit** | Combines linear replay with explicit merge documentation | Linear history with merge markers |

### Advanced Quality Gates

#### Continuous Integration
- **Build validation**: Automatic pre-merge validation through CI pipeline execution
- **Status checks**: Multi-service validation requiring successful external verification
- **Automated testing**: Comprehensive test suite execution before merge authorization

**Azure Pipelines Integration**:
```yaml
# Branch policy configuration
trigger:
  branches:
    include:
    - main
    - release/*

pr:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- script: npm install
  displayName: 'Install dependencies'

- script: npm test
  displayName: 'Run tests'

- script: npm run lint
  displayName: 'Code quality checks'
```

#### Stakeholder Engagement
- **Code owner notification**: Automatic reviewer assignment based on file modification patterns
- **Subject matter expert inclusion**: Specialized review requirements for critical code areas

**CODEOWNERS File Example**:
```
# Repository owners
* @devops-team

# Frontend code
/src/frontend/** @frontend-team

# Backend API
/src/api/** @backend-team

# Security-sensitive code
/src/auth/** @security-team @senior-developers

# Infrastructure as code
/infrastructure/** @platform-team @sre-team
```

### Security and Compliance Controls

**Branch locking**: Read-only enforcement for maintenance periods or release freezes

**Policy bypass management**: Controlled override capabilities for emergency scenarios

**Bypass Configuration**:
- **Emergency merge permissions**: Bypass policies for critical production fixes
- **Administrative override**: Senior developer access for exceptional circumstances

⚠️ **Security Best Practice**: Limit bypass permissions to designated personnel who understand compliance implications and can exercise appropriate judgment during emergency situations.

## GitHub: Advanced Branch Protection Rules

### Configuration Access and Scope

Access branch protection configuration through your repository's Settings → Branches interface. Apply rules to specific branches or use pattern matching for scalable policy management across branch hierarchies.

**Pattern Matching Examples**:
- `main` - Protect main branch only
- `release/*` - Protect all release branches
- `feature/*` - Apply policies to all feature branches

### Enterprise Protection Framework

#### Collaborative Review Requirements
- **Mandatory pull request workflow**: Enforces structured review processes for all changes
- **Status check integration**: Multi-service validation ensuring comprehensive quality assessment
- **Discussion resolution**: Ensures all code review feedback is addressed before integration

**GitHub Protection Rules Configuration**:
```
Require pull request reviews before merging:
  ☑ Required number of approvals: 2
  ☑ Dismiss stale pull request approvals when new commits are pushed
  ☑ Require review from Code Owners
  ☑ Restrict who can dismiss pull request reviews
  ☑ Allow specified actors to bypass required pull requests

Require status checks to pass before merging:
  ☑ Require branches to be up to date before merging
  Status checks that are required:
    - CI Build
    - Unit Tests
    - Security Scan
    - Code Quality

Require conversation resolution before merging: ☑

Other rules:
  ☑ Require linear history
  ☑ Require signed commits
  ☑ Include administrators
  ☐ Allow force pushes
  ☐ Allow deletions
```

#### Security and Authenticity Controls
- **Signed commit enforcement**: Cryptographic verification of code authorship and integrity
- **Linear history requirements**: Prevents merge commits to maintain simplified, auditable history
- **Deployment validation**: Pre-merge deployment testing in staging environments

**GPG Commit Signing**:
```bash
# Configure Git to sign commits
git config --global user.signingkey YOUR_GPG_KEY_ID
git config --global commit.gpgsign true

# Sign commits manually
git commit -S -m "feat: add new feature"
```

#### Advanced Governance Features
- **Administrator protection**: Prevents policy bypass even by repository administrators
- **Force push controls**: Emergency override capabilities with audit trail requirements
- **Branch deletion protection**: Safeguards against accidental or malicious branch removal

### Strategic Implementation Considerations

**Emergency Access Management**: Configure emergency override capabilities judiciously, balancing operational flexibility with security requirements.

**Audit and Compliance**: Leverage protection rules to create comprehensive audit trails supporting regulatory compliance and security reviews.

**Developer Experience**: Balance protection rigor with development velocity to maintain team productivity while ensuring quality standards.

### GitHub Enterprise Security Model

Modern GitHub Enterprise implementations provide sophisticated protection capabilities that exceed basic open-source requirements:
- **Organizational policy inheritance** for consistent protection across repositories
- **Advanced audit logging** supporting compliance and security monitoring
- **Integration with enterprise identity systems** for seamless access control
- **Automated policy enforcement** reducing administrative overhead while maintaining governance standards

## Branch Protection Comparison Matrix

| Feature | Azure DevOps | GitHub Enterprise |
|---------|--------------|-------------------|
| **Minimum Reviewers** | ✅ Configurable threshold | ✅ 1-6 reviewers |
| **Code Owner Review** | ✅ CODEOWNERS integration | ✅ CODEOWNERS integration |
| **Build Validation** | ✅ Azure Pipelines integration | ✅ GitHub Actions integration |
| **Work Item Linking** | ✅ Required work item association | ⚠️ Via PR templates (not enforced) |
| **Signed Commits** | ⚠️ Recommended, not enforced | ✅ Can be enforced |
| **Linear History** | ✅ Via merge strategy control | ✅ Enforce linear history option |
| **Administrator Bypass** | ✅ Configurable | ✅ Can include administrators |
| **Pattern-Based Rules** | ✅ Branch pattern matching | ✅ Wildcard branch protection |
| **Audit Logging** | ✅ Comprehensive audit trail | ✅ Enterprise audit log |

## Implementation Best Practices

### Start Simple, Add Incrementally
1. **Phase 1**: Require pull requests + 1 reviewer
2. **Phase 2**: Add build validation
3. **Phase 3**: Enforce code owner reviews
4. **Phase 4**: Add status checks (tests, security, quality)
5. **Phase 5**: Implement signed commits and linear history

### Team Communication
- **Announce changes**: Give teams advance notice of new policies
- **Provide training**: Educate on workflow changes and bypass procedures
- **Document exceptions**: Clear process for emergency overrides
- **Collect feedback**: Iterate based on team experience

### Monitoring and Optimization
- **Track metrics**: Pull request completion time, bypass frequency
- **Review policies**: Quarterly assessment of effectiveness
- **Adjust as needed**: Balance security with developer productivity

## Critical Notes
- 🎯 **Enterprise value**: Automated quality gates, compliance automation, risk mitigation
- 💡 **Platform parity**: Azure DevOps and GitHub both provide enterprise-grade protection
- ⚠️ **Azure DevOps**: Work item linking, merge strategy control, build validation
- 📊 **GitHub**: Signed commits, linear history, administrator protection
- 🔒 **Security best practice**: Limit bypass permissions to designated personnel
- ✨ **Phased approach**: Start simple, add protections incrementally based on team maturity

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-branches-workflows/7-implement-branch-merging-restrictions)
