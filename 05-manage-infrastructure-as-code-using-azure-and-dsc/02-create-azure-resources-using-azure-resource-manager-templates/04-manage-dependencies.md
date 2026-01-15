# Manage Dependencies

## Overview

Azure Resource Manager templates define multiple interconnected resources that often depend on each other. A virtual machine requires a network interface, which requires a virtual network. A SQL database requires a SQL server. Understanding and managing these dependencies correctly ensures resources deploy in the proper order and your infrastructure functions correctly.

This unit explores dependency management in ARM templates, examining both explicit dependencies using `dependsOn` and implicit dependencies created by the `reference()` function. You'll learn how Azure Resource Manager analyzes dependencies, optimizes deployment parallelism, and handles complex dependency scenarios including circular dependencies.

## Why Dependencies Matter

### The Dependency Challenge

Azure resources frequently have interdependencies that must be resolved in a specific deployment order:

**Example Scenario**: Deploying a Virtual Machine
1. **Virtual Network** must exist first
2. **Subnet** created within the virtual network
3. **Public IP Address** can deploy independently
4. **Network Interface** requires the subnet and public IP
5. **Virtual Machine** requires the network interface

If you try to create the VM before its network interface exists, deployment fails.

### Without Dependency Management

**Imperative Script Problem**:
```bash
# Must execute in exact order
az network vnet create ...
az network subnet create ...  # Must wait for vnet
az network public-ip create ...
az network nic create ...  # Must wait for subnet and public IP
az vm create ...  # Must wait for NIC
```

**Issues**:
- Order-dependent: Wrong sequence causes failures
- No parallelization: Resources that could deploy simultaneously must wait
- Error-prone: Easy to miss dependency relationships
- Fragile: Adding resources requires careful ordering

### With ARM Template Dependency Management

**Declarative Approach**:
```json
{
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "myVNet",
      ...
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "name": "myNIC",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'myVNet')]"
      ],
      ...
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "myVM",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC')]"
      ],
      ...
    }
  ]
}
```

**Benefits**:
- **Resource Manager handles order**: You declare dependencies, Azure determines optimal deployment sequence
- **Parallel deployment**: Independent resources deploy simultaneously
- **Automatic validation**: Circular dependencies detected before deployment
- **Clear relationships**: Dependencies explicitly documented in template

## How Resource Manager Handles Dependencies

### Dependency Analysis Process

When you submit an ARM template, Resource Manager:

1. **Parses template**: Reads all resource definitions
2. **Builds dependency graph**: Creates map of which resources depend on others
3. **Validates graph**: Checks for circular dependencies and invalid references
4. **Creates deployment plan**: Determines optimal order and parallelization
5. **Executes deployment**: Deploys resources in waves based on dependencies

### Deployment Waves

Resource Manager groups resources into "waves" that can deploy in parallel:

**Wave 1** (no dependencies):
- Virtual network
- Public IP address
- Storage account

**Wave 2** (depends on Wave 1):
- Subnet (depends on virtual network)
- Network security group

**Wave 3** (depends on Wave 2):
- Network interface (depends on subnet and public IP)

**Wave 4** (depends on Wave 3):
- Virtual machine (depends on network interface)

**Result**: Maximum parallelism with correct ordering.

### Parallel Deployment Example

```json
{
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet1",
      ...
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet2",
      ...
    },
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "storage1",
      ...
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "name": "publicIP1",
      ...
    }
  ]
}
```

**Deployment Behavior**: All four resources deploy **simultaneously** because they have no dependencies.

**Time Savings**: 4 resources × 3 minutes each = 12 minutes sequential, but only 3 minutes with parallelization.

## The dependsOn Element

The `dependsOn` element explicitly declares that one resource requires another resource to be deployed first.

### Basic Syntax

```json
{
  "type": "Microsoft.ResourceType/resourceName",
  "name": "childResource",
  "dependsOn": [
    "[resourceId('Microsoft.ResourceType/resourceName', 'parentResource')]"
  ]
}
```

