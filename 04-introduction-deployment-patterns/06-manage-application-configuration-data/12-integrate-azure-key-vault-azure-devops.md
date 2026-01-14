# Integrate Azure Key Vault with Azure DevOps

**Duration**: 5 minutes

Comprehensive integration patterns for accessing Azure Key Vault secrets within Azure DevOps through variable groups, pipeline tasks, and service connections, enabling secure credential management across build and release workflows.

---

## Integration Methods

### 1. Variable Groups Linked to Key Vault
**Purpose**: Share Key Vault secrets across multiple pipelines as reusable variables.

### 2. AzureKeyVault@2 Pipeline Task
**Purpose**: Dynamically retrieve secrets during pipeline execution.

### 3. Service Connections
**Purpose**: Authenticate Azure DevOps to access Azure resources including Key Vault.

---

## Variable Groups with Key Vault Integration

### Creating Variable Group (Azure DevOps UI)

**Steps**:
```
Pipelines → Library → Variable Groups → + Variable group
├── Variable group name: Production-Secrets
├── ☑ Link secrets from an Azure key vault as variables
├── Azure subscription: Select service connection
├── Key vault name: prod-keyvault
├── Authorize: (Grant Azure DevOps access)
└── + Add → Select secrets: DatabasePassword, ApiKey, StorageConnectionString
   └── Save
```

**Result**: Variable group contains references to Key Vault secrets (not actual values).

### Creating Variable Group (Azure CLI)

**Prerequisites**:
```bash
# Install Azure DevOps extension
az extension add --name azure-devops

# Set default organization and project
az devops configure --defaults \
  organization=https://dev.azure.com/myorg \
  project=MyProject
```

**Create Variable Group**:
```bash
# Create variable group
GROUP_ID=$(az pipelines variable-group create \
  --name Production-Secrets \
  --variables \
    placeholder=dummy \
  --authorize true \
  --query id -o tsv)

echo "Variable group created with ID: $GROUP_ID"
```

**Link to Key Vault (REST API)**:
```bash
# Get Azure DevOps PAT
PAT="your-personal-access-token"

# API call to link Key Vault
curl -X PUT \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $(echo -n :$PAT | base64)" \
  -d '{
    "type": "AzureKeyVault",
    "name": "Production-Secrets",
    "variables": {},
    "providerData": {
      "serviceEndpointId": "service-connection-guid",
      "vault": "prod-keyvault"
    }
  }' \
  "https://dev.azure.com/myorg/MyProject/_apis/distributedtask/variablegroups/$GROUP_ID?api-version=7.0"
```

### Using Variable Group in Pipeline

```yaml
trigger:
  branches:
    include:
    - main

# Link variable group
variables:
- group: Production-Secrets

pool:
  vmImage: 'ubuntu-latest'

steps:
# Secrets available as $(VariableName)
- script: |
    echo "Connecting to database..."
    mysql -h myserver.com -u admin -p$(DatabasePassword) mydb
  displayName: 'Database Connection'

- script: |
    curl -H "X-API-Key: $(ApiKey)" https://api.example.com/data
  displayName: 'API Request'

- task: AzureWebApp@1
  inputs:
    appName: 'myapp-prod'
    appSettings: |
      -ConnectionString "$(StorageConnectionString)"
      -ApiKey "$(ApiKey)"
```

**Benefits**:
- ✅ Centralized secret management (one variable group → many pipelines)
- ✅ Automatic secret updates (change Key Vault → pipelines use new value)
- ✅ Secret masking (values hidden in logs)

---

## Service Connection Configuration

### Creating Service Connection

**Azure DevOps UI**:
```
Project Settings → Service connections → New service connection
├── Choose: Azure Resource Manager
├── Authentication method: Service principal (automatic)
├── Scope level: Subscription
├── Subscription: Select subscription containing Key Vault
├── Resource group: (Optional) Restrict to specific resource group
├── Service connection name: Production-ServiceConnection
└── ☑ Grant access permission to all pipelines
   └── Save
```

**Azure CLI (Service Principal)**:
```bash
# Create service principal
SP=$(az ad sp create-for-rbac \
  --name "AzureDevOps-KeyVault-SP" \
  --role "Key Vault Secrets User" \
  --scopes /subscriptions/{sub-id}/resourceGroups/myapp-rg \
  --output json)

# Extract values
APP_ID=$(echo $SP | jq -r '.appId')
PASSWORD=$(echo $SP | jq -r '.password')
TENANT=$(echo $SP | jq -r '.tenant')

echo "Application ID: $APP_ID"
echo "Client Secret: $PASSWORD"
echo "Tenant ID: $TENANT"

# Create service connection in Azure DevOps UI using these credentials
```

