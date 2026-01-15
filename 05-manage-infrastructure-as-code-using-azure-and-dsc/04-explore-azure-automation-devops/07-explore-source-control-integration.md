# Explore Source Control Integration

## Overview

Source control integration connects your Azure Automation account with Git repositories (GitHub or Azure DevOps), enabling you to manage runbooks as code. Instead of editing runbooks directly in the Azure Portal, you can develop locally, use version control, collaborate with your team, and implement CI/CD pipelines for your automation infrastructure.

This unit covers how to configure source control integration, sync runbooks between repositories and Automation accounts, implement branching strategies, and leverage source control for professional automation development workflows.

## Why Source Control for Runbooks?

**Benefits of Source Control Integration**:

1. **Version History**: Track every change to runbooks with commit history
2. **Collaboration**: Multiple team members can work on automation simultaneously
3. **Code Reviews**: Pull requests ensure quality before deployment
4. **Rollback**: Revert to previous runbook versions instantly
5. **Branching**: Develop features in isolation, test before merging
6. **CI/CD**: Automate testing and deployment of runbooks
7. **Disaster Recovery**: Repository serves as backup of all runbooks
8. **Compliance**: Audit trail of who changed what and when

**Without Source Control** (Manual Management):
```
Developer A: Edits runbook in Azure Portal
Developer B: Also edits same runbook (unaware of A's changes)
Result: B's changes overwrite A's work (lost changes)
No history, no rollback, no collaboration
```

**With Source Control** (Professional Workflow):
```
Developer A: Creates feature branch, commits changes, opens pull request
Developer B: Reviews A's PR, suggests improvements, approves
CI/CD: Automated tests run on PR
Merge: Changes deployed to Automation account
History: Full audit trail preserved in Git
```

## Supported Source Control Systems

Azure Automation supports three source control types:

### 1. GitHub

**Characteristics**:
- **Repository**: Public or private GitHub repositories
- **Authentication**: Personal Access Token (PAT) or GitHub App
- **Branch Support**: Any branch (main, develop, feature branches)
- **Organization**: Supports GitHub organizations and personal repos

**Best For**:
- Open-source automation projects
- Teams already using GitHub
- Integration with GitHub Actions for CI/CD
- Public documentation and community sharing

### 2. Azure DevOps Git

**Characteristics**:
- **Repository**: Azure DevOps Git repositories (Azure Repos)
- **Authentication**: Personal Access Token (PAT)
- **Branch Support**: Any branch
- **Integration**: Native Azure DevOps integration

**Best For**:
- Enterprise Azure DevOps shops
- Organizations using Azure Boards for planning
- Integration with Azure Pipelines
- Private enterprise repositories

### 3. Azure DevOps TFVC (Team Foundation Version Control)

**Characteristics**:
- **Repository**: Azure DevOps TFVC repositories
- **Authentication**: Personal Access Token (PAT)
- **Folder Support**: Specific folder paths
- **Legacy**: Older version control system (Git recommended for new projects)

**Best For**:
- Organizations with existing TFVC investments
- Legacy projects not yet migrated to Git
- Centralized version control workflows

**Recommendation**: Use **Git** (GitHub or Azure DevOps Git) for new projects. TFVC is legacy.

## Source Control Integration Setup

### Prerequisites

**Required Permissions**:
- **Automation Account**: Contributor or Automation Contributor role
- **GitHub**: Repository admin or write access
- **Azure DevOps**: Project administrator or repository contributor

**Required Resources**:
- Published runbooks in Automation account (optional, can start empty)
- Git repository with runbooks folder structure
- Personal Access Token (PAT) for authentication

### GitHub Integration

#### Step 1: Create Personal Access Token

**GitHub PAT Creation**:

1. Navigate to GitHub Settings
2. Click **Developer settings** → **Personal access tokens** → **Tokens (classic)**
3. Click **Generate new token** → **Generate new token (classic)**
4. Configure token:
   ```
   Note: Azure Automation Integration - MyAutomationAccount
   Expiration: 90 days (or custom)
   Scopes:
     ☑ repo (Full control of private repositories)
       ☑ repo:status
       ☑ repo_deployment
       ☑ public_repo
       ☑ repo:invite
   ```
