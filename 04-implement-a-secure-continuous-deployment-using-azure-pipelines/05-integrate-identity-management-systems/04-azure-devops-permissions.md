# Design and Implement Permissions and Security Groups in Azure DevOps

**Duration**: 5 minutes

Learn how Azure DevOps implements a three-tier access control model combining membership management, permission assignment, and access levels for granular security governance.

---

## Overview

Azure DevOps employs a **3-tier authorization framework**: **Membership Management** (security principals in groups), **Permission Management** (explicit/inherited allow/deny rules), and **Access Level Management** (license-based feature visibility). Understanding this hybrid model enables least-privilege design and automated identity lifecycle management through Microsoft Entra ID integration.

---

## Three-Tier Access Control Model

### Authorization Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  TIER 1: MEMBERSHIP MANAGEMENT                               │
│  "Who belongs to which groups?"                              │
│  ┌────────────────┐   ┌────────────────┐  ┌──────────────┐ │
│  │ Entra ID Groups│──▶│Azure DevOps    │  │ Custom       │ │
│  │ (External)     │   │Built-in Groups │  │ Security     │ │
│  │                │   │ (Collection,   │  │ Groups       │ │
│  │- Engineering   │   │  Project)      │  │              │ │
│  │- DevOps Team   │   └────────┬───────┘  └──────┬───────┘ │
│  └────────┬───────┘            │                  │         │
└───────────┼────────────────────┼──────────────────┼─────────┘
            │                    │                  │
            ▼                    ▼                  ▼
┌─────────────────────────────────────────────────────────────┐
│  TIER 2: PERMISSION MANAGEMENT                               │
│  "What actions can groups/users perform?"                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Permission Scopes:                                    │  │
│  │ - Organization-level (Collection Administrators)      │  │
│  │ - Project-level (Contributors, Project Admins)        │  │
│  │ - Object-level (Repository, Pipeline, Work Item)      │  │
│  │                                                        │  │
│  │ Rules: Allow, Deny, Not Set (inherited)               │  │
│  │ Precedence: Deny > Explicit Allow > Inherited Allow   │  │
│  └──────────────────────────────────────────────────────┘  │
└───────────────────────────────────┬─────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────┐
│  TIER 3: ACCESS LEVEL MANAGEMENT                             │
│  "Which features are visible/accessible?"                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ License Tiers:                                        │  │
│  │ - Stakeholder (Free, view-only + limited work items) │  │
│  │ - Basic (Paid, read/write repos, pipelines, boards)  │  │
│  │ - Basic + Test Plans (Test case management)          │  │
│  │ - Visual Studio Enterprise (Advanced features)       │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

