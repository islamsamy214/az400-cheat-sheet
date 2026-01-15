# Examine Checkpoint and Parallel Processing

## Overview

This unit dives deep into two of the most powerful PowerShell Workflow features: **Checkpoints** and **Parallel Processing**. These capabilities transform ordinary automation scripts into resilient, high-performance workflows capable of handling enterprise-scale operations.

Checkpoints enable workflows to survive failures and resume from saved states, while parallel processing dramatically reduces execution time by running multiple operations concurrently. Together, they enable mission-critical automation that's both fast and reliable.

## Checkpoints Deep Dive

### What is a Checkpoint?

A **checkpoint** is a snapshot of the workflow's state saved to persistent storage. It captures:
- All variable values
- Current execution position
- Output generated so far
- Parallel execution state

**Persistence**: Checkpoints are saved to Azure Storage, surviving:
- System crashes
- Network failures
- Workflow suspensions
- Runbook worker failures

### Checkpoint-Workflow Activity

**Syntax**:
```powershell
Checkpoint-Workflow
```

**When Checkpoint is Taken**:
```powershell
workflow Checkpoint-Example {
    Write-Output "Step 1: Initialize"
    $count = 0
    
    Checkpoint-Workflow  # State saved here
    
    Write-Output "Step 2: Process"
    $count = 100
    
    Checkpoint-Workflow  # New state saved here
    
    Write-Output "Step 3: Complete"
}
```

**What Happens at Checkpoint**:
1. Workflow execution pauses briefly (1-3 seconds)
2. Current state serialized to JSON/XML
3. State uploaded to Azure Storage
4. Execution resumes

### Checkpoint Recovery Example

**Scenario: Long-Running Deployment**

```powershell
workflow Deploy-MultiTierApplication {
    param([string]$Environment)
    
    Write-Output "=== Stage 1: Deploy Database ==="
    InlineScript {
        # Deploy database (takes 10 minutes)
        New-AzSqlDatabase -ServerName "sql-server" -DatabaseName "appdb"
    }
    Write-Output "✓ Database deployed"
    Checkpoint-Workflow  # Checkpoint 1
    
    Write-Output "=== Stage 2: Deploy Web Tier ==="
    InlineScript {
        # Deploy web VMs (takes 15 minutes)
        New-AzVM -Name "web01" -ResourceGroupName "rg-app"
        New-AzVM -Name "web02" -ResourceGroupName "rg-app"
    }
    Write-Output "✓ Web tier deployed"
    Checkpoint-Workflow  # Checkpoint 2
    
    Write-Output "=== Stage 3: Deploy App Tier ==="
    InlineScript {
        # Deploy app VMs (takes 15 minutes)
        New-AzVM -Name "app01" -ResourceGroupName "rg-app"
        New-AzVM -Name "app02" -ResourceGroupName "rg-app"
    }
    Write-Output "✓ App tier deployed"
    Checkpoint-Workflow  # Checkpoint 3
    
    Write-Output "=== Stage 4: Configure Load Balancer ==="
    InlineScript {
        # Configure load balancer (takes 5 minutes)
        New-AzLoadBalancer -Name "lb-app" -ResourceGroupName "rg-app"
    }
    Write-Output "✓ Load balancer configured"
    Checkpoint-Workflow  # Checkpoint 4
    
    Write-Output "🎉 Deployment complete!"
}
```

**Failure Scenario**:
```
Execution Timeline:
00:00 - Stage 1 starts (Database deployment)
00:10 - Stage 1 completes → Checkpoint 1 saved
00:10 - Stage 2 starts (Web tier deployment)
00:18 - ❌ Network failure! Workflow crashes

Recovery (Automatic):
00:20 - Workflow resumes from Checkpoint 1
00:20 - Stage 1 skipped (already completed)
00:20 - Stage 2 resumes (Web tier deployment)
00:35 - Stage 2 completes → Checkpoint 2 saved
00:35 - Stage 3 starts (App tier deployment)
00:50 - Stage 3 completes → Checkpoint 3 saved
...
01:00 - ✓ Deployment completes

Result: Stage 1 not repeated, saved 10 minutes
```

### Activity2 Resumed After Activity1 Failure

**Detailed Recovery Example**:

```powershell
workflow Multi-Activity-Workflow {
    Write-Output "Activity 1: Download files"
    InlineScript {
        # Activity 1: Download 100 files (5 minutes)
        1..100 | ForEach-Object {
            Invoke-WebRequest -Uri "https://storage/file$_.zip" -OutFile "C:\Files\file$_.zip"
        }
    }
    Write-Output "✓ Activity 1 complete"
    Checkpoint-Workflow  # Checkpoint after Activity 1
    
    Write-Output "Activity 2: Process files"
    InlineScript {
        # Activity 2: Process files (10 minutes)
        Get-ChildItem "C:\Files\*.zip" | ForEach-Object {
            Expand-Archive $_.FullName -DestinationPath "C:\Processed"
        }
    }
    Write-Output "✓ Activity 2 complete"
    Checkpoint-Workflow  # Checkpoint after Activity 2
    
    Write-Output "Activity 3: Upload results"
    InlineScript {
        # Activity 3: Upload to cloud (3 minutes)
        Get-ChildItem "C:\Processed" | ForEach-Object {
            Copy-Item $_.FullName -Destination "\\cloud-storage\results"
        }
    }
    Write-Output "✓ Activity 3 complete"
}
```

**Failure During Activity 2**:
```
Timeline:
00:00 - Activity 1 starts (Download 100 files)
00:05 - Activity 1 completes → Checkpoint saved
00:05 - Activity 2 starts (Process files)
00:08 - ❌ Worker crashes during file processing

Recovery:
00:10 - Workflow resumes from last checkpoint
00:10 - Activity 1 SKIPPED (checkpoint shows already complete)
00:10 - Activity 2 RESUMES (processing starts again)
00:20 - Activity 2 completes → Checkpoint saved
00:20 - Activity 3 starts
00:23 - ✓ Workflow completes

Benefit: Saved 5 minutes by not re-downloading files
```

### Strategic Checkpoint Placement

**1. After Expensive Operations**:
```powershell
workflow Expensive-Operations {
    # Expensive: 30-minute VM deployment
    InlineScript { New-AzVM -Name "vm01" }
    Checkpoint-Workflow  # Don't repeat if failure occurs later
    
    # Expensive: 20-minute database restore
    InlineScript { Restore-AzSqlDatabase }
    Checkpoint-Workflow  # Don't repeat if failure occurs later
    
    # Cheap: 5-second tag update
    InlineScript { Set-AzResource -Tag @{Deployed=$true} }
    # No checkpoint needed (cheap to repeat)
}
```

**2. Between Logical Stages**:
```powershell
workflow Logical-Stages {
    # Stage 1: Infrastructure provisioning
    Deploy-Infrastructure
    Checkpoint-Workflow
    
    # Stage 2: Application deployment
    Deploy-Application
    Checkpoint-Workflow
    
    # Stage 3: Configuration
    Configure-Settings
    Checkpoint-Workflow
    
    # Stage 4: Testing
    Run-Tests
    Checkpoint-Workflow
}
```

**3. Periodic Checkpoints in Loops**:
```powershell
workflow Process-ManyItems {
    param([array]$Items)  # 1000 items
    
    $processed = 0
    foreach ($item in $Items) {
        Process-Item $item
        $processed++
        
        # Checkpoint every 50 items (20 checkpoints total)
        if ($processed % 50 -eq 0) {
            Checkpoint-Workflow
            Write-Output "Checkpoint: $processed of $($Items.Count) processed"
        }
    }
}
```

### Checkpoint Performance Impact

**Overhead**:
- Serialization time: 0.5-2 seconds
- Upload time: 0.5-1 second
- **Total per checkpoint**: 1-3 seconds

**Optimization Guidelines**:

```powershell
# ❌ BAD: Checkpoint in tight loop (huge overhead)
workflow Bad-Checkpointing {
    foreach ($i in 1..10000) {
        Process-Item $i
        Checkpoint-Workflow  # 10,000 checkpoints × 2 seconds = 5.5 hours overhead!
    }
}

# ✅ GOOD: Checkpoint periodically
workflow Good-Checkpointing {
    $count = 0
    foreach ($i in 1..10000) {
        Process-Item $i
        $count++
        
        if ($count % 100 -eq 0) {
            Checkpoint-Workflow  # 100 checkpoints × 2 seconds = 3.3 minutes overhead
        }
    }
}

# ✅ BETTER: Checkpoint after significant milestones
workflow Better-Checkpointing {
    $count = 0
    foreach ($i in 1..10000) {
        Process-Item $i
        $count++
        
        if ($count % 500 -eq 0) {
            Checkpoint-Workflow  # 20 checkpoints × 2 seconds = 40 seconds overhead
        }
    }
}
```