5. Click **Generate token**
6. **Copy token immediately** (shown only once)
   ```
   ghp_abc123xyz789...
   ```

**Store Token Securely**:
```powershell
# Store in Azure Key Vault
$tokenSecure = ConvertTo-SecureString "ghp_abc123xyz789..." -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName "kv-automation-secrets" `
    -Name "GitHub-PAT-Automation" `
    -SecretValue $tokenSecure
```

#### Step 2: Configure Integration in Azure Portal

**Azure Portal Steps**:

1. Navigate to Automation Account
2. Click **Source control** in left menu
3. Click **+ Add**
4. Configure connection:
   ```
   Name: github-runbooks-sync
   Source control type: GitHub
   Repository: https://github.com/myorg/azure-automation-runbooks
   Branch: main
   Folder path: /runbooks (path within repo)
   Auto sync: Enabled
   Publish runbook: Enabled
   ```
5. **Provide Credentials**:
   - Click **Configure credentials**
   - Paste GitHub PAT
6. Click **Save**

**Configuration Fields Explained**:

| Field | Description | Example |
|-------|-------------|---------|
| **Name** | Friendly name for connection | github-runbooks-sync |
| **Source control type** | Git provider | GitHub, Azure DevOps Git, TFVC |
| **Repository** | Full repository URL | https://github.com/myorg/automation |
| **Branch** | Target branch to sync | main, develop, production |
| **Folder path** | Folder containing runbooks | /runbooks, /automation/runbooks |
| **Auto sync** | Automatic sync on commits | Enabled (recommended) |
| **Publish runbook** | Auto-publish after sync | Enabled for prod, Disabled for dev |

#### Step 3: Verify Integration

**Check Sync Status**:

```powershell
# Get source control configuration
Get-AzAutomationSourceControl -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" -Name "github-runbooks-sync" |
    Select-Object Name, SourceType, Branch, FolderPath, AutoSync, PublishRunbook

# Trigger manual sync (if auto-sync disabled)
Start-AzAutomationSourceControlSyncJob -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-runbooks-sync"

# Check sync job status
Get-AzAutomationSourceControlSyncJob -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-runbooks-sync" |
    Select-Object SourceControlSyncJobId, CreationTime, ProvisioningState, StartTime, EndTime
```

### Azure DevOps Git Integration

**Differences from GitHub**:
- Repository URL format: `https://dev.azure.com/organization/project/_git/repository`
- PAT created in Azure DevOps user settings
- Required scope: **Code (Read & Write)**

**Azure DevOps PAT Creation**:

1. Navigate to Azure DevOps organization
2. Click **User Settings** (icon) → **Personal access tokens**
3. Click **+ New Token**
4. Configure:
   ```
   Name: Azure Automation Integration
   Organization: myorg
   Expiration: 90 days
   Scopes: Custom defined
     ☑ Code (Read & Write)
   ```
5. Click **Create** and copy token

**Integration Configuration**:

```powershell
# Configure Azure DevOps Git integration
New-AzAutomationSourceControl `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "azdo-git-sync" `
    -SourceType "VsoGit" `
    -RepoUrl "https://dev.azure.com/myorg/MyProject/_git/automation" `
    -Branch "main" `
    -FolderPath "/runbooks" `
    -AccessToken $azdoPAT `
    -PublishRunbook $true `
    -AutoSync $true
```

## Repository Structure

### Recommended Folder Structure

```
azure-automation-runbooks/
├── runbooks/
│   ├── vm-management/
│   │   ├── Start-AzureVMs.ps1
│   │   ├── Stop-AzureVMs.ps1
│   │   └── Restart-AzureVMs.ps1
│   ├── monitoring/
│   │   ├── Check-ResourceHealth.ps1
│   │   └── Send-StatusReport.ps1
│   ├── backup/
│   │   └── Backup-AzureResources.ps1
│   └── utilities/
│       ├── Send-Notification.ps1
│       └── Write-Log.ps1
├── modules/
│   └── CustomModule.psm1
├── tests/
│   ├── Start-AzureVMs.Tests.ps1
│   └── Stop-AzureVMs.Tests.ps1
├── .github/
│   └── workflows/
│       └── runbook-ci.yml
├── README.md
└── .gitignore
```