FINAL ACCESS = (Membership ∩ Permissions ∩ Access Level)
```

---

## Tier 1: Membership Management

**Purpose**: Define security principals (users, groups) and their group associations.

### Security Principal Types

#### Individual Users
- **Source**: Microsoft Entra ID (preferred), Microsoft Account (MSA), Active Directory
- **Identity**: user@organization.com
- **Management**: Direct group assignment or Entra ID group inheritance

#### Azure DevOps Groups
- **Built-In Groups**: Pre-configured groups (Collection Administrators, Contributors)
- **Custom Groups**: Organization-defined groups for specific permissions
- **Nested Groups**: Groups containing other groups (inheritance hierarchy)

#### Microsoft Entra ID Groups
- **Security Groups**: Synchronized from Entra ID tenant
- **Dynamic Membership**: Rule-based automatic assignment
- **Integration**: Azure DevOps group rules map Entra ID groups → Azure DevOps groups

---

## Organization Security Groups (Collection-Level)

**Scope**: Organization-wide permissions applying across all projects

### 1. Project Collection Administrators (Collection Administrators)

**Definition**: Highest privilege level with full organizational control.

#### Core Permissions
- ✅ **Create/delete projects**: Provision new projects, archive old ones
- ✅ **Manage organization settings**: Billing, security policies, extensions
- ✅ **Create/manage build agents**: Configure self-hosted agent pools
- ✅ **Manage service connections**: Create Azure subscriptions, external service connections
- ✅ **Configure organization-level permissions**: Add/remove collection administrators
- ✅ **Install/uninstall extensions**: Marketplace extensions (SonarQube, Slack)
- ✅ **Audit log access**: View organization audit events
- ✅ **Billing management**: Update payment methods, view invoices
- ✅ **Full access to all projects**: Inherit Project Administrator role

**When to Assign**:
- **Platform administrators**: DevOps leads, infrastructure engineers
- **Executive oversight**: CTO, VP Engineering (strategic control)
- **Avoid**: Project team members, developers (use Project Administrators)

**Best Practice**: Limit to 2-5 members for small orgs (<100 users), 10-15 for large orgs (>1000 users)

---

### 2. Project Collection Build Administrators (Build Administrators)

**Definition**: Specialized role for managing organization-wide build infrastructure.

#### Core Permissions
- ✅ **Manage organization-level build agents**: Create/delete agent pools
- ✅ **Configure build service accounts**: Build Service Account permissions
- ✅ **Administer build resources**: Storage, artifacts, retention policies
- ✅ **Configure build queues**: Priority, concurrency limits
- ❌ **Cannot** create/delete projects
- ❌ **Cannot** manage organization settings (non-build)

**When to Assign**:
- **Build/release engineers**: Teams managing CI/CD infrastructure
- **DevOps engineers**: Platform teams administering build agents

---

### 3. Project Collection Build Service Accounts

**Definition**: System identities for build/release pipeline execution.

#### Service Account Format
```
[Organization Name] Build Service ([Organization Name])
Example: "Contoso Build Service (Contoso)"
```

#### Permissions
- ✅ **Execute build/release pipelines**: Run tasks, scripts, deployments
- ✅ **Access repositories**: Clone code, push commits (if configured)
- ✅ **Publish artifacts**: Upload build outputs to Azure Artifacts
- ✅ **Update work items**: Link builds to user stories, bugs

**Automatic Management**: Azure DevOps automatically adds build service accounts to required groups

---

### 4. Project Collection Proxy Service Accounts

**Definition**: System identities for Azure Artifacts proxy caching service.

**Use Case**: Organizations with distributed teams caching packages locally

---

### 5. Project Collection Service Accounts

**Definition**: Generic service account group for automated systems.

**When to Assign**: Custom automation scripts, third-party integrations requiring organizational access

---

### 6. Project Collection Test Service Accounts

**Definition**: System identities for test execution and reporting.

**Permissions**: Execute test plans, publish test results, update test cases

---

### 7. Project Collection Valid Users

**Definition**: Auto-populated group containing ALL organization members.

#### Characteristics
- ✅ **Automatic membership**: Everyone added to organization automatically included
- ✅ **Read-only default permissions**: View projects, pipelines (configurable)
- ❌ **Cannot remove members manually**: Membership controlled by organization inclusion
- ❌ **Should not grant write permissions**: Use specific project groups instead

**Purpose**: Baseline permissions applying to entire organization

---

### 8. Project-Scoped Users

**Definition**: Users restricted to specific projects (cannot view other projects).

**Use Case**: Contractors, vendors with single-project access requirements

**Configuration**:
```
Organization Settings → Users → Add users
✅ "Limit user visibility and collaboration to specific projects"
Select Projects: [Project-Alpha]
```

**Result**: User can only see Project-Alpha, not other organization projects

---

### 9. Security Service Group

**Definition**: System group for managing security namespace operations.

**Internal Use**: Azure DevOps platform internal operations

---

## Project Security Groups (Project-Level)

**Scope**: Project-specific permissions not inherited across projects

### 1. [Project Name] Build Administrators

**Definition**: Project-level build infrastructure management.

#### Core Permissions
- ✅ **Manage project build pipelines**: Create, edit, delete pipelines
- ✅ **Configure pipeline permissions**: Grant/revoke pipeline access
- ✅ **Manage pipeline variables**: Create/edit secret variables
- ✅ **Configure build retention policies**: Artifact retention, cleanup rules
- ❌ **Cannot** manage organization-level build agents
- ❌ **Cannot** access other projects' pipelines

**When to Assign**: Project build/release engineers, CI/CD leads

---

### 2. [Project Name] Contributors (Default Group)

**Definition**: Primary group for team members with read/write access.

#### Core Permissions
- ✅ **Read/write repositories**: Clone, push, create branches, PRs
- ✅ **Manage work items**: Create user stories, bugs, tasks; update states
- ✅ **Run build/release pipelines**: Queue builds, trigger releases
- ✅ **Publish artifacts**: Upload packages to Azure Artifacts
- ✅ **Create test plans**: Author test cases, execute test suites
- ✅ **Edit wiki**: Documentation collaboration
- ❌ **Cannot** delete project
- ❌ **Cannot** manage project settings (rename, archive)
- ❌ **Cannot** grant permissions to other users
- ❌ **Cannot** delete repositories

**Default Access Level**: Basic (read/write across all project features)

**When to Assign**: **Default group for all team members** (developers, QA engineers, analysts)

---

### 3. [Project Name] Project Administrators

**Definition**: Project ownership role with comprehensive project control.

#### Core Permissions
- ✅ **All Contributors permissions** +
- ✅ **Manage project settings**: Rename project, configure areas/iterations
- ✅ **Create/delete repositories**: Provision new repos, archive old ones
- ✅ **Manage teams**: Create sub-teams, configure team settings
- ✅ **Configure project-level permissions**: Grant/revoke group memberships
- ✅ **Manage service connections**: Create Azure subscriptions, external connections
- ✅ **Configure branch policies**: Require PR reviews, status checks
- ✅ **Delete work items**: Permanently remove backlog items
- ❌ **Cannot** delete project (requires Collection Administrator)
- ❌ **Cannot** manage organization settings

**When to Assign**: Project leads, technical program managers, senior architects

---

### 4. [Project Name] Project Valid Users

**Definition**: Auto-populated group containing ALL project members.

#### Characteristics
- ✅ **Automatic membership**: Everyone added to project automatically included
- ✅ **Read-only default permissions**: View repositories, work items, pipelines
- ❌ **Cannot remove members manually**: Membership controlled by project inclusion
- ❌ **Should not grant write permissions**: Use Contributors or custom groups

**Purpose**: Baseline read access for all project participants

---

### 5. [Project Name] Readers

**Definition**: Read-only access group for stakeholders, auditors, and observers.

#### Core Permissions
- ✅ **View repositories**: Read code, view commit history (no clone/download)
- ✅ **View work items**: Read user stories, bugs, tasks (no editing)
- ✅ **View pipelines**: See pipeline runs, logs, results (no triggering)
- ✅ **View test plans**: Read test cases, test results (no execution)
- ✅ **View wiki**: Read documentation (no editing)
- ❌ **Cannot** clone repositories
- ❌ **Cannot** create/edit work items
- ❌ **Cannot** run pipelines
- ❌ **Cannot** access artifacts

**When to Assign**: Executive stakeholders, auditors, compliance reviewers, external observers

---

### 6. [Project Name] Release Administrators

**Definition**: Release pipeline management role.

#### Core Permissions
- ✅ **Manage release pipelines**: Create, edit, delete release definitions
- ✅ **Configure release approvals**: Set up manual/automated gates
- ✅ **Manage release variables**: Create/edit pipeline variables
- ✅ **Configure deployment permissions**: Grant/revoke release access
- ❌ **Cannot** manage build pipelines (different permission scope)
- ❌ **Cannot** access other projects' releases

**When to Assign**: Release managers, deployment engineers, production gatekeepers

---

### 7. [Project Name] Team (Default Project Team)

**Definition**: Automatically created team corresponding to project.

**Purpose**: Default team for work item tracking (Boards area path, iteration path)

---

## Default Permission Provisioning

### New User Default Access

**When user added to Azure DevOps organization**:
```
User: developer@contoso.com added to organization
    └── Automatically assigned:
        ├── Membership: [Project Name] Contributors group
        ├── Access Level: Basic (read/write access)
        └── Effective Permissions:
            ├── Repos: Clone, push, create branches, PRs
            ├── Pipelines: Queue builds, view logs
            ├── Boards: Create work items, update states
            ├── Artifacts: Publish packages
            └── Test Plans: Create/execute test cases (if licensed)
