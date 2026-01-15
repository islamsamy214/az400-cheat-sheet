# Module Assessment

Test your knowledge of Azure Automation with DevOps concepts covered in this module. This assessment contains scenario-based questions that evaluate your understanding of automation accounts, runbook types, shared resources, webhooks, source control, PowerShell workflows, hybrid workers, and advanced features.

---

## Question 1: Automation Account Strategy

**Scenario**: Your organization has development, staging, and production environments. You're designing an Azure Automation strategy to manage VM lifecycle operations across these environments. Which approach provides the best balance of isolation, cost, and management overhead?

**A)** Create a single Automation account and use tags to separate environments  
**B)** Create separate Automation accounts for each environment (dev, staging, prod)  
**C)** Create one Automation account per subscription  
**D)** Use Azure DevOps pipelines instead of Automation accounts

<details>
<summary><strong>✅ Correct Answer</strong></summary>

**B) Create separate Automation accounts for each environment (dev, staging, prod)**

**Explanation**:

Separate Automation accounts per environment is the **best practice** for several reasons:

1. **Isolation**: Dev changes can't accidentally affect production
2. **Security**: Different RBAC permissions per environment (developers access dev, only ops access prod)
3. **Run As Accounts**: Separate service principals with environment-specific permissions
4. **Compliance**: Audit trails clearly separated by environment
5. **Cost Tracking**: Easy to track automation costs per environment
6. **Disaster Recovery**: Prod Automation account failure doesn't impact dev/staging

**Why other options are incorrect**:

**A) Single account with tags**: 
- ❌ Security risk: All runbooks share same Run As account
- ❌ No isolation: Error in dev runbook could affect prod
- ❌ Complex RBAC: Hard to grant environment-specific permissions

**C) One per subscription**:
- ❌ Overkill: Most orgs have 1-5 subscriptions, not aligned with environments
- ❌ Inflexible: What if dev and staging share a subscription?

**D) Azure DevOps pipelines**:
- ❌ Different use case: Pipelines for CI/CD, Automation for operational tasks
- ❌ Missing features: No scheduled runbooks, no Hybrid Workers, no State Configuration

**Implementation Example**:
```powershell
# Create separate Automation accounts
New-AzAutomationAccount -Name "automation-dev" -ResourceGroupName "rg-automation-dev" -Location "East US"
New-AzAutomationAccount -Name "automation-staging" -ResourceGroupName "rg-automation-staging" -Location "East US"
New-AzAutomationAccount -Name "automation-prod" -ResourceGroupName "rg-automation-prod" -Location "East US"

# Different Run As accounts with environment-specific permissions
# Dev: Contributor on dev resource groups
# Staging: Contributor on staging resource groups
# Prod: Limited permissions, requires approval for changes
```

**Key Takeaway**: Environment separation at the Automation account level provides security, isolation, and manageability.

</details>

---

## Question 2: Runbook Type Selection

**Scenario**: You need to create a runbook that backs up 200 SQL databases across 50 servers. The backup process typically takes 4 hours total. If a server fails during backup, the runbook should resume from the last successful server backup instead of starting over. Network interruptions are common. Which runbook type is most appropriate?

**A)** PowerShell Runbook  
**B)** PowerShell Workflow Runbook  
**C)** Python Runbook  
**D)** Graphical Runbook

<details>
<summary><strong>✅ Correct Answer</strong></summary>

**B) PowerShell Workflow Runbook**

**Explanation**:

**PowerShell Workflow** is the clear choice because:

1. **Checkpoints**: Resume from last server backup after failures
2. **Long-running**: 4-hour execution supported (regular PowerShell limited to 3 hours fair share)
3. **Parallel processing**: Backup 50 servers concurrently
4. **Network resilience**: Automatic retry on transient failures
5. **State persistence**: Survives runbook worker crashes

**Example Implementation**:
```powershell
workflow Backup-SqlDatabases {
    param([string[]]$DatabaseServers)  # 50 servers
    
    ForEach -Parallel -ThrottleLimit 10 ($server in $DatabaseServers) {
        $databases = InlineScript {
            Get-SqlDatabase -ServerInstance $using:server
        }
        
        ForEach ($db in $databases) {
            InlineScript {
                Backup-SqlDatabase -ServerInstance $using:server -Database $using:db.Name
            }
        }
        
        # Checkpoint after each server
        Checkpoint-Workflow  # Resume from here if failure occurs
        Write-Output "✓ Completed server: $server"
    }
}

# Execution:
# 0:00 - Start backup (all 50 servers in parallel)
# 2:30 - Server 25 fails (network issue)
# 2:35 - Workflow resumes from Checkpoint (server 25 repeats, others not affected)
# 4:00 - All 200 databases backed up
```