**Folder Path Configuration**:
- If `Folder path = /runbooks`, Azure Automation syncs only `runbooks/` folder
- All `.ps1` files in that folder become runbooks
- Subfolders ignored (flat structure in Automation account)

### Runbook Naming Convention

**File Names**:
```
Verb-Noun-Details.ps1

Examples:
Start-AzureVMs.ps1          → Runbook name: Start-AzureVMs
Stop-ProductionVMs.ps1      → Runbook name: Stop-ProductionVMs
Backup-DatabaseDaily.ps1    → Runbook name: Backup-DatabaseDaily
```

**Runbook Type Detection**:
- `.ps1` files → PowerShell runbooks
- `.ps1` with `workflow` keyword → PowerShell Workflow runbooks
- `.py` files → Python runbooks
- `.graphrunbook` files → Graphical runbooks (JSON format)

### .gitignore Template

```gitignore
# Azure Automation .gitignore

# Secrets and credentials (NEVER commit)
*.secret
*.token
credentials.json
appsettings.json

# Local testing artifacts
*.log
*.tmp
test-results/

# IDE and editor files
.vscode/
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# PowerShell Gallery artifacts
*.nupkg

# Backup files
*.bak
*.backup
```

## Sync Workflow

### Auto-Sync (Recommended)

**How It Works**:

```
Developer commits to GitHub/Azure DevOps
    ↓
Webhook notifies Azure Automation (if configured)
    ↓
Azure Automation polls repository (every 5 minutes by default)
    ↓
Detects new commit
    ↓
Downloads changed runbooks
    ↓
Creates/updates runbooks in Automation account
    ↓
Publishes runbooks (if PublishRunbook = true)
    ↓
Runbooks available for execution
```

**Sync Frequency**:
- **Auto-Sync Enabled**: Every ~5 minutes
- **Webhook Triggered**: Near real-time (if webhook configured)
- **Manual Sync**: On-demand via portal or PowerShell

### Manual Sync

**Trigger Sync via PowerShell**:

```powershell
# Start sync job
$syncJob = Start-AzAutomationSourceControlSyncJob `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-runbooks-sync"

Write-Output "Sync job started: $($syncJob.SourceControlSyncJobId)"

# Wait for completion
$timeout = 300  # 5 minutes
$elapsed = 0
do {
    Start-Sleep -Seconds 10
    $elapsed += 10
    
    $status = Get-AzAutomationSourceControlSyncJob `
        -ResourceGroupName "rg-automation-prod" `
        -AutomationAccountName "automation-account-prod" `
        -Name "github-runbooks-sync" `
        -SourceControlSyncJobId $syncJob.SourceControlSyncJobId
    
    Write-Output "Sync status: $($status.ProvisioningState) (elapsed: ${elapsed}s)"
    
} while ($status.ProvisioningState -eq "Running" -and $elapsed -lt $timeout)

if ($status.ProvisioningState -eq "Completed") {
    Write-Output "✓ Sync completed successfully"
    
    # Get sync job output
    $output = Get-AzAutomationSourceControlSyncJobOutput `
        -ResourceGroupName "rg-automation-prod" `
        -AutomationAccountName "automation-account-prod" `
        -Name "github-runbooks-sync" `
        -SourceControlSyncJobId $syncJob.SourceControlSyncJobId
    
    Write-Output "Sync Details:"
    $output | ForEach-Object { Write-Output $_.Summary }
}
else {
    Write-Error "✗ Sync failed or timed out. Status: $($status.ProvisioningState)"
}
```

### Sync Conflicts

**Handling Conflicts**:

Source control sync is **one-way** (repository → Automation account):
- Changes in repository **overwrite** Automation account runbooks
- Changes made in Azure Portal are **lost** on next sync
- **Best Practice**: Always edit runbooks in repository, never in portal

**Workflow to Avoid Conflicts**:

```
✅ CORRECT Workflow:
1. Edit runbook in local Git repository
2. Commit and push to GitHub/Azure DevOps
3. Auto-sync pulls changes to Automation account
4. Test runbook in Automation account
5. If issues, fix in repository and repeat

