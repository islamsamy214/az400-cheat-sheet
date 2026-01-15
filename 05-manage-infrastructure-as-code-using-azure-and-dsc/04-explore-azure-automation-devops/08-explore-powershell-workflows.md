# Explore PowerShell Workflows

## Overview

PowerShell Workflow is a specialized type of runbook built on Windows Workflow Foundation (WF) that brings enterprise-grade resilience to Azure Automation. Unlike standard PowerShell scripts that execute sequentially and fail completely on errors, PowerShell Workflows can checkpoint their state, execute activities in parallel, automatically retry operations, and resume after interruptions.

This unit explores when and why to use PowerShell Workflows, their unique syntax and capabilities, and how they differ from regular PowerShell runbooks. You'll learn how workflows enable long-running, mission-critical automation that can survive system failures and network interruptions.

## What is Windows Workflow Foundation?

**Windows Workflow Foundation (WF)**:
- Microsoft framework for building workflow-enabled applications
- Built into .NET Framework (System.Activities namespace)
- Provides runtime engine for executing workflows
- Supports persistence (saving state to disk/database)
- Enables tracking, versioning, and compensation logic

**PowerShell Workflow Built on WF**:
```
PowerShell Workflow Syntax
    ↓ (compiled to)
XAML Workflow Definition
    ↓ (executed by)
Windows Workflow Foundation Runtime
    ↓ (produces)
Resilient, Long-Running Automation
```

**Key Concept**: PowerShell Workflow code is **compiled** into workflow activities, not directly executed like regular PowerShell. This enables persistence and resilience but introduces syntax restrictions.

## PowerShell Script vs PowerShell Workflow

### PowerShell Script (Regular Runbook)

**Characteristics**:
- **Execution**: Direct execution of PowerShell commands
- **State**: Runs in-memory, no persistence
- **Failure**: If system crashes, script must restart from beginning
- **Parallel**: No native parallel execution (requires jobs/runspaces)
- **Scope**: Single-machine scope (local session)

**Example**:
```powershell
# PowerShell Script
param([string[]]$ResourceGroupNames)

# Authenticate
Connect-AzAccount -Identity

# Process each resource group sequentially
foreach ($rgName in $ResourceGroupNames) {
    Write-Output "Processing: $rgName"
    
    # Get all VMs in resource group
    $vms = Get-AzVM -ResourceGroupName $rgName
    
    # Start each VM (one at a time)
    foreach ($vm in $vms) {
        Write-Output "  Starting: $($vm.Name)"
        Start-AzVM -ResourceGroupName $rgName -Name $vm.Name -NoWait
    }
}

# If script crashes here, must restart from beginning
```

### PowerShell Workflow Runbook

**Characteristics**:
- **Execution**: Compiled to workflow activities, executed by WF runtime
- **State**: Persisted to disk at checkpoints
- **Failure**: Resumes from last checkpoint after recovery
- **Parallel**: Native parallel execution with `Parallel` keyword
- **Scope**: Multi-machine, can target hundreds of nodes simultaneously

**Equivalent Workflow**:
```powershell
workflow Start-VirtualMachines {
    param([string[]]$ResourceGroupNames)
    
    # Authenticate (in workflow context)
    $AzConnection = Get-AutomationConnection -Name "AzureRunAsConnection"
    Connect-AzAccount -ServicePrincipal `
        -Tenant $AzConnection.TenantId `
        -ApplicationId $AzConnection.ApplicationId `
        -CertificateThumbprint $AzConnection.CertificateThumbprint
    
    # Checkpoint: Save state before processing
    Checkpoint-Workflow
    
    # Process resource groups in parallel
    ForEach -Parallel ($rgName in $ResourceGroupNames) {
        Write-Output "Processing: $rgName"
        
        # Get all VMs (within parallel block)
        $vms = InlineScript {
            Get-AzVM -ResourceGroupName $using:rgName
        }
        
        # Start VMs in parallel (up to 10 concurrent)
        ForEach -Parallel -ThrottleLimit 10 ($vm in $vms) {
            Write-Output "  Starting: $($vm.Name)"
            InlineScript {
                Start-AzVM -ResourceGroupName $using:rgName -Name $using:vm.Name -NoWait
            }
        }
        
        # Checkpoint after each resource group
        Checkpoint-Workflow
    }
}

