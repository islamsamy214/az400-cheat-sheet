# Integrate Azure Key Vault with Azure Pipelines

**Duration**: 5 minutes

Azure Pipelines securely retrieves secrets from Azure Key Vault at runtime, eliminating hardcoded credentials and enabling centralized secret management across CI/CD workflows.

---

## Overview

### Integration Purpose
**Problem**: Pipeline YAML files should not contain secrets (connection strings, passwords, API keys).

**Solution**: Azure Key Vault integration allows pipelines to:
- Fetch secrets dynamically at runtime
- Avoid committing secrets to source control
- Centralize secret management across pipelines
- Maintain secret audit trails

---

## AzureKeyVault@2 Task

### Core Task for Secret Retrieval

**Purpose**: Downloads Key Vault secrets as pipeline variables during runtime.

### Basic Usage

```yaml
steps:
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'  # Service connection name
    KeyVaultName: 'prod-keyvault'                       # Key Vault name
    SecretsFilter: '*'                                  # Retrieve all secrets
    RunAsPreJob: false                                  # Run inline (not pre-job)
```

### Downloaded Secrets as Variables

```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'DatabasePassword,ApiKey,StorageConnectionString'

# Secrets now available as pipeline variables (masked)
- script: |
    echo "Connecting to database..."
    mysql -h myserver.com -u admin -p$(DatabasePassword) mydb
    
    echo "Calling API with key: $(ApiKey)"
    curl -H "X-API-Key: $(ApiKey)" https://api.example.com
```

**Secret Naming**: Hyphens in Key Vault secret names convert to variables with hyphens replaced by empty strings.
```
Key Vault Secret: database-password
Pipeline Variable: $(databasepassword)
```

---

## Prerequisites

### 1. Azure Key Vault with Secrets

```bash
# Create Key Vault
az keyvault create \
  --name prod-keyvault \
  --resource-group myapp-rg \
  --location eastus

# Add secrets
az keyvault secret set \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --value 'P@ssw0rd123!'

az keyvault secret set \
  --vault-name prod-keyvault \
  --name ApiKey \
  --value 'sk_live_abc123xyz789'

az keyvault secret set \
  --vault-name prod-keyvault \
  --name StorageConnectionString \
  --value 'DefaultEndpointsProtocol=https;AccountName=mystore;AccountKey=...'
```

### 2. Service Connection

**Definition**: Azure Resource Manager connection authorizing Azure Pipelines to access Azure resources (including Key Vault).

**Creation**:
```
Azure DevOps → Project Settings → Service Connections → New Service Connection
├── Choose: Azure Resource Manager
├── Authentication: Service Principal (automatic)
├── Scope: Subscription or Resource Group
└── Name: Production-ServiceConnection
```

**Azure CLI Creation**:
```bash
# Create service principal for Azure DevOps
az ad sp create-for-rbac \
  --name "AzureDevOps-Pipeline-SP" \
  --role contributor \
  --scopes /subscriptions/{subscription-id}

# Output provides credentials for service connection
{
  "appId": "abc-123-def-456",       # Application (client) ID
  "password": "secret-value",        # Client secret
  "tenant": "xyz-789-uvw-012"       # Tenant ID
}
```

### 3. Key Vault Access Policy

**Grant Pipeline Access**:
```bash
# Get service principal object ID
SP_OBJECT_ID=$(az ad sp show \
  --id "abc-123-def-456" \
  --query objectId -o tsv)

# Grant "Get" and "List" secret permissions
az keyvault set-policy \
  --name prod-keyvault \
  --object-id $SP_OBJECT_ID \
  --secret-permissions get list
```

**Portal Configuration**:
```
Key Vault → Access Policies → Add Access Policy
├── Secret Permissions: Get, List
├── Select Principal: AzureDevOps-Pipeline-SP
└── Add → Save
```

### 4. RBAC Alternative (Recommended)

```bash
# Enable RBAC authorization on Key Vault
az keyvault update \
  --name prod-keyvault \
  --resource-group myapp-rg \
  --enable-rbac-authorization true

# Assign "Key Vault Secrets User" role to service principal
az role assignment create \
  --assignee "abc-123-def-456" \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault
```

---

