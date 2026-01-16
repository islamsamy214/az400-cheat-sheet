# Integrate GitHub and Azure DevOps with Microsoft Teams

## Key Concepts
- **Reduce context switching**: Bring DevOps notifications into collaboration hub (Teams)
- **Increase visibility**: Real-time updates on PRs, builds, deployments, work items
- **Accelerate responses**: Act on notifications directly from Teams
- **GitHub + Teams**: Subscribe to repos, PRs, issues, workflows, deployments
- **Azure Boards + Teams**: Monitor work items, create items, view Kanban boards
- **Azure Repos + Teams**: Track PRs, commits, repository changes
- **Azure Pipelines + Teams**: Monitor builds/releases, approve deployments

## Why Integrate DevOps Tools with Teams?

| Benefit | Description | Example |
|---------|-------------|---------|
| **Reduce Context Switching** | Stay in Teams instead of jumping between tools | Approve deployment from Teams chat vs. opening Azure DevOps |
| **Increase Visibility** | Real-time notifications for team-relevant events | PR created → immediate Teams notification → faster review |
| **Accelerate Responses** | Act on events quickly from familiar interface | Merge PR directly from Teams notification |
| **Centralize Communication** | Discuss DevOps events in context with team | Pipeline failure → threaded Teams conversation |
| **Improve Collaboration** | Team sees same information simultaneously | Deployment notification → coordinated team response |

## GitHub + Microsoft Teams Integration

### Setup
**Install GitHub App**:
1. Open Microsoft Teams
2. Click **Apps** in left sidebar
3. Search for **GitHub**
4. Click **Add** to install app
5. Configure GitHub authentication

### Subscribe to Repository Events
```bash
# In Teams channel, use GitHub app commands

# Subscribe to repository (all events)
@github subscribe owner/repo

# Subscribe to specific events
@github subscribe owner/repo pulls,issues,commits

# Subscribe with filters
@github subscribe owner/repo pulls:opened,merged
@github subscribe owner/repo issues:opened+label:bug
```

### Supported Event Types

| Event Type | Trigger | Teams Notification |
|------------|---------|-------------------|
| **pulls** | PR opened/updated/merged/closed | PR details, author, reviewers, status |
| **issues** | Issue opened/edited/closed | Issue title, description, labels, assignee |
| **commits** | Push to repository | Commit SHA, author, message, changed files |
| **releases** | Release published | Release version, notes, assets |
| **deployments** | Deployment status changed | Environment, status, commit, actor |
| **workflows** | GitHub Actions workflow run | Workflow name, status, branch, logs link |

### Subscription Filters

**Pull Requests**:
```bash
# Only opened and merged PRs
@github subscribe owner/repo pulls:opened,merged

# PRs to specific branch
@github subscribe owner/repo pulls+branch:main

# PRs with specific label
@github subscribe owner/repo pulls+label:priority
```

**Issues**:
```bash
# Only issues with specific label
@github subscribe owner/repo issues+label:bug

# Issues assigned to specific user
@github subscribe owner/repo issues+assignee:username
```

**Workflows**:
```bash
# Specific workflow runs
@github subscribe owner/repo workflows:{name:"CI Pipeline"}

# Failed workflows only
@github subscribe owner/repo workflows:failed
```

### GitHub Notifications in Teams

**Pull Request Card**:
- PR title and number
- Author and creation date
- Description (truncated)
- Review status (approved/changes requested)
- CI status (checks passing/failing)
- **Actions**: View in GitHub, Add comment, Approve, Merge

**Workflow Run Card**:
- Workflow name
- Trigger (push, PR, manual)
- Branch/commit
- Status (success, failure, in progress)
- **Actions**: View logs, Re-run workflow

**Deployment Card**:
- Environment (production, staging, etc.)
- Deployment status
- Commit and actor
- **Actions**: View deployment, Approve (if required)

### Link Previews
```
# Paste GitHub URL in Teams
https://github.com/owner/repo/pull/123

# Teams automatically shows rich preview
- PR title, author, status
- Description preview
- Review and CI status
- Quick actions
```

### PR Reminders
```bash
# Schedule reminders for open PRs needing review
@github subscribe owner/repo pulls reminders:daily

# Custom reminder schedule
@github subscribe owner/repo pulls reminders:weekly
```

## Azure Boards + Microsoft Teams Integration

### Setup
**Install Azure Boards App**:
1. Open Microsoft Teams
2. Click **Apps** in left sidebar
3. Search for **Azure Boards**
4. Click **Add** to install
5. Authenticate with Azure DevOps

### Subscribe to Work Item Events
```bash
# In Teams channel, use Azure Boards commands

# Subscribe to area path (all work items)
@azure boards subscribe https://dev.azure.com/org/project/_workitems

# Subscribe to specific area path
@azure boards subscribe https://dev.azure.com/org/project/_workitems area="TeamA"

# Subscribe to specific work item types
@azure boards subscribe https://dev.azure.com/org/project/_workitems type="Bug"
```

### Supported Work Item Events

