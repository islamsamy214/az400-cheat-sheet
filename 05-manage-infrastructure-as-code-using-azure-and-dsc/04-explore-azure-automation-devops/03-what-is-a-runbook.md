# What is a Runbook?

## Overview

A runbook is the fundamental unit of automation in Azure Automation. Think of runbooks as containers for your automation logic—whether it's PowerShell scripts, Python code, or graphical workflows. Runbooks can manage Azure resources, on-premises infrastructure, third-party services, or orchestrate complex multi-step processes.

Understanding the different runbook types and their characteristics is crucial for choosing the right tool for your automation needs. This unit explores the four runbook types available in Azure Automation, when to use each, and how to create and manage them effectively.

## What is a Runbook?

A **runbook** is a set of automation tasks organized as a script or workflow. In Azure Automation context, runbooks are:

- **Repositories for Scripts**: Store PowerShell, Python, or graphical workflows
- **Reusable Automation**: Execute on-demand, on schedule, or via webhooks
- **Resource Managers**: Create, configure, monitor, and delete Azure resources
- **Orchestrators**: Coordinate multiple systems and services
- **Integrators**: Connect Azure with on-premises, multi-cloud, and third-party platforms

### Key Capabilities

**Runbooks Can**:
- Reference **shared resources** (credentials, variables, connections, certificates)
- Invoke **other runbooks** as child runbooks (modular design)
- Execute **on-demand** via manual trigger
- Run on **schedules** (hourly, daily, weekly, custom cron expressions)
- Respond to **webhooks** (external triggers from Azure DevOps, monitoring alerts, ServiceNow)
- Target **Azure resources**, **hybrid workers** (on-premises), or both
- Include **input parameters** for dynamic behavior
- Return **output** for integration with other systems

### Runbook Repository Concept

Think of your Automation account as a **runbook repository**:

```
Automation Account: automation-account-prod
├── Runbooks/
│   ├── Start-AzureVMs (PowerShell)
│   ├── Stop-AzureVMs (PowerShell)
│   ├── Backup-Databases (PowerShell Workflow)
│   ├── Monitor-Resources (Python)
│   ├── Provision-Infrastructure (Graphical)
│   └── Incident-Response (PowerShell Workflow)
├── Schedules/
│   ├── DailyMorning (7:00 AM)
│   └── NightlyCleanup (11:00 PM)
└── Jobs/
    ├── Job-12345 (Start-AzureVMs, Success, 2026-01-14 07:00)
    ├── Job-12346 (Stop-AzureVMs, Success, 2026-01-14 19:00)
    └── Job-12347 (Backup-Databases, Running, 2026-01-14 23:00)
```

## Runbook Types

Azure Automation supports **four runbook types**, each with distinct characteristics and use cases.

### Overview Comparison

| Feature | PowerShell | PowerShell Workflow | Python | Graphical |
|---------|-----------|---------------------|--------|-----------|
| **Language** | PowerShell | PowerShell Workflow | Python 2 or 3 | Visual drag-and-drop |
| **Parallel Execution** | No (sequential) | Yes (Parallel keyword) | Manual (threading) | Yes (parallel branches) |
| **Checkpoints** | No | Yes (Checkpoint-Workflow) | Manual | Yes |
| **Startup Time** | Fast (seconds) | Slow (compilation) | Fast | Slow (compilation) |
| **Editing** | Text editor, VS Code | Text editor, PowerShell ISE | Text editor, VS Code | Azure Portal only |
| **Learning Curve** | Low (standard PS) | Medium (Workflow syntax) | Low (standard Python) | Low (visual) |
| **Best For** | Simple automation | Long-running resilient tasks | Python ecosystem integration | Non-coders, visual workflows |
| **Hybrid Workers** | Yes | Yes | Yes | Yes |
| **InlineScript Support** | N/A | Yes (for standard PS) | N/A | N/A |

### 1. PowerShell Runbooks

**Characteristics**:
- Standard PowerShell scripting (no Workflow syntax)
- Sequential execution (one command at a time)
- Fast startup (no compilation required)
- No native checkpoint or parallel processing support
- Ideal for short, straightforward automation tasks

