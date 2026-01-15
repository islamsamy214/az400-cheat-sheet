# Module Assessment - Knowledge Check

## Overview

This knowledge check assesses your understanding of Azure Resource Manager templates, covering template structure, dependencies, modularization, deployment modes, and secrets management. Review the questions carefully and consider the explanations for each answer to reinforce your learning.

---

## Question 1: Template Components

**What is the minimum set of required sections in a valid ARM template?**

A) `$schema`, `contentVersion`, and `parameters`

B) `$schema`, `contentVersion`, and `resources`

C) `$schema`, `resources`, and `outputs`

D) `$schema`, `parameters`, `variables`, and `resources`

### Answer: B) `$schema`, `contentVersion`, and `resources`

**Explanation**:

A valid ARM template requires only three sections:
- **$schema**: Specifies the template schema version and enables validation
- **contentVersion**: Your version number for tracking template changes
- **resources**: Defines the Azure resources to deploy (can be empty array `[]`)

All other sections are optional:
- **parameters**: Optional, provides customization at deployment time
- **variables**: Optional, stores calculated or reused values
- **functions**: Optional, custom user-defined functions
- **outputs**: Optional, returns values after deployment

**Minimum valid template**:
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": []
}
```

**Why other answers are incorrect**:

**A)** Parameters are optional, not required.

**C)** Outputs are optional; contentVersion is required but missing in this option.

**D)** Parameters and variables are both optional, making this more than the minimum requirement.

---

## Question 2: Managing Dependencies

**You are deploying a virtual machine that requires a network interface, which in turn requires a subnet and public IP address. What is the BEST approach to define these dependencies?**

A) Use explicit `dependsOn` for all relationships: VM depends on NIC, NIC depends on subnet and public IP, subnet depends on virtual network

B) Use only `dependsOn` for the VM to depend on the virtual network; Resource Manager will handle the rest automatically

C) Use `dependsOn` for the VM to depend on the NIC, and use `resourceId()` references in the NIC properties for subnet and public IP (implicit dependencies)

D) Don't specify any dependencies; deploy resources in alphabetical order by name

### Answer: C) Use `dependsOn` for the VM to depend on the NIC, and use `resourceId()` references in the NIC properties for subnet and public IP (implicit dependencies)

**Explanation**:

The best practice combines explicit and implicit dependencies:

**Explicit dependency** (VM → NIC):
```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "dependsOn": [
    "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
  ]
}
```

The VM must wait for the NIC to exist before deployment.

**Implicit dependencies** (NIC → Subnet, NIC → Public IP):
```json
{
  "type": "Microsoft.Network/networkInterfaces",
  "properties": {
    "ipConfigurations": [
      {
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

Using `resourceId()` in the NIC properties creates implicit dependencies—Resource Manager automatically knows the NIC requires the subnet and public IP to exist first.

**Why other answers are incorrect**:

**A)** While functionally correct, this is overly explicit. Using `resourceId()` in properties creates implicit dependencies automatically, making redundant `dependsOn` entries unnecessary and cluttering the template.

**B)** Too simplistic—the VM depending directly on the virtual network doesn't establish the necessary dependency on the NIC. The VM requires the NIC, not the virtual network.

**D)** Resource names have no impact on deployment order. Without dependencies, Resource Manager will attempt to deploy all resources in parallel, which will fail when the VM tries to reference a non-existent NIC.

**Best Practice**: Prefer implicit dependencies (through `resourceId()` in properties) when accessing resource properties, and use explicit `dependsOn` only for non-property ordering requirements.

---

## Question 3: Linked vs. Nested Templates

**Your organization has a networking module used across 15 projects. The module defines virtual networks, subnets, NSGs, and route tables (300 lines of JSON). What is the BEST approach for managing this module?**

A) Copy the networking JSON into each project's main template as a nested template

B) Store the networking module as a separate JSON file in Azure Blob Storage and reference it as a linked template

C) Store the networking module in each project's Git repository and reference it as a linked template using relative file paths

D) Duplicate the networking resources in each project's main template without using nested or linked templates

### Answer: B) Store the networking module as a separate JSON file in Azure Blob Storage and reference it as a linked template

**Explanation**:

Linked templates are ideal for reusable modules used across multiple projects:

**Centralized Storage**:
```json
{
  "type": "Microsoft.Resources/deployments",
  "name": "networkingDeployment",
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "https://companytemplatesstorage.blob.core.windows.net/templates/networking.json",
      "contentVersion": "1.0.0.0"
    },
    "parameters": {
      "environment": {
        "value": "[parameters('environment')]"
      }
    }
  }
}
```

**Benefits**:
- **Single source of truth**: One networking module stored centrally
- **Version control**: Update module once, all projects can reference the new version
- **Security**: Secure with SAS tokens for private storage
- **Team ownership**: Network team maintains module independently
- **Independent versioning**: Module has its own contentVersion
- **Testing**: Module tested independently before use in projects

**Why other answers are incorrect**:

**A)** Nested templates embed content directly in the parent template. With 300 lines of JSON, this makes each project's main template large (300+ lines added per project). When the networking module needs updates, you must modify all 15 project templates individually—losing the benefit of modularization.

**C)** Storing the module in each project's repository defeats the purpose of reusability. You now have 15 copies of the networking module that must be updated individually. Relative file paths also don't work with ARM template deployments—templates must be accessible via HTTPS URI.

**D)** Duplicating resources is the worst approach: 15 copies to maintain, no version control, high risk of configuration drift between projects, and massive maintenance burden when networking standards change.

**Best Practice**: Use linked templates stored in centralized Azure Blob Storage for reusable modules shared across multiple projects.

---

## Question 4: Deployment Modes

**You manage a production resource group containing a web app, storage account, and SQL database. You need to deploy a new version of the web app configuration via ARM template. The template only defines the web app resource. What deployment mode should you use to ensure the storage account and SQL database are NOT deleted?**

A) Complete mode—it updates only the resources defined in the template

B) Incremental mode—it updates matching resources and leaves others unchanged

C) Validate mode—it tests the deployment without making changes

D) Either Complete or Incremental mode will work—both protect existing resources

### Answer: B) Incremental mode—it updates matching resources and leaves others unchanged

**Explanation**:

**Incremental Mode** (default and safe):
```bash
az deployment group create \
  --mode Incremental \
  --template-file webapp-update.json \
  --resource-group production-rg
```

**Behavior**:
- **Resources in template**: Updated to match template definition (web app updated)
- **Resources NOT in template**: **Left unchanged** (storage account and SQL database remain)
- **New resources in template**: Created

This is the safe choice when your template doesn't define the complete infrastructure.

**Complete Mode** (dangerous in this scenario):
```bash
az deployment group create \
  --mode Complete \
  --template-file webapp-update.json \
  --resource-group production-rg
```

**Behavior**:
- **Resources in template**: Updated to match template definition (web app updated)
- **Resources NOT in template**: **DELETED** (storage account and SQL database DELETED ⚠️)
- **New resources in template**: Created

This ensures the resource group matches the template exactly, but will delete the storage account and SQL database because they're not defined in your web-app-only template.

**Why other answers are incorrect**:

**A)** Complete mode is incorrect and dangerous. It would delete the storage account and SQL database because they're not defined in the template. Complete mode ensures the resource group matches the template exactly—removing resources not in the template.

**C)** Validate mode doesn't deploy anything—it only tests template validity. While useful for pre-deployment validation, it doesn't update the web app as required.

**D)** This is false. Complete mode would delete resources not in the template, while Incremental mode would preserve them. The modes have fundamentally different behaviors regarding unmanaged resources.

**Best Practice**: 
- Use **Incremental mode** (default) for most deployments, especially when template doesn't define all resources in the group
- Use **Complete mode** only when you want the resource group to match the template exactly and are confident all required resources are defined
- Always test Complete mode deployments in non-production environments first

---

## Question 5: Key Vault Integration

**You need to deploy a SQL database with an administrator password stored in Azure Key Vault. Your ARM template has a `securestring` parameter for the password. What configuration is required for Resource Manager to retrieve the secret during deployment?**

A) The Key Vault must have `enabledForTemplateDeployment` set to `true`, and the deployment identity must have `get` secret permission

B) The Key Vault must have public network access enabled, and the template must include the Key Vault resource ID

C) The deployment user must be a Key Vault Administrator, and the secret must be in the same resource group as the deployment

D) The template must use the `reference()` function to retrieve the secret value from Key Vault

### Answer: A) The Key Vault must have `enabledForTemplateDeployment` set to `true`, and the deployment identity must have `get` secret permission

**Explanation**:

Two requirements for Key Vault integration with ARM templates:

**1. Key Vault Configuration**:
```bash
az keyvault create \
  --name mycompanykeyvault \
  --resource-group keyvault-rg \
  --enabled-for-template-deployment true
```

The `--enabled-for-template-deployment` flag grants Azure Resource Manager service principal permission to retrieve secrets during template deployments.

**2. Deployment Identity Permissions**:
```bash
# Grant user access (deploying via Azure CLI)
az keyvault set-policy \
  --name mycompanykeyvault \
  --upn user@company.com \
  --secret-permissions get

# Or grant service principal access (deploying via CI/CD)
az keyvault set-policy \
  --name mycompanykeyvault \
  --object-id <service-principal-object-id> \
  --secret-permissions get
```

The identity running the deployment (your user account or service principal) needs `get` permission to read secret values.

**Parameter File Reference**:
```json
{
  "parameters": {
    "sqlAdminPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/{sub}/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault"
        },
        "secretName": "sqlAdminPassword"
      }
    }
  }
}
```

**Why other answers are incorrect**:

**B)** Public network access is NOT required—Key Vault can use private endpoints and still work with ARM templates. The template doesn't need to "include" the Key Vault resource ID beyond the parameter file reference. This confuses template requirements with network requirements.

**C)** The deployment user does NOT need Key Vault Administrator role (excessive permissions). The minimum required permission is `get` secret access (via access policy or "Key Vault Secrets User" RBAC role). The Key Vault and deployment can be in different resource groups—there's no same-resource-group requirement.

**D)** The `reference()` function is used to retrieve runtime properties of deployed resources, NOT to retrieve Key Vault secrets. Key Vault integration happens automatically through parameter file references—the template itself doesn't directly interact with Key Vault.

**Template Usage** (template doesn't interact with Key Vault directly):
```json
{
  "parameters": {
    "sqlAdminPassword": {
      "type": "securestring"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Sql/servers",
      "properties": {
        "administratorLoginPassword": "[parameters('sqlAdminPassword')]"
      }
    }
  ]
}
```

The template simply receives the password as a secure parameter—it doesn't know the value comes from Key Vault.

**Best Practice**:
- Enable Key Vault for template deployment during creation
- Grant only `get` permission to deployment identities (least privilege)
- Use access policies or RBAC for permission management
- Reference secrets in parameter files, not templates
- Use `securestring` type for all password/secret parameters

---

## Question 6: Circular Dependencies

**Your template defines two virtual networks (vnet1 and vnet2) with bidirectional peering. The template fails with a circular dependency error. What is the BEST solution?**

A) Remove the `dependsOn` elements from both virtual networks

B) Deploy virtual networks first without peering, then add peering configuration in a second deployment or as separate child resources

C) Use nested templates to deploy each virtual network independently

D) Change the API version to resolve the circular dependency

### Answer: B) Deploy virtual networks first without peering, then add peering configuration in a second deployment or as separate child resources

**Explanation**:

Circular dependencies occur when resources depend on each other in a loop. Virtual network peering is a classic example:

**Problem** (circular dependency):
```json
{
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet1",
      "dependsOn": ["[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"],
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
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet2",
      "dependsOn": ["[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]"],
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
      }
    }
  ]
}
```

vnet1 can't deploy until vnet2 exists, and vnet2 can't deploy until vnet1 exists—deadlock.

**Solution** (separate resources):
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
      // No peering defined here
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "vnet2",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.1.0.0/16"]
        }
      }
      // No peering defined here
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
1. Both vnets deploy in parallel (no dependencies between them)
2. After both vnets exist, peering resources deploy
3. Peerings depend on both vnets but not on each other
4. No circular dependency

**Why other answers are incorrect**:

**A)** Removing `dependsOn` doesn't solve the problem. The circular dependency exists because each vnet's peering configuration references the other vnet. The issue is structural, not just about explicit dependencies. Without proper ordering, the template will still fail.

**C)** Nested templates don't solve circular dependencies—they just move the problem into a nested context. The circular reference between vnet1 and vnet2 still exists regardless of whether they're in the main template or nested templates.

**D)** API version has nothing to do with circular dependencies. Circular dependencies are logical structural problems in your template design, not API version compatibility issues. Changing API versions won't resolve the loop.

**Alternative Solution**: Use separate deployments:

**First deployment** (vnets only):
```bash
az deployment group create --template-file vnets.json
```