**Why other options are incorrect**:

**A) PowerShell Runbook**:
- ❌ No checkpoints: Crash requires full restart
- ❌ 3-hour fair share limit: May not complete
- ❌ No native parallel processing
- ❌ Manual retry logic required

**C) Python Runbook**:
- ❌ No checkpoints: Crash requires restart
- ❌ No parallel execution within runbook
- ❌ Limited SQL Server management libraries compared to PowerShell

**D) Graphical Runbook**:
- ❌ No checkpoints
- ❌ Complex to manage 200 database backups in visual designer
- ❌ Limited parallel capabilities

**Performance Comparison**:
```
Sequential (PowerShell Runbook):
- 50 servers × 200 databases / 50 = 4 databases per server
- 4 databases × 2 minutes each = 8 minutes per server
- 50 servers × 8 minutes = 400 minutes (6.7 hours)
- Failure at server 25: Restart from server 1 (3.3 hours lost)

Parallel Workflow (PowerShell Workflow):
- 50 servers processed concurrently (ThrottleLimit 10: 5 batches)
- 10 servers × 8 minutes per batch = 80 minutes per batch
- 5 batches × 80 minutes = 400 minutes (6.7 hours)
- BUT: Parallel within servers too!
- Actual time: ~2-3 hours (depends on SQL Server load)
- Failure at server 25: Resume from server 25 checkpoint (only 8 minutes lost)
```

**Key Takeaway**: PowerShell Workflow is essential for long-running, resilient automation with parallel processing.

</details>

---

## Question 3: Shared Resources Usage

**Scenario**: Your runbook needs to connect to an on-premises SQL Server using a domain admin account. The password must be rotated every 90 days per security policy. The runbook runs 50 times per day. Which shared resource type should store the credentials?

**A)** Variable (String) with encrypted password  
**B)** Credential asset  
**C)** Connection asset with custom fields  
**D)** Certificate for authentication

<details>
<summary><strong>✅ Correct Answer</strong></summary>

**B) Credential asset**

**Explanation**:

**Credential asset** is the correct choice because:

1. **Native encryption**: Automatically encrypted at rest
2. **Secure retrieval**: `Get-AutomationPSCredential` returns PSCredential object
3. **Password rotation**: Update once in Azure Portal, all runbooks use new password
4. **No code changes**: Runbooks reference by name, not hardcoded password
5. **Audit trail**: Changes logged in Azure Activity Log

**Implementation**:
```powershell
# Create credential in Azure Portal:
# Automation Account → Credentials → + Add credential
# Name: SqlAdminCred
# Username: CONTOSO\sqladmin
# Password: ••••••••

# Use in runbook (no password in code!)
workflow Backup-SqlDatabase {
    param([string]$DatabaseServer, [string]$DatabaseName)
    
    InlineScript {
        # Retrieve credential (automatically decrypted)
        $sqlCred = Get-AutomationPSCredential -Name "SqlAdminCred"
        
        # Use credential for SQL connection
        Backup-SqlDatabase -ServerInstance $using:DatabaseServer `
            -Database $using:DatabaseName `
            -Credential $sqlCred `
            -BackupFile "\\backup\$using:DatabaseName.bak"
    }
}

