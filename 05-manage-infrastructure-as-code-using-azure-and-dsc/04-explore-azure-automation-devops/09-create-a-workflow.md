# Create a Workflow

## Overview

This unit provides the practical guide to authoring PowerShell Workflow runbooks. While Unit 8 covered *why* and *when* to use workflows, this unit focuses on *how* to write them: workflow syntax, key features (checkpoints, parallel processing, InlineScript), variable scoping, and common patterns.

You'll learn the critical differences between PowerShell scripts and workflows, understand how to work around workflow restrictions, and master the three key workflow features that make them powerful: checkpoints for resilience, parallel execution for performance, and InlineScript for advanced PowerShell capabilities.

## Workflow Syntax Basics

### Workflow Keyword

**Defining a Workflow**:
```powershell
workflow Verb-Noun {
    # Workflow body
}
```

**Key Differences from Functions**:

| Aspect | PowerShell Function | PowerShell Workflow |
|--------|---------------------|---------------------|
| **Keyword** | `function Verb-Noun { }` | `workflow Verb-Noun { }` |
| **Execution** | Interpreted directly | Compiled to XAML, then executed |
| **State** | In-memory only | Persisted at checkpoints |
| **Parallel** | Not built-in | `ForEach -Parallel` |
| **Restrictions** | Minimal | Many (covered below) |

### Naming Convention

**Follow PowerShell Verb-Noun Pattern**:
```powershell
# ✅ GOOD: Verb-Noun naming
workflow Start-VirtualMachines { }
workflow Backup-Databases { }
workflow Deploy-Application { }

# ❌ BAD: Non-standard names
workflow VMStarter { }
workflow BackupScript { }
workflow Deploy { }
```

**Approved Verbs**: Use `Get-Verb` to see approved verbs:
```powershell
Get-Verb | Select-Object -First 10
# Verb     AliasPrefix Group
# Add      a           Common
# Clear    cl          Common
# Close    cs          Common
# Copy     cp          Common
# Enter    et          Common
# Exit     ex          Common
# ...
```

### Parameters

**Parameter Declaration**:
```powershell
workflow Process-ResourceGroups {
    param(
        [Parameter(Mandatory=$true)]
        [string[]]$ResourceGroupNames,
        
        [Parameter(Mandatory=$false)]
        [string]$Location = "East US",
        
        [ValidateSet("Start", "Stop", "Restart")]
        [string]$Action = "Start",
        
        [int]$ThrottleLimit = 10
    )
    
    # Workflow body
    Write-Output "Processing $($ResourceGroupNames.Count) resource groups"
    Write-Output "Action: $Action"
    Write-Output "Location: $Location"
}
```

**Parameter Best Practices**:
- Always use `param()` block at top of workflow
- Use `[Parameter(Mandatory=$true)]` for required parameters
- Provide default values for optional parameters
- Use `[ValidateSet()]` for enumerated options
- Document parameters with comments

### Adding Commands to Workflows

**Basic Command Execution**:
```powershell
workflow My-FirstWorkflow {
    # Simple commands work directly
    Write-Output "Workflow started"
    
    $timestamp = Get-Date
    Write-Output "Current time: $timestamp"
    
    # Variables
    $count = 0
    $count++
    Write-Output "Count: $count"
    
    # Conditional logic
    if ($count -gt 0) {
        Write-Output "Count is positive"
    }
    
    # Loops
    foreach ($number in 1..5) {
        Write-Output "Number: $number"
    }
}
```

**Important**: Many PowerShell cmdlets and features work in workflows, but with restrictions (detailed below).

## Complete Workflow Example

### MyFirstRunbook-Workflow

