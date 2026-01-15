# Modularize Templates

## Overview

As ARM templates grow in complexity, maintaining them as single monolithic JSON files becomes challenging. Modularization breaks large templates into smaller, focused, reusable components that are easier to develop, test, and maintain.

This unit explores ARM template modularization through linked templates (external files) and nested templates (embedded definitions). You'll learn different deployment modes (validate, incremental, complete), how to secure external templates with SAS tokens, and best practices for organizing template libraries.

## Why Modularize Templates?

### Problems with Monolithic Templates

**Single Large Template Issues**:
- **Complexity**: 2,000+ line JSON files are difficult to navigate and understand
- **Maintenance burden**: Changes to one component risk breaking others
- **Testing challenges**: Can't test individual components in isolation
- **No reusability**: Can't share networking configuration across projects
- **Team collaboration**: Merge conflicts when multiple people edit same file
- **Limited readability**: Hard to understand overall architecture

### Benefits of Modularization

**Reusability**: Write networking module once, use in 10 projects.

**Maintainability**: Update load balancer configuration in one file, all deployments benefit.

**Team Collaboration**: Network team owns networking module, storage team owns storage module, platform team composes them.

**Testing**: Test individual modules independently before integration.

**Separation of Concerns**: Each module focuses on specific infrastructure area.

**Version Control**: Tag and version modules independently.

**Documentation**: Smaller files easier to document and understand.

### Real-World Example

**Monolithic Approach** (single 2,500-line template):
```
main.json (2,500 lines)
├── Virtual network definitions
├── Load balancer configuration
├── Storage accounts
├── Virtual machines
├── Databases
└── Monitoring setup
```

**Modular Approach** (composed from smaller templates):
```
main.json (200 lines - orchestration)
├── linked: networking.json (300 lines)
├── linked: storage.json (150 lines)
├── linked: compute.json (400 lines)
├── linked: database.json (250 lines)
└── linked: monitoring.json (200 lines)
```

**Benefits**:
- Each module maintained by subject matter experts
- Modules tested independently
- Changes to storage don't risk breaking networking
- Networking module reused across 15 projects

## Linked Templates

Linked templates are external ARM templates referenced by URI from a parent template. They enable true modularization where each component is a separate file.

### Linked Template Architecture

**Main Template** (orchestrator):
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2021-04-01",
      "name": "networkingDeployment",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "https://mystorageaccount.blob.core.windows.net/templates/networking.json",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "vnetName": {
            "value": "[parameters('vnetName')]"
          },
          "addressPrefix": {
            "value": "10.0.0.0/16"
          }
        }
      }
    }
  ]
}
```

**Linked Template** (networking.json):
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vnetName": {
      "type": "string"
    },
    "addressPrefix": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2023-04-01",
      "name": "[parameters('vnetName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["[parameters('addressPrefix')]"]
        }
      }
    }
  ],
  "outputs": {
    "vnetId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName'))]"
    }
  }
}
```

### How Linked Templates Work

1. **Main template submitted** to Azure Resource Manager
2. **Resource Manager reads templateLink** and fetches external template from URI
3. **Parameters passed** from main template to linked template
4. **Linked template deploys** as separate deployment within same resource group
5. **Outputs returned** from linked template to main template
6. **Main template continues** with next resources or linked templates

### Microsoft.Resources/deployments Resource

This special resource type deploys other templates:

```json
{
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2021-04-01",
  "name": "storageDeployment",
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "[variables('storageTemplateUri')]",
      "contentVersion": "1.0.0.0"
    },
    "parameters": {
      "storageAccountName": {
        "value": "[variables('storageAccountName')]"
      },
      "location": {
        "value": "[parameters('location')]"
      }
    }
  }
}
```

**Key Properties**:

**type**: Always `Microsoft.Resources/deployments` for linked/nested templates.

**name**: Deployment name (visible in Azure Portal deployment history).

**mode**: Deployment mode (Incremental or Complete - explained later).

**templateLink**: Object containing URI to external template.

**uri**: Full HTTPS URL to template JSON file.

**contentVersion**: Expected version of linked template (validates correct version).

