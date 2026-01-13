# Exercise - Set up service connections

⏱️ **Duration**: ~30 minutes | 🎯 **Type**: Hands-on Lab

## Overview

This exercise demonstrates how to create and configure Azure service connections in Azure DevOps to enable secure deployment operations to Azure resources. You'll configure service principal authentication and test the connection with Azure File Copy tasks.

## Learning Objectives

After completing this exercise, you'll be able to:
- ✅ Create Azure Resource Manager service connections
- ✅ Configure service principal authentication
- ✅ Add Azure File Copy tasks to pipelines
- ✅ Test deployments to Azure Storage
- ✅ Troubleshoot connection issues

## Prerequisites

| Requirement | Description |
|------------|-------------|
| **Azure Subscription** | Active Azure subscription with contributor access |
| **Azure DevOps Project** | Project with pipeline creation permissions |
| **Storage Account** | Azure Storage account for testing file copy |
| **Permissions** | Ability to create service principals in Azure AD |

---

## Exercise Steps

### Step 1: Create Azure Resource Manager Service Connection

#### 1.1 Access Service Connections

**Azure DevOps Navigation**:
```
Project Settings → Pipelines → Service connections → New service connection
```

**Steps**:
1. Navigate to your Azure DevOps project
2. Click **Project Settings** (bottom left)
3. Under **Pipelines**, select **Service connections**
4. Click **New service connection** button
5. Select **Azure Resource Manager** from the list
6. Click **Next**

#### 1.2 Choose Authentication Method

**Available Methods**:
- ✅ **Service principal (automatic)** - Recommended for most scenarios
- ✅ **Service principal (manual)** - When automatic doesn't work
- ✅ **Managed identity** - For Azure-hosted agents
- ✅ **Publish profile** - For App Service deployments

**Select**: Service principal (automatic)

#### 1.3 Configure Connection Details

**Configuration Form**:
```
Subscription: [Select your Azure subscription]
Resource group: [Optional - leave empty for subscription-level access]
Service connection name: AzureServiceConnection
Description: Service connection for Azure deployments
Security: ✓ Grant access permission to all pipelines (optional)
```

**Important Fields**:
| Field | Value | Notes |
|-------|-------|-------|
| **Subscription** | Your Azure subscription | Must have contributor access |
| **Resource group** | Optional | Leave empty for subscription-level or select specific RG |
| **Service connection name** | `AzureServiceConnection` | Use descriptive names |
| **Grant access** | ✓ Check for unrestricted use | Uncheck for manual pipeline authorization |

#### 1.4 Verify Connection

**Automatic Verification**:
```
Azure DevOps will:
1. Create a service principal in Azure AD
2. Assign contributor role to the service principal
3. Verify the connection can authenticate
4. Display success confirmation
```

**Success Indicators**:
- ✅ Green checkmark: "Connection verified successfully"
- ✅ Service principal name displayed (e.g., `your-project-name-subscription-id`)
- ✅ Connection appears in service connections list

---

### Step 2: Create Azure Storage Account (If Needed)

#### 2.1 Using Azure Portal

**PowerShell Command**:
```powershell
# Create resource group
New-AzResourceGroup -Name "rg-devops-lab" -Location "eastus"

# Create storage account
New-AzStorageAccount `
    -ResourceGroupName "rg-devops-lab" `
    -Name "stdevopslab$(Get-Random -Minimum 1000 -Maximum 9999)" `
    -Location "eastus" `
    -SkuName "Standard_LRS" `
    -Kind "StorageV2"
```

#### 2.2 Using Azure CLI

```bash
# Create resource group
az group create --name rg-devops-lab --location eastus

# Create storage account
STORAGE_NAME="stdevopslab$RANDOM"
az storage account create \
    --name $STORAGE_NAME \
    --resource-group rg-devops-lab \
    --location eastus \
    --sku Standard_LRS \
    --kind StorageV2

