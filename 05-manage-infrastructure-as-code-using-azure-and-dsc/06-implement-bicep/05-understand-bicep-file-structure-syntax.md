# Understand Bicep File Structure and Syntax

Azure Bicep uses a clean, readable syntax. This unit covers the main concepts with practical examples.

## Sample Bicep File

```bicep
@minLength(3)
@maxLength(11)
param storagePrefix string

param storageSKU string = 'Standard_LRS'
param location string = resourceGroup().location

var uniqueStorageName = '${storagePrefix}${uniqueString(resourceGroup().id)}'

resource stg 'Microsoft.Storage/storageAccounts@2019-04-01' = {
    name: uniqueStorageName
    location: location
    sku: { name: storageSKU }
    kind: 'StorageV2'
    properties: { supportsHttpsTrafficOnly: true }

    resource service 'fileServices' = {
        name: 'default'
        resource share 'shares' = {
            name: 'exampleshare'
        }
    }
}

module webModule './webApp.bicep' = {
    name: 'webDeploy'
    params: {
        skuName: 'S1'
        location: location
    }
}

output storageEndpoint object = stg.properties.primaryEndpoints
```

## Scope

Defines deployment target: `resourceGroup` (default), `subscription`, `managementGroup`, or `tenant`.

## Parameters

Input values for reusability:

```bicep
@description('Storage account name')
@minLength(3)
@maxLength(24)
param storageAccountName string

@allowed(['Standard_LRS', 'Standard_GRS'])
param sku string = 'Standard_LRS'

@secure()
param adminPassword string
```

**Common decorators**: `@description`, `@minLength`, `@maxLength`, `@minValue`, `@maxValue`, `@allowed`, `@secure`

## Variables

Calculated values with global scope:

```bicep
var storageAccountName = '${prefix}${uniqueString(resourceGroup().id)}'
var tags = {
    environment: environment
    project: projectName
}
```

## Resources

Core infrastructure definitions:

```bicep
resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}
```

### Parent-Child Relationships

```bicep
resource parent 'Microsoft.Provider/type@version' = {
  name: 'parentName'
  
  resource child 'childType' = {
    name: 'childName'
  }
}
```

### Conditional Deployment

```bicep
resource conditionalResource 'type@version' = if (condition) {
  // resource definition
}
```

## Modules

Reusable Bicep files:

```bicep
module moduleName 'path/to/module.bicep' = {
  name: 'deploymentName'
  params: {
    parameterName: parameterValue
  }
}
```

Benefits: Code reuse, logical separation, easier testing.

## Outputs

Return values from deployment:

```bicep
output storageEndpoint string = storage.properties.primaryEndpoints.blob
output storageId string = storage.id
```

## Other Features

- **Loops**: `[for i in range(0, 5): { ... }]`
- **Conditional**: `condition ? trueValue : falseValue`
- **Multiline strings**: `'''...'''`
- **Existing resources**: `existing` keyword
- **Functions**: Same as ARM templates

---

**Module**: Implement Bicep  
**Unit**: 5 of 10  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/5-understand-bicep-file-structure-syntax
