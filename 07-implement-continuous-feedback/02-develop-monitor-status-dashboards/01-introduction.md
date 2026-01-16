# Introduction

## Key Concepts
- **Monitoring dashboards** transform raw telemetry into actionable visual insights for operations teams
- **Multiple tools available**: GitHub monitoring, Azure Dashboards, Workbooks, Power BI, custom apps
- **Pipeline health tracking**: Monitor failure rate, duration, and flaky tests for CI/CD reliability
- **Pipeline optimization**: Balance cost, time, performance, and reliability for efficient delivery
- **Tool selection matters**: Each platform (GitHub, Azure Dashboards, Workbooks, Power BI) serves specific use cases

## Module Coverage

| Tool | Best For | Key Features |
|------|----------|--------------|
| **GitHub Monitoring** | Repository insights, Actions analytics | Insights charts, status badges, workflow metrics |
| **Azure Dashboards** | Operational monitoring, NOC displays | Auto-refresh, tile-based, RBAC, full-screen mode |
| **Azure Monitor Workbooks** | Investigation, troubleshooting, reporting | Interactive parameters, narrative flow, templates |
| **Power BI** | Executive reporting, BI analytics | Rich visuals, cross-source integration, mobile apps |
| **Custom Apps** | Unique requirements, customer-facing | Full flexibility, API access, tailored workflows |

## Learning Objectives
1. Configure GitHub monitoring with Insights, Actions, and status badges
2. Create Azure Dashboards with tiles, parameters, and shared access
3. Build Azure Monitor Workbooks with interactive parameters and queries
4. Connect Power BI to Log Analytics, Azure DevOps, and Application Insights
5. Access Azure Monitor REST APIs for custom monitoring applications
6. Track pipeline health metrics: failure rate, duration, flaky tests
7. Optimize pipelines for cost, time, performance, and reliability
8. Implement pipeline concurrency optimization for faster feedback

## Critical Notes
- ⚠️ **Different tools, different purposes**: Dashboards for monitoring, Workbooks for investigation, Power BI for executive reporting
- 💡 **Start with built-in tools**: Use native Azure/GitHub features before building custom solutions
- 🎯 **Pipeline health is critical**: Monitor failure rate, duration, and flaky tests to maintain CI/CD reliability
- 📊 **Optimize across dimensions**: Balance cost, time, performance, and reliability—not just speed

## Quick Reference

**Tool Selection Decision Tree**:
```
Need real-time monitoring? → Azure Dashboards
Need interactive investigation? → Azure Monitor Workbooks
Need executive reporting? → Power BI
Need repository insights? → GitHub Monitoring
Need unique custom features? → Custom Application
```

**Pipeline Health Metrics**:
- **Failure Rate** = (Failed Runs / Total Runs) × 100
- **Duration** = Queue Time + Execution Time + Approval Time
- **Flaky Tests** = Tests with inconsistent pass/fail results

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/1-introduction)