**parameters**: Values passed to linked template (similar to parameter file).

### Passing Parameters to Linked Templates

**Inline Parameter Passing**:
```json
"parameters": {
  "vnetName": {
    "value": "[parameters('vnetName')]"
  },
  "location": {
    "value": "[resourceGroup().location]"
  },
  "tags": {
    "value": {
      "Environment": "[parameters('environment')]",
      "ManagedBy": "ARM Template"
    }
  }
}
```

**Parameter File Reference**:
```json
"parametersLink": {
  "uri": "https://mystorageaccount.blob.core.windows.net/parameters/networking-params.json",
  "contentVersion": "1.0.0.0"
}
```

**Parameter File Example** (networking-params.json):
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vnetName": {
      "value": "production-vnet"
    },
    "addressPrefix": {
      "value": "10.0.0.0/16"
    }
  }
}
```

### Using Linked Template Outputs

Linked templates can return outputs that the main template uses:

**Linked Template Output** (networking.json):
```json
"outputs": {
  "vnetId": {
    "type": "string",
    "value": "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName'))]"
  },
  "subnetId": {
    "type": "string",
    "value": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), 'default')]"
  }
}
```

**Main Template Using Output**:
```json
{
  "type": "Microsoft.Resources/deployments",
  "name": "computeDeployment",
  "dependsOn": [
    "networkingDeployment"
  ],
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "[variables('computeTemplateUri')]"
    },
    "parameters": {
      "subnetId": {
        "value": "[reference('networkingDeployment').outputs.subnetId.value]"
      }
    }
  }
}
```

**How it works**:
1. Networking template deploys, returns `subnetId` output
2. Main template retrieves output using `reference('deploymentName').outputs.outputName.value`
3. Output passed as parameter to compute template
4. Compute template uses subnet ID for VM network interface

### Complete Linked Template Example

**Main Template** (main.json):
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environment": {
      "type": "string",
      "allowedValues": ["dev", "staging", "production"]
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "templateBaseUri": "https://mystorageaccount.blob.core.windows.net/templates/",
    "networkingTemplateUri": "[concat(variables('templateBaseUri'), 'networking.json')]",
    "storageTemplateUri": "[concat(variables('templateBaseUri'), 'storage.json')]",
    "computeTemplateUri": "[concat(variables('templateBaseUri'), 'compute.json')]"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2021-04-01",
      "name": "networkingDeployment",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[variables('networkingTemplateUri')]",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "environment": {
            "value": "[parameters('environment')]"
          },
          "location": {
            "value": "[parameters('location')]"
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2021-04-01",
      "name": "storageDeployment",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[variables('storageTemplateUri')]",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "environment": {
            "value": "[parameters('environment')]"
          },
          "location": {
            "value": "[parameters('location')]"
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2021-04-01",
      "name": "computeDeployment",
      "dependsOn": [
        "networkingDeployment",
        "storageDeployment"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[variables('computeTemplateUri')]",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "environment": {
            "value": "[parameters('environment')]"
          },
          "location": {
            "value": "[parameters('location')]"
          },
          "subnetId": {
            "value": "[reference('networkingDeployment').outputs.subnetId.value]"
          }
        }
      }
    }
  ]
}
```

**Deployment**:
```bash
az deployment group create \
  --resource-group myResourceGroup \
  --template-file main.json \
  --parameters environment=production
```

## Nested Templates

Nested templates embed child template definitions directly within the parent template—no external files required.

### When to Use Nested vs. Linked

**Use Nested Templates When**:
- Template is small and specific to this deployment
- You want single-file deployment (no external dependencies)
- Template isn't reused across projects
- Simplifying CI/CD pipelines (fewer files to manage)

**Use Linked Templates When**:
- Template is reused across multiple projects
- Module maintained by different team
- Template is large and would make parent file unwieldy
- You want independent versioning of modules
- Template stored in shared template library

### Nested Template Syntax

```json
{
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2021-04-01",
  "name": "nestedDeployment",
  "properties": {
    "mode": "Incremental",
    "template": {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "apiVersion": "2023-01-01",
          "name": "nestedstorage",
          "location": "[resourceGroup().location]",
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "StorageV2"
        }
      ]
    }
  }
}
```

