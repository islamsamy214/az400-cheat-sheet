# Explore Template Components

## Overview

Azure Resource Manager templates use JSON (JavaScript Object Notation) to define infrastructure declaratively. Understanding the structure and components of ARM templates is essential for creating effective infrastructure as code solutions.

This unit explores the anatomy of ARM templates, examining each component in detail—from the required schema and contentVersion through parameters, variables, functions, resources, and outputs. You'll learn how these components work together to create flexible, maintainable, and reusable infrastructure definitions.

## Understanding JSON Structure

JSON provides a structured, human-readable format for defining data. ARM templates leverage JSON's simplicity while providing powerful capabilities for infrastructure definition.

### JSON Basics

A JSON document consists of key-value pairs where keys are strings and values can be various types:

**String Values**:
```json
"name": "myStorageAccount"
"location": "eastus"
"description": "Production storage account"
```

**Number Values**:
```json
"count": 3
"port": 443
"maxValue": 100
```

**Boolean Values**:
```json
"enabled": true
"allowPublicAccess": false
```

**Array Values**:
```json
"locations": ["eastus", "westus", "centralus"]
"ports": [80, 443, 8080]
"allowedValues": ["Standard_B2s", "Standard_D2s_v3"]
```

**Object Values**:
```json
"sku": {
  "name": "Standard_LRS",
  "tier": "Standard"
}
"properties": {
  "supportsHttpsTrafficOnly": true,
  "minimumTlsVersion": "TLS1_2"
}
```

### Why JSON for Infrastructure?

**Machine-Readable**: Azure Resource Manager can parse and execute JSON templates programmatically.

**Language-Agnostic**: JSON works across all programming languages and platforms.

**Widely Supported**: Excellent tooling support in Visual Studio Code, Azure Portal, and CI/CD systems.

**Human-Readable**: Structure is clear and understandable, unlike binary formats.

**Validation**: JSON schema validation catches errors before deployment.

## ARM Template Structure

Every ARM template follows a standardized structure with specific sections:

### Basic Template Structure

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "functions": [],
  "resources": [],
  "outputs": {}
}
```

### Template Sections Explained

| Section | Purpose | Required |
|---------|---------|----------|
| `$schema` | Specifies template schema version and validates structure | Yes |
| `contentVersion` | Your version number for tracking template changes | Yes |
| `parameters` | Values customizable during deployment | No |
| `variables` | Values calculated or reused within template | No |
| `functions` | Custom user-defined functions | No |
| `resources` | Actual Azure resources to deploy | Yes |
| `outputs` | Values returned after deployment | No |

**Minimum Valid Template**:
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": []
}
```

## Schema and Content Version

### $schema Property

The `$schema` property defines which template schema version to use for validation:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#"
}
```

**Available Schema Versions**:

**Resource Group Deployments**:
- `2019-04-01/deploymentTemplate.json#` (current recommended)
- `2015-01-01/deploymentTemplate.json#` (legacy)

**Subscription-Level Deployments**:
- `2018-05-01/subscriptionDeploymentTemplate.json#`

**Management Group Deployments**:
- `2019-08-01/managementGroupDeploymentTemplate.json#`

**Tenant-Level Deployments**:
- `2019-08-01/tenantDeploymentTemplate.json#`

**Why It Matters**:
- Enables IntelliSense in Visual Studio Code
- Validates template structure before deployment
- Ensures compatibility with Azure Resource Manager
- Documents which features are available

### contentVersion Property

The `contentVersion` is your own version tracking system:

```json
{
  "contentVersion": "1.0.0.0"
}
```

**Version Format**: Typically follows semantic versioning (major.minor.patch.build).

**Common Practices**:
- Start with `"1.0.0.0"`
- Increment for breaking changes: `"2.0.0.0"`
- Increment minor for new features: `"1.1.0.0"`
- Increment patch for bug fixes: `"1.0.1.0"`

**Use Cases**:
- Track template evolution in version control
- Linked templates can require specific contentVersion
- Audit which template version deployed which resources
- Rollback to specific template versions

## Parameters Section

Parameters make templates reusable across environments by allowing customization at deployment time.

### Parameter Definition