```

**Rationale**: Productive baseline permissions enabling immediate contribution

---

### Valid User Groups (Auto-Population)

#### Collection Valid Users
**Membership**: ALL organization members (automatic inclusion)
**Default Permissions**: Read-only organizational visibility
**Use Case**: Organization-wide announcements, dashboards

#### Project Valid Users
**Membership**: ALL project members (automatic inclusion)
**Default Permissions**: Read-only project visibility
**Use Case**: Project-wide reports, documentation access

**Permission Example**:
```
Project Valid Users:
    ├── View project dashboard: ✅ Allow
    ├── View work items: ✅ Allow
    ├── View repository: ✅ Allow
    ├── Clone repository: ❌ Deny (requires Contributors)
    └── Edit work items: ❌ Deny (requires Contributors)
```

---

## Tier 2: Permission Management

### Permission Scopes

#### Organization-Level Permissions
**Scope**: Organization-wide settings, cross-project features

**Examples**:
- Create/delete projects
- Manage organization-level extensions
- Configure organization billing
- Create organization-level service connections

**Groups**: Collection Administrators, Build Administrators

---

#### Project-Level Permissions
**Scope**: Project-specific features, settings

**Examples**:
- Manage project settings (areas, iterations, teams)
- Delete project
- Create/delete repositories
- Manage project-level service connections

**Groups**: Project Administrators, Contributors, Readers

---

#### Object-Level Permissions
**Scope**: Specific object instances (repository, pipeline, work item)

**Examples**:

**Repository**:
- Read, Contribute, Force push, Manage permissions
- Branch-specific: Create branch, Contribute to pull requests

**Pipeline**:
- View builds, Queue builds, Edit build pipeline, Delete build pipeline

**Work Item**:
- View, Edit, Permanently delete work items

**Service Connection**:
- View, Use, Administer service connections

---

### Permission States

#### Allow
**Effect**: Explicitly grants permission
**Inheritance**: Can be inherited by child objects
**Precedence**: Overridden by Deny

#### Deny
**Effect**: Explicitly denies permission (strongest rule)
**Inheritance**: Blocks permission regardless of other Allow grants
**Precedence**: Overrides all Allow rules (inherited or explicit)

**Use Case**: Prevent Project Administrators from deleting specific critical repositories

**Example**:
```
User: senior-dev@contoso.com
├── Group: Project Administrators (Allow: Delete repository)
├── Explicit Deny: "production-api" repository (Deny: Delete repository)
└── Effective Permission: DENY (Deny overrides Allow)
```

#### Not Set (Inherited)
**Effect**: Inherits permission from parent scope or group membership
**Default State**: Most permissions not explicitly set
**Resolution**: Traverse hierarchy until explicit Allow/Deny found

---

### Permission Precedence Rules

**Resolution Order**:
```
1. Explicit Deny (object-level)
    ↓ Overrides all