**Comprehensive Example**:
```powershell
workflow MyFirstRunbook-Workflow {
    param(
        [Parameter(Mandatory=$true)]
        [string]$ResourceGroupName,
        
        [Parameter(Mandatory=$false)]
        [ValidateSet("Start", "Stop")]
        [string]$Action = "Start"
    )
    
    # Authenticate to Azure
    Write-Output "Authenticating to Azure..."
    $AzConnection = Get-AutomationConnection -Name "AzureRunAsConnection"
    
    Connect-AzAccount -ServicePrincipal `
        -Tenant $AzConnection.TenantId `
        -ApplicationId $AzConnection.ApplicationId `
        -CertificateThumbprint $AzConnection.CertificateThumbprint
    
    Write-Output "✓ Authenticated successfully"
    
    # Checkpoint before processing (enables resume)
    Checkpoint-Workflow
    
    # Get all VMs in resource group
    Write-Output "Retrieving VMs from resource group: $ResourceGroupName"
    
    $vms = InlineScript {
        Get-AzVM -ResourceGroupName $using:ResourceGroupName -Status
    }
    
    Write-Output "Found $($vms.Count) VMs"
    
    # Process each VM in parallel
    ForEach -Parallel -ThrottleLimit 5 ($vm in $vms) {
        $vmName = $vm.Name
        $vmStatus = ($vm.Statuses | Where-Object Code -like "PowerState/*").Code
        
        Write-Output "Processing VM: $vmName (Current status: $vmStatus)"
        
        if ($Action -eq "Start") {
            if ($vmStatus -eq "PowerState/deallocated" -or $vmStatus -eq "PowerState/stopped") {
                Write-Output "  Starting $vmName..."
                
                InlineScript {
                    Start-AzVM -ResourceGroupName $using:ResourceGroupName -Name $using:vmName -NoWait
                }
                
                Write-Output "  ✓ Start command issued for $vmName"
            }
            else {
                Write-Output "  ⊘ $vmName already running, skipping"
            }
        }
        elseif ($Action -eq "Stop") {
            if ($vmStatus -eq "PowerState/running") {
                Write-Output "  Stopping $vmName..."
                
                InlineScript {
                    Stop-AzVM -ResourceGroupName $using:ResourceGroupName -Name $using:vmName -Force -NoWait
                }
                
                Write-Output "  ✓ Stop command issued for $vmName"
            }
            else {
                Write-Output "  ⊘ $vmName already stopped, skipping"
            }
        }
    }
    
    # Checkpoint after all VMs processed
    Checkpoint-Workflow
    
    Write-Output "✓ Workflow completed: $Action action applied to $($vms.Count) VMs"
}
```

**Usage**:
```powershell
# Start all VMs in resource group
MyFirstRunbook-Workflow -ResourceGroupName "rg-prod-vms" -Action "Start"

# Stop all VMs in resource group
MyFirstRunbook-Workflow -ResourceGroupName "rg-dev-vms" -Action "Stop"
```

## Key Workflow Features

### 1. Checkpoints

**What is Checkpoint-Workflow?**

`Checkpoint-Workflow` saves the current state of the workflow to persistent storage (Azure Storage). If the workflow fails or is suspended, it can resume from the last checkpoint instead of restarting from the beginning.

**What Gets Saved in Checkpoint**:
- All workflow variables and their values
- Current position in workflow execution
- Output generated so far
- Parallel execution state

**Syntax**:
```powershell
workflow Use-Checkpoints {
    # Stage 1: Initialize
    $resourceCount = 0
    Write-Output "Initialization complete"
    
    Checkpoint-Workflow  # Save state after initialization
    
    # Stage 2: Process resources (long-running)
    for ($i = 1; $i -le 100; $i++) {
        InlineScript {
            # Simulate processing (e.g., deploy resource)
            Start-Sleep -Seconds 2
        }
        $resourceCount++
        
        # Checkpoint every 10 resources
        if ($resourceCount % 10 -eq 0) {
            Checkpoint-Workflow
            Write-Output "Checkpoint: $resourceCount resources processed"
        }
    }
    
    # Stage 3: Finalization
    Write-Output "All $resourceCount resources processed"
    Checkpoint-Workflow
}
```

**Checkpoint Placement Strategies**:

1. **After expensive operations**:
   ```powershell
   workflow Deploy-Infrastructure {
       # Deploy VNet (expensive, 5 minutes)
       InlineScript { New-AzVirtualNetwork ... }
       Checkpoint-Workflow  # Don't repeat VNet creation if failure occurs later
       
       # Deploy VMs (expensive, 20 minutes)
       InlineScript { New-AzVM ... }
       Checkpoint-Workflow  # Don't repeat VM creation if failure occurs later
   }
   ```

2. **After logical stages**:
   ```powershell
   workflow Multi-Stage-Deployment {
       # Stage 1: Infrastructure
       Deploy-Infrastructure
       Checkpoint-Workflow
       
       # Stage 2: Configuration
       Configure-Resources
       Checkpoint-Workflow
       
       # Stage 3: Validation
       Test-Deployment
       Checkpoint-Workflow
   }
   ```

3. **Inside loops (periodic checkpoints)**:
   ```powershell
   workflow Process-ManyItems {
       param([array]$Items)  # 1000 items
       
       $processed = 0
       foreach ($item in $Items) {
           Process-Item $item
           $processed++
           
           # Checkpoint every 50 items
           if ($processed % 50 -eq 0) {
               Checkpoint-Workflow
           }
       }
   }
   ```

**Checkpoint Overhead**:
- Each checkpoint serializes workflow state to storage (adds 1-3 seconds)
- Don't checkpoint too frequently (inside tight loops: bad)
- Balance between resilience and performance

**Best Practice**:
```powershell
# ❌ BAD: Checkpoint in tight loop (too frequent)
foreach ($item in $largeArray) {  # 10,000 items
    Process-Item $item
    Checkpoint-Workflow  # 10,000 checkpoints! Huge overhead!
}