```json
"parameters": {
  "parameterName": {
    "type": "string",
    "defaultValue": "default value",
    "allowedValues": ["value1", "value2"],
    "minValue": 1,
    "maxValue": 100,
    "minLength": 1,
    "maxLength": 50,
    "metadata": {
      "description": "Description of parameter"
    }
  }
}
```

### Parameter Types

**string**: Text values
```json
"adminUsername": {
  "type": "string",
  "metadata": {
    "description": "Administrator username for virtual machine"
  }
}
```

**securestring**: Sensitive text (passwords, API keys) - masked in logs
```json
"adminPassword": {
  "type": "securestring",
  "metadata": {
    "description": "Administrator password (not logged)"
  }
}
```

**int**: Integer numbers
```json
"vmCount": {
  "type": "int",
  "defaultValue": 2,
  "minValue": 1,
  "maxValue": 10,
  "metadata": {
    "description": "Number of VMs to deploy"
  }
}
```

**bool**: True or false values
```json
"enableBackup": {
  "type": "bool",
  "defaultValue": false,
  "metadata": {
    "description": "Enable Azure Backup for VMs"
  }
}
```

**array**: List of values
```json
"allowedIPAddresses": {
  "type": "array",
  "defaultValue": ["10.0.0.0/24", "10.1.0.0/24"],
  "metadata": {
    "description": "IP addresses allowed through firewall"
  }
}
```

**object**: Complex structured data
```json
"networkConfig": {
  "type": "object",
  "defaultValue": {
    "vnetAddressPrefix": "10.0.0.0/16",
    "subnetPrefix": "10.0.1.0/24"
  }
}
```

**secureObject**: Sensitive structured data (masked in logs)
```json
"databaseConfig": {
  "type": "secureObject",
  "metadata": {
    "description": "Database configuration including credentials"
  }
}
```

### Parameter Constraints

**allowedValues**: Restrict to specific choices
```json
"vmSize": {
  "type": "string",
  "defaultValue": "Standard_B2s",
  "allowedValues": [
    "Standard_B2s",
    "Standard_D2s_v3",
    "Standard_D4s_v3"
  ],
  "metadata": {
    "description": "Size of virtual machine"
  }
}
```

**minValue / maxValue**: Numeric range constraints
```json
"instanceCount": {
  "type": "int",
  "minValue": 1,
  "maxValue": 100,
  "defaultValue": 3
}
```

**minLength / maxLength**: String length constraints
```json
"storageAccountName": {
  "type": "string",
  "minLength": 3,
  "maxLength": 24,
  "metadata": {
    "description": "Storage account names must be 3-24 characters"
  }
}
```

### Parameter Best Practices

**1. Provide Metadata Descriptions**
```json
"location": {
  "type": "string",
  "defaultValue": "[resourceGroup().location]",
  "metadata": {
    "description": "Azure region for resource deployment. Defaults to resource group location."
  }
}
```

**2. Use Default Values for Optional Parameters**
```json
"enableMonitoring": {
  "type": "bool",
  "defaultValue": true,
  "metadata": {
    "description": "Enable Azure Monitor for resources"
  }
}
```

**3. Use allowedValues to Prevent Errors**
```json
"environment": {
  "type": "string",
  "allowedValues": ["dev", "staging", "production"],
  "metadata": {
    "description": "Environment name (dev, staging, or production)"
  }
}
```

**4. Use securestring for Sensitive Data**
```json
"sqlAdminPassword": {
  "type": "securestring",
  "minLength": 12,
  "metadata": {
    "description": "SQL Server administrator password (min 12 characters)"
  }
}
```

## Variables Section

Variables store calculated values or reduce duplication throughout the template.

### Variable Definition

```json
"variables": {
  "variableName": "value",
  "calculatedValue": "[expression]",
  "complexObject": {
    "property1": "value1",
    "property2": "value2"
  }
}
```

### When to Use Variables

**1. Reduce Repetition**
```json
"variables": {
  "nicName": "myVMNic",
  "publicIPName": "myPublicIP",
  "vnetName": "myVNet",
  "subnetName": "mySubnet"
}
```

Now reference `[variables('nicName')]` instead of repeating `"myVMNic"` everywhere.

**2. Complex Expressions**
```json
"variables": {
  "storageAccountName": "[concat('storage', uniqueString(resourceGroup().id))]",
  "vmName": "[concat(parameters('vmNamePrefix'), '-vm-', copyIndex())]",
  "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
}
```