2. Explicit Allow (object-level)
    ↓ If no Deny
3. Inherited Deny (parent scope)
    ↓ If no explicit rules
4. Inherited Allow (parent scope)
    ↓ If no explicit rules
5. Not Set (no permission)
```

**Example Scenario**:
```
Repository: "critical-service"
├── Project Contributors: Allow Contribute
├── Explicit Deny: user@contoso.com (Deny: Force push)
└── Effective for user@contoso.com:
    ├── Contribute: ✅ Allow (inherited from Contributors)
    └── Force push: ❌ Deny (explicit Deny overrides group Allow)
```

---

## Tier 3: Access Level Management

**Purpose**: License-based feature visibility and access control

### Access Level Tiers

#### Stakeholder (Free)
**Cost**: Free (unlimited users)

**Features**:
- ✅ **View work items**: Read-only boards, backlogs
- ✅ **Add/edit work items**: Create bugs, tasks (limited)
- ✅ **View pipelines**: See build/release status (no triggering)
- ✅ **View dashboards**: Organization/project dashboards
- ❌ **Cannot** access repositories (code viewing blocked)
- ❌ **Cannot** access test plans
- ❌ **Cannot** access artifacts (packages)

**When to Assign**: Business stakeholders, executives, external observers, auditors

---

#### Basic (Paid)
**Cost**: $6 per user/month (Visual Studio subscribers: 5 Basic included free)

**Features**:
- ✅ **All Stakeholder features** +
- ✅ **Full repository access**: Clone, push, pull, branches, PRs
- ✅ **Build/release pipelines**: Queue builds, create pipelines
- ✅ **Azure Artifacts**: Publish/consume packages
- ✅ **Wiki**: Create/edit documentation
- ✅ **Analytics**: Basic reporting, dashboards
- ❌ **Cannot** access test case management (Test Plans)

**When to Assign**: **Default for developers, engineers, DevOps team members**

---

#### Basic + Test Plans
**Cost**: $52 per user/month

**Features**:
- ✅ **All Basic features** +
- ✅ **Test Plans**: Create test cases, test suites, test configurations
- ✅ **Exploratory testing**: Manual testing, session recording
- ✅ **Test execution**: Run manual tests, log results
- ✅ **Test analytics**: Test reports, traceability matrices

**When to Assign**: QA engineers, test managers, manual testers

---

#### Visual Studio Enterprise Subscription
**Cost**: $250 per user/month (includes Visual Studio IDE + Azure DevOps)

**Features**:
- ✅ **All Basic + Test Plans features** +
- ✅ **Advanced analytics**: Power BI integration, custom reports
- ✅ **Package management**: Unlimited Azure Artifacts storage
- ✅ **Pipeline parallelism**: Increased concurrent job limits

**When to Assign**: Senior engineers, architects (if Visual Studio IDE required)

---

### Access Level Assignment

**Organization Settings → Users**:
```
User: developer@contoso.com
├── Access Level: Basic ($6/month)
├── Group Membership: [Project] Contributors
└── Effective Access:
    ├── Repository: ✅ Full access (Basic license + Contributors permissions)
    ├── Pipelines: ✅ Full access
    ├── Test Plans: ❌ Blocked (requires Basic + Test Plans license)
    └── Advanced Analytics: ❌ Blocked (requires VS Enterprise)