### Single Dependency Example

SQL Database depends on SQL Server:

```json
{
  "resources": [
    {
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2021-11-01",
      "name": "[variables('sqlServerName')]",
      "location": "[parameters('location')]",
      "properties": {
        "administratorLogin": "[parameters('sqlAdminUsername')]",
        "administratorLoginPassword": "[parameters('sqlAdminPassword')]"
      }
    },
    {
      "type": "Microsoft.Sql/servers/databases",
      "apiVersion": "2021-11-01",
      "name": "[concat(variables('sqlServerName'), '/', variables('databaseName'))]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Sql/servers', variables('sqlServerName'))]"
      ],
      "properties": {
        "collation": "SQL_Latin1_General_CP1_CI_AS",
        "maxSizeBytes": 2147483648,
        "sampleName": "AdventureWorksLT"
      }
    }
  ]
}
```

**How it works**:
1. SQL Server deploys first
2. Resource Manager waits for server deployment to complete
3. Database deploys after server exists

### Multiple Dependencies Example

Network interface depends on multiple resources:

```json
{
  "type": "Microsoft.Network/networkInterfaces",
  "apiVersion": "2023-04-01",
  "name": "[variables('nicName')]",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]",
    "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]",
    "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
  ],
  "properties": {
    "ipConfigurations": [
      {
        "name": "ipconfig1",
        "properties": {
          "subnet": {
            "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
          },
          "publicIPAddress": {
            "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
          }
        }
      }
    ],
    "networkSecurityGroup": {
      "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
    }
  }
}
```

**Deployment Behavior**:
- Virtual network, public IP, and NSG deploy in parallel (Wave 1)
- Network interface waits for all three to complete (Wave 2)
- Ensures NIC has valid references to all required resources

### Dependency Chain Example

Complete VM deployment dependency chain:

```json
{
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "myVNet",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        },
        "subnets": [
          {
            "name": "appSubnet",
            "properties": {
              "addressPrefix": "10.0.1.0/24"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "name": "myPublicIP",
      "location": "[parameters('location')]",
      "properties": {
        "publicIPAllocationMethod": "Dynamic"
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "name": "myNIC",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'myVNet')]",
        "[resourceId('Microsoft.Network/publicIPAddresses', 'myPublicIP')]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'myVNet', 'appSubnet')]"
              },
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', 'myPublicIP')]"
              }
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "myVM",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC')]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_B2s"
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC')]"
            }
          ]
        }
      }
    }
  ]
}
```

**Deployment Order**:
1. **Wave 1**: Virtual network and public IP (parallel)
2. **Wave 2**: Network interface (after Wave 1 completes)
3. **Wave 3**: Virtual machine (after Wave 2 completes)

### Using resourceId() Function

The `resourceId()` function constructs proper resource identifiers for dependencies:

**Syntax**:
```json
"[resourceId('resourceType', 'resourceName')]"
```

**Examples**:
```json
// Storage account
"[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"

// Virtual network
"[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]"

// Subnet (child resource)
"[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"

// SQL database (child resource)
"[resourceId('Microsoft.Sql/servers/databases', variables('sqlServerName'), variables('databaseName'))]"
```

**Why use resourceId()**:
- Generates correct Azure resource ID format
- Works across resource groups and subscriptions
- Enables IntelliSense in Visual Studio Code
- Validates resource type and name at deployment time

## Implicit Dependencies with reference()

The `reference()` function retrieves properties from deployed resources. When you use `reference()`, Resource Manager automatically creates an implicit dependency—you don't need `dependsOn`.

### How reference() Works

```json
{
  "type": "Microsoft.Network/networkInterfaces",
  "name": "myNIC",
  "properties": {
    "ipConfigurations": [
      {
        "name": "ipconfig1",
        "properties": {
          "subnet": {
            "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
          },
          "publicIPAddress": {
            "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
          }
        }
      }
    ]
  }
}
```

**Explicit dependency not needed**: Using `resourceId()` within properties automatically creates implicit dependency. Resource Manager knows the NIC requires the virtual network and public IP to exist.