# Password rotation (every 90 days):
# 1. Update password in Active Directory
# 2. Update "SqlAdminCred" credential in Azure Portal (1 minute)
# 3. All 50 daily runbook executions use new password automatically ✅
# 4. No code deployment required ✅
```

**Why other options are incorrect**:

**A) Variable (String) with encrypted password**:
- ❌ Manual encryption/decryption required in runbook code
- ❌ Encryption key management complexity
- ❌ No built-in rotation mechanism
- ❌ Error-prone (easy to forget decryption step)

**Example (problematic)**:
```powershell
# ❌ BAD: Manual encryption
$encryptedPassword = Get-AutomationVariable -Name "SqlPasswordEncrypted"
$password = ConvertTo-SecureString $encryptedPassword -AsPlainText -Force
$username = "CONTOSO\sqladmin"
$cred = New-Object PSCredential($username, $password)
# Problem: Encryption key management, more code, error-prone
```

**C) Connection asset with custom fields**:
- ❌ Designed for Azure connections (Run As), not custom credentials
- ❌ No built-in PSCredential creation
- ❌ Overkill for simple username/password

**D) Certificate for authentication**:
- ❌ SQL Server may not support certificate authentication (depends on config)
- ❌ More complex setup (certificate installation on SQL Server)
- ❌ Certificate rotation more complex than password rotation

**Password Rotation Workflow**:
```powershell
# Automated credential rotation runbook
workflow Rotate-SqlCredential {
    param(
        [string]$CredentialName = "SqlAdminCred",
        [string]$KeyVaultName = "kv-automation-secrets",
        [string]$SecretName = "sql-admin-password"
    )
    
    InlineScript {
        # Generate new complex password
        $newPassword = -join ((48..57) + (65..90) + (97..122) | Get-Random -Count 16 | % {[char]$_})
        $securePassword = ConvertTo-SecureString $newPassword -AsPlainText -Force
        
        # Get current username
        $currentCred = Get-AutomationPSCredential -Name $using:CredentialName
        $username = $currentCred.UserName
        
        # Update Active Directory password (simplified)
        Set-ADAccountPassword -Identity $username -NewPassword $securePassword -Reset
        
        # Update Azure Automation Credential
        $newCred = New-Object PSCredential($username, $securePassword)
        Set-AzAutomationCredential -ResourceGroupName "rg-automation" `
            -AutomationAccountName "automation-prod" `
            -Name $using:CredentialName `
            -Value $newCred
        
        # Backup password to Key Vault
        Set-AzKeyVaultSecret -VaultName $using:KeyVaultName `
            -Name $using:SecretName `
            -SecretValue $securePassword
        
        Write-Output "✓ Credential rotated successfully"
    }
}

# Schedule to run every 90 days
```

**Key Takeaway**: Credential assets provide secure, centralized, easy-to-rotate credential management for runbooks.

</details>

---

## Question 4: Webhook Integration

**Scenario**: You need to trigger a runbook from an Azure Monitor alert when VM CPU exceeds 90%. The alert should pass the VM name and resource group to the runbook. The webhook must be secure and expire after 1 year. What is the correct webhook configuration?

**A)** Use HTTP trigger with URL parameter for VM name  
**B)** Create webhook with 1-year expiry, store URL in Key Vault, use alert action group with webhook action  
**C)** Use Logic App to receive alert and call runbook  
**D)** Store webhook URL in plaintext in alert configuration

<details>
<summary><strong>✅ Correct Answer</strong></summary>

**B) Create webhook with 1-year expiry, store URL in Key Vault, use alert action group with webhook action**

**Explanation**:

This approach follows **security best practices**:

1. **Webhook creation** with appropriate expiry (1 year = balance between security and maintenance)
2. **Secure storage** in Key Vault (webhook URL contains secret token)
3. **Alert action group** for reusable webhook integration
4. **Parameter passing** via WebhookData

**Complete Implementation**:

**Step 1: Create runbook**:
```powershell
workflow Remediate-HighCPU {
    param([object]$WebhookData)
    
    if ($WebhookData) {
        # Parse webhook data
        $alertData = ConvertFrom-Json $WebhookData.RequestBody
        $vmName = $alertData.data.context.resourceName
        $resourceGroup = $alertData.data.context.resourceGroupName
        $cpuPercent = $alertData.data.context.metricValue
        
        Write-Output "Alert: $vmName CPU at $cpuPercent%"
        Write-Output "Resource Group: $resourceGroup"
        
        # Remediation logic
        InlineScript {
            # Option 1: Restart VM
            Restart-AzVM -ResourceGroupName $using:resourceGroup -Name $using:vmName -NoWait
            
            # Option 2: Scale up VM (if enabled)
            # $vm = Get-AzVM -ResourceGroupName $using:resourceGroup -Name $using:vmName
            # $vm.HardwareProfile.VmSize = "Standard_D4s_v3"
            # Update-AzVM -VM $vm -ResourceGroupName $using:resourceGroup
            
            Write-Output "✓ Remediation complete"
        }
    }
    else {
        Write-Error "Webhook data is null. Runbook must be triggered via webhook."
    }
}
```

**Step 2: Create webhook**:
```powershell
# Create webhook with 1-year expiry
$webhook = New-AzAutomationWebhook `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod" `
    -Name "webhook-remediate-highcpu" `
    -RunbookName "Remediate-HighCPU" `
    -IsEnabled $true `
    -ExpiryTime (Get-Date).AddYears(1) `
    -Force

# Webhook URL (shown only once!)
$webhookUrl = $webhook.WebhookURI

Write-Warning "⚠️ Copy this URL now (shown only once): $webhookUrl"

# Store URL in Key Vault immediately
$secureWebhookUrl = ConvertTo-SecureString $webhookUrl -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName "kv-automation-secrets" `
    -Name "webhook-remediate-highcpu" `
    -SecretValue $secureWebhookUrl

Write-Output "✓ Webhook URL stored in Key Vault"
```

**Step 3: Create alert action group**:
```powershell
# Create action group with webhook
$webhookUrlFromKeyVault = Get-AzKeyVaultSecret -VaultName "kv-automation-secrets" -Name "webhook-remediate-highcpu" -AsPlainText