# If workflow crashes, resumes from last checkpoint
# Parallel execution: All resource groups processed simultaneously
```

## Workflow Characteristics

### 1. Long-Running Operations

**Standard PowerShell Limitation**:
- Maximum runtime: 3 hours in Azure Automation (fair share limit)
- No persistence: Crash = restart from beginning
- No automatic retry

**PowerShell Workflow Solution**:
- Can run for days/weeks (with checkpoints)
- Persists state: Crash = resume from checkpoint
- Automatic retry on transient failures

**Example: Backup 1,000 Databases**:

```powershell
workflow Backup-ThousandDatabases {
    param([string[]]$DatabaseServers)  # Array of 100 servers, 10 DBs each
    
    $backupCount = 0
    
    ForEach -Parallel ($server in $DatabaseServers) {
        # Get databases on this server
        $databases = InlineScript {
            Get-SqlDatabase -ServerInstance $using:server
        }
        
        # Backup each database
        ForEach ($db in $databases) {
            InlineScript {
                Backup-SqlDatabase -ServerInstance $using:server -Database $using:db.Name
            }
            
            $backupCount++
            
            # Checkpoint every 50 backups (resume point if failure)
            if ($backupCount % 50 -eq 0) {
                Checkpoint-Workflow
                Write-Output "Checkpoint: $backupCount backups completed"
            }
        }
    }
    
    Write-Output "✓ All 1,000 databases backed up"
}

# Runtime: Several hours
# Resilience: If server crashes at backup #523, resumes from checkpoint at #500
# Parallel: 100 servers processed simultaneously
```

### 2. Repeated Operations

**Scenario**: Monitoring task that checks system health every 5 minutes indefinitely.

**PowerShell Script Approach** (unreliable):
```powershell
# Runs until crash or 3-hour timeout
while ($true) {
    Check-SystemHealth
    Start-Sleep -Seconds 300
}
# Problem: No persistence, no automatic restart
```

**PowerShell Workflow Approach** (reliable):
```powershell
workflow Monitor-SystemHealth {
    # Run indefinitely with checkpoints
    while ($true) {
        InlineScript {
            # Check health
            $health = Test-AzResourceHealth
            if ($health.Status -ne "Available") {
                Send-AlertEmail -Status $health.Status
            }
        }
        
        # Checkpoint before sleeping (enables resume)
        Checkpoint-Workflow
        
        # Sleep 5 minutes
        Start-Sleep -Seconds 300
    }
}
```

### 3. Parallel Execution

**Without Workflow** (complex, verbose):
```powershell
# PowerShell Script: Manual parallel execution
$jobs = @()
foreach ($vm in $vms) {
    $jobs += Start-Job -ScriptBlock {
        param($vmName, $rgName)
        Start-AzVM -ResourceGroupName $rgName -Name $vmName
    } -ArgumentList $vm.Name, $resourceGroupName
}

# Wait for all jobs
$jobs | Wait-Job
$jobs | Receive-Job
$jobs | Remove-Job
```

**With Workflow** (simple, declarative):
```powershell
workflow Start-AllVMs {
    param($vms, $resourceGroupName)
    
    ForEach -Parallel ($vm in $vms) {
        InlineScript {
            Start-AzVM -ResourceGroupName $using:resourceGroupName -Name $using:vm.Name
        }
    }
}
```

### 4. Interruptible Tasks

**Scenario**: Deploy software to 500 servers. Deployment paused for maintenance window, then resumed.

**PowerShell Workflow**:
```powershell
workflow Deploy-SoftwareToServers {
    param([string[]]$Servers)
    
    $deployedCount = 0
    
    ForEach -Parallel -ThrottleLimit 20 ($server in $Servers) {
        # Checkpoint every 10 deployments
        if ($deployedCount % 10 -eq 0) {
            Checkpoint-Workflow
        }
        
        InlineScript {
            # Deploy software (may take 10 minutes per server)
            Invoke-SoftwareDeployment -Server $using:server
        }
        
        $deployedCount++
    }
}

