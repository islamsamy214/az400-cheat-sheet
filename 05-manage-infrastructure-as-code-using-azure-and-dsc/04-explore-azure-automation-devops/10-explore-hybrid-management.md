# Explore Hybrid Management

## Overview

Hybrid Runbook Workers extend Azure Automation capabilities beyond Azure to your on-premises datacenters, private clouds, and other cloud providers. Instead of runbooks executing only in Azure sandbox environments, they can run directly on machines in your infrastructure, accessing local resources, databases, file servers, and applications that aren't exposed to the internet.

This unit covers the Hybrid Runbook Worker architecture, setup, configuration, and use cases for managing hybrid and multi-cloud environments from a centralized Azure Automation platform.

## What is a Hybrid Runbook Worker?

**Hybrid Runbook Worker**: An on-premises or non-Azure server that executes Azure Automation runbooks locally, providing access to resources in your datacenter while maintaining centralized management in Azure.

**Architecture Flow**:
```
Azure Automation Account (Cloud)
    ↓ (Runbook job assigned)
Hybrid Runbook Worker (On-Premises)
    ↓ (Executes locally)
On-Premises Resources (Databases, File Servers, Applications)
    ↓ (Results returned)
Azure Automation Account (Job output stored)
```

**Key Characteristics**:

1. **No Inbound Firewall Rules**: Worker initiates outbound HTTPS (port 443) connection to Azure
2. **Local Execution**: Runbooks run on the worker machine with local context
3. **Worker Groups**: Multiple workers organized into groups for load balancing and high availability
4. **Managed Identity**: Workers can use Azure Managed Identity for authentication
5. **Cross-Platform**: Supports Windows and Linux workers

## Why Hybrid Runbook Workers?

### Use Cases

**1. On-Premises Resource Management**
```
Problem: Azure Automation can't reach on-premises SQL Server
Solution: Hybrid Worker on corporate network executes database maintenance runbooks
```

**2. Legacy Systems Integration**
```
Problem: Mainframe accessible only from internal network
Solution: Hybrid Worker bridges Azure Automation with legacy systems
```

**3. Compliance and Data Sovereignty**
```
Problem: Sensitive data cannot leave on-premises environment
Solution: Hybrid Worker processes data locally, only metadata sent to Azure
```

**4. VMware/Hyper-V Management**
```
Problem: Private cloud VMs not in Azure
Solution: Hybrid Worker uses PowerCLI or Hyper-V cmdlets locally
```

**5. Physical Hardware Management**
```
Problem: Managing physical servers, network devices, storage arrays
Solution: Hybrid Worker has physical network access to devices
```

**6. Multi-Cloud Orchestration**
```
Problem: Manage AWS/GCP resources alongside Azure
Solution: Hybrid Worker with AWS CLI/GCP SDK installed
```

## Hybrid Runbook Worker Architecture

### Components

**1. Azure Automation Account**
- Stores runbooks, schedules, assets
- Job queue and orchestration
- Centralized monitoring and logging

**2. Log Analytics Workspace**
- Worker registration and health monitoring
- Log collection and analysis
- Agent management

**3. Hybrid Worker Machine**
- Windows Server 2012 R2+ or Linux
- Azure Monitor Agent (AMA) or Log Analytics Agent (legacy)
- Hybrid Worker Extension
- Local execution environment

**4. Worker Group**
- Logical grouping of workers
- Load balancing across workers
- Failover and redundancy

### Communication Flow

**Outbound Only (No Inbound Ports)**:
```
Hybrid Worker → Azure Automation (HTTPS 443)
    - Job requests (poll every 30 seconds)
    - Job status updates
    - Job output/logs

Hybrid Worker → Log Analytics (HTTPS 443)
    - Heartbeat (every 5 minutes)
    - Health metrics
    - Log data
```

**Firewall Requirements**:
```
Outbound HTTPS (443) to:
- *.azure-automation.net
- *.ods.opinsights.azure.com (Log Analytics)
- *.oms.opinsights.azure.com (Log Analytics)
- *.blob.core.windows.net (Package downloads)
```

**No Inbound Requirements**:
- No ports opened on worker
- No VPN required
- No Express Route required
- Worker initiates all connections

## Installing Hybrid Runbook Workers

### Prerequisites

**System Requirements**:
- **Windows**: Windows Server 2012 R2 or later, Windows 10/11 (1809+)
- **Linux**: Ubuntu 18.04+, RHEL 7/8, SUSE 12/15
- **Memory**: 4 GB RAM minimum
- **CPU**: 2 cores minimum
- **.NET Framework**: 4.6.2 or later (Windows)
- **PowerShell**: 5.1 or later (Windows), PowerShell 7+ (Linux)

