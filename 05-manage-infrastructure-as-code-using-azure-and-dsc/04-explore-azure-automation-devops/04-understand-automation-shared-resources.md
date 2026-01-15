# Understand Automation Shared Resources

## Overview

Automation shared resources are reusable assets that runbooks reference during execution. Instead of hardcoding values like credentials, connection strings, or configuration settings directly in runbook code, you store them centrally as shared resources. This approach promotes consistency, maintainability, security, and reusability across all your automation workflows.

Think of shared resources as a centralized configuration store for your automation infrastructure—change a password once in Credentials, and all runbooks using that credential automatically use the updated value.

This unit explores the eight categories of shared resources, their use cases, best practices for management, and how to leverage them effectively in your runbooks.

## What are Shared Resources?

**Shared resources** are assets stored in your Automation account that can be referenced by multiple runbooks. They provide:

- **Centralized Management**: Update once, affect all runbooks
- **Security**: Encrypt sensitive data (credentials, certificates)
- **Consistency**: Same connection parameters across all automation
- **Maintainability**: Change configuration without editing runbook code
- **Reusability**: Share common values across runbooks

### Benefits

**Without Shared Resources** (hardcoded values):
```powershell
# ❌ BAD: Hardcoded values
$username = "admin@company.com"
$password = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($username, $password)

$storageAccountName = "staprod001"
$storageAccountKey = "abc123xyz789..."

Connect-AzAccount -Credential $credential
```

**Problems**:
- Passwords visible in code (security risk)
- Every runbook has its own copy (inconsistency)
- Changing password requires editing every runbook
- No audit trail of value changes
- Cannot share across team members safely

**With Shared Resources** (best practice):
```powershell
# ✅ GOOD: Using shared resources
$credential = Get-AutomationPSCredential -Name "AzureAdminAccount"
$storageAccountName = Get-AutomationVariable -Name "StorageAccountName"
$storageAccountKey = Get-AutomationVariable -Name "StorageAccountKey"

Connect-AzAccount -Credential $credential
```

**Advantages**:
- Credentials encrypted at rest
- Single source of truth
- Update once, all runbooks updated
- Audit trail through Azure activity logs
- Secure sharing within team

## Eight Categories of Shared Resources

Azure Automation provides eight types of shared resources, each designed for specific use cases.

### 1. Schedules

**Purpose**: Define when runbooks should execute automatically.

**Types**:
- **One-time**: Execute once at specific date/time
- **Recurring**: Execute repeatedly on a schedule
- **Complex**: Custom schedules using cron expressions

**Use Cases**:
- Start VMs every weekday at 7:00 AM
- Stop VMs every night at 7:00 PM
- Run monthly compliance reports on the 1st of each month
- Execute backups every 6 hours
- Perform maintenance windows on weekends

**Creating Schedules**:

```powershell
# Create one-time schedule
New-AzAutomationSchedule `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "MaintenanceWindow-Jan15" `
    -StartTime "2026-01-15 02:00:00" `
    -OneTime `
    -TimeZone "Eastern Standard Time"

# Create daily recurring schedule
New-AzAutomationSchedule `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "DailyBackup" `
    -StartTime "2026-01-15 00:00:00" `
    -DayInterval 1 `
    -TimeZone "UTC"

# Create weekly schedule (weekdays only)
New-AzAutomationSchedule `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "WeekdayStartup" `
    -StartTime "2026-01-15 07:00:00" `
    -WeekInterval 1 `
    -DaysOfWeek Monday, Tuesday, Wednesday, Thursday, Friday `
    -TimeZone "Pacific Standard Time"

# Create monthly schedule (first day of month)
New-AzAutomationSchedule `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "MonthlyReport" `
    -StartTime "2026-02-01 08:00:00" `
    -MonthInterval 1 `
    -DaysOfMonth 1 `
    -TimeZone "UTC"
```

**Link Schedule to Runbook**:

```powershell
# Associate schedule with runbook
Register-AzAutomationScheduledRunbook `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -RunbookName "Start-ProductionVMs" `
    -ScheduleName "WeekdayStartup" `
    -Parameters @{ResourceGroupName="rg-prod-vms"; Environment="Production"}
