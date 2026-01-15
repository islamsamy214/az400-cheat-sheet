# Explore Runbook Gallery

## Overview

The Azure Automation Runbook Gallery is a centralized repository of pre-built runbooks created by Microsoft and the community. Instead of building automation from scratch, you can browse hundreds of ready-to-use runbooks, import them into your Automation account, and customize them for your specific needs.

Think of the Runbook Gallery as a "marketplace" for automation—find proven solutions, learn from expert implementations, and accelerate your automation journey by standing on the shoulders of the community.

This unit covers how to find, evaluate, import, and customize runbooks from the gallery, along with best practices for leveraging community contributions.

## What is the Runbook Gallery?

The **Runbook Gallery** is Microsoft's curated collection of automation runbooks available through:

- **Azure Portal**: Integrated browsing and import directly in your Automation account
- **GitHub Repository**: https://github.com/azureautomation — Source code for all gallery runbooks
- **PowerShell Gallery**: Some runbooks available as PowerShell scripts
- **Script Center**: Legacy repository (being migrated to GitHub)

### Gallery Content

**Types of Runbooks Available**:
- **Azure Management**: VM lifecycle, resource provisioning, cost optimization
- **Monitoring and Alerting**: Health checks, performance monitoring, incident response
- **Backup and Recovery**: Automated backups, disaster recovery orchestration
- **Security and Compliance**: Security audits, policy enforcement, vulnerability remediation
- **Integration**: Connecting Azure with on-premises systems, third-party services
- **Utility**: Helper functions, logging frameworks, notification systems

**Runbook Formats**:
- PowerShell runbooks (.ps1)
- PowerShell Workflow runbooks (.ps1)
- Python runbooks (.py)
- Graphical runbooks (Azure Portal visual format)

**Contributors**:
- **Microsoft**: Official runbooks maintained by Azure Automation team
- **Azure CAT (Customer Advisory Team)**: Best practices from Azure experts
- **MVPs**: Community leaders and Azure experts
- **Partners**: ISVs and consulting firms
- **Community**: Individual contributors worldwide

## Accessing the Runbook Gallery

### Method 1: Azure Portal (Recommended)

**Steps to Browse Gallery**:

1. Navigate to your Automation account
2. Click **Process Automation** → **Runbooks**
3. Click **Browse gallery**
4. Browse or search for runbooks

**Gallery Interface Features**:
- **Search Bar**: Find runbooks by keyword (e.g., "start VM", "backup", "monitoring")
- **Filter by Type**: PowerShell, PowerShell Workflow, Python, Graphical
- **Sort Options**: Relevance, Name, Published Date
- **Runbook Details**: Description, author, version, ratings, Q&A

**Example Search Queries**:
```
Search: "start stop VM schedule"
→ Returns runbooks for scheduled VM management

Search: "backup blob storage"
→ Returns runbooks for Azure Blob backup automation

Search: "security assessment"
→ Returns runbooks for security auditing

Search: "cost optimization"
→ Returns runbooks for cost management
```

### Method 2: GitHub Repository

Browse the source code directly:

**Azure Automation GitHub**:
- URL: https://github.com/azureautomation
- Repositories organized by category:
  - `runbooks` - General automation runbooks
  - `scripts` - Utility scripts
  - `modules` - Custom PowerShell modules
  - `dsc` - Desired State Configuration resources

**Advantages of GitHub Approach**:
- View complete source code before importing
- Check commit history and updates
- Submit issues and feature requests
- Fork and contribute improvements
- Review community pull requests

**Cloning Repository Locally**:

```bash
# Clone the entire runbook repository
git clone https://github.com/azureautomation/runbooks.git

# Browse locally
cd runbooks
ls -la

# Example structure:
# ├── Utility/
# │   ├── Start-AzureVMs.ps1
# │   ├── Stop-AzureVMs.ps1
# │   └── Get-AzureVMInventory.ps1
# ├── Monitoring/
# │   ├── Check-WebsiteHealth.ps1
# │   └── Monitor-AzureServices.ps1
# └── Backup/
#     ├── Backup-AzureVMs.ps1
#     └── Backup-StorageAccounts.ps1
```

