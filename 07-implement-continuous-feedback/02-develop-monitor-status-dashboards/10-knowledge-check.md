# Knowledge Check

This unit typically contains interactive quiz questions in the Microsoft Learn platform. Here are the key concepts to remember for assessment:

## Key Takeaways for Assessment

### GitHub Monitoring
- **GitHub Insights** provides current charts (real-time) and historical charts (trends)
- **Historical charts** require GitHub Team/Enterprise for private projects (Free/Pro only for public)
- **Chart limits**: GitHub Free gets 2 charts per private project
- **Status badges** display workflow status in README files

### Azure Dashboards
- **Automatic refresh**: Metrics refresh every 5-30 minutes based on time range
- **RBAC roles**: Owner (full control), Contributor (edit only), Reader (view only)
- **Log limitations**: 30-day max for logs, shared dashboard required for log tiles
- **Best for**: Continuous operations monitoring, NOC displays, executive reporting

### Azure Monitor Workbooks
- **No automatic refresh**: Manual refresh required (designed for investigation)
- **Interactive parameters**: Time range, dropdowns, resource pickers enable dynamic filtering
- **Template ecosystem**: Microsoft, community, and organizational templates available
- **Best for**: Troubleshooting, ad-hoc analysis, reporting, incident investigations

### Power BI
- **Logs only**: No native real-time metrics support (must export metrics to logs)
- **Import model**: Must import data, no DirectQuery for most Azure sources
- **Refresh limits**: 8 refreshes/day (shared/Pro), unlimited (Premium)
- **Best for**: Executive dashboards, long-term trends, cross-domain integration

### Custom Applications
- **REST APIs**: Access Azure Monitor, Log Analytics, App Insights, Azure DevOps data
- **Authentication**: Microsoft Entra ID (recommended), service principals, API keys
- **When to build**: Unique requirements, customer-facing, embedded monitoring
- **When to avoid**: Standard tools sufficient, time constraints, limited resources

### Pipeline Health Metrics
- **Failure rate** = (Failed Runs / Total Runs) × 100
- **Duration** = Queue + Execution + Approval time
- **Flaky tests**: Tests with inconsistent pass/fail results without code changes
- **Quarantine**: Separate flaky tests, don't fail pipeline based on them

### Pipeline Optimization
- **Cost optimization**: Self-hosted agents, caching, conditional execution
- **Time optimization**: Parallelization, shallow clone, incremental builds
- **Performance optimization**: Right-size agents, task optimization, parallel tests
- **Reliability optimization**: Retry logic, quality gates, health checks

### Concurrency Optimization
- **Parallel jobs**: Run independent jobs simultaneously (same cost, faster feedback)
- **Agent sizing**: Small (2 vCPU), Medium (4 vCPU), Large (8+ vCPU)
- **Queue time target**: < 1 minute for interactive builds
- **Agent utilization target**: 60-80% (allows burst capacity)

## Common Assessment Topics

### Tool Selection Questions
```yaml
Q: When should you use Azure Dashboards vs. Workbooks?
A: Dashboards for continuous monitoring (auto-refresh), Workbooks for investigation (interactive)

Q: When is Power BI the right choice?
A: Executive reporting, long-term trends (months/years), cross-domain integration

Q: When should you build custom applications?
A: Unique requirements, customer-facing, advanced automation with clear ROI
```

### Configuration Questions
```yaml
Q: How to share Azure Dashboard with team?
A: Publish to shared resource group, assign RBAC roles (Owner/Contributor/Reader)

Q: How to enable historical charts in GitHub Projects?
A: Requires GitHub Team/Enterprise for private projects (included for public)

Q: How to schedule Power BI refresh?
A: Publish to Power BI Service → Dataset settings → Configure scheduled refresh
```

### Pipeline Health Questions
```yaml
Q: How to calculate pipeline failure rate?
A: (Failed Runs / Total Runs) × 100

Q: What are flaky tests?
A: Tests that produce inconsistent results (pass/fail) without code changes

Q: How to remediate flaky tests?
A: Quarantine (separate from stable tests), fix timing issues, improve isolation
```

### Optimization Questions
```yaml
Q: How to reduce pipeline cost?
A: Self-hosted agents, caching, conditional execution, incremental builds

Q: How to reduce pipeline duration?
A: Parallelization, shallow clone, parallel tests, build optimization

Q: How to improve pipeline reliability?
A: Retry logic, quality gates, health checks, monitor failure rates
```

## Quick Reference

| Tool | Auto-Refresh | Best For | Limitations |
|------|--------------|----------|-------------|
| **GitHub Insights** | ✅ Yes | Project analytics | Historical charts require Team/Enterprise |
| **Azure Dashboards** | ✅ Yes | Operations monitoring | 30-day logs, limited interactivity |
| **Azure Workbooks** | ❌ No | Investigation | No real-time monitoring |
| **Power BI** | ⚠️ Scheduled | Executive reporting | 8 refreshes/day (shared/Pro) |
| **Custom Apps** | ✅ Yes (custom) | Unique requirements | High development cost |

## Remember
- ⚠️ **Tool selection matters**: Match tool to use case (monitoring vs. investigation vs. reporting)
- 💡 **Start with standard tools**: Build custom only when necessary
- 🎯 **Monitor pipeline health**: Track failure rate, duration, flaky tests
- 📊 **Optimize across dimensions**: Balance cost, time, performance, reliability

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/10-knowledge-check)
