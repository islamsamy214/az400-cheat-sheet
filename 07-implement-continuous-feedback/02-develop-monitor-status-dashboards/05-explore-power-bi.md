# Explore Power BI

## Key Concepts
- **Power BI** is a business intelligence platform for executive dashboards and long-term trend analysis
- **Cross-source integration**: Combine Azure Monitor, Azure DevOps, and business systems (CRM, ERP)
- **Rich visualizations**: 150+ chart types, custom visuals, conditional formatting, interactive cross-filtering
- **Organization-wide sharing**: Web, mobile apps, email subscriptions, embedded analytics
- **Logs only**: No native real-time metrics support (workaround: export metrics to logs)
- **Import model required**: Must import query results (no DirectQuery for most Azure sources)

## Power BI for DevOps Monitoring

| Use Case | Metrics Displayed | Business Value |
|----------|-------------------|----------------|
| **Executive Dashboards** | Deployment frequency, lead time, MTTR, change failure rate | Align DevOps with business goals |
| **Business Impact Analysis** | Revenue correlation, customer satisfaction, feature adoption | Connect performance to outcomes |
| **Compliance Reporting** | Audit trails, SLA uptime, security posture, vulnerability trends | Audit-ready documentation |

## Data Integration Approaches

### Azure Monitor Log Analytics Integration

**Integration Methods**:

| Method | Description | Best For | License Required |
|--------|-------------|----------|------------------|
| **Direct Import** | One-time export to Power BI | Historical snapshots | Power BI Free |
| **Scheduled Refresh** | Auto re-run queries (hourly/daily) | Automatic updates | Power BI Pro/Premium |
| **Power BI Connector** | Native connector to Log Analytics | Simplified setup | Power BI Pro/Premium |

**Configuration**:
```
1. In Power BI Desktop:
   - Get Data → Azure → Azure Monitor Logs
2. Provide:
   - Workspace ID
   - Authentication credentials (Microsoft Entra ID)
3. Write KQL query or select tables
4. Load data into Power BI model
```

### Azure DevOps Integration

**Integration Options**:
- **Power BI Connector**: Native Azure DevOps connector for work items, builds, releases, tests
- **Analytics Views**: Pre-built data connections to Azure DevOps Analytics
- **Custom OData Queries**: Specialized analytics via OData endpoint

**Example Query**:
```
1. Get Data → Online Services → Azure DevOps
2. Enter organization URL: https://dev.azure.com/{org}
3. Select data: Work items, Builds, Releases, Test results
4. Transform and load
```

### Application Insights Integration

**Integration Options**:
- **Continuous Export**: Stream telemetry to Azure Storage/Event Hubs → Power BI
- **Power BI Connector**: Direct connection to Application Insights resources
- **Analytics API**: REST API calls from Power BI for custom retrieval

## Key Advantages

### Rich Visualization Library
- **150+ visualization types**: Standard charts, maps, gauges, KPIs
- **Custom visuals**: Import from marketplace or build your own
- **Conditional formatting**: Color scales, data bars, icons based on values
- **Themes and branding**: Corporate styling, color schemes

### Extensive Interactivity
- **Cross-filtering**: Click one visual → all related visuals filter automatically
- **Drill-down hierarchies**: Year → Quarter → Month → Day
- **Slicers and filters**: User-controlled filtering without editing
- **Bookmarks**: Save specific filter states and navigate
- **Rich tooltips**: Additional context on hover

### Organization-Wide Sharing
- **Power BI Service**: Web access from any browser
- **Mobile apps**: Native iOS/Android apps with offline support
- **Email subscriptions**: Scheduled report delivery
- **Embedded analytics**: Embed in SharePoint, Teams, custom apps
- **Public publishing**: Share with external stakeholders (with controls)

### Multi-Source Data Integration
- **100+ data connectors**: Azure, databases, SaaS apps, files, web APIs
- **Data mashup**: Combine monitoring with business data (sales, support tickets)
- **Relationships**: Define joins between tables from different sources
- **Unified view**: Technical metrics alongside business outcomes

### Performance Through Caching
- **In-memory engine**: Query results cached in columnar format
- **Fast queries**: Sub-second response times
- **Aggregation caching**: Pre-calculated aggregates
- **Incremental refresh**: Update only changed data

## Limitations to Consider

