# Design and Implement Metrics and Queries

## Key Concepts
- **Project metrics**: Track development lifecycle (velocity, cycle time, lead time)
- **Platform comparison**: GitHub vs. Azure DevOps capabilities
- **Integration**: Azure Monitor + Application Insights for DevOps workflows
- **Automation**: Release annotations, quality gates, work item creation

## Why Project Metrics Matter
✅ **Enhanced visibility**: Real-time status of projects and teams
✅ **Performance monitoring**: Velocity, cycle time, throughput
✅ **Process optimization**: Identify bottlenecks, data-driven decisions
✅ **Risk management**: Proactive issue identification

## Platform Comparison

### GitHub Tracking
| Feature | Capability | Limitation |
|---------|-----------|------------|
| Insights | Repository activity, PR performance | Limited project mgmt metrics |
| Pulse View | Activity summary | Basic visualizations |
| Contributors | Code contributions over time | No built-in Azure Monitor integration |
| Traffic | Views, clones, referrers | Fewer custom reporting options |

### Azure DevOps Tracking
| Feature | Capability | Advantage |
|---------|-----------|-----------|
| Analytics | Comprehensive project monitoring | Purpose-built for agile |
| Dashboards | Customizable drag-and-drop widgets | Extensive widget library |
| Reporting | Power BI integration, OData feeds | Advanced query capabilities |
| Metrics | Velocity, burndown, cycle time, lead time | Deep Azure integration |

## Azure DevOps Metrics

### Work Item Metrics
```kusto
// Example: Track work item velocity
WorkItems
| where WorkItemType == "User Story"
| where State == "Done"
| summarize Velocity = count() by Sprint, Team
| render columnchart
```

**Key Metrics**:
- Count of work items (created/completed/in progress)
- Work item age (identify overdue tasks)
- Velocity (rate of completion per sprint)
- Backlog growth (new items vs. completed)
- Distribution by type (bugs, features, tasks)

### Build and Release Metrics
| Metric | Purpose | Target |
|--------|---------|--------|
| Build success rate | Percentage of successful builds | >95% |
| Release deployment frequency | How often deploying to prod | Daily/weekly (per team) |
| Deployment success rate | Successful deployment percentage | >98% |
| Build duration | Time to complete build | <10 minutes |
| Deployment lead time | Commit to production | <1 hour |

### Test Metrics
- **Pass rate**: % of tests passing
- **Execution time**: Duration of test runs
- **Failure trends**: Recurring test failures
- **Test coverage**: % of code covered
- **Flaky test detection**: Unreliable tests

### Team Performance Metrics
```kusto
// Example: Calculate cycle time
WorkItems
| where State == "Done"
| extend CycleTime = CompletedDate - ActivatedDate
| summarize AvgCycleTime = avg(CycleTime) by WorkItemType
| order by AvgCycleTime desc
```

**Key Metrics**:
- Sprint burndown
- Team velocity (average work completed per sprint)
- Lead time (creation to completion)
- Cycle time (start work to done)
- Throughput (items completed per period)

## Common Analytical Queries

### 1. Cycle Time Analysis
```kusto
WorkItems
| where State in ("Done", "Closed")
| extend CycleTime = datetime_diff('day', ClosedDate, ActivatedDate)
| summarize 
    AvgCycleTime = avg(CycleTime),
    P50 = percentile(CycleTime, 50),
    P95 = percentile(CycleTime, 95)
    by WorkItemType
```

### 2. Lead Time Distribution
```kusto
WorkItems
| extend LeadTime = datetime_diff('day', ClosedDate, CreatedDate)
| summarize 
    Count = count(),
    AvgLeadTime = avg(LeadTime)
    by AreaPath
| order by AvgLeadTime desc
```

### 3. Cumulative Flow Diagram (CFD)
```kusto
WorkItems
| where ChangedDate >= ago(30d)
| summarize count() by State, bin(ChangedDate, 1d)
| render areachart
```

### 4. Work Item Aging Analysis
```kusto
WorkItems
| where State in ("Active", "New")
| extend Age = datetime_diff('day', now(), CreatedDate)
| summarize 
    TotalItems = count(),
    AvgAge = avg(Age),
    OldestItem = max(Age)
    by Priority, WorkItemType
| order by Priority asc, AvgAge desc
```

## Azure Monitor Integration