❌ INCORRECT Workflow:
1. Edit runbook in Azure Portal
2. Commit unrelated change to repository
3. Auto-sync overwrites portal changes with repository version
4. Portal changes lost forever
```

**Emergency Portal Edit Recovery**:

```powershell
# If you accidentally edited in portal and need to save:

# 1. Export runbook from portal
Export-AzAutomationRunbook -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "Modified-Runbook" `
    -Slot Published `
    -OutputFolder "C:\Temp"

# 2. Copy content to repository
Copy-Item "C:\Temp\Modified-Runbook.ps1" `
    -Destination "C:\Git\azure-automation-runbooks\runbooks\Modified-Runbook.ps1" `
    -Force

# 3. Commit to repository
cd C:\Git\azure-automation-runbooks
git add runbooks/Modified-Runbook.ps1
git commit -m "Emergency: Recovered portal edits for Modified-Runbook"
git push origin main

# 4. Next sync will preserve your changes
```

## Branching Strategies

### Strategy 1: Environment Branches

**Structure**:
```
Repository Branches:
- main (production)
- staging
- develop

Azure Automation Accounts:
- automation-account-prod → syncs from "main" branch
- automation-account-staging → syncs from "staging" branch  
- automation-account-dev → syncs from "develop" branch
```

**Workflow**:
```
Developer creates feature branch from develop
    ↓
Commits and tests locally
    ↓
Opens PR to develop branch
    ↓
Code review and approval
    ↓
Merge to develop → syncs to dev Automation account
    ↓
Test in dev environment
    ↓
Open PR: develop → staging
    ↓
Merge to staging → syncs to staging Automation account
    ↓
Test in staging (production-like)
    ↓
Open PR: staging → main
    ↓
Merge to main → syncs to prod Automation account
    ↓
Production deployment complete
```

**PowerShell Configuration**:

```powershell
# Production Automation Account → main branch
New-AzAutomationSourceControl `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-main-sync" `
    -SourceType "GitHub" `
    -RepoUrl "https://github.com/myorg/automation" `
    -Branch "main" `
    -FolderPath "/runbooks" `
    -PublishRunbook $true `
    -AutoSync $true

# Staging Automation Account → staging branch
New-AzAutomationSourceControl `
    -ResourceGroupName "rg-automation-staging" `
    -AutomationAccountName "automation-account-staging" `
    -Name "github-staging-sync" `
    -SourceType "GitHub" `
    -RepoUrl "https://github.com/myorg/automation" `
    -Branch "staging" `
    -FolderPath "/runbooks" `
    -PublishRunbook $true `
    -AutoSync $true
```

### Strategy 2: Single Branch with Manual Promotion

**Structure**:
- Single `main` branch
- All Automation accounts sync from `main`
- Use tags/releases for production deployments

**Workflow**:
```
All development on main branch
    ↓
Tag release: v1.2.3
    ↓
Manually trigger sync in production Automation account
    ↓
Verify production runbooks match tagged release
```

**Git Tagging**:

```bash
# Create release tag
git tag -a v1.2.3 -m "Release 1.2.3: Added VM auto-scaling runbook"
git push origin v1.2.3

# List tags
git tag -l
```

### Strategy 3: Trunk-Based Development

**Structure**:
- Single `main` branch
- Short-lived feature branches (< 2 days)
- Continuous integration to main
- Feature flags for incomplete features

**Best For**:
- Small teams
- High maturity DevOps practices
- Frequent deployments

## CI/CD Integration

### GitHub Actions Pipeline

**Runbook Testing and Validation**:

```yaml
name: Runbook CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install PowerShell
        run: |
          sudo apt-get update
          sudo apt-get install -y powershell
      
      - name: Install Pester (PowerShell testing framework)
        shell: pwsh
        run: Install-Module -Name Pester -Force -SkipPublisherCheck
      
      - name: Run Pester Tests
        shell: pwsh
        run: |
          $testResults = Invoke-Pester -Path ./tests -OutputFormat NUnitXml -OutputFile TestResults.xml -PassThru
          if ($testResults.FailedCount -gt 0) {
            Write-Error "Tests failed: $($testResults.FailedCount) failed"
            exit 1
          }
      
      - name: Publish Test Results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: TestResults.xml
  
  validate-syntax:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate PowerShell Syntax
        shell: pwsh
        run: |
          $errors = @()
          Get-ChildItem -Path ./runbooks -Filter *.ps1 -Recurse | ForEach-Object {
            $content = Get-Content $_.FullName -Raw
            $null = [System.Management.Automation.PSParser]::Tokenize($content, [ref]$null)
            if ($?) {
              Write-Output "✓ $($_.Name) syntax valid"
            } else {
              $errors += "✗ $($_.Name) syntax error"
            }
          }
          if ($errors.Count -gt 0) {
            $errors | ForEach-Object { Write-Error $_ }
            exit 1
          }
  
  deploy-to-staging:
    needs: [test, validate-syntax]
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Azure Automation Sync
        run: |
          echo "Azure Automation auto-sync will pull changes from develop branch"
          # Or trigger manual sync via webhook/API
