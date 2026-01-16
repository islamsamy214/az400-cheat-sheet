# Explore Azure Dashboards

## Key Concepts
- **Azure Dashboards** provide tile-based, customizable views of Azure resources and monitoring data
- **Native Azure integration**: Pin visualizations from any Azure service without configuration
- **Dual data sources**: Support both metrics (time-series) and logs (events) on same dashboard
- **Automatic refresh**: Dashboards update automatically based on time range (5-30 minutes)
- **RBAC integration**: Share dashboards with role-based access control (Owner, Contributor, Reader)
- **Limitations**: 30-day max for logs, no interactivity, limited log visualization options

## Azure Dashboards Overview

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Single Pane of Glass** | Unified view across multiple services | Infrastructure + application monitoring |
| **Tile-Based Interface** | Drag-and-drop tiles from Azure services | Customizable layouts |
| **Portal-Based** | Built into Azure portal | No installation required |
| **Quick Issue ID** | Critical metrics visible immediately | Operations centers, incident response |

## Key Advantages

### Deep Azure Integration

**Pinnable Sources**:
- Metrics Explorer charts (real-time/historical)
- Log Analytics queries (KQL visualizations)
- Application Insights (performance, analytics)
- Azure Monitor maps (dependency visualization)
- Availability tests (endpoint monitoring)
- Resource blade charts (VM metrics, DB performance)
- Azure Advisor recommendations
- Azure Service Health

**Pin Workflow**:
```
1. Navigate to visualization in Azure portal
2. Click Pin icon (pushpin) in top-right
3. Choose existing dashboard or create new
4. Tile appears on selected dashboard
```

### Dual Data Source Support

| Data Type | Characteristics | Refresh Rate | Use Case |
|-----------|-----------------|--------------|----------|
| **Metrics** | High-frequency time-series | 5-30 min (depends on time range) | Real-time monitoring, capacity planning |
| **Logs** | Discrete events with context | 1 minute (shared dashboards) | Troubleshooting, compliance, detailed analytics |

**Combined Value**: Correlate CPU metrics (metrics) with error log volume (logs) to reveal causation

### Multi-Source Data Combination

**Example: Full-Stack Application Monitoring**
```yaml
Frontend:
  - Application Insights: Page load times, user sessions
API Layer:
  - API Management: Request rates, latency, errors
Backend:
  - App Service: CPU, memory, response times
Database:
  - Azure SQL: DTU consumption, query performance
Dependencies:
  - Azure Storage: Blob transactions, latency
```

### Personal and Shared Dashboards

| Type | Visibility | Storage | Use Case | RBAC |
|------|------------|---------|----------|------|
| **Personal** | Creating user only | User's Azure profile | Individual monitoring, drafts | Not required |
| **Shared** | Authorized users | Azure resource group | Team dashboards, operational views | Required |

**RBAC Roles**:
- **Owner**: Full control (edit, delete, manage permissions)
- **Contributor**: Edit content only (no RBAC changes)
- **Reader**: View-only access

### Automatic Refresh

**Refresh Intervals**:
- **Last hour**: ~5 minutes
- **Last 24 hours**: ~15 minutes
- **Last 30 days**: ~30 minutes
- **Logs**: 1 minute (all shared dashboards)

**Manual Refresh**:
- Full dashboard: Select **Refresh** button in toolbar
- Individual tile: Hover → More options (...) → Refresh

### Parameterized Dashboards

**Parameter Types**:

| Parameter | Purpose | Example Values | Use Case |
|-----------|---------|----------------|----------|
| **Timestamp** | Time range for all tiles | Last hour, 24h, 7d, 30d, Custom | Quick time range switching |
| **Custom** | Filter by values | Subscription, Resource Group, Environment | Single dashboard for multiple contexts |

**Benefits**:
- Reusability: One template serves multiple scenarios
- Reduced management: Fewer dashboards to maintain
- User flexibility: Customize view without editing

### Flexible Layout Options

