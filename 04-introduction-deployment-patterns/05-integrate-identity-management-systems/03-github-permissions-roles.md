# Design and Implement Permissions and Roles in GitHub

**Duration**: 5 minutes

Learn how GitHub implements role-based access control across personal accounts, organizations, and enterprises with graduated permission models for secure collaboration.

---

## Overview

GitHub provides **three permission tiers** reflecting different collaboration scopes: **Personal Accounts** (binary Owner/Collaborator roles), **Organizations** (7 built-in roles for teams), and **Enterprise** (3 additional roles for multi-organization governance). Understanding these permission models enables least-privilege access control design and secure DevOps workflows.

---

## Personal Account Permissions

**Scope**: Individual repositories owned by personal GitHub accounts

### Permission Model: Binary Roles

GitHub personal accounts support **two permission levels**:

#### 1. Owner
**Definition**: Account holder with full administrative control

**Permissions**:
- ✅ Read/write all repositories (public + private)
- ✅ Delete repositories
- ✅ Manage repository settings (webhooks, deploy keys, secrets)
- ✅ Transfer repository ownership
- ✅ Manage account security settings (2FA, SSH keys, PATs)
- ✅ Grant collaborator access
- ✅ Billing and subscription management

**Assignment**: Automatic (account creator)

#### 2. Collaborator
**Definition**: External user granted explicit repository access

**Permissions**:
- ✅ Read/write repository code (clone, push, pull)
- ✅ Create issues and pull requests
- ✅ Review pull requests
- ✅ Manage issues (labels, milestones, assignments)
- ✅ Edit wiki (if enabled)
- ❌ **Cannot** delete repository
- ❌ **Cannot** change repository settings
- ❌ **Cannot** add other collaborators

**Assignment**: Owner grants collaborator access (Settings → Collaborators)

### Limitation: No Read-Only Collaborators

**Critical Constraint**: Personal accounts **cannot grant read-only access** to private repositories.

**Collaborator Access Levels**:
- ✅ **Public repositories**: Anyone can read (no collaborator access needed)
- ✅ **Private repositories**: Collaborators get **read AND write** access (no read-only option)
- ❌ **Read-only private access**: Not supported (requires organization)

**Workaround**: Create organization for granular read-only permissions.

---

## Organization Permissions

**Scope**: Collaborative entities grouping multiple repositories and teams for centralized administration

### Organization Role Hierarchy

Organizations support **7 built-in roles** with graduated permissions:

```
Organization Owner (Full administrative control)
├── Organization Member (Default baseline permissions)
├── Organization Moderator (Community governance + blocking)
├── Billing Manager (Financial administration only)
├── Security Manager (Security alerts + universal read) [PUBLIC BETA]
├── GitHub App Manager (App registration settings)
└── Outside Collaborator (Non-member repository access)
```

---

### 1. Organization Owner

**Definition**: Comprehensive administrative authority over organization, repositories, teams, and members.

#### Core Permissions
- ✅ **Full repository control**: Create, delete, transfer repositories
- ✅ **Member management**: Invite, remove, change member roles
- ✅ **Team administration**: Create teams, manage team membership, set team permissions
- ✅ **Billing management**: Update payment methods, view invoices, manage subscriptions
- ✅ **Security policies**: Enforce 2FA, SAML SSO, IP allow lists
- ✅ **Organization settings**: Rename organization, delete organization
- ✅ **Webhooks**: Configure organization-wide webhooks
- ✅ **GitHub Apps**: Install, configure, uninstall apps
- ✅ **Audit log access**: View organization audit log

#### Security Best Practices
- **Minimum 2 owners**: Prevent single point of failure
- **Limit owner count**: Minimize attack surface (5-10% of members)
- **Enable 2FA requirement**: Settings → Security → Require two-factor authentication
- **Review ownership quarterly**: Remove inactive owners