$webhookAction = New-AzActionGroupReceiver `
    -Name "webhook-remediate" `
    -WebhookReceiver `
    -ServiceUri $webhookUrlFromKeyVault

New-AzActionGroup `
    -ResourceGroupName "rg-monitoring" `
    -Name "ag-vm-high-cpu" `
    -ShortName "HighCPU" `
    -Receiver $webhookAction
```

**Step 4: Create metric alert**:
```powershell
# Create alert rule for CPU > 90%
$condition = New-AzMetricAlertRuleV2Criteria `
    -MetricName "Percentage CPU" `
    -MetricNamespace "Microsoft.Compute/virtualMachines" `
    -TimeAggregation Average `
    -Operator GreaterThan `
    -Threshold 90

Add-AzMetricAlertRuleV2 `
    -Name "alert-vm-high-cpu" `
    -ResourceGroupName "rg-vms" `
    -WindowSize (New-TimeSpan -Minutes 5) `
    -Frequency (New-TimeSpan -Minutes 1) `
    -TargetResourceScope "/subscriptions/SUBSCRIPTION_ID/resourceGroups/rg-vms" `
    -Condition $condition `
    -ActionGroupId "/subscriptions/SUBSCRIPTION_ID/resourceGroups/rg-monitoring/providers/microsoft.insights/actionGroups/ag-vm-high-cpu" `
    -Severity 2
```

**Why other options are incorrect**:

**A) HTTP trigger with URL parameter**:
- ❌ Azure Automation webhooks use POST body, not URL parameters
- ❌ Less secure (parameters visible in logs)
- ❌ No authentication token in URL

**C) Logic App to receive alert and call runbook**:
- ❌ Additional cost (Logic App + Automation)
- ❌ More complexity (two resources instead of one)
- ❌ Higher latency (alert → Logic App → Automation vs alert → Automation)
- ✅ Use Logic App only if complex workflow logic needed

**D) Store webhook URL in plaintext**:
- ❌ **Major security risk**: Webhook URL contains secret token
- ❌ Anyone with URL can trigger runbook
- ❌ Violates security best practices

**Security Risk Example**:
```
Plaintext webhook URL in alert config:
https://eastus.webhook.automation.azure.com/webhooks?token=abc123secret

If config exported/logged:
- Token exposed: abc123secret
- Attacker can trigger runbook repeatedly
- Unauthorized VM restarts
- Denial of service
```

**Webhook Expiry Management**:
```powershell
# Monitor webhook expiry
workflow Monitor-WebhookExpiry {
    $webhooks = Get-AzAutomationWebhook -ResourceGroupName "rg-automation" `
        -AutomationAccountName "automation-prod"
    
    $expiringWebhooks = $webhooks | Where-Object {
        $_.ExpiryTime -lt (Get-Date).AddDays(30)  # Expiring in 30 days
    }
    
    if ($expiringWebhooks) {
        foreach ($webhook in $expiringWebhooks) {
            $daysUntilExpiry = ($webhook.ExpiryTime - (Get-Date)).Days
            
            Send-MailMessage -To "ops@contoso.com" `
                -Subject "Webhook Expiry Warning" `
                -Body "Webhook '$($webhook.Name)' expires in $daysUntilExpiry days"
        }
    }
}

# Schedule monthly
```

**Key Takeaway**: Webhooks enable event-driven automation but require secure URL management and expiry monitoring.

</details>

---

## Question 5: Source Control Integration Strategy

**Scenario**: Your team has 20 developers working on 50+ runbooks. Changes need code review before production deployment. You want to test runbooks in dev Automation account before promoting to production. Which source control strategy is most appropriate?

**A)** Single main branch, all developers commit directly, auto-sync to prod Automation account  
**B)** Main/staging/develop branches, feature branches for changes, PR workflow, separate Automation accounts per branch  
**C)** No source control, edit runbooks in Azure Portal  
**D)** Store runbooks in SharePoint with version history

<details>
<summary><strong>✅ Correct Answer</strong></summary>

**B) Main/staging/develop branches, feature branches for changes, PR workflow, separate Automation accounts per branch**

**Explanation**:

This **GitFlow-style branching strategy** provides:

1. **Code review**: Pull requests required for all changes
2. **Testing**: Each environment (dev/staging/prod) has own Automation account
3. **Rollback**: Git history enables easy rollback
4. **Collaboration**: 20 developers work on feature branches without conflicts
5. **CI/CD**: Automated testing in pipelines before merge

**Implementation**:

**Repository Structure**:
```
azure-automation-runbooks/
├── runbooks/
│   ├── vm-management/
│   │   ├── Start-VMs.ps1
│   │   ├── Stop-VMs.ps1
│   │   └── Restart-VMs.ps1
│   ├── backup/
│   │   ├── Backup-Databases.ps1
│   │   └── Backup-Files.ps1
│   └── monitoring/
│       └── Check-Health.ps1
├── tests/
│   ├── Start-VMs.Tests.ps1
│   └── Stop-VMs.Tests.ps1
├── .github/workflows/
│   └── runbook-ci.yml
└── README.md
```

**Branching Strategy**:
```
main (production)
    ├── release/v1.5.0
    └── staging
            └── develop
                    ├── feature/add-vm-tagging
                    ├── feature/improve-backup-logging
                    └── bugfix/fix-credential-issue
```

**Automation Account Sync Configuration**:
```powershell
# Production Automation Account → main branch
New-AzAutomationSourceControl `
    -Name "github-main-sync" `
    -SourceType "GitHub" `
    -RepoUrl "https://github.com/contoso/azure-automation-runbooks" `
    -Branch "main" `
    -FolderPath "/runbooks" `
    -PublishRunbook $true `
    -AutoSync $true `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-prod"

# Staging Automation Account → staging branch
New-AzAutomationSourceControl `
    -Name "github-staging-sync" `
    -SourceType "GitHub" `
    -RepoUrl "https://github.com/contoso/azure-automation-runbooks" `
    -Branch "staging" `
    -FolderPath "/runbooks" `
    -PublishRunbook $true `
    -AutoSync $true `
    -ResourceGroupName "rg-automation-staging" `
    -AutomationAccountName "automation-staging"

# Dev Automation Account → develop branch
New-AzAutomationSourceControl `
    -Name "github-develop-sync" `
    -SourceType "GitHub" `
    -RepoUrl "https://github.com/contoso/azure-automation-runbooks" `
    -Branch "develop" `
    -FolderPath "/runbooks" `
    -PublishRunbook $true `
    -AutoSync $true `
    -ResourceGroupName "rg-automation-dev" `
    -AutomationAccountName "automation-dev"
```

**Development Workflow**:
```bash
# Developer creates feature branch
git checkout develop
git pull origin develop
git checkout -b feature/add-vm-tagging

# Develop runbook locally
code runbooks/vm-management/Tag-VMs.ps1

# Test locally with Pester
Invoke-Pester -Path tests/

# Commit changes
git add runbooks/vm-management/Tag-VMs.ps1
git commit -m "Add VM tagging runbook with cost center logic"
git push origin feature/add-vm-tagging

# Open Pull Request: feature/add-vm-tagging → develop
# PR triggers CI pipeline (syntax validation, Pester tests)
# Code review by 2 team members
# Merge to develop

# Auto-sync pulls changes to dev Automation account
# Test runbook in dev environment

# Promote to staging (if tests pass)
# Open PR: develop → staging
# Merge to staging
# Auto-sync to staging Automation account
# Integration testing in staging

# Promote to production (if staging tests pass)
# Open PR: staging → main
# Merge to main (requires approval from ops team)
# Auto-sync to prod Automation account
# Runbook live in production ✅
```

**CI/CD Pipeline** (.github/workflows/runbook-ci.yml):
```yaml
name: Runbook CI

on:
  pull_request:
    branches: [ develop, staging, main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install PowerShell
        run: sudo apt-get install -y powershell
      
      - name: Validate PowerShell Syntax
        shell: pwsh
        run: |
          $errors = @()
          Get-ChildItem runbooks -Filter *.ps1 -Recurse | ForEach-Object {
            $null = [System.Management.Automation.PSParser]::Tokenize((Get-Content $_.FullName -Raw), [ref]$null)
            if ($?) {
              Write-Host "✓ $($_.Name) syntax valid"
            } else {
              $errors += "✗ $($_.Name) syntax error"
            }
          }
          if ($errors) {
            $errors | ForEach-Object { Write-Error $_ }
            exit 1
          }
      
      - name: Run Pester Tests
        shell: pwsh
        run: |
          Install-Module -Name Pester -Force -SkipPublisherCheck
          $results = Invoke-Pester -Path tests/ -OutputFormat NUnitXml -OutputFile TestResults.xml -PassThru
          if ($results.FailedCount -gt 0) {
            Write-Error "Tests failed: $($results.FailedCount) failed"
            exit 1
          }
      
      - name: Publish Test Results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: TestResults.xml
```

**Why other options are incorrect**:

**A) Single main branch, direct commits, auto-sync to prod**:
- ❌ **No code review**: Untested changes go directly to production
- ❌ **No rollback**: Difficult to revert broken runbooks
- ❌ **Collaboration nightmare**: 20 developers committing to same branch (merge conflicts)
- ❌ **High risk**: Production failures inevitable