**3. Naming Conventions**
```json
"variables": {
  "resourcePrefix": "[concat(parameters('environment'), '-', parameters('projectName'))]",
  "storageAccountName": "[concat(variables('resourcePrefix'), 'storage')]",
  "vnetName": "[concat(variables('resourcePrefix'), '-vnet')]",
  "nsgName": "[concat(variables('resourcePrefix'), '-nsg')]"
}
```

**4. Environment-Specific Configuration**
```json
"variables": {
  "vmSizeMap": {
    "dev": "Standard_B2s",
    "staging": "Standard_D2s_v3",
    "production": "Standard_D4s_v3"
  },
  "selectedVmSize": "[variables('vmSizeMap')[parameters('environment')]]"
}
```

### Variable Advantages

**Single Source of Truth**: Change value once, all references update automatically.

**Cleaner Resources Section**: Move complex expressions to variables for readability.

**Template Functions**: Use ARM functions like `concat()`, `uniqueString()`, `resourceGroup()` to build dynamic values.

### Example: Networking Configuration

```json
"variables": {
  "vnetName": "myVNet",
  "vnetAddressPrefix": "10.0.0.0/16",
  "subnetName": "appSubnet",
  "subnetPrefix": "10.0.1.0/24",
  "nsgName": "appNSG",
  "publicIPName": "myPublicIP",
  "nicName": "[concat(parameters('vmName'), '-nic')]",
  "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
}
```

**Benefits**:
- Change subnet name in one place
- `subnetRef` calculated once, used multiple times
- NIC name derived from VM name parameter
- Clear, maintainable structure

## Functions Section

Custom user-defined functions encapsulate complex logic for reuse throughout the template.

### Function Definition

```json
"functions": [
  {
    "namespace": "contoso",
    "members": {
      "uniqueName": {
        "parameters": [
          {
            "name": "namePrefix",
            "type": "string"
          }
        ],
        "output": {
          "type": "string",
          "value": "[concat(toLower(parameters('namePrefix')), uniqueString(resourceGroup().id))]"
        }
      }
    }
  }
]
```