#### When to Assign
- **Executive leadership**: CTO, VP Engineering (strategic oversight)
- **Platform administrators**: DevOps leads, security leads (operational control)
- **Avoid**: Individual contributors, contractors, temporary members

---

### 2. Organization Member

**Definition**: Default non-administrative role for regular organization contributors.

#### Core Permissions
- ✅ **View members**: See organization member list
- ✅ **Create repositories**: If organization settings permit (configurable)
- ✅ **Create teams**: If organization settings permit (configurable)
- ✅ **Access public repositories**: Read public organization repos
- ✅ **Access private repositories**: Only repos explicitly granted via team membership
- ❌ **Cannot** manage organization settings
- ❌ **Cannot** manage billing
- ❌ **Cannot** remove members
- ❌ **Cannot** enforce security policies

#### Visibility Levels
**Public Membership**: Profile displays organization badge
**Private Membership**: Organization affiliation hidden from public

#### Repository Access
**Access Mechanism**: Team membership drives permissions
```
Organization Member (baseline)
    └── Team: "Backend Engineers" (write access)
        └── Repository: "api-service" (inherited write permission)
```

#### When to Assign
- **Default role**: All employees, full-time contributors
- **Automated provisioning**: Entra ID/Okta SSO syncs to Member role
- **Long-term collaborators**: Contractors with multi-year engagements

---

### 3. Organization Moderator

**Definition**: Community governance role for managing contributor interactions and enforcing code of conduct.

#### Core Permissions
- ✅ **Block/unblock users**: Prevent malicious actors from organization access
- ✅ **Limit interactions**: Restrict who can comment, open issues, create PRs
- ✅ **Lock conversations**: Prevent further comments on controversial issues/PRs
- ✅ **Hide comments**: Remove spam, harassment, off-topic discussions
- ✅ **Mark comments as duplicate/off-topic/resolved**: Organize discussions
- ✅ **View member list**: See organization members
- ✅ **View private repository names**: (Visibility only, no code access)
- ❌ **Cannot** access private repository code (unless granted separately)
- ❌ **Cannot** manage members or teams
- ❌ **Cannot** change organization settings

#### Use Cases
- **Community managers**: Enforce code of conduct in open-source projects
- **Support teams**: Manage issue/PR interactions
- **Developer advocates**: Moderate public discussions

#### Best Practices
- **Document moderation policies**: Transparent enforcement guidelines
- **Train moderators**: Code of conduct, escalation procedures
- **Audit moderation actions**: Review blocked users quarterly

#### When to Assign
- **Open-source maintainers**: Projects with large external contributor base
- **Community managers**: Organizations with public-facing repositories
- **Support engineering teams**: Issue triage and management

---

### 4. Billing Manager

**Definition**: Financial administration role with payment instrument management, separated from code/repository access.

#### Core Permissions
- ✅ **View billing information**: Invoices, payment history, usage reports
- ✅ **Update payment methods**: Credit cards, bank accounts, purchase orders
- ✅ **Download invoices**: PDF receipts for accounting
- ✅ **Change billing email**: Update invoice recipient
- ✅ **View organization membership count**: For billing calculation
- ❌ **Cannot** access repositories (public or private)
- ❌ **Cannot** view organization code
- ❌ **Cannot** manage members or teams
- ❌ **Cannot** change organization settings (non-billing)

#### Financial Segregation of Duties
**Principle**: Separate financial control from technical access

**Access Pattern**:
```
Billing Manager (Finance Team)
    ├── View: Subscription, invoices, payment history
    ├── Update: Payment methods, billing email
    └── NO ACCESS: Repositories, code, teams, members

Organization Owner (Engineering Leadership)
    ├── Full control: Repositories, teams, members
    └── View: Billing information (read-only optional)
```

#### Compliance Benefits
- ✅ **Audit compliance**: Financial access segregated from technical
- ✅ **Least privilege**: Finance team doesn't need code access
- ✅ **Cost accountability**: Billing visibility without operational control

