# Manage Secrets in Templates

## Overview

ARM templates often require sensitive data like passwords, connection strings, API keys, and certificates. Hardcoding these secrets in template files or parameter files creates security risks—secrets visible in version control, deployment logs, and accessible to unauthorized users.

Azure Key Vault integration with ARM templates solves this problem by centralizing secret management with encryption, access control, auditing, and rotation capabilities. This unit explores how to securely reference Key Vault secrets in ARM template parameters without exposing sensitive values.

## Why Use Key Vault with ARM Templates?

### Problems with Hardcoded Secrets

**Template with Hardcoded Password** (insecure):
```json
{
  "parameters": {
    "adminPassword": {
      "type": "string",
      "defaultValue": "P@ssw0rd123!"
    }
  }
}
```

**Security Issues**:
- **Visible in version control**: Git history contains password forever
- **Exposed in logs**: Deployment logs may contain parameter values
- **Accessible to developers**: Everyone with template access sees production passwords
- **No rotation capability**: Changing password requires updating all templates
- **Compliance violations**: Many regulations prohibit storing secrets in plain text
- **Audit trail gaps**: No record of who accessed passwords

### Key Vault Solution Benefits

**Centralized Management**: All secrets stored in one secure location.

**Encryption**: Secrets encrypted at rest and in transit using Azure's encryption infrastructure.

**Access Control**: RBAC and access policies control who can read secrets.

**Audit Trail**: Azure Monitor logs every secret access with user, time, and IP address.

**Secret Rotation**: Update secrets in Key Vault without changing templates.

**Compliance**: Meets regulatory requirements (HIPAA, PCI-DSS, SOC 2).

**Separation of Duties**: Security team manages Key Vault, infrastructure team deploys templates.

### Real-World Scenario

**Challenge**: Financial services company deploying SQL databases across 50 environments.

**Without Key Vault**:
- Database passwords in parameter files
- Parameter files in Git repository
- Security audit finds passwords accessible to 200+ employees
- Compliance violation, forced to rotate all passwords
- Manual update of 50 parameter files required

**With Key Vault**:
- Single SQL admin password stored in Key Vault
- Templates reference Key Vault secret
- Access restricted to 5 security team members
- Password rotation in Key Vault automatically flows to all environments
- Full audit trail of password access
- Compliance requirement satisfied

## Deploy Key Vault and Secret

### Create Key Vault

**Azure CLI**:
```bash
# Create resource group
az group create \
  --name keyvault-rg \
  --location eastus

# Create Key Vault
az keyvault create \
  --name mycompanykeyvault \
  --resource-group keyvault-rg \
  --location eastus \
  --enabled-for-template-deployment true
```

**Critical Parameter**: `--enabled-for-template-deployment true`

**What it does**: Grants Azure Resource Manager service principal permission to retrieve secrets during ARM template deployments.

**PowerShell**:
```powershell
# Create resource group
New-AzResourceGroup `
  -Name keyvault-rg `
  -Location eastus

# Create Key Vault
New-AzKeyVault `
  -VaultName mycompanykeyvault `
  -ResourceGroupName keyvault-rg `
  -Location eastus `
  -EnabledForTemplateDeployment
```

### ARM Template to Create Key Vault

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Key Vault"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "tenantId": {
      "type": "string",
      "defaultValue": "[subscription().tenantId]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "apiVersion": "2023-02-01",
      "name": "[parameters('keyVaultName')]",
      "location": "[parameters('location')]",
      "properties": {
        "tenantId": "[parameters('tenantId')]",
        "sku": {
          "family": "A",
          "name": "standard"
        },
        "enabledForTemplateDeployment": true,
        "enabledForDiskEncryption": false,
        "enabledForDeployment": false,
        "enableRbacAuthorization": false,
        "accessPolicies": []
      }
    }
  ],
  "outputs": {
    "keyVaultId": {
      "type": "string",
      "value": "[resourceId('Microsoft.KeyVault/vaults', parameters('keyVaultName'))]"
    }
  }
}
```

**Key Properties**:

**enabledForTemplateDeployment**: `true` allows ARM template deployments to retrieve secrets.

**enabledForDiskEncryption**: `true` if Azure Disk Encryption needs secrets.

**enabledForDeployment**: `true` if VMs need to retrieve certificates during provisioning.

**enableRbacAuthorization**: `true` uses Azure RBAC for access control (modern approach), `false` uses access policies (legacy).

### Add Secrets to Key Vault

**Azure CLI**:
```bash
# Add SQL admin password
az keyvault secret set \
  --vault-name mycompanykeyvault \
  --name sqlAdminPassword \
  --value 'MyC0mpl3xP@ssw0rd!'