**Key Difference**: `template` property (object) instead of `templateLink` property (URI).

### Nested Template Example

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageNamePrefix": {
      "type": "string"
    }
  },
  "variables": {
    "storageName": "[concat(parameters('storageNamePrefix'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2021-04-01",
      "name": "storageDeployment",
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "storageAccountName": {
              "type": "string"
            }
          },
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2023-01-01",
              "name": "[parameters('storageAccountName')]",
              "location": "[resourceGroup().location]",
              "sku": {
                "name": "Standard_LRS"
              },
              "kind": "StorageV2",
              "properties": {
                "supportsHttpsTrafficOnly": true,
                "minimumTlsVersion": "TLS1_2"
              }
            }
          ],
          "outputs": {
            "storageAccountId": {
              "type": "string",
              "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]"
            }
          }
        },
        "parameters": {
          "storageAccountName": {
            "value": "[variables('storageName')]"
          }
        }
      }
    }
  ],
  "outputs": {
    "storageId": {
      "type": "string",
      "value": "[reference('storageDeployment').outputs.storageAccountId.value]"
    }
  }
}
```

### Expression Evaluation Scope

Nested templates have two evaluation scopes:

**Outer Scope** (default): Expressions evaluated in parent template context
```json
{
  "type": "Microsoft.Resources/deployments",
  "properties": {
    "expressionEvaluationOptions": {
      "scope": "outer"
    },
    "template": {
      "resources": [
        {
          "name": "[parameters('storageName')]"  // Uses parent's parameters
        }
      ]
    }
  }
}
```

**Inner Scope**: Expressions evaluated in nested template context
```json
{
  "type": "Microsoft.Resources/deployments",
  "properties": {
    "expressionEvaluationOptions": {
      "scope": "inner"
    },
    "template": {
      "parameters": {
        "storageName": {
          "type": "string"
        }
      },
      "resources": [
        {
          "name": "[parameters('storageName')]"  // Uses nested template's parameters
        }
      ]
    },
    "parameters": {
      "storageName": {
        "value": "[variables('storageAccountName')]"
      }
    }
  }
}
```

**Best Practice**: Use `"scope": "inner"` for clarity and isolation between parent and nested templates.

## Deployment Modes

Deployment mode determines how Resource Manager handles resources during template deployment.

### Validate Mode

Tests template without deploying resources.

```bash
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file main.json \
  --parameters @parameters.json
