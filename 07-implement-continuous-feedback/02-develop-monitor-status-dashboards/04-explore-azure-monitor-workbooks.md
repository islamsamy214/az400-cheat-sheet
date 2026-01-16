# Explore Azure Monitor Workbooks

## Key Concepts
- **Azure Monitor Workbooks** are interactive, document-like reports combining text, queries, and visualizations
- **Interactive parameters** enable dynamic filtering and drill-down without editing workbook structure
- **Narrative flow**: Sequential steps guide users through logical analysis (introduction → analysis → recommendations)
- **Template ecosystem**: Leverage Microsoft, community, and organizational templates for fast deployment
- **No automatic refresh**: Manual refresh required (designed for analytical sessions, not real-time monitoring)
- **Vertical document format**: Scrollable structure optimized for investigation, not "single pane of glass"

## Azure Monitor Workbooks Overview

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Interactive Exploration** | Parameters + drill-down + cross-filtering | Troubleshooting, incident investigation |
| **Narrative Structure** | Markdown + queries + visualizations | Guided workflows, documentation |
| **Rich Visualizations** | Charts, grids, metrics, custom HTML | Performance analysis, reporting |
| **Template Ecosystem** | Pre-built Microsoft/community templates | Fast deployment, best practices |

## Key Advantages

### Dual Data Source Support

| Data Source | Capabilities | Use Case |
|-------------|--------------|----------|
| **Metrics** | Platform metrics, custom metrics, metric charts | Real-time performance, infrastructure monitoring |
| **Logs** | Log Analytics (KQL), Application Insights, tables, complex aggregations | Detailed analysis, troubleshooting |

**Cross-Source Correlation Example**:
```yaml
Scenario: Correlate performance with errors
Step 1: Show application request rate (metrics)
Step 2: Show error logs (logs) with same time range
Insight: CPU spike at 14:30 → Error log volume increased 5× → Identify causation
```

### Interactive Parameterized Reporting

**Parameter Types**:

| Parameter Type | Purpose | Example | Dynamic Behavior |
|----------------|---------|---------|------------------|
| **Time Range** | Control time window | Last hour, 24h, 7d, Custom | All queries update automatically |
| **Dropdown** | Select from list | Subscription, Resource Group, App | Static or query-derived values |
| **Text Input** | Free-form search | User ID, error message, transaction | Regex validation optional |
| **Resource Picker** | Select Azure resources | VMs, databases, web apps | Built-in Azure resource picker |
| **Query-Derived** | Values from query results | Distinct app names, error codes | Dynamic values update with data |

**Interactive Visualization Updates**:
```yaml
Example Workflow:
1. Table shows "Top 10 Operations by Request Count"
2. User clicks specific operation name
3. Charts below auto-filter to show only that operation
4. Additional tables show detailed telemetry for selection

Benefits:
- Flexible exploration without pre-configured views
- One workbook replaces dozens of static variations
- Non-technical users explore data without KQL
```

### Document-Like Narrative Flow

**Content Structure**:

| Step Type | Purpose | Example |
|-----------|---------|---------|
| **Markdown Text** | Context, explanations, instructions | Section headers, troubleshooting steps |
| **Query Steps** | Visualize data via charts/tables | Performance metrics, error analysis |
| **Parameter Steps** | Collect user input | Time range, resource selection |
| **Group Steps** | Organize related content | Collapsible sections by topic |

**Example Troubleshooting Guide**:
```yaml
1. Introduction (Markdown): Problem description & approach
2. Parameters: Time range, resource selection
3. Overview (Queries): High-level metrics showing problem scope
4. Hypothesis 1 (Markdown + Queries): "Is it a network issue?" + network metrics
5. Hypothesis 2 (Markdown + Queries): "Is it a database issue?" + DB performance
6. Detailed Analysis (Queries): Drill-down into root cause
7. Recommendations (Markdown): Remediation & prevention steps
```

### Personal and Shared Workbooks

| Type | Scope | Storage | Use Case | RBAC |
|------|-------|---------|----------|------|
| **Personal** | Private to user | User's Azure Monitor workspace | Individual analysis, experimentation | Not required |
| **Shared** | Authorized users | Azure resource (subscription/resource group) | Team troubleshooting, reporting | Required |

**Development Workflow**:
```
1. Create personal workbook → Experiment & build content
2. Test interactivity → Validate parameters, selections
3. Share with team → Publish to shared workbook
4. Gather feedback → Team reviews & suggests improvements
5. Iterate → Make updates (versioning supported)
6. Establish standard → Document usage, link from dashboards
```

### Collaborative-Friendly Authoring

**Edit Mode Features**:
- **Visual editor**: Add/drag/drop steps, settings panels, live preview
- **Query builder**: Syntax highlighting, auto-completion, example queries
- **Visualization config**: Chart types, colors, thresholds, conditional formatting
- **Parameter config**: Type selection, query for values, defaults, dependencies

