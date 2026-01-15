# Explore Azure Automation State Configuration (DSC)

**Azure Automation State Configuration (DSC)** is an Azure cloud-based implementation of PowerShell DSC, available as part of Azure Automation. Azure Automation State Configuration allows you to write, manage, and compile PowerShell DSC configurations, import DSC Resources, and assign configurations to target nodes, **all in the cloud**.

This eliminates the need to set up and maintain your own DSC pull server infrastructure while providing enterprise-grade features like centralized management, compliance reporting, and integration with Azure services.

## Why Use Azure Automation State Configuration?

### Built-in Pull Server

Azure Automation State Configuration provides a **DSC pull server** similar to the Windows Feature DSC service so that target nodes automatically receive configurations, conform to the desired state, and report back on their compliance. The built-in pull server in Azure Automation **eliminates the need to set up and maintain your own pull server**.

**Traditional DSC pull server (self-hosted)**:
```
Your responsibilities:
❌ Deploy Windows Server for pull server
❌ Install IIS with OData endpoint
❌ Configure HTTPS with certificates
❌ Set up SQL Server for compliance database
❌ Configure firewall rules
❌ Maintain server (patching, updates, backups)
❌ Scale for high availability (multiple servers)
❌ Monitor pull server health
❌ Troubleshoot connectivity issues

Estimated setup time: 2-4 days
Ongoing maintenance: 4-8 hours/month
```

**Azure Automation State Configuration (managed service)**:
```
Your responsibilities:
✅ Upload DSC configurations
✅ Onboard nodes
✅ Monitor compliance

Azure handles:
✅ Pull server infrastructure
✅ High availability (99.9% SLA)
✅ Automatic scaling
✅ Security (HTTPS, encryption)
✅ Patching and updates
✅ Monitoring and diagnostics

Setup time: 15 minutes
Ongoing maintenance: ~1 hour/month
```

### Centralized Management

You can **manage all your DSC configurations, resources, and target nodes from the Azure portal or PowerShell**. This centralized management simplifies configuration deployment and monitoring across your entire infrastructure.

**Central dashboard features**:
- 📊 Node compliance overview (compliant/non-compliant/pending)
- 📈 Compliance trends over time
- 🔔 Drift detection alerts
- 📝 Configuration version history
- 🔍 Node-level compliance details
- 📦 DSC module management
- 🚀 Configuration compilation status

**Example: Managing 500 nodes**:
```
Azure Portal → Automation Account → State Configuration (DSC)

Nodes Tab:
├─ Total Nodes: 500
├─ Compliant: 487 (97.4%) ✅
├─ Non-Compliant: 8 (1.6%) ❌
├─ Pending: 5 (1.0%) ⏳
└─ Failed: 0 (0%)

Configurations Tab:
├─ WebServerConfig (150 nodes)
├─ DatabaseServerConfig (200 nodes)
├─ AppServerConfig (100 nodes)
└─ MonitoringAgentConfig (50 nodes)

Modules Tab:
├─ PSDesiredStateConfiguration (built-in)
├─ xWebAdministration v3.2.0
├─ xNetworking v5.7.0
└─ xSQLServer v15.2.0

Compliance Dashboard:
├─ Last 24 hours: 23 drift events detected, 21 auto-remediated ✅
├─ Top drift causes:
│   1. IIS service stopped (8 events)
│   2. Registry key modified (5 events)
│   3. Unauthorized software installed (3 events)
└─ Compliance rate: 97.4% (target: 95%)
```

### Integration with Log Analytics

Nodes managed with Azure Automation State Configuration send **detailed reporting status data** to the built-in pull server. You can configure Azure Automation State Configuration to send this data to your **Log Analytics workspace** for advanced monitoring, alerting, and compliance reporting.

**Log Analytics integration benefits**:

```kusto
// Query: Find all non-compliant nodes in last 24 hours
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.AUTOMATION"
| where Category == "DscNodeStatus"
| where Status == "NonCompliant"
| where TimeGenerated > ago(24h)
| summarize Count=count() by NodeName_s, ConfigurationName_s
| order by Count desc

// Results:
// NodeName        ConfigurationName       Count
// ------------    ------------------      -----
// WebServer05     WebServerConfig         12
// AppServer12     AppServerConfig         8
// DBServer03      DatabaseServerConfig    3

// Query: Top drift resource types
AzureDiagnostics
| where Category == "DscNodeStatus"
| where OperationName == "DscResourceStatusData"
| extend ResourceType = parse_json(ResourceInfo_s).ResourceName
| summarize DriftCount=count() by tostring(ResourceType)
| order by DriftCount desc

// Results:
// ResourceType          DriftCount
// ----------------      ----------
// Service               45
// Registry              28
// WindowsFeature        12
// File                  8
```

**Create alerts**:
```powershell
# Alert when node is non-compliant for > 1 hour
$condition = New-AzScheduledQueryRuleCondition `
    -Query "AzureDiagnostics | where Status == 'NonCompliant' | where TimeGenerated > ago(1h)" `
    -TimeAggregation "Count" `
    -Operator "GreaterThan" `
    -Threshold 0

New-AzScheduledQueryRuleAlertingAction `
    -AznsAction @{
        ActionGroup = @("/subscriptions/.../actionGroups/ops-team")
        EmailSubject = "DSC Node Non-Compliant"
    } `
    -Trigger $condition
```

### Additional Benefits

**1. No Infrastructure Management**:
- No VMs to provision for pull server
- No patching or maintenance windows
- No capacity planning (Azure scales automatically)

**2. Global Availability**:
- Pull server available in all Azure regions
- Low latency (nodes pull from nearest region)
- 99.9% SLA (high availability built-in)

**3. Security**:
- HTTPS by default (encrypted communication)
- Azure AD authentication
- RBAC for configuration management
- Credentials stored in encrypted Automation account

**4. Cost-Effective**:
- No VM costs for pull server
- Pay per node (first 5 nodes free)
- No SQL licensing for compliance database

**5. Integration**:
- Azure Monitor (alerting)
- Log Analytics (advanced queries)
- Azure Policy (governance)
- Azure DevOps (CI/CD for configurations)

## How Azure Automation State Configuration Works

The general process for how Azure Automation State Configuration works is as follows:

### Step 1: Create a PowerShell Configuration Script

Create a PowerShell script with the **configuration element**. This script defines the desired state for your target nodes using DSC syntax.

**Example configuration**:
```powershell
Configuration WebServerConfig {
    param(
        [Parameter(Mandatory)]
        [string]$WebsiteName,
        
        [Parameter(Mandatory)]
        [int]$Port = 80
    )
    
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    Import-DscResource -ModuleName xWebAdministration
    
    Node $AllNodes.NodeName {
        # Install IIS
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
            IncludeAllSubFeature = $true
        }
        
        # Stop default website
        xWebsite DefaultSite {
            Ensure = "Present"
            Name = "Default Web Site"
            State = "Stopped"
            DependsOn = "[WindowsFeature]IIS"
        }
        
        # Create custom website
        xWebsite CustomSite {
            Ensure = "Present"
            Name = $WebsiteName
            State = "Started"
            PhysicalPath = "C:\inetpub\wwwroot"
            BindingInfo = @(
                MSFT_xWebBindingInformation {
                    Protocol = "HTTP"
                    Port = $Port
                    IPAddress = "*"
                }
            )
            DependsOn = @("[WindowsFeature]IIS", "[xWebsite]DefaultSite")
        }
        
        # Ensure IIS service running
        Service W3SVC {
            Name = "W3SVC"
            State = "Running"
            StartupType = "Automatic"
            DependsOn = "[WindowsFeature]IIS"
        }
    }
}
```

**Configuration best practices**:
- ✅ Use parameters for reusability
- ✅ Import required DSC resources
- ✅ Use `DependsOn` for resource ordering
- ✅ Add comments for complex logic
- ✅ Version control in Git/Azure DevOps