```

**Cron Expressions** (for advanced scheduling):

```
# Every 15 minutes
0 */15 * * * *

# Every weekday at 9 AM
0 0 9 * * 1-5

# First and 15th of month at midnight
0 0 0 1,15 * *

# Every 6 hours
0 0 */6 * * *
```

### 2. Modules

**Purpose**: PowerShell modules providing cmdlets for runbooks.

**Default Modules** (pre-installed):
- **Azure PowerShell modules**: Az.Accounts, Az.Compute, Az.Storage, etc.
- **Azure Automation modules**: Contains Get-AutomationVariable, Get-AutomationConnection
- **Orchestrator modules**: Activity cmdlets

**Custom Modules**:
- Import from PowerShell Gallery
- Upload custom .zip modules
- Organization-specific cmdlet libraries

**Use Cases**:
- Azure resource management (Az modules)
- Active Directory management (ActiveDirectory module)
- VMware automation (VMware.PowerCLI)
- Custom business logic modules
- Third-party service integration

**Importing Modules**:

```powershell
# Import from PowerShell Gallery
New-AzAutomationModule `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "Pester" `
    -ContentLinkUri "https://www.powershellgallery.com/api/v2/package/Pester"

# Update Azure module to latest version
Update-AzAutomationModule `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "Az.Compute"

# Check module status
Get-AzAutomationModule `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "Az.Storage" | Select-Object Name, Version, ProvisioningState
```

**Module Versions**:
- Automation accounts can have multiple module versions
- Runbooks use version available at execution time
- Update modules during maintenance windows (updates affect all runbooks)

**Best Practice**: Test module updates in dev Automation account before updating production.

### 3. Modules Gallery

**Purpose**: Centralized repository of community and Microsoft-published PowerShell modules.

**Access**: 
- Azure Portal: Automation Account → Modules gallery → Browse gallery
- Browse PowerShell Gallery: https://www.powershellgallery.com/

**Popular Modules**:
- **Az modules**: Azure resource management (Az.Compute, Az.Network, Az.Storage)
- **AzureAD**: Azure Active Directory management
- **ExchangeOnlineManagement**: Office 365 Exchange administration
- **Microsoft.Graph**: Microsoft Graph API integration
- **VMware.PowerCLI**: VMware vSphere automation
- **SqlServer**: SQL Server management
- **Pester**: PowerShell testing framework

**Importing from Gallery**:

1. Navigate to Modules gallery in Azure Portal
2. Search for module (e.g., "VMware.PowerCLI")
3. Click module → Click "Import"
4. Specify module version (or latest)
5. Click "OK" and wait for import (can take several minutes)

**Checking Gallery for Updates**:

```powershell
# List installed modules with versions
$modules = Get-AzAutomationModule -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod"

foreach ($module in $modules) {
    # Check PowerShell Gallery for latest version
    $galleryModule = Find-Module -Name $module.Name -Repository PSGallery -ErrorAction SilentlyContinue
    
    if ($galleryModule -and $galleryModule.Version -gt $module.Version) {
        Write-Output "Update available: $($module.Name) $($module.Version) → $($galleryModule.Version)"
    }
}
```

### 4. Python Packages

**Purpose**: Python libraries for Python runbooks.

**Supported Formats**:
- `.whl` files (wheel format)
- `.tar.gz` files (source distribution)

**Use Cases**:
- AWS automation (boto3)
- HTTP requests (requests library)
- Data processing (pandas, numpy)
- Azure SDK (azure-mgmt-* packages)
- Custom Python libraries

**Uploading Python Packages**:

```bash
# Download package locally first
pip download requests -d ./packages

# Upload via Azure Portal:
# Automation Account → Python packages → Add a Python package
# Select .whl file → Upload

# Verify import in runbook
python3 runbook:
import requests
response = requests.get('https://api.github.com')
print(response.status_code)
```

**Managing Python Package Dependencies**:

```bash
# Create requirements.txt
requests==2.31.0
pandas==2.0.3
azure-mgmt-compute==30.0.0