**Cost-Benefit Analysis**:
```
Scenario: 1000-item processing, each item takes 10 seconds (2.7 hours total)

Checkpoint Every 10 Items:
- 100 checkpoints
- Overhead: 100 × 2s = 200s (3.3 minutes)
- Failure at item 505: Resume from 500, lose 50s work
- Trade-off: Low data loss, low overhead ✅

Checkpoint Every 100 Items:
- 10 checkpoints
- Overhead: 10 × 2s = 20s
- Failure at item 505: Resume from 500, lose 50s work
- Trade-off: Low data loss, minimal overhead ✅✅

Checkpoint Every 500 Items:
- 2 checkpoints
- Overhead: 2 × 2s = 4s
- Failure at item 505: Resume from 500, lose 50s work
- Trade-off: Low data loss, negligible overhead ✅✅✅

No Checkpoints:
- 0 overhead
- Failure at item 505: Restart from item 1, lose 5050s work (1.4 hours!)
- Trade-off: No overhead, catastrophic data loss ❌
```

## Parallel Processing Deep Dive

### Basic Parallel Example

**Sequential vs Parallel**:

```powershell
# SEQUENTIAL: Start 10 VMs one by one
workflow Start-VMs-Sequential {
    $vmNames = @("vm01", "vm02", "vm03", "vm04", "vm05", "vm06", "vm07", "vm08", "vm09", "vm10")
    
    foreach ($vmName in $vmNames) {
        InlineScript {
            Start-AzVM -Name $using:vmName -ResourceGroupName "rg-vms"
        }
    }
}
# Time: 10 VMs × 30 seconds each = 5 minutes

# PARALLEL: Start all 10 VMs simultaneously
workflow Start-VMs-Parallel {
    $vmNames = @("vm01", "vm02", "vm03", "vm04", "vm05", "vm06", "vm07", "vm08", "vm09", "vm10")
    
    ForEach -Parallel ($vmName in $vmNames) {
        InlineScript {
            Start-AzVM -Name $using:vmName -ResourceGroupName "rg-vms"
        }
    }
}
# Time: 30 seconds (all start simultaneously)
# Speedup: 10x faster! ✅
```

### ForEach -Parallel Construct

**Syntax**:
```powershell
ForEach -Parallel [-ThrottleLimit <int>] ($item in $collection) {
    # Parallel activities
}
```

**How It Works**:
```
Input: Array of 100 items
Without -ThrottleLimit: 100 concurrent executions (default)
With -ThrottleLimit 10: 10 concurrent executions at a time

Execution Flow:
    Items 1-10: Execute concurrently
    Item 11: Starts when item 1 completes
    Item 12: Starts when item 2 completes
    ...
    Item 100: Starts when item 90 completes
```

**Complete Example**:
```powershell
workflow Process-Servers-Parallel {
    $servers = 1..50 | ForEach-Object { "server$($_.ToString('D2'))" }
    
    ForEach -Parallel -ThrottleLimit 10 ($server in $servers) {
        Write-Output "Processing: $server (Worker: $(Get-Date))"
        
        InlineScript {
            # Simulate server processing (10 seconds each)
            Start-Sleep -Seconds 10
            
            # Actual work
            Invoke-Command -ComputerName $using:server -ScriptBlock {
                Get-Service | Where-Object Status -eq "Stopped"
            }
        }
        
        Write-Output "✓ Completed: $server"
    }
}

# Sequential: 50 servers × 10 seconds = 500 seconds (8.3 minutes)
# Parallel (no throttle): 10 seconds (all at once, but may overload)
# Parallel (throttle 10): 50 seconds (5 batches of 10)
# Speedup: 10x faster than sequential ✅
```

### ThrottleLimit Tuning

**Purpose**: Control concurrency to prevent resource exhaustion and rate limiting.

**Tuning by Scenario**:

**Example 1: Azure VM Operations**
```powershell
workflow Start-AzureVMs {
    param([string[]]$VMNames)  # 200 VMs
    
    # ARM throttling: 800 requests per 5 minutes
    # Each Start-AzVM: ~3 API calls
    # Safe rate: ~250 VMs per 5 minutes
    # Therefore: Throttle to 20 concurrent operations
    
    ForEach -Parallel -ThrottleLimit 20 ($vmName in $VMNames) {
        InlineScript {
            Start-AzVM -Name $using:vmName -ResourceGroupName "rg-vms" -NoWait
        }
    }
}
```

**Example 2: Database Queries**
```powershell
workflow Query-Databases {
    param([string[]]$Databases)  # 100 databases
    
    # SQL Server connection limit: 32,767 (but practical limit ~100-200)
    # Database can handle 20 concurrent queries comfortably
    
    ForEach -Parallel -ThrottleLimit 20 ($db in $Databases) {
        InlineScript {
            Invoke-Sqlcmd -Query "SELECT COUNT(*) FROM Orders" -Database $using:db
        }
    }
}
```

**Example 3: File Operations**
```powershell
workflow Copy-ManyFiles {
    param([string[]]$Files)  # 1000 files
    
    # Network bandwidth: 1 Gbps
    # Each file: 100 MB
    # Network can handle ~50 concurrent transfers
    
    ForEach -Parallel -ThrottleLimit 50 ($file in $Files) {
        InlineScript {
            Copy-Item -Path $using:file -Destination "\\backup-server\files"
        }
    }
}
```

**Dynamic ThrottleLimit**:
```powershell
workflow Dynamic-Throttle {
    param([string[]]$Items)
    
    # Adjust throttle based on item count
    $throttle = switch ($Items.Count) {
        {$_ -le 10}    { 5 }   # Small: Low concurrency
        {$_ -le 100}   { 10 }  # Medium: Moderate concurrency
        {$_ -le 1000}  { 20 }  # Large: High concurrency
        default        { 50 }  # Very large: Max concurrency
    }
    
    Write-Output "Processing $($Items.Count) items with throttle limit: $throttle"
    
    ForEach -Parallel -ThrottleLimit $throttle ($item in $Items) {
        InlineScript {
            Process-Item -Item $using:item
        }
    }
}
```

### Real-World Example: File Copy Workflow

**Scenario**: Copy 1000 files from source to destination with parallel processing and checkpoints.

```powershell
workflow Copy-LargeFileSet {
    param(
        [string]$SourcePath = "\\source-server\data",
        [string]$DestinationPath = "\\backup-server\data",
        [int]$ThrottleLimit = 50
    )
    
    Write-Output "=== Starting file copy workflow ==="
    Write-Output "Source: $SourcePath"
    Write-Output "Destination: $DestinationPath"
    Write-Output "Throttle Limit: $ThrottleLimit concurrent copies"
    
    # Get all files to copy
    $files = InlineScript {
        Get-ChildItem -Path $using:SourcePath -File -Recurse |
            Select-Object FullName, Name, Length
    }
    
    $totalFiles = $files.Count
    $totalSizeGB = [Math]::Round(($files | Measure-Object -Property Length -Sum).Sum / 1GB, 2)
    
    Write-Output "Found $totalFiles files (Total: $totalSizeGB GB)"
    Checkpoint-Workflow  # Checkpoint 1: File list retrieved
    
    # Copy files in parallel with throttling
    $copiedCount = 0
    
    ForEach -Parallel -ThrottleLimit $ThrottleLimit ($file in $files) {
        $sourceFile = $file.FullName
        $relativePath = $sourceFile.Replace($SourcePath, "")
        $destFile = Join-Path $DestinationPath $relativePath
        
        InlineScript {
            # Create destination directory if needed
            $destDir = Split-Path $using:destFile -Parent
            if (-not (Test-Path $destDir)) {
                New-Item -Path $destDir -ItemType Directory -Force | Out-Null
            }
            
            # Copy file with retry logic
            $maxRetries = 3
            $retryCount = 0
            $copied = $false
            
            while (-not $copied -and $retryCount -lt $maxRetries) {
                try {
                    Copy-Item -Path $using:sourceFile -Destination $using:destFile -Force
                    $copied = $true
                    Write-Output "✓ Copied: $($using:file.Name) ($([Math]::Round($using:file.Length / 1MB, 2)) MB)"
                }
                catch {
                    $retryCount++
                    if ($retryCount -lt $maxRetries) {
                        Write-Output "⚠ Retry $retryCount for $($using:file.Name): $_"
                        Start-Sleep -Seconds (2 * $retryCount)  # Exponential backoff
                    }
                    else {
                        Write-Error "✗ Failed to copy $($using:file.Name) after $maxRetries attempts: $_"
                    }
                }
            }
        }
        
        # Increment counter and checkpoint periodically
        $copiedCount++
        if ($copiedCount % 50 -eq 0) {
            Checkpoint-Workflow
            $percentComplete = [Math]::Round(($copiedCount / $totalFiles) * 100, 1)
            Write-Output "Progress: $copiedCount of $totalFiles files copied ($percentComplete%)"
        }
    }
    
    # Final checkpoint
    Checkpoint-Workflow
    
    Write-Output "=== Copy workflow completed ==="
    Write-Output "Total files copied: $totalFiles"
    Write-Output "Total size: $totalSizeGB GB"
    
    # Verify copy
    $verifyResults = InlineScript {
        $sourceCou nt = (Get-ChildItem -Path $using:SourcePath -File -Recurse).Count
        $destCount = (Get-ChildItem -Path $using:DestinationPath -File -Recurse).Count
        
        [PSCustomObject]@{
            SourceFiles = $sourceCount
            DestinationFiles = $destCount
            Match = ($sourceCount -eq $destCount)
        }
    }
    
    if ($verifyResults.Match) {
        Write-Output "✓ Verification passed: Source and destination file counts match"
    }
    else {
        Write-Warning "⚠ Verification warning: File counts don't match"
        Write-Output "Source: $($verifyResults.SourceFiles), Destination: $($verifyResults.DestinationFiles)"
    }
}
```

