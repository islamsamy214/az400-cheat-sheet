# Implement Managed Identities

**Duration**: 4 minutes

Learn how managed identities eliminate credential management through passwordless Azure authentication, enabling secure service-to-service communication with zero secret rotation overhead.

---

## Overview

**Managed identities** provide Azure resource-native authentication without credential storage, automatically provisioned and lifecycle-managed by the Azure platform. This **passwordless authentication** pattern eliminates client secret rotation, prevents credential leakage, and enables zero-trust security architectures for DevOps workflows.

---

## Managed Identity Fundamentals

### What is a Managed Identity?

**Definition**: Specialized service principal automatically created and managed by Azure, associated with Azure resources for passwordless authentication to Azure services.

**Key Characteristics**:
- ✅ **No credential management**: Azure handles token provisioning automatically
- ✅ **Automatic rotation**: Credentials rotated by platform (invisible to applications)
- ✅ **Lifecycle-managed**: Created/deleted with Azure resource (system-assigned) or independently (user-assigned)
- ✅ **Zero secret storage**: No client secrets, certificates, or passwords
- ✅ **Native Azure integration**: Supported by 50+ Azure services

**Problem Solved**: Traditional service principals require client secret rotation every 12-24 months, creating operational overhead and security risk.

---

## Azure Data Factory Example: Passwordless SQL Authentication

### Traditional Approach (SQL Authentication)

```
Azure Data Factory → Azure SQL Database
├── Connection String: "Server=sql-prod.database.windows.net;Database=analytics;User ID=sqladmin;Password=P@ssw0rd123!"
├── Credential Storage: Connection string in ADF linked service
├── Rotation Overhead: Change password every 90 days
│   ├── Update SQL Database password
│   ├── Update ADF linked service connection string
│   ├── Test all pipelines
│   └── Risk: Forgot to update → pipeline failures
├── Security Risks:
│   ├── Password stored in ADF configuration
│   ├── Visible to ADF administrators
│   ├── Logged in ADF activity logs
│   └── Credential leakage risk
└── Management Cost: High (recurring manual rotation)
```

---

### Modern Approach (Managed Identity)

```
Azure Data Factory → Azure SQL Database
├── Authentication: Managed Identity (passwordless)
├── Connection String: "Server=sql-prod.database.windows.net;Database=analytics;Authentication=Active Directory Managed Identity"
├── Credential Storage: None (Azure handles tokens automatically)
├── Rotation Overhead: Zero (automatic platform-managed rotation)
├── Security Benefits:
│   ├── No passwords stored anywhere
│   ├── Credentials invisible to administrators
│   ├── Token-based authentication (short-lived)
│   └── Zero credential leakage risk
└── Management Cost: Minimal (one-time configuration)
```

**Implementation Steps**:
```
1. Enable ADF Managed Identity:
   - Automatically enabled when ADF resource created
   - Principal ID: 11111111-2222-3333-4444-555555555555

2. Grant SQL Database Access:
   - Connect to SQL Database with Entra ID admin account
   - CREATE USER [adf-prod-etl] FROM EXTERNAL PROVIDER;
   - ALTER ROLE db_datareader ADD MEMBER [adf-prod-etl];
   - ALTER ROLE db_datawriter ADD MEMBER [adf-prod-etl];

3. Configure ADF Linked Service:
   - Connection String: Server=sql-prod.database.windows.net;Database=analytics
   - Authentication: Managed Identity
   - No credentials required

4. Test Pipeline:
   - ADF fetches access token from Azure Instance Metadata Service (IMDS)
   - Token presented to SQL Database for authentication
   - SQL Database validates token with Entra ID
   - Pipeline executes successfully
```

**Result**: ADF pipelines access SQL Database with zero credential management.

---

## Two Managed Identity Types

### 1. System-Assigned Managed Identity

**Definition**: Service-native identity automatically provisioned when creating Azure resource, lifecycle-coupled to parent resource.

#### Characteristics
- **Lifecycle**: Created with parent resource, deleted when resource deleted
- **Shareability**: Not shareable (1:1 relationship with Azure resource)
- **Use Case**: Single Azure resource authenticating to other services
- **Identity Object**: Automatically created, no manual provisioning