**C) No source control, edit in Portal**:
- ❌ **No version history**: Can't see who changed what
- ❌ **No rollback**: Broken runbook = manual recovery
- ❌ **No collaboration**: Only one person can edit at a time
- ❌ **No testing**: Changes go directly to production
- ❌ **Disaster recovery**: Runbooks lost if Automation account deleted

**D) SharePoint with version history**:
- ❌ **Not designed for code**: No diff viewing, no merge tools
- ❌ **No CI/CD integration**: Manual testing, manual deployment
- ❌ **Poor collaboration**: Checkout/checkin model doesn't scale
- ❌ **No PR workflow**: No code review mechanism

**Collaboration Metrics** (50 runbooks, 20 developers):

| Strategy | Code Review | Deployment Time | Rollback Time | Collaboration Score |
|----------|-------------|-----------------|---------------|---------------------|
| **GitFlow (Answer B)** | ✅ PR required | 5-10 min (auto) | 2 min (git revert) | ⭐⭐⭐⭐⭐ |
| Single branch direct | ❌ None | 5 min (auto) | 30+ min (manual fix) | ⭐⭐ |
| Portal editing | ❌ None | Immediate | 30+ min (restore from backup) | ⭐ |
| SharePoint | ⚠️ Manual review | 30+ min (manual) | 30+ min (find old version) | ⭐⭐ |

**Key Takeaway**: Git-based source control with branching strategy enables professional collaboration, testing, and deployment for automation at scale.

</details>

---

## Question 6: PowerShell Workflow vs Script Decision

**Scenario**: You need to create a runbook that queries Azure Monitor logs for the past 24 hours and sends a summary email to the operations team. The query takes 30 seconds. The runbook runs once per day at 8 AM. Network is reliable. Which runbook type is most appropriate?

**A)** PowerShell Workflow (for resilience)  
**B)** PowerShell Runbook  
**C)** Python Runbook  
**D)** Graphical Runbook

<details>
<summary><strong>✅ Correct Answer</strong></summary>

**B) PowerShell Runbook**

**Explanation**:

**PowerShell Runbook** is the best choice because:

1. **Quick execution**: 30 seconds (well under 3-hour fair share limit)
2. **Simple task**: Query logs + send email (no parallel processing needed)
3. **Reliable network**: Checkpoints not required
4. **Fast startup**: No workflow compilation overhead
5. **Easier debugging**: Standard PowerShell syntax

**Implementation**:
```powershell
# Simple PowerShell runbook (no workflow needed)
param(
    [string]$EmailTo = "ops@contoso.com",
    [int]$HoursBack = 24
)

# Authenticate
Connect-AzAccount -Identity

# Query Azure Monitor logs
$query = @"
AzureActivity
| where TimeGenerated > ago($($HoursBack)h)
| where Level == "Error"
| summarize ErrorCount = count() by ResourceGroup, ResourceType
| order by ErrorCount desc
"@

$results = Invoke-AzOperationalInsightsQuery `
    -WorkspaceId "WORKSPACE_ID" `
    -Query $query

# Format results
$htmlBody = $results.Results | ConvertTo-Html -Fragment

# Send email
$mailParams = @{
    To = $EmailTo
    Subject = "Daily Azure Error Summary - $(Get-Date -Format 'yyyy-MM-dd')"
    Body = "<h2>Errors in Past $HoursBack Hours</h2>$htmlBody"
    BodyAsHtml = $true
    SmtpServer = "smtp.contoso.com"
    From = "automation@contoso.com"
}

Send-MailMessage @mailParams

Write-Output "✓ Summary email sent to $EmailTo"
```

**Execution Time**:
```
Workflow overhead: ~5-10 seconds (compilation)
Standard runbook: 0 seconds (immediate execution)

Total time:
- PowerShell Workflow: 35-40 seconds
- PowerShell Runbook: 30 seconds ✅ (faster)
```

**Why other options are incorrect**:

**A) PowerShell Workflow**:
- ❌ **Unnecessary overhead**: Workflow compilation adds 5-10 seconds
- ❌ **Overcomplicated**: No checkpoints needed for 30-second task
- ❌ **No parallel benefit**: Single query, no parallelization opportunity
- ❌ **Harder to debug**: Workflow syntax restrictions

**When to use Workflow instead**:
```powershell
# If requirements changed to:
# - Query 100 Log Analytics workspaces (long-running)
# - Process results in parallel
# - Network issues common (need resilience)
# THEN use Workflow:

workflow Query-MultipleWorkspaces {
    param([string[]]$WorkspaceIds)  # 100 workspaces
    
    ForEach -Parallel -ThrottleLimit 20 ($workspaceId in $WorkspaceIds) {
        InlineScript {
            Invoke-AzOperationalInsightsQuery -WorkspaceId $using:workspaceId -Query "..."
        }
    }
    
    Checkpoint-Workflow  # Resume if failure during processing
}
```