# Download all dependencies
pip download -r requirements.txt -d ./packages

# Upload each .whl file to Automation account
```

**Python 2 vs Python 3**:
- Python 2.7 support (legacy, use Python 3 for new runbooks)
- Python 3.8+ recommended (current standard)
- Packages specific to Python version

### 5. Credentials

**Purpose**: Securely store username/password combinations.

**Encryption**: Credentials encrypted at rest using Azure encryption.

**Use Cases**:
- Service account credentials for on-premises systems
- Database connection credentials
- API authentication (username/password)
- Domain administrator accounts
- Third-party service credentials

**Creating Credentials**:

```powershell
# Create credential asset
$username = "serviceaccount@company.com"
$password = ConvertTo-SecureString "ComplexP@ssw0rd!" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($username, $password)

New-AzAutomationCredential `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "ServiceAccountCredential" `
    -Value $credential `
    -Description "Service account for on-premises server management"
```

**Using Credentials in Runbooks**:

```powershell
# Retrieve credential
$credential = Get-AutomationPSCredential -Name "ServiceAccountCredential"

# Use with PowerShell remoting
Invoke-Command -ComputerName "server01.company.com" -Credential $credential -ScriptBlock {
    Get-Service | Where-Object {$_.Status -eq "Running"}
}

# Use with SQL connection
$sqlConnection = New-Object System.Data.SqlClient.SqlConnection
$sqlConnection.ConnectionString = "Server=sql01;Database=Production;User ID=$($credential.UserName);Password=$($credential.GetNetworkCredential().Password)"
$sqlConnection.Open()

# Use with REST API (Basic Auth)
$base64Cred = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("$($credential.UserName):$($credential.GetNetworkCredential().Password)"))
$headers = @{Authorization = "Basic $base64Cred"}
Invoke-RestMethod -Uri "https://api.example.com/data" -Headers $headers
```

**Best Practices**:
- ✅ Use separate credentials for different systems
- ✅ Rotate credentials regularly (update in Automation, not runbook code)
- ✅ Use descriptive names (e.g., "SQL-ServiceAccount-Prod")
- ❌ Don't use credentials for Azure authentication (use Run As account or Managed Identity)
- ❌ Don't log credential values (even to verbose stream)

### 6. Connections

**Purpose**: Store connection information for external services (API endpoints, server names, etc.).

**Common Connection Types**:
- **Azure**: Azure Run As connection (service principal)
- **Azure Classic Certificate**: Legacy Azure authentication
- **Azure Service Principal**: Custom service principal connection

**Use Cases**:
- Azure Run As account authentication
- Custom service principal connections
- Azure Stack environments
- Multiple Azure subscriptions

**Azure Run As Connection** (automatically created):

```powershell
# Retrieve Run As connection
$connection = Get-AutomationConnection -Name "AzureRunAsConnection"

# Connection contains:
# - ApplicationId: Service principal app ID
# - TenantId: Azure AD tenant ID
# - CertificateThumbprint: Certificate for authentication
# - SubscriptionId: Target subscription

# Authenticate using connection
Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $connection.TenantId `
    -ApplicationId $connection.ApplicationId `
    -CertificateThumbprint $connection.CertificateThumbprint

# Set subscription context
Set-AzContext -SubscriptionId $connection.SubscriptionId
```

**Custom Connection** (for multi-subscription scenarios):

```powershell
# Create custom service principal connection
$connectionFields = @{
    ApplicationId = "12345678-1234-1234-1234-123456789012"
    TenantId = "87654321-4321-4321-4321-210987654321"
    CertificateThumbprint = "ABC123DEF456..."
    SubscriptionId = "11111111-2222-3333-4444-555555555555"
}

New-AzAutomationConnection `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "AzureConnection-DevSubscription" `
    -ConnectionTypeName "Azure" `
    -ConnectionFieldValues $connectionFields `
    -Description "Connection to Development subscription"
```

**Using Custom Connection**:

```powershell
# Authenticate to different subscription
$devConnection = Get-AutomationConnection -Name "AzureConnection-DevSubscription"

Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $devConnection.TenantId `
    -ApplicationId $devConnection.ApplicationId `
    -CertificateThumbprint $devConnection.CertificateThumbprint

Set-AzContext -SubscriptionId $devConnection.SubscriptionId

# Now managing resources in dev subscription
Get-AzVM | Where-Object {$_.Tags["Environment"] -eq "Development"}
```

### 7. Certificates

**Purpose**: Store X.509 certificates for authentication and encryption.

**Certificate Formats**:
- **.cer**: Public key only (certificate)
- **.pfx**: Public and private key (personal information exchange)

**Use Cases**:
- Service principal authentication (Run As account certificate)
- SSL/TLS certificate management
- Code signing certificates
- Encrypted communication with external services
- Certificate-based API authentication

**Creating Certificate Asset**:

```powershell
# Export certificate to file
$cert = Get-ChildItem Cert:\CurrentUser\My\ABC123DEF456...
Export-PfxCertificate -Cert $cert -FilePath "C:\Temp\MyCert.pfx" -Password (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force)

# Import to Automation account
New-AzAutomationCertificate `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "ServicePrincipalCert" `
    -Path "C:\Temp\MyCert.pfx" `
    -Password (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) `
    -Exportable $false `
    -Description "Certificate for service principal authentication"
```

**Using Certificates in Runbooks**:

```powershell
# Retrieve certificate
$cert = Get-AutomationCertificate -Name "ServicePrincipalCert"

# Use for service principal authentication
Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $tenantId `
    -ApplicationId $appId `
    -Certificate $cert

# Use for REST API with certificate authentication
$response = Invoke-RestMethod `
    -Uri "https://api.example.com/secure" `
    -Certificate $cert `
    -Method Get
```

**Certificate Expiration Management**:

```powershell
# Check certificate expiration
$certs = Get-AzAutomationCertificate -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod"

$expiringCerts = $certs | Where-Object {
    $_.ExpiryTime -lt (Get-Date).AddDays(30)
}

if ($expiringCerts) {
    Write-Warning "Certificates expiring within 30 days:"
    $expiringCerts | Select-Object Name, ExpiryTime
    
    # Send alert
    Send-MailMessage -To "admin@company.com" `
        -Subject "Certificate Expiration Warning" `
        -Body "The following certificates are expiring soon: $($expiringCerts.Name -join ', ')"
}
```

### 8. Variables

**Purpose**: Store configuration values and settings.

**Variable Types**:
- **String**: Text values
- **Integer**: Numeric values
- **Boolean**: True/false flags
- **DateTime**: Date and time values
- **Encrypted**: Sensitive strings (stored encrypted)

**Use Cases**:
- Environment names (Production, Staging, Development)
- Resource group names
- API endpoints
- Configuration flags (EnableDebugMode, SendNotifications)
- Threshold values (MaxRetries, TimeoutSeconds)
- Sensitive configuration (API keys, connection strings)

**Creating Variables**:

```powershell
# String variable
New-AzAutomationVariable `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "EnvironmentName" `
    -Value "Production" `
    -Encrypted $false

# Integer variable
New-AzAutomationVariable `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "MaxRetries" `
    -Value 3 `
    -Encrypted $false

# Boolean variable
New-AzAutomationVariable `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "EnableDebugLogging" `
    -Value $false `
    -Encrypted $false

# Encrypted variable (for sensitive data)
New-AzAutomationVariable `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "ApiKey-ExternalService" `
    -Value "sk_live_abc123xyz789..." `
    -Encrypted $true `
    -Description "API key for external service integration"
```

**Using Variables in Runbooks**:

```powershell
# Retrieve variables
$environment = Get-AutomationVariable -Name "EnvironmentName"
$maxRetries = Get-AutomationVariable -Name "MaxRetries"
$debugEnabled = Get-AutomationVariable -Name "EnableDebugLogging"
$apiKey = Get-AutomationVariable -Name "ApiKey-ExternalService"

# Use in logic
$resourceGroupName = "rg-$environment-vms"