**Second deployment** (peering):
```bash
az deployment group create --template-file peering.json
```

**Best Practice**: When you encounter circular dependencies, evaluate whether all dependencies are truly necessary. Often the solution is to break resources into multiple deployments or use child resources that can depend on multiple parents without creating loops.

---

## Question 7: Template Best Practices

**Which of the following represents ARM template best practices? (Select TWO)**

A) Hardcode resource names to ensure consistency across deployments

B) Use parameters for environment-specific values and `securestring` for sensitive data

C) Store templates and parameter files in version control (Git)

D) Use Complete deployment mode for all production deployments to prevent drift

E) Reference the latest API version by using `"apiVersion": "latest"` in resource definitions

### Answers: B) and C)

**B) Use parameters for environment-specific values and `securestring` for sensitive data** ✅

**Explanation**:

Parameters enable template reusability:
```json
{
  "parameters": {
    "environment": {
      "type": "string",
      "allowedValues": ["dev", "staging", "production"]
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_B2s"
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password (not logged)"
      }
    }
  }
}
```

**Benefits**:
- Single template works across dev, staging, production
- `securestring` prevents passwords from appearing in logs
- Environment-specific sizing through parameters
- Secure secret handling

**C) Store templates and parameter files in version control (Git)** ✅

**Explanation**:

Infrastructure as code means treating templates as code:
```bash
git add templates/
git commit -m "Add SQL database ARM template"
git push origin main
```

**Benefits**:
- Track all infrastructure changes
- Code review for infrastructure modifications
- Rollback to previous versions when issues arise
- Audit trail of who changed what and when
- Collaboration through pull requests
- Branch strategies for environment promotions

**Why other answers are incorrect**:

**A)** Hardcoding resource names prevents template reusability and creates deployment conflicts. Use parameters or variables for names:

**Bad** (hardcoded):
```json
{
  "name": "production-webapp-001"
}
```

**Good** (parameterized):
```json
{
  "name": "[concat(parameters('environment'), '-webapp-', uniqueString(resourceGroup().id))]"
}
```

**D)** Complete mode isn't appropriate for ALL production deployments. While useful for preventing drift, it deletes resources not in the template. Use Complete mode only when:
- Template defines ALL resources in the resource group
- You want to ensure exact match between template and reality
- You've thoroughly tested in non-production

For most deployments, Incremental mode (default) is safer and more appropriate.

**E)** `"apiVersion": "latest"` is **not valid** in ARM templates. API versions must be explicit version strings:

**Invalid**:
```json
{
  "apiVersion": "latest"  // ❌ Not supported
}
```

**Valid**:
```json
{
  "apiVersion": "2023-03-01"  // ✅ Explicit version
}
```

**Why explicit versions**:
- Ensures consistent behavior across deployments
- API versions determine available properties
- Prevents breaking changes from automatic upgrades
- Documents which API version template requires

**Additional Best Practices**:
- Use variables to reduce duplication
- Document parameters with metadata descriptions
- Organize resources logically within template
- Use linked templates for reusable modules
- Validate templates before production deployment
- Implement naming conventions consistently

---

## Assessment Summary

This module covered comprehensive ARM template knowledge:

**Template Structure**: Required sections ($schema, contentVersion, resources) and optional sections (parameters, variables, functions, outputs)

**Dependency Management**: Explicit dependencies with `dependsOn`, implicit dependencies with `resourceId()` references, parallel deployment optimization

**Modularization**: Linked templates for reusable modules, nested templates for embedded logic, deployment modes (Validate, Incremental, Complete)

**Security**: Key Vault integration for secrets, `securestring` parameters, SAS token secured template storage

**Circular Dependencies**: Identifying and resolving resource dependency loops through restructuring

**Best Practices**: Parameters for flexibility, version control for templates, appropriate deployment modes, explicit API versions

**Score Interpretation**:
- **7/7**: Excellent understanding of ARM templates
- **5-6/7**: Good grasp of concepts, review areas where you missed questions
- **3-4/7**: Fair understanding, revisit module content and practice
- **0-2/7**: Review the entire module and practice with hands-on deployments

**Next Steps**: Apply this knowledge by creating ARM templates for your own infrastructure scenarios, starting with simple resources and progressing to complex multi-resource deployments.

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-azure-resources-using-azure-resource-manager-templates/7-knowledge-check)