# ✅ GOOD: Checkpoint periodically
$count = 0
foreach ($item in $largeArray) {
    Process-Item $item
    $count++
    
    if ($count % 100 -eq 0) {  # Every 100 items
        Checkpoint-Workflow
    }
}
```

### 2. Parallel Keyword

**Basic Parallel Execution**:

```powershell
workflow Start-MultipleVMs {
    param([string[]]$VMNames)
    
    # Execute in parallel (all VMs started simultaneously)
    ForEach -Parallel ($vmName in $VMNames) {
        InlineScript {
            Write-Output "Starting: $using:vmName"
            Start-AzVM -Name $using:vmName -ResourceGroupName "rg-vms"
        }
    }
}
```

**Parallel Block**:

The `Parallel` keyword creates a block where all activities execute concurrently:

```powershell
workflow Deploy-ThreeTierApp {
    Parallel {
        # All three activities run simultaneously
        
        # Activity 1: Deploy web tier
        InlineScript {
            Write-Output "Deploying web tier..."
            New-AzVM -Name "web01" -ResourceGroupName "rg-app"
        }
        
        # Activity 2: Deploy app tier
        InlineScript {
            Write-Output "Deploying app tier..."
            New-AzVM -Name "app01" -ResourceGroupName "rg-app"
        }
        
        # Activity 3: Deploy database tier
        InlineScript {
            Write-Output "Deploying database tier..."
            New-AzVM -Name "db01" -ResourceGroupName "rg-app"
        }
    }
    
    # Workflow waits here until all three activities complete
    Write-Output "All tiers deployed"
}
```

**Sequence Inside Parallel**:

You can have sequential execution within parallel blocks:

```powershell
workflow Complex-Parallel {
    Parallel {
        # Branch 1: Sequential steps
        Sequence {
            InlineScript { Write-Output "Branch 1: Step 1" }
            InlineScript { Write-Output "Branch 1: Step 2" }
            InlineScript { Write-Output "Branch 1: Step 3" }
        }
        
        # Branch 2: Sequential steps (runs in parallel with Branch 1)
        Sequence {
            InlineScript { Write-Output "Branch 2: Step 1" }
            InlineScript { Write-Output "Branch 2: Step 2" }
        }
        
        # Branch 3: Single step (runs in parallel with Branches 1 and 2)
        InlineScript { Write-Output "Branch 3: Single step" }
    }
}
```

### 3. ThrottleLimit Parameter

**Controlling Parallel Concurrency**:

`ThrottleLimit` controls how many parallel iterations execute simultaneously:

```powershell
workflow Start-VMs-ThrottleLimited {
    param([string[]]$VMNames)  # 100 VMs
    
    # Start 10 VMs at a time (not all 100 simultaneously)
    ForEach -Parallel -ThrottleLimit 10 ($vmName in $VMNames) {
        InlineScript {
            Start-AzVM -Name $using:vmName -ResourceGroupName "rg-vms"
        }
    }
}