**Network Requirements**:
- Outbound HTTPS (443) access to Azure endpoints
- Proxy support available if required

**Permissions**:
- Local administrator on worker machine
- Contributor on Automation account
- Contributor on Log Analytics workspace

### Method 1: Azure Portal (Windows)

**Step-by-Step**:

1. **Navigate to Automation Account**
   - Azure Portal → Automation Account
   - Select "Hybrid worker groups" under "Process Automation"

2. **Create Worker Group**
   - Click "+ Create hybrid worker group"
   - Name: `onprem-workers`
   - Type: User hybrid worker group
   - Click "Create"

3. **Add Worker to Group**
   - Select the worker group
   - Click "+ Add hybrid worker"
   - Select target machine (must have Azure Arc or be Azure VM)
   - Configure credentials
   - Click "Add"

4. **Verify Installation**
   - Worker appears in group
   - Status: "Healthy"
   - Last heartbeat: < 5 minutes ago

### Method 2: PowerShell (Windows)

**Install Hybrid Worker via PowerShell**:

```powershell
# 1. Install Azure Monitor Agent (if not already installed)
Invoke-WebRequest -Uri "https://aka.ms/AzureMonitorAgent-windows" -OutFile "AMA-Windows.msi"
Start-Process msiexec.exe -ArgumentList "/i AMA-Windows.msi /quiet" -Wait

# 2. Connect machine to Log Analytics workspace
$WorkspaceId = "YOUR_WORKSPACE_ID"
$WorkspaceKey = "YOUR_WORKSPACE_KEY"

$MMAPath = "${env:ProgramFiles}\Microsoft Monitoring Agent\Agent\Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridWorker.dll"

# Import Hybrid Worker module
Import-Module $MMAPath

# Register worker
Add-HybridRunbookWorker -GroupName "onprem-workers" `
    -EndPoint "https://YOUR_REGION.agentsvc.azure-automation.net/accounts/YOUR_AUTOMATION_ACCOUNT_ID" `
    -Token "YOUR_AUTOMATION_ACCOUNT_PRIMARY_KEY"

# Verify registration
Get-HybridRunbookWorker
```

### Method 3: Azure Arc (Recommended for Non-Azure Machines)

**Arc-Enabled Server + Hybrid Worker Extension**:

```powershell
# 1. Install Azure Arc agent (on-premises server)
# Download and run Arc installation script from Azure Portal

# 2. Install Hybrid Worker extension via Azure CLI
az connectedmachine extension create `
    --machine-name "onprem-server-01" `
    --resource-group "rg-hybrid-workers" `
    --name "HybridWorkerExtension" `
    --type "HybridWorkerExtension" `
    --publisher "Microsoft.Azure.Automation.HybridWorker" `
    --settings '{
        "AutomationAccountURL": "https://eastus2.agentsvc.azure-automation.net/accounts/12345678-1234-1234-1234-123456789012"
    }'

# 3. Add Arc machine to worker group
az automation hrwg hrw create `
    --automation-account-name "automation-account-prod" `
    --resource-group "rg-automation" `
    --hybrid-runbook-worker-group-name "onprem-workers" `
    --hybrid-runbook-worker-id "onprem-server-01" `
    --vm-resource-id "/subscriptions/SUBSCRIPTION_ID/resourceGroups/rg-hybrid-workers/providers/Microsoft.HybridCompute/machines/onprem-server-01"
```

### Method 4: DSC (Automated Deployment)

**Deploy Hybrid Worker via DSC Configuration**:

```powershell
Configuration InstallHybridWorker {
    param(
        [string]$AutomationAccountUrl,
        [string]$WorkerGroupName,
        [PSCredential]$RegistrationCredential
    )
    
    Import-DscResource -ModuleName xPSDesiredStateConfiguration
    Import-DscResource -ModuleName HybridRunbookWorkerDsc
    
    Node localhost {
        # Install Hybrid Worker prerequisites
        WindowsFeature NET-Framework-Core {
            Name = "NET-Framework-Core"
            Ensure = "Present"
        }
        
        # Configure Hybrid Worker
        HybridRunbookWorker Worker {
            Ensure = "Present"
            EndPoint = $AutomationAccountUrl
            GroupName = $WorkerGroupName
            Credential = $RegistrationCredential
            DependsOn = "[WindowsFeature]NET-Framework-Core"
        }
    }
}