**C) Python Runbook**:
- ❌ Limited Azure SDK compared to PowerShell
- ❌ No native Azure authentication cmdlets (more code required)
- ❌ Email libraries less mature than PowerShell

**D) Graphical Runbook**:
- ❌ Overkill for simple query + email
- ❌ Harder to maintain (visual designer cumbersome for this task)
- ❌ Less flexible for query customization

**Decision Matrix**:

| Requirement | Workflow Needed? | Reasoning |
|-------------|------------------|-----------|
| < 5 minutes runtime | ❌ No | Standard runbook sufficient |
| Single resource query | ❌ No | No parallel benefit |
| Reliable network | ❌ No | No checkpoint benefit |
| Daily schedule | ❌ No | Re-run next day if failure |
| Simple logic | ❌ No | Workflow syntax restrictions unnecessary |

**Key Takeaway**: Use PowerShell Workflow only when you need resilience, parallelism, or long-running capabilities. For simple, quick tasks, standard PowerShell runbooks are faster and simpler.

</details>

---

## Question 7: Hybrid Runbook Worker Deployment

**Scenario**: You need to manage 100 on-premises SQL Servers that aren't accessible from the internet. The servers are in two datacenters (DC1 and DC2). Each datacenter has 50 SQL Servers. You want high availability and load balancing. Which Hybrid Worker deployment strategy is most appropriate?

**A)** Single Hybrid Worker in DC1, all jobs run on one machine  
**B)** Two worker groups: "dc1-workers" (3 workers) and "dc2-workers" (3 workers)  
**C)** One worker group: "onprem-workers" with 6 workers (3 in each datacenter)  
**D)** Install Hybrid Worker on each SQL Server (100 workers)

<details>
<summary><strong>✅ Correct Answer</strong></summary>

**B) Two worker groups: "dc1-workers" (3 workers) and "dc2-workers" (3 workers)**

**Explanation**:

**Separate worker groups per datacenter** provides:

1. **Geographic isolation**: DC1 jobs run in DC1, DC2 jobs run in DC2
2. **Network efficiency**: Workers access local SQL Servers (no cross-datacenter traffic)
3. **High availability**: 3 workers per datacenter (redundancy)
4. **Load balancing**: Jobs distributed across 3 workers in each group
5. **Targeted execution**: Runbooks specify which datacenter to target

**Implementation**:

**Step 1: Deploy workers in DC1**:
```powershell
# Install Hybrid Workers on 3 servers in DC1
$dc1Workers = @("onprem-worker-dc1-01", "onprem-worker-dc1-02", "onprem-worker-dc1-03")

foreach ($worker in $dc1Workers) {
    # Install Azure Arc agent on worker
    # Then add to worker group
    az automation hrwg hrw create `
        --automation-account-name "automation-prod" `
        --resource-group "rg-automation" `
        --hybrid-runbook-worker-group-name "dc1-workers" `
        --hybrid-runbook-worker-id $worker `
        --vm-resource-id "/subscriptions/.../machines/$worker"
}
```

**Step 2: Deploy workers in DC2** (same process for DC2):
```powershell
$dc2Workers = @("onprem-worker-dc2-01", "onprem-worker-dc2-02", "onprem-worker-dc2-03")

foreach ($worker in $dc2Workers) {
    az automation hrwg hrw create `
        --automation-account-name "automation-prod" `
        --resource-group "rg-automation" `
        --hybrid-runbook-worker-group-name "dc2-workers" `
        --hybrid-runbook-worker-id $worker `
        --vm-resource-id "/subscriptions/.../machines/$worker"
}
```

**Step 3: Create runbook with datacenter targeting**:
```powershell
workflow Backup-SqlDatabases {
    param(
        [string]$Datacenter,  # "DC1" or "DC2"
        [string[]]$SqlServers
    )
    
    # Determine worker group based on datacenter
    $workerGroup = switch ($Datacenter) {
        "DC1" { "dc1-workers" }
        "DC2" { "dc2-workers" }
    }
    
    Write-Output "Running on worker group: $workerGroup"
    
    # Backup databases in parallel (10 at a time)
    ForEach -Parallel -ThrottleLimit 10 ($server in $SqlServers) {
        InlineScript {
            Backup-SqlDatabase -ServerInstance $using:server -Database "master"
            Write-Output "✓ Backed up: $using:server"
        }
    }
}

# Execute on DC1 workers
Start-AzAutomationRunbook -Name "Backup-SqlDatabases" `
    -RunOn "dc1-workers" `
    -Parameters @{
        Datacenter = "DC1"
        SqlServers = @("sql-dc1-01", "sql-dc1-02", ..., "sql-dc1-50")
    }

# Execute on DC2 workers
Start-AzAutomationRunbook -Name "Backup-SqlDatabases" `
    -RunOn "dc2-workers" `
    -Parameters @{
        Datacenter = "DC2"
        SqlServers = @("sql-dc2-01", "sql-dc2-02", ..., "sql-dc2-50")
    }
```

**High Availability Scenario**:
```
Datacenter 1: Worker Group "dc1-workers"
    ├── worker-dc1-01 (Active) ✅
    ├── worker-dc1-02 (Active) ✅
    └── worker-dc1-03 (Active) ✅

Job Assignment:
- Job 1 (backup sql-dc1-01..10) → worker-dc1-01
- Job 2 (backup sql-dc1-11..20) → worker-dc1-02
- Job 3 (backup sql-dc1-21..30) → worker-dc1-03

Failure Scenario:
- worker-dc1-01 crashes during Job 1
- Job 1 re-routed to worker-dc1-02 (automatic failover) ✅
- Jobs continue processing
```

**Why other options are incorrect**:

**A) Single worker in DC1**:
- ❌ **Single point of failure**: Worker down = all automation stops
- ❌ **No load balancing**: One machine handles all 100 SQL Servers
- ❌ **Cross-datacenter traffic**: DC2 SQL Servers accessed from DC1 (slow, inefficient)
- ❌ **Performance bottleneck**: One worker can't handle parallel load