Write-Output "Targeting environment: $environment"
Write-Output "Resource group: $resourceGroupName"

# Conditional logic based on variable
if ($debugEnabled) {
    Write-Verbose "Debug mode enabled - verbose logging active" -Verbose
}

# Retry logic using variable
$attempt = 0
$success = $false

while ($attempt -lt $maxRetries -and !$success) {
    try {
        $attempt++
        Write-Output "Attempt $attempt of $maxRetries"
        
        # API call with encrypted API key
        $headers = @{Authorization = "Bearer $apiKey"}
        $response = Invoke-RestMethod -Uri "https://api.example.com/data" -Headers $headers
        
        $success = $true
        Write-Output "API call successful"
    }
    catch {
        Write-Warning "Attempt $attempt failed: $_"
        Start-Sleep -Seconds 5
    }
}
```

**Updating Variables** (without editing runbooks):

```powershell
# Update variable value
Set-AzAutomationVariable `
    -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" `
    -Name "MaxRetries" `
    -Value 5 `
    -Encrypted $false

# All runbooks now use new value (no code changes required)
```

**Variable Naming Conventions**:

```
Category-Purpose-Environment

Examples:
- ResourceGroup-Production-EastUS
- ApiKey-SendGrid-Prod
- Threshold-CPUAlert-Percent
- Feature-EnableAutoScale-Flag
- Connection-SqlServer-Primary
```

## Best Practices for Shared Resources

### 1. Naming Conventions

Use consistent, descriptive names:

```
Resource Type: Schedule
Pattern: <Frequency>-<Action>-<Target>
Examples:
- Daily-Backup-ProductionVMs
- Hourly-Monitor-WebApps
- Weekly-Cleanup-TempResources

Resource Type: Credential
Pattern: <System>-<AccountType>-<Environment>
Examples:
- ActiveDirectory-ServiceAccount-Prod
- SQL-AdminAccount-Dev
- VMware-AutomationAccount-Prod

Resource Type: Variable
Pattern: <Category>-<Name>-<Environment>
Examples:
- ResourceGroup-WebApps-Production
- ApiKey-SendGrid-Prod
- Threshold-DiskSpace-Warning
```

### 2. Security and Encryption

**Always Encrypt Sensitive Data**:

```powershell
# ❌ BAD: Unencrypted sensitive data
New-AzAutomationVariable -Name "DatabasePassword" -Value "P@ssw0rd123" -Encrypted $false

# ✅ GOOD: Encrypted sensitive data
New-AzAutomationVariable -Name "DatabasePassword" -Value "P@ssw0rd123" -Encrypted $true
```

**What to Encrypt**:
- ✅ Passwords and credentials
- ✅ API keys and tokens
- ✅ Connection strings with credentials
- ✅ Encryption keys
- ✅ Private certificates

**What NOT to Encrypt** (performance overhead):
- ❌ Public configuration (resource group names, regions)
- ❌ Boolean flags
- ❌ Public API endpoints
- ❌ Non-sensitive identifiers

### 3. Environment Separation

Use separate shared resources for each environment:

```
Development:
- Credential: SQL-AdminAccount-Dev
- Variable: ResourceGroup-WebApps-Dev (value: rg-dev-webapps)
- Connection: AzureConnection-DevSubscription

Staging:
- Credential: SQL-AdminAccount-Staging
- Variable: ResourceGroup-WebApps-Staging (value: rg-staging-webapps)
- Connection: AzureConnection-StagingSubscription

Production:
- Credential: SQL-AdminAccount-Prod
- Variable: ResourceGroup-WebApps-Prod (value: rg-prod-webapps)
- Connection: AzureConnection-ProdSubscription
```

**Parameterize Environment**:

```powershell
param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("Dev", "Staging", "Prod")]
    [string]$Environment
)

# Dynamic resource names based on environment
$credentialName = "SQL-AdminAccount-$Environment"
$resourceGroupVar = "ResourceGroup-WebApps-$Environment"

$credential = Get-AutomationPSCredential -Name $credentialName
$resourceGroup = Get-AutomationVariable -Name $resourceGroupVar
```

### 4. Version Control and Documentation

**Document Shared Resources**:

```markdown
# Shared Resources Inventory