### Using Custom Functions

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[contoso.uniqueName('mystorage')]",
    ...
  }
]
```

### When to Use Functions

**1. Complex Naming Logic**
```json
"functions": [
  {
    "namespace": "naming",
    "members": {
      "resourceName": {
        "parameters": [
          {
            "name": "resourceType",
            "type": "string"
          },
          {
            "name": "environment",
            "type": "string"
          }
        ],
        "output": {
          "type": "string",
          "value": "[concat(parameters('environment'), '-', parameters('resourceType'), '-', uniqueString(resourceGroup().id))]"
        }
      }
    }
  }
]
```

Usage:
```json
"storageAccountName": "[naming.resourceName('storage', parameters('environment'))]",
"vmName": "[naming.resourceName('vm', parameters('environment'))]"
```

**2. Reusable Calculations**
```json
"functions": [
  {
    "namespace": "math",
    "members": {
      "calculateSize": {
        "parameters": [
          {
            "name": "baseSize",
            "type": "int"
          },
          {
            "name": "multiplier",
            "type": "int"
          }
        ],
        "output": {
          "type": "int",
          "value": "[mul(parameters('baseSize'), parameters('multiplier'))]"
        }
      }
    }
  }
]
```

**3. Business Rules**
```json
"functions": [
  {
    "namespace": "policy",
    "members": {
      "isProduction": {
        "parameters": [
          {
            "name": "envName",
            "type": "string"
          }
        ],
        "output": {
          "type": "bool",
          "value": "[equals(toLower(parameters('envName')), 'production')]"
        }
      }
    }
  }
]
```

### Function Benefits

**Namespace Isolation**: Group functions under namespace (e.g., `contoso`, `naming`) to avoid conflicts.

**Reusability**: Call same function multiple times with different parameters.

**Consistency**: Ensure all resources follow same naming/calculation conventions.

**Maintainability**: Update logic once, affects all usages.

### Built-In vs. Custom Functions

**Built-In Functions** (use these most of the time):
- `concat()`: Concatenate strings
- `uniqueString()`: Generate unique hash
- `resourceId()`: Get resource ID
- `reference()`: Get resource properties
- `parameters()`: Access parameter values
- `variables()`: Access variable values

**Custom Functions** (use for organization-specific logic):
- Company naming standards
- Complex calculations specific to your needs
- Business rules unique to your organization

## Resources Section

The resources section is the heart of your template—where you define actual Azure resources to deploy.

### Resource Definition Structure

```json
"resources": [
  {
    "type": "Microsoft.ResourceProvider/resourceType",
    "apiVersion": "2023-01-01",
    "name": "resourceName",
    "location": "eastus",
    "tags": {},
    "dependsOn": [],
    "properties": {},
    "sku": {},
    "kind": ""
  }
]
```

### Resource Properties

**type**: Azure resource type (required)
```json
"type": "Microsoft.Storage/storageAccounts"
"type": "Microsoft.Compute/virtualMachines"
"type": "Microsoft.Network/virtualNetworks"
```

**apiVersion**: API version determines available properties (required)
```json
"apiVersion": "2023-01-01"  // Storage account API version
"apiVersion": "2023-03-01"  // VM API version
```

**Why API versions matter**: As Azure evolves, resource types gain new features. Using recent API versions allows access to latest capabilities.

**name**: Resource name following Azure naming rules (required)
```json
"name": "[variables('storageAccountName')]"
"name": "myvm-prod-001"
```

**location**: Azure region for deployment (required for most resources)
```json
"location": "[resourceGroup().location]"  // Same as resource group
"location": "[parameters('location')]"     // Parameter-based
"location": "eastus"                       // Hardcoded
```

**tags**: Metadata for organization and cost tracking
```json
"tags": {
  "Environment": "[parameters('environment')]",
  "ManagedBy": "ARM Template",
  "CostCenter": "Engineering",
  "Owner": "Platform Team"
}
```

**dependsOn**: Explicit resource dependencies
```json
"dependsOn": [
  "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]",
  "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
]
```

**properties**: Resource-specific configuration
```json
"properties": {
  "hardwareProfile": {
    "vmSize": "[parameters('vmSize')]"
  },
  "storageProfile": {
    "imageReference": {
      "publisher": "Canonical",
      "offer": "UbuntuServer",
      "sku": "18.04-LTS",
      "version": "latest"
    }
  }
}
```

### Example: Public IP Address Resource

```json
{
  "type": "Microsoft.Network/publicIPAddresses",
  "apiVersion": "2023-04-01",
  "name": "[variables('publicIPAddressName')]",
  "location": "[parameters('location')]",
  "properties": {
    "publicIPAllocationMethod": "Dynamic",
    "dnsSettings": {
      "domainNameLabel": "[parameters('dnsLabelPrefix')]"
    }
  },
  "tags": {
    "Environment": "[parameters('environment')]",
    "ManagedBy": "ARM Template"
  }
}
```

### Multiple Resources Example

```json
"resources": [
  {
    "type": "Microsoft.Network/virtualNetworks",
    "apiVersion": "2023-04-01",
    "name": "[variables('vnetName')]",
    "location": "[parameters('location')]",
    "properties": {
      "addressSpace": {
        "addressPrefixes": ["10.0.0.0/16"]
      },
      "subnets": [
        {
          "name": "[variables('subnetName')]",
          "properties": {
            "addressPrefix": "10.0.1.0/24"
          }
        }
      ]
    }
  },
  {
    "type": "Microsoft.Network/publicIPAddresses",
    "apiVersion": "2023-04-01",
    "name": "[variables('publicIPName')]",
    "location": "[parameters('location')]",
    "properties": {
      "publicIPAllocationMethod": "Static"
    }
  },
  {
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2023-03-01",
    "name": "[variables('vmName')]",
    "location": "[parameters('location')]",
    "dependsOn": [
      "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
    ],
    "properties": {
      "hardwareProfile": {
        "vmSize": "[parameters('vmSize')]"
      }
    }
  }
]
```

### Resource Copy (Loops)

Deploy multiple similar resources:

```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "name": "[concat(parameters('vmNamePrefix'), '-', copyIndex())]",
  "copy": {
    "name": "vmCopy",
    "count": "[parameters('vmCount')]"
  },
  "properties": {
    ...
  }
}
```

This creates multiple VMs: `webserver-0`, `webserver-1`, `webserver-2`, etc.

## Outputs Section

Outputs return information after deployment completes—useful for values unknown until resources are created.

### Output Definition

```json
"outputs": {
  "outputName": {
    "type": "string",
    "value": "[expression]"
  }
}
```

### Common Output Use Cases

**1. IP Addresses**
```json
"outputs": {
  "publicIPAddress": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))).ipAddress]"
  }
}
```

**2. FQDNs (Fully Qualified Domain Names)**
```json
"outputs": {
  "hostname": {
    "type": "string",
    "value": "[reference(variables('publicIPName')).dnsSettings.fqdn]"
  }
}
```

**3. Connection Strings**
```json
"outputs": {
  "sqlConnectionString": {
    "type": "string",
    "value": "[concat('Server=', reference(variables('sqlServerName')).fullyQualifiedDomainName, ';Database=', parameters('databaseName'))]"
  }
}
```

**4. Resource IDs**
```json
"outputs": {
  "vnetId": {
    "type": "string",
    "value": "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]"
  }
}
```

**5. Storage Account Keys**
```json
"outputs": {
  "storageAccountKey": {
    "type": "securestring",
    "value": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2023-01-01').keys[0].value]"
  }
}
```

### Using the reference() Function

The `reference()` function retrieves runtime state of a resource:

```json
"outputs": {
  "vmPrivateIP": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))).ipConfigurations[0].properties.privateIPAddress]"
  }
}
```

**How it works**:
1. Resource deploys and gets properties assigned (e.g., IP address)
2. `reference()` retrieves those properties
3. Output returns the value

### Accessing Outputs After Deployment

**Azure CLI**:
```bash
az deployment group show \
  --resource-group myResourceGroup \
  --name myDeployment \
  --query properties.outputs