### Step 2: Upload and Compile the Configuration

Upload the script to Azure Automation and **compile the script into a MOF file**. The MOF (Managed Object Format) file is transferred to the DSC pull server, provided as part of the Azure Automation service.

**Upload configuration to Azure Automation**:
```powershell
# Option 1: PowerShell
Import-AzAutomationDscConfiguration `
    -SourcePath "C:\DSC\WebServerConfig.ps1" `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod" `
    -Published `
    -Force

# Option 2: Azure Portal
# Automation Account → State Configuration (DSC) → Configurations → + Add
# Browse to WebServerConfig.ps1 → Upload

# Option 3: Azure CLI
az automation dsc-configuration create `
    --automation-account-name "automation-prod" `
    --resource-group "rg-automation" `
    --name "WebServerConfig" `
    --location "eastus" `
    --source-path "C:\DSC\WebServerConfig.ps1"
```

**Compile configuration**:
```powershell
# Compile without parameters (uses defaults)
Start-AzAutomationDscCompilationJob `
    -ConfigurationName "WebServerConfig" `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod"

# Compile with parameters
$configData = @{
    AllNodes = @(
        @{
            NodeName = "WebServer"
            WebsiteName = "ContosoSite"
            Port = 8080
        }
    )
}

Start-AzAutomationDscCompilationJob `
    -ConfigurationName "WebServerConfig" `
    -ConfigurationData $configData `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod"

# Monitor compilation job
$job = Start-AzAutomationDscCompilationJob <params>
$job | Get-AzAutomationDscCompilationJob

# Output:
# ResourceGroupName   : rg-automation
# AutomationAccountName : automation-prod
# Id                  : 12345678-1234-1234-1234-123456789012
# CreationTime        : 1/15/2024 10:00:00 AM
# Status              : Completed
# StatusDetails       : 
# StartTime           : 1/15/2024 10:00:15 AM
# EndTime             : 1/15/2024 10:00:45 AM
# ConfigurationName   : WebServerConfig
```

**Compilation creates MOF file**:
```
WebServerConfig.ps1 (configuration script)
    ↓ Compilation
WebServerConfig.WebServer.mof (Managed Object Format)
    ↓ Stored in
Azure Automation State Configuration (pull server)
```

### Step 3: Assign Configurations to Nodes

Define the nodes that will use the configuration, and then **apply the configuration**. The nodes automatically pull the configuration from the DSC pull server and apply it to maintain the desired state.

**Workflow diagram**:

```
┌────────────────────────────────────────────────────────────────┐
│          Azure Automation State Configuration                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  DSC Configurations (.ps1 scripts)                       │  │
│  │  - WebServerConfig.ps1                                   │  │
│  │  - DatabaseServerConfig.ps1                              │  │
│  │  - AppServerConfig.ps1                                   │  │
│  └────────────────────┬─────────────────────────────────────┘  │
│                       │ Compilation                             │
│                       ↓                                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Node Configurations (MOF files)                         │  │
│  │  - WebServerConfig.WebServer.mof                         │  │
│  │  - DatabaseServerConfig.SqlServer.mof                    │  │
│  │  - AppServerConfig.AppServer.mof                         │  │
│  └────────────────────┬─────────────────────────────────────┘  │
│                       │ Assignment                              │
│                       ↓                                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  DSC Nodes (registered VMs)                              │  │
│  │  - WebServer01 → WebServerConfig.WebServer               │  │
│  │  - WebServer02 → WebServerConfig.WebServer               │  │
│  │  - SqlServer01 → DatabaseServerConfig.SqlServer          │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
         ↑                          ↑                      ↑
         │ Pull config every        │ Pull config         │ Pull config
         │ 30 minutes               │ every 30 min        │ every 30 min
         ↓                          ↓                      ↓
    ┌───────────┐            ┌───────────┐          ┌───────────┐
    │WebServer01│            │WebServer02│          │SqlServer01│
    │  (Azure)  │            │  (Azure)  │          │ (On-prem) │
    └───────────┘            └───────────┘          └───────────┘
         ↕                          ↕                      ↕
    Check drift                Check drift            Check drift
    every 15 min              every 15 min           every 15 min
```