## Credentials
| Name | Purpose | Rotation Schedule | Owner |
|------|---------|-------------------|-------|
| SQL-AdminAccount-Prod | SQL Server admin access | Quarterly | DBA Team |
| AD-ServiceAccount-Prod | Domain operations | Bi-annually | Infrastructure |

## Variables
| Name | Type | Purpose | Example Value |
|------|------|---------|---------------|
| ResourceGroup-WebApps-Prod | String | Target RG for web apps | rg-prod-webapps |
| Threshold-CPUAlert-Percent | Integer | CPU alert threshold | 80 |
| EnableDebugLogging | Boolean | Debug mode flag | false |

## Schedules
| Name | Frequency | Time | Linked Runbooks |
|------|-----------|------|-----------------|
| Daily-Backup-VMs | Daily | 00:00 UTC | Backup-AllVMs |
| Weekly-Cleanup | Weekly (Sunday) | 02:00 UTC | Remove-TempResources |
```

**Track Changes**:

```powershell
# Script to document current shared resources
$account = Get-AzAutomationAccount -ResourceGroupName "rg-automation-prod" -Name "automation-account-prod"

# Export all variables to CSV
Get-AzAutomationVariable -ResourceGroupName $account.ResourceGroupName -AutomationAccountName $account.AutomationAccountName |
    Select-Object Name, Value, Encrypted, CreationTime, LastModifiedTime |
    Export-Csv "AutomationVariables_$(Get-Date -Format 'yyyyMMdd').csv" -NoTypeInformation

# Export credentials list (values not accessible, only names)
Get-AzAutomationCredential -ResourceGroupName $account.ResourceGroupName -AutomationAccountName $account.AutomationAccountName |
    Select-Object Name, Description, CreationTime, LastModifiedTime |
    Export-Csv "AutomationCredentials_$(Get-Date -Format 'yyyyMMdd').csv" -NoTypeInformation
```

### 5. Least Privilege Access

Grant minimum required permissions:

```
Automation Operator Role:
- Can start/stop runbooks
- Cannot view/modify shared resources

Automation Job Operator Role:
- Can manage jobs
- Cannot access credentials or variables

Custom Role (Read Shared Resources):
{
  "Actions": [
    "Microsoft.Automation/automationAccounts/variables/read",
    "Microsoft.Automation/automationAccounts/credentials/read",
    "Microsoft.Automation/automationAccounts/connections/read"
  ],
  "NotActions": [
    "Microsoft.Automation/automationAccounts/variables/write",
    "Microsoft.Automation/automationAccounts/credentials/write"
  ]
}
```

### 6. Maintenance and Lifecycle

**Regular Audits**:

```powershell
# Find unused variables (no recent access logs)
$variables = Get-AzAutomationVariable -ResourceGroupName "rg-automation-prod" -AutomationAccountName "automation-account-prod"

foreach ($var in $variables) {
    # Check if variable is referenced in any runbook
    $runbooks = Get-AzAutomationRunbook -ResourceGroupName "rg-automation-prod" -AutomationAccountName "automation-account-prod"
    $referenced = $false
    
    foreach ($runbook in $runbooks) {
        $content = Export-AzAutomationRunbook -ResourceGroupName "rg-automation-prod" `
            -AutomationAccountName "automation-account-prod" `
            -Name $runbook.Name -Slot Published -OutputFolder "C:\Temp" -Force
        
        $runbookContent = Get-Content "C:\Temp\$($runbook.Name).ps1" -Raw
        
        if ($runbookContent -match $var.Name) {
            $referenced = $true
            break
        }
    }
    
    if (!$referenced) {
        Write-Warning "Variable '$($var.Name)' is not referenced in any runbook"
    }
}
```

**Credential Rotation**:

```powershell
# Schedule credential rotation
workflow Rotate-ServiceAccountPassword {
    param(
        [string]$CredentialName,
        [string]$ADUserName
    )
    
    # Generate new complex password
    $newPassword = -join ((48..57) + (65..90) + (97..122) | Get-Random -Count 16 | ForEach-Object {[char]$_})
    $securePassword = ConvertTo-SecureString $newPassword -AsPlainText -Force
    
    # Update Active Directory password
    InlineScript {
        Import-Module ActiveDirectory
        Set-ADAccountPassword -Identity $using:ADUserName -NewPassword $using:securePassword -Reset
    }
    
    # Update Automation credential
    $credential = New-Object System.Management.Automation.PSCredential($ADUserName, $securePassword)
    
    Set-AzAutomationCredential `
        -ResourceGroupName "rg-automation-prod" `
        -AutomationAccountName "automation-account-prod" `
        -Name $CredentialName `
        -Value $credential
    
    Write-Output "Password rotated successfully for $CredentialName"
}
```

## Practical Example: Using Multiple Shared Resources

Complete runbook demonstrating shared resources:

```powershell
<#
.SYNOPSIS
    Comprehensive shared resource usage example

