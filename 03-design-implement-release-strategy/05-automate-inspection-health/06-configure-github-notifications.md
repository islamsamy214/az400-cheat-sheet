# Configure GitHub Notifications

⏱️ **Duration**: ~2 minutes | 📚 **Type**: Configuration Guide

## Overview

GitHub notifications deliver **real-time updates** for subscribed activities through a centralized inbox system that supports customization, triage workflows, and comprehensive update management. Master GitHub's notification system to stay informed without being overwhelmed.

---

## GitHub Notification System

GitHub provides granular control over communication preferences through:
- **Conversation-specific monitoring**: Individual issues, PRs, gists
- **Repository-wide surveillance**: Complete activity monitoring
- **CI/CD workflow integration**: GitHub Actions status updates  
- **Repository lifecycle events**: Issues, PRs, releases, security alerts

---

## Notification Subscription Options

### Conversation-Specific Monitoring
**Targeted alerts** for individual discussions:
- Issue #123 updates (comments, status changes, assignments)
- Pull request #456 activity (reviews, approvals, commits)
- Gist discussion threads

**Use Case**: Follow specific high-priority issues/PRs without noise from entire repository

---

### Repository-Wide Surveillance
**Comprehensive activity monitoring**:
- All issues created/updated
- All pull requests opened/merged
- All releases published
- Security advisories
- Team discussions

**Use Case**: Maintain awareness across all repository activity (best for maintainers)

---

### CI/CD Workflow Integration
**Automated status updates** for GitHub Actions:
- Workflow run started
- Workflow completed (success/failure)
- Required status checks passed/failed
- Deployment status updates

**Use Case**: Monitor build/deploy pipelines without manual checks

---

## Default Automatic Subscriptions

**You automatically watch** repositories you create and own by your personal account.

**You automatically subscribe** to conversations when you:
- ✅ Have NOT disabled automatic watching in notification settings
- ✅ Been assigned to an issue or pull request
- ✅ Opened a pull request, issue, or created a team discussion post
- ✅ Commented on a thread
- ✅ Subscribed to a thread manually by clicking **Watch** or **Subscribe**
- ✅ Had your username @mentioned
- ✅ Changed the thread's state (closing issue, merging PR)
- ✅ Had a team you're a member of @mentioned

---

## Manage Subscriptions

### Unsubscribe from Conversations

**Option 1**: Change notification settings (global preference)
```
Settings → Notifications → Automatic watching
└── Uncheck: "Automatically watch repositories"
```

**Option 2**: Unsubscribe directly on GitHub.com
```
Issue/PR page → Sidebar → Notifications
└── Click: "Unsubscribe" or "Unwatch"
```

**Tip**: Use selective watching instead of automatic subscriptions

---

## Notification Configuration Management

Access notification configuration through **GitHub.com interface**:

### Navigation Path
```
Profile photo (top right)
    ↓
Settings
    ↓
Notifications
```

---

## Comprehensive Notification Management Capabilities

### 1. Multi-Channel Delivery

**Configure delivery preferences**:

| Channel | Use Case | Configuration |
|---------|----------|---------------|
| **Web Interface** | In-app notifications | Always enabled (default) |
| **Email Notifications** | Detailed updates with links | Enable/disable, set frequency |
| **GitHub Mobile** | On-the-go awareness | Download GitHub Mobile app |

**Example Email Configuration**:
```
Participating and @mentions: ✅ Email
Watching: ❌ No email (too many)
CI activity: ✅ Email (build failures only)
Dependabot alerts: ✅ Email
```

---

### 2. Automated Subscription Management

**Control automatic subscriptions**:

#### Participating Conversations
```
✅ Automatically subscribe when:
    ├── I comment on an issue/PR
    ├── I'm assigned to an issue/PR
    └── I'm @mentioned in a discussion
```

#### Repository Watching
```
⚠️ Automatically watch:
    ├── Repositories I create
    ├── Repositories I'm granted admin access
    └── Repositories I fork (optional)

💡 Recommendation: Disable auto-watch, use selective watching
```

#### GitHub Actions Workflows
```
✅ Notify on workflow runs I triggered
❌ Don't notify on all workflow runs (too noisy)
```

---

### 3. Email Notification Optimization

**Fine-tune email delivery**:

#### Email Routing by Activity Type
```
Email Address: primary@example.com
├── Participating and @mentions → primary@example.com
├── Watching → work@example.com  
├── CI activity → devops@example.com
└── Security alerts → security@example.com
```

#### Frequency Settings
- **Real-time**: Immediate emails (< 1 minute latency)
- **Hourly digest**: Batch notifications (top of each hour)
- **Daily digest**: Single email per day (9:00 AM your timezone)
- **Weekly digest**: Single email per week (Monday 9:00 AM)

---

### 4. Event-Driven Trigger Configuration

**Specify which activities generate notifications**:

#### Issues
```
✅ Issue opened
✅ Issue assigned to me
✅ Issue mentioned me
❌ Issue labeled (too noisy)
❌ Issue milestoned (informational only)
```

#### Pull Requests
```
✅ PR review requested
✅ PR approved/changes requested
✅ PR merged
✅ PR mentioned me
❌ PR labeled (too noisy)
```

#### Actions
```
✅ Workflow run failed (my workflows)
❌ Workflow run succeeded (expected outcome)
❌ All workflow runs (too many)
```

---

### 5. Digest Email Automation

**Enable weekly summary emails** for watched repository activity:

**Configuration**:
```
Settings → Notifications → Weekly digest
└── ✅ Send weekly digest email for watched repositories
```