### Method 3: PowerShell Gallery

Some runbooks available as PowerShell scripts:

```powershell
# Search PowerShell Gallery for Azure Automation scripts
Find-Script -Tag "AzureAutomation" | Select-Object Name, Description

# Download specific script
Save-Script -Name "Start-AzureV2VMs" -Path "C:\AutomationScripts"

# Review and import to Automation account
Import-AzAutomationRunbook `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Path "C:\AutomationScripts\Start-AzureV2VMs.ps1" `
    -Type PowerShell
```

## Evaluating Gallery Runbooks

Before importing, evaluate runbooks for quality and fit:

### Evaluation Checklist

**1. Functionality Match**:
- ✅ Does it solve your specific problem?
- ✅ Does it support your Azure resources (VMs, Storage, SQL, etc.)?
- ✅ Does it handle your target environment (cloud-only, hybrid, multi-cloud)?

**2. Code Quality**:
- ✅ Is the code well-commented and documented?
- ✅ Does it follow PowerShell/Python best practices?
- ✅ Does it include error handling?
- ✅ Is it parameterized (not hardcoded values)?

**3. Security**:
- ❌ Does it contain hardcoded credentials? (Red flag)
- ✅ Does it use Automation credentials/variables properly?
- ✅ Does it follow least privilege principles?
- ✅ Is it from a trusted source (Microsoft, MVP, verified contributor)?

**4. Maintenance and Support**:
- ✅ When was it last updated? (Check publish date)
- ✅ Does it use current modules (Az vs. deprecated AzureRM)?
- ✅ Are there community ratings and reviews?
- ✅ Is there active Q&A or GitHub issues?

**5. Dependencies**:
- ✅ What modules does it require? (Check Import-Module statements)
- ✅ Are those modules available in your Automation account?
- ✅ Does it require specific Azure permissions?
- ✅ Does it depend on other runbooks or resources?

### Example: Evaluating a VM Management Runbook

**Runbook**: "Start-Stop-AzureV2VMs"

**Review Checklist**:

```powershell
# Open runbook in gallery, review code:

<#
.SYNOPSIS
    Start or stop Azure VMs based on tags and schedule
.DESCRIPTION
    This runbook starts or stops Azure Resource Manager VMs based on:
    - Tag filtering (AutoShutdown = True)
    - Time-based scheduling
.PARAMETER Action
    Start or Stop
.PARAMETER ResourceGroupName
    Target resource group
.NOTES
    Author: Microsoft Azure Automation Team
    Last Updated: 2025-12-15
    Required Modules: Az.Accounts, Az.Compute
#>

param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("Start", "Stop")]
    [string]$Action,
    
    [Parameter(Mandatory=$true)]
    [string]$ResourceGroupName
)

# ✅ GOOD: Uses Run As connection (no hardcoded credentials)
$connection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzAccount -ServicePrincipal `
    -Tenant $connection.TenantID `
    -ApplicationId $connection.ApplicationID `
    -CertificateThumbprint $connection.CertificateThumbprint

# ✅ GOOD: Parameterized, not hardcoded
$vms = Get-AzVM -ResourceGroupName $ResourceGroupName