#### When to Assign
- **Finance department**: Accounts payable, procurement teams
- **Office managers**: Administrative staff handling subscriptions
- **Accounting teams**: Invoice management, budget tracking
- **Avoid**: Developers, engineers (use Member role + billing read access if needed)

---

### 5. Security Manager (Public Beta)

**Definition**: Security-focused role with comprehensive alert visibility and code security configuration authority.

#### Core Permissions
- ✅ **View security alerts**: Dependabot, code scanning, secret scanning (all repos)
- ✅ **Manage security settings**: Enable/disable security features organization-wide
- ✅ **Configure code scanning**: Set up CodeQL, third-party analysis tools
- ✅ **Secret scanning configuration**: Enable secret detection, custom patterns
- ✅ **Universal read access**: Read all repositories (public + private) [CRITICAL]
- ✅ **View organization teams**: See team structure
- ✅ **View security audit log**: Track security-related events
- ❌ **Cannot** write to repositories (read-only code access)
- ❌ **Cannot** manage members (except security settings)
- ❌ **Cannot** delete repositories or change non-security settings

#### Universal Read Access Implications
**CRITICAL**: Security Managers can **read all private repository code** without explicit permission grants.

**Risk Assessment**:
```
Security Manager Role
    └── Read Access: ALL private repositories
        ├── Source code
        ├── Commit history
        ├── Pull requests
        ├── Issues (including private security advisories)
        └── Repository settings (read-only)
```

**Least-Privilege Consideration**: Only assign to trusted security personnel with organizational security responsibilities.

#### Security Features Management
**Dependabot**:
- Configure vulnerability alerts
- Enable automatic security updates
- Customize alert thresholds

**Code Scanning**:
- Enable CodeQL analysis
- Configure custom scanning workflows
- Manage third-party integrations (Snyk, Checkmarx)

**Secret Scanning**:
- Enable automatic secret detection
- Add custom secret patterns
- Configure alert routing

#### When to Assign
- **Security engineers**: Application security, vulnerability management
- **Compliance teams**: Security audit, regulatory requirements
- **Security operations**: Incident response, threat monitoring
- **Avoid**: Developers (use Member role + repository-specific security permissions)

#### Public Beta Status
**Availability**: GitHub Enterprise Cloud (not available for Free/Team plans)
**Stability**: Feature may change before general availability

---

### 6. GitHub App Manager

**Definition**: Specialized role for managing GitHub App registrations, excluding installation/uninstall operations.

#### Core Permissions
- ✅ **Modify GitHub App settings**: Update app name, description, permissions, webhook URL
- ✅ **Rotate private keys**: Generate new authentication keys
- ✅ **Manage client secrets**: Create/revoke OAuth credentials
- ✅ **Configure webhooks**: Update event subscriptions
- ✅ **View app installations**: See where app is installed
- ❌ **Cannot** install apps to repositories
- ❌ **Cannot** uninstall apps
- ❌ **Cannot** approve installation requests
- ❌ **Cannot** access repositories

#### Installation vs Configuration Separation
**Design Principle**: Separate app development from deployment control

**Role Separation**:
```
GitHub App Manager (App Development Team)
    ├── Update: App settings, webhooks, permissions
    ├── Rotate: Private keys, client secrets
    └── NO CONTROL: Installation approval, deployment scope

Organization Owner (Platform Team)
    ├── Install: Approve app installations
    ├── Configure: Repository access scope
    └── Uninstall: Remove app access
```

#### Use Cases
- **App development teams**: Maintain GitHub Apps without organization-wide installation control
- **Integration engineers**: Update webhook endpoints, rotate credentials
- **DevOps teams**: Manage CI/CD app configurations

#### When to Assign
- **GitHub App developers**: Teams building custom GitHub integrations
- **Integration maintainers**: Engineers responsible for app configuration
- **DevOps engineers**: Managing CI/CD GitHub App credentials
- **Avoid**: General organization members (unless actively developing apps)

---

### 7. Outside Collaborator

**Definition**: Non-member users granted specific repository access for limited-scope contributions.