### Continuous Monitoring in Pipelines
```yaml
# Quality gate example
gates:
- task: QueryAzureMonitor@1
  inputs:
    azureSubscription: 'Azure-Subscription'
    resourceGroupName: 'rg-prod'
    resourceType: 'Microsoft.Insights/components'
    resourceName: 'app-insights-prod'
    metricName: 'requests/failed'
    operator: 'GreaterThan'
    threshold: 10
    aggregation: 'count'
    timeRange: '00:15:00'  # Last 15 minutes
```

**Quality Gate Logic**:
1. Pipeline deploys to staging
2. Post-deployment gate monitors App Insights (15 min)
3. If error rate >1% → Block deployment
4. If metrics healthy → Auto-proceed to production

### Release Annotations
```powershell
# Create release annotation
$releaseProperties = @{
    "ReleaseDescription" = "v2.3.1 - New payment gateway"
    "TriggerBy" = $env:BUILD_REQUESTEDFOR
    "BuildNumber" = $env:BUILD_BUILDNUMBER
}

New-AzApplicationInsightsAnnotation `
    -ResourceGroupName "rg-prod" `
    -Name "app-insights-prod" `
    -Properties $releaseProperties `
    -EventTime (Get-Date).ToUniversalTime()
```

**Benefits**:
- See deployment markers on performance charts
- Correlate performance changes with releases
- Identify problematic deployments immediately
- Link to build/release details

### Work Items from Alerts
```json
{
  "alert": {
    "condition": "requests/failed > 100",
    "actions": [
      {
        "actionType": "CreateWorkItem",
        "workItemType": "Bug",
        "project": "MyProject",
        "title": "High error rate in ProductService",
        "description": "{{kqlQuery}}",
        "assignedTo": "oncall-team@company.com",
        "priority": 1,
        "tags": ["production", "auto-generated"]
      }
    ]
  }
}
```

**Automation Flow**:
1. App Insights detects elevated exception rate
2. Alert fires automatically
3. Work item created with:
   - KQL query results
   - Affected users count
   - Stack traces
   - Deployment annotation link
4. On-call engineer notified

## GitHub Tracking Options

### GitHub Actions for Metrics
```yaml
name: Collect Metrics
on:
  schedule:
    - cron: '0 0 * * *'  # Daily
jobs:
  metrics:
    runs-on: ubuntu-latest
    steps:
      - name: Collect PR Metrics
        run: |
          gh api repos/${{ github.repository }}/pulls \
            --jq '.[] | {number, created_at, merged_at}'
      
      - name: Collect Issue Metrics
        run: |
          gh api repos/${{ github.repository }}/issues \
            --jq '.[] | {number, created_at, closed_at, labels}'
```

### GitHub REST API
```javascript
// Fetch PR metrics
const response = await fetch(
  'https://api.github.com/repos/owner/repo/pulls?state=all',
  { headers: { 'Authorization': `token ${GITHUB_TOKEN}` }}
);

const prs = await response.json();
const avgTimeToMerge = prs
  .filter(pr => pr.merged_at)
  .map(pr => new Date(pr.merged_at) - new Date(pr.created_at))
  .reduce((a, b) => a + b, 0) / prs.length;
```

## Critical Notes
- ⚠️ **Azure DevOps**: Native integration with Azure Monitor (better than GitHub)
- 💡 **Release annotations**: Essential for correlating deployments with performance
- 🎯 **Quality gates**: Automate deployment decisions based on telemetry
- 📊 **Work item automation**: Reduce manual effort, faster incident response
- 🔄 **Custom dashboards**: Combine project metrics with application telemetry

## Quick Reference: Key Metrics

| Metric | Formula | Target |
|--------|---------|--------|
| **Cycle Time** | ActivatedDate → CompletedDate | <7 days |
| **Lead Time** | CreatedDate → CompletedDate | <14 days |
| **Velocity** | Story points/sprint | Stable trend |
| **Deployment Frequency** | Deployments/week | Daily (elite teams) |
| **Change Fail Rate** | Failed changes / total changes | <15% |
| **MTTR** | Time to resolve incidents | <1 hour |

## Integration Benefits
✅ Unified visibility: Project + application metrics
✅ Automated workflows: Alerts → Work items
✅ Deployment correlation: Performance ↔ Releases
✅ Data-driven decisions: Hard evidence for retrospectives
✅ Proactive management: Detect issues early

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-tools-track-usage-flow/8-design-implement-metrics-queries)