# ✅ GOOD: Error handling
try {
    foreach ($vm in $vms) {
        if ($Action -eq "Start") {
            Start-AzVM -ResourceGroupName $ResourceGroupName -Name $vm.Name -NoWait
        } else {
            Stop-AzVM -ResourceGroupName $ResourceGroupName -Name $vm.Name -Force -NoWait
        }
    }
} catch {
    Write-Error "Failed to $Action VMs: $_"
    throw
}
```

**Evaluation Result**:
- ✅ Well-documented with Synopsis/Description
- ✅ Uses current Az modules (not deprecated AzureRM)
- ✅ Secure authentication (Run As connection)
- ✅ Parameterized (flexible)
- ✅ Error handling included
- ✅ From trusted source (Microsoft)
- ⚠️ Last updated 2 months ago (acceptable)

**Decision**: ✅ Safe to import and use

## Importing Runbooks from Gallery

### Import via Azure Portal

**Steps**:

1. **Browse Gallery**:
   - Automation Account → Runbooks → Browse gallery
   - Search: "Start Stop VM"
   - Select runbook: "Start-Stop-AzureV2VMs"

2. **Review Details**:
   - Read description and requirements
   - Check required modules (Az.Accounts, Az.Compute)
   - Review sample code

3. **Import Runbook**:
   - Click **Import**
   - Confirm runbook name (or customize)
   - Optionally change description
   - Click **OK**

4. **Wait for Import**:
   - Status shows "Importing" (typically 10-30 seconds)
   - Refresh to see "Completed"

5. **Verify Import**:
   - Runbook appears in your runbooks list
   - Status: "New" (draft version created)

**Azure Portal Import Interface**:

```
┌──────────────────────────────────────────────────┐
│ Import Runbook: Start-Stop-AzureV2VMs           │
├──────────────────────────────────────────────────┤
│                                                  │
│ Name: Start-Stop-AzureV2VMs                    │
│ [________________________________]               │
│                                                  │
│ Runbook type: PowerShell                        │
│ [PowerShell ▼]                                  │
│                                                  │
│ Description:                                     │
│ [____________________________________________]   │
│ [____________________________________________]   │
│                                                  │
│ [Cancel]                        [OK]            │
└──────────────────────────────────────────────────┘
```

### Import via PowerShell

```powershell
# Import runbook from URL (GitHub raw content)
$runbookUrl = "https://raw.githubusercontent.com/azureautomation/runbooks/master/Utility/Start-AzureVMs.ps1"

Import-AzAutomationRunbook `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Path $runbookUrl `
    -Name "Start-AzureVMs" `
    -Type PowerShell `
    -Description "Start VMs based on tags (imported from gallery)" `
    -LogVerbose $true

# Verify import
Get-AzAutomationRunbook `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "Start-AzureVMs" | Select-Object Name, RunbookType, State, CreationTime
```

### Import Local File

```powershell
# Download runbook from GitHub
Invoke-WebRequest `
    -Uri "https://raw.githubusercontent.com/azureautomation/runbooks/master/Utility/Stop-AzureVMs.ps1" `
    -OutFile "C:\Temp\Stop-AzureVMs.ps1"

# Review locally before importing
code "C:\Temp\Stop-AzureVMs.ps1"

# Import after review
Import-AzAutomationRunbook `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Path "C:\Temp\Stop-AzureVMs.ps1" `
    -Type PowerShell `
    -Published  # Directly to published state (skip draft)
```

## Customizing Gallery Runbooks

Gallery runbooks are starting points—customize for your environment:

### Customization Workflow

```
Import Gallery Runbook (Draft)
    ↓
Edit in Azure Portal or Export Locally
    ↓
Customize for Your Environment:
    - Update hardcoded values → Use variables
    - Add organization-specific logic
    - Enhance error handling
    - Add logging/notifications
    - Integrate with your monitoring
    ↓
Test in Test Pane
    ↓
Publish Customized Version
    ↓
Schedule or Link to Webhook
```

### Example Customization: Add Email Notification

**Original Gallery Runbook**:

```powershell
# Basic VM start runbook from gallery
param([string]$ResourceGroupName)

$connection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzAccount -ServicePrincipal `
    -Tenant $connection.TenantID `
    -ApplicationId $connection.ApplicationID `
    -CertificateThumbprint $connection.CertificateThumbprint