### Granting Key Vault Access

#### Method 1: RBAC (Recommended)

```bash
# Enable RBAC authorization on Key Vault
az keyvault update \
  --name prod-keyvault \
  --resource-group myapp-rg \
  --enable-rbac-authorization true

# Assign "Key Vault Secrets User" role to service principal
az role assignment create \
  --assignee $APP_ID \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault
```

#### Method 2: Access Policies (Legacy)

```bash
# Grant Get and List permissions
az keyvault set-policy \
  --name prod-keyvault \
  --spn $APP_ID \
  --secret-permissions get list
```

### Verifying Service Connection

```bash
# Test service principal access to Key Vault
az login --service-principal \
  --username $APP_ID \
  --password $PASSWORD \
  --tenant $TENANT

# Attempt to list secrets
az keyvault secret list --vault-name prod-keyvault

# If successful, service connection configured correctly
```

---

## AzureKeyVault@2 Pipeline Task

### Basic Usage

```yaml
steps:
- task: AzureKeyVault@2
  displayName: 'Retrieve Secrets from Key Vault'
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: '*'  # Retrieve all secrets
    RunAsPreJob: false
```

### Selective Secret Retrieval

```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'DatabasePassword,ApiKey,StorageConnectionString'  # Comma-separated

# Use retrieved secrets
- script: |
    echo "Database password: $(DatabasePassword)"  # Masked
    echo "API key: $(ApiKey)"  # Masked
```

### Pattern-Based Filtering

```yaml
# Retrieve secrets matching pattern
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'Prod-*'  # All secrets starting with "Prod-"
```

---

## Complete Integration Examples

### Example 1: Multi-Environment Deployment

```yaml
trigger:
  branches:
    include:
    - main

variables:
  buildConfiguration: 'Release'

stages:
# Staging deployment
- stage: DeployStaging
  variables:
  - group: Staging-Secrets  # Variable group linked to staging-keyvault
  
  jobs:
  - deployment: StagingDeploy
    environment: 'Staging'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Staging-ServiceConnection'
              appName: 'myapp-staging'
              appSettings: |
                -ConnectionStrings__DefaultConnection "$(DatabaseConnectionString)" 
                -ApiSettings__ApiKey "$(ApiKey)"

# Production deployment (requires approval)
- stage: DeployProduction
  dependsOn: DeployStaging
  condition: succeeded()
  variables:
  - group: Production-Secrets  # Variable group linked to prod-keyvault
  
  jobs:
  - deployment: ProductionDeploy
    environment: 'Production'  # Manual approval required
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Production-ServiceConnection'
              appName: 'myapp-prod'
              appSettings: |
                -ConnectionStrings__DefaultConnection "$(DatabaseConnectionString)" 
                -ApiSettings__ApiKey "$(ApiKey)"
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

steps:
# Retrieve Docker registry credentials from Key Vault
- task: AzureKeyVault@2
  displayName: 'Get Docker Registry Credentials'
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'DockerUsername,DockerPassword'

# Login to registry
- script: |
    echo "$(DockerPassword)" | docker login $(dockerRegistry) \
      --username $(DockerUsername) \
      --password-stdin
  displayName: 'Docker Login'

# Build and push image
- script: |
    docker build -t $(dockerRegistry)/$(imageName):$(Build.BuildId) .
    docker push $(dockerRegistry)/$(imageName):$(Build.BuildId)
  displayName: 'Build and Push Docker Image'
```

### Example 3: Database Migration with Secrets

```yaml
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - database/migrations/*

pool:
  vmImage: 'ubuntu-latest'

variables:
- group: Production-Secrets  # Contains DatabasePassword

steps:
- task: DotNetCoreCLI@2
  displayName: 'Install EF Core Tools'
  inputs:
    command: 'custom'
    custom: 'tool'
    arguments: 'install --global dotnet-ef'

# Apply migrations to production database
- script: |
    dotnet ef database update \
      --connection "Server=prod-db.database.windows.net;Database=myapp;User Id=admin;Password=$(DatabasePassword)"
  displayName: 'Apply Database Migrations'
  workingDirectory: '$(System.DefaultWorkingDirectory)/MyApp.Data'
```

---

## Variable Group Management

### Updating Variable Group

**Azure DevOps UI**:
```
Pipelines → Library → Variable Groups → Select group → Add/Remove secrets
└── Save
```