**When to Use**:
- Quick automation tasks (< 5 minutes)
- Sequential operations (no need for parallelism)
- Simple Azure resource management
- When you need fast execution start time
- Migrating from local PowerShell scripts

**Advantages**:
- ✅ Familiar syntax (standard PowerShell)
- ✅ Fast execution startup
- ✅ Easy to test locally before importing
- ✅ Rich cmdlet ecosystem (Az modules, third-party modules)
- ✅ Simpler debugging than Workflows

**Limitations**:
- ❌ No checkpoints (cannot resume after failure)
- ❌ No parallel processing (without manual runspaces)
- ❌ Not ideal for very long-running tasks (3-hour job limit, no resume)
- ❌ More susceptible to transient errors (no automatic retry)

**Example Use Cases**:
- Start/stop VMs on schedule
- Send email notifications
- Query Azure resources and generate reports
- Trigger Azure DevOps pipelines
- Rotate access keys for storage accounts

**Sample PowerShell Runbook**:

```powershell
<#
.SYNOPSIS
    Start all VMs in a resource group tagged with AutoStart=True

.PARAMETER ResourceGroupName
    Name of the resource group containing VMs

.PARAMETER TagName
    Tag name to filter VMs (default: AutoStart)

.PARAMETER TagValue
    Tag value to filter VMs (default: True)
#>

param(
    [Parameter(Mandatory=$true)]
    [string]$ResourceGroupName,
    
    [Parameter(Mandatory=$false)]
    [string]$TagName = "AutoStart",
    
    [Parameter(Mandatory=$false)]
    [string]$TagValue = "True"
)

# Authenticate using Run As account
$connection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzAccount -ServicePrincipal `
    -Tenant $connection.TenantID `
    -ApplicationId $connection.ApplicationID `
    -CertificateThumbprint $connection.CertificateThumbprint

# Get all VMs with matching tag
$vms = Get-AzVM -ResourceGroupName $ResourceGroupName | Where-Object {
    $_.Tags[$TagName] -eq $TagValue
}

if ($vms.Count -eq 0) {
    Write-Output "No VMs found with tag $TagName=$TagValue in resource group $ResourceGroupName"
    exit
}

Write-Output "Found $($vms.Count) VMs to start"

# Start each VM (sequential)
foreach ($vm in $vms) {
    Write-Output "Starting VM: $($vm.Name)"
    $result = Start-AzVM -ResourceGroupName $ResourceGroupName -Name $vm.Name -NoWait
    
    if ($result.Status -eq "Succeeded") {
        Write-Output "✓ $($vm.Name) start command sent successfully"
    } else {
        Write-Error "✗ Failed to start $($vm.Name)"
    }
}

Write-Output "All start commands completed"
```

**Execution Output**:
```
Found 3 VMs to start
Starting VM: vm-webapp-01
✓ vm-webapp-01 start command sent successfully
Starting VM: vm-webapp-02
✓ vm-webapp-02 start command sent successfully
Starting VM: vm-database-01
✓ vm-database-01 start command sent successfully
All start commands completed
```

### 2. PowerShell Workflow Runbooks

**Characteristics**:
- Uses Windows Workflow Foundation (WWF)
- Supports parallel execution (Parallel keyword)
- Supports checkpoints (Checkpoint-Workflow cmdlet)
- Slower startup due to compilation
- Can resume after interruption
- Designed for long-running, resilient operations

**When to Use**:
- Long-running tasks (> 30 minutes)
- Operations requiring resilience (must survive interruptions)
- Tasks benefiting from parallel execution
- Automating hundreds of similar operations simultaneously
- Complex multi-step workflows with potential failure points

**Advantages**:
- ✅ **Parallel Processing**: Execute multiple operations concurrently
- ✅ **Checkpoints**: Save state and resume after failure/restart
- ✅ **Automatic Retries**: Built-in connection and activity retry logic
- ✅ **Long-Running**: Designed for tasks lasting hours
- ✅ **Resilient**: Survive system restarts, network interruptions

**Limitations**:
- ❌ Slower startup (workflow compilation takes 10-30 seconds)
- ❌ Different syntax (not pure PowerShell)
- ❌ Some PowerShell features unavailable (need InlineScript blocks)
- ❌ More complex debugging
- ❌ Steeper learning curve

**Example Use Cases**:
- Parallel VM provisioning (create 100 VMs simultaneously)
- Multi-region disaster recovery orchestration
- Large-scale configuration management
- Long-running data migration tasks
- Complex approval workflows with wait states

**Workflow Syntax Differences**:

| Standard PowerShell | PowerShell Workflow |
|---------------------|---------------------|
| `function MyFunction {}` | `workflow MyWorkflow {}` |
| N/A | `Checkpoint-Workflow` (save state) |
| N/A | `Parallel { }` block |
| N/A | `ForEach -Parallel ( )` construct |
| All cmdlets work | Some require `InlineScript { }` |

**Sample PowerShell Workflow Runbook**:

```powershell
<#
.SYNOPSIS
    Parallel VM backup workflow with checkpoints