| Event Type | Trigger | Teams Notification |
|------------|---------|-------------------|
| **Created** | New work item created | Work item type, title, assigned to, area |
| **Updated** | Work item field changed | Changed fields, old/new values, changed by |
| **State Changed** | State transition | Previous state → new state, work item details |
| **Assigned** | Assignee changed | New assignee, work item details |
| **Commented** | Comment added | Comment text, author, work item link |

### Subscription Filters

```bash
# Bugs only
@azure boards subscribe https://dev.azure.com/org/project/_workitems type="Bug"

# Specific area path
@azure boards subscribe https://dev.azure.com/org/project/_workitems area="MyTeam\Sprint1"

# High priority items
@azure boards subscribe https://dev.azure.com/org/project/_workitems priority="1"

# Specific state changes
@azure boards subscribe https://dev.azure.com/org/project/_workitems state="Active"
```

### Create Work Items from Teams
**Using Compose Extension**:
1. In Teams message box, click **...** (More options)
2. Search for **Azure Boards**
3. Select **Create work item**
4. Fill in work item form (title, type, description)
5. Submit to Azure DevOps

**Using @mention**:
```bash
# Create work item from Teams
@azure boards create

# Interactive form appears in Teams
```

### Kanban Board in Teams Tab
**Add Board as Tab**:
1. In Teams channel, click **+** to add tab
2. Search for **Azure Boards**
3. Select your project and board
4. Board appears as Teams tab (view-only or interactive)

**Benefits**:
- View sprint board without leaving Teams
- Drag-and-drop work items (if permissions allow)
- See real-time board updates
- Discuss board items in channel

## Azure Repos + Microsoft Teams Integration

### Setup
**Install Azure Repos App**:
1. Open Microsoft Teams
2. Click **Apps** → Search **Azure Repos**
3. Click **Add** and authenticate

### Subscribe to Repository Events
```bash
# Subscribe to repository
@azure repos subscribe https://dev.azure.com/org/project/_git/repo

# Subscribe to specific events
@azure repos subscribe https://dev.azure.com/org/project/_git/repo pullrequest,commit
```

### Supported Repository Events

| Event Type | Trigger | Teams Notification |
|------------|---------|-------------------|
| **Pull Request Created** | New PR opened | PR title, author, source/target branches, reviewers |
| **Pull Request Updated** | PR updated or commits pushed | Updated details, new commits |
| **Pull Request Merged** | PR completed | Merged by, commit SHA, target branch |
| **Pull Request Commented** | Comment on PR | Comment text, author, file/line context |
| **Code Pushed** | Commits pushed to branch | Commit SHAs, messages, author, changed files |

### Subscription Management
```bash
# List current subscriptions
@azure repos subscriptions

# Remove subscription
@azure repos unsubscribe <subscription-id>

# Update subscription filters
@azure repos subscribe https://dev.azure.com/org/project/_git/repo pullrequest+branch:main
```

### PR Notifications in Teams

**Pull Request Card**:
- PR title and ID
- Author, created date
- Source branch → Target branch
- Reviewers and approval status
- Build validation status
- **Actions**: View PR, Approve, Add comment

**Threaded Conversations**:
- PR notifications support Teams threading
- Team discussions stay organized under PR notification
- Conversations preserved even after PR merged

### Compose Extension
**Search and Share PRs**:
1. In Teams message, click **...** → **Azure Repos**
2. Search for PR by title, ID, or author
3. Select PR to share rich card in conversation
4. Recipients see PR details and can act

## Azure Pipelines + Microsoft Teams Integration

### Setup
**Install Azure Pipelines App**:
1. Open Microsoft Teams
2. Click **Apps** → Search **Azure Pipelines**
3. Click **Add** and authenticate

### Subscribe to Pipeline Events
```bash
# Subscribe to all pipelines in project
@azure pipelines subscribe https://dev.azure.com/org/project

# Subscribe to specific pipeline
@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123

# Subscribe with filters
@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123 runStage:failure
```

### Supported Pipeline Events

| Event Type | Trigger | Teams Notification |
|------------|---------|-------------------|
| **Run Started** | Pipeline run initiated | Pipeline name, triggered by, branch, commit |
| **Run Completed** | Pipeline finished | Status (succeeded/failed), duration, artifacts |
| **Run Stage Started** | Stage execution started | Stage name, pipeline, run ID |
| **Run Stage Completed** | Stage execution finished | Stage name, status, logs link |
| **Approval Pending** | Manual approval required | Environment, pipeline, approver, timeout |
| **Deployment Completed** | Release to environment finished | Environment, status, deployed by, commit |

### Event Filters

```bash
# Only failed runs
@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123 runStage:failure

# Specific branch
@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123 branch:main

# Specific stage
@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123 stageName:"Deploy to Production"

# Multiple filters
@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123 branch:main runStage:failure
```

### Approve Deployments from Teams
**Approval Card in Teams**:
- Pipeline requires manual approval
- Approval request sent to Teams
- Card shows: Pipeline, environment, commit, timeout
- **Actions**: Approve, Reject, Reassign