**Performance Analysis**:
```
Scenario: 1000 files, 100 MB each (100 GB total), 10 Mbps per transfer

Sequential Copy:
- Time per file: 100 MB / 10 Mbps = 80 seconds
- Total time: 1000 files × 80 seconds = 80,000 seconds (22.2 hours)

Parallel Copy (ThrottleLimit 50):
- 50 concurrent transfers
- Time for 50 files: 80 seconds
- Total batches: 1000 / 50 = 20 batches
- Total time: 20 batches × 80 seconds = 1,600 seconds (26.7 minutes)
- Speedup: 50x faster! ✅

Checkpoint Benefits:
- Checkpoint every 50 files
- If failure at file 523: Resume from file 500
- Time saved: 500 files = 13.3 hours of work preserved
```

### Parallel Block vs ForEach -Parallel

**Parallel Block**: Execute different activities simultaneously
```powershell
workflow Parallel-Block-Example {
    Parallel {
        # All three run concurrently
        InlineScript { Deploy-WebTier }
        InlineScript { Deploy-AppTier }
        InlineScript { Deploy-DatabaseTier }
    }
    # Continues when all three complete
}
```

**ForEach -Parallel**: Execute same activity for multiple items
```powershell
workflow ForEach-Parallel-Example {
    $servers = @("server01", "server02", "server03")
    
    ForEach -Parallel ($server in $servers) {
        # Same activity for each server, all run concurrently
        InlineScript { Configure-Server -Name $using:server }
    }
}
```

**Combined Usage**:
```powershell
workflow Combined-Parallel {
    Parallel {
        # Branch 1: Process databases in parallel
        Sequence {
            $databases = @("db01", "db02", "db03")
            ForEach -Parallel ($db in $databases) {
                InlineScript { Backup-Database -Name $using:db }
            }
        }
        
        # Branch 2: Process file servers in parallel (runs alongside Branch 1)
        Sequence {
            $fileServers = @("fs01", "fs02", "fs03")
            ForEach -Parallel ($fs in $fileServers) {
                InlineScript { Cleanup-FileServer -Name $using:fs }
            }
        }
        
        # Branch 3: Process VMs in parallel (runs alongside Branches 1 and 2)
        Sequence {
            $vms = @("vm01", "vm02", "vm03")
            ForEach -Parallel ($vm in $vms) {
                InlineScript { Patch-VM -Name $using:vm }
            }
        }
    }
    # All branches must complete before continuing
    Write-Output "All parallel branches completed"
}
```

## Best Practices: Combining Checkpoints and Parallel Processing

### Pattern 1: Checkpoint After Parallel Batches