#### When Azure Resource Created
```
az vm create \
  --resource-group rg-prod \
  --name vm-app-prod \
  --image Ubuntu2022 \
  --assign-identity  # ← Enables system-assigned managed identity

Result:
├── VM Created: vm-app-prod
├── System-Assigned Managed Identity:
│   ├── Principal ID: 11111111-2222-3333-4444-555555555555
│   ├── Tenant ID: tenant123-4567-8901-2345-678901234567
│   ├── Name: Same as resource name (vm-app-prod)
│   └── Lifecycle: Coupled (deleted when VM deleted)
└── Role Assignments: None (must grant permissions separately)
```

#### Azure Resources Supporting System-Assigned MI
- Virtual Machines (VMs)
- App Services (Web Apps, Function Apps)
- Azure Container Instances (ACI)
- Azure Kubernetes Service (AKS) pods
- Azure Data Factory (ADF)
- Logic Apps
- API Management
- Azure Spring Cloud
- 50+ Azure services

---

### 2. User-Assigned Managed Identity

**Definition**: Standalone Azure resource explicitly provisioned, shareable across multiple service instances.

#### Characteristics
- **Lifecycle**: Independent (persists after parent resources deleted)
- **Shareability**: Shareable (1:many relationship, assigned to multiple resources)
- **Use Case**: Multiple Azure resources needing shared identity
- **Identity Object**: Manually provisioned as separate Azure resource

#### Provisioning User-Assigned MI
```bash
# Create user-assigned managed identity
az identity create \
  --resource-group rg-prod \
  --name mi-shared-app-identity

# Output:
{
  "clientId": "22222222-3333-4444-5555-666666666666",
  "principalId": "33333333-4444-5555-6666-777777777777",
  "resourceId": "/subscriptions/.../resourceGroups/rg-prod/providers/Microsoft.ManagedIdentity/userAssignedIdentities/mi-shared-app-identity",
  "tenantId": "tenant123-4567-8901-2345-678901234567"
}
```

#### Assigning to Multiple Resources
```bash
# Assign to VM 1
az vm identity assign \
  --resource-group rg-prod \
  --name vm-app-01 \
  --identities /subscriptions/.../mi-shared-app-identity

# Assign to VM 2
az vm identity assign \
  --resource-group rg-prod \
  --name vm-app-02 \
  --identities /subscriptions/.../mi-shared-app-identity

# Assign to App Service
az webapp identity assign \
  --resource-group rg-prod \
  --name webapp-app \
  --identities /subscriptions/.../mi-shared-app-identity

Result:
    User-Assigned MI: mi-shared-app-identity
    ├── Assigned to: vm-app-01 ✅
    ├── Assigned to: vm-app-02 ✅
    └── Assigned to: webapp-app ✅
    
    All 3 resources share same identity for authentication
```

---

## System-Assigned vs User-Assigned Comparison

| Feature | System-Assigned | User-Assigned |
|---------|----------------|---------------|
| **Lifecycle** | Coupled to parent resource | Independent standalone resource |
| **Creation** | Automatic (with parent resource) | Manual (explicit provisioning) |
| **Deletion** | Automatic (when parent deleted) | Manual (persists after parent deletion) |
| **Shareability** | No (1:1 with resource) | Yes (1:many across resources) |
| **Use Case** | Single resource authentication | Shared identity across multiple resources |
| **Role Assignment Scope** | Per-resource identity | Centralized identity permissions |
| **Management Overhead** | Lower (no separate object) | Higher (separate resource lifecycle) |
| **Flexibility** | Lower (deleted with resource) | Higher (reusable, portable) |
| **Cost** | Free (part of parent resource) | Free (charged as part of Entra ID) |
| **Naming** | Same as parent resource | Custom naming |

### When to Choose Each Type

**System-Assigned** (Recommended Default):
```
✅ Single Azure resource needs authentication
Example: Azure Function App accessing Key Vault
    └── Function App → Key Vault (system-assigned MI)

✅ Identity lifecycle matches resource lifecycle
Example: VM exists only during project (delete VM = delete identity)

✅ Simplified management (no separate identity object)
Example: App Service accessing SQL Database (1:1 relationship)
```

