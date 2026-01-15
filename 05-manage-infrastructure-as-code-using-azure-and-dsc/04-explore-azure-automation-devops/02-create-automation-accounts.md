# Create Automation Accounts

## Overview

An Azure Automation account is a container that stores all your automation artifacts—runbooks, jobs, shared resources (credentials, variables, connections), and configurations. It serves as a logical boundary for organizing automation assets and managing access control. Understanding how to create, configure, and organize Automation accounts is fundamental to implementing automation at scale.

This unit covers the process of creating Automation accounts, configuring authentication through Run As accounts, implementing best practices for account organization, and understanding permissions and limitations.

## What is an Automation Account?

An Automation account is similar to an Azure Storage account—it provides a namespace and container for your automation resources. All automation artifacts are stored within this account and managed as a cohesive unit.

### Key Components

**Automation Account Contains**:
- **Runbooks**: Your automation scripts (PowerShell, Python, Graphical)
- **Jobs**: Execution history and logs of runbook runs
- **Schedules**: Time-based triggers for automatic runbook execution
- **Modules**: PowerShell modules providing cmdlets
- **Assets**: Shared resources (credentials, variables, connections, certificates)
- **DSC Configurations**: Desired State Configuration scripts
- **Hybrid Worker Groups**: On-premises automation agents

### Account Properties

**Key Attributes**:
- **Name**: Unique within a region (3-50 characters, alphanumeric and hyphens)
- **Resource Group**: Logical container for related Azure resources
- **Location**: Azure region where account metadata is stored (runbooks can execute anywhere)
- **Subscription**: Azure subscription that owns and bills the account
- **Pricing Tier**: Basic (included with subscription) or Advanced (additional features)

## Creating an Automation Account

### Method 1: Azure Portal

**Step-by-Step Process**:

1. **Navigate to Azure Automation**:
   - Sign in to Azure Portal (portal.azure.com)
   - Search for "Automation Accounts" in the search bar
   - Click "Create" or "+ Add"

2. **Configure Basic Settings**:
   ```
   Subscription: Select your Azure subscription
   Resource Group: Create new or select existing (e.g., "rg-automation-prod")
   Name: automation-account-prod
   Region: East US (or closest to your resources)
   ```

3. **Configure Authentication**:
   - **Create Azure Run As account**: Yes (default)
     - Automatically creates an Azure AD service principal
     - Grants Contributor role to the automation account
     - Allows runbooks to authenticate to Azure
   
   **Important**: Creating Run As accounts requires subscription Owner permissions. If you lack permissions, you'll see a warning:
   
   ```
   ⚠️ You don't have permissions to create an Azure Run As account
   
   To create Run As accounts, you must be a subscription Owner or have equivalent permissions.
   Without a Run As account, runbooks cannot authenticate to Azure automatically.
   
   Options:
   - Request Owner permissions from your subscription administrator
   - Create service principal manually and configure in Automation account
   - Use alternative authentication (managed identity, stored credentials)
   ```

4. **Review and Create**:
   - Review all settings
   - Check estimated cost (Basic tier is free; charges apply for job runtime)
   - Click "Create"
   - Deployment takes 1-2 minutes

**Visual Reference**: The Azure Portal shows an "Add Automation Account" blade with fields for subscription, resource group, name, region, and the "Create Azure Run As account" option highlighted.

### Method 2: Azure CLI

Create Automation account using Azure CLI:

```bash
#!/bin/bash
# Create Automation account via CLI

RESOURCE_GROUP="rg-automation-prod"
LOCATION="eastus"
ACCOUNT_NAME="automation-account-prod"

# Create resource group
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION

# Create Automation account
az automation account create \
    --resource-group $RESOURCE_GROUP \
    --name $ACCOUNT_NAME \
    --location $LOCATION \
    --sku Basic

# Verify creation
az automation account show \
    --resource-group $RESOURCE_GROUP \
    --name $ACCOUNT_NAME \
    --output table
```

**Output**:
```
Name                     ResourceGroup          Location    State      Sku
-----------------------  ---------------------  ----------  ---------  ------
automation-account-prod  rg-automation-prod     eastus      Ok         Basic
```