$vms = Get-AzVM -ResourceGroupName $ResourceGroupName
foreach ($vm in $vms) {
    Start-AzVM -ResourceGroupName $ResourceGroupName -Name $vm.Name -NoWait
}
```

**Customized with Email Notifications**:

```powershell
# Customized: Added email notifications using SendGrid
param(
    [string]$ResourceGroupName,
    [string]$NotificationEmail = "ops-team@company.com"
)

# Authenticate
$connection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzAccount -ServicePrincipal `
    -Tenant $connection.TenantID `
    -ApplicationId $connection.ApplicationID `
    -CertificateThumbprint $connection.CertificateThumbprint

# Get VMs
$vms = Get-AzVM -ResourceGroupName $ResourceGroupName
Write-Output "Found $($vms.Count) VMs in $ResourceGroupName"

# Start VMs and track results
$results = @()
foreach ($vm in $vms) {
    try {
        Write-Output "Starting VM: $($vm.Name)"
        Start-AzVM -ResourceGroupName $ResourceGroupName -Name $vm.Name -NoWait -ErrorAction Stop
        $results += [PSCustomObject]@{
            VMName = $vm.Name
            Status = "Started"
            Error = $null
        }
    }
    catch {
        Write-Error "Failed to start $($vm.Name): $_"
        $results += [PSCustomObject]@{
            VMName = $vm.Name
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

# Send email notification
$sendGridApiKey = Get-AutomationVariable -Name "SendGrid-ApiKey"
$fromEmail = "azure-automation@company.com"

$emailBody = @"
Azure VM Start Automation Report

Resource Group: $ResourceGroupName
Total VMs: $($vms.Count)
Successful: $($results | Where-Object {$_.Status -eq "Started"} | Measure-Object | Select-Object -ExpandProperty Count)
Failed: $($results | Where-Object {$_.Status -eq "Failed"} | Measure-Object | Select-Object -ExpandProperty Count)

Details:
$($results | Format-Table -AutoSize | Out-String)

Timestamp: $(Get-Date -Format "yyyy-MM-dd HH:mm:ss UTC")
"@

$emailPayload = @{
    personalizations = @(
        @{
            to = @(@{email = $NotificationEmail})
            subject = "VM Start Automation Report - $ResourceGroupName"
        }
    )
    from = @{email = $fromEmail}
    content = @(
        @{
            type = "text/plain"
            value = $emailBody
        }
    )
} | ConvertTo-Json -Depth 5

$headers = @{
    Authorization = "Bearer $sendGridApiKey"
    "Content-Type" = "application/json"
}

try {
    Invoke-RestMethod -Uri "https://api.sendgrid.com/v3/mail/send" `
        -Method Post -Headers $headers -Body $emailPayload
    Write-Output "Email notification sent to $NotificationEmail"
}
catch {
    Write-Warning "Failed to send email notification: $_"
}

Write-Output "VM start automation completed"
```

**Customizations Added**:
1. ✅ Email notification parameter
2. ✅ Result tracking ($results array)
3. ✅ Enhanced error handling
4. ✅ SendGrid integration for email
5. ✅ Detailed email report with success/failure counts

### Example Customization: Add Azure Monitor Integration

```powershell
# Original: Simple backup runbook
# Customized: Added Azure Monitor custom metrics

param([string]$ResourceGroupName)

# ... existing backup logic ...

# NEW: Send custom metric to Azure Monitor
$metricData = @{
    time = (Get-Date).ToUniversalTime().ToString("o")
    data = @{
        baseData = @{
            metric = "BackupJobDuration"
            namespace = "AzureAutomation/Backup"
            dimNames = @("ResourceGroup", "Status")
            series = @(
                @{
                    dimValues = @($ResourceGroupName, "Success")
                    min = $durationSeconds
                    max = $durationSeconds
                    sum = $durationSeconds
                    count = 1
                }
            )
        }
    }
} | ConvertTo-Json -Depth 10