# Compile and apply configuration
$configData = @{
    AllNodes = @(
        @{ NodeName = "localhost"; PSDscAllowPlainTextPassword = $true }
    )
}

InstallHybridWorker -AutomationAccountUrl "https://eastus.agentsvc.azure-automation.net/accounts/GUID" `
    -WorkerGroupName "onprem-workers" `
    -RegistrationCredential $cred `
    -ConfigurationData $configData `
    -OutputPath "C:\DSC"

Start-DscConfiguration -Path "C:\DSC" -Wait -Force -Verbose
```

## Worker Groups

### Concept

**Worker Group**: Logical collection of Hybrid Runbook Workers that receive jobs as a unit.

**Purpose**:
- **Load Balancing**: Jobs distributed across workers in group
- **High Availability**: If one worker fails, job routed to another
- **Geographic Distribution**: Workers in different locations for regional access
- **Role-Based Separation**: Separate groups for different environments or functions

### Creating Worker Groups

**Via Azure Portal**:
```
Automation Account → Hybrid worker groups → + Create hybrid worker group
Name: production-workers
Type: User
Click "Create"
```

**Via PowerShell**:
```powershell
# Create worker group
New-AzAutomationHybridRunbookWorkerGroup `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-account-prod" `
    -Name "production-workers"

# Add worker to group
Add-AzAutomationHybridRunbookWorker `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-account-prod" `
    -HybridRunbookWorkerGroupName "production-workers" `
    -WorkerId "worker-01" `
    -VmResourceId "/subscriptions/.../machines/worker-01"
```

### Targeting Worker Groups

**Assign Runbook to Specific Worker Group**:

```powershell
# Start runbook on specific worker group (PowerShell)
Start-AzAutomationRunbook `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-account-prod" `
    -Name "Backup-OnPremDatabase" `
    -RunOn "production-workers" `
    -Parameters @{
        DatabaseServer = "sql-server-01"
        DatabaseName = "ProductionDB"
    }
```

**Runbook with Worker Group Parameter**:
```powershell
# In Azure Portal, when starting runbook manually:
# 1. Navigate to runbook
# 2. Click "Start"
# 3. Select "Run on": Hybrid Worker
# 4. Select worker group: production-workers
# 5. Enter parameters
# 6. Click "OK"
```

### High Availability Configuration

**Multiple Workers in Group**:

```
Worker Group: production-workers
    ├── worker-01 (Active)
    ├── worker-02 (Active)
    └── worker-03 (Active)

Job Assignment:
- Job 1 → worker-01
- Job 2 → worker-02
- Job 3 → worker-03
- Job 4 → worker-01 (round-robin)

Failover:
- If worker-01 fails: Jobs routed to worker-02 and worker-03
- When worker-01 recovers: Resumes receiving jobs
```

**Configure Multiple Workers**:
```powershell
# Add 3 workers to same group
$workers = @("worker-01", "worker-02", "worker-03")