## Complete Pipeline Examples

### Example 1: Web App Deployment with Database Secrets

```yaml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  webAppName: 'myapp-prod'

stages:
- stage: Deploy
  jobs:
  - job: DeployWebApp
    steps:
    # Retrieve secrets from Key Vault
    - task: AzureKeyVault@2
      displayName: 'Get Secrets from Key Vault'
      inputs:
        azureSubscription: 'Production-ServiceConnection'
        KeyVaultName: 'prod-keyvault'
        SecretsFilter: 'DatabasePassword,ConnectionString,ApiKey'
        RunAsPreJob: false
    
    # Build application
    - task: DotNetCoreCLI@2
      displayName: 'Build Application'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
        arguments: '--configuration Release'
    
    # Deploy with secrets as environment variables
    - task: AzureWebApp@1
      displayName: 'Deploy to Azure Web App'
      inputs:
        azureSubscription: 'Production-ServiceConnection'
        appName: '$(webAppName)'
        package: '$(System.DefaultWorkingDirectory)/**/*.zip'
        appSettings: |
          -ConnectionStrings__DefaultConnection "$(ConnectionString)" 
          -DatabasePassword "$(DatabasePassword)"
          -ExternalApi__ApiKey "$(ApiKey)"
```

### Example 2: Docker Build with Registry Credentials

```yaml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  dockerRegistry: 'myregistry.azurecr.io'
  imageName: 'myapp'
  imageTag: '$(Build.BuildId)'

steps:
# Fetch Docker registry credentials
- task: AzureKeyVault@2
  displayName: 'Get Registry Credentials'
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'DockerUsername,DockerPassword'

# Login to Docker registry
- script: |
    echo "$(DockerPassword)" | docker login $(dockerRegistry) \
      --username $(DockerUsername) \
      --password-stdin
  displayName: 'Docker Login'

# Build and push image
- script: |
    docker build -t $(dockerRegistry)/$(imageName):$(imageTag) .
    docker push $(dockerRegistry)/$(imageName):$(imageTag)
  displayName: 'Build and Push Docker Image'
```

### Example 3: Multi-Stage with Different Key Vaults

```yaml
trigger:
  branches:
    include:
    - main

stages:
# Staging environment
- stage: DeployStaging
  variables:
    environment: 'staging'
  jobs:
  - job: StagingDeploy
    steps:
    - task: AzureKeyVault@2
      inputs:
        azureSubscription: 'Staging-ServiceConnection'
        KeyVaultName: 'staging-keyvault'
        SecretsFilter: '*'
    
    - script: |
        echo "Deploying to staging with DB: $(DatabasePassword)"
        # Deployment logic here
      displayName: 'Deploy to Staging'

# Production environment (requires approval)
- stage: DeployProduction
  dependsOn: DeployStaging
  condition: succeeded()
  variables:
    environment: 'production'
  jobs:
  - deployment: ProductionDeploy
    environment: 'Production'  # Requires manual approval
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureKeyVault@2
            inputs:
              azureSubscription: 'Production-ServiceConnection'
              KeyVaultName: 'prod-keyvault'
              SecretsFilter: '*'
          
          - script: |
              echo "Deploying to production with DB: $(DatabasePassword)"
              # Deployment logic here
            displayName: 'Deploy to Production'
```

---

## Secret Masking

### Automatic Secret Masking
**Behavior**: Secrets retrieved from Key Vault are automatically masked in pipeline logs.

```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'ApiKey'

- script: |
    echo "API Key: $(ApiKey)"  # Logs: "API Key: ***"
  displayName: 'Test Secret Masking'
```

**Why It Matters**: Prevents accidental secret exposure in logs shared with team members or public repositories.

### Custom Secret Variables

```yaml
variables:
  customSecret: 'my-secret-value'  # Not automatically masked

steps:
- bash: |
    echo "##vso[task.setvariable variable=customSecret;issecret=true]my-secret-value"
  displayName: 'Mark Variable as Secret'

- script: |
    echo "Custom Secret: $(customSecret)"  # Now masked: "Custom Secret: ***"
```

---

## Variable Groups (Alternative Method)

### Concept
**Variable Group**: Collection of variables (including Key Vault secrets) shared across multiple pipelines.