```

### Azure DevOps Pipeline

```yaml
trigger:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: 'windows-latest'

stages:
  - stage: Test
    jobs:
      - job: RunPesterTests
        steps:
          - powershell: |
              Install-Module -Name Pester -Force -Scope CurrentUser
              $testResults = Invoke-Pester -Path $(Build.SourcesDirectory)\tests -OutputFormat NUnitXml -OutputFile testResults.xml -PassThru
              if ($testResults.FailedCount -gt 0) {
                Write-Error "Tests failed"
                exit 1
              }
            displayName: 'Run Pester Tests'
          
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'NUnit'
              testResultsFiles: '**/testResults.xml'
            condition: always()
  
  - stage: Deploy
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - job: TriggerAutomationSync
        steps:
          - task: AzurePowerShell@5
            inputs:
              azureSubscription: 'Azure-Automation-ServiceConnection'
              scriptType: 'InlineScript'
              inline: |
                Start-AzAutomationSourceControlSyncJob `
                  -ResourceGroupName "rg-automation-prod" `
                  -AutomationAccountName "automation-account-prod" `
                  -Name "azdo-git-sync"
                
                Write-Output "Sync job triggered for production"
              azurePowerShellVersion: 'LatestVersion'
```

## Best Practices

### 1. Never Edit Runbooks in Portal

```powershell
# ❌ BAD: Editing in Azure Portal
# Changes lost on next sync from repository

# ✅ GOOD: Always edit in repository
git clone https://github.com/myorg/automation.git
cd automation/runbooks
code Start-AzureVMs.ps1  # Edit locally
git add Start-AzureVMs.ps1
git commit -m "Updated VM start logic for new tagging strategy"
git push origin develop
# Auto-sync pulls changes to Automation account
```

### 2. Use Pull Requests for Quality

```bash
# Create feature branch
git checkout -b feature/add-cost-optimization-runbook

# Develop runbook
cat > runbooks/Optimize-AzureCosts.ps1 << 'EOF'
# Cost optimization runbook
param([string]$ResourceGroupName)
# ... runbook code ...
EOF

# Commit changes
git add runbooks/Optimize-AzureCosts.ps1
git commit -m "Add: Cost optimization runbook for unused resources"

# Push and create PR
git push origin feature/add-cost-optimization-runbook
# Open PR in GitHub/Azure DevOps web UI
```

### 3. Implement Automated Testing

**Pester Test Example**:

```powershell
# tests/Start-AzureVMs.Tests.ps1

Describe "Start-AzureVMs Runbook Tests" {
    BeforeAll {
        # Import runbook for testing
        . "$PSScriptRoot/../runbooks/Start-AzureVMs.ps1"
    }
    
    Context "Parameter Validation" {
        It "Should accept valid ResourceGroupName" {
            { ValidateParameters -ResourceGroupName "rg-prod-vms" } | Should -Not -Throw
        }
        
        It "Should reject empty ResourceGroupName" {
            { ValidateParameters -ResourceGroupName "" } | Should -Throw
        }
    }
    
    Context "VM Start Logic" {
        It "Should skip already running VMs" {
            Mock Get-AzVM { @{Name="vm01"; PowerState="VM running"} }
            Mock Start-AzVM { }
            
            Start-TaggedVMs -ResourceGroupName "rg-test"
            
            Assert-MockCalled Start-AzVM -Times 0
        }
        
        It "Should start stopped VMs" {
            Mock Get-AzVM { @{Name="vm01"; PowerState="VM deallocated"} }
            Mock Start-AzVM { }
            
            Start-TaggedVMs -ResourceGroupName "rg-test"
            
            Assert-MockCalled Start-AzVM -Times 1
        }
    }
}
```

