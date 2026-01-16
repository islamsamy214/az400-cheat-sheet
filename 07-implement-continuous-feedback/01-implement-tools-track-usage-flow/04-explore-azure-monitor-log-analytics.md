# Explore Azure Monitor and Log Analytics

## Key Concepts
- **Azure Monitor**: Unified observability platform for all Azure resources
- **Log Analytics**: Query engine for analyzing telemetry with KQL
- **Microsoft Monitoring Agent (MMA)**: Agent for collecting VM telemetry
- Data flow: Sources → Agent → Log Analytics Workspace → KQL Queries → Insights

## Azure Monitor Architecture
```
Data Sources:
├── Azure Resources (platform metrics, activity logs)
├── Applications (Application Insights SDK)
├── Virtual Machines (performance counters, event logs)
├── Containers (logs, Kubernetes metrics)
└── Custom Sources (REST API, Data Collector API)
        ↓
Log Analytics Workspace (columnar storage, optimized for analytics)
        ↓
Query & Analysis (KQL, dashboards, alerts, workbooks)
```

## Log Analytics Performance
| Feature | Capability | Benefit |
|---------|------------|---------|
| **Fast Ingestion** | Billions of events/day | Handle massive scale |
| **Compression** | 90%+ storage reduction | Cost optimization |
| **Query Speed** | Terabytes in seconds | Real-time insights |
| **Retention** | 30 days to 12 years | Compliance & historical analysis |

## Setup Walkthrough

### Step 1: Create Workspace
```powershell
# Create resource group and workspace
$ResourceGroup = "rg-monitoring-prod"
$WorkspaceName = "law-monitoring-prod"
$Location = "eastus"

# Create workspace
New-AzOperationalInsightsWorkspace `
    -Location $Location `
    -Name $WorkspaceName `
    -ResourceGroupName $ResourceGroup

# Enable solutions
$Solutions = "CapacityPerformance", "LogManagement", 
             "ChangeTracking", "ProcessInvestigator"
foreach ($solution in $Solutions) {
    Set-AzOperationalInsightsIntelligencePack `
        -ResourceGroupName $ResourceGroup `
        -WorkspaceName $WorkspaceName `
        -IntelligencePackName $solution `
        -Enabled $true
}

# Enable IIS logs
Enable-AzOperationalInsightsIISLogCollection `
    -ResourceGroupName $ResourceGroup `
    -WorkspaceName $WorkspaceName
```

### Step 2: Connect VMs
```powershell
# Install MMA extension
$PublicSettings = @{"workspaceId" = "<workspace-id>"}
$ProtectedSettings = @{"workspaceKey" = "<workspace-key>"}

Set-AzVMExtension `
    -ExtensionName "MicrosoftMonitoringAgent" `
    -ResourceGroupName "rg-vms-prod" `
    -VMName "vm-web01" `
    -Publisher "Microsoft.EnterpriseCloud.Monitoring" `
    -ExtensionType "MicrosoftMonitoringAgent" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location "eastus"
```

### Step 3: Configure Performance Counters
```powershell
# Example: Add CPU counter
New-AzOperationalInsightsWindowsPerformanceCounterDataSource `
    -ResourceGroupName "rg-monitoring-prod" `
    -WorkspaceName "law-monitoring-prod" `
    -ObjectName "Processor" `
    -InstanceName "_Total" `
    -CounterName "% Processor Time" `
    -IntervalSeconds 60 `
    -Name "CPU Monitoring"
```

## Performance Counter Categories
| Category | Counters | Purpose |
|----------|----------|---------|
| **Processor** | % Processor Time, Queue Length | CPU utilization |
| **Memory** | Available MBytes, Page Faults/sec | Memory pressure |
| **LogicalDisk** | Disk Transfers/sec, Avg Disk sec/Read | Disk I/O |
| **Network** | Bytes Received/sec, Bytes Sent/sec | Network throughput |
| **SQL Server** | Lock Escalations, Deadlocks/sec | Database performance |

## Sampling Interval Trade-offs
| Interval | Use Case | Cost |
|----------|----------|------|
| 10 seconds | Troubleshooting, high resolution | High |
| 60 seconds | Standard operational monitoring | Balanced |
| 300 seconds (5 min) | Capacity planning | Low |

## Microsoft Monitoring Agent Architecture
```
Agent Components:
├── Data Collection (counters, logs, events)
├── Local Processing (filtering, parsing, enrichment)
├── Secure Transmission (TLS 1.2+, compression, batching)
└── Reliability (10 GB local buffer, retry logic, health monitoring)
```

## Agent Features
- **Local buffering**: 10 GB disk buffer for network outages
- **Automatic retry**: Exponential backoff for failed transmissions
- **Multi-workspace**: Can report to multiple workspaces
- **Health monitoring**: Agent reports its own status
- **Auto-recovery**: Service restart on crashes

## Data Latency
⏱️ **Typical**: 5-10 minutes from event to queryability
⏱️ **Maximum**: Up to 15 minutes during high volume
💡 **Tip**: Use `TimeGenerated` field for event timestamp

## Validation Steps
```powershell
# Check workspace connection
Get-AzOperationalInsightsWorkspace `
    -ResourceGroupName "rg-monitoring-prod" `
    -Name "law-monitoring-prod"

# Verify agent installation (on VM)
# Navigate to: C:\Program Files\Microsoft Monitoring Agent\MMA
# Check: Microsoft Monitoring Agent Control Panel
```

## Cost Optimization
```
Data Tiers:
├── Basic Logs: Low-cost, limited queries (search only)
└── Analytics Logs: Full KQL capability, higher cost

Commitment Tiers:
├── Pay-as-you-go: $2.30/GB
├── 100 GB/day tier: $1.93/GB (16% discount)
└── 500 GB/day tier: $1.64/GB (29% discount)
```

## Critical Notes
- ⚠️ **Workspace keys are secrets**: Store securely, rotate periodically
- 💡 **Infrastructure as Code**: Use ARM templates/PowerShell for consistency
- 🎯 **Collection rules**: Filter before ingestion to reduce costs
- 📊 **Monitor agent health**: Check Heartbeat table regularly
- 🔄 **Agent updates**: Plan for periodic MMA updates

## Quick Reference
```bash
# Query workspace ID
az monitor log-analytics workspace show \
    --resource-group "rg-monitoring-prod" \
    --workspace-name "law-monitoring-prod" \
    --query customerId -o tsv

# List configured performance counters
az monitor log-analytics workspace data-source list \
    --resource-group "rg-monitoring-prod" \
    --workspace-name "law-monitoring-prod"
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-tools-track-usage-flow/4-explore-azure-monitor-log-analytics)