# Execution:
# 1. Workflow deploys to 150 servers (75 minutes)
# 2. Administrator suspends workflow for maintenance window
# 3. Maintenance completes
# 4. Administrator resumes workflow
# 5. Workflow continues from server #151 (not restarting from #1)
```

### 5. Continue After Interruption

**Automatic Retry on Transient Failures**:

```powershell
workflow Update-AzureResources {
    param([string[]]$ResourceGroups)
    
    ForEach -Parallel ($rg in $ResourceGroups) {
        # Automatic retry on network failures
        $retryCount = 0
        $maxRetries = 3
        
        do {
            try {
                InlineScript {
                    # This may fail due to transient network issue
                    Update-AzResourceGroup -Name $using:rg -Tag @{Updated=(Get-Date)}
                }
                break  # Success, exit retry loop
            }
            catch {
                $retryCount++
                if ($retryCount -lt $maxRetries) {
                    Write-Output "Retry $retryCount for $rg after error: $_"
                    Start-Sleep -Seconds (5 * $retryCount)  # Exponential backoff
                }
                else {
                    throw "Failed after $maxRetries retries: $_"
                }
            }
        } while ($retryCount -lt $maxRetries)
        
        Checkpoint-Workflow
    }
}
```

### 6. Checkpoint Support

**What is a Checkpoint?**
- Snapshot of workflow state (variables, current activity, output)
- Saved to Azure Storage (persisted)
- Enables resume after failure

**When Checkpoints Are Useful**:
1. **Long-running workflows**: Resume after timeout/crash
2. **Expensive operations**: Don't repeat completed work
3. **Multi-stage processes**: Clearly defined resume points
4. **Debugging**: Suspend/resume for troubleshooting

**Checkpoint Example**:

```powershell
workflow Provision-ComplexEnvironment {
    param([string]$EnvironmentName)
    
    # Stage 1: Create resource group
    InlineScript {
        New-AzResourceGroup -Name $using:EnvironmentName -Location "East US"
    }
    Checkpoint-Workflow  # ✓ Resource group created
    
    # Stage 2: Deploy virtual network (5 minutes)
    InlineScript {
        New-AzVirtualNetwork -Name "$using:EnvironmentName-vnet" `
            -ResourceGroupName $using:EnvironmentName -AddressPrefix "10.0.0.0/16"
    }
    Checkpoint-Workflow  # ✓ VNet created
    
    # Stage 3: Deploy VMs (20 minutes)
    $vmNames = @("web01", "web02", "db01")
    ForEach -Parallel ($vmName in $vmNames) {
        InlineScript {
            New-AzVM -Name $using:vmName -ResourceGroupName $using:EnvironmentName
        }
    }
    Checkpoint-Workflow  # ✓ VMs created
    
    # Stage 4: Configure load balancer (10 minutes)
    InlineScript {
        New-AzLoadBalancer -Name "$using:EnvironmentName-lb" `
            -ResourceGroupName $using:EnvironmentName
    }
    Checkpoint-Workflow  # ✓ Load balancer created
    
    # If workflow crashes during Stage 3 (VM deployment):
    # - Stage 1 (RG) NOT re-executed
    # - Stage 2 (VNet) NOT re-executed
    # - Stage 3 (VMs) resumes from checkpoint
}
```

## Workflow Benefits

### 1. PowerShell Scripting Syntax

**Familiar to PowerShell Developers**:
```powershell
workflow My-Workflow {
    # Looks like regular PowerShell
    param([string]$Parameter1)
    
    $variable = "Hello"
    Write-Output $variable
    
    if ($Parameter1 -eq "test") {
        # Conditional logic works
    }
    
    foreach ($item in @(1,2,3)) {
        # Loops work (with caveats)
    }
}
```

**Key Difference**: Workflows have restrictions (covered in Unit 9).

### 2. Multi-Device Management

**Scenario**: Configure 500 on-premises servers simultaneously.

```powershell
workflow Configure-Servers {
    param([string[]]$ComputerNames)  # 500 server names
    
    # Execute configuration on all servers in parallel
    ForEach -Parallel -ThrottleLimit 50 ($computer in $ComputerNames) {
        InlineScript {
            # This runs on the target computer
            Invoke-Command -ComputerName $using:computer -ScriptBlock {
                # Install software
                Install-WindowsFeature -Name Web-Server
                
                # Configure firewall
                New-NetFirewallRule -DisplayName "Allow HTTP" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow
                
                # Start service
                Start-Service -Name W3SVC
            }
        }
    }
}