**Digest Email Example**:
```
Your GitHub Weekly Digest - Week of January 15, 2024

Repository: myorg/myapp (watching)
├── 12 new issues (5 assigned to you)
├── 8 pull requests merged
├── 3 releases published (v1.2.3, v1.2.4, v1.2.5)
└── 2 security advisories (1 high, 1 medium)

Repository: myorg/infrastructure (watching)
├── 4 new issues
├── 15 pull requests merged
└── 1 release published (v2.0.0)

[View All Activity] [Update Watch Settings]
```

**Benefit**: Stay informed without daily email flood

---

### 6. Account Integration Verification

**Review and manage email addresses** associated with your GitHub account:

**Purpose**: Ensure notifications delivered to correct email

**Configuration Path**:
```
Settings → Emails
```

**Verification Status**:
```
Primary Email: john.doe@company.com ✅ Verified
Backup Email: john.personal@gmail.com ✅ Verified
Work Email: j.doe@work.com ⚠️ Unverified (notifications blocked)
```

**Action Required**: Verify all email addresses to receive notifications

---

## Notification Inbox (Web Interface)

### Access Notification Inbox
```
GitHub.com → Bell icon (top right) → Notifications
```

### Inbox Features

#### Filters
```
Inbox (all unread)
├── Participating: Direct involvement (assigned, mentioned)
├── Mentions: @username references
├── Review requested: PR review needed
└── Done: Archived notifications
```

#### Triage Actions
- **Mark as done**: Archive notification
- **Save**: Pin important notifications
- **Unsubscribe**: Stop receiving updates for this thread
- **Mark as read**: Acknowledge without archiving

#### Bulk Actions
```
Select multiple notifications
├── Mark all as read
├── Mark all as done
└── Unsubscribe from all
```

---

## Notification Best Practices

### 1. Use Selective Watching

**❌ Bad Practice**: Watch all repositories (notification overload)
```
Watching: 50 repositories
Notifications/day: 200+
Result: Inbox overwhelm, ignore all notifications
```

**✅ Good Practice**: Watch only critical repositories
```
Watching: 5 critical repositories
Participating: All repositories (only when involved)
Notifications/day: 10-20 (actionable)
Result: Stay informed, respond quickly
```

---

### 2. Configure Email Filters

**Gmail Filter Example**:
```
Filter: GitHub notifications
├── From: notifications@github.com
├── To: john.doe@company.com
├── Action: Apply label "GitHub" + Archive
└── Result: Organized inbox, email doesn't clutter main inbox
```

**Outlook Rule Example**:
```
Rule: GitHub CI Failures
├── From: notifications@github.com
├── Subject contains: "[CI] Failed"
├── Action: Move to "GitHub Alerts" folder + High importance
└── Result: Critical failures highlighted, successes archived
```

---

### 3. Leverage GitHub Mobile App

**Benefits**:
- **Real-time push notifications**: < 10 seconds latency
- **Quick triage**: Mark as done, reply to comments
- **On-the-go awareness**: Respond to urgent PRs from anywhere

**Recommended Settings**:
```
Mobile Push Notifications:
✅ Participating and @mentions
✅ Review requested
✅ CI activity (failures only)
❌ Watching (too many, use web inbox)
```

---

## Quick Reference

### Notification Levels

| Level | Description | Email Volume |
|-------|-------------|--------------|
| **Participating** | You're directly involved | Low (5-10/day) |
| **Watching** | All activity in repository | High (50-100/day per repo) |
| **Ignoring** | No notifications (unsubscribed) | Zero |
| **Custom** | Specific event types only | Medium (configurable) |

### Common Notification Patterns

| Scenario | Configuration | Rationale |
|----------|---------------|-----------|
| **Core maintainer** | Watch repository | Need full awareness |
| **Contributor** | Participating only | Involved in specific issues/PRs |
| **Casual observer** | Weekly digest | Stay informed, low noise |
| **CI/CD engineer** | CI activity only | Monitor build/deploy pipelines |

---

## Key Takeaways

- 📬 **Centralized inbox**: All notifications in one place with filtering and triage
- 🎯 **Granular control**: Per-repository, per-event-type configuration
- 📧 **Multi-channel delivery**: Web, email, mobile (choose appropriate channels)
- 🔕 **Automatic subscriptions**: Configurable based on participation patterns
- 📊 **Weekly digests**: Stay informed without email overload
- 📱 **Mobile app**: Real-time push notifications for critical events

---

## Next Steps

✅ **Completed**: GitHub notification configuration and best practices

**Continue to**: Unit 7 - Explore how to measure quality of your release process (metrics and dashboards)

---

## Additional Resources

- [About notifications - GitHub Docs](https://docs.github.com/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/about-notifications)
- [Configuring notifications - GitHub Docs](https://docs.github.com/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/configuring-notifications)
- [Viewing your subscriptions - GitHub Docs](https://docs.github.com/account-and-profile/managing-subscriptions-and-notifications-on-github/managing-subscriptions-for-activity-on-github/viewing-your-subscriptions)
- [Triaging notifications from your inbox](https://docs.github.com/account-and-profile/managing-subscriptions-and-notifications-on-github/viewing-and-triaging-notifications/triaging-a-single-notification)

[↩️ Back to Module Overview](01-introduction.md) | [⬅️ Previous: Configure Azure DevOps Notifications](05-configure-azure-devops-notifications.md) | [➡️ Next: Measure Quality of Release Process](07-explore-how-measure-quality-release-process.md)