**Azure CLI**:
```bash
# Add variable to group
az pipelines variable-group variable create \
  --group-id $GROUP_ID \
  --name NewSecret \
  --value "new-secret-value" \
  --secret true

# Update existing variable
az pipelines variable-group variable update \
  --group-id $GROUP_ID \
  --name ExistingSecret \
  --value "updated-value" \
  --secret true

# Delete variable
az pipelines variable-group variable delete \
  --group-id $GROUP_ID \
  --name ObsoleteSecret \
  --yes
```

### Variable Group Permissions

**Grant Pipeline Access**:
```
Variable Group → Security → Pipeline permissions
├── + Add pipeline
└── Select pipelines authorized to use this variable group
```

**Grant User Access**:
```
Variable Group → Security → User permissions
├── + Add user or group
├── Reader: View variable group
├── User: View and use in pipelines
└── Administrator: Full control
```

---

## Troubleshooting

### 1. "The user, group or application does not have secrets get permission"

**Cause**: Service principal lacks Key Vault access.

**Solution**:
```bash
# Verify current permissions
az keyvault show --name prod-keyvault --query properties.accessPolicies

# Grant access (RBAC method)
az role assignment create \
  --assignee $APP_ID \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault

# OR grant access (Access Policy method)
az keyvault set-policy \
  --name prod-keyvault \
  --spn $APP_ID \
  --secret-permissions get list
```

### 2. "Service connection authorization failed"

**Cause**: Service connection not authorized for pipeline.

**Solution**:
```
Project Settings → Service connections → Select connection
├── Security → Pipeline permissions
└── + Add pipeline → Select pipeline
   └── Save
```

### 3. "Variable group not found"

**Cause**: Variable group name mismatch or insufficient permissions.

**Solution**:
```bash
# List all variable groups
az pipelines variable-group list --query "[].{Name:name, ID:id}" -o table

# Use exact name in YAML
variables:
- group: Exact-Variable-Group-Name  # Case-sensitive
```

### 4. Secret not retrieved (empty value)

**Cause**: Secret name in Key Vault doesn't match filter.

**Solution**:
```bash
# List secrets in Key Vault
az keyvault secret list --vault-name prod-keyvault --query "[].name" -o table

# Update SecretsFilter to match exact names
- task: AzureKeyVault@2
  inputs:
    SecretsFilter: 'DatabasePassword'  # Exact match (case-sensitive)
```

---

## Security Best Practices

### 1. Least Privilege Access

```bash
# ✅ Grant only required permissions
az role assignment create \
  --assignee $APP_ID \
  --role "Key Vault Secrets User"  # Read-only

# ❌ Avoid excessive permissions
# --role "Key Vault Administrator"  # Too broad
```

### 2. Separate Key Vaults per Environment

```yaml
# ✅ Environment-specific Key Vaults
stages:
- stage: Staging
  variables:
  - group: Staging-Secrets  # → staging-keyvault

- stage: Production
  variables:
  - group: Production-Secrets  # → prod-keyvault

# ❌ Single Key Vault for all environments (blast radius)
```

### 3. Rotate Service Principal Credentials

```bash
# Rotate service principal client secret
az ad sp credential reset \
  --id $APP_ID \
  --query password -o tsv

# Update service connection in Azure DevOps with new secret
```

### 4. Audit Variable Group Access

```
Variable Group → Security → Audit
└── Review access logs (who accessed, when, what changes)
```

### 5. Enable Key Vault Logging

```bash
# Enable diagnostic settings
az monitor diagnostic-settings create \
  --name keyvault-audit \
  --resource /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault \
  --logs '[{"category":"AuditEvent","enabled":true}]' \
  --workspace /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.OperationalInsights/workspaces/myapp-logs

# Query access from Azure DevOps
az monitor log-analytics query \
  --workspace myapp-logs \
  --analytics-query "
    AzureDiagnostics
    | where ResourceProvider == 'MICROSOFT.KEYVAULT'
    | where CallerIPAddress contains 'Azure DevOps'
    | project TimeGenerated, OperationName, identity_claim_appid_g, ResultSignature
  "
```

---

## Comparison: Variable Groups vs AzureKeyVault Task

| Aspect | Variable Groups | AzureKeyVault@2 Task |
|--------|-----------------|----------------------|
| **Configuration** | Library → Variable Groups | YAML pipeline definition |
| **Reusability** | Across multiple pipelines | Single pipeline |
| **Updates** | Automatic (linked to Key Vault) | Dynamic (latest version at runtime) |
| **Secret Masking** | Automatic | Automatic |
| **Access Control** | Variable group permissions | Service connection permissions |
| **Flexibility** | Static list of secrets | Dynamic filtering (patterns) |
| **Best For** | Shared secrets across pipelines | Pipeline-specific secrets |