```

---

## Microsoft Entra ID Integration

### Automated Group Synchronization

**Workflow**:
```
1. Entra ID Security Group: "Engineering-Backend"
   ├── Members: 25 developers
   └── Management: Automated (HR system feeds Entra ID)

2. Azure DevOps Group Rule:
   Organization Settings → Permissions → Create Group Rule
   ├── Entra ID Group: "Engineering-Backend"
   ├── Azure DevOps Group: [Project-API] Contributors
   └── Action: "When Entra ID group membership changes, sync to Azure DevOps"

3. Automated Provisioning:
   ├── User added to "Engineering-Backend" in Entra ID
   │   └── Automatically added to [Project-API] Contributors in Azure DevOps
   ├── User removed from "Engineering-Backend" in Entra ID
   │   └── Automatically removed from [Project-API] Contributors in Azure DevOps
   └── Result: Zero manual Azure DevOps group management
```

### Group Rule Configuration

**Organization Settings → Permissions → Azure Active Directory → Group Rules**:
```
Rule: "Backend Engineers Auto-Provisioning"
├── Entra ID Group: "Engineering-Backend" (Object ID: 12345678-1234-1234-1234-123456789abc)
├── Azure DevOps Project: "Project-API"
├── Azure DevOps Group: [Project-API] Contributors
└── Access Level: Basic (auto-assign on sync)

Result:
    New "Engineering-Backend" member → Auto-added to "Project-API" with Basic license
```

---

## Six Best Practices for Permission Management

### 1. Plan Microsoft Entra ID Groups

**Strategy**: Align Entra ID groups with organizational structure and role requirements

**Implementation**:
```
Entra ID Security Groups:
├── "Engineering-All" (Everyone in Engineering)
├── "Engineering-Backend" (Backend developers)
├── "Engineering-Frontend" (Frontend developers)
├── "Engineering-DevOps" (Platform/DevOps team)
├── "Engineering-QA" (QA engineers)
└── "Engineering-Architects" (Senior architects)