echo "Storage account name: $STORAGE_NAME"
```

#### 2.3 Create Blob Container

```bash
# Create container for file uploads
az storage container create \
    --name deployments \
    --account-name $STORAGE_NAME \
    --auth-mode login
```

---

### Step 3: Configure Pipeline with Azure File Copy Task

#### 3.1 Create Pipeline YAML

**File**: `azure-pipelines-filecopy.yml`

```yaml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  storageAccountName: 'stdevopslab1234'  # Replace with your storage account
  containerName: 'deployments'

stages:
- stage: Build
  displayName: 'Build Artifacts'
  jobs:
  - job: CreateArtifacts
    displayName: 'Create test artifacts'
    steps:
    - script: |
        mkdir -p $(Build.ArtifactStagingDirectory)/app
        echo "Build timestamp: $(date)" > $(Build.ArtifactStagingDirectory)/app/build-info.txt
        echo "Build ID: $(Build.BuildId)" >> $(Build.ArtifactStagingDirectory)/app/build-info.txt
        echo "Pipeline Name: $(Build.DefinitionName)" >> $(Build.ArtifactStagingDirectory)/app/build-info.txt
      displayName: 'Generate build artifacts'

    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'

- stage: Deploy
  displayName: 'Deploy to Azure Storage'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployToStorage
    displayName: 'Copy files to Azure Storage'
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: DownloadBuildArtifacts@1
            displayName: 'Download artifacts'
            inputs:
              buildType: 'current'
              downloadType: 'single'
              artifactName: 'drop'
              downloadPath: '$(System.ArtifactsDirectory)'

          - task: AzureFileCopy@4
            displayName: 'Copy files to Azure Blob Storage'
            inputs:
              SourcePath: '$(System.ArtifactsDirectory)/drop/app/*'
              azureSubscription: 'AzureServiceConnection'  # Your service connection name
              Destination: 'AzureBlob'
              storage: '$(storageAccountName)'
              ContainerName: '$(containerName)'
              BlobPrefix: 'builds/$(Build.BuildId)/'
              
          - task: AzurePowerShell@5
            displayName: 'Verify file upload'
            inputs:
              azureSubscription: 'AzureServiceConnection'
              ScriptType: 'InlineScript'
              Inline: |
                $ctx = New-AzStorageContext -StorageAccountName '$(storageAccountName)' -UseConnectedAccount
                $blobs = Get-AzStorageBlob -Container '$(containerName)' -Context $ctx -Prefix 'builds/$(Build.BuildId)/'
                
                Write-Host "Uploaded files:"
                $blobs | ForEach-Object {
                    Write-Host "  - $($_.Name) ($($_.Length) bytes)"
                }
                
                if ($blobs.Count -eq 0) {
                    Write-Error "No files found in storage!"
                    exit 1
                }
              azurePowerShellVersion: 'LatestVersion'