### Creating Variable Group Linked to Key Vault

**Azure DevOps UI**:
```
Pipelines → Library → Variable Groups → + Variable Group
├── Name: Production-Secrets
├── Enable: "Link secrets from an Azure key vault as variables"
├── Azure Subscription: Production-ServiceConnection
├── Key Vault Name: prod-keyvault
├── Add Secrets: DatabasePassword, ApiKey, StorageConnectionString
└── Save
```

**Azure CLI**:
```bash
# Create variable group
az pipelines variable-group create \
  --name Production-Secrets \
  --variables \
    DatabasePassword="$(az keyvault secret show --vault-name prod-keyvault --name DatabasePassword --query value -o tsv)" \
    ApiKey="$(az keyvault secret show --vault-name prod-keyvault --name ApiKey --query value -o tsv)"

# Link variable group to Key Vault (requires API call - see Microsoft docs)
```

### Using Variable Group in Pipeline

```yaml
trigger:
  branches:
    include:
    - main

# Reference variable group
variables:
- group: Production-Secrets

pool:
  vmImage: 'ubuntu-latest'

steps:
# Secrets from variable group available as $(variableName)
- script: |
    echo "Connecting to database with password: $(DatabasePassword)"
    mysql -h myserver.com -u admin -p$(DatabasePassword) mydb
  displayName: 'Connect to Database'

- script: |
    curl -H "X-API-Key: $(ApiKey)" https://api.example.com/data
  displayName: 'Call External API'
```

---

## Service Connection Configuration

### Service Principal Authentication

**Steps**:
1. Create service principal in Azure AD
2. Grant Key Vault access to service principal
3. Configure service connection in Azure DevOps

```bash
# 1. Create service principal
az ad sp create-for-rbac \
  --name "AzureDevOps-Production" \
  --role "Key Vault Secrets User" \
  --scopes /subscriptions/{sub-id}/resourceGroups/myapp-rg

# Output
{
  "appId": "abc-123",
  "password": "secret",
  "tenant": "xyz-789"
}

# 2. Create service connection in Azure DevOps
# (Use appId, password, tenant from above)

# 3. Grant Key Vault access (if using access policies)
az keyvault set-policy \
  --name prod-keyvault \
  --spn "abc-123" \
  --secret-permissions get list
```

### Managed Identity Authentication (Azure-Hosted Agents)

**For Self-Hosted Agents with Managed Identity**:
```bash
# Enable system-assigned managed identity on VM
az vm identity assign \
  --name pipeline-agent-vm \
  --resource-group agents-rg

# Get managed identity principal ID
IDENTITY_ID=$(az vm show \
  --name pipeline-agent-vm \
  --resource-group agents-rg \
  --query identity.principalId -o tsv)

# Grant Key Vault access
az role assignment create \
  --assignee $IDENTITY_ID \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault
```

**Service Connection (Managed Identity)**:
```
Azure DevOps → Service Connections → New → Azure Resource Manager
├── Authentication: Managed Identity
├── Subscription: Select subscription
└── Name: Production-ManagedIdentity
```

---

## Troubleshooting

### 1. "The user, group or application does not have secrets get permission"

**Cause**: Service principal lacks Key Vault access.

**Solution**:
```bash
# Check current permissions
az keyvault show --name prod-keyvault --query properties.accessPolicies

# Grant access (access policies method)
az keyvault set-policy \
  --name prod-keyvault \
  --spn "abc-123" \
  --secret-permissions get list

# Grant access (RBAC method)
az role assignment create \
  --assignee "abc-123" \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault
```

### 2. "Service connection not found"

**Cause**: Service connection name mismatch in YAML.

**Solution**:
```bash
# List service connections
az devops service-endpoint list \
  --organization https://dev.azure.com/myorg \
  --project MyProject

# Use exact name from list in YAML
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Exact-ServiceConnection-Name'  # Case-sensitive
```

### 3. Secrets Not Retrieved

**Cause**: `SecretsFilter` pattern doesn't match secret names.