**Approval Process**:
1. Deployment reaches approval gate
2. Teams notification sent to approvers
3. Approver clicks **Approve** or **Reject** in Teams
4. Optional: Add comment explaining decision
5. Pipeline continues or stops based on decision

### Pipeline Notifications

**Run Completed Card**:
- Pipeline name and run number
- Trigger (manual, CI, scheduled)
- Branch and commit
- Duration and status
- Failed tasks (if any)
- **Actions**: View logs, Download artifacts, Rerun pipeline

**Approval Required Card**:
- Environment name (e.g., "Production")
- Pipeline and run ID
- Commit being deployed
- Timeout countdown
- **Actions**: Approve, Reject, Reassign, View run

### Compose Extension
**Trigger Pipeline from Teams**:
1. In Teams message, click **...** → **Azure Pipelines**
2. Select **Run pipeline**
3. Choose pipeline and branch
4. Optional: Set variables
5. Pipeline starts, notification sent to channel

## Integration Best Practices

### 1. Start with Focused Subscriptions
```yaml
❌ Don't:
  - Subscribe to all events from all repositories
  - Create notifications for every commit
  - Flood channels with noise

✅ Do:
  - Subscribe to high-value events (PR reviews, build failures)
  - Start narrow, expand based on team feedback
  - Use filters to reduce notification volume
```

### 2. Use Filters Strategically
```yaml
Effective Filters:
  GitHub:
    - "pulls:opened,merged+branch:main" (only main branch PRs)
    - "issues+label:bug+label:priority" (critical bugs only)
    - "workflows:failed" (only failures)
  
  Azure Boards:
    - type="Bug" area="MyTeam" (team-specific bugs)
    - priority="1" OR priority="2" (high-priority items)
  
  Azure Pipelines:
    - runStage:failure branch:main (main branch failures only)
    - stageName:"Production" (production deployments)
```

### 3. Organize with Dedicated Channels
```yaml
Channel Strategy:
  #dev-prs:
    - PR opened/merged notifications
    - Code review discussions
  
  #dev-builds:
    - Build/deployment status
    - Pipeline failure alerts
  
  #project-planning:
    - Work item updates
    - Sprint planning discussions
  
  #prod-deployments:
    - Production deployment notifications
    - Approval requests
    - Incident alerts
```

### 4. Configure Threading
```yaml
Threading Benefits:
  - Keeps related discussions together
  - Reduces channel clutter
  - Maintains conversation context
  
Enable Threading:
  - Teams supports threaded replies by default
  - Reply to notification card (not new message)
  - Conversations stay organized under original event
```

### 5. Establish Team Conventions
```yaml
Notification Response Conventions:
  Pull Requests:
    - 👀 emoji = "I'm reviewing this"
    - ✅ emoji = "Looks good to me"
    - 💬 emoji = "I left comments"
  
  Build Failures:
    - 🔍 emoji = "I'm investigating"
    - 🔧 emoji = "Fix in progress"
    - ✅ emoji = "Resolved"
  
  Approvals:
    - Respond in thread with approval reasoning
    - Tag relevant team members if needed
    - Use "Approve" button (not just emoji)
```

### 6. Review and Refine Regularly
```yaml
Quarterly Review:
  - Are notifications still valuable?
  - Is anyone ignoring channels due to noise?
  - Are critical events being missed?
  - Should filters be adjusted?
  
Adjust Based On:
  - Team feedback
  - Notification volume
  - Response times
  - Channel activity patterns
```

## Critical Notes
- ⚠️ **Too many notifications = ignored notifications**: Start narrow, expand carefully
- 💡 **Use filters**: Reduce noise, increase signal
- 🎯 **Dedicated channels**: Organize by purpose (PRs, builds, work items)
- 📊 **Threading**: Keep conversations organized under events
- 🔗 **Direct actions**: Approve, merge, create from Teams (no context switch)

## Quick Commands Cheat Sheet

**GitHub in Teams**:
```bash
# Subscribe to repository
@github subscribe owner/repo

# Subscribe with filters
@github subscribe owner/repo pulls:opened,merged+branch:main

# List subscriptions
@github subscriptions

# Unsubscribe
@github unsubscribe owner/repo

# PR reminders
@github subscribe owner/repo pulls reminders:daily
```

**Azure Boards in Teams**:
```bash
# Subscribe to work items
@azure boards subscribe https://dev.azure.com/org/project/_workitems

# Filter by type
@azure boards subscribe https://dev.azure.com/org/project/_workitems type="Bug"

# Create work item
@azure boards create

# List subscriptions
@azure boards subscriptions
```

**Azure Repos in Teams**:
```bash
# Subscribe to repository
@azure repos subscribe https://dev.azure.com/org/project/_git/repo

# Subscribe to PRs only
@azure repos subscribe https://dev.azure.com/org/project/_git/repo pullrequest

# List subscriptions
@azure repos subscriptions
```

**Azure Pipelines in Teams**:
```bash
# Subscribe to pipeline
@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123

# Subscribe to failures only
@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123 runStage:failure

# List subscriptions
@azure pipelines subscriptions
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/share-knowledge-within-teams/4-integrate-github-azure-devops-microsoft-teams)