.DESCRIPTION
    Demonstrates using all shared resource types in a single runbook
#>

param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("Production", "Staging", "Development")]
    [string]$Environment
)

# 1. CREDENTIALS: Authenticate to Azure
$azureConnection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzAccount -ServicePrincipal `
    -Tenant $azureConnection.TenantId `
    -ApplicationId $azureConnection.ApplicationId `
    -CertificateThumbprint $azureConnection.CertificateThumbprint

# 2. VARIABLES: Get configuration
$resourceGroupName = Get-AutomationVariable -Name "ResourceGroup-VMs-$Environment"
$maxRetries = Get-AutomationVariable -Name "Config-MaxRetries"
$sendNotifications = Get-AutomationVariable -Name "Feature-SendNotifications"
$apiKey = Get-AutomationVariable -Name "ApiKey-MonitoringService"  # Encrypted

# 3. Get VMs in target resource group
$vms = Get-AzVM -ResourceGroupName $resourceGroupName

# 4. CREDENTIALS: Connect to on-premises server
$onPremCred = Get-AutomationPSCredential -Name "OnPrem-ServiceAccount-$Environment"

Invoke-Command -ComputerName "server01.company.com" -Credential $onPremCred -ScriptBlock {
    Write-Output "Connected to on-premises server"
    Get-Service | Where-Object {$_.Status -eq "Running"} | Select-Object -First 5
}

# 5. Process VMs with retry logic
foreach ($vm in $vms) {
    $attempt = 0
    $success = $false
    
    while ($attempt -lt $maxRetries -and !$success) {
        try {
            $attempt++
            Write-Output "Processing $($vm.Name) - Attempt $attempt"
            
            # Perform operation
            $vmStatus = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vm.Name -Status
            Write-Output "VM Status: $($vmStatus.Statuses[1].DisplayStatus)"
            
            $success = $true
        }
        catch {
            Write-Warning "Attempt $attempt failed: $_"
            if ($attempt -lt $maxRetries) {
                Start-Sleep -Seconds 5
            }
        }
    }
}

# 6. Send notification if enabled
if ($sendNotifications) {
    $headers = @{
        Authorization = "Bearer $apiKey"
        ContentType = "application/json"
    }
    
    $body = @{
        environment = $Environment
        vm_count = $vms.Count
        timestamp = (Get-Date).ToString("o")
    } | ConvertTo-Json
    
    try {
        Invoke-RestMethod -Uri "https://monitoring.example.com/api/events" `
            -Method Post -Headers $headers -Body $body
        Write-Output "Notification sent successfully"
    }
    catch {
        Write-Warning "Failed to send notification: $_"
    }
}

Write-Output "Runbook completed successfully"
```

## Next Steps

Now that you understand shared resources, proceed to **Unit 5: Explore Runbook Gallery** to learn how to discover and import pre-built runbooks from the Azure Automation community, accelerating your automation development.

**Quick Start Actions**:
1. Create a test variable in your Automation account
2. Create a test credential
3. Modify an existing runbook to use these shared resources
4. Test the runbook and observe how values are retrieved

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/4-understand-automation-shared-resources