**User-Assigned** (Shared Identity Scenarios):
```
✅ Multiple Azure resources share same permissions
Example: 10 VMs accessing same Storage Account
    └── All 10 VMs → User-assigned MI → Storage Account

✅ Identity lifecycle independent of resources
Example: Identity persists during resource upgrades/replacements

✅ Complex permission management
Example: Centralized identity with role assignments reused across resources

✅ Workload identity federation (Azure Pipelines, GitHub Actions)
Example: GitHub Actions → User-assigned MI → Azure resources
```

---

## Workload Identity Federation

**Definition**: OpenID Connect (OIDC) trust relationship between external identity providers (GitHub Actions, Azure Pipelines) and Azure, enabling passwordless authentication without service principal secrets.

### Problem with Traditional Service Principals

```
GitHub Actions → Azure Resources (Traditional)
├── Authentication: Service Principal
├── Credentials Required:
│   ├── Client ID: 12345678-1234-1234-1234-123456789abc
│   ├── Client Secret: "abc123def456..." ⚠️ Secret storage required
│   └── Tenant ID: tenant123-4567-8901-2345-678901234567
├── Secret Management:
│   ├── Store in GitHub Secrets (encrypted at rest)
│   ├── Rotate every 12 months (operational overhead)
│   ├── Risk: Secret leakage in logs, console output
│   └── Audit: Track secret usage across workflows
└── Security Risk: Medium-High (credential-based)
```

---

### Solution: Workload Identity Federation

```
GitHub Actions → Azure Resources (Workload Identity Federation)
├── Authentication: User-Assigned Managed Identity + OIDC Federation
├── Credentials Required: None (passwordless)
├── Trust Relationship:
│   ├── GitHub identity provider: https://token.actions.githubusercontent.com
│   ├── Federated credential: organization/repo/environment
│   └── Azure managed identity trusts GitHub tokens
├── Secret Management: Zero (no secrets to store/rotate)
├── Authentication Flow:
│   1. GitHub Actions requests OIDC token from GitHub
│   2. GitHub issues token with claims (repo, workflow, branch)
│   3. GitHub Actions presents token to Azure
│   4. Azure validates token with GitHub's public keys
│   5. Azure issues access token to managed identity
│   6. Workflow accesses Azure resources
└── Security Risk: Low (token-based, short-lived)
```

---

## Workload Identity Federation Setup

### Scenario: GitHub Actions Deploying Azure Resources

#### Step 1: Create User-Assigned Managed Identity

```bash
# Create managed identity
az identity create \
  --resource-group rg-prod-identity \
  --name mi-github-actions-prod

# Grant permissions (Contributor on subscription)
az role assignment create \
  --assignee $(az identity show --resource-group rg-prod-identity --name mi-github-actions-prod --query principalId -o tsv) \
  --role Contributor \
  --scope /subscriptions/<subscription-id>
```

---

#### Step 2: Configure Federated Credential

**Azure Portal**: Managed Identity → Federated credentials → Add credential

```
Federated Credential Configuration:
├── Scenario: GitHub Actions deploying Azure resources
├── Federated credential name: github-prod-workflow
├── Organization: contoso (GitHub organization)
├── Repository: webapp-infrastructure (GitHub repository)
├── Entity type: Environment (or Branch, Pull request, Tag)
├── GitHub environment name: production (matches workflow environment)
├── Issuer: https://token.actions.githubusercontent.com (GitHub OIDC provider)
├── Subject identifier: repo:contoso/webapp-infrastructure:environment:production
└── [Add]

Result:
    Federation trust established between:
    - GitHub Actions workflows in "contoso/webapp-infrastructure" repo
    - Deploying to "production" environment
    - Azure managed identity "mi-github-actions-prod"
```

**Subject Identifier Patterns**:
```
Entity Type: Branch
Subject: repo:organization/repository:ref:refs/heads/main

Entity Type: Pull Request
Subject: repo:organization/repository:pull_request

Entity Type: Tag
Subject: repo:organization/repository:ref:refs/tags/v1.0.0

Entity Type: Environment
Subject: repo:organization/repository:environment:production
```