| Limitation | Impact | Workaround |
|------------|--------|------------|
| **Logs only, no native metrics** | Real-time infrastructure monitoring challenging | Export metrics to logs via diagnostic settings |
| **No ARM integration** | Can't deploy via IaC (ARM, Bicep, Terraform) | Manage in Power BI Service separately |
| **Import required** | Must import data, no real-time DirectQuery | Schedule frequent refreshes |
| **Result size limits** | 1 GB per dataset (shared), 500K rows per query | Use aggregation, time-based filtering |
| **Refresh limitations** | 8 refreshes/day (shared/Pro), unlimited (Premium) | Upgrade to Premium for high-frequency needs |

## When to Use Power BI

### Ideal Scenarios ✅
- **Executive reporting**: Business-focused dashboards for leadership with KPIs
- **Historical analysis**: Long-term trend analysis (months/years)
- **Cross-domain integration**: Monitoring + business/financial/customer data
- **Organization-wide distribution**: Share with large audiences via web/mobile
- **Complex analytics**: Advanced calculations, forecasting, predictive analysis

### Use Alternatives ❌
- **Real-time monitoring**: → Azure Dashboards (automatic refresh, metrics support)
- **Ad-hoc investigation**: → Azure Monitor Workbooks (interactive parameters, KQL)
- **IaC deployment**: → Azure Dashboards/Workbooks (ARM integration)
- **Minute-level refresh**: → Azure Dashboards or custom applications

## Quick Commands

### Connect to Log Analytics
```powerquery
// In Power BI Desktop
let
    Source = AzureMonitorLogs.Contents(
        "WorkspaceId", 
        "SubscriptionId", 
        "ResourceGroupName"
    ),
    Query = Source{[Kind="Query"]}[Data],
    Result = Query(
        "AppRequests
        | where TimeGenerated > ago(30d)
        | summarize RequestCount = count() by bin(TimeGenerated, 1d)"
    )
in
    Result
```

### Schedule Refresh
```
1. Publish report to Power BI Service
2. Go to Dataset settings
3. Configure Scheduled refresh:
   - Frequency: Daily, Weekly
   - Time: Select specific times
   - Time zone: Set appropriate zone
4. Configure credentials (Microsoft Entra ID)
5. Apply
```

### Embed in Teams
```
1. Publish report to Power BI Service
2. In Teams: Add tab → Power BI
3. Select workspace and report
4. Team members see embedded report
```

## Critical Notes
- ⚠️ **Not for real-time monitoring**: Use Dashboards for continuous operations
- 💡 **Multi-tool strategy**: Power BI as executive layer atop Dashboards (ops) + Workbooks (investigation)
- 🎯 **Combine with business data**: Link technical metrics to revenue, NPS, support tickets
- 📊 **Schedule refreshes carefully**: Balance freshness with API costs and quotas

## Example Power BI Reports

### Executive DevOps Dashboard
```yaml
Deployment Metrics:
  - Deployment frequency: Releases per week
  - Lead time: Commit to production (days)
  - Change failure rate: % of deployments causing incidents
  - MTTR: Mean time to recovery (hours)

Performance Trends:
  - Response time P95: 30-day trend
  - Error rate: By application tier
  - Availability: Monthly uptime %
  
Cost Management:
  - Azure spending: By service, by month
  - Agent minutes consumed: Trend line
  - Cost per deployment: Calculated measure
```

### Business Impact Analysis
```yaml
Revenue Correlation:
  - Transaction volume vs. response time
  - Revenue lost during incidents
  - Feature adoption rate by customer segment

Customer Satisfaction:
  - NPS score vs. application uptime
  - Support ticket volume vs. error rate
  - User engagement vs. performance
```

## Integration Pattern
```yaml
Power BI as BI Layer:
  Layer 1 (Operations): Azure Dashboards → Real-time monitoring
  Layer 2 (Investigation): Azure Workbooks → Troubleshooting
  Layer 3 (Executive): Power BI → Long-term trends, business insights

Data Flow:
  Azure Monitor/DevOps → Log Analytics → Power BI → Executive Reports
  
Refresh Strategy:
  - Daily refresh: Historical trends
  - Real-time needs: Use Dashboards instead
  - Weekly reports: Email subscriptions to stakeholders
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/5-explore-power-bi)