# Add storage account connection string
az keyvault secret set \
  --vault-name mycompanykeyvault \
  --name storageConnectionString \
  --value 'DefaultEndpointsProtocol=https;AccountName=...'

# Add API key
az keyvault secret set \
  --vault-name mycompanykeyvault \
  --name thirdPartyApiKey \
  --value 'api_key_abc123xyz789'
```

**PowerShell**:
```powershell
# Add SQL admin password
$secretPassword = ConvertTo-SecureString 'MyC0mpl3xP@ssw0rd!' -AsPlainText -Force
Set-AzKeyVaultSecret `
  -VaultName mycompanykeyvault `
  -Name sqlAdminPassword `
  -SecretValue $secretPassword

# Add storage connection string
$secretConnString = ConvertTo-SecureString 'DefaultEndpointsProtocol=https;AccountName=...' -AsPlainText -Force
Set-AzKeyVaultSecret `
  -VaultName mycompanykeyvault `
  -Name storageConnectionString `
  -SecretValue $secretConnString
```

**Best Practices**:
- Use descriptive secret names (e.g., `prod-sql-admin-password`, `dev-api-key`)
- Set expiration dates for secrets requiring rotation
- Tag secrets with metadata (environment, application, owner)
- Document secret purpose and usage in Key Vault description

### Verify Secret

```bash
# List all secrets
az keyvault secret list \
  --vault-name mycompanykeyvault \
  --output table

# Show secret metadata (not value)
az keyvault secret show \
  --vault-name mycompanykeyvault \
  --name sqlAdminPassword

# Get secret value (requires appropriate permissions)
az keyvault secret show \
  --vault-name mycompanykeyvault \
  --name sqlAdminPassword \
  --query value \
  --output tsv
```

## Enable Access to Secrets

### Understanding Key Vault Permissions

Azure Key Vault supports two authorization models:

**1. Access Policies** (legacy, but still widely used)
**2. Azure RBAC** (modern, recommended)

### Access Policies Approach

Grant specific permissions to users, groups, or service principals:

**Azure CLI**:
```bash
# Grant user access to read secrets
az keyvault set-policy \
  --name mycompanykeyvault \
  --upn user@company.com \
  --secret-permissions get list

# Grant service principal access
az keyvault set-policy \
  --name mycompanykeyvault \
  --spn <service-principal-id> \
  --secret-permissions get list

# Grant Azure DevOps service connection access
az keyvault set-policy \
  --name mycompanykeyvault \
  --object-id <service-connection-object-id> \
  --secret-permissions get list
```

**PowerShell**:
```powershell
# Grant user access
Set-AzKeyVaultAccessPolicy `
  -VaultName mycompanykeyvault `
  -UserPrincipalName user@company.com `
  -PermissionsToSecrets get,list

# Grant service principal access
Set-AzKeyVaultAccessPolicy `
  -VaultName mycompanykeyvault `
  -ServicePrincipalName <app-id> `
  -PermissionsToSecrets get,list
```

**Available Secret Permissions**:
- `get`: Read secret value
- `list`: List secrets in vault
- `set`: Create/update secrets
- `delete`: Delete secrets
- `recover`: Recover deleted secrets (soft-delete enabled)
- `backup`: Backup secrets
- `restore`: Restore backed-up secrets
- `purge`: Permanently delete secrets

**Least Privilege**: ARM template deployments only need `get` permission.

### Azure RBAC Approach (Recommended)

Enable RBAC authorization on Key Vault:

```bash
# Enable RBAC authorization
az keyvault update \
  --name mycompanykeyvault \
  --resource-group keyvault-rg \
  --enable-rbac-authorization true
```

**Grant RBAC roles**:

```bash
# Grant user "Key Vault Secrets User" role (read secrets)
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee user@company.com \
  --scope /subscriptions/{subscription-id}/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault

# Grant service principal access
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <service-principal-id> \
  --scope /subscriptions/{subscription-id}/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault
```

**Built-In RBAC Roles for Key Vault**:

| Role | Description | Use Case |
|------|-------------|----------|
| Key Vault Administrator | Full access to Key Vault and all objects | Key Vault admins |
| Key Vault Secrets Officer | Manage secrets (create, update, delete) | Security team |
| Key Vault Secrets User | Read secret values | Applications, ARM templates |
| Key Vault Reader | Read Key Vault metadata (not secrets) | Auditors |

**Benefits of RBAC**:
- Consistent with other Azure services
- Easier management at scale
- Inheritance from management groups and subscriptions
- Conditional access support

### Granting ARM Template Deployment Access

ARM template deployments run under your user identity (Azure CLI/PowerShell) or service principal (CI/CD pipeline).

**Scenario 1: Deploying via Azure CLI (User Identity)**

Your user account needs permissions:

```bash
# Check your current user
az ad signed-in-user show --query id -o tsv

# Grant yourself access
az keyvault set-policy \
  --name mycompanykeyvault \
  --upn $(az ad signed-in-user show --query userPrincipalName -o tsv) \
  --secret-permissions get
```

**Scenario 2: Deploying via Azure DevOps Pipeline (Service Principal)**

Service connection's service principal needs permissions:

```bash
# Get service principal object ID from Azure DevOps service connection
# Grant access
az keyvault set-policy \
  --name mycompanykeyvault \
  --object-id <service-principal-object-id> \
  --secret-permissions get
```

**Scenario 3: Deploying via GitHub Actions (Federated Identity)**

GitHub Actions workflow identity needs permissions:

```bash
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <github-actions-app-id> \
  --scope /subscriptions/{sub}/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault
```

## Reference Secret with Static ID

### Template with Secure Parameter

**ARM Template** (main.json):
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "sqlAdminUsername": {
      "type": "string",
      "metadata": {
        "description": "SQL Server administrator username"
      }
    },
    "sqlAdminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "SQL Server administrator password (from Key Vault)"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "sqlServerName": "[concat('sqlserver-', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2021-11-01",
      "name": "[variables('sqlServerName')]",
      "location": "[parameters('location')]",
      "properties": {
        "administratorLogin": "[parameters('sqlAdminUsername')]",
        "administratorLoginPassword": "[parameters('sqlAdminPassword')]",
        "version": "12.0"
      }
    }
  ],
  "outputs": {
    "sqlServerFQDN": {
      "type": "string",
      "value": "[reference(variables('sqlServerName')).fullyQualifiedDomainName]"
    }
  }
}
```

**Key Points**:
- Parameter type is `securestring` (not visible in logs)
- Template receives password as parameter value
- Template doesn't know password comes from Key Vault

### Parameter File with Key Vault Reference

**Parameter File** (main.parameters.json):
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "sqlAdminUsername": {
      "value": "sqladmin"
    },
    "sqlAdminPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/{subscription-id}/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault"
        },
        "secretName": "sqlAdminPassword"
      }
    }
  }
}
```

**Key Vault Reference Structure**:
```json
{
  "reference": {
    "keyVault": {
      "id": "<key-vault-resource-id>"
    },
    "secretName": "<secret-name>",
    "secretVersion": "<secret-version>"  // Optional: omit for latest version
  }
}
```

**Properties**:

**keyVault.id**: Full Azure resource ID of Key Vault.

**secretName**: Name of secret in Key Vault.

**secretVersion**: (Optional) Specific version of secret. Omit to use latest version.

### Getting Key Vault Resource ID

**Azure CLI**:
```bash
az keyvault show \
  --name mycompanykeyvault \
  --query id \
  --output tsv
```

**Output**: `/subscriptions/12345678-1234-1234-1234-123456789012/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault`

**PowerShell**:
```powershell
(Get-AzKeyVault -VaultName mycompanykeyvault).ResourceId
```

### Deployment with Key Vault Secret

**Azure CLI**:
```bash
az deployment group create \
  --resource-group sql-rg \
  --template-file main.json \
  --parameters @main.parameters.json
```

**What Happens**:
1. Resource Manager reads parameter file
2. Detects Key Vault reference for `sqlAdminPassword`
3. Retrieves secret value from Key Vault using your identity
4. Passes secret value to template as secure parameter
5. SQL Server deploys with password from Key Vault
6. Password never appears in logs or deployment history

**PowerShell**:
```powershell
New-AzResourceGroupDeployment `
  -ResourceGroupName sql-rg `
  -TemplateFile main.json `
  -TemplateParameterFile main.parameters.json
```

### Inline Parameter File (Azure CLI)

You can also specify parameters inline:

```bash
az deployment group create \
  --resource-group sql-rg \
  --template-file main.json \
  --parameters sqlAdminUsername=sqladmin \
               sqlAdminPassword="@Microsoft.KeyVault(SecretUri=https://mycompanykeyvault.vault.azure.net/secrets/sqlAdminPassword/)"
```

**SecretUri Format**: `https://<vault-name>.vault.azure.net/secrets/<secret-name>/<secret-version>`

### Using Specific Secret Version

Reference specific secret version for consistency:

```json
{
  "sqlAdminPassword": {
    "reference": {
      "keyVault": {
        "id": "/subscriptions/{subscription-id}/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault"
      },
      "secretName": "sqlAdminPassword",
      "secretVersion": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
    }
  }
}
```

**Get secret version**:
```bash
az keyvault secret show \
  --vault-name mycompanykeyvault \
  --name sqlAdminPassword \
  --query id
```

**Output**: `https://mycompanykeyvault.vault.azure.net/secrets/sqlAdminPassword/a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6`

**When to use versions**:
- Production deployments requiring specific password version
- Compliance requirements for secret versioning
- Rollback scenarios to previous configurations

**When to omit versions**:
- Development environments where latest secret is acceptable
- Automatic password rotation scenarios

## Complete Deployment Example

### Scenario: Deploy VM with Password from Key Vault

**1. Create Key Vault and Store Password**

```bash
# Create Key Vault
az keyvault create \
  --name mycompanykeyvault \
  --resource-group keyvault-rg \
  --location eastus \
  --enabled-for-template-deployment true

# Store VM admin password
az keyvault secret set \
  --vault-name mycompanykeyvault \
  --name vmAdminPassword \
  --value 'MyV3ryStr0ngP@ssw0rd!'

# Grant yourself access
az keyvault set-policy \
  --name mycompanykeyvault \
  --upn $(az ad signed-in-user show --query userPrincipalName -o tsv) \
  --secret-permissions get
```

**2. Create ARM Template** (vm-template.json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "myVM"
    },
    "adminUsername": {
      "type": "string",
      "defaultValue": "azureuser"
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "VM admin password from Key Vault"
      }
    }
  },
  "variables": {
    "nicName": "[concat(parameters('vmName'), '-nic')]",
    "publicIPName": "[concat(parameters('vmName'), '-pip')]",
    "vnetName": "myVNet",
    "subnetName": "default"
  },
  "resources": [
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2023-04-01",
      "name": "[variables('publicIPName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "publicIPAllocationMethod": "Dynamic"
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2023-04-01",
      "name": "[variables('vnetName')]",
      "location": "[resourceGroup().location]",
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
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2023-04-01",
      "name": "[variables('nicName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]",
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
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
        ]
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2023-03-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_B2s"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "0001-com-ubuntu-server-jammy",
            "sku": "22_04-lts-gen2",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "Standard_LRS"
            }
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        }
      }
    }
  ]
}
```

**3. Create Parameter File** (vm-parameters.json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "value": "myVM"
    },
    "adminUsername": {
      "value": "azureuser"
    },
    "adminPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/{subscription-id}/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault"
        },
        "secretName": "vmAdminPassword"
      }
    }
  }
}
```

**4. Deploy Template**

```bash
az deployment group create \
  --resource-group vm-rg \
  --template-file vm-template.json \
  --parameters @vm-parameters.json
```

**Result**: VM deploys with password retrieved from Key Vault, password never visible in logs or deployment history.

## Benefits of Key Vault Integration

### 1. Security

**Secrets Encrypted**: Azure manages encryption keys.

**No Plain Text**: Secrets never appear in templates, parameter files checked into Git, or deployment logs.

**Access Control**: Fine-grained permissions control who can read secrets.

### 2. Flexibility

**Single Secret Update**: Change password in Key Vault, next deployment automatically uses new value.

**Environment-Specific Secrets**: Different Key Vaults for dev, staging, production.

**Secret Versioning**: Maintain multiple versions of secrets for rollback scenarios.

### 3. Auditability

**Access Logs**: Azure Monitor logs every secret retrieval with timestamp, user identity, IP address.

**Compliance**: Demonstrates proper secret handling for auditors.

**Change Tracking**: Know when secrets were created, updated, accessed.

### 4. Centralized Management

**Single Source of Truth**: All secrets in one location, not scattered across repositories.

**Rotation Automation**: Scripts update Key Vault secrets, deployments automatically use new values.