#### Core Permissions
- ✅ **Access granted repositories**: Only repositories explicitly assigned
- ✅ **Repository-level permissions**: Read, Triage, Write, Maintain, Admin (per repo)
- ✅ **Create issues/PRs**: Contribute to granted repositories
- ✅ **Participate in discussions**: Comment, review, collaborate
- ❌ **Cannot** view organization member list
- ❌ **Cannot** see other private repositories
- ❌ **Cannot** create teams
- ❌ **Cannot** view organization settings

#### Repository Access Levels (Outside Collaborator)

**Read**:
- Clone repository, view code, download releases
- Cannot push commits or create branches

**Triage**:
- Read + Manage issues/PRs (labels, milestones, assignments)
- Useful for support teams, issue triage engineers

**Write**:
- Read + Push commits, create branches, merge PRs
- Standard contributor access

**Maintain**:
- Write + Manage repository settings (webhooks, branch protection)
- Senior contributor/maintainer access

**Admin**:
- Maintain + Delete repository, transfer ownership
- Delegated repository ownership (uncommon for outside collaborators)

#### Use Cases

**Contractors/Consultants**:
```
Outside Collaborator: "consultant@external.com"
    └── Repository: "project-alpha" (Write access)
        ├── Duration: 6-month contract
        ├── Scope: Feature development
        └── Revocation: Automatic contract end
```

**Client Collaboration**:
```
Outside Collaborator: "client-developer@partner.com"
    └── Repository: "partner-integration" (Read access)
        ├── Purpose: API integration development
        ├── Visibility: Single repository only
        └── Security: No organization visibility
```

**Open-Source Contributors**:
```
Outside Collaborator: "community-contributor@gmail.com"
    └── Repository: "open-source-project" (Triage access)
        ├── Purpose: Issue triage, community support
        ├── Privileges: Label management, issue assignment
        └── Trust level: Earned through contributions
```

#### Security Considerations
- ✅ **Minimal organization visibility**: Outside collaborators cannot see org structure
- ✅ **Repository-scoped access**: Granular permission control
- ✅ **Easy revocation**: Remove access without affecting other repos
- ⚠️ **Branch protection applies**: Outside collaborators subject to repo policies
- ⚠️ **Activity audited**: Actions logged in repository audit log

#### Outside Collaborator vs Organization Member