# Executes on 50 servers simultaneously (ThrottleLimit)
# Completes in minutes instead of hours
```

### 3. Single Task Runs Multiple Scripts

**Scenario**: Execute different scripts based on server role.

```powershell
workflow Configure-ServersByRole {
    param(
        [string[]]$WebServers,
        [string[]]$DatabaseServers,
        [string[]]$AppServers
    )
    
    Parallel {
        # All three server types configured simultaneously
        
        # Web servers configuration
        ForEach -Parallel ($server in $WebServers) {
            InlineScript {
                Invoke-Command -ComputerName $using:server -ScriptBlock {
                    Install-WindowsFeature -Name Web-Server, Web-Asp-Net45
                    Copy-Item "\\share\webapps\*" -Destination "C:\inetpub\wwwroot" -Recurse
                }
            }
        }
        
        # Database servers configuration
        ForEach -Parallel ($server in $DatabaseServers) {
            InlineScript {
                Invoke-Command -ComputerName $using:server -ScriptBlock {
                    Install-Module -Name SqlServer -Force
                    Invoke-Sqlcmd -InputFile "\\share\scripts\configure-db.sql"
                }
            }
        }
        
        # App servers configuration
        ForEach -Parallel ($server in $AppServers) {
            InlineScript {
                Invoke-Command -ComputerName $using:server -ScriptBlock {
                    Install-WindowsFeature -Name NET-Framework-45-Features
                    Copy-Item "\\share\apps\*" -Destination "C:\Apps" -Recurse
                }
            }
        }
    }
    
    Checkpoint-Workflow
    Write-Output "✓ All servers configured"
}
```

### 4. Automated Failure Recovery

**Built-in Retry Logic**:

```powershell
workflow Download-LargeFiles {
    param([string[]]$FileUrls)
    
    ForEach -Parallel ($url in $FileUrls) {
        $downloaded = $false
        $attempts = 0
        $maxAttempts = 5
        
        while (-not $downloaded -and $attempts -lt $maxAttempts) {
            try {
                $attempts++
                
                InlineScript {
                    # Download may fail due to network issues
                    Invoke-WebRequest -Uri $using:url -OutFile "C:\Downloads\$($using:url.Split('/')[-1])"
                }
                
                $downloaded = $true
                Write-Output "✓ Downloaded: $url"
            }
            catch {
                Write-Output "Attempt $attempts failed for $url : $_"
                
                if ($attempts -lt $maxAttempts) {
                    # Exponential backoff: 2s, 4s, 8s, 16s
                    Start-Sleep -Seconds ([Math]::Pow(2, $attempts))
                }
                else {
                    Write-Error "Failed to download $url after $maxAttempts attempts"
                }
            }
        }
    }
}
```

### 5. Connection and Activity Retries

**Automatic Connection Retry**:

```powershell
workflow Process-AzureResources {
    param([string[]]$ResourceGroups)
    
    # Workflow automatically retries Connect-AzAccount on failure
    $AzConnection = Get-AutomationConnection -Name "AzureRunAsConnection"
    
    # If network drops during authentication, workflow retries automatically
    Connect-AzAccount -ServicePrincipal `
        -Tenant $AzConnection.TenantId `
        -ApplicationId $AzConnection.ApplicationId `
        -CertificateThumbprint $AzConnection.CertificateThumbprint
    
    ForEach -Parallel ($rg in $ResourceGroups) {
        # Each Azure cmdlet automatically retries on transient failures
        InlineScript {
            Get-AzResource -ResourceGroupName $using:rg | ForEach-Object {
                # Automatic retry if network drops mid-operation
                Set-AzResource -ResourceId $_.ResourceId -Tag @{Processed=$true} -Force
            }
        }
    }
}
```

### 6. Connect and Disconnect Capability

**Persistent Connections**:

```powershell
workflow Manage-RemoteServers {
    param([string[]]$Servers)
    
    # Establish PSSession to each server
    $sessions = InlineScript {
        $using:Servers | ForEach-Object {
            New-PSSession -ComputerName $_ -Credential (Get-AutomationPSCredential -Name "AdminCreds")
        }
    }
    
    Checkpoint-Workflow  # Sessions persisted
    
    # If workflow suspends here, sessions are saved
    # When workflow resumes, sessions are reconnected
    
    ForEach -Parallel ($session in $sessions) {
        InlineScript {
            # Execute commands using persisted session
            Invoke-Command -Session $using:session -ScriptBlock {
                Get-Service | Where-Object Status -eq "Stopped"
            }
        }
    }
    
    # Cleanup: Disconnect sessions
    InlineScript {
        $using:sessions | Remove-PSSession
    }
}
```

### 7. Task Scheduling

**Scheduled Execution with State**:

```powershell
workflow Schedule-MaintenanceTasks {
    # Task 1: Morning backup (8 AM)
    $currentHour = (Get-Date).Hour
    if ($currentHour -eq 8) {
        InlineScript {
            Backup-AzureResources
        }
        Checkpoint-Workflow
    }
    
    # Task 2: Midday health check (12 PM)
    if ($currentHour -eq 12) {
        InlineScript {
            Test-SystemHealth
        }
        Checkpoint-Workflow
    }
    
    # Task 3: Evening cleanup (6 PM)
    if ($currentHour -eq 18) {
        InlineScript {
            Remove-OldBackups -DaysOld 30
        }
        Checkpoint-Workflow
    }
    
    # Task 4: Night shutdown (10 PM)
    if ($currentHour -eq 22) {
        InlineScript {
            Stop-AzVM -ResourceGroupName "dev-resources" -Name "*" -Force
        }
        Checkpoint-Workflow
    }
}

# Schedule this workflow to run every hour
# Checkpoint ensures state is maintained across executions
```

## When to Use PowerShell Workflows

### ✅ Use PowerShell Workflow When:

1. **Long-running operations** (> 30 minutes)
   - Multi-server deployments
   - Large-scale backups
   - Data migrations

2. **Parallel processing required**
   - Hundreds of Azure resources
   - Multi-region deployments
   - Concurrent server configuration

3. **Resilience critical**
   - Mission-critical automation
   - Expensive operations (can't afford to restart)
   - Network-dependent tasks (may experience intermittent failures)

4. **Multi-machine targeting**
   - Configure on-premises servers
   - Deploy to Hybrid Runbook Workers
   - Orchestrate across datacenters

5. **State persistence needed**
   - Checkpointed workflows
   - Resume after interruption
   - Audit trail of workflow stages

### ❌ Use Regular PowerShell When:

1. **Quick tasks** (< 5 minutes)
   - Start/stop single VM
   - Query resource properties
   - Simple notifications

2. **Complex PowerShell features needed**
   - Advanced function features (begin/process/end blocks)
   - .NET interop
   - Custom objects with methods

3. **Rapid iteration/debugging**
   - Workflow compilation slows development
   - Simpler syntax for testing

4. **Workflow restrictions problematic**
   - Need to use `$_` in pipelines
   - Dynamic variable scopes
   - Specific PowerShell language features not supported in workflows

## Performance Considerations

### Workflow Overhead

**Compilation Time**:
- Workflows compiled to XAML (adds 5-10 seconds startup time)
- Regular scripts execute immediately

**Memory Usage**:
- Workflows maintain state (higher memory footprint)
- Checkpoints require serialization (adds overhead)

**When Overhead Acceptable**:
- Long-running tasks (10+ minutes): overhead negligible
- Parallel execution gains outweigh compilation cost
- Resilience requirements justify overhead

**When Overhead Problematic**:
- Sub-minute tasks: overhead is significant portion of runtime
- High-frequency executions (every 30 seconds): compilation cost multiplied

### Optimal Workflow Structure

```powershell
workflow Optimal-Structure {
    param([string[]]$Targets)
    
    # ✅ GOOD: Minimize InlineScript blocks
    # Wrap large operations, not individual cmdlets
    
    ForEach -Parallel ($target in $Targets) {
        # ❌ BAD: Many small InlineScript blocks
        # InlineScript { $vm = Get-AzVM -Name $target }
        # InlineScript { Start-AzVM -Name $vm.Name }
        # InlineScript { Set-AzVMTag -VM $vm -Tag @{Started=$true} }
        
        # ✅ GOOD: Single InlineScript block
        InlineScript {
            $vm = Get-AzVM -Name $using:target
            Start-AzVM -Name $vm.Name -NoWait
            Set-AzResource -ResourceId $vm.Id -Tag @{Started=(Get-Date)} -Force
        }
    }
}
```

## Syntax Differences: Script vs Workflow

| Feature | PowerShell Script | PowerShell Workflow |
|---------|-------------------|---------------------|
| **Parallel execution** | Manual (jobs/runspaces) | `ForEach -Parallel` keyword |
| **Checkpoints** | Not supported | `Checkpoint-Workflow` |
| **Variable scope** | Dynamic scoping | Lexical scoping (use `$using:`) |
| **Pipeline** | Full support | Limited (use InlineScript) |
| **Return** | `return` keyword | Not supported (use output) |
| **$_** | Automatic variable | Not supported (use named variable) |
| **Function definitions** | Inside script | Must be in InlineScript |
| **.NET classes** | Direct access | Use InlineScript |
| **Execution** | Interpreted | Compiled to XAML |

**More details in Unit 9: Create a Workflow (syntax deep dive).**

## Next Steps

Now that you understand the "why" of PowerShell Workflows (resilience, parallel processing, checkpoints), proceed to **Unit 9: Create a Workflow** to learn the "how" — detailed syntax, InlineScript blocks, Parallel keywords, and practical workflow authoring.

**Quick Comparison**:
- **Unit 8 (this unit)**: Why use workflows? When to use workflows?
- **Unit 9 (next)**: How to write workflows? Syntax and features?

**Quick Decision Matrix**:

```
Need to run for > 30 minutes? → Use Workflow
Need parallel execution? → Use Workflow
Need checkpoints/resume? → Use Workflow
Simple task < 5 minutes? → Use Script
Need advanced PowerShell features? → Use Script
```

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/8-explore-powershell-workflows