foreach ($worker in $workers) {
    Add-AzAutomationHybridRunbookWorker `
        -ResourceGroupName "rg-automation" `
        -AutomationAccountName "automation-account-prod" `
        -HybridRunbookWorkerGroupName "production-workers" `
        -WorkerId $worker `
        -VmResourceId "/subscriptions/.../machines/$worker"
}
```

## Runbook Execution on Hybrid Workers

### Differences from Azure Sandbox

| Aspect | Azure Sandbox | Hybrid Worker |
|--------|---------------|---------------|
| **Execution Location** | Azure datacenter | On-premises/your infrastructure |
| **Resource Access** | Azure resources only | Local + Azure resources |
| **Network** | Internet-facing | Private network access |
| **Modules** | Pre-installed Az modules | Manually installed modules |
| **Runtime** | 3-hour limit (fair share) | No time limit |
| **Credentials** | Automation assets | Local + Automation assets |
| **OS** | Windows Server (managed by Azure) | Your OS (Windows/Linux) |

### Installing Modules on Workers

**Hybrid Workers don't inherit Azure sandbox modules. You must install required modules manually.**

**Install Az Modules**:
```powershell
# On Hybrid Worker machine
Install-PackageProvider -Name NuGet -Force
Install-Module -Name Az -AllowClobber -Force -Scope AllUsers

# Verify installation
Get-Module -Name Az -ListAvailable
```

**Install Custom Modules**:
```powershell
# On Hybrid Worker
Install-Module -Name CustomModule -Repository PSGallery -Force

# Or install from local file
Import-Module "C:\Modules\CustomModule.psd1"
```

**Automate Module Installation via Runbook**:
```powershell
# Runbook: Install-RequiredModules
workflow Install-RequiredModules {
    param([string[]]$ModuleNames)
    
    InlineScript {
        foreach ($module in $using:ModuleNames) {
            if (-not (Get-Module -Name $module -ListAvailable)) {
                Write-Output "Installing module: $module"
                Install-Module -Name $module -Force -AllowClobber -Scope AllUsers
            }
            else {
                Write-Output "Module already installed: $module"
            }
        }
    }
}

# Run on all workers
Start-AzAutomationRunbook -Name "Install-RequiredModules" `
    -RunOn "production-workers" `
    -Parameters @{ ModuleNames = @("Az.Compute", "Az.Storage", "SqlServer") }
```

### Accessing Local Resources

**Example: SQL Server Backup**:
```powershell
# Runbook: Backup-SqlDatabase
# Runs on Hybrid Worker with access to SQL Server

workflow Backup-SqlDatabase {
    param(
        [string]$SqlServer = "sql-server-01.contoso.local",
        [string]$DatabaseName = "ProductionDB",
        [string]$BackupPath = "\\backup-server\sql-backups"
    )
    
    InlineScript {
        # Import SQL module (must be installed on Hybrid Worker)
        Import-Module SqlServer
        
        # Get credential from Automation Account
        $sqlCred = Get-AutomationPSCredential -Name "SqlAdminCred"
        
        # Backup database (direct access to on-prem SQL Server)
        $timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
        $backupFile = Join-Path $using:BackupPath "$using:DatabaseName_$timestamp.bak"
        
        Backup-SqlDatabase -ServerInstance $using:SqlServer `
            -Database $using:DatabaseName `
            -BackupFile $backupFile `
            -Credential $sqlCred `
            -CompressionOption On
        
        Write-Output "✓ Backup completed: $backupFile"
        
        # Return backup file info
        Get-Item $backupFile | Select-Object Name, Length, CreationTime
    }
}
```

**Example: File Server Management**:
```powershell
# Runbook: Cleanup-FileServer
# Runs on Hybrid Worker in domain

workflow Cleanup-FileServer {
    param(
        [string]$FileServerPath = "\\fileserver01\shares\temp",
        [int]$DaysOld = 30
    )
    
    InlineScript {
        # Access local file share
        $cutoffDate = (Get-Date).AddDays(-$using:DaysOld)
        
        $oldFiles = Get-ChildItem -Path $using:FileServerPath -Recurse -File |
            Where-Object { $_.LastWriteTime -lt $cutoffDate }
        
        $totalSize = ($oldFiles | Measure-Object -Property Length -Sum).Sum
        $sizeGB = [Math]::Round($totalSize / 1GB, 2)
        
        Write-Output "Found $($oldFiles.Count) files older than $using:DaysOld days"
        Write-Output "Total size: $sizeGB GB"
        
        # Delete files
        $oldFiles | Remove-Item -Force
        
        Write-Output "✓ Cleanup completed"
    }
}
```

## Security Considerations

### Credential Management

**Use Automation Credentials**:
```powershell
# Store credentials in Automation Account (Azure Portal)
# Automation Account → Credentials → + Add credential

# Access in runbook on Hybrid Worker
workflow Use-Credentials {
    InlineScript {
        # Credential automatically decrypted on worker
        $cred = Get-AutomationPSCredential -Name "DomainAdmin"
        
        # Use for on-premises authentication
        Invoke-Command -ComputerName "server01" -Credential $cred -ScriptBlock {
            Get-Service
        }
    }
}
```

**Managed Identity for Azure Resources**:
```powershell
# Hybrid Worker can use Managed Identity (if Arc-enabled)
workflow Use-ManagedIdentity {
    InlineScript {
        # Authenticate to Azure using Managed Identity
        Connect-AzAccount -Identity
        
        # Access Azure resources
        Get-AzVM -ResourceGroupName "rg-prod"
    }
}
```

### Least Privilege

**Worker Service Account**:
- Create dedicated service account for Hybrid Worker
- Grant minimum permissions required
- Use group-managed service accounts (gMSA) on Windows

**Runbook-Specific Permissions**:
- Each runbook should use credentials with least privilege
- Separate credentials for different operations
- Regularly rotate credentials

## Monitoring and Troubleshooting

### Health Monitoring

**Check Worker Status**:
```powershell
# Get worker health
Get-AzAutomationHybridWorkerGroup `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-account-prod" |
    Select-Object Name, @{N="Workers";E={$_.Workers.Count}}, @{N="Status";E={$_.Workers.Status}}

# Check last heartbeat
Get-AzAutomationHybridWorker `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-account-prod" `
    -HybridRunbookWorkerGroupName "production-workers" |
    Select-Object Name, LastSeenDateTime, State
```

### Common Issues

**Issue 1: Worker Not Reporting**

**Symptoms**: Worker shows "Not Responding" or no heartbeat

**Causes**:
- Firewall blocking outbound 443
- Log Analytics Agent stopped
- Network connectivity issues

**Solution**:
```powershell
# On Hybrid Worker machine:

# 1. Check Log Analytics Agent
Get-Service -Name HealthService | Select-Object Name, Status

# If stopped, start it
Start-Service -Name HealthService

# 2. Test connectivity
Test-NetConnection -ComputerName "eastus.agentsvc.azure-automation.net" -Port 443

# 3. Check agent logs
Get-Content "C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State\Logs\MMA.log" -Tail 50
```

**Issue 2: Module Not Found**

**Symptoms**: Runbook fails with "Module 'ModuleName' not found"

**Solution**:
```powershell
# On Hybrid Worker, install missing module
Install-Module -Name ModuleName -Force -Scope AllUsers

# Verify installation
Get-Module -Name ModuleName -ListAvailable
```

**Issue 3: Runbook Fails on Worker but Works in Azure**

**Causes**:
- Different PowerShell version
- Missing dependencies
- Different OS (Windows vs Linux)

**Solution**:
- Check PowerShell version: `$PSVersionTable`
- Install missing prerequisites
- Test runbook locally on worker before scheduling

## Best Practices

### 1. Multiple Workers per Group

```powershell
# ✅ GOOD: 3 workers for high availability
Worker Group: production-workers
    ├── worker-01
    ├── worker-02
    └── worker-03

# ❌ BAD: Single worker (no redundancy)
Worker Group: production-workers
    └── worker-01 (single point of failure)
```

### 2. Separate Groups by Environment

```
Worker Groups:
    ├── dev-workers (development environment access)
    ├── staging-workers (staging environment access)
    └── production-workers (production environment access)
```

### 3. Keep Workers Updated

```powershell
# Regularly update workers
# On each worker:
Update-Module -Name Az -Force
Update-Help -Force

# Check for OS updates
Install-WindowsUpdate -AcceptAll -Install
```

### 4. Monitor Worker Health

```powershell
# Create monitoring runbook
workflow Monitor-HybridWorkers {
    $workers = Get-AzAutomationHybridWorker -ResourceGroupName "rg-automation" `
        -AutomationAccountName "automation-account-prod" `
        -HybridRunbookWorkerGroupName "production-workers"
    
    foreach ($worker in $workers) {
        $lastSeen = $worker.LastSeenDateTime
        $minutesSinceHeartbeat = ((Get-Date) - $lastSeen).TotalMinutes
        
        if ($minutesSinceHeartbeat -gt 10) {
            Send-AlertEmail -Subject "Hybrid Worker Alert" `
                -Body "Worker $($worker.Name) not responding (last seen: $lastSeen)"
        }
    }
}

# Schedule to run every 15 minutes
```

### 5. Document Worker Dependencies

```markdown
# Hybrid Worker: production-workers

## Installed Software
- PowerShell 7.2
- Az Modules (all)
- SqlServer Module
- VMware PowerCLI
- Custom modules: CompanyAutomation

## Network Access
- SQL Servers: sql-01, sql-02, sql-03
- File Servers: fs-01, fs-02
- VMware vCenter: vcenter.contoso.local

## Credentials Used
- SqlAdminCred (SQL Server access)
- DomainAdmin (Active Directory)
- VMwareAdmin (vCenter)

## Scheduled Runbooks
- Backup-SqlDatabases (Daily 2 AM)
- Cleanup-TempFiles (Daily 3 AM)
- Restart-Services (Sunday 4 AM)
```

## Next Steps

With hybrid management capabilities mastered, proceed to **Unit 11: Examine Checkpoint and Parallel Processing** to explore advanced PowerShell Workflow features for optimizing long-running automation tasks across both Azure and hybrid workers.

**Key Takeaways**:
- ✅ Hybrid Workers extend automation to on-premises and other clouds
- ✅ No inbound firewall rules required (outbound HTTPS only)
- ✅ Worker groups provide load balancing and high availability
- ✅ Runbooks access local resources directly
- ✅ Modules must be installed manually on workers

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/10-explore-hybrid-management