# Execution:
# Iteration 1-10: Start simultaneously
# Iteration 11: Starts when iteration 1 completes
# Iteration 12: Starts when iteration 2 completes
# ...
# All 100 VMs started, but never more than 10 concurrent operations
```

**Why ThrottleLimit?**

1. **Resource Limits**: Prevent overwhelming target systems
   ```powershell
   # Without throttle: 500 simultaneous connections to SQL server (overload)
   # With throttle 20: Max 20 connections (manageable)
   ```

2. **API Rate Limits**: Azure Resource Manager has rate limits (800 requests per 5 minutes per subscription)
   ```powershell
   # 1000 VMs, no throttle: Exceeds ARM rate limits (failures)
   # 1000 VMs, throttle 50: Stays under rate limits
   ```

3. **Memory/CPU Constraints**: Runbook worker has finite resources
   ```powershell
   # Each parallel iteration consumes memory
   # Too many concurrent: Worker runs out of memory
   ```

**Choosing ThrottleLimit**:

| Scenario | Recommended ThrottleLimit | Reasoning |
|----------|---------------------------|-----------|
| **Azure VM operations** | 10-20 | ARM throttling, VM operation time |
| **SQL queries** | 5-10 | Database connection limits |
| **File operations** | 20-50 | I/O bound, network bandwidth |
| **API calls (external)** | 5-20 | API rate limits |
| **Heavy computation** | CPU count | Maximize CPU utilization |

**Tuning Example**:
```powershell
workflow Tune-ThrottleLimit {
    param([string[]]$Targets)  # 200 targets
    
    # Start with conservative limit
    $throttle = 10
    
    # Adjust based on target system capacity
    if ($Targets.Count -gt 500) {
        $throttle = 20  # More targets, increase concurrency
    }
    
    ForEach -Parallel -ThrottleLimit $throttle ($target in $Targets) {
        InlineScript {
            Invoke-Operation -Target $using:target
        }
    }
}
```

### 4. InlineScript Blocks

**Why InlineScript?**

Workflows have syntax restrictions. `InlineScript` allows standard PowerShell execution inside workflows:

```powershell
workflow Use-InlineScript {
    # This is workflow syntax (restricted)
    
    InlineScript {
        # This is standard PowerShell (unrestricted)
        # Full PowerShell language features available
    }
}
```

**What Works in InlineScript (but not in workflow)**:

1. **Pipeline with $_**:
   ```powershell
   workflow Pipeline-Example {
       InlineScript {
           # ✅ Works: $_ in pipeline
           Get-AzVM | Where-Object { $_.Name -like "web*" }
       }
       
       # ❌ Doesn't work: $_ not available in workflow
       # $vms = Get-AzVM | Where-Object { $_.Name -like "web*" }
   }
   ```

2. **Advanced Functions**:
   ```powershell
   workflow Function-Example {
       InlineScript {
           # ✅ Works: Function definition
           function Get-CustomData {
               [CmdletBinding()]
               param([string]$Name)
               
               begin { Write-Verbose "Starting" }
               process { Write-Output "Processing $Name" }
               end { Write-Verbose "Ending" }
           }
           
           Get-CustomData -Name "Test"
       }
   }
   ```

3. **.NET Interop**:
   ```powershell
   workflow DotNet-Example {
       InlineScript {
           # ✅ Works: Direct .NET calls
           $timestamp = [System.DateTime]::Now.ToString("yyyy-MM-dd HH:mm:ss")
           $guid = [System.Guid]::NewGuid().ToString()
           
           [System.IO.File]::WriteAllText("C:\Temp\log.txt", "Log entry")
       }
   }
   ```

4. **Complex Object Methods**:
   ```powershell
   workflow Object-Methods {
       InlineScript {
           # ✅ Works: PSCustomObject with methods
           $obj = [PSCustomObject]@{
               Name = "Server01"
               GetInfo = { "Server: $($this.Name)" }
           }
           
           & $obj.GetInfo
       }
   }
   ```

**Passing Variables to InlineScript**:

Use `$using:` scope modifier:

```powershell
workflow Variable-Scoping {
    param([string]$ResourceGroup)
    
    $vmName = "vm01"
    $location = "East US"
    
    InlineScript {
        # Access workflow variables with $using:
        Write-Output "Resource Group: $using:ResourceGroup"
        Write-Output "VM Name: $using:vmName"
        Write-Output "Location: $using:location"
        
        # Create VM using workflow variables
        New-AzVM -ResourceGroupName $using:ResourceGroup `
                 -Name $using:vmName `
                 -Location $using:location
    }
}
```

**Returning Values from InlineScript**:

InlineScript output becomes workflow variable:

```powershell
workflow Return-From-InlineScript {
    # Get VM count
    $vmCount = InlineScript {
        $vms = Get-AzVM -ResourceGroupName "rg-prod"
        $vms.Count  # Last statement is returned
    }
    
    Write-Output "VM Count: $vmCount"
    
    # Get complex object
    $vmData = InlineScript {
        Get-AzVM -ResourceGroupName "rg-prod" -Status | Select-Object Name, PowerState
    }
    
    # Use returned data (array of objects)
    foreach ($vm in $vmData) {
        Write-Output "$($vm.Name): $($vm.PowerState)"
    }
}
```

**InlineScript Best Practices**:

1. **Minimize InlineScript blocks** (each adds overhead):
   ```powershell
   # ❌ BAD: Many small InlineScript blocks
   workflow Bad-Example {
       $vm1 = InlineScript { Get-AzVM -Name "vm01" }
       $vm2 = InlineScript { Get-AzVM -Name "vm02" }
       $vm3 = InlineScript { Get-AzVM -Name "vm03" }
   }
   
   # ✅ GOOD: One InlineScript block
   workflow Good-Example {
       $vms = InlineScript {
           $vm1 = Get-AzVM -Name "vm01"
           $vm2 = Get-AzVM -Name "vm02"
           $vm3 = Get-AzVM -Name "vm03"
           @($vm1, $vm2, $vm3)  # Return array
       }
   }
   ```

2. **Use InlineScript for complex operations** only:
   ```powershell
   workflow When-To-Use-InlineScript {
       # ✅ Simple operations: No InlineScript needed
       $count = 0
       $count++
       Write-Output "Count: $count"
       
       # ✅ Complex operations: Use InlineScript
       $result = InlineScript {
           Get-AzResource | 
               Where-Object { $_.Tags.Environment -eq "Production" } |
               Group-Object ResourceType |
               Select-Object Name, Count
       }
   }
   ```

3. **Minimize variable passing** (use `$using:` sparingly):
   ```powershell
   # ❌ BAD: Pass many individual variables
   InlineScript {
       Process-Data -Param1 $using:var1 -Param2 $using:var2 -Param3 $using:var3 -Param4 $using:var4
   }
   
   # ✅ GOOD: Pass hashtable/object
   $params = @{
       Param1 = $var1
       Param2 = $var2
       Param3 = $var3
       Param4 = $var4
   }
   
   InlineScript {
       Process-Data @using:params
   }
   ```

## Workflow Restrictions and Workarounds

### Restriction 1: No Return Keyword

**Workflow Issue**:
```powershell
workflow Bad-Return {
    if ($condition) {
        return "Early exit"  # ❌ Error: return not supported
    }
}
```

**Workaround**:
```powershell
workflow Good-Output {
    if ($condition) {
        Write-Output "Early exit"
        # Control flow continues, but use conditionals to skip logic
    }
}
```

### Restriction 2: Limited Pipeline Support

**Workflow Issue**:
```powershell
workflow Bad-Pipeline {
    # ❌ Doesn't work: $_ not available
    $vms = Get-AzVM | Where-Object { $_.PowerState -eq "Running" }
}
```

**Workaround**:
```powershell
workflow Good-Pipeline {
    # ✅ Works: Use InlineScript
    $vms = InlineScript {
        Get-AzVM | Where-Object { $_.PowerState -eq "Running" }
    }
}
```

### Restriction 3: Variable Scope

**Workflow Issue**:
```powershell
workflow Bad-Scope {
    $outer = "value"
    
    InlineScript {
        # ❌ $outer not accessible directly
        Write-Output $outer
    }
}
```

**Workaround**:
```powershell
workflow Good-Scope {
    $outer = "value"
    
    InlineScript {
        # ✅ Use $using: scope
        Write-Output $using:outer
    }
}
```

### Restriction 4: No Dynamic Functions

**Workflow Issue**:
```powershell
workflow Bad-Function {
    # ❌ Function definition not allowed in workflow
    function Get-Data {
        # ...
    }
}
```

**Workaround**:
```powershell
workflow Good-Function {
    # ✅ Define function in InlineScript
    InlineScript {
        function Get-Data {
            param([string]$Name)
            "Data for $Name"
        }
        
        Get-Data -Name "Server01"
    }
}
```

## Testing and Debugging Workflows

### Testing in PowerShell ISE

**PowerShell ISE Recommendation**:

Microsoft recommends using **PowerShell ISE** (Integrated Scripting Environment) for workflow development:

1. **Install PowerShell ISE**:
   ```powershell
   # Already included in Windows (PowerShell 5.1)
   # Or install PowerShell 7 with ISE:
   Install-Module -Name PowerShellISE -Force
   ```

2. **Create Workflow File**:
   - Open ISE
   - Create new file (`.ps1`)
   - Write workflow code
   - Save file

3. **Test Workflow Locally**:
   ```powershell
   # In ISE console
   . .\MyWorkflow.ps1  # Dot-source to load workflow
   
   # Execute workflow
   MyWorkflow -Parameter "value"
   ```

**Test Output**:
```powershell
workflow Test-Output {
    Write-Output "Output 1"
    Write-Verbose "Verbose message" -Verbose
    Write-Warning "Warning message"
    Write-Error "Error message"
    
    InlineScript {
        Write-Output "InlineScript output"
    }
}