```

**PowerShell**:
```powershell
(Get-AzResourceGroupDeployment -ResourceGroupName myResourceGroup -Name myDeployment).Outputs
```

**Azure DevOps Pipeline**:
```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentName: 'myDeployment'
  outputs:
    hostname: 'deploymentOutputs.hostname.value'
```

### Output Types

All parameter types are valid for outputs:
- `string`: Text values like FQDNs
- `securestring`: Sensitive values (still masked in logs)
- `int`: Numbers like port numbers
- `bool`: True/false values
- `array`: Lists of values
- `object`: Complex structured data

### Complete Example with Outputs

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string"
    }
  },
  "variables": {
    "publicIPName": "[concat(parameters('vmName'), '-pip')]",
    "nicName": "[concat(parameters('vmName'), '-nic')]"
  },
  "resources": [
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2023-04-01",
      "name": "[variables('publicIPName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "publicIPAllocationMethod": "Static",
        "dnsSettings": {
          "domainNameLabel": "[toLower(parameters('vmName'))]"
        }
      }
    }
  ],
  "outputs": {
    "publicIP": {
      "type": "string",
      "value": "[reference(variables('publicIPName')).ipAddress]"
    },
    "fqdn": {
      "type": "string",
      "value": "[reference(variables('publicIPName')).dnsSettings.fqdn]"
    },
    "resourceId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
    }
  }
}
```

**Deployment Output**:
```json
{
  "publicIP": {
    "type": "String",
    "value": "20.121.34.56"
  },
  "fqdn": {
    "type": "String",
    "value": "myvm.eastus.cloudapp.azure.com"
  },
  "resourceId": {
    "type": "String",
    "value": "/subscriptions/.../resourceGroups/myRG/providers/Microsoft.Network/publicIPAddresses/myvm-pip"
  }
}
```

## Summary

ARM template components work together to create flexible, maintainable infrastructure definitions:

**$schema & contentVersion**: Define template version and enable validation

**Parameters**: Enable reusability through customizable values at deployment time

**Variables**: Reduce duplication and store calculated values

**Functions**: Encapsulate custom logic for organization-specific requirements

**Resources**: Define actual Azure resources to deploy (the core of every template)

**Outputs**: Return deployment information for automation and integration

**Best Practices**:
- Use parameters for environment-specific values
- Use variables to reduce duplication
- Use functions for complex reusable logic
- Use outputs to pass information to other systems
- Document with metadata descriptions
- Validate templates before deployment

**Next Unit**: Learn how to manage resource dependencies using `dependsOn` and `reference()` to ensure resources deploy in the correct order.

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-azure-resources-using-azure-resource-manager-templates/3-explore-template-components)