### 4. Secure PAT Management

```powershell
# Store PAT in Key Vault, not code
$pat = Get-AzKeyVaultSecret -VaultName "kv-automation" -Name "GitHub-PAT" -AsPlainText

# Rotate PAT regularly (every 90 days recommended)
# Script to rotate PAT and update source control
$newPat = Read-Host "Enter new GitHub PAT" -AsSecureString
$newPatPlain = [Runtime.InteropServices.Marshal]::PtrToStringAuto(
    [Runtime.InteropServices.Marshal]::SecureStringToBSTR($newPat)
)

# Update source control with new PAT
Update-AzAutomationSourceControl `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-runbooks-sync" `
    -AccessToken $newPatPlain

# Store new PAT in Key Vault
Set-AzKeyVaultSecret -VaultName "kv-automation" `
    -Name "GitHub-PAT" `
    -SecretValue $newPat
```

### 5. Document Repository Structure

**README.md Template**:

```markdown
# Azure Automation Runbooks

## Overview
This repository contains Azure Automation runbooks for production infrastructure management.

## Repository Structure
```
runbooks/
  vm-management/     - VM lifecycle automation
  monitoring/        - Health checks and monitoring
  backup/           - Backup automation
  utilities/        - Helper runbooks
```

## Development Workflow
1. Create feature branch from `develop`
2. Develop and test runbook locally
3. Open PR to `develop` branch
4. Code review and approval required
5. Merge to `develop` → syncs to dev Automation account
6. Test in dev environment
7. Promote through staging to production

## Testing
Run Pester tests locally before committing:
```powershell
Invoke-Pester -Path ./tests
```

## Source Control Integration
- **Dev**: `develop` branch → automation-account-dev
- **Staging**: `staging` branch → automation-account-staging
- **Production**: `main` branch → automation-account-prod

## Contributing
See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.
```

## Troubleshooting

### Issue 1: Sync Job Fails with Authentication Error

**Symptoms**:
```
Sync job failed: Authentication failed
ProvisioningState: Failed
```

**Causes**:
- PAT expired
- PAT revoked
- Insufficient permissions

**Solution**:
```powershell
# Generate new PAT in GitHub/Azure DevOps
# Update source control with new token
Update-AzAutomationSourceControl `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-runbooks-sync" `
    -AccessToken "NEW_PAT_HERE"
```

### Issue 2: Runbooks Not Appearing After Sync

**Causes**:
- Incorrect folder path
- Files not in synced folder
- File extension not recognized

**Solution**:
```powershell
# Verify folder path matches repository structure
Get-AzAutomationSourceControl -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" -Name "github-runbooks-sync" |
    Select-Object FolderPath

# Expected: /runbooks
# Actual repository structure: runbooks/Start-VMs.ps1

# If mismatch, update folder path
Update-AzAutomationSourceControl `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-runbooks-sync" `
    -FolderPath "/runbooks"
```

### Issue 3: Sync Overwriting Portal Changes

**Prevention**:
- Disable source control temporarily for emergency edits
- Always export runbook from portal before editing
- Re-enable source control after fixing in repository

```powershell
# Temporary disable auto-sync
Update-AzAutomationSourceControl `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-runbooks-sync" `
    -AutoSync $false

# Make emergency portal edits
# Export and commit to repository

# Re-enable auto-sync
Update-AzAutomationSourceControl `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "github-runbooks-sync" `
    -AutoSync $true
```

## Next Steps

Now that you understand source control integration for runbooks, proceed to **Unit 8: Explore PowerShell Workflows** to learn about PowerShell Workflow runbooks that provide checkpoints, parallel processing, and resilience for long-running automation tasks.

**Quick Start Actions**:
1. Create a GitHub or Azure DevOps repository for runbooks
2. Generate a Personal Access Token
3. Configure source control integration in your Automation account
4. Commit a simple test runbook to the repository
5. Verify auto-sync pulls the runbook into Automation account

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/7-explore-source-control-integration