### Method 3: Azure PowerShell

Create Automation account using PowerShell:

```powershell
# Create Automation account via PowerShell

$resourceGroup = "rg-automation-prod"
$location = "East US"
$accountName = "automation-account-prod"

# Create resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create Automation account
New-AzAutomationAccount `
    -ResourceGroupName $resourceGroup `
    -Name $accountName `
    -Location $location `
    -Plan Basic

# Get account details
Get-AzAutomationAccount `
    -ResourceGroupName $resourceGroup `
    -Name $accountName | Format-List
```

**Output**:
```
ResourceGroupName   : rg-automation-prod
Location            : eastus
State               : Ok
CreationTime        : 1/14/2026 10:30:00 AM
LastModifiedTime    : 1/14/2026 10:30:00 AM
```

### Method 4: ARM Template

Create Automation account using Infrastructure as Code:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "accountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Automation account"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for the Automation account"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Automation/automationAccounts",
      "apiVersion": "2021-06-22",
      "name": "[parameters('accountName')]",
      "location": "[parameters('location')]",
      "properties": {
        "sku": {
          "name": "Basic"
        }
      },
      "tags": {
        "Environment": "Production",
        "Department": "IT Operations"
      }
    }
  ],
  "outputs": {
    "accountId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Automation/automationAccounts', parameters('accountName'))]"
    }
  }
}
```

**Deploy Template**:
```bash
az deployment group create \
    --resource-group rg-automation-prod \
    --template-file automation-account.json \
    --parameters accountName=automation-account-prod
```

## Permissions and Requirements

### Required Permissions

**To Create Automation Account**:
- **Minimum**: Contributor role on the resource group
- **For Run As Account**: Owner role on the subscription

**Permission Hierarchy**:
```
Subscription Owner
└── Can create Automation account with Run As account
    └── Run As account gets Contributor role on subscription
    
Subscription Contributor
└── Can create Automation account
    └── Cannot create Run As account (lacks permissions)
        └── Must use alternative authentication
```

### Run As Account Components

When you create a Run As account, Azure automatically creates:

1. **Azure AD Application**:
   - Application registration in Azure Active Directory
   - Identifier: Application (client) ID
   - Secret: Certificate-based authentication (more secure than password)

2. **Service Principal**:
   - Service principal linked to the AD application
   - Used for runbook authentication to Azure
   - Assigned Contributor role on subscription by default

3. **Certificate**:
   - Self-signed certificate for authentication
   - Stored in Automation account Certificates asset
   - Valid for 1 year (auto-renewed before expiration)
   - Certificate name: `AzureRunAsCertificate`

4. **Connection**:
   - Automation connection asset named `AzureRunAsConnection`
   - Contains: Application ID, Tenant ID, Subscription ID, Certificate thumbprint
   - Used by runbooks to authenticate: `Connect-AzAccount -ServicePrincipal -CertificateThumbprint`

**Authentication Flow**:
```
Runbook Execution
    ↓
Get-AutomationConnection -Name "AzureRunAsConnection"
    ↓
Extract: ApplicationId, TenantId, CertificateThumbprint
    ↓
Connect-AzAccount -ServicePrincipal -Tenant $TenantId -ApplicationId $ApplicationId -CertificateThumbprint $Thumbprint
    ↓
Authenticated Session (Contributor access to subscription)
    ↓
Execute Azure Commands (New-AzVM, Get-AzStorageAccount, etc.)
```

### Alternative Authentication Methods

If you cannot create Run As account:

**1. Managed Identity (Recommended for Azure-hosted runbooks)**:
```powershell
# Enable system-assigned managed identity on Automation account
# Azure Portal: Automation Account → Identity → System assigned → On

# In runbook, authenticate using managed identity
Connect-AzAccount -Identity
```

**Advantages**:
- No certificates or secrets to manage
- Automatic credential rotation
- Scoped to specific Automation account
- Works only for runbooks executing in Azure (not hybrid workers)

**2. Stored Credentials**:
```powershell
# Store username/password in Automation Credentials
# Azure Portal: Automation Account → Credentials → Add

