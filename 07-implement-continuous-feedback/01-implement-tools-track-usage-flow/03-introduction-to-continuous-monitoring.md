# Introduction to Continuous Monitoring

## Key Concepts
- **Continuous monitoring**: Process of incorporating monitoring across all DevOps lifecycle phases
- Shift from reactive (users report issues) to proactive (detect before user impact)
- Azure Monitor = unified monitoring solution for full-stack observability
- Monitoring across: Development → Testing → Staging → Production

## Continuous Monitoring Lifecycle
```
Development: Local testing, performance profiling, test telemetry
     ↓
Testing: Load testing, integration monitoring, synthetic transactions
     ↓
Staging: Pre-production checks, canary monitoring, configuration validation
     ↓
Production: Real-time APM, infrastructure health, user analytics, alerting
```

## Enable Monitoring for Applications
| Approach | Method | Use Case |
|----------|--------|----------|
| **Azure DevOps Projects** | Auto-setup CI/CD + monitoring | Quick start, best practices |
| **Pipeline Integration** | Quality gates in release pipeline | Automated rollback on metrics |
| **Status Monitor** | Runtime instrumentation (no code) | Legacy apps, production systems |
| **SDK Integration** | Application Insights SDK | Full control, custom telemetry |

### SDK Custom Telemetry Example
```csharp
// Track custom business event
telemetryClient.TrackEvent("OrderPlaced",
    properties: new Dictionary<string, string> { 
        {"Category", "Electronics"} 
    },
    metrics: new Dictionary<string, double> { 
        {"OrderValue", 599.99} 
    });
```

## Infrastructure Monitoring
```powershell
# Enable VM Insights
Set-AzVMExtension -ResourceGroupName "rg-prod" `
    -VMName "vm-web01" `
    -Name "MicrosoftMonitoringAgent" `
    -Publisher "Microsoft.EnterpriseCloud.Monitoring"

# Configure performance counters
New-AzOperationalInsightsWindowsPerformanceCounterDataSource `
    -ObjectName "Processor" `
    -CounterName "% Processor Time" `
    -IntervalSeconds 60
```

## Azure Monitor Capabilities
- **Metrics**: Real-time performance data (CPU, memory, network)
- **Logs**: Detailed event data for analysis with KQL
- **Alerts**: Metric/log-based alerts with dynamic thresholds
- **Dashboards**: Shared visualizations across teams
- **Workbooks**: Interactive reports + troubleshooting guides

## Deployment Quality Gates
```yaml
# Example quality gate thresholds
Pre-deployment:
  - No active P0/P1 incidents
  - Error rate < 0.5% (last 15 minutes)

Post-deployment (monitor 20 minutes):
  - Response time P95 < 2 seconds
  - Error rate < 1%
  - No increase in exceptions vs. previous version
  - Minimum 100 requests processed

Action: Auto-rollback if gates fail
```

## Actionable Alerting Strategy
| Alert Type | Purpose | Configuration |
|-----------|---------|---------------|
| **Metric Alerts** | Threshold violations | Dynamic thresholds (ML-based) |
| **Log Alerts** | Query result conditions | KQL-based complex logic |
| **Smart Detection** | Anomaly detection | Auto-learns baseline patterns |

### Dynamic Thresholds Benefit
```
Static threshold problems:
- Business hours: 5,000 req/min (normal)
- Off-hours: 500 req/min (normal)
- Static threshold at 1,000 = false positives

Dynamic thresholds:
- Learn daily/weekly patterns
- Adapt to growth
- Alert on anomalies at any time
```

## Dashboards & Workbooks
**Dashboards**: Real-time operational views
- Executive: Business KPIs, uptime, costs
- Operations: Infrastructure health, active incidents
- Development: Performance, errors, dependencies
- Support: User experience, error search, transactions

**Workbooks**: Interactive troubleshooting guides
- Step-by-step investigation workflows
- Parameterized queries
- Conditional content based on data
- Version-controlled in source control

## Build-Measure-Learn Cycle
```
Build → Deploy feature/optimization
  ↓
Measure → Collect performance/business metrics
  ↓
Learn → Analyze impact, validate hypothesis
  ↓
Iterate → Improve based on insights
```

## Critical Notes
- ⚠️ **Separate monitoring instances** per environment (dev/test/prod)
- 💡 **Data latency**: 5-10 minutes typical (up to 15 minutes)
- 🎯 **Resource groups**: Organize by application lifecycle
- 📊 **Quality gates**: Prevent bad deployments automatically
- 🔄 **Infrastructure as Code**: ARM templates for consistent monitoring

## Quick Reference Commands
```bash
# Query cross-environment
az monitor log-analytics query \
    --workspace "workspace-id" \
    --analytics-query "requests | where timestamp > ago(1h)"

# Set up autoscaling
az monitor autoscale create \
    --resource "vm-scale-set" \
    --min-count 2 \
    --max-count 10 \
    --count 2
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-tools-track-usage-flow/3-introduction-to-continuous-monitoring)