**Layout Capabilities**:
- Drag-and-drop positioning
- Resizable tiles (grid snapping)
- Multi-column layouts
- Visual hierarchy (tile sizing)

**Best Practices**:
```yaml
Visual Hierarchy:
  - Top left: Most critical/actionable metrics
  - Larger tiles: More important information
  - Smaller tiles: Supporting context

Grouping:
  - Infrastructure | Applications | Security | Costs
  - Use whitespace for section separation
  - Consistent sizing for related metrics

Optimize for Audience:
  - Operations: Dense layouts, detailed metrics
  - Executives: Sparse layouts, high-level KPIs
  - Mixed: Hierarchical (summary top, details below)
```

### Full-Screen Mode

**Use Cases**:
- **Operations Centers (NOC)**: Wall-mounted monitors for 24/7 visibility
- **Incident Response**: Maximize screen space during outages
- **Presentations**: Professional demos without portal chrome
- **Focus Mode**: Eliminate distractions during monitoring sessions

**Activation**: Select **Full screen** button in toolbar or press F11

## Limitations to Consider

| Limitation | Impact | Workaround |
|------------|--------|------------|
| **No data tables for logs** | Cannot display row-by-row log data | Use Workbooks or link to Log Analytics |
| **10 data series max** | High-cardinality data grouped as "Other" | Filter to top 10 or use Workbooks |
| **No custom parameters for logs** | Log tiles don't support filtering | Create separate dashboards per context |
| **30-day max for logs** | Cannot show quarters/years of log data | Use Workbooks or export to Power BI |
| **Shared dashboard required** | Log charts need shared dashboards | Create shared dashboard with restricted RBAC |
| **No interactivity** | Can't drill-down on click | Configure tiles to link to resources/queries |
| **Limited drill-down** | Minimal context for investigations | Use Workbooks for detailed analysis |

## When to Use Azure Dashboards

### Ideal Scenarios ✅
- **Continuous operations monitoring**: NOC displays, always-on views
- **Executive reporting**: High-level KPIs, SLA tracking
- **Cross-service views**: Full-stack monitoring, multi-subscription visibility
- **Quick creation**: Need dashboard in minutes

### Consider Alternatives ❌
- **Deep log analysis with tables**: → Azure Monitor Workbooks
- **Long-term trends (>30 days logs)**: → Workbooks or Power BI
- **Highly interactive exploration**: → Workbooks or Power BI
- **Complex parameterized log views**: → Azure Monitor Workbooks

## Quick Commands

**Create Dashboard**:
```bash
# Via Portal
Navigate to: Dashboards → New dashboard → Blank dashboard

# Pin existing visualization
Navigate to resource → Metrics/Logs → Create chart → Pin to dashboard
```

**Share Dashboard**:
```bash
# Via Portal
Dashboard → Share → Select resource group → Assign RBAC roles

# Example RBAC Assignment
Resource Group: dashboards-rg
Role: Reader (for viewers)
Role: Contributor (for editors)
```

## Critical Notes
- ⚠️ **Personal dashboards for logs not supported**: Must use shared dashboards
- 💡 **Use time range parameters**: Enable quick switching between real-time and historical
- 🎯 **Configure "On click" actions**: Link tiles to resources for deeper investigation
- 📊 **Multi-tool approach**: Dashboards for monitoring + Workbooks for investigation + Power BI for reporting

## Common Dashboard Patterns

```yaml
Infrastructure Dashboard:
  - VM fleet: CPU, memory, disk, network
  - Databases: Query performance, connections
  - Storage: Transactions, latency, availability
  - Networks: Gateway health, request rates

Application Dashboard:
  - Request rates (App Insights)
  - Response times (App Insights)
  - Failure rates (App Insights)
  - Dependency performance (App Insights)

DevOps Dashboard:
  - Build success rates (Azure DevOps)
  - Deployment frequency (Azure DevOps)
  - Application health (App Insights)
  - Cost trends (Cost Management)
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/3-explore-azure-dashboards)