```

**What it checks**:
- Template JSON syntax is valid
- Resource types and API versions are correct
- Resource names follow Azure naming rules
- Required properties are present
- Parameter types match expectations

**What it doesn't check**:
- Whether resources will actually deploy successfully
- Quota or limit constraints
- Some service-specific validations

**Use Cases**:
- CI/CD pipeline validation before merge
- Pre-deployment verification
- Template development testing

### Incremental Mode (Default)

Adds or updates resources without affecting existing resources not defined in template.

```json
{
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "..."
    }
  }
}
```

**Behavior**:
- **Existing resources in template**: Updated to match template definition
- **Existing resources NOT in template**: Left unchanged
- **New resources in template**: Created

**Example Scenario**:

**Resource Group Contains**:
- Virtual machine "vm1"
- Storage account "storage1"
- Virtual network "vnet1"

**Template Defines**:
- Virtual machine "vm1" (with updated size)
- Storage account "storage2" (new)

**After Incremental Deployment**:
- Virtual machine "vm1" (updated size) ✅
- Storage account "storage1" (unchanged) ✅
- Storage account "storage2" (created) ✅
- Virtual network "vnet1" (unchanged) ✅

**Result**: 4 resources (1 updated, 1 created, 2 left alone).

**When to Use**:
- Development and testing environments
- Adding new resources to existing environment
- Updating specific resource configurations
- Most common deployment mode

### Complete Mode

Ensures resource group matches template exactly—deletes resources not defined in template.

```json
{
  "properties": {
    "mode": "Complete",
    "templateLink": {
      "uri": "..."
    }
  }
}
```

**Behavior**:
- **Existing resources in template**: Updated to match template definition
- **Existing resources NOT in template**: **DELETED**
- **New resources in template**: Created

**Example Scenario**:

**Resource Group Contains**:
- Virtual machine "vm1"
- Storage account "storage1"
- Virtual network "vnet1"

**Template Defines**:
- Virtual machine "vm1" (with updated size)
- Storage account "storage2" (new)

**After Complete Deployment**:
- Virtual machine "vm1" (updated size) ✅
- Storage account "storage2" (created) ✅
- Storage account "storage1" (DELETED) ⚠️
- Virtual network "vnet1" (DELETED) ⚠️

**Result**: 2 resources (1 updated, 1 created, 2 deleted).

**When to Use**:
- Production infrastructure should match template exactly
- Preventing configuration drift
- Compliance requirements (only approved resources)
- Disaster recovery scenarios (rebuild from template)

**⚠️ WARNING**: Complete mode deletes resources. Use carefully in production.

### Complete Mode Protection

Some resources are never deleted in Complete mode:

**Protected Resources**:
- Role assignments
- Locks
- Azure Blueprints assignments

**Resource Types Supporting Complete Mode Deletion**:
- Check Azure documentation for specific resource type behavior
- Most compute, networking, and storage resources support deletion
- Some managed services have special handling

### Deployment Mode Comparison

| Aspect | Incremental | Complete |
|--------|-------------|----------|
| **Default** | Yes | No |
| **Updates existing resources** | Yes | Yes |
| **Creates new resources** | Yes | Yes |
| **Deletes unmanaged resources** | No | Yes |
| **Risk level** | Low | High |
| **Use case** | Dev/test, incremental changes | Production, drift prevention |
| **Idempotency** | Partial | Full |

### Specifying Deployment Mode

**Azure CLI**:
```bash
# Incremental (default)
az deployment group create \
  --mode Incremental \
  --template-file main.json

# Complete
az deployment group create \
  --mode Complete \
  --template-file main.json
```

**PowerShell**:
```powershell
# Incremental (default)
New-AzResourceGroupDeployment `
  -Mode Incremental `
  -TemplateFile main.json

# Complete
New-AzResourceGroupDeployment `
  -Mode Complete `
  -TemplateFile main.json
```

**ARM Template (Linked/Nested)**:
```json
{
  "properties": {
    "mode": "Complete",  // or "Incremental"
    "templateLink": {
      "uri": "..."
    }
  }
}
```

## Storing and Securing External Templates

### Storage Options for Linked Templates

**Azure Blob Storage**:
- Most common approach
- Templates stored in Azure Storage Account
- Access controlled with SAS tokens or public access
- Version control through blob versioning

**GitHub/Git Repositories**:
- Raw GitHub URLs for public templates
- GitHub releases for versioned templates
- Good for open-source template libraries

**Azure DevOps Artifacts**:
- Templates as build artifacts
- Access controlled through Azure DevOps permissions
- Integrated with CI/CD pipelines

### Securing Templates with SAS Tokens

Shared Access Signature (SAS) tokens provide time-limited, permission-specific access to private storage accounts.

**Why Use SAS Tokens**:
- Templates contain sensitive information (naming conventions, architecture)
- Prevent unauthorized access to template library
- Time-limited access (tokens expire)
- Fine-grained permissions (read-only access)

### Creating SAS Token for Template Access

**Step 1: Upload Template to Private Storage Account**

```bash
# Create storage account
az storage account create \
  --name mytemplatestorage \
  --resource-group templates-rg \
  --location eastus \
  --sku Standard_LRS

# Create container
az storage container create \
  --name templates \
  --account-name mytemplatestorage \
  --auth-mode login

# Upload template
az storage blob upload \
  --container-name templates \
  --file networking.json \
  --name networking.json \
  --account-name mytemplatestorage \
  --auth-mode login
```

**Step 2: Generate SAS Token**

```bash
# Generate SAS token (valid for 1 hour)
END_DATE=$(date -u -d "1 hour" '+%Y-%m-%dT%H:%MZ')