Azure DevOps Group Mapping:
├── "Engineering-All" → [All Projects] Project Valid Users (read-only)
├── "Engineering-Backend" → [API Project] Contributors
├── "Engineering-Frontend" → [Web Project] Contributors
├── "Engineering-DevOps" → Collection Administrators
├── "Engineering-QA" → [All Projects] Contributors + Basic + Test Plans
└── "Engineering-Architects" → [All Projects] Project Administrators
```

**Benefits**:
- ✅ Centralized access control (Entra ID as source of truth)
- ✅ Automated lifecycle management (onboarding/offboarding)
- ✅ Role-based permissions (job function drives access)
- ✅ Audit compliance (identity changes logged in Entra ID)

---

### 2. Automate Entra ID → Azure DevOps Group Associations

**Strategy**: Use Azure DevOps group rules to automatically sync Entra ID group membership

**Configuration**:
```
Organization Settings → Permissions → Group Rules → Add Group Rule

Group Rule: "Backend Team Auto-Sync"
├── Entra ID Group: "Engineering-Backend"
├── Azure DevOps Groups:
│   ├── [API Project] Contributors
│   └── [Shared Libraries Project] Contributors
├── Access Level: Basic
└── Sync Frequency: Real-time (change notification via webhook)
```

**Result**:
```
HR System → Entra ID → Azure DevOps (automated chain)
    ├── Day 1: New hire added to "Engineering-Backend" in Entra ID
    │   └── Azure DevOps: Auto-added to [API Project] Contributors with Basic license
    ├── Day 180: Employee moves to frontend team
    │   ├── Entra ID: Removed from "Engineering-Backend", added to "Engineering-Frontend"
    │   └── Azure DevOps: Auto-removed from API Project, auto-added to Web Project
    └── Termination: Employee removed from Entra ID
        └── Azure DevOps: Auto-removed from all projects (license released)
```

**Benefits**:
- ✅ 80-90% reduction in manual access management
- ✅ Zero-delay provisioning (real-time sync)
- ✅ Automatic deprovisioning (terminated employees lose access immediately)
- ✅ Audit trail (all changes logged)

---

### 3. Use Default Azure DevOps Groups (Avoid Custom Proliferation)

**Strategy**: Leverage built-in groups (Contributors, Project Administrators, Readers) rather than creating numerous custom groups

**Anti-Pattern** (Too Many Custom Groups):
```
❌ Custom Groups (Excessive):
    ├── "Backend-ReadOnly"
    ├── "Backend-WriteOnly"
    ├── "Backend-PipelineAccess"
    ├── "Backend-FullAccess"
    ├── "Frontend-ReadOnly"
    └── ... (20+ custom groups)

Problems:
    ├── High management overhead
    ├── Permission conflicts (overlapping rules)
    ├── Difficult troubleshooting
    └── Audit complexity
```

**Best Practice** (Use Built-In Groups):
```
✅ Built-In Groups + Entra ID Sync:
    ├── [API Project] Contributors ← Entra ID: "Engineering-Backend"
    ├── [API Project] Readers ← Entra ID: "Engineering-Auditors"
    ├── [API Project] Project Administrators ← Entra ID: "Engineering-Backend-Leads"
    └── [All Projects] Collection Administrators ← Entra ID: "Engineering-DevOps"

Benefits:
    ├── Simple permission model
    ├── Clear role definitions
    ├── Easy troubleshooting
    └── Reduced management overhead
```

**Guidelines**:
- ✅ **Use Contributors** for standard team members (read/write access)
- ✅ **Use Readers** for stakeholders (read-only access)
- ✅ **Use Project Administrators** for project leads
- ✅ **Create custom groups** ONLY for specialized permissions (e.g., "Release Approvers")
- ❌ **Avoid Deny rules** unless absolutely necessary (explicit Deny creates complexity)

---

### 4. Delegate Management Responsibilities

**Strategy**: Distribute administrative authority to reduce bottlenecks and enable team autonomy

**Delegation Model**:
```
Collection Administrators (Platform Team)
├── Manages: Organization settings, billing, cross-project features
└── Delegates to: Project Administrators

Project Administrators (Project Leads)
├── Manages: Project settings, repository creation, team permissions
└── Delegates to: Repository Maintainers (Maintain permission)