```

#### 3.2 Key Configuration Points

**AzureFileCopy Task Parameters**:
| Parameter | Value | Purpose |
|-----------|-------|---------|
| `SourcePath` | `$(System.ArtifactsDirectory)/drop/app/*` | Files to copy |
| `azureSubscription` | `AzureServiceConnection` | Service connection name |
| `Destination` | `AzureBlob` | Target type (Blob/VMs) |
| `storage` | Variable or hardcoded | Storage account name |
| `ContainerName` | `deployments` | Blob container |
| `BlobPrefix` | `builds/$(Build.BuildId)/` | Folder structure in blob |

---

### Step 4: Run and Validate Pipeline

#### 4.1 Commit and Run Pipeline

```bash
# Add pipeline file
git add azure-pipelines-filecopy.yml
git commit -m "Add Azure File Copy pipeline"
git push origin main
```

**Azure DevOps Steps**:
1. Navigate to **Pipelines** → **Pipelines**
2. Click **New pipeline**
3. Select **Azure Repos Git** (or your repo type)
4. Select your repository
5. Choose **Existing Azure Pipelines YAML file**
6. Select `azure-pipelines-filecopy.yml`
7. Click **Run**

#### 4.2 Monitor Pipeline Execution

**Expected Output**:

**Build Stage**:
```
Build Artifacts
└── Create test artifacts
    ✓ Generate build artifacts (5s)
    ✓ Publish artifacts (2s)
```

**Deploy Stage**:
```
Deploy to Azure Storage
└── Copy files to Azure Storage
    ✓ Download artifacts (3s)
    ✓ Copy files to Azure Blob Storage (8s)
    ✓ Verify file upload (4s)
    
    Uploaded files:
      - builds/123/build-info.txt (142 bytes)
```

#### 4.3 Verify in Azure Portal

**Navigation**: Azure Portal → Storage Accounts → [Your Storage Account] → Containers → deployments

**Verification Steps**:
1. Navigate to the storage account
2. Click **Containers** under Data storage
3. Open the `deployments` container
4. Navigate to `builds/[BuildId]/` folder
5. Verify `build-info.txt` exists
6. Download and inspect file contents

---

### Step 5: Troubleshooting Common Issues

#### Issue 1: "Service connection not found"

**Cause**: Pipeline doesn't have permission to use the service connection

**Solution**:
```
1. Go to Project Settings → Service connections
2. Click on your service connection
3. Click "..." (three dots) → Security
4. Add your pipeline to the authorized list
   OR
5. Check "Grant access permission to all pipelines"
```

#### Issue 2: "Insufficient permissions to deploy"

**Cause**: Service principal lacks permissions on target resource

**Solution**:
```bash
# Get service principal object ID from Azure DevOps service connection details
# Assign contributor role to storage account

az role assignment create \
    --assignee <service-principal-object-id> \
    --role "Storage Blob Data Contributor" \
    --scope /subscriptions/<subscription-id>/resourceGroups/rg-devops-lab/providers/Microsoft.Storage/storageAccounts/<storage-account-name>
```

#### Issue 3: "Storage account not found"

**Cause**: Incorrect storage account name or service principal lacks reader permission

**Solution**:
```yaml
# Verify storage account name in variables
variables:
  storageAccountName: 'stdevopslab1234'  # Must match actual name (no typos)

# Ensure service principal can read storage account metadata
# Assign "Reader" role at subscription or resource group level
```

#### Issue 4: "Container not found"

**Cause**: Blob container doesn't exist or naming mismatch

**Solution**:
```bash
# Create container if missing
az storage container create \
    --name deployments \
    --account-name <storage-account-name> \
    --auth-mode login

# Or let AzureFileCopy create it automatically
# Add parameter: CreateContainer: true
```

---

## Manual Service Principal Configuration (Alternative)

### When to Use Manual Configuration

Use manual service principal configuration when:
- ✅ Automatic creation fails due to Azure AD permissions
- ✅ You need to use an existing service principal
- ✅ Organization policies require manual app registration
- ✅ You need custom role assignments

### Step-by-Step Manual Configuration

#### 1. Create Service Principal in Azure

```bash
# Create service principal
SP_OUTPUT=$(az ad sp create-for-rbac \
    --name "sp-azuredevops-manual" \
    --role contributor \
    --scopes /subscriptions/<subscription-id> \
    --sdk-auth)

# Extract values (save securely!)
echo $SP_OUTPUT | jq -r '.clientId'        # Application (client) ID
echo $SP_OUTPUT | jq -r '.clientSecret'    # Client secret
echo $SP_OUTPUT | jq -r '.subscriptionId'  # Subscription ID
echo $SP_OUTPUT | jq -r '.tenantId'        # Tenant ID
```

**PowerShell Alternative**:
```powershell
# Create service principal
$sp = New-AzADServicePrincipal -DisplayName "sp-azuredevops-manual"
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($sp.Secret)
$secret = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)

# Display values
Write-Host "Application ID: $($sp.ApplicationId)"
Write-Host "Client Secret: $secret"
Write-Host "Tenant ID: $(Get-AzContext).Tenant.Id"
Write-Host "Subscription ID: $(Get-AzContext).Subscription.Id"

# Assign contributor role
New-AzRoleAssignment -ObjectId $sp.Id -RoleDefinitionName Contributor
```

#### 2. Configure in Azure DevOps

**Steps**:
1. New service connection → Azure Resource Manager
2. Choose "Service principal (manual)"
3. Enter the following:

**Form Fields**:
```
Environment: Azure Cloud
Scope Level: Subscription
Subscription ID: [From step 1]
Subscription Name: [Your subscription name]

Authentication:
Service Principal ID (Client ID): [From step 1]
Service Principal Key (Secret): [From step 1]
Tenant ID: [From step 1]

Service connection name: AzureServiceConnectionManual
Description: Manually configured service principal
✓ Grant access permission to all pipelines
```

4. Click **Verify** to test the connection
5. Click **Verify and save**

---

## Best Practices

### Security

| Practice | Description | Priority |
|----------|-------------|----------|
| **Least Privilege** | Assign only required roles (avoid Owner) | 🔴 Critical |
| **Scope Limitation** | Restrict to resource group instead of subscription | 🟠 High |
| **Secret Rotation** | Rotate service principal secrets regularly (90 days) | 🟠 High |
| **Connection Security** | Uncheck "Grant access to all pipelines" for sensitive connections | 🟡 Medium |
| **Audit Logging** | Enable Azure AD audit logs for service principal activity | 🟡 Medium |

### Performance

```yaml
# Use service connection caching
- task: AzureFileCopy@4
  inputs:
    # ... other inputs
    CleanTargetBeforeCopy: false  # Don't delete all files first
    enableCopyPrerequisites: false  # Skip prerequisite checks if not needed
```

### Naming Conventions

**Service Connection Names**:
```
Pattern: [Environment]-[ResourceType]-[Purpose]

Examples:
- Production-Azure-Deployments
- Staging-AKS-Application
- Dev-StorageAccount-BuildArtifacts
```

---

## Validation Checklist

- [ ] Service connection created successfully
- [ ] Connection verification passed
- [ ] Service principal has correct role assignments
- [ ] Pipeline can authenticate to Azure
- [ ] Files successfully copied to Azure Storage
- [ ] Verification script confirms file upload
- [ ] Pipeline runs without permission errors
- [ ] Storage account accessible from Azure Portal
- [ ] Blob container created and files visible
- [ ] Pipeline logs show successful deployment

---

## Key Takeaways

- 🎯 **Service connections** enable secure, reusable Azure authentication in pipelines
- 🔐 **Automatic service principal** creation is fastest but requires Azure AD permissions
- 🛠️ **Manual configuration** provides more control and works around permission limitations
- ✅ **Always verify** connections after creation to catch permission issues early
- 📊 **Scope appropriately** - use resource group level for production workloads
- 🔄 **Reuse connections** across multiple pipelines to centralize management
- 🚨 **Monitor usage** through Azure AD audit logs and pipeline run history

---

## Next Steps

✅ **Exercise Complete!** You've successfully:
- Created Azure service connections
- Configured service principal authentication
- Deployed files to Azure Storage using pipelines
- Verified deployments programmatically

**Continue to**: Unit 4 - Configure automated integration and functional test automation

---

## Additional Resources

- [Service connections in Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints)
- [Connect to Microsoft Azure](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure)
- [Azure File Copy task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/azure-file-copy)
- [Troubleshoot Azure Resource Manager service connections](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-rm-endpoint)

[↩️ Back to Module Overview](01-introduction.md) | [➡️ Next: Test Automation](04-configure-automated-test-automation.md)