```powershell
workflow Batch-Processing {
    $allItems = 1..1000
    $batchSize = 100
    
    for ($i = 0; $i -lt $allItems.Count; $i += $batchSize) {
        $batch = $allItems[$i..([Math]::Min($i + $batchSize - 1, $allItems.Count - 1))]
        
        Write-Output "Processing batch: $($i + 1) to $($i + $batch.Count)"
        
        # Process batch in parallel
        ForEach -Parallel -ThrottleLimit 20 ($item in $batch) {
            InlineScript { Process-Item -Item $using:item }
        }
        
        # Checkpoint after each batch
        Checkpoint-Workflow
    }
}
```

### Pattern 2: Nested Parallel with Strategic Checkpoints

```powershell
workflow Nested-Parallel {
    $regions = @("East US", "West US", "Central US")
    
    # Process each region (outer parallelism)
    ForEach -Parallel ($region in $regions) {
        Write-Output "Processing region: $region"
        
        $resourceGroups = InlineScript {
            Get-AzResourceGroup -Location $using:region
        }
        
        # Process resource groups within region (nested parallelism)
        ForEach -Parallel -ThrottleLimit 10 ($rg in $resourceGroups) {
            InlineScript {
                $resources = Get-AzResource -ResourceGroupName $using:rg.ResourceGroupName
                foreach ($resource in $resources) {
                    # Process each resource
                    Set-AzResource -ResourceId $resource.ResourceId -Tag @{Region=$using:region} -Force
                }
            }
        }
        
        # Checkpoint after each region
        Checkpoint-Workflow
    }
}
```

### Pattern 3: Parallel with Progress Tracking

```powershell
workflow Progress-Tracking {
    $items = 1..500
    $completed = 0
    $mutex = New-Object System.Threading.Mutex($false, "ProgressMutex")
    
    ForEach -Parallel -ThrottleLimit 25 ($item in $items) {
        InlineScript {
            Process-Item -Item $using:item
        }
        
        # Thread-safe counter increment
        $workflow:completed++
        
        # Checkpoint every 25 items
        if ($workflow:completed % 25 -eq 0) {
            Checkpoint-Workflow
            $percentComplete = [Math]::Round(($workflow:completed / 500) * 100, 1)
            Write-Output "Progress: $($workflow:completed)/500 ($percentComplete%)"
        }
    }
}
```

## Performance Optimization Summary

### Checkpoint Optimization

| Checkpoint Frequency | Overhead | Data Loss Risk | Recommendation |
|---------------------|----------|----------------|----------------|
| Every iteration | Very High (❌) | Minimal | Avoid |
| Every 10 iterations | High | Very Low | Small datasets only |
| Every 50 iterations | Medium | Low | ✅ Balanced |
| Every 100 iterations | Low | Medium | Large datasets |
| Every 500 iterations | Very Low | Medium-High | Very large datasets |
| After major stages | Minimal (✅) | High | ✅ Best for staged workflows |

### Parallel Processing Optimization

| Concurrency Level | Use Case | ThrottleLimit |
|------------------|----------|---------------|
| Low (5-10) | Database operations, API calls | 5-10 |
| Medium (10-20) | Azure resource operations | 10-20 |
| High (20-50) | File operations, network transfers | 20-50 |
| Very High (50+) | CPU-bound, independent operations | 50-100 |

## Next Steps

You've now mastered the advanced workflow features that enable enterprise-scale automation. Proceed to **Unit 12: Module Assessment** to test your comprehensive understanding of Azure Automation concepts, from basic runbooks to advanced hybrid workflows.

**Key Takeaways**:
- ✅ Checkpoints enable resilience by saving workflow state
- ✅ Strategic checkpoint placement balances overhead vs data loss
- ✅ Parallel processing dramatically reduces execution time
- ✅ ThrottleLimit prevents resource exhaustion
- ✅ Combine checkpoints and parallel processing for optimal performance

**Quick Reference**:
```powershell
# Checkpoint every 50 items in parallel loop
$count = 0
ForEach -Parallel -ThrottleLimit 20 ($item in $items) {
    InlineScript { Process $using:item }
    $workflow:count++
    if ($workflow:count % 50 -eq 0) {
        Checkpoint-Workflow
    }
}
```

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/11-examine-checkpoint-parallel-processing
