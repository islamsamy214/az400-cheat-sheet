# Exercise - Create Bicep Templates

This unit provides hands-on guidance for creating your first Bicep templates, from simple storage accounts to complex multi-resource deployments.

## Exercise 1: Create a Simple Storage Account

**Objective**: Write your first Bicep template to deploy an Azure storage account.

### Step 1: Create the Bicep File

Create `storage.bicep`:

```bicep
@description('Storage account name (3-24 chars, lowercase, numbers only)')
@minLength(3)
@maxLength(24)
param storageAccountName string

@description('Location for all resources')
param location string = resourceGroup().location

@description('Storage account SKU')
@allowed([
  'Standard_LRS'
  'Standard_GRS'
  'Standard_RAGRS'
  'Premium_LRS'
])
param storageSKU string = 'Standard_LRS'

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: storageSKU
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
  }
}

output storageAccountId string = storageAccount.id
output primaryEndpoints object = storageAccount.properties.primaryEndpoints
```

### Step 2: Deploy the Template

```bash
# Create resource group
az group create --name rg-bicep-demo --location eastus

# Deploy Bicep template
az deployment group create \
  --resource-group rg-bicep-demo \
  --template-file storage.bicep \
  --parameters storageAccountName=mystorageacct123

# View outputs
az deployment group show \
  --resource-group rg-bicep-demo \
  --name storage \
  --query properties.outputs
```

### Step 3: Verify Deployment

```bash
# List storage accounts
az storage account list --resource-group rg-bicep-demo --output table

# Check HTTPS-only setting
az storage account show \
  --name mystorageacct123 \
  --resource-group rg-bicep-demo \
  --query supportsHttpsTrafficOnly
```

**Key learning**: Basic Bicep syntax, parameters with decorators, resource declaration, outputs.

## Exercise 2: Web App with App Service Plan

**Objective**: Deploy a web app with automatic dependency management.

### Step 1: Create `webapp.bicep`

```bicep
param webAppName string
param location string = resourceGroup().location

@allowed(['F1', 'D1', 'B1', 'B2', 'B3', 'S1', 'S2', 'S3', 'P1', 'P2', 'P3'])
param skuName string = 'F1'

@allowed(['Windows', 'Linux'])
param osType string = 'Linux'

// App Service Plan
resource appServicePlan 'Microsoft.Web/serverfarms@2021-02-01' = {
  name: '${webAppName}-plan'
  location: location
  kind: osType == 'Linux' ? 'linux' : ''
  properties: {
    reserved: osType == 'Linux'
  }
  sku: {
    name: skuName
  }
}

// Web App - automatically depends on App Service Plan
resource webApp 'Microsoft.Web/sites@2021-02-01' = {
  name: webAppName
  location: location
  properties: {
    serverFarmId: appServicePlan.id  // Automatic dependency!
    httpsOnly: true
    siteConfig: {
      ftpsState: 'Disabled'
      minTlsVersion: '1.2'
      linuxFxVersion: osType == 'Linux' ? 'NODE|14-lts' : ''
    }
  }
}

output webAppUrl string = 'https://${webApp.properties.defaultHostName}'
output appServicePlanId string = appServicePlan.id
```

### Step 2: Deploy

```bash
az deployment group create \
  --resource-group rg-bicep-demo \
  --template-file webapp.bicep \
  --parameters webAppName=mywebapp-$RANDOM skuName=B1
```

**Key learning**: Automatic dependency inference (no `dependsOn` needed), conditional expressions, string interpolation.

## Exercise 3: Multi-Resource Environment with Variables

**Objective**: Deploy storage, Key Vault, and App Insights using variables.

### Step 1: Create `environment.bicep`

```bicep
param projectName string
param location string = resourceGroup().location
param environment string = 'dev'

// Variables for naming conventions
var storageAccountName = toLower('${projectName}${environment}sa${uniqueString(resourceGroup().id)}')
var keyVaultName = '${projectName}-${environment}-kv'
var appInsightsName = '${projectName}-${environment}-ai'
var logAnalyticsName = '${projectName}-${environment}-la'

// Log Analytics Workspace
resource logAnalytics 'Microsoft.OperationalInsights/workspaces@2021-06-01' = {
  name: logAnalyticsName
  location: location
  properties: {
    sku: {
      name: 'PerGB2018'
    }
    retentionInDays: 30
  }
}

// Application Insights
resource appInsights 'Microsoft.Insights/components@2020-02-02' = {
  name: appInsightsName
  location: location
  kind: 'web'
  properties: {
    Application_Type: 'web'
    WorkspaceResourceId: logAnalytics.id
  }
}

// Storage Account
resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}

// Key Vault
resource keyVault 'Microsoft.KeyVault/vaults@2021-06-01-preview' = {
  name: keyVaultName
  location: location
  properties: {
    sku: {
      family: 'A'
      name: 'standard'
    }
    tenantId: subscription().tenantId
    enableRbacAuthorization: true
  }
}

output storageAccountName string = storage.name
output keyVaultUri string = keyVault.properties.vaultUri
output appInsightsInstrumentationKey string = appInsights.properties.InstrumentationKey
```

