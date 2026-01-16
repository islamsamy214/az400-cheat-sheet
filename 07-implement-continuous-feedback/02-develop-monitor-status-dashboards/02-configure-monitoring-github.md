# Configure Monitoring in GitHub

## Key Concepts
- **GitHub Insights** provides built-in analytics for projects with current and historical charts
- **Current charts** show real-time work distribution, team workload, and status breakdowns
- **Historical charts** track velocity, trends, and completion patterns over time (burnup)
- **GitHub Actions monitoring** tracks workflow execution, success rates, and duration
- **Status badges** display build/test status directly in README files
- **Chart creation limits**: Free accounts get 2 charts per private project, unlimited for public projects

## GitHub Monitoring Categories

| Category | Insights Provided | Use Case |
|----------|-------------------|----------|
| **Repository Insights** | Traffic, contributors, network graphs, community health | Understand repo activity and engagement |
| **GitHub Actions Analytics** | Workflow metrics, job performance, runner utilization | Monitor CI/CD pipeline health |
| **Security Monitoring** | Dependabot alerts, code scanning, secret scanning | Track vulnerabilities and security posture |
| **Project Insights** | Work item distribution, burndown, velocity charts | Track team progress and productivity |

## GitHub Insights Charts

### Current Charts (Real-Time Snapshots)

**Default Status Chart**:
- Shows items by status (To Do, In Progress, Done)
- Instant snapshot of current work distribution

**Common Use Cases**:
```yaml
Team Workload:
  X-axis: Assignees
  Y-axis: Count of items
  Benefit: Identify workload imbalances

Iteration Progress:
  X-axis: Milestone
  Y-axis: Count of items
  Group by: Status
  Benefit: Track sprint progress

Label Categorization:
  X-axis: Labels (bug, feature, docs)
  Y-axis: Count of items
  Benefit: Understand work composition
```

### Historical Charts (Trend Analysis)

**Default Burnup Chart**:
- Shows total scope + completed work over time
- Reveals scope changes vs. actual progress
- Better than burndown for transparency

**Common Use Cases**:
```yaml
Velocity Tracking:
  X-axis: Time (weekly)
  Y-axis: Count of items
  Filter: Status = "Done"
  Insight: Measure sustainable pace

Issue Closure Rate:
  X-axis: Time
  Y-axis: Count of items
  Filter: Type = "Issue" AND Status = "Closed"
  Insight: Measure responsiveness

PR Merge Patterns:
  X-axis: Time
  Y-axis: Count of items
  Filter: Type = "Pull Request" AND Status = "Merged"
  Insight: Understand code review throughput
```

## Creating Custom Charts

**Step-by-Step Process**:
1. Navigate to Projects → Insights icon
2. Select **New chart** from left menu
3. Name chart descriptively (e.g., "Frontend Team Workload Q1")
4. Configure chart settings:
   - **Layout**: Bar, column, line, stacked area, stacked bar, stacked column
   - **X-axis**: Assignees, Labels, Milestone, Repository, Status, Time
   - **Group by** (optional): Secondary dimension for breaking down data
   - **Y-axis**: Count, Sum, Average, Min, Max of fields
5. Apply filters (e.g., `status:"In Progress"`, `label:"bug"`)
6. Save and share

## Chart Configuration Options

| Configuration | Options | Best For |
|---------------|---------|----------|
| **Layout** | Bar, Column, Line, Stacked Area/Bar/Column | Depends on data type and comparison needs |
| **X-axis** | Assignees, Labels, Milestone, Repository, Status, Time | Primary dimension for grouping |
| **Group by** | None, Assignees, Labels, Milestone, Repository | Secondary breakdown within primary |
| **Y-axis** | Count, Sum, Average, Min, Max | How to aggregate data |

## GitHub Actions Monitoring

**Workflow-Level Metrics**:
- **Success rate**: Percentage of successful workflow runs
- **Average duration**: Mean execution time
- **Failure patterns**: Identify which tasks/jobs fail most
- **Trigger analysis**: Which events cause most runs

**Accessing Actions Analytics**:
1. Navigate to repository → Actions tab
2. Select specific workflow
3. Select **Insights** (if available for your plan)

**Example Insights**:
- Identify slow workflows → Optimize bottleneck jobs
- Track failure patterns → Fix flaky tests, improve error handling
- Optimize runner costs → Monitor minutes consumed, add caching

## Status Badges

**Creating Workflow Status Badges**:
1. Go to repository → Actions tab
2. Select target workflow
3. Click **...** (more options) → **Create status badge**
4. Copy Markdown/HTML code
5. Paste in README.md

**Example Badge Markdown**:
```markdown
# My Project

![CI](https://github.com/owner/repo/actions/workflows/ci.yml/badge.svg)
![Tests](https://github.com/owner/repo/actions/workflows/tests.yml/badge.svg)
![Coverage](https://img.shields.io/codecov/c/github/owner/repo)
```

**Common Badge Types**:
- **Workflow status**: Pass/fail status of CI/CD workflows
- **Test coverage**: Code coverage percentage
- **Dependencies**: Dependency update status
- **License**: Project license type

## Plan Availability

| Plan | Historical Charts | Chart Limit | Notes |
|------|-------------------|-------------|-------|
| **GitHub Free** | ❌ Private projects<br/>✅ Public projects | 2 charts (private) | Unlimited for public |
| **GitHub Pro** | ✅ Individual users | Unlimited | For private projects |
| **GitHub Team** | ✅ Organizations | Unlimited | For organization accounts |
| **GitHub Enterprise Cloud** | ✅ Organizations & Enterprise | Unlimited | Advanced features |

## Best Practices

**Chart Creation**:
- ✅ Use descriptive names: "Frontend Team Workload Q1" vs. "Team Chart"
- ✅ Create role-specific dashboards: Developer, Manager, Product Owner views
- ✅ Start simple, iterate based on feedback
- ✅ Use consistent filtering across related charts
- ✅ Document chart purposes in team wiki

**Chart Maintenance**:
- Review and retire outdated charts periodically
- Free up chart slots (GitHub Free) by removing unused visualizations
- Update filters as team structure or workflows change

## Critical Notes
- ⚠️ **Historical charts limited**: Not available for private projects on GitHub Free
- 💡 **Archived items excluded**: Historical charts don't track archived/deleted items
- 🎯 **Use "Closed as Not Planned"**: Create custom "Canceled" status before archiving for tracking
- 📊 **Public projects unlimited**: All features available regardless of plan tier

## Example Filter Expressions

```yaml
# Single Condition
status:"In Progress"
assignee:@username
label:"bug"

# Multiple Conditions (AND)
status:"Done" label:"feature"
assignee:@username milestone:"Sprint 5"

# Multiple Conditions (OR)
status:"To Do" OR status:"In Progress"
label:"bug" OR label:"security"

# Exclusion (NOT)
-status:"Done"
-assignee:@username

# Date-Based
updated:>2024-01-01
created:<2024-06-30
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/2-configure-monitoring-github)