| Feature | Outside Collaborator | Organization Member |
|---------|---------------------|---------------------|
| **Organization visibility** | No (minimal) | Yes (full member list) |
| **Repository access** | Explicit per-repo grants | Team-based inheritance |
| **Create teams** | No | Yes (if permitted) |
| **View private repo names** | No | Yes (names only) |
| **Cost** | Free (doesn't count toward license) | Paid seat |
| **Use case** | Contractors, clients, limited collaboration | Employees, long-term contributors |

#### When to Assign
- **Short-term contractors**: 3-6 month engagements, single-project focus
- **External partners**: Client developers, integration engineers
- **Trial contributors**: Evaluate before full organization membership
- **Temporary access**: Specific project contributions
- **Cost optimization**: Free seats for limited-scope collaborators

#### Best Practices
- **Document collaboration scope**: Clear expectations for access duration
- **Regular access reviews**: Quarterly audit of outside collaborators
- **Revoke on project completion**: Remove access when contract ends
- **Use branch protection**: Require PR reviews even for outside collaborators
- **Monitor activity**: Review audit logs for unusual access patterns

---

## Enterprise Account Permissions

**Scope**: Multi-organization governance for large enterprises managing multiple GitHub organizations under unified policies.

### Enterprise Role Hierarchy

GitHub Enterprise adds **3 additional roles** for cross-organization administration:

```
Enterprise Owner (Full enterprise control)
├── Enterprise Member (Organization auto-inclusion)
└── Guest Collaborator (Restricted internal visibility)
```

---

### 1. Enterprise Owner

**Definition**: Top-tier administrative role with comprehensive authority across all organizations, policies, and billing within the enterprise.

#### Core Permissions
- ✅ **Manage enterprise administrators**: Add/remove enterprise owners, billing managers
- ✅ **Create/delete organizations**: Provision new organizations, decommission old ones
- ✅ **Enforce enterprise policies**: Security, repository, member policies cascaded to all orgs
- ✅ **Centralized billing**: View/manage subscriptions across all organizations
- ✅ **SAML SSO configuration**: Enterprise-wide identity provider integration
- ✅ **Audit log access**: View enterprise-level audit events
- ✅ **Support ticket management**: Enterprise support access
- ✅ **Organization ownership**: Automatically inherit owner role in all enterprise orgs
- ❌ **Cannot** automatically access all repositories (organization-level permissions apply)

#### Enterprise Policies (Cascading Control)

**Repository Policies**:
```
Enterprise Owner Sets Policy
    └── Repository creation: Restrict to organization owners only
        └── Applied to: ALL organizations in enterprise
            ├── Engineering Org: ✅ Policy enforced
            ├── Marketing Org: ✅ Policy enforced
            └── Data Science Org: ✅ Policy enforced
```

**Security Policies**:
- **2FA enforcement**: Require MFA for all enterprise members
- **IP allow lists**: Restrict access to corporate network ranges
- **SSH certificate authorities**: Centralized SSH CA management

**Member Policies**:
- **Base permissions**: Default repository access for organization members
- **Repository visibility**: Allow/restrict private repository creation

#### Multi-Organization Governance Example

**Enterprise Structure**:
```
ACME Corporation (Enterprise)
├── Organization: Engineering
│   ├── Teams: Backend, Frontend, DevOps
│   └── Repositories: 200+ repos
├── Organization: Data Science
│   ├── Teams: ML Engineering, Analytics
│   └── Repositories: 50+ repos
└── Organization: Open Source
    ├── Teams: Community, Maintainers
    └── Repositories: 15+ public repos

Enterprise Owner: "cto@acme.com"
    ├── Policy: Enforce 2FA across ALL organizations
    ├── Policy: Restrict repo creation to org owners
    ├── Billing: Centralized payment for 500 seats
    └── Organization access: Owner in Engineering, Data Science, Open Source
```

#### When to Assign
- **C-level executives**: CTO, CISO (strategic enterprise oversight)
- **Enterprise architects**: Cross-organization platform governance
- **Security leadership**: Enterprise-wide security policy enforcement
- **Avoid**: Individual organization administrators (use Organization Owner instead)

---

### 2. Enterprise Member

**Definition**: Enterprise-level membership automatically granting access to internal-visibility repositories across all enterprise organizations.

#### Core Permissions
- ✅ **Automatic organization inclusion**: Visible in enterprise member directory
- ✅ **Internal repository access**: Read internal-visibility repos across organizations
- ✅ **Organization-specific permissions**: Inherit role within each org (Member, Owner, etc.)
- ❌ **Cannot** access private repositories (requires explicit org/team grants)
- ❌ **Cannot** view all organizations (only orgs where explicitly added as member)
- ❌ **Cannot** enforce enterprise policies

#### Repository Visibility Levels (Enterprise Context)

**Private**: Only accessible to organization members with explicit permissions
**Internal**: Accessible to **all enterprise members** (cross-organization)
**Public**: Accessible to everyone (internet-wide)

**Internal Visibility Example**:
```
Repository: "acme-design-system" (Internal visibility)
└── Accessible by:
    ├── Engineering Org members: ✅ Auto-access
    ├── Data Science Org members: ✅ Auto-access
    ├── Marketing Org members: ✅ Auto-access (if enterprise members)
    └── External contractors: ❌ No access (not enterprise members)

Purpose: Shared internal resources without manual permission grants
```

#### Use Cases
- **Cross-organization collaboration**: Shared libraries, design systems, documentation
- **InnerSource programs**: Internal open-source across business units
- **Enterprise-wide standards**: Coding guidelines, security policies accessible to all

#### When Assigned
- **Automatic**: SSO-provisioned users become enterprise members
- **Manual**: Enterprise owners add members to enterprise
- **Inherited**: Organization membership may confer enterprise membership

---

### 3. Guest Collaborator

**Definition**: External users granted restricted access to internal-visibility repositories without full enterprise membership.

#### Core Permissions
- ✅ **Access internal repositories**: Only internal repos explicitly granted
- ✅ **Repository-level permissions**: Read/Write/Admin per repo
- ✅ **Collaborate with teams**: Participate in granted repository workflows
- ❌ **Cannot** see enterprise member directory
- ❌ **Cannot** discover other internal repositories
- ❌ **Cannot** access private repositories (unless separately granted)

#### Use Cases

**Vendor Collaboration**:
```
Guest Collaborator: "vendor@external.com"
└── Repository: "acme-vendor-integration" (Internal visibility)
    ├── Access: Write (push code, create PRs)
    ├── Visibility: Single repo only (not other internal repos)
    └── Duration: Contract period (6 months)
```

**Client Development**:
```
Guest Collaborator: "client-dev@partner.com"
└── Repository: "acme-client-sdk" (Internal visibility)
    ├── Access: Read (view code, documentation)
    ├── Purpose: Integration development
    └── Security: No enterprise-wide discovery
```

#### Guest Collaborator vs Enterprise Member

| Feature | Guest Collaborator | Enterprise Member |
|---------|-------------------|-------------------|
| **Internal repo discovery** | No (explicit grants only) | Yes (all internal repos) |
| **Enterprise directory** | Not visible | Visible to other members |
| **Cross-org access** | Repo-specific only | All internal repos across orgs |
| **Cost** | Varies by license | Included in enterprise seat |
| **Use case** | External vendors, clients | Employees, long-term contractors |

#### When to Assign
- **External vendors**: Contractors from partner companies
- **Client developers**: Customer-side integration engineers
- **Temporary consultants**: Short-term specialized expertise
- **Audit firms**: External auditors needing code review access

---

## Permission Inheritance and Precedence

### Organization Permission Hierarchy

```
Organization Owner (Global control)
├── Repository-level permissions
│   ├── Repository Admin (full repo control)
│   ├── Repository Maintain (settings + write)
│   ├── Repository Write (push commits)
│   ├── Repository Triage (manage issues/PRs)
│   └── Repository Read (view code)
│
└── Team-based permissions
    ├── Team Maintainer (manage team members)
    └── Team Member (inherit repository access)
```

### Permission Resolution

**Precedence Rules**:
1. **Organization Owner**: Always has full access (overrides all repo permissions)
2. **Repository Admin**: Full repository control (overrides team permissions)
3. **Team Permissions**: Inherited by team members
4. **Direct Collaborator**: Explicit repository-level grants
5. **Base Permissions**: Organization-wide default (Public/None)

**Example**:
```
User: "developer@acme.com"
├── Organization Role: Member (baseline)
├── Team: "Backend Engineers" (Write access to "api-service")
├── Direct Grant: "api-service" Admin (explicit)
└── Effective Permission: Admin (direct grant takes precedence)
```

---

## Repository Permission Levels (Detailed)

### Read
- ✅ Clone repository
- ✅ View code, issues, pull requests
- ✅ Download releases
- ✅ Comment on issues/PRs
- ❌ Cannot push commits
- ❌ Cannot create branches
- ❌ Cannot close/reopen issues

**Use Case**: Contractors reviewing code, QA engineers testing releases

### Triage
- ✅ Read permissions +
- ✅ Manage issues (labels, milestones, assignments)
- ✅ Manage pull requests (request reviews, labels)
- ✅ Apply/dismiss stale PR reviews
- ❌ Cannot push commits
- ❌ Cannot merge pull requests

**Use Case**: Support engineers, issue triage teams, community managers

### Write
- ✅ Triage permissions +
- ✅ Push commits to non-protected branches
- ✅ Create branches
- ✅ Merge pull requests (if branch protection allows)
- ✅ Edit wiki
- ❌ Cannot change repository settings
- ❌ Cannot delete repository
- ❌ Cannot force-push to protected branches

**Use Case**: Software engineers, feature developers, team contributors

### Maintain
- ✅ Write permissions +
- ✅ Manage repository settings (webhooks, deploy keys)
- ✅ Configure branch protection rules
- ✅ Manage GitHub Actions secrets
- ✅ Edit repository description, topics
- ❌ Cannot delete repository
- ❌ Cannot transfer repository ownership
- ❌ Cannot manage team access

**Use Case**: Tech leads, repository maintainers, senior engineers

### Admin
- ✅ Maintain permissions +
- ✅ Delete repository
- ✅ Transfer repository ownership
- ✅ Manage collaborators and team access
- ✅ Configure security settings (secret scanning, code scanning)
- ✅ Archive repository
- ✅ Manage repository visibility (private ↔ public)

**Use Case**: Repository owners, project leads, organization owners

---

## Best Practices Summary

### Organization Role Assignment

#### Minimize Owners
- **Recommendation**: 2-3 owners for small orgs (<50 members), 5-10 for large orgs (>500 members)
- **Rationale**: Reduce attack surface, prevent accidental destructive actions
- **Enforcement**: Regular quarterly reviews, remove inactive owners

#### Default to Member Role
- **Recommendation**: Assign Member role to all employees by default
- **Rationale**: Grant additional permissions via team membership (least privilege)
- **Implementation**: SSO auto-provisioning → Member role

#### Use Outside Collaborator for Temporary Access
- **Recommendation**: Contractors, vendors, trial contributors → Outside Collaborator
- **Rationale**: Free seats, isolated access, easy revocation
- **Review**: Quarterly audit, remove completed projects

#### Delegate with Maintain Role
- **Recommendation**: Grant Maintain (not Admin) to repository maintainers
- **Rationale**: Sufficient permissions for daily operations without deletion risk
- **Example**: Tech leads managing branch protection, webhooks

### Security Hardening

#### Enforce 2FA
```
Organization Settings → Security → Authentication security
✅ Require two-factor authentication for everyone
```

#### Enable SAML SSO
```
Organization Settings → Security → SAML single sign-on
✅ Enable SAML authentication
✅ Require SAML SSO authentication for all members
```

#### Review Permissions Quarterly
```
Organization Settings → People
    ├── Filter: Organization Owners → Review necessity
    ├── Filter: Outside Collaborators → Remove completed projects
    └── Audit Log → Review permission changes
```

#### Use Branch Protection
```
Repository Settings → Branches → Branch protection rules
✅ Require pull request reviews before merging
✅ Require status checks to pass
✅ Restrict who can push to matching branches (Owners + Maintain only)
```

---

## Permission Comparison: GitHub vs Azure DevOps

| Feature | GitHub Organizations | Azure DevOps Organizations |
|---------|---------------------|---------------------------|
| **Permission Model** | Role-based (7 roles) | Hybrid (groups + permissions + access levels) |
| **Granularity** | Repository-level | Project-level + object-level |
| **Default Assignment** | Member role | Contributors group + Basic access |
| **Read-Only Option** | Yes (Read permission) | Yes (Readers group) |
| **External Collaborators** | Outside Collaborator (free) | Stakeholder (free, limited) |
| **Identity Integration** | SAML SSO + team sync | Native Entra ID sync |
| **Role Inheritance** | Team → Repository | Group → Project → Object |
| **Cost Model** | Per-seat licensing | Per-seat + access level tiers |

---

## Next Steps

✅ **Completed**: GitHub permission models and role-based access control

**Next Unit**: Learn about Azure DevOps permissions, security groups, and Entra ID integration →

---

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-identity-management-systems/3-design-implement-permissions-roles-github)