# Send to Application Insights
$appInsightsKey = Get-AutomationVariable -Name "AppInsights-InstrumentationKey"
Invoke-RestMethod -Uri "https://dc.services.visualstudio.com/v2/track" `
    -Method Post `
    -Headers @{"Content-Type"="application/json"} `
    -Body $metricData
```

## Finding Python Runbooks

The gallery includes Python runbooks for Python-specific scenarios:

### Filter by Python

**Azure Portal**:
1. Browse gallery
2. Filter: **Runbook type** → **Python**
3. Results show only Python runbooks

**Popular Python Gallery Runbooks**:
- **AWS EC2 Management**: Using boto3 for multi-cloud
- **Data Processing**: pandas-based data transformation
- **REST API Integration**: Complex API workflows
- **Log Analysis**: Python-based log parsing

### Example Python Gallery Runbook

```python
"""
Azure VM Inventory Report (Python)
Generates detailed VM inventory report with Python pandas
"""

import os
import pandas as pd
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.resource import SubscriptionClient

def main():
    # Authenticate
    credential = DefaultAzureCredential()
    subscription_id = os.environ.get('AZURE_SUBSCRIPTION_ID')
    
    # Get compute client
    compute_client = ComputeManagementClient(credential, subscription_id)
    
    # Collect VM data
    vm_data = []
    
    for vm in compute_client.virtual_machines.list_all():
        vm_data.append({
            'Name': vm.name,
            'ResourceGroup': vm.id.split('/')[4],
            'Location': vm.location,
            'VMSize': vm.hardware_profile.vm_size,
            'OS': vm.storage_profile.os_disk.os_type,
            'Tags': str(vm.tags) if vm.tags else 'None'
        })
    
    # Create DataFrame
    df = pd.DataFrame(vm_data)
    
    # Generate report
    print("VM Inventory Report")
    print("=" * 80)
    print(df.to_string(index=False))
    print("\n")
    print(f"Total VMs: {len(df)}")
    print(f"By Region:\n{df.groupby('Location').size()}")
    print(f"By Size:\n{df.groupby('VMSize').size()}")

if __name__ == "__main__":
    main()
```

## Module Deprecation: AzureRM → Az

**Important Notice**: Many gallery runbooks were created before the Az module transition.

### Identifying Deprecated Runbooks

**Red Flags**:
```powershell
# ❌ OLD: AzureRM modules (deprecated February 2024)
Import-Module AzureRM
Login-AzureRmAccount
Get-AzureRmVM

# ✅ NEW: Az modules (current standard)
Import-Module Az.Accounts
Connect-AzAccount
Get-AzVM
```

### Migrating AzureRM Runbooks to Az

**Migration Steps**:

1. **Enable Az Module Compatibility**:
```powershell
# This allows AzureRM cmdlets to work with Az modules (temporary fix)
Enable-AzureRmAlias
```

2. **Update Module References**:
```powershell
# Replace all AzureRM references
(Get-Content "C:\Temp\Runbook.ps1") `
    -replace "AzureRM", "Az" `
    -replace "Login-AzureRmAccount", "Connect-AzAccount" `
    -replace "Get-AzureRmVM", "Get-AzVM" `
    | Set-Content "C:\Temp\Runbook-Updated.ps1"
```