# In runbook, retrieve and use credential
$credential = Get-AutomationPSCredential -Name "AzureAdmin"
Connect-AzAccount -Credential $credential
```

**Disadvantages**:
- Less secure (username/password vs. certificate)
- Manual credential rotation required
- Risk of password expiration breaking automation

**3. Azure Key Vault Integration**:
```powershell
# Store service principal secret in Key Vault
# Grant Automation account Managed Identity access to Key Vault

# In runbook
Connect-AzAccount -Identity
$secret = Get-AzKeyVaultSecret -VaultName "kv-automation" -Name "sp-secret" -AsPlainText
$credential = New-Object System.Management.Automation.PSCredential($appId, (ConvertTo-SecureString $secret -AsPlainText -Force))
Connect-AzAccount -ServicePrincipal -Credential $credential -Tenant $tenantId
```

**Best Practice**: Use Key Vault for all sensitive values, not Automation Credentials.

## Best Practices for Automation Accounts

### 1. Account Organization Strategy

**Multiple Accounts for Isolation**:

Create separate Automation accounts to segregate and limit scope:

```
Organizational Structure:

automation-account-dev
└── Purpose: Development and testing of new runbooks
└── Environment: Non-production resources only
└── Access: Development team has Contributor access
└── Risk: Low (mistakes only affect dev resources)

automation-account-staging
└── Purpose: Pre-production testing and validation
└── Environment: Staging subscription
└── Access: Limited to DevOps engineers
└── Risk: Medium (similar to prod configuration)

automation-account-prod
└── Purpose: Production workload automation
└── Environment: Production subscription
└── Access: Restricted (read-only for most, Contributor for automation admin)
└── Risk: High (impacts customer-facing services)

automation-account-onprem
└── Purpose: Managing on-premises resources
└── Environment: Hybrid Runbook Workers in datacenter
└── Access: Infrastructure team only
└── Risk: High (impacts datacenter operations)
```

**Naming Convention**:
```
automation-<purpose>-<environment>-<region>

Examples:
- automation-vm-lifecycle-prod-eastus
- automation-backup-prod-westeurope
- automation-compliance-dev-eastus
- automation-incident-response-prod-centralus
```

### 2. Subscription Limits and Quotas

**Key Limits** (per subscription):
- **Automation Accounts**: Maximum 30 per subscription
- **Jobs**: 1,000 concurrent jobs per account
- **Job Duration**: Maximum 3 hours per job (fair-share enforcement)
- **Modules**: 100 modules per account
- **Runbooks**: No hard limit, but 100+ may impact performance

**Design Considerations**:
- If hitting 30-account limit: Consolidate by using runbook parameters for environment differentiation
- For long-running jobs (>3 hours): Use checkpoints in PowerShell Workflow runbooks or break into smaller jobs
- For massive parallelism (>1,000 jobs): Stagger job execution or use multiple accounts

### 3. Access Control and RBAC

**Role-Based Access Control Best Practices**:

```
Production Automation Account Roles:

Owner
├── Automation administrators only
└── Can create/delete Run As accounts, manage access

Contributor
├── DevOps engineers
└── Can create/modify runbooks, jobs, assets
└── Cannot change access permissions

Automation Operator
├── Operations team
└── Can start/stop runbooks
└── Cannot modify runbooks or assets

Reader
├── Auditors, managers
└── View-only access to configurations and logs
└── Cannot execute or modify anything
```

**Custom Role Example** (Read-Only Runbook Executor):
```json
{
  "Name": "Automation Runbook Executor",
  "Description": "Can execute runbooks but cannot modify them",
  "Actions": [
    "Microsoft.Automation/automationAccounts/read",
    "Microsoft.Automation/automationAccounts/runbooks/read",
    "Microsoft.Automation/automationAccounts/jobs/write",
    "Microsoft.Automation/automationAccounts/jobs/read"
  ],
  "NotActions": [
    "Microsoft.Automation/automationAccounts/runbooks/write",
    "Microsoft.Automation/automationAccounts/runbooks/delete"
  ],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}/resourceGroups/rg-automation-prod"
  ]
}
```

### 4. Tagging Strategy

Apply consistent tags for management and cost tracking:

```powershell
# Tag Automation account
$tags = @{
    "Environment" = "Production"
    "CostCenter" = "IT-Operations"
    "Owner" = "devops-team@company.com"
    "Purpose" = "VM-Lifecycle-Management"
    "Criticality" = "High"
    "DataClassification" = "Internal"
}

