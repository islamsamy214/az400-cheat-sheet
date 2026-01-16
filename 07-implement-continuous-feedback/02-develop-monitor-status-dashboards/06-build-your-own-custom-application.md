# Build Your Own Custom Monitoring Application

## Key Concepts
- **Custom applications** access Azure monitoring data through REST APIs for tailored experiences
- **Complete flexibility**: Full control over UI, visualizations, interactions, and workflows
- **Multi-source integration**: Combine Azure Monitor, Log Analytics, App Insights, Azure DevOps, and external systems
- **Significant investment**: Requires weeks/months of development + ongoing maintenance
- **Build when**: Unique requirements, customer-facing needs, or advanced automation justify ROI
- **Start with standard tools**: Use Dashboards/Workbooks/Power BI first, build custom only when necessary

## Data Access Through APIs

### Azure Monitor REST API

| API | Endpoint Pattern | Capabilities | Use Case |
|-----|------------------|--------------|----------|
| **Metrics API** | `https://management.azure.com/{resourceId}/providers/Microsoft.Insights/metrics` | Platform/custom metrics, aggregations, filtering | Real-time resource monitoring |
| **Logs API** | `https://api.loganalytics.io/v1/workspaces/{workspaceId}/query` | Execute KQL queries, JSON results | Log analysis, troubleshooting |
| **Application Insights API** | `https://api.applicationinsights.io/v1/apps/{appId}/query` | Application telemetry, traces, events | Application performance monitoring |
| **Azure DevOps API** | `https://dev.azure.com/{org}/_apis/` | Work items, builds, releases, tests | CI/CD dashboards |

### Authentication Approaches

| Method | Use Case | Security | Implementation |
|--------|----------|----------|----------------|
| **Microsoft Entra ID (OAuth 2.0)** | Production applications | ✅ Secure, integrates with Azure RBAC | Obtain token → Include in Authorization header |
| **Service Principals** | Automated scripts, CI/CD pipelines | ✅ Secure for background services | Create App Registration → Grant permissions → Client credentials flow |
| **API Keys** | Application Insights-specific | ⚠️ Less secure | Generate API key → Include in header |

## Building Custom Dashboards

### Technology Stacks

| Type | Technologies | Hosting | Use Case |
|------|-------------|---------|----------|
| **Web Applications** | React/Angular/Vue + Chart.js/D3/Highcharts<br/>Backend: ASP.NET Core, Node.js, Python | Azure App Service, Static Web Apps, Container Apps | Team dashboards, internal portals |
| **Mobile Applications** | Native: Swift/Kotlin<br/>Cross-platform: React Native, Flutter, .NET MAUI | App stores | On-call engineer dashboards, field technician tools |
| **Desktop Applications** | Electron, WPF/.NET | Distributed executables | Operations center displays, executive dashboards |
| **Real-Time Dashboards** | SignalR/WebSockets | Azure SignalR Service | Live monitoring with push updates |

### Common Patterns

**Aggregation Layer**:
```
Pattern: Backend queries multiple Azure Monitor sources
→ Aggregates results
→ Caches for performance
→ Serves to clients

Benefits:
- Reduced API calls
- Faster client response
- Consolidated data model

Implementation:
- Azure Functions (scheduled triggers)
- Azure App Service (background workers)
```

**Custom Alerting Logic**:
```
Pattern: Application queries monitoring data
→ Applies custom business rules
→ Triggers notifications

Use Case:
- Complex alert conditions beyond Azure Monitor alerts
- Multi-source correlation
- Business rule enforcement

Implementation:
- Azure Logic Apps
- Azure Functions with custom code
```

**Multi-Tenancy**:
```
Pattern: Single app serves multiple customers
→ Isolated monitoring data per tenant
→ Row-level security

Use Case:
- MSP platforms
- SaaS vendor monitoring portals

Implementation:
- Tenant-specific workspace IDs
- Data layer security isolation
```

## Advantages of Custom Applications

| Advantage | Description | Example |
|-----------|-------------|---------|
| **Complete Flexibility** | Full control over UI, UX, branding, features | Custom drag-and-drop, voice commands, annotations |
| **Multi-Source Integration** | Unlimited data sources | Azure Monitor + CRM + ticketing + on-premises |
| **Tailored Workflows** | Industry/role-specific experiences | Healthcare HIPAA workflows, finance compliance reporting |
| **Customer-Facing** | Public status pages, SLA reports | Branded service health pages for B2B customers |

## Disadvantages and Considerations