**C) One worker group with 6 workers across datacenters**:
- ❌ **Can't target specific datacenter**: Job may run on DC1 worker but target DC2 SQL Servers (cross-datacenter traffic)
- ❌ **Inefficient routing**: Azure picks worker randomly (no geographic awareness)
- ❌ **Network latency**: DC1 worker connecting to DC2 SQL Server across WAN

**Example of problem**:
```
Worker Group: onprem-workers (6 workers total)
- dc1-worker-01, dc1-worker-02, dc1-worker-03
- dc2-worker-01, dc2-worker-02, dc2-worker-03

Job: Backup sql-dc1-server-01
Azure assigns job to: dc2-worker-01 ❌
Result: dc2-worker-01 connects to DC1 SQL Server over WAN (slow)
Should run on: dc1-worker-XX (local network access) ✅
```

**D) Worker on each SQL Server (100 workers)**:
- ❌ **Massive overhead**: 100 workers × agent overhead × Azure Monitor costs
- ❌ **Management nightmare**: Update modules on 100 machines
- ❌ **Unnecessary**: SQL Servers are targets, not executors
- ❌ **Security risk**: Hybrid Worker on production SQL Server (attack surface)

**Resource Requirements Comparison**:

| Strategy | Workers | Management Overhead | HA Score | Cost |
|----------|---------|---------------------|----------|------|
| **2 groups, 3 workers each (Answer B)** | 6 | ⭐⭐⭐⭐⭐ Low | ⭐⭐⭐⭐⭐ High | 💰💰 Medium |
| Single worker | 1 | ⭐⭐⭐⭐⭐ Low | ⭐ None | 💰 Low |
| 1 group, 6 workers | 6 | ⭐⭐⭐⭐ Low | ⭐⭐⭐ Medium | 💰💰 Medium |
| 100 workers | 100 | ⭐ Very High | ⭐⭐⭐⭐⭐ High | 💰💰💰💰💰 Very High |

**Key Takeaway**: Worker groups should align with geographic/network boundaries for efficiency and targeted execution. Multiple workers per group provide HA and load balancing.

</details>

---

## Module Assessment Summary

**Score yourself**:
- 7/7 correct: ⭐⭐⭐⭐⭐ **Expert** - Ready for production Azure Automation implementations
- 5-6 correct: ⭐⭐⭐⭐ **Proficient** - Strong understanding, review missed topics
- 3-4 correct: ⭐⭐⭐ **Competent** - Good foundation, practice more scenarios
- 1-2 correct: ⭐⭐ **Developing** - Review module content, hands-on labs recommended
- 0 correct: ⭐ **Beginner** - Re-read module, complete practice exercises

**Key Topics Covered**:
1. ✅ Automation account strategy (environment separation)
2. ✅ Runbook type selection (Workflow vs Script)
3. ✅ Shared resources (Credentials for secure storage)
4. ✅ Webhooks (Event-driven automation with security)
5. ✅ Source control (GitFlow for collaboration)
6. ✅ Workflow decision criteria (When to use workflows)
7. ✅ Hybrid Workers (Geographic deployment strategy)

## Next Steps

Congratulations on completing the module assessment! Proceed to **Unit 13: Summary** for a comprehensive review of all Azure Automation concepts, additional resources, and recommendations for practical implementation in your environment.

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: This assessment is based on concepts from https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/
