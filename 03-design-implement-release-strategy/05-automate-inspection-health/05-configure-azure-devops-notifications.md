# Configure Azure DevOps Notifications

⏱️ **Duration**: ~2 minutes | 📚 **Type**: Configuration Guide

## Overview

Notification configuration requires **systematic audience analysis** followed by comprehensive rule-based subscription management to ensure effective team communication and project awareness. Azure DevOps enables notification configuration through default templates, custom subscriptions, and hierarchical scoping for comprehensive project activity monitoring.

---

## Azure DevOps Notification System

Azure DevOps enables notification configuration through:
- **Rule-based subscriptions**: Define custom notification rules
- **Default out-of-the-box (OOB) templates**: Pre-configured common scenarios  
- **Custom administrator-defined scenarios**: Organization-specific workflows

---

## Notification Triggers

Comprehensive project lifecycle events:

### Work Items
- ✅ Assignment changes (work item assigned to me)
- ✅ State transitions (bug moved from Active to Resolved)
- ✅ Field updates (priority changed from Medium to High)
- ✅ Comment additions (new discussion on my work item)

### Code Reviews
- ✅ Review requests (I'm added as reviewer)
- ✅ Approval status (my approval is required)
- ✅ Completion notifications (code review completed)

### Pull Requests
- ✅ Creation (new PR in my repository)
- ✅ Approval (PR approved by reviewer)
- ✅ Merge completion (PR merged to main branch)
- ✅ Conflict resolution (merge conflicts detected)

### Source Control Files
- ✅ Check-ins (code committed to my branch)
- ✅ Branch operations (branch created, deleted, or renamed)
- ✅ Repository changes (TFVC or Git)

### Builds
- ✅ Completion status (build succeeded or failed)
- ✅ Failure alerts (my build failed)
- ✅ Queue notifications (build queued for execution)

### Releases
- ✅ Deployment success (production deployment succeeded)
- ✅ Failure conditions (staging deployment failed)
- ✅ Approval requirements (approval needed for production)

---

## Event-Specific Scenarios

### Example 1: Build Completion Notifications

**Use Case**: Get immediate feedback on continuous integration builds

**Configuration**:
```
Subscription: "My Build Completions"
├── Event: Build completed
├── Filters:
│   ├── Status: Any (succeeded, failed, partiallySucceeded)
│   ├── Requested by: @Me
│   └── Definition: All build definitions
└── Delivery: Email + Slack DM
```

**Notification Example**:
```
✅ Build Succeeded - MyApp-CI #20240115.3

Branch: main
Commit: abc1234 "Fix payment gateway timeout"
Duration: 8 minutes 23 seconds
Tests: 120 passed, 0 failed
Coverage: 87% (+2% from last build)
Artifacts: myapp-build-1.2.3.zip (45 MB)

[View Build] [Download Artifacts] [Queue New Build]
```

---

### Example 2: Release Failure Alerts

**Use Case**: Immediate response to production deployment failures

**Configuration**:
```
Subscription: "Production Deployment Failures"
├── Event: Release deployment completed
├── Filters:
│   ├── Status: Failed
│   ├── Release definition: MyApp-Production
│   ├── Stage name: Production
│   └── Requested by: Any
└── Delivery: Email + Slack #production-alerts + Mobile push
```

**Notification Example**:
```
🚨 PRODUCTION DEPLOYMENT FAILED - IMMEDIATE ACTION REQUIRED

Release: Release-456
Environment: Production
Stage: Deploy to Azure Web App
Status: ❌ Failed
Error: Health check failed - HTTP 500 Internal Server Error
Duration: 2 minutes 34 seconds (failed at health check)
Triggered by: john.doe@company.com
Build: Build-123 (commit abc1234)

Health Check Details:
├── Endpoint: https://myapp-prod.azurewebsites.net/health
├── Expected: HTTP 200
├── Actual: HTTP 500 (85% error rate)
└── Detection: 2 minutes after deployment

Actions:
[View Logs] [Retry Deployment] [Rollback] [Contact On-Call]
```

---

## Notification Management Hierarchy

Azure DevOps notification management operates across **four hierarchical scopes** for comprehensive organizational coverage:

### 1. Personal Notifications

**Scope**: Individual user subscriptions

**Use Case**: Relevant to specific user's work

**Examples**:
- Work items assigned to me
- My pull requests need review
- Builds I requested failed
- Code I pushed triggered policy violation

**Configuration Path**:
```
User Settings → Notifications → Personal notifications
```

**Example Subscription**:
```
Subscription: "My High Priority Work Items"
├── Event: Work item created or modified
├── Filters:
│   ├── Assigned to: @Me
│   ├── Priority: High or Critical
│   └── State: Active
└── Delivery: Email + mobile push
```

---

### 2. Team Notifications

**Scope**: Group-specific alerts

**Use Case**: Collaborative workflows and shared responsibilities

**Examples**:
- Pull requests targeting team's repository
- Builds for team's build definitions
- Work items in team's area path
- Releases for team's release pipelines

**Configuration Path**:
```
Project Settings → Notifications → Team: [TeamName]
```

**Example Subscription**:
```
Subscription: "Team Pull Request Activity"
├── Event: Pull request created
├── Filters:
│   ├── Target branch: main
│   ├── Repository: MyApp
│   └── Area path: MyTeam/**
└── Delivery: Team email alias + Slack #team-channel
```

---

### 3. Project Notifications

**Scope**: Project-wide announcements

**Use Case**: Cross-team awareness, milestone communications

**Examples**:
- Release to production (any team)
- Critical bugs created
- Security vulnerabilities detected
- Major build definition changes

**Configuration Path**:
```
Project Settings → Notifications → Project notifications
```

**Example Subscription**:
```
Subscription: "Project-Wide Production Deployments"
├── Event: Release deployment completed
├── Filters:
│   ├── Environment: Production
│   ├── Status: Succeeded or Failed
│   └── Project: MyApplication
└── Delivery: Project email list + Slack #project-updates
```

---

### 4. Global Notifications

**Scope**: Organization-level alerts

**Use Case**: Enterprise-wide visibility and governance

**Examples**:
- New projects created in organization
- Organization policy violations
- License usage alerts
- Security compliance issues

**Configuration Path**:
```
Organization Settings → Notifications → Global notifications
```

**Example Subscription**:
```
Subscription: "Organization Security Alerts"
├── Event: Code scanning alert created
├── Filters:
│   ├── Severity: High or Critical
│   ├── State: Active
│   └── Organization: MyOrganization
└── Delivery: Security team email + PagerDuty
```

---

## Configure Global Notifications (Step-by-Step)

### Steps to Manage Global Notifications

**1. Navigate to Organization Settings**:
```
URL: https://dev.azure.com/{organization}/_settings/organizationOverview
```

**2. Access Global Notifications**:
```
Organization settings (bottom left)
    ↓
General tab
    ↓
Global notifications
```

![Screenshot Placeholder: Azure DevOps global notifications page]

---

### Default Subscriptions Tab

**What You'll See**:
- List of all default global subscriptions available
- Globe icon (🌐) indicates default subscription
- Pre-configured notification scenarios

**Default Subscription Examples**:
```
🌐 Build completes (organization-wide)
🌐 Release deployment approval pending (organization-wide)
🌐 Work item assigned to me (personal)
🌐 Pull request created (team)
```

**View All Default Subscriptions**: [Default notification subscriptions documentation](https://learn.microsoft.com/en-us/azure/devops/organizations/notifications/oob-built-in-notifications)

---

### Context Menu Options

**Available Actions** (...) for each subscription:
- ✅ **Enable**: Activate subscription for organization
- ❌ **Disable**: Deactivate subscription
- 👁️ **View Details**: See subscription configuration
- ✏️ **Edit**: Modify filters and delivery (admins only)
- 📊 **View Statistics**: See notification volume

**Permission Requirements**:

| Action | Required Permission |
|--------|---------------------|
| **View subscription details** | Project Collection Valid Users |
| **Enable/Disable subscription** | Project Collection Administrators |
| **Edit subscription** | Project Collection Administrators |
| **Delete custom subscription** | Project Collection Administrators |

---

### Subscribers Tab

**Purpose**: View users subscribed to each notification

**Information Displayed**:
- User/group list for each subscription
- Subscription delivery preferences (email, Slack, etc.)
- Opt-in/opt-out status

**Example**:
```
Subscription: "Build completes (global)"
├── Subscribers: 45 users, 3 teams
├── Delivery Preferences:
│   ├── Email: 40 users
│   ├── Slack: 8 users
│   └── Disabled: 0 users
└── Opt-out Users: john.doe@company.com (too many notifications)
```

---

### Settings Section

**Default Delivery Option Setting**:

**Purpose**: Set organization-wide default delivery channel

**Options**:
- Email (default)
- Microsoft Teams
- Slack
- Custom webhook

**Inheritance**: All teams and groups inherit this setting (can override at team level)

**Configuration Example**:
```
Organization Default Delivery: Email
    ↓ (inherits)
Team A: Email (default)
Team B: Slack (overridden)
Team C: Email (default)
User preferences: Can override team setting
```

---

## Manage Personal Notifications

### Personal Notification Configuration

**Access Path**:
```
User avatar (top right)
    ↓
User settings
    ↓
Notifications
    ↓
Personal notifications
```

**Personal Notification Categories**:

#### 1. Default Notifications
**Pre-configured subscriptions** relevant to individual users:
- Work items assigned to me
- Pull requests where I'm a reviewer
- Builds I queued or failed
- Code pushes to my branches

**Management**:
```
View all default notifications
├── Enable/Disable individual subscriptions
├── Change delivery channel (email, Slack, mobile)
└── Adjust notification frequency (immediate, daily digest)
```

---

#### 2. Custom Subscriptions
**User-created notification rules** for specific scenarios:

**Example 1**: High-Priority Bugs in My Area
```
Subscription: "My High-Priority Bugs"
├── Event: Work item created
├── Filters:
│   ├── Work item type: Bug
│   ├── Priority: High or Critical
│   ├── Area path: MyTeam/MyComponent
│   └── State: Active
└── Delivery: Email + Slack DM
```

**Example 2**: Main Branch Build Failures
```
Subscription: "Main Branch CI Failures"
├── Event: Build completed
├── Filters:
│   ├── Status: Failed
│   ├── Branch: refs/heads/main
│   ├── Definition: MyApp-CI
│   └── Requested for: Any
└── Delivery: Mobile push (high priority)
```

---

### Notification Frequency Options

**Immediate**: Notification sent within seconds of event
- Use for: Critical alerts, approval requests, production failures

**Daily Digest**: Summary email sent once per day (9:00 AM user's timezone)
- Use for: Non-urgent updates, work item changes, low-priority builds

**Weekly Digest**: Summary email sent weekly (Monday 9:00 AM)
- Use for: Long-term tracking, trend analysis, less critical updates

**Example Daily Digest**:
```
Your Azure DevOps Daily Digest - January 15, 2024

Work Items (3 updates):
├── Bug #4567 assigned to you by Jane Smith
├── Task #890 completed by John Doe
└── User Story #123 commented by Product Owner

Pull Requests (2 activities):
├── PR #45 approved by 2 reviewers (awaiting 1 more)
└── PR #46 merged to main by Jane Smith

Builds (5 completions):
├── MyApp-CI: 4 succeeded, 1 failed
└── MyApp-Release: 1 succeeded

[View All Activity] [Update Preferences]
```

---

## Notification Filtering Best Practices

### Avoid Notification Overload

**Problem**: Too many notifications = ignored notifications

**Solutions**:

#### 1. Filter by Relevance
```
❌ Bad: Notify on ALL work item changes (500 notifications/day)
✅ Good: Notify on work items assigned to ME (5 notifications/day)
```

#### 2. Filter by Criticality
```
❌ Bad: Notify on every build (50 builds/day)
✅ Good: Notify on failed builds only (5 failures/day)
```

#### 3. Filter by Environment
```
❌ Bad: Notify on all deployments (Dev, Test, Staging, Prod = 100/day)
✅ Good: Notify on Production deployments only (5/day)
```

#### 4. Use Appropriate Delivery Channel
```
❌ Bad: Email for all alerts (inbox flooded)
✅ Good: 
    - Email: Daily digests, non-urgent updates
    - Slack: Real-time alerts, team coordination
    - Mobile push: Critical production issues only
```

---

## Quick Reference

### Notification Scope Hierarchy

```
Organization (global)
    ↓ inherits ↓
Project (project-wide)
    ↓ inherits ↓
Team (team-specific)
    ↓ overrides ↓
Personal (user-specific)
```

**Precedence**: Personal overrides Team overrides Project overrides Organization

---

### Common Notification Patterns

| Scenario | Scope | Event | Filters | Delivery |
|----------|-------|-------|---------|----------|
| **My failed builds** | Personal | Build completed | Status=Failed, RequestedBy=@Me | Email |
| **Team PR reviews** | Team | PR created | TargetBranch=main, Repo=TeamRepo | Slack |
| **Production deploys** | Project | Release deployment | Environment=Production | Email + Slack |
| **Security alerts** | Global | Security scan | Severity=High/Critical | PagerDuty |

---

## Key Takeaways

- 🔔 **Four scopes**: Personal, Team, Project, Global (hierarchical inheritance)
- 🎯 **Rule-based subscriptions**: Define custom event filters for targeted notifications
- 📧 **Multiple delivery channels**: Email, Slack, Teams, mobile push, webhooks
- 🚫 **Prevent alert fatigue**: Filter by relevance, criticality, and environment
- ⚙️ **Frequency options**: Immediate, daily digest, weekly digest
- 👥 **Permission-based**: Project Collection Admins manage global/project subscriptions

---

## Next Steps

✅ **Completed**: Azure DevOps notification configuration across all scopes

**Continue to**: Unit 6 - Configure GitHub notifications (repository watching, conversation subscriptions)

---

## Additional Resources

- [Get started with notifications in Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/organizations/notifications/about-notifications)
- [Manage notifications for a team, project, organization](https://learn.microsoft.com/en-us/azure/devops/organizations/notifications/manage-team-group-global-organization-notifications)
- [Events, subscriptions, and notifications](https://learn.microsoft.com/en-us/azure/devops/organizations/notifications/about-notifications)
- [Manage your personal notifications](https://learn.microsoft.com/en-us/azure/devops/notifications/manage-your-personal-notifications)

[↩️ Back to Module Overview](01-introduction.md) | [⬅️ Previous: Service Hooks](04-explore-service-hooks.md) | [➡️ Next: Configure GitHub Notifications](06-configure-github-notifications.md)