**Onboard Azure VM**:
```powershell
# Register Azure VM to Azure Automation State Configuration
Register-AzAutomationDscNode `
    -AzureVMName "WebServer01" `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod" `
    -NodeConfigurationName "WebServerConfig.WebServer" `
    -ConfigurationMode "ApplyAndAutoCorrect" `
    -RebootNodeIfNeeded $true

# Verify registration
Get-AzAutomationDscNode `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod" `
    -Name "WebServer01"

# Output:
# Name              : WebServer01
# Status            : Compliant
# NodeConfigurationName : WebServerConfig.WebServer
# LastSeen          : 1/15/2024 10:30:00 AM
# ConfigurationMode : ApplyAndAutoCorrect
# RebootNodeIfNeeded : True
```

**Onboard on-premises/non-Azure VM**:
```powershell
# Step 1: Get registration info from Azure Automation
$registrationInfo = Get-AzAutomationRegistrationInfo `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod"

$registrationUrl = $registrationInfo.Endpoint
$registrationKey = $registrationInfo.PrimaryKey

# Step 2: On the target VM, configure LCM
[DSCLocalConfigurationManager()]
Configuration OnboardToAzureAutomation {
    Node localhost {
        Settings {
            RefreshMode = "Pull"
            ConfigurationMode = "ApplyAndAutoCorrect"
            RebootNodeIfNeeded = $true
            ActionAfterReboot = "ContinueConfiguration"
            RefreshFrequencyMins = 30
            ConfigurationModeFrequencyMins = 15
        }
        
        ConfigurationRepositoryWeb AzureAutomation {
            ServerURL = $registrationUrl
            RegistrationKey = $registrationKey
        }
        
        ResourceRepositoryWeb AzureAutomation {
            ServerURL = $registrationUrl
            RegistrationKey = $registrationKey
        }
        
        ReportServerWeb AzureAutomation {
            ServerURL = $registrationUrl
            RegistrationKey = $registrationKey
        }
    }
}

# Step 3: Apply LCM configuration
OnboardToAzureAutomation -OutputPath "C:\DSC\LCM"
Set-DscLocalConfigurationManager -Path "C:\DSC\LCM" -Verbose

# Step 4: Verify registration (run on target VM)
Get-DscLocalConfigurationManager

# Output:
# ConfigurationMode : ApplyAndAutoCorrect
# RefreshMode       : Pull
# RebootNodeIfNeeded : True
# RefreshFrequencyMins : 30
# ConfigurationModeFrequencyMins : 15
```

**Assign configuration to node**:
```powershell
# After node is registered, assign configuration
Set-AzAutomationDscNode `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod" `
    -NodeName "WebServer01" `
    -NodeConfigurationName "WebServerConfig.WebServer"

# Node will pull configuration within 30 minutes (or force immediate pull)
# On target VM:
Update-DscConfiguration -Wait -Verbose
```

## Understanding MOF Files

**MOF (Managed Object Format)** is a text-based format for representing management information. In DSC, the **MOF file is the compiled configuration** that gets applied to target nodes.

**What's in a MOF file**:
```mof
/*
@TargetNode='WebServer01'
@GeneratedBy=Administrator
@GenerationDate=01/15/2024 10:00:00
@GenerationHost=automation-prod
*/

instance of MSFT_WindowsFeature as $MSFT_WindowsFeature1ref
{
    ResourceID = "[WindowsFeature]IIS";
    Name = "Web-Server";
    Ensure = "Present";
    IncludeAllSubFeature = True;
    ModuleName = "PSDesiredStateConfiguration";
    ModuleVersion = "1.0";
    ConfigurationName = "WebServerConfig";
};

instance of MSFT_ServiceResource as $MSFT_ServiceResource1ref
{
    ResourceID = "[Service]W3SVC";
    Name = "W3SVC";
    State = "Running";
    StartupType = "Automatic";
    ModuleName = "PSDesiredStateConfiguration";
    DependsOn = {
        "[WindowsFeature]IIS"
    };
    ConfigurationName = "WebServerConfig";
};
```