### Step 2: Deploy

```bash
az deployment group create \
  --resource-group rg-bicep-demo \
  --template-file environment.bicep \
  --parameters projectName=contoso environment=dev
```

**Key learning**: Variables for naming, `uniqueString()` function, resource references, multiple resources.

## Exercise 4: Using Modules

**Objective**: Create reusable modules for common patterns.

### Step 1: Create Module `modules/storage.bicep`

```bash
mkdir modules
```

Create `modules/storage.bicep`:

```bicep
param storageAccountName string
param location string
param skuName string = 'Standard_LRS'

resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: location
  sku: { name: skuName }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
  }
}

output storageId string = storage.id
output storageName string = storage.name
```

### Step 2: Create Main Template Using Module

Create `main.bicep`:

```bicep
param location string = resourceGroup().location

module devStorage './modules/storage.bicep' = {
  name: 'devStorageDeployment'
  params: {
    storageAccountName: 'devstore${uniqueString(resourceGroup().id)}'
    location: location
    skuName: 'Standard_LRS'
  }
}

module prodStorage './modules/storage.bicep' = {
  name: 'prodStorageDeployment'
  params: {
    storageAccountName: 'prodstore${uniqueString(resourceGroup().id)}'
    location: location
    skuName: 'Standard_GRS'
  }
}

output devStorageId string = devStorage.outputs.storageId
output prodStorageId string = prodStorage.outputs.storageId
```

### Step 3: Deploy

```bash
az deployment group create \
  --resource-group rg-bicep-demo \
  --template-file main.bicep
```

**Key learning**: Module creation, module reuse, passing parameters, accessing module outputs.

## Exercise 5: Loops and Conditional Deployment

**Objective**: Deploy multiple resources using loops and conditions.

Create `advanced.bicep`:

```bicep
param storageCount int = 3
param deployKeyVault bool = true
param location string = resourceGroup().location

// Deploy multiple storage accounts using loop
resource storageAccounts 'Microsoft.Storage/storageAccounts@2021-04-01' = [for i in range(0, storageCount): {
  name: 'storage${i}${uniqueString(resourceGroup().id)}'
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}]

// Conditional deployment
resource keyVault 'Microsoft.KeyVault/vaults@2021-06-01-preview' = if (deployKeyVault) {
  name: 'kv-${uniqueString(resourceGroup().id)}'
  location: location
  properties: {
    sku: { family: 'A'; name: 'standard' }
    tenantId: subscription().tenantId
    enableRbacAuthorization: true
  }
}

output storageAccountIds array = [for i in range(0, storageCount): storageAccounts[i].id]
output keyVaultDeployed bool = deployKeyVault
```

Deploy:

```bash
az deployment group create \
  --resource-group rg-bicep-demo \
  --template-file advanced.bicep \
  --parameters storageCount=5 deployKeyVault=true
```

**Key learning**: Loops with `for`, conditional deployment with `if`, array outputs.

## Common Commands Reference

```bash
# Build Bicep to ARM template (inspect generated JSON)
az bicep build --file main.bicep

# Validate template without deploying
az deployment group validate \
  --resource-group rg-bicep-demo \
  --template-file main.bicep

# What-if analysis (preview changes)
az deployment group what-if \
  --resource-group rg-bicep-demo \
  --template-file main.bicep

# Deploy with parameter file
az deployment group create \
  --resource-group rg-bicep-demo \
  --template-file main.bicep \
  --parameters @parameters.json

# Clean up
az group delete --name rg-bicep-demo --yes --no-wait
```

## Best Practices Demonstrated

1. **Use decorators**: `@description`, `@minLength`, `@allowed` for validation
2. **Default values**: Provide sensible defaults for parameters
3. **Variables for naming**: Consistent naming conventions
4. **Modules for reuse**: DRY principle
5. **Outputs**: Return important values (IDs, URLs, keys)
6. **Security**: HTTPS-only, TLS 1.2, RBAC authorization
7. **Unique names**: Use `uniqueString()` for globally unique names

---

**Module**: Implement Bicep  
**Unit**: 4 of 10  
**Next**: [Unit 5: Understand Bicep File Structure and Syntax](#)  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/4-exercise-create-bicep-templates