### reference() Function Examples

**Get Public IP Address**:
```json
"outputs": {
  "publicIPAddress": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))).ipAddress]"
  }
}
```

**Implicit dependency**: Output can only execute after public IP resource deploys.

**Get Storage Account Primary Endpoint**:
```json
"outputs": {
  "blobEndpoint": {
    "type": "string",
    "value": "[reference(variables('storageAccountName')).primaryEndpoints.blob]"
  }
}
```

**Get SQL Server FQDN**:
```json
"outputs": {
  "sqlServerFQDN": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Sql/servers', variables('sqlServerName'))).fullyQualifiedDomainName]"
  }
}
```

### Using reference() in Resource Properties

You can use `reference()` within resource definitions to access properties from other resources:

```json
{
  "type": "Microsoft.Web/sites",
  "name": "[variables('webAppName')]",
  "properties": {
    "siteConfig": {
      "appSettings": [
        {
          "name": "STORAGE_CONNECTION_STRING",
          "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2023-01-01').keys[0].value)]"
        }
      ]
    }
  },
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
  ]
}
```

**Note**: Using `listKeys()` (which internally uses `reference()`) requires explicit `dependsOn` because it's inside a complex expression.

### When to Use dependsOn vs. reference()

**Use dependsOn when**:
- Resource depends on another but doesn't reference its properties
- Dependency is purely for ordering (e.g., extensions depend on VM)
- Using complex expressions where implicit dependency isn't detected

**Example**: VM extension depends on VM but doesn't reference properties:
```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(variables('vmName'), '/CustomScriptExtension')]",
  "dependsOn": [
    "[resourceId('Microsoft.Compute/virtualMachines', variables('vmName'))]"
  ]
}
```

**Use reference() (implicit) when**:
- Accessing resource properties within outputs
- Retrieving runtime values like IP addresses or FQDNs
- Building connection strings from deployed resources

**Example**: Output doesn't need dependsOn:
```json
"outputs": {
  "vmFQDN": {
    "type": "string",
    "value": "[reference(variables('publicIPName')).dnsSettings.fqdn]"
  }
}
```

## Circular Dependencies

Circular dependencies occur when resources depend on each other in a loop. Resource Manager detects these before deployment and returns an error.

### Example Circular Dependency

**Invalid Template**:
```json
{
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet1",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"
      ]
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet2",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]"
      ]
    }
  ]
}
```

**Error**: "Circular dependency detected: vnet1 → vnet2 → vnet1"

**Why it fails**: vnet1 can't deploy until vnet2 exists, but vnet2 can't deploy until vnet1 exists. Deadlock.

### Real-World Circular Dependency Scenario

**Problem**: Virtual network peering between two vnets

```json
{
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet1",
      "properties": {
        "virtualNetworkPeerings": [
          {
            "name": "vnet1-to-vnet2",
            "properties": {
              "remoteVirtualNetwork": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"
              }
            }
          }
        ]
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"
      ]
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet2",
      "properties": {
        "virtualNetworkPeerings": [
          {
            "name": "vnet2-to-vnet1",
            "properties": {
              "remoteVirtualNetwork": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]"
              }
            }
          }
        ]
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]"
      ]
    }
  ]
}
```

**Circular dependency**: Each vnet's peering references the other vnet.

### Resolving Circular Dependencies

**Solution 1: Remove Unnecessary Dependencies**

Deploy virtual networks first, then add peering configuration separately:

```json
{
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet1",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        }
      }
      // No peering here
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet2",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.1.0.0/16"]
        }
      }
      // No peering here
    },
    {
      "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
      "name": "vnet1/vnet1-to-vnet2",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]",
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"
      ],
      "properties": {
        "remoteVirtualNetwork": {
          "id": "[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"
        }
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
      "name": "vnet2/vnet2-to-vnet1",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]",
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"
      ],
      "properties": {
        "remoteVirtualNetwork": {
          "id": "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]"
        }
      }
    }
  ]
}
```

