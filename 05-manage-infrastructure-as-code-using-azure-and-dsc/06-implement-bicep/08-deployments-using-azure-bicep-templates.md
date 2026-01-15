# Deployments using Azure Bicep Templates

Hands-on lab for creating and deploying modularized Bicep templates.

## Lab Objectives

- Understand and create Azure Bicep templates
- Create reusable Bicep modules for storage resources
- Modify main template to use modules
- Deploy all resources to Azure using Bicep

## Lab Scenario

Create a Bicep template with modularized storage resources, demonstrating:
- Modular architecture
- Parameter passing
- Resource dependencies
- Deployment orchestration

## Lab Requirements

- Azure DevOps organization
- Azure subscription with Owner role
- Visual Studio Code with Bicep extension

## Lab Exercises

### Exercise 1: Author Bicep Templates

**Tasks**:
1. Create resource group
2. Author main Bicep template
3. Create storage module
4. Reference module in main template

**Example Module** (`modules/storage.bicep`):
```bicep
param storageAccountName string
param location string
param sku string = 'Standard_LRS'

resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: location
  sku: { name: sku }
  kind: 'StorageV2'
}

output storageId string = storage.id
```

**Main Template** (`main.bicep`):
```bicep
param location string = resourceGroup().location

module storage './modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageAccountName: 'store${uniqueString(resourceGroup().id)}'
    location: location
    sku: 'Standard_GRS'
  }
}
```

### Exercise 2: Deploy Templates

```bash
# Deploy
az deployment group create \
  --resource-group rg-lab \
  --template-file main.bicep

# Verify
az resource list --resource-group rg-lab --output table
```

### Exercise 3: Clean Up

```bash
az group delete --name rg-lab --yes --no-wait
```

## Key Learnings

- Bicep module creation and reuse
- Template structure and organization
- Deployment validation and monitoring
- Resource dependencies

For full lab instructions: https://go.microsoft.com/fwlink/\?linkid\=2270115

---

**Module**: Implement Bicep  
**Unit**: 8 of 10  
**Duration**: 60 minutes  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/8-azure-deployment-use-resource-manager-templates