---

#### Step 3: GitHub Workflow Configuration

```yaml
# .github/workflows/deploy-prod.yml
name: Deploy to Production

on:
  push:
    branches: [main]

permissions:
  id-token: write  # ← Required for OIDC token request
  contents: read

env:
  AZURE_CLIENT_ID: '22222222-3333-4444-5555-666666666666'  # Managed identity client ID
  AZURE_TENANT_ID: 'tenant123-4567-8901-2345-678901234567'
  AZURE_SUBSCRIPTION_ID: 'sub456-7890-1234-5678-901234567890'

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # ← Matches federated credential environment
    steps:
      - uses: actions/checkout@v3
      
      - name: Azure Login (Workload Identity Federation)
        uses: azure/login@v1
        with:
          client-id: ${{ env.AZURE_CLIENT_ID }}
          tenant-id: ${{ env.AZURE_TENANT_ID }}
          subscription-id: ${{ env.AZURE_SUBSCRIPTION_ID }}
          # NO CLIENT SECRET REQUIRED! ✅
      
      - name: Deploy Infrastructure
        uses: azure/arm-deploy@v1
        with:
          resourceGroupName: rg-prod-webapp
          template: ./infrastructure/main.bicep
          parameters: environment=production
      
      - name: Deploy Application
        uses: azure/webapps-deploy@v2
        with:
          app-name: webapp-prod-contoso
          package: './app.zip'
```

**Authentication Flow**:
```
1. Workflow Starts: GitHub Actions runner begins execution
2. Request OIDC Token: Workflow requests token from GitHub OIDC provider
   POST https://token.actions.githubusercontent.com/.well-known/openid-configuration
3. GitHub Issues Token: JWT token with claims:
   {
     "iss": "https://token.actions.githubusercontent.com",
     "sub": "repo:contoso/webapp-infrastructure:environment:production",
     "aud": "https://management.azure.com",
     "repository": "contoso/webapp-infrastructure",
     "environment": "production",
     ...
   }
4. Present Token to Azure: azure/login action sends token to Azure
   POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
   Body: client_assertion={github_token}&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
5. Azure Validates Token:
   - Verify issuer: https://token.actions.githubusercontent.com
   - Verify subject matches federated credential: repo:contoso/webapp-infrastructure:environment:production
   - Verify signature with GitHub's public keys
   - Check token expiration
6. Azure Issues Access Token: Azure returns access token for managed identity
7. Workflow Accesses Resources: Use access token to call Azure Resource Manager APIs
8. Deploy Complete: Infrastructure and application deployed
```