**How it works**:
1. Both vnets deploy in parallel (no dependencies)
2. After both exist, peering resources deploy
3. No circular dependency because peerings don't depend on each other

**Solution 2: Use Child Resources**

Define peerings as child resources:

```json
{
  "type": "Microsoft.Network/virtualNetworks",
  "name": "vnet1",
  "resources": [
    {
      "type": "virtualNetworkPeerings",
      "name": "vnet1-to-vnet2",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]",
        "[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"
      ]
    }
  ]
}
```

**Solution 3: Break into Multiple Deployments**

Use linked templates or separate deployments:

**First deployment** (creates vnets):
```bash
az deployment group create --template-file vnets.json
```

**Second deployment** (creates peering):
```bash
az deployment group create --template-file peering.json
```

### Identifying Circular Dependencies

**Error Message Example**:
```
Deployment failed with error:
  Code: CircularDependency
  Message: Circular dependency detected for resource 'vnet1':
  vnet1 -> vnet2 -> vnet1
```

**Troubleshooting Steps**:
1. Draw dependency graph on paper
2. Identify loops in the graph
3. Evaluate if all dependencies are necessary
4. Break unnecessary dependencies or use separate deployments

## Best Practices for Dependency Management

### 1. Let Resource Manager Handle Dependencies When Possible

**Prefer implicit dependencies** (using resourceId() in properties) over explicit dependsOn:

**Good**:
```json
{
  "type": "Microsoft.Network/networkInterfaces",
  "properties": {
    "subnet": {
      "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
    }
  }
}
```

**Also acceptable** (but redundant):
```json
{
  "type": "Microsoft.Network/networkInterfaces",
  "dependsOn": [
    "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]"
  ],
  "properties": {
    "subnet": {
      "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
    }
  }
}
```

### 2. Use dependsOn for Non-Property Dependencies

Some resources must deploy in order but don't reference each other's properties:

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "myVM/installApache",
  "dependsOn": [
    "[resourceId('Microsoft.Compute/virtualMachines', 'myVM')]"
  ]
}
```

**Why**: Extension installs on VM but doesn't reference VM properties.

### 3. Avoid Unnecessary Dependencies

Don't create dependencies unless required:

**Bad**:
```json
{
  "type": "Microsoft.Storage/storageAccounts",
  "dependsOn": [
    "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]"
  ]
}
```

**Problem**: Storage account has no relationship with virtual network. Dependency creates unnecessary serialization, slowing deployment.

### 4. Use Variables for Resource IDs

Store frequently-used resource IDs in variables:

```json
"variables": {
  "subnetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]",
  "nsgId": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
}
```

**Benefits**:
- Reduces duplication
- Easier to update if resource names change
- Clearer code

### 5. Document Complex Dependencies

Add comments to explain non-obvious dependencies:

```json
{
  "type": "Microsoft.Web/sites",
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    // Web app requires storage for diagnostic logs
  ]
}
```

## Summary

Dependency management is critical for successful ARM template deployments. Resource Manager handles the heavy lifting of analyzing dependencies, creating optimal deployment plans, and maximizing parallelization.

**Key Concepts**:

**Explicit Dependencies**: Use `dependsOn` array with `resourceId()` function to declare that one resource requires another.

**Implicit Dependencies**: Using `resourceId()` or `reference()` in resource properties automatically creates dependencies.

**Parallel Deployment**: Resource Manager deploys independent resources simultaneously for faster deployments.

**Circular Dependencies**: Resource Manager detects and rejects templates with circular dependency loops before deployment.

**Best Practices**:
- Prefer implicit dependencies when accessing resource properties
- Use explicit dependsOn for non-property ordering requirements
- Avoid unnecessary dependencies that prevent parallelization
- Break circular dependencies using child resources or separate deployments
- Use variables to store frequently-referenced resource IDs

**Next Unit**: Learn how to modularize templates using linked and nested templates for better organization and reusability.

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-azure-resources-using-azure-resource-manager-templates/4-manage-dependencies)