**Recommendation**: 
- Use **variable groups** for shared configuration (database connection strings, API endpoints)
- Use **AzureKeyVault@2 task** for pipeline-specific secrets or dynamic secret selection

---

## Advanced Patterns

### Conditional Secret Retrieval

```yaml
parameters:
- name: environment
  type: string
  default: 'staging'
  values:
  - staging
  - production

variables:
- ${{ if eq(parameters.environment, 'staging') }}:
  - group: Staging-Secrets
- ${{ if eq(parameters.environment, 'production') }}:
  - group: Production-Secrets

steps:
- script: |
    echo "Deploying to ${{ parameters.environment }}"
    echo "Database: $(DatabaseConnectionString)"
```

### Secret Expiration Handling

```yaml
# Retrieve secret with version (for specific version pinning)
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'DatabasePassword'

# Validate secret retrieved (fail pipeline if missing)
- script: |
    if [ -z "$(DatabasePassword)" ]; then
      echo "##vso[task.logissue type=error]Database password not retrieved"
      exit 1
    fi
  displayName: 'Validate Secret Retrieved'
```

---

## Real-World Example: Complete CI/CD Pipeline

### Scenario: Web Application with Multiple Environments

**Key Vault Setup**:
```bash
# Staging Key Vault
az keyvault create --name staging-keyvault --resource-group staging-rg --location eastus
az keyvault secret set --vault-name staging-keyvault --name DatabasePassword --value "StagingP@ss123"
az keyvault secret set --vault-name staging-keyvault --name ApiKey --value "staging-api-key"

# Production Key Vault
az keyvault create --name prod-keyvault --resource-group prod-rg --location eastus
az keyvault secret set --vault-name prod-keyvault --name DatabasePassword --value "ProdP@ss456"
az keyvault secret set --vault-name prod-keyvault --name ApiKey --value "prod-api-key"
```

**Variable Groups**:
```
Staging-Secrets → staging-keyvault (DatabasePassword, ApiKey)
Production-Secrets → prod-keyvault (DatabasePassword, ApiKey)
```

**Pipeline**:
```yaml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Build
  jobs:
  - job: BuildApp
    steps:
    - task: DotNetCoreCLI@2
      inputs:
        command: 'build'
        projects: '**/*.csproj'
    
    - task: DotNetCoreCLI@2
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration Release --output $(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

- stage: DeployStaging
  dependsOn: Build
  variables:
  - group: Staging-Secrets
  
  jobs:
  - deployment: StagingDeploy
    environment: 'Staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Staging-ServiceConnection'
              appName: 'myapp-staging'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
              appSettings: |
                -ConnectionStrings__DefaultConnection "Server=staging-db.database.windows.net;Database=myapp;User Id=admin;Password=$(DatabasePassword)" 
                -ApiSettings__ApiKey "$(ApiKey)"

- stage: DeployProduction
  dependsOn: DeployStaging
  condition: succeeded()
  variables:
  - group: Production-Secrets
  
  jobs:
  - deployment: ProductionDeploy
    environment: 'Production'  # Manual approval required
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Production-ServiceConnection'
              appName: 'myapp-prod'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
              appSettings: |
                -ConnectionStrings__DefaultConnection "Server=prod-db.database.windows.net;Database=myapp;User Id=admin;Password=$(DatabasePassword)" 
                -ApiSettings__ApiKey "$(ApiKey)"
```

**Result**:
- Secrets never committed to source control
- Automatic secret updates (change Key Vault → pipeline uses new value)
- Environment isolation (staging and production secrets separated)
- Full audit trail (Key Vault access logs)

---

## Key Takeaways

✅ **Variable groups**: Share Key Vault secrets across multiple pipelines  
✅ **AzureKeyVault@2 task**: Dynamically retrieve secrets during pipeline execution  
✅ **Service connections**: Authenticate Azure DevOps to Azure Key Vault  
✅ **RBAC**: Recommended access control method (Key Vault Secrets User role)  
✅ **Security**: Least privilege, environment-specific vaults, audit logging  
✅ **Best practices**: Secret masking, rotation policies, access reviews  

---

**Next**: Learn about enabling dynamic configuration and feature flags →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/12-integrate-azure-key-vault-azure-devops)