Set-AzAutomationAccount `
    -ResourceGroupName "rg-automation-prod" `
    -Name "automation-account-prod" `
    -Tags $tags
```

**Benefits**:
- **Cost Allocation**: Track automation costs by department/project
- **Compliance**: Identify accounts handling sensitive data
- **Lifecycle Management**: Find development accounts for cleanup
- **Incident Response**: Quickly identify automation account owners

### 5. Monitoring and Alerting

**Configure Diagnostics**:

```powershell
# Enable diagnostic logs to Log Analytics
$workspaceId = "/subscriptions/{sub-id}/resourceGroups/rg-monitor/providers/Microsoft.OperationalInsights/workspaces/law-automation"
$accountId = "/subscriptions/{sub-id}/resourceGroups/rg-automation-prod/providers/Microsoft.Automation/automationAccounts/automation-account-prod"

Set-AzDiagnosticSetting `
    -ResourceId $accountId `
    -WorkspaceId $workspaceId `
    -Enabled $true `
    -Category "JobLogs", "JobStreams", "DscNodeStatus"
```

**Key Metrics to Monitor**:
- Job success/failure rates
- Job duration trends (detect performance degradation)
- Runbook execution frequency
- Hybrid Worker connectivity status
- Resource consumption (for capacity planning)

**Alerting Rules**:
```
Alert: Runbook Failure Rate > 10%
Action: Send email to DevOps team, create incident in ServiceNow

Alert: Job Duration > 2 hours
Action: Send Teams notification (approaching 3-hour limit)

Alert: Hybrid Worker Offline
Action: Page on-call engineer immediately
```

### 6. Disaster Recovery and Backup

**Backup Strategy**:

1. **Runbook Backup**:
   - Primary: Source control integration (GitHub/Azure DevOps)
   - Secondary: Export runbooks to Azure Blob Storage weekly

```powershell
# Export all runbooks to Blob Storage
$runbooks = Get-AzAutomationRunbook -ResourceGroupName "rg-automation-prod" -AutomationAccountName "automation-account-prod"
$storageAccount = "stautomationbackup"
$container = "runbooks"

foreach ($runbook in $runbooks) {
    $content = Export-AzAutomationRunbook -ResourceGroupName "rg-automation-prod" `
        -AutomationAccountName "automation-account-prod" `
        -Name $runbook.Name -Slot Published
    
    $blobName = "$($runbook.Name)_$(Get-Date -Format 'yyyyMMdd').ps1"
    Set-AzStorageBlobContent -File $content -Container $container `
        -Blob $blobName -Context (Get-AzStorageAccount -ResourceGroupName "rg-backup" -Name $storageAccount).Context
}
```

2. **Configuration Backup**:
   - Export Automation account configuration (assets, modules) monthly
   - Store in ARM template format for quick redeployment

3. **Geo-Redundancy**:
   - For critical automation, deploy identical account in paired region
   - Use Traffic Manager or manual failover process

## Cost Optimization

### Pricing Model

**Azure Automation Pricing** (as of 2026):
- **Basic Tier**: Included with Azure subscription
- **Job Runtime**:
  - First 500 minutes/month: Free
  - Additional minutes: $0.002 per minute
- **Watchers**: $0.002 per hour
- **Update Management**: Free for Azure VMs, charged for non-Azure machines
- **DSC Node Management**: $6 per node per month

**Example Cost Calculation**:
```
Monthly Usage:
- 50 runbooks running 10 times/day
- Average runtime: 5 minutes per job
- Monthly jobs: 50 × 10 × 30 = 15,000 jobs
- Monthly runtime: 15,000 × 5 = 75,000 minutes