# Run and see output
Test-Output
```

### Debugging with Breakpoints

**Set Breakpoints in ISE**:

1. Click in left margin of ISE to set breakpoint (red dot)
2. Run workflow (F5)
3. Execution pauses at breakpoint
4. Inspect variables in Variables pane
5. Step through code (F10 = step over, F11 = step into)

**Debug Example**:
```powershell
workflow Debug-Example {
    param([int]$Count)
    
    $result = 0
    
    for ($i = 0; $i -lt $Count; $i++) {
        $result += $i
        # Set breakpoint here to inspect $result each iteration
        Write-Output "Iteration $i : Result = $result"
    }
    
    Write-Output "Final result: $result"
}
```

### Common Debugging Patterns

**Add Verbose Output**:
```powershell
workflow Debug-With-Verbose {
    param([string[]]$Servers)
    
    Write-Verbose "Starting workflow for $($Servers.Count) servers" -Verbose
    
    ForEach -Parallel ($server in $Servers) {
        Write-Verbose "Processing server: $server" -Verbose
        
        InlineScript {
            Write-Verbose "  Connecting to $using:server" -Verbose
            # Processing logic
            Write-Verbose "  ✓ Completed $using:server" -Verbose
        }
    }
    
    Write-Verbose "Workflow completed" -Verbose
}
```

**Checkpoint for Debugging**:
```powershell
workflow Debug-With-Checkpoints {
    Write-Output "Stage 1: Starting"
    # Stage 1 logic
    Checkpoint-Workflow  # Resume from here if crash occurs
    
    Write-Output "Stage 2: Processing"
    # Stage 2 logic (if crash here, resume from Stage 2)
    Checkpoint-Workflow
    
    Write-Output "Stage 3: Finalizing"
    # Stage 3 logic
}
```

## Best Practices Summary

### 1. Workflow Structure

```powershell
workflow Well-Structured-Workflow {
    param(
        [Parameter(Mandatory=$true)]
        [string[]]$Targets
    )
    
    # 1. Authentication
    Connect-ToAzure
    Checkpoint-Workflow
    
    # 2. Get data
    $data = InlineScript {
        Get-AzResource | Where-Object { $_.Tags.Environment -eq "Production" }
    }
    Checkpoint-Workflow
    
    # 3. Process in parallel
    ForEach -Parallel -ThrottleLimit 10 ($item in $data) {
        InlineScript {
            Process-Item -Item $using:item
        }
    }
    Checkpoint-Workflow
    
    # 4. Finalize
    Write-Output "Processing complete: $($data.Count) items"
}
```

### 2. Checkpoint Strategy

- Checkpoint after expensive operations
- Checkpoint between logical stages
- Checkpoint periodically in long loops (every 50-100 iterations)
- Don't checkpoint in tight loops (overhead)

### 3. Parallel Execution

- Use `ForEach -Parallel` for independent operations
- Set appropriate `ThrottleLimit` (10-20 for most scenarios)
- Group related operations in `Parallel` blocks
- Use `Sequence` for sequential steps within parallel branches

### 4. InlineScript Usage

- Minimize InlineScript blocks (performance)
- Use InlineScript for complex PowerShell features
- Group multiple cmdlets in single InlineScript
- Use `$using:` for workflow variable access

## Next Steps

With workflow authoring skills mastered, proceed to **Unit 10: Explore Hybrid Management** to learn how workflows can execute on on-premises Hybrid Runbook Workers, extending Azure Automation to your datacenter.

**What You've Learned**:
- ✅ Workflow syntax and naming conventions
- ✅ Checkpoint-Workflow for resilience
- ✅ Parallel execution with ThrottleLimit
- ✅ InlineScript for advanced PowerShell
- ✅ Debugging and testing workflows

**Quick Reference Card**:
```powershell
workflow Quick-Reference {
    param([string[]]$Items)
    
    # Checkpoint
    Checkpoint-Workflow
    
    # Parallel (all items)
    ForEach -Parallel ($item in $Items) {
        InlineScript { Process $using:item }
    }
    
    # Parallel (throttled)
    ForEach -Parallel -ThrottleLimit 10 ($item in $Items) {
        InlineScript { Process $using:item }
    }
    
    # InlineScript with $using:
    $result = InlineScript {
        Get-Data -Name $using:item
    }
}
```

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/9-create-workflow