az storage blob generate-sas \
  --container-name templates \
  --name networking.json \
  --account-name mytemplatestorage \
  --permissions r \
  --expiry $END_DATE \
  --https-only \
  --output tsv
```

**Output**: `sv=2021-06-08&se=2024-01-15T15%3A30Z&sr=b&sp=r&sig=...`

**Step 3: Construct Template URI with SAS Token**

```json
"variables": {
  "templateBaseUri": "https://mytemplatestorage.blob.core.windows.net/templates/",
  "sasToken": "[parameters('_artifactsLocationSasToken')]",
  "networkingTemplateUri": "[concat(variables('templateBaseUri'), 'networking.json', variables('sasToken'))]"
}
```

**Step 4: Pass SAS Token as Secure Parameter**

```json
"parameters": {
  "_artifactsLocationSasToken": {
    "type": "securestring",
    "defaultValue": "",
    "metadata": {
      "description": "SAS token for accessing private template storage (if required)"
    }
  }
}
```

**Deployment with SAS Token**:
```bash
SAS_TOKEN="?sv=2021-06-08&se=2024-01-15T15%3A30Z&sr=b&sp=r&sig=..."

az deployment group create \
  --resource-group myResourceGroup \
  --template-file main.json \
  --parameters _artifactsLocationSasToken="$SAS_TOKEN"
```

### PowerShell SAS Token Generation

```powershell
# Set expiry time
$sasExpiry = (Get-Date).AddHours(1).ToString("yyyy-MM-ddTHH:mmZ")

# Generate SAS token
$sasToken = New-AzStorageBlobSASToken `
  -Container "templates" `
  -Blob "networking.json" `
  -Context (Get-AzStorageAccount -ResourceGroupName templates-rg -Name mytemplatestorage).Context `
  -Permission r `
  -ExpiryTime $sasExpiry `
  -Protocol HttpsOnly

Write-Output $sasToken
```

### Best Practices for Template Storage

**1. Use Private Storage for Production Templates**
```bash
# Disable public blob access
az storage account update \
  --name mytemplatestorage \
  --resource-group templates-rg \
  --allow-blob-public-access false
```

**2. Organize Templates in Folder Structure**
```
templates/
├── networking/
│   ├── vnet.json
│   ├── nsg.json
│   └── load-balancer.json
├── compute/
│   ├── vm.json
│   └── vmss.json
└── storage/
    └── storage-account.json
```

**3. Use Blob Versioning for Template History**
```bash
az storage account blob-service-properties update \
  --account-name mytemplatestorage \
  --enable-versioning true
```

**4. Implement Access Policies**
```bash
# Grant specific users/service principals read access
az role assignment create \
  --role "Storage Blob Data Reader" \
  --assignee user@company.com \
  --scope /subscriptions/{sub}/resourceGroups/templates-rg/providers/Microsoft.Storage/storageAccounts/mytemplatestorage
```

## Summary

Template modularization transforms ARM template development from managing large monolithic files to composing flexible, reusable infrastructure modules.

**Key Concepts**:

**Linked Templates**: External template files referenced by URI, enabling true module reusability across projects.

**Nested Templates**: Embedded templates within parent template, useful for single-file deployments.

**Deployment Modes**:
- **Validate**: Test template without deploying
- **Incremental**: Add/update resources, leave others unchanged (default)
- **Complete**: Ensure resource group matches template exactly, delete unmanaged resources

**Template Storage**: Store linked templates in Azure Blob Storage, GitHub, or Azure DevOps with appropriate access controls.

**SAS Tokens**: Secure template access with time-limited, read-only permissions to private storage.

**Best Practices**:
- Use linked templates for reusable modules
- Use nested templates for deployment-specific logic
- Store templates in version-controlled storage
- Secure production templates with SAS tokens
- Use Incremental mode for dev/test, Complete mode for production drift prevention
- Organize templates in logical folder structures

**Next Unit**: Learn how to manage secrets in templates using Azure Key Vault integration for secure password and connection string handling.

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-azure-resources-using-azure-resource-manager-templates/5-modularize-templates)