**Team Collaboration**: Security team manages Key Vault, infrastructure team deploys templates without seeing secrets.

## Best Practices

### 1. Separate Key Vaults per Environment

```bash
# Development
az keyvault create --name mycompany-dev-kv --resource-group keyvault-dev-rg

# Staging
az keyvault create --name mycompany-staging-kv --resource-group keyvault-staging-rg

# Production
az keyvault create --name mycompany-prod-kv --resource-group keyvault-prod-rg
```

**Benefits**:
- Different access permissions per environment
- Production secrets isolated from development
- Accidental use of production secrets in dev prevented

### 2. Use Consistent Secret Naming

**Good Naming Convention**:
- `{environment}-{service}-{type}`: `prod-sql-admin-password`
- `{application}-{component}-{secret}`: `webapp-storage-connectionstring`
- `{resource}-{credential}`: `sqlserver-adminpassword`

**Bad Naming**:
- `password1`, `secret2` (unclear purpose)
- `P@ssw0rd!` (doesn't describe usage)

### 3. Set Expiration Dates for Secrets

```bash
# Set secret with 90-day expiration
EXPIRY_DATE=$(date -u -d "+90 days" '+%Y-%m-%dT%H:%M:%SZ')

az keyvault secret set \
  --vault-name mycompanykeyvault \
  --name sqlAdminPassword \
  --value 'MyC0mpl3xP@ssw0rd!' \
  --expires "$EXPIRY_DATE"
```

**Benefits**:
- Enforces password rotation policies
- Azure alerts on expiring secrets
- Prevents use of outdated credentials

### 4. Enable Key Vault Diagnostic Logging

```bash
# Enable diagnostics
az monitor diagnostic-settings create \
  --name keyvault-diagnostics \
  --resource /subscriptions/{sub}/resourceGroups/keyvault-rg/providers/Microsoft.KeyVault/vaults/mycompanykeyvault \
  --logs '[{"category": "AuditEvent", "enabled": true}]' \
  --workspace /subscriptions/{sub}/resourceGroups/monitoring-rg/providers/Microsoft.OperationalInsights/workspaces/myworkspace
```

**Captures**:
- Who accessed which secrets
- When secrets were retrieved
- Source IP addresses
- Success/failure status

### 5. Use Soft Delete and Purge Protection

```bash
# Enable soft-delete (90-day retention) and purge protection
az keyvault update \
  --name mycompanykeyvault \
  --resource-group keyvault-rg \
  --enable-soft-delete true \
  --enable-purge-protection true \
  --retention-days 90
```

**Benefits**:
- Deleted secrets recoverable for 90 days
- Purge protection prevents permanent deletion
- Protects against accidental or malicious secret deletion

### 6. Limit Secret Access with Least Privilege

**Grant minimum necessary permissions**:

```bash
# Deployments only need "get" permission
az keyvault set-policy \
  --name mycompanykeyvault \
  --object-id <deployment-service-principal> \
  --secret-permissions get

# NOT this (excessive permissions):
# --secret-permissions get list set delete purge
```

## Summary

Azure Key Vault integration transforms ARM template security by eliminating hardcoded secrets and centralizing secret management with encryption, access control, and auditing.

**Key Concepts**:

**Key Vault Setup**: Enable `--enabled-for-template-deployment` to allow ARM template access to secrets.

**Access Control**: Grant users/service principals minimum required permissions (`get` permission for deployments).

**Secure Parameters**: Define template parameters as `securestring` or `secureObject` type.

**Parameter File Reference**: Use Key Vault reference syntax in parameter files to retrieve secrets during deployment.

**Secret Retrieval**: Resource Manager automatically retrieves secrets from Key Vault using deployment identity.

**Benefits**:
- No secrets in version control or deployment logs
- Centralized secret management and rotation
- Fine-grained access control with RBAC or access policies
- Complete audit trail of secret access
- Compliance with security regulations

**Best Practices**:
- Separate Key Vaults per environment
- Use consistent secret naming conventions
- Set expiration dates for password rotation
- Enable diagnostic logging for audit trail
- Enable soft-delete and purge protection
- Grant least-privilege access permissions
- Reference latest secret versions for automatic updates

**Next Unit**: Test your knowledge with assessment questions covering ARM template components, dependencies, modularization, and secrets management.

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-azure-resources-using-azure-resource-manager-templates/6-manage-secrets-in-templates)