3. **Update Modules in Automation Account**:
```powershell
# Remove old AzureRM modules
Get-AzAutomationModule -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" |
    Where-Object {$_.Name -like "AzureRM*"} |
    Remove-AzAutomationModule -Force

# Import latest Az modules
$azModules = @("Az.Accounts", "Az.Compute", "Az.Storage", "Az.Network")
foreach ($module in $azModules) {
    New-AzAutomationModule -ResourceGroupName "rg-automation-prod" `
        -AutomationAccountName "automation-account-prod" `
        -Name $module `
        -ContentLinkUri "https://www.powershellgallery.com/api/v2/package/$module"
}
```

4. **Test Updated Runbook**:
   - Import updated runbook
   - Test in Test Pane
   - Verify all cmdlets work
   - Publish if successful

## Contributing to the Gallery

You can contribute your own runbooks back to the community:

### Contribution Process

1. **Create High-Quality Runbook**:
   - Well-documented (Synopsis, Description, Examples)
   - Error handling
   - No hardcoded credentials
   - Parameterized

2. **Test Thoroughly**:
   - Multiple scenarios
   - Edge cases
   - Different environments

3. **Submit to GitHub**:
   ```bash
   # Fork the repository
   git clone https://github.com/\<your-username\>/azureautomation-runbooks.git
   
   # Add your runbook
   cp MyAwesomeRunbook.ps1 azureautomation-runbooks/Utility/
   
   # Commit and push
   git add .
   git commit -m "Add: VM cost optimization runbook"
   git push origin main
   
   # Create pull request on GitHub
   ```

4. **Microsoft Review**:
   - Code quality review
   - Security assessment
   - Testing validation
   - Approval and merge

## Best Practices

### 1. Always Review Before Importing

```powershell
# Download and review locally first
Invoke-WebRequest `
    -Uri "https://raw.githubusercontent.com/azureautomation/runbooks/master/Utility/Unknown-Runbook.ps1" `
    -OutFile "C:\Temp\Review.ps1"

# Open in editor
code "C:\Temp\Review.ps1"

# Check for:
# - Hardcoded credentials (search for "password", "apikey")
# - Suspicious commands (Invoke-Expression, Download files)
# - External URLs (data exfiltration risk)
# - Deprecated modules (AzureRM)
```

### 2. Use Version Control for Customizations

```bash
# Track your customizations in Git
git init
git add Start-AzureVMs-Custom.ps1
git commit -m "Initial: Imported from gallery"

# Make customizations
# Edit file...

git add Start-AzureVMs-Custom.ps1
git commit -m "Added: Email notification integration"

# View history
git log --oneline
```

### 3. Maintain Documentation

```markdown
# Custom Runbook Documentation

## Source
- **Original**: Azure Automation Gallery - "Start-Stop-AzureV2VMs"
- **Import Date**: 2026-01-14
- **Original Author**: Microsoft Azure Automation Team

## Customizations Made
1. Added email notification via SendGrid (2026-01-14)
2. Integrated with Azure Monitor custom metrics (2026-01-15)
3. Enhanced error handling with retry logic (2026-01-16)

## Dependencies
- Az.Accounts (v2.10.0+)
- Az.Compute (v5.0.0+)
- SendGrid API key (stored in Automation Variable: SendGrid-ApiKey)

## Testing
- Tested in Dev environment: 2026-01-14 ✅
- Tested in Staging environment: 2026-01-15 ✅
- Deployed to Production: 2026-01-16 ✅
```

### 4. Keep Gallery Runbooks Updated

```powershell
# Check for updates (manual process)
# 1. Review GitHub repository for changes
# 2. Compare local version with gallery version
# 3. Re-import if significant updates

# Example: Check last commit date
$runbookUrl = "https://api.github.com/repos/azureautomation/runbooks/commits?path=Utility/Start-AzureVMs.ps1"
$commits = Invoke-RestMethod -Uri $runbookUrl
$latestCommit = $commits[0]

Write-Output "Latest update: $($latestCommit.commit.author.date)"
Write-Output "Message: $($latestCommit.commit.message)"
```

## Next Steps

Now that you know how to discover and import runbooks from the gallery, proceed to **Unit 6: Examine Webhooks** to learn how to trigger your runbooks (including gallery runbooks) from external systems like Azure DevOps, monitoring alerts, and third-party services.

**Quick Start Actions**:
1. Browse the Runbook Gallery in your Automation account
2. Import a simple utility runbook (e.g., "Get-AzureVMInventory")
3. Review the code in the Azure Portal editor
4. Test the runbook in Test Pane
5. Customize it with your organization's specific requirements

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/5-explore-runbook-gallery