Repository Maintainers (Senior Engineers)
├── Manages: Branch policies, webhooks, repository settings
└── Delegates to: Contributors (Write permission)
```

**Benefits**:
- ✅ Faster decision-making (no central bottleneck)
- ✅ Project autonomy (teams manage own resources)
- ✅ Reduced Collection Admin workload
- ✅ Ownership accountability (project leads responsible for access)

**Implementation**:
```
Project: API Service
├── Project Administrator: "tech-lead@contoso.com"
│   └── Responsibilities:
│       ├── Add/remove Contributors
│       ├── Configure branch policies
│       ├── Create service connections
│       └── Manage project teams
├── Repository "api-core" Maintainer: "senior-dev@contoso.com"
│   └── Responsibilities:
│       ├── Configure webhooks
│       ├── Manage deploy keys
│       ├── Set branch protection rules
│       └── Review PR settings
└── Collection Administrator: Only intervenes for project deletion, org settings
```

---

### 5. Review and Test Permissions Regularly

**Strategy**: Implement periodic permission audits and validation workflows

**Quarterly Review Process**:
```
Month 1 (January, April, July, October):
├── Collection Administrators Review:
│   ├── Export organization members: Organization Settings → Users → Export
│   ├── Identify inactive users: No sign-in > 90 days
│   ├── Remove terminated employees: Verify against HR system
│   └── Review Collection Admin membership: Minimum necessary (2-5 members)
│
├── Project Administrators Review:
│   ├── Export project members: Project Settings → Teams → Export
│   ├── Review group memberships: Custom groups, team assignments
│   ├── Validate Entra ID sync: Compare Entra ID groups vs Azure DevOps groups
│   └── Remove external collaborators: Completed contracts, expired access
│
└── Access Level Review:
    ├── Review license assignments: Basic, Basic + Test Plans, VS Enterprise
    ├── Identify underutilized licenses: Users not signing in, wrong access level
    ├── Optimize costs: Downgrade to Stakeholder where appropriate
    └── Forecast growth: Plan license procurement
```

**Automated Validation**:
```
Azure DevOps REST API Script (PowerShell):
# Check for users with Collection Administrator permissions
$orgUrl = "https://dev.azure.com/contoso"
$pat = $env:AZURE_DEVOPS_PAT

$headers = @{
    Authorization = "Basic " + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$pat"))
}

$collectionAdmins = Invoke-RestMethod -Uri "$orgUrl/_apis/graph/groups?api-version=7.0" -Headers $headers `
    | Where-Object { $_.displayName -eq "Project Collection Administrators" }

# Alert if > 5 members
if ($collectionAdmins.members.Count -gt 5) {
    Send-Alert "Excessive Collection Administrators: $($collectionAdmins.members.Count) members"
}
```

**Test Permission Changes**:
```
Before deploying permission changes:
1. Test in non-production project
2. Verify access with test user accounts
3. Document expected vs actual behavior
4. Review audit log for unintended changes
5. Rollback plan: Save previous permission state
```

---

### 6. Monitor and Audit Access

**Strategy**: Enable comprehensive activity tracking and anomaly detection

**Azure DevOps Auditing Feature**:
```
Organization Settings → Auditing
├── Enable Auditing: ✅
├── Audit Events:
│   ├── Permission changes (group membership, explicit grants)
│   ├── Security group modifications (create, delete, rename)
│   ├── Project creation/deletion
│   ├── Pipeline execution (build/release runs)
│   ├── Service connection usage
│   └── Extension installation
├── Audit Log Retention: 90 days (default), up to 365 days (configurable)
└── Export Options: JSON, CSV for SIEM integration
```

**Audit Streaming (SIEM Integration)**:
```
Organization Settings → Auditing → Streams
├── Stream: "Azure Sentinel Integration"
├── Target: Azure Event Hubs / Azure Monitor Logs
├── Events:
│   ├── Permissions.PermissionUpdate
│   ├── Group.GroupCreate
│   ├── Project.ProjectCreate
│   └── Pipelines.PipelineRun
└── Real-time Alerting:
    ├── Alert: "Collection Administrator added" → Immediate notification
    ├── Alert: "Project deleted" → Escalation to platform team
    └── Alert: "Unusual pipeline execution" → Security investigation
```