**MOF characteristics**:
- ✅ Human-readable (text format)
- ✅ Platform-independent (CIM standard)
- ✅ Contains all resource configurations
- ✅ Includes dependencies and ordering
- ✅ Generated automatically from PS configuration

**MOF workflow**:
```
Configuration Script (.ps1)
    ↓ Author in PowerShell ISE/VS Code
    ↓ Upload to Azure Automation
    ↓ Compile
MOF File (.mof)
    ↓ Store in Azure Automation pull server
    ↓ Node pulls MOF
LCM reads MOF
    ↓ For each resource in MOF:
    ↓   - Test current state
    ↓   - If drift: Apply desired state
    ↓ Report compliance
```

For more information about Managed Object Format (MOF) files, visit [Managed Object Format (MOF) file](https://learn.microsoft.com/en-us/windows/win32/wmisdk/managed-object-format--mof-).

## Complete Workflow Example

**Scenario**: Deploy web server configuration to 50 Azure VMs

```powershell
# Step 1: Import DSC modules (if needed)
$modules = @("xWebAdministration", "xNetworking")
foreach ($module in $modules) {
    New-AzAutomationModule `
        -ResourceGroupName "rg-automation" `
        -AutomationAccountName "automation-prod" `
        -Name $module `
        -ContentLinkUri "https://www.powershellgallery.com/api/v2/package/$module"
}

# Step 2: Upload configuration
Import-AzAutomationDscConfiguration `
    -SourcePath "C:\DSC\WebServerConfig.ps1" `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod" `
    -Published

# Step 3: Compile configuration
$compileJob = Start-AzAutomationDscCompilationJob `
    -ConfigurationName "WebServerConfig" `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod"

# Wait for compilation
$compileJob | Get-AzAutomationDscCompilationJob -Wait

# Step 4: Onboard VMs
$vms = Get-AzVM -ResourceGroupName "rg-web-servers" | Select-Object -First 50

foreach ($vm in $vms) {
    Write-Host "Onboarding $($vm.Name)..."
    
    Register-AzAutomationDscNode `
        -AzureVMName $vm.Name `
        -ResourceGroupName "rg-automation" `
        -AutomationAccountName "automation-prod" `
        -NodeConfigurationName "WebServerConfig.WebServer" `
        -ConfigurationMode "ApplyAndAutoCorrect" `
        -RebootNodeIfNeeded $true
}

# Step 5: Monitor compliance
Start-Sleep -Seconds 1800  # Wait 30 minutes for initial pull

$nodes = Get-AzAutomationDscNode `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod"

$complianceSummary = $nodes | Group-Object Status | Select-Object Name, Count
$complianceSummary | Format-Table

# Output:
# Name          Count
# ----          -----
# Compliant     48
# Pending       2
# NonCompliant  0
```

## Key Takeaways

1. **Managed service**: Azure handles pull server infrastructure (no VMs to maintain)
2. **Centralized**: Single dashboard for all nodes, configurations, compliance
3. **Scalable**: 5 to 5,000+ nodes without infrastructure changes
4. **Integrated**: Log Analytics, Azure Monitor, Azure DevOps
5. **Secure**: HTTPS, Azure AD, RBAC, encrypted credentials
6. **Cost-effective**: No VM costs, first 5 nodes free

## Next Steps

Now that you understand Azure Automation State Configuration and its benefits, let's dive into **DSC configuration files** to learn the syntax, structure, and best practices for authoring configurations.

---

**Module**: Implement Desired State Configuration (DSC)  
**Unit**: 4 of 8  
**Duration**: 3 minutes  
**Next**: [Unit 5: Examine DSC Configuration File](#)  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/implement-desired-state-configuration-dsc/4-explore-azure-automation