**Solution**:
```yaml
# ❌ Wrong: Secret name is "DatabasePassword" but filter has typo
- task: AzureKeyVault@2
  inputs:
    SecretsFilter: 'DatabasePasswrod'  # Typo

# ✅ Correct: Exact match or wildcard
- task: AzureKeyVault@2
  inputs:
    SecretsFilter: 'DatabasePassword,ApiKey'  # Exact names
    # OR
    SecretsFilter: '*'  # All secrets

# List secrets to verify names
az keyvault secret list --vault-name prod-keyvault --query "[].name"
```

### 4. "Key Vault does not exist"

**Cause**: Key Vault name incorrect or in different subscription.

**Solution**:
```bash
# Verify Key Vault exists and service principal has access
az keyvault show --name prod-keyvault

# If Key Vault in different subscription:
az account set --subscription "Other-Subscription-ID"
az keyvault show --name prod-keyvault
```

---

## Best Practices

### 1. Least Privilege Access
```bash
# ✅ Grant only "Get" and "List" permissions (not "Set" or "Delete")
az keyvault set-policy \
  --name prod-keyvault \
  --spn "abc-123" \
  --secret-permissions get list

# ❌ Avoid granting full permissions
# --secret-permissions all  # Too permissive
```

### 2. Environment-Specific Key Vaults
```yaml
# ✅ Separate Key Vaults per environment
stages:
- stage: Staging
  jobs:
  - job: Deploy
    steps:
    - task: AzureKeyVault@2
      inputs:
        KeyVaultName: 'staging-keyvault'

- stage: Production
  jobs:
  - job: Deploy
    steps:
    - task: AzureKeyVault@2
      inputs:
        KeyVaultName: 'prod-keyvault'  # Different Key Vault

# ❌ Avoid single Key Vault for all environments (blast radius)
```

### 3. Secret Rotation
```bash
# Rotate secrets regularly (Azure Key Vault supports versioning)
az keyvault secret set \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --value 'NewP@ssw0rd456!'

# Pipelines automatically use latest secret version (default behavior)
```

### 4. Audit Logging
```bash
# Enable diagnostic settings for Key Vault
az monitor diagnostic-settings create \
  --name keyvault-audit-logs \
  --resource /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault \
  --logs '[{"category":"AuditEvent","enabled":true}]' \
  --workspace /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.OperationalInsights/workspaces/myapp-logs

# Query access logs
az monitor log-analytics query \
  --workspace myapp-logs \
  --analytics-query "AzureDiagnostics | where ResourceProvider == 'MICROSOFT.KEYVAULT' | where OperationName == 'SecretGet'"
```

### 5. Fail Fast on Missing Secrets
```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'RequiredSecret'

# Validate secret retrieved
- script: |
    if [ -z "$(RequiredSecret)" ]; then
      echo "##vso[task.logissue type=error]Required secret not found"
      exit 1
    fi
  displayName: 'Validate Secret Retrieved'
```

---

## Comparison: AzureKeyVault Task vs Variable Groups

| Aspect | AzureKeyVault@2 Task | Variable Groups |
|--------|----------------------|-----------------|
| **Definition Location** | YAML pipeline | Azure DevOps Library |
| **Reusability** | Per-pipeline | Across pipelines |
| **Dynamic Retrieval** | Runtime | Pre-populated |
| **Secret Updates** | Automatic (latest version) | Manual sync required |
| **Access Control** | Service connection | Variable group permissions |
| **Best For** | Pipeline-specific secrets | Shared secrets |

**Recommendation**: Use **AzureKeyVault@2 task** for dynamic secrets; use **Variable Groups** for shared configuration across pipelines.

---

## Key Takeaways

✅ **AzureKeyVault@2 task**: Retrieves Key Vault secrets as pipeline variables at runtime  
✅ **Prerequisites**: Key Vault, service connection, access policy/RBAC  
✅ **Secret masking**: Automatic for Key Vault secrets; manual for custom variables  
✅ **Variable groups**: Share Key Vault secrets across multiple pipelines  
✅ **Service connections**: Service principal or managed identity authentication  
✅ **Best practices**: Least privilege, environment-specific vaults, secret rotation, audit logging  

---

**Next**: Learn about comprehensive secrets, tokens, and certificates management →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/9-integrate-azure-key-vault-azure-pipelines)