Cost Calculation:
- First 500 minutes: Free
- Remaining: 74,500 minutes × $0.002 = $149.00/month
```

### Cost Optimization Tips

1. **Optimize Runbook Efficiency**:
   - Use `--no-wait` for Azure CLI commands where results aren't needed immediately
   - Batch operations instead of looping individual commands
   - Cache data to reduce API calls

2. **Schedule Strategically**:
   - Run non-critical automation during off-peak hours
   - Consolidate frequent small jobs into less frequent larger jobs
   - Use on-demand execution via webhooks instead of polling schedules

3. **Clean Up Unused Resources**:
   ```powershell
   # Find automation accounts with no recent job execution
   $accounts = Get-AzAutomationAccount
   foreach ($account in $accounts) {
       $lastJob = Get-AzAutomationJob -ResourceGroupName $account.ResourceGroupName `
           -AutomationAccountName $account.AutomationAccountName `
           -StartTime (Get-Date).AddDays(-90) | Select-Object -First 1
       
       if (!$lastJob) {
           Write-Warning "Account $($account.AutomationAccountName) has no jobs in last 90 days"
       }
   }
   ```

## Verification and Testing

After creating Automation account, verify setup:

### Verification Checklist

**1. Account Accessibility**:
```powershell
# Verify you can access the account
Get-AzAutomationAccount -ResourceGroupName "rg-automation-prod" -Name "automation-account-prod"
```

**2. Run As Account Configuration**:
```powershell
# Verify Run As connection exists
Get-AzAutomationConnection -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "AzureRunAsConnection"

# Verify certificate exists and is valid
Get-AzAutomationCertificate -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "AzureRunAsCertificate" | Select-Object Name, ExpiryTime
```

**3. Test Authentication**:
```powershell
# Create simple test runbook
$runbookContent = @'
$connection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzAccount -ServicePrincipal -Tenant $connection.TenantID `
    -ApplicationId $connection.ApplicationID -CertificateThumbprint $connection.CertificateThumbprint

# Test access
Get-AzSubscription | Select-Object Name, Id
'@

# Import and test
Import-AzAutomationRunbook -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Path "Test-Authentication.ps1" -Type PowerShell -Published

$job = Start-AzAutomationRunbook -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" -Name "Test-Authentication"

# Wait and check output
Wait-Job -Id $job.JobId
Get-AzAutomationJobOutput -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" -Id $job.JobId -Stream Output
```

## Troubleshooting Common Issues

### Issue 1: Cannot Create Run As Account

**Symptom**: Warning message "You don't have permissions to create an Azure Run As account"

**Root Cause**: User lacks subscription Owner role

**Solutions**:
1. Request Owner permissions from subscription administrator
2. Have someone with Owner create the account
3. Use managed identity instead of Run As account
4. Manually create service principal and configure connection

### Issue 2: Automation Account Creation Fails

**Symptom**: Deployment fails with "The subscription is not registered to use namespace 'Microsoft.Automation'"

**Root Cause**: Automation resource provider not registered in subscription

**Solution**:
```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Automation

# Verify registration
Get-AzResourceProvider -ProviderNamespace Microsoft.Automation | Select-Object ProviderNamespace, RegistrationState
```

### Issue 3: Run As Certificate Expired

**Symptom**: Runbooks fail with authentication error after 1 year

**Root Cause**: Run As certificate expired (valid for 12 months)

**Solution**:
```powershell
# Renew Run As certificate
# Azure Portal: Automation Account → Run as accounts → Azure Run As Account → Renew certificate

# Or recreate entirely:
Remove-AzAutomationConnection -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" -Name "AzureRunAsConnection" -Force

# Recreate through Portal or use New-AzAutomationConnection with new certificate
```

## Next Steps

Now that you understand how to create and configure Automation accounts, proceed to **Unit 3: What is a Runbook?** to learn about the different types of runbooks you can create and when to use each type.

**Quick Start Actions**:
1. Create a development Automation account in your subscription
2. Verify Run As account configuration
3. Configure diagnostic logging to Log Analytics
4. Apply consistent tags for governance

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/2-create-automation-accounts