**Access Review Automation**:
```
Azure Logic App / Power Automate:
Trigger: Monthly schedule (1st day of month)
Actions:
1. Call Azure DevOps REST API: Get organization members
2. Call Entra ID API: Get active employees
3. Compare: Find Azure DevOps users NOT in Entra ID (terminated employees)
4. Generate Report: List of users to remove
5. Send Email: To Collection Administrators for review
6. Optional: Auto-remove users (if approved workflow)
```

**Security Incident Response**:
```
Incident: "Unauthorized repository deletion"
Investigation:
1. Audit Log: Search for "Permissions.RepositoryDelete" event
2. Identify: User "contractor@external.com" deleted "critical-api" repo
3. Timeline: Event timestamp, IP address, user agent
4. Root Cause: User had Project Administrator role (excessive permission)
5. Remediation: Restore from backup, revoke user access, adjust permissions
6. Prevention: Implement Deny rule for critical repos, require approval workflow
```

---

## Permission Comparison: Azure DevOps vs GitHub

| Feature | Azure DevOps | GitHub Organizations |
|---------|-------------|---------------------|
| **Permission Model** | 3-tier (Membership + Permissions + Access Levels) | Role-based (7 org roles) |
| **Identity Integration** | Native Entra ID sync (group rules) | SAML SSO + team sync |
| **Default Access** | Contributors + Basic | Organization Member |
| **Read-Only Option** | Readers group + Stakeholder access level | Read permission (repository-level) |
| **Group Hierarchy** | Nested groups, complex inheritance | Flat role model (simpler) |
| **External Collaborators** | Project-Scoped Users (limited projects) | Outside Collaborator (free, per-repo) |
| **License Cost** | Basic: $6/user/month | Team: $4/user/month, Enterprise: $21/user/month |
| **Audit Granularity** | Object-level (repository, pipeline, work item) | Organization-level + repository-level |
| **Access Level Tiers** | 4 tiers (Stakeholder, Basic, Basic+Test, VS Enterprise) | Binary (member vs non-member) |

---

## Quick Reference: Common Permission Scenarios

### Scenario 1: New Developer Onboarding
```
Requirement: Junior developer needs access to API project

Solution:
1. Add user to Entra ID group: "Engineering-Backend"
2. Group rule auto-syncs: User → [API Project] Contributors
3. Access level auto-assigned: Basic ($6/month)
4. Effective permissions: Read/write repos, run pipelines, manage work items
```

### Scenario 2: External Auditor (Read-Only)
```
Requirement: Compliance auditor needs code visibility, no editing

Solution:
1. Add user to organization: auditor@external.com
2. Assign access level: Stakeholder (free) or Basic (read-only repos)
3. Add to group: [All Projects] Readers
4. Effective permissions: View code, work items, pipelines (no modifications)
```

### Scenario 3: Contractor (Single Project, 6 Months)
```
Requirement: Contractor builds feature in single project, expires after contract

Solution:
1. Add user with project scoping: contractor@vendor.com
   ✅ "Limit user visibility to specific projects" → [Feature-X Project]
2. Assign access level: Basic ($6/month)
3. Add to group: [Feature-X Project] Contributors
4. Effective permissions: Full access to Feature-X only (cannot see other projects)
5. Offboarding: Remove user → automatic license release
```

### Scenario 4: Release Manager (Deployment Approval)
```
Requirement: Release manager approves production deployments

Solution:
1. Create custom security group: "Production Approvers"
2. Add user to group: release-manager@contoso.com
3. Configure release pipeline:
   - Pre-deployment approval: Required
   - Approvers: "Production Approvers" group
4. Effective permissions: Approve/reject production releases (no pipeline editing)
```

### Scenario 5: Prevent Critical Repository Deletion
```
Requirement: No one (including Project Admins) can delete "core-api" repository

Solution:
1. Navigate: Project Settings → Repositories → "core-api" → Security
2. Find: Project Administrators group
3. Set explicit Deny: "Delete repository" → Deny
4. Result: Project Admins have full access EXCEPT deletion (Deny overrides Allow)
```

---

## Next Steps

✅ **Completed**: Azure DevOps permission management and Entra ID integration

**Next Unit**: Learn about workload identities and service principals for automation →

---

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-identity-management-systems/4-design-implement-permissions-security-groups)