**Low-Code Benefits**:
- Non-developers create sophisticated reports
- Build workbooks in hours, not days
- Lower maintenance than code-based tools
- More team members contribute

**Advanced Options (Optional)**:
- JSON editing for advanced customization
- Custom HTML/JavaScript visualizations
- API integration for data enrichment

### Template Ecosystem

**Template Sources**:

| Source | Coverage | Quality | Examples |
|--------|----------|---------|----------|
| **Microsoft** | App Insights, VM Insights, Container Insights | Maintained by Microsoft | App Performance, VM Performance, AKS |
| **GitHub Community** | Industry/tech-specific | Community contributions | Retail, healthcare, Cosmos DB, Azure SQL |
| **Organizational** | Internal patterns | Internal best practices | Company-specific standards |

**Using Templates**:
```
1. Browse gallery: Workbooks → Templates
2. Select template: Choose matching scenario
3. Configure parameters: Subscriptions, resources, etc.
4. View results: Template populates with your data
5. Customize (optional): Add org-specific content
6. Save: Personal or shared workbook for future use
```

## Limitations to Understand

| Limitation | Impact | Workaround |
|------------|--------|------------|
| **No automatic refresh** | Not suitable for real-time monitoring | Use Azure Dashboards for monitoring |
| **No dense layout** | Cannot see all metrics at once (vertical scroll) | Use table of contents, collapsible groups |
| **Manual refresh required** | Users must click Refresh | Acceptable for investigation sessions |
| **Not "single pane of glass"** | Must scroll through workbook | Use dashboards for at-a-glance views |

## When to Use Azure Monitor Workbooks

### Ideal Scenarios ✅
- **Investigation & troubleshooting**: Root cause analysis, performance diagnosis
- **Ad-hoc analysis**: Exploring data, validating hypotheses
- **Reporting & documentation**: Performance reports, incident postmortems
- **Template-based monitoring**: Standardized analysis across resources
- **Training & knowledge sharing**: Teaching team members monitoring skills

### Use Dashboards Instead ❌
- **Continuous monitoring**: Always-on system health displays
- **At-a-glance status**: See all metrics without scrolling
- **Multiple simultaneous consumers**: Many viewers watching real-time data

## Common Workbook Patterns

### Troubleshooting Guide
```yaml
Structure:
  - Introduction: Problem description
  - Parameters: Time range, resource
  - Overview: High-level metrics
  - Hypothesis Sections: Test each potential cause
  - Detailed Analysis: Drill-down into findings
  - Recommendations: Remediation steps

Example: Application Slowness
  → Check request rates → Analyze dependencies
  → Examine database queries → Identify bottleneck
```

### Performance Analysis Report
```yaml
Structure:
  - Executive Summary: Key KPIs
  - Trend Analysis: Performance over time
  - Bottleneck Identification: Slowest operations
  - Comparative Analysis: Across regions/versions
  - Optimization Recommendations: Specific actions
```

## Quick Commands

**Create Workbook**:
```bash
# Via Portal
Azure Monitor → Workbooks → New → Empty workbook

# From Template
Azure Monitor → Workbooks → Templates → Select template → Save copy
```

**Add Interactive Parameters**:
```kql
// Time Range Parameter
Type: Time range picker
Name: TimeRange
Default: Last 24 hours

// Resource Picker Parameter
Type: Resource picker
Resource type: Virtual machines
Allow multiple selections: Yes

// Query-Derived Parameter
Type: Dropdown
Query: 
  AppRequests
  | distinct AppName
  | order by AppName asc
```

**Collapsible Sections**:
```
1. Add → Group
2. Name: "Detailed Analysis"
3. Advanced Settings → Collapsed by default: Yes
4. Add steps inside group
```

## Critical Notes
- ⚠️ **Manual refresh only**: Not for real-time NOC displays
- 💡 **Use for investigation**: Complement dashboards (monitoring) with workbooks (analysis)
- 🎯 **Leverage templates**: Start with Microsoft templates, customize for your needs
- 📊 **Parameter-driven design**: One workbook replaces many static reports

## Example Use Cases

```yaml
Incident Investigation:
  Parameters: Time range, affected resources
  Steps:
    1. Show request volume + error rate spike
    2. Drill into failed requests
    3. Analyze dependency failures
    4. View related logs
    5. Document findings for postmortem

VM Performance Analysis:
  Parameters: Subscription, Resource Group, VM
  Steps:
    1. CPU/Memory/Disk trends over 30 days
    2. Compare to baselines
    3. Identify resource bottlenecks
    4. Show process-level details
    5. Provide scaling recommendations

Security Audit:
  Parameters: Time range, Subscription
  Steps:
    1. Failed login attempts by geography
    2. Suspicious activity patterns
    3. Security alert summary
    4. Compliance status
    5. Remediation actions
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/4-explore-azure-monitor-workbooks)