| Concern | Impact | Mitigation |
|---------|--------|------------|
| **Engineering Investment** | Weeks to months for MVP + ongoing maintenance | Start with standard tools, build only when clear ROI |
| **Ongoing Maintenance** | API updates, security patches, performance tuning | Budget for 20-30% time for maintenance |
| **Scaling & Reliability** | Infrastructure management, monitoring the monitor | Use managed Azure services (App Service, Functions) |
| **Security Responsibilities** | Auth/authz, data protection, audit logging | Leverage Azure AD, encrypt data, implement RBAC |

## When to Build Custom Applications

### Build Custom When ✅
- **Unique requirements**: Standard tools can't support specialized workflows
- **Customer-facing**: Need branded status pages or customer-specific dashboards
- **Embedded monitoring**: Integrate into existing applications or products
- **Advanced automation**: Custom alerting, ML-based anomaly detection, automated remediation
- **Multi-source integration**: Complex aggregation from Azure + on-premises + third-party
- **Strong ROI**: Benefits clearly justify costs (efficiency gains, revenue protection)

### Use Existing Tools When ❌
- **Standard monitoring**: Dashboards/Workbooks/Power BI meet needs
- **Time constraints**: Need solution quickly
- **Limited resources**: Small team can't maintain long-term
- **Frequent changes**: Requirements evolve rapidly

## Hybrid Approach
```yaml
Phase 1: Start
  - Use Azure Dashboards for immediate monitoring
  - Use Workbooks for investigation

Phase 2: Assess
  - Identify specific gaps after using standard tools
  - Quantify pain points and ROI

Phase 3: Build Incrementally
  - Create custom components for high-value scenarios
  - Keep standard tools for internal monitoring
  - Example: Custom customer status page + Azure Dashboards for ops

Phase 4: Iterate
  - Continuously evaluate if custom investment justified
  - Consider evolving standard tool capabilities
```

## Quick Commands

### Call Azure Monitor Metrics API
```bash
# Get Access Token
az account get-access-token --resource https://management.azure.com/

# Query Metrics
curl -X GET \
  "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/providers/Microsoft.Insights/metrics?api-version=2021-05-01&metricnames=Percentage%20CPU&aggregation=Average" \
  -H "Authorization: Bearer {access_token}"
```

### Call Log Analytics API
```bash
# Get Access Token
az account get-access-token --resource https://api.loganalytics.io

# Query Logs
curl -X POST \
  "https://api.loganalytics.io/v1/workspaces/{workspaceId}/query" \
  -H "Authorization: Bearer {access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "AppRequests | where TimeGenerated > ago(1h) | summarize count() by bin(TimeGenerated, 5m)"
  }'
```

### Call Application Insights API
```bash
# Using API Key
curl -X GET \
  "https://api.applicationinsights.io/v1/apps/{appId}/metrics/requests/count?timespan=PT1H" \
  -H "x-api-key: {api_key}"
```

### Simple React Dashboard Component
```javascript
// Example: Fetch and display metrics
import React, { useEffect, useState } from 'react';
import { Line } from 'react-chartjs-2';

function MetricsDashboard() {
  const [data, setData] = useState([]);

  useEffect(() => {
    // Fetch from backend API aggregation layer
    fetch('/api/metrics/cpu')
      .then(res => res.json())
      .then(data => setData(data));
  }, []);

  return (
    <div>
      <h1>CPU Utilization</h1>
      <Line data={formatChartData(data)} />
    </div>
  );
}
```

## Critical Notes
- ⚠️ **High development cost**: Only build when standard tools insufficient
- 💡 **Aggregation layer pattern**: Cache API results to reduce costs and improve performance
- 🎯 **Secure authentication**: Always use Microsoft Entra ID for production apps
- 📊 **Monitor the monitor**: Custom monitoring apps need monitoring themselves

## Common Custom Application Scenarios

```yaml
MSP Multi-Tenant Portal:
  Requirements:
    - Each customer sees only their resources
    - White-labeled branding per customer
    - Custom billing integration
  Stack:
    - React frontend
    - ASP.NET Core backend
    - Azure App Service + Azure SQL
  Integration:
    - Azure Monitor APIs with tenant filtering
    - Custom billing API
    - Multi-tenant data isolation

Public Status Page:
  Requirements:
    - Customer-facing service health
    - No authentication required (public)
    - Custom domain & branding
  Stack:
    - Next.js static site
    - Azure Static Web Apps
  Integration:
    - Application Insights availability tests
    - Azure Service Health API
    - Scheduled updates every 5 minutes

Mobile On-Call App:
  Requirements:
    - Real-time alerts on mobile
    - Acknowledge/assign incidents
    - Quick remediation actions
  Stack:
    - React Native (iOS + Android)
    - Azure Functions backend
    - Azure Notification Hubs
  Integration:
    - Azure Monitor alerts via webhooks
    - Azure DevOps API for incident tickets
    - Push notifications
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/6-build-your-own-custom-application)