.DESCRIPTION
    Backs up multiple VMs in parallel with checkpoint resilience.
    If workflow is interrupted, it resumes from last checkpoint.
#>

workflow Backup-AzureVMsParallel {
    param(
        [Parameter(Mandatory=$true)]
        [string]$ResourceGroupName,
        
        [Parameter(Mandatory=$false)]
        [int]$ThrottleLimit = 10
    )
    
    # Authenticate (runs once at workflow start)
    $connection = Get-AutomationConnection -Name "AzureRunAsConnection"
    Connect-AzAccount -ServicePrincipal `
        -Tenant $connection.TenantID `
        -ApplicationId $connection.ApplicationID `
        -CertificateThumbprint $connection.CertificateThumbprint
    
    # Checkpoint after authentication
    Checkpoint-Workflow
    
    # Get all VMs
    $vms = Get-AzVM -ResourceGroupName $ResourceGroupName
    Write-Output "Found $($vms.Count) VMs to backup"
    
    # Checkpoint before parallel processing
    Checkpoint-Workflow
    
    # Parallel backup execution
    ForEach -Parallel -ThrottleLimit $ThrottleLimit ($vm in $vms) {
        InlineScript {
            $vmName = $using:vm.Name
            $rgName = $using:ResourceGroupName
            
            Write-Output "Starting backup for VM: $vmName"
            
            try {
                # Simulate backup operation (replace with actual backup logic)
                $backup = New-AzRecoveryServicesBackupProtectionPolicy `
                    -Name "Policy-$vmName" `
                    -WorkloadType AzureVM `
                    -RetentionPolicy (Get-AzRecoveryServicesBackupRetentionPolicyObject) `
                    -SchedulePolicy (Get-AzRecoveryServicesBackupSchedulePolicyObject)
                
                # Trigger backup
                $container = Get-AzRecoveryServicesBackupContainer `
                    -ContainerType AzureVM -FriendlyName $vmName
                $item = Get-AzRecoveryServicesBackupItem `
                    -Container $container -WorkloadType AzureVM
                
                Backup-AzRecoveryServicesBackupItem -Item $item
                
                Write-Output "✓ Backup completed for $vmName"
            }
            catch {
                Write-Error "✗ Backup failed for $vmName: $_"
            }
        }
    }
    
    # Final checkpoint
    Checkpoint-Workflow
    
    Write-Output "All VM backups completed"
}
```

**Parallel Execution Visualization**:
```
Start Workflow
    ↓
Authenticate (sequential)
    ↓
Checkpoint #1
    ↓
Get VMs (sequential)
    ↓
Checkpoint #2
    ↓
┌────────────────────────────────────┐
│  Parallel Execution (10 concurrent) │
├────────────────────────────────────┤
│  Thread 1: Backup vm-01            │
│  Thread 2: Backup vm-02            │
│  Thread 3: Backup vm-03            │
│  ...                               │
│  Thread 10: Backup vm-10           │
│                                    │
│  (When thread completes, next VM   │
│   is assigned until all processed) │
└────────────────────────────────────┘
    ↓
Checkpoint #3
    ↓
Write Summary (sequential)
    ↓
End Workflow
```

### 3. Python Runbooks

**Characteristics**:
- Standard Python scripting (Python 2.7 or Python 3.8+)
- Support for Python standard library
- Can import custom packages (upload via Automation account)
- Ideal for leveraging Python ecosystem
- Sequential execution (unless using threading/asyncio)
- Fast startup

**When to Use**:
- Leveraging Python-specific libraries (pandas, requests, boto3 for AWS)
- Teams with Python expertise (less PowerShell knowledge)
- Data processing and analysis tasks
- Integration with RESTful APIs
- Cross-platform automation (Linux-focused workflows)
- Machine learning workflows (scikit-learn, TensorFlow)

**Advantages**:
- ✅ Rich Python ecosystem (PyPI packages)
- ✅ Familiar for Python developers
- ✅ Excellent for data manipulation
- ✅ Cross-platform (Linux and Windows)
- ✅ Easy REST API integration
- ✅ Fast startup

**Limitations**:
- ❌ No native checkpoint support
- ❌ No native parallel processing (without manual threading)
- ❌ Limited Azure SDK compared to PowerShell Az modules
- ❌ Package management more complex

**Example Use Cases**:
- AWS resource management (using boto3)
- Data extraction and transformation
- REST API interactions
- Log analysis and reporting
- Custom monitoring and alerting
- Integration with non-Microsoft services

**Sample Python Runbook**:

```python
#!/usr/bin/env python3
"""
Azure VM Management with Python

This runbook demonstrates Python for Azure automation:
- Authenticate using Azure SDK
- Query VMs by tag
- Start/stop VMs based on schedule
- Send notifications via REST API
"""

import os
import sys
import json
import logging
from datetime import datetime
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.resource import ResourceManagementClient
import requests

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)


def authenticate():
    """Authenticate to Azure using Managed Identity or Run As account"""
    try:
        # Use Default Azure Credential (supports managed identity, env vars, etc.)
        credential = DefaultAzureCredential()
        return credential
    except Exception as e:
        logger.error(f"Authentication failed: {e}")
        sys.exit(1)


def get_vms_by_tag(compute_client, resource_group, tag_name, tag_value):
    """
    Get all VMs in resource group matching specific tag
    
    Args:
        compute_client: Azure Compute client
        resource_group: Resource group name
        tag_name: Tag name to filter
        tag_value: Tag value to match
    
    Returns:
        List of VM objects matching criteria
    """
    vms = []
    
    try:
        all_vms = compute_client.virtual_machines.list(resource_group)
        
        for vm in all_vms:
            if vm.tags and vm.tags.get(tag_name) == tag_value:
                vms.append(vm)
                logger.info(f"Found VM: {vm.name} with tag {tag_name}={tag_value}")
        
        return vms
    
    except Exception as e:
        logger.error(f"Error retrieving VMs: {e}")
        return []


def start_vm(compute_client, resource_group, vm_name):
    """Start a VM asynchronously"""
    try:
        logger.info(f"Starting VM: {vm_name}")
        async_vm_start = compute_client.virtual_machines.begin_start(
            resource_group, vm_name
        )
        # Don't wait for completion (async)
        logger.info(f"✓ Start command sent for {vm_name}")
        return True
    except Exception as e:
        logger.error(f"✗ Failed to start {vm_name}: {e}")
        return False


def stop_vm(compute_client, resource_group, vm_name):
    """Stop (deallocate) a VM asynchronously"""
    try:
        logger.info(f"Stopping VM: {vm_name}")
        async_vm_stop = compute_client.virtual_machines.begin_deallocate(
            resource_group, vm_name
        )
        logger.info(f"✓ Stop command sent for {vm_name}")
        return True
    except Exception as e:
        logger.error(f"✗ Failed to stop {vm_name}: {e}")
        return False


def send_teams_notification(webhook_url, message):
    """Send notification to Microsoft Teams channel"""
    payload = {
        "@type": "MessageCard",
        "@context": "https://schema.org/extensions",
        "summary": "Azure Automation Notification",
        "themeColor": "0078D4",
        "title": "VM Automation Status",
        "text": message,
        "potentialAction": [
            {
                "@type": "OpenUri",
                "name": "View in Azure Portal",
                "targets": [
                    {
                        "os": "default",
                        "uri": "https://portal.azure.com"
                    }
                ]
            }
        ]
    }
    
    try:
        response = requests.post(webhook_url, json=payload)
        response.raise_for_status()
        logger.info("Teams notification sent successfully")
    except Exception as e:
        logger.error(f"Failed to send Teams notification: {e}")


def main():
    """Main execution function"""
    # Configuration (in production, use Automation variables)
    subscription_id = os.environ.get('AZURE_SUBSCRIPTION_ID')
    resource_group = "rg-production-vms"
    tag_name = "AutoStart"
    tag_value = "True"
    action = "start"  # or "stop"
    teams_webhook = os.environ.get('TEAMS_WEBHOOK_URL')
    
    if not subscription_id:
        logger.error("AZURE_SUBSCRIPTION_ID environment variable not set")
        sys.exit(1)
    
    # Authenticate
    credential = authenticate()
    
    # Create compute client
    compute_client = ComputeManagementClient(credential, subscription_id)
    
    # Get VMs by tag
    vms = get_vms_by_tag(compute_client, resource_group, tag_name, tag_value)
    
    if not vms:
        logger.info(f"No VMs found with tag {tag_name}={tag_value}")
        return
    
    logger.info(f"Found {len(vms)} VMs to {action}")
    
    # Perform action on each VM
    success_count = 0
    failed_count = 0
    
    for vm in vms:
        if action == "start":
            result = start_vm(compute_client, resource_group, vm.name)
        else:
            result = stop_vm(compute_client, resource_group, vm.name)
        
        if result:
            success_count += 1
        else:
            failed_count += 1
    
    # Summary
    summary = f"""
    VM Automation Summary:
    - Action: {action.upper()}
    - Total VMs: {len(vms)}
    - Successful: {success_count}
    - Failed: {failed_count}
    - Timestamp: {datetime.now().isoformat()}
    """
    
    logger.info(summary)
    
    # Send notification
    if teams_webhook:
        send_teams_notification(teams_webhook, summary)


if __name__ == "__main__":
    main()
```

**Python Package Management**:

```bash
# Install custom packages in Automation account
# Azure Portal: Automation Account → Python packages → Add

# Upload .whl file or install from PyPI:
# - Package name: requests
# - Version: 2.31.0

# In runbook, import as normal:
import requests
response = requests.get('https://api.example.com/data')
```

### 4. Graphical Runbooks

**Characteristics**:
- Visual drag-and-drop interface
- No coding required (but can include code snippets)
- Built on PowerShell Workflow (supports checkpoints, parallel execution)
- Edited exclusively in Azure Portal
- Activities connected by links (control flow)
- Ideal for non-developers and visual learners

**When to Use**:
- Teams without coding experience
- Visual representation of complex workflows
- Quick prototyping and demonstration
- When visual documentation of automation is valuable
- Standardized workflows with limited customization

**Advantages**:
- ✅ No coding required (low barrier to entry)
- ✅ Visual representation (easy to understand workflow)
- ✅ Built-in activities for common tasks
- ✅ Supports parallel branches
- ✅ Checkpoint support (resilience)
- ✅ Parameter validation built-in

**Limitations**:
- ❌ Can only be edited in Azure Portal (no local editing)
- ❌ Less flexible than code-based runbooks
- ❌ Limited version control (can't use Git effectively)
- ❌ Complex logic becomes visually cluttered
- ❌ Cannot convert to/from PowerShell Workflow text format

**Example Use Cases**:
- Approval workflows
- Simple VM lifecycle management
- Scheduled resource cleanup
- Standard IT service requests
- Visual representation of multi-step processes

**Graphical Runbook Structure**:

```
┌─────────────────────┐
│  Start              │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Get-AzVM           │ ← Activity: Retrieve all VMs
│  Parameters:        │
│  - ResourceGroup    │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  ForEach VM         │ ← Loop through VMs
└──────┬──────────────┘
       │
       ├──────────────────────┬──────────────────────┐
       │                      │                      │
       ▼                      ▼                      ▼
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│ Start-AzVM  │      │ Stop-AzVM   │      │ Restart-AzVM│
│ (Parallel)  │      │ (Parallel)  │      │ (Parallel)  │
└─────────────┘      └─────────────┘      └─────────────┘
       │                      │                      │
       └──────────────────────┴──────────────────────┘
       │
       ▼
┌─────────────────────┐
│  Send-Email         │ ← Activity: Notification
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  End                │
└─────────────────────┘
```

**Graphical Runbook Activities**:
- **Azure Activities**: Get-AzVM, Start-AzVM, New-AzResourceGroup, etc.
- **Control Activities**: If-Else, ForEach, Parallel, Sequence
- **Data Activities**: Get Variable, Set Variable, Get Credential
- **Script Activities**: Run PowerShell Code, Run Python Script
- **Integration Activities**: Send Email, Invoke Webhook, Call Child Runbook

## Creating and Managing Runbooks

### Creation Methods

**Method 1: Create New Runbook**

```powershell
# Create new empty runbook
New-AzAutomationRunbook `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "Start-VMs-Daily" `
    -Type PowerShell `
    -Description "Start VMs tagged with AutoStart=True"
```

**Method 2: Import Existing Script**

```powershell
# Import runbook from local file
Import-AzAutomationRunbook `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Path "C:\Scripts\Start-VMs-Daily.ps1" `
    -Type PowerShell `
    -Published
```

**Method 3: Import from Gallery**

```
Azure Portal: 
Automation Account → Runbooks → Browse gallery → Select runbook → Import
```

### Runbook Lifecycle

**States**:
1. **New**: Just created, no content yet
2. **Edit**: Being edited (draft version)
3. **Published**: Ready for execution (production version)

**Version Control**:
- **Draft Version**: Editable, not executable
- **Published Version**: Executable, read-only
- Can have both simultaneously (edit draft while published version runs)

**Workflow**:
```
Create Runbook (New)
    ↓
Edit Draft Version
    ↓
Test Draft in Test Pane
    ↓
Fix Issues (repeat testing)
    ↓
Publish Draft → Published Version
    ↓
Execute Published Version (via schedule, webhook, manual)
    ↓
(Later) Edit Published → Creates New Draft
    ↓
Test and Publish New Draft
```

## Choosing the Right Runbook Type

### Decision Matrix

```
┌─────────────────────────────────────────────────────────┐
│          START: What are you automating?                │
└────────────────────┬────────────────────────────────────┘
                     │
      ┌──────────────┴──────────────┐
      │                             │
  Need Coding?                  Visual Only?
      │                             │
     YES                            NO
      │                             │
      ├─────────────┐               │
      │             │               │
  PowerShell    Python?         Graphical
      │             │           Runbook
      │            YES             │
      │             │               └──→ Use for visual
      │         Python                  workflows, non-coders
      │         Runbook
      │             │
      └──→ Use for Python
          ecosystem
      
  PowerShell Path:
      │
  Long-running     YES ──→ PowerShell Workflow
  (>30 min)?                  (with checkpoints)
      │
     NO
      │
  Need Parallel?   YES ──→ PowerShell Workflow
                            (Parallel keyword)
      │
     NO
      │
  Simple &         YES ──→ PowerShell Runbook
  Fast?                     (standard PS)
```

### Recommendations by Scenario

| Scenario | Recommended Type | Reason |
|----------|------------------|--------|
| Start/stop VMs daily | PowerShell | Simple, fast, sequential |
| Provision 100 VMs simultaneously | PowerShell Workflow | Parallel processing |
| Long-running database migration | PowerShell Workflow | Checkpoints for resilience |
| AWS resource management | Python | boto3 library support |
| Data analysis and reporting | Python | pandas, numpy libraries |
| Approval workflow for managers | Graphical | Visual, non-technical users |
| REST API integration | Python or PowerShell | Both work; team expertise decides |
| Multi-step incident response | PowerShell Workflow | Checkpoints, parallel remediation |

## Best Practices

### 1. Runbook Development

**Use Source Control**:
- Store runbooks in Git repository (GitHub, Azure DevOps)
- Enable source control integration in Automation account
- Commit changes with descriptive messages
- Use branches for feature development

**Modular Design**:
```powershell
# Main runbook
workflow Backup-AllVMs {
    $vms = .\Get-VMs-By-Tag.ps1 -Tag "Backup" -Value "True"
    
    ForEach -Parallel ($vm in $vms) {
        .\Backup-Single-VM.ps1 -VMName $vm.Name
    }
    
    .\Send-Backup-Report.ps1 -VMs $vms
}
```

**Error Handling**:
```powershell
# PowerShell runbook with comprehensive error handling
try {
    $connection = Get-AutomationConnection -Name "AzureRunAsConnection"
    Connect-AzAccount -ServicePrincipal `
        -Tenant $connection.TenantID `
        -ApplicationId $connection.ApplicationID `
        -CertificateThumbprint $connection.CertificateThumbprint `
        -ErrorAction Stop
}
catch {
    Write-Error "Authentication failed: $_"
    throw
}

try {
    Start-AzVM -ResourceGroupName "rg-prod" -Name "vm-critical" -ErrorAction Stop
    Write-Output "VM started successfully"
}
catch {
    Write-Error "Failed to start VM: $_"
    
    # Send alert
    $webhookUrl = Get-AutomationVariable -Name "TeamsAlertWebhook"
    Invoke-RestMethod -Uri $webhookUrl -Method Post -Body (@{text="VM start failed: $_"} | ConvertTo-Json)
    
    throw
}
```

### 2. Testing Strategy

**Test Pane in Azure Portal**:
- Use Test Pane for draft testing (doesn't create job records)
- Test with various input parameters
- Verify output and logs

**Development Automation Account**:
- Create separate dev account for testing
- Test against non-production resources
- Validate before promoting to production

**Unit Testing** (for PowerShell):
```powershell
# Pester test for runbook function
Describe "Start-TaggedVMs" {
    It "Should start all VMs with AutoStart=True tag" {
        Mock Start-AzVM { return @{Status="Succeeded"} }
        Mock Get-AzVM { return @(@{Name="vm-01"; Tags=@{AutoStart="True"}}) }
        
        $result = .\Start-TaggedVMs.ps1 -ResourceGroup "rg-test"
        
        Assert-MockCalled Start-AzVM -Times 1
    }
}
```

### 3. Performance Optimization

**PowerShell**:
- Use `-NoWait` for asynchronous operations
- Batch API calls where possible
- Cache reusable data

```powershell
# BAD: Sequential with waiting
foreach ($vm in $vms) {
    Start-AzVM -ResourceGroupName $rg -Name $vm.Name  # Waits for completion
}

# GOOD: Parallel without waiting
foreach ($vm in $vms) {
    Start-AzVM -ResourceGroupName $rg -Name $vm.Name -NoWait  # Returns immediately
}
```

**PowerShell Workflow**:
- Use `Parallel` blocks for independent operations
- Tune `ThrottleLimit` based on resource constraints
- Place checkpoints strategically (not in loops)

### 4. Security

**Credential Management**:
```powershell
# Store credentials in Automation Credentials asset
# Retrieve in runbook
$credential = Get-AutomationPSCredential -Name "ServiceAccountCred"

# Use credential
Invoke-Command -ComputerName "server01" -Credential $credential -ScriptBlock {
    # Commands run as service account
}
```

**Sensitive Data**:
- Never hardcode passwords, API keys, or secrets
- Use Automation Variables (encrypted) or Azure Key Vault
- Enable diagnostic logs but mask sensitive output

**Least Privilege**:
- Grant Run As account minimum required permissions
- Use custom RBAC roles instead of Contributor
- Separate accounts for different security zones

## Next Steps

Now that you understand the four runbook types and when to use each, proceed to **Unit 4: Understand Automation Shared Resources** to learn how runbooks leverage shared assets like credentials, variables, and connections for reusable, maintainable automation.

**Quick Start Actions**:
1. Create a simple PowerShell runbook that lists Azure VMs
2. Test the runbook in Test Pane
3. Publish and execute on demand
4. Experiment with input parameters

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/3-what-runbook