**Security Benefits**:
- ✅ Zero secrets stored in GitHub (no CLIENT_SECRET in repository secrets)
- ✅ Short-lived tokens (5-15 minutes, auto-expired)
- ✅ Cryptographically signed tokens (GitHub's private keys)
- ✅ Granular trust (specific repo/branch/environment only)
- ✅ Audit trail (token claims include repo, workflow, commit SHA)

---

### Azure Pipelines Workload Identity Federation

**Azure DevOps Service Connection Configuration**:

```
Service Connection: Azure Resource Manager (Workload Identity Federation)
├── Authentication Method: Workload Identity federation (automatic)
├── Subscription: Production
├── Resource Group: rg-prod (optional scope)
├── Service connection name: Azure-Production-ManagedIdentity
└── Automatic Configuration:
    ├── Creates user-assigned managed identity
    ├── Grants Contributor role to subscription
    ├── Configures federated credential (Azure DevOps trust)
    └── No client secrets generated ✅
```

**Manual Configuration** (Existing Managed Identity):
```
Service Connection: Azure Resource Manager (Workload Identity Federation)
├── Authentication Method: Workload Identity federation (manual)
├── Subscription: Production
├── Service connection name: Azure-Production-ManagedIdentity
├── Managed Identity Configuration:
│   ├── User-assigned managed identity: mi-azdo-pipelines-prod
│   ├── Client ID: 22222222-3333-4444-5555-666666666666
│   ├── Tenant ID: tenant123-4567-8901-2345-678901234567
│   └── Federated credential: Already configured with Azure DevOps issuer
└── Verify connection → [Save]
```

**Azure Pipelines YAML**:
```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  azureServiceConnection: 'Azure-Production-ManagedIdentity'  # ← Workload identity federation service connection

stages:
- stage: Deploy
  jobs:
  - job: DeployInfrastructure
    steps:
    - task: AzureCLI@2
      displayName: 'Deploy Bicep Template'
      inputs:
        azureSubscription: $(azureServiceConnection)  # ← No secrets, passwordless auth
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az deployment group create \
            --resource-group rg-prod-webapp \
            --template-file infrastructure/main.bicep \
            --parameters environment=production
```

---

## OpenID Connect Configuration

### Azure DevOps Federation Trust

**Federated Credential Configuration**:
```
Managed Identity → Federated credentials → Add credential
├── Scenario: Other (Azure DevOps)
├── Federated credential name: azdo-prod-pipelines
├── Issuer: https://vstoken.dev.azure.com/<organization-id>
├── Subject identifier: sc://<organization>/<project>/<service-connection-name>
├── Audiences: api://AzureADTokenExchange (default)
└── [Add]

Example:
    Issuer: https://vstoken.dev.azure.com/12345678-1234-1234-1234-123456789abc
    Subject: sc://contoso/WebApp-Project/Azure-Production-ManagedIdentity
```

**Subject Identifier Components**:
- `sc://`: Service connection prefix
- `contoso`: Azure DevOps organization name
- `WebApp-Project`: Project name
- `Azure-Production-ManagedIdentity`: Service connection name

---

### GitHub Actions Federation Trust

**Federated Credential Configuration**:
```
Managed Identity → Federated credentials → Add credential
├── Scenario: GitHub Actions deploying Azure resources
├── Federated credential name: github-prod-deploy
├── Issuer: https://token.actions.githubusercontent.com
├── Subject identifier: repo:organization/repository:environment:production
├── Audiences: api://AzureADTokenExchange (default)
└── [Add]
```

**Advanced Subject Patterns** (Multiple Triggers):
```
# Accept from main branch OR production environment
repo:contoso/webapp-infrastructure:ref:refs/heads/main
repo:contoso/webapp-infrastructure:environment:production

# Accept from any tag v*
repo:contoso/webapp-infrastructure:ref:refs/tags/v*

# Accept from pull requests
repo:contoso/webapp-infrastructure:pull_request
```

---

## Benefits of Managed Identities

### 1. Eliminate Credential Management
```
Traditional Service Principal:
├── Create client secret (expires 12 months)
├── Store in Azure Key Vault / GitHub Secrets
├── Rotate every 12 months:
│   ├── Generate new secret
│   ├── Update all service connections
│   ├── Test all pipelines
│   └── Delete old secret
└── Operational Overhead: High (recurring manual process)

Managed Identity:
├── Enable managed identity (one-time)
├── Grant permissions (one-time)
└── Operational Overhead: Zero (automatic token management)
```

### 2. Prevent Credential Leakage
```
Service Principal Risks:
├── Secret printed in logs (accidental console.log(process.env.CLIENT_SECRET))
├── Secret committed to repository (forgotten in config file)
├── Secret shared via email/Slack (team collaboration)
└── Secret exposed in error messages (stack traces)

Managed Identity Protection:
├── No secrets exist (nothing to leak)
├── Tokens short-lived (5-15 minutes)
├── Tokens non-extractable (Azure internal token endpoint)
└── Zero credential leakage risk
```

### 3. Simplified Multi-Tenant Deployment
```
Service Principal (Multi-Tenant SaaS):
├── Customer A: Create service principal, store secret
├── Customer B: Create service principal, store secret
├── Customer C: Create service principal, store secret
├── ...
├── Customer Z: Create service principal, store secret
└── Management: 26 service principals, 26 secrets to rotate

Managed Identity (Workload Identity Federation):
├── Single user-assigned managed identity
├── Federated credentials for each customer tenant
├── No secrets to store or rotate
└── Management: 1 managed identity, 0 secrets
```

### 4. Enhanced Security Posture
```
✅ Zero-trust architecture (no long-lived secrets)
✅ Short-lived tokens (automatic expiration)
✅ Platform-managed rotation (invisible to applications)
✅ Audit trail (token issuance logged)
✅ Least-privilege (managed identity role assignments)
```

---

## Implementation Checklist

### System-Assigned Managed Identity
```
☐ Enable managed identity on Azure resource
   Example: az vm identity assign --resource-group rg-prod --name vm-app-01

☐ Grant permissions to managed identity
   Example: az role assignment create --assignee <principal-id> --role Contributor --scope <resource-id>

☐ Update application code (if applicable)
   Example: Use DefaultAzureCredential() in application for automatic token retrieval

☐ Test authentication
   Example: Verify resource can access target service (Key Vault, Storage Account)

☐ Remove hardcoded credentials
   Example: Delete connection strings with passwords, client secrets from configuration
```

### User-Assigned Managed Identity
```
☐ Create user-assigned managed identity
   Example: az identity create --resource-group rg-prod-identity --name mi-shared-app

☐ Assign identity to Azure resources
   Example: az vm identity assign --resource-group rg-prod --name vm-app-01 --identities <identity-resource-id>

☐ Grant permissions to managed identity
   Example: az role assignment create --assignee <principal-id> --role Contributor --scope <subscription-id>

☐ Update application code (specify client ID)
   Example: ManagedIdentityCredential(client_id="22222222-3333-4444-5555-666666666666")

☐ Test authentication across all assigned resources
   Example: Verify all VMs/App Services can authenticate using shared identity
```

### Workload Identity Federation
```
☐ Create user-assigned managed identity
   Example: az identity create --name mi-github-actions-prod

☐ Configure federated credential
   Example: Azure Portal → Managed Identity → Federated credentials → Add (GitHub Actions scenario)

☐ Grant Azure permissions
   Example: Contributor role on subscription/resource group

☐ Update GitHub workflow / Azure Pipeline
   Example: Add azure/login@v1 with client-id (no client-secret)

☐ Test workflow authentication
   Example: Trigger pipeline, verify passwordless authentication succeeds

☐ Remove service principal secrets
   Example: Delete AZURE_CREDENTIALS secret from GitHub repository
```

---

## Quick Reference

### Enable System-Assigned MI (Azure CLI)
```bash
# VM
az vm identity assign --resource-group rg-prod --name vm-app-01

# App Service
az webapp identity assign --resource-group rg-prod --name webapp-prod

# Azure Data Factory
az datafactory identity assign --resource-group rg-prod --factory-name adf-prod

# Function App
az functionapp identity assign --resource-group rg-prod --name func-app-prod
```

### Create User-Assigned MI
```bash
# Create identity
az identity create --resource-group rg-prod-identity --name mi-shared-app

# Assign to VM
az vm identity assign \
  --resource-group rg-prod \
  --name vm-app-01 \
  --identities /subscriptions/.../mi-shared-app

# Grant permissions
PRINCIPAL_ID=$(az identity show --resource-group rg-prod-identity --name mi-shared-app --query principalId -o tsv)
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role Contributor \
  --scope /subscriptions/<subscription-id>
```

### Application Code (Python)
```python
from azure.identity import DefaultAzureCredential, ManagedIdentityCredential
from azure.keyvault.secrets import SecretClient
from azure.storage.blob import BlobServiceClient

# System-assigned MI (automatic detection)
credential = DefaultAzureCredential()
secret_client = SecretClient(vault_url="https://kv-prod.vault.azure.net", credential=credential)
secret = secret_client.get_secret("DatabasePassword")

# User-assigned MI (specify client ID)
credential = ManagedIdentityCredential(client_id="22222222-3333-4444-5555-666666666666")
blob_service = BlobServiceClient(account_url="https://stproddata.blob.core.windows.net", credential=credential)
```

---

## Next Steps

✅ **Completed**: Managed identities and workload identity federation

**Next Unit**: Test your knowledge with module assessment questions →

---

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-identity-management-systems/6-explore-managed-identity)
