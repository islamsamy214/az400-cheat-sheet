# Explore Workload Identities

**Duration**: 4 minutes

Learn how to provision service principals for workload authentication, enabling Azure Pipelines and automated systems to access Azure resources without hardcoded credentials.

---

## Overview

**Workload identities** are identity constructs representing software workloads (applications, services, scripts, containers) requiring authenticated access to protected resources. Azure DevOps and GitHub Actions leverage **service principals** to authenticate CI/CD pipelines for Azure resource deployments without credential management overhead.

---

## Workload Identity Concepts

### What are Workload Identities?

**Definition**: Identity objects enabling non-human entities (applications, services, automation) to authenticate and access protected environments.

**Problem Solved**: Traditional user credentials (username/password) unsuitable for automation workflows

**Use Cases**:
- Azure Pipelines deploying Azure resources
- GitHub Actions executing infrastructure-as-code
- Background services accessing databases
- Containerized applications calling APIs

---

## Three Workload Identity Types

### 1. Application (Microsoft Entra ID Object)

**Definition**: Global application definition representing software across all tenants.

#### Characteristics
- **Scope**: Multi-tenant capable (shared application definition)
- **Location**: Microsoft Entra ID app registrations
- **Purpose**: Defines token issuance policies, resource access scopes
- **Permissions**: Specifies permissible actions across Azure resources
- **Lifecycle**: Independent (deleted separately from service principals)

#### Application Properties
```
Application Object:
├── Application (client) ID: 12345678-1234-1234-1234-123456789abc
├── Object ID: abcdef12-3456-7890-abcd-ef1234567890
├── Display Name: "CI/CD Pipeline App"
├── Supported Account Types: Single tenant / Multi-tenant / Personal accounts
├── Redirect URIs: https://app.example.com/auth/callback
├── API Permissions: Azure Resource Manager (read/write)
└── Certificates & Secrets: Client secrets, certificate credentials
```

**Analogy**: Application is the **blueprint** defining how authentication works globally

---

### 2. Service Principal

**Definition**: Local tenant representation of an application, created during app registration within a specific Microsoft Entra tenant.

#### Characteristics
- **Scope**: Tenant-specific (each tenant has own service principal instance)
- **Location**: Enterprise applications in Entra ID
- **Purpose**: Defines tenant-specific permissions and accessible resources
- **Permissions**: Role assignments within tenant (Contributor, Reader, Owner)
- **Authentication**: TenantID + ApplicationID (ClientID) + Client Secret
- **Authorization**: Determines which Azure resources workload can access

#### Service Principal Properties
```
Service Principal:
├── Service Principal Object ID: 87654321-4321-4321-4321-210987654321
├── Application (client) ID: 12345678-1234-1234-1234-123456789abc (links to Application)
├── Display Name: "CI/CD Pipeline App"
├── Tenant ID: tenant123-4567-8901-2345-678901234567
├── Role Assignments:
│   ├── Subscription "Production": Contributor role
│   ├── Resource Group "rg-web-prod": Owner role
│   └── Key Vault "kv-prod-secrets": Secrets User role
└── Authentication: Client secret (password) or certificate
```

**Analogy**: Service Principal is the **local instance** of the application blueprint within your tenant

---

### 3. Managed Identity

**Definition**: Specialized service principal associated with Azure resources, featuring automatic credential management by Azure platform.

#### Characteristics
- **Scope**: Azure resource-specific (VM, Function App, ADF)
- **Credential Management**: **Automatic** (platform-managed, no secrets to rotate)
- **Lifecycle**: Coupled to Azure resource (system-assigned) or independent (user-assigned)
- **Authentication**: **Passwordless** (Azure handles token provisioning)
- **Purpose**: Eliminate credential storage for Azure-to-Azure authentication

#### Managed Identity Properties
```
Managed Identity (System-Assigned):
├── Principal ID: 11111111-2222-3333-4444-555555555555
├── Tenant ID: tenant123-4567-8901-2345-678901234567
├── Associated Resource: Azure Data Factory "adf-prod-etl"
├── Lifecycle: Created/deleted with parent resource
├── Role Assignments:
│   ├── SQL Database "sql-prod-db": SQL DB Contributor
│   └── Storage Account "stproddata": Storage Blob Data Contributor
└── Authentication: Azure Instance Metadata Service (IMDS) token endpoint
```

**Analogy**: Managed Identity is a **service principal with platform-managed credentials**

---

## Workload Identity Type Comparison

| Feature | Application | Service Principal | Managed Identity |
|---------|------------|-------------------|------------------|
| **Definition** | Global app blueprint | Tenant-local instance | Azure resource identity |
| **Scope** | Multi-tenant | Tenant-specific | Resource-specific |
| **Credential Management** | Manual (client secrets) | Manual (client secrets) | **Automatic** (platform-managed) |
| **Lifetime** | Independent | Independent | Coupled (system) / Independent (user) |
| **Use Case** | Multi-tenant SaaS | Azure Pipelines | Azure service-to-service auth |
| **Authentication** | TenantID + AppID + Secret | TenantID + AppID + Secret | **Passwordless** (IMDS token) |
| **Rotation Overhead** | High (90-day secrets) | High (90-day secrets) | **None** (auto-rotated) |
| **Azure Resource Access** | Yes (via service principal) | Yes | Yes |
| **Cross-Tenant** | Yes (multi-tenant app) | No (single tenant) | No (single tenant) |

---

## Service Principal Implementation

### Step 1: Application Registration

**Azure Portal**: Entra ID → App registrations → New registration

#### Configuration Options
```
Register an Application:
├── Name: "AzurePipelines-ServiceConnection"
├── Supported account types:
│   ├── ○ Accounts in this organizational directory only (Contoso - Single tenant)
│   ├── ○ Accounts in any organizational directory (Any Microsoft Entra directory - Multi-tenant)
│   ├── ○ Accounts in any organizational directory + personal Microsoft accounts
│   └── ○ Personal Microsoft accounts only
├── Redirect URI (optional): https://dev.azure.com/contoso (for interactive flows)
└── [Register]

Result:
    ├── Application (client) ID: 12345678-1234-1234-1234-123456789abc
    ├── Directory (tenant) ID: tenant123-4567-8901-2345-678901234567
    └── Object ID: abcdef12-3456-7890-abcd-ef1234567890
```

**Tenant Scope Selection**:

**Single Tenant** (Most Common):
- Use Case: Internal CI/CD pipelines, corporate applications
- Security: Application only authenticates within your Entra tenant
- Recommendation: ✅ Default choice for DevOps workflows

**Multi-Tenant**:
- Use Case: SaaS applications serving multiple customer tenants
- Security: Customers authorize app in their own tenants
- Complexity: Requires consent framework, cross-tenant permissions

---

### Step 2: Client Secret Generation

**App Registrations → Your App → Certificates & secrets → New client secret**

#### Client Secret Configuration
```
Add a Client Secret:
├── Description: "Azure Pipelines Production"
├── Expires:
│   ├── ○ 6 months
│   ├── ● 12 months (recommended for automation)
│   ├── ○ 24 months
│   └── ○ Custom (max 24 months)
└── [Add]

Result:
    ├── Secret ID: 87654321-4321-4321-4321-210987654321
    ├── Value: "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz~" ⚠️ Copy immediately
    ├── Description: "Azure Pipelines Production"
    └── Expires: 2025-01-14
```

**Critical**: Secret value displayed **once only**. Store securely immediately (Azure Key Vault, password manager)

**Security Recommendations**:
- ✅ **12-month expiration**: Balance security (rotation) vs operational overhead
- ✅ **Descriptive names**: "Azure Pipelines Production" not "Secret1"
- ✅ **Rotation calendar**: Set reminder 30 days before expiration
- ✅ **Secure storage**: Azure Key Vault for production secrets
- ❌ **Never commit secrets**: Scan with git-secrets, detect-secrets
- ❌ **Avoid 24-month expiration**: Excessive security risk

---

### Step 3: Grant Permissions

**Assign Role to Service Principal**:

#### Azure Portal Method
```
Azure Subscription → Access control (IAM) → Add role assignment

Role Assignment:
├── Role: Contributor (full resource management)
├── Assign access to: User, group, or service principal
├── Members: Select "AzurePipelines-ServiceConnection" (service principal)
└── [Review + assign]

Effective Permissions:
    └── Subscription "Production" (Contributor)
        ├── Create/delete/modify resources
        ├── Read resource configurations
        └── Cannot grant permissions to others (requires Owner role)
```

#### Azure CLI Method
```bash
# Get service principal object ID
az ad sp list --display-name "AzurePipelines-ServiceConnection" --query "[].id" -o tsv

# Assign Contributor role at subscription scope
az role assignment create \
  --assignee <service-principal-object-id> \
  --role Contributor \
  --scope /subscriptions/<subscription-id>

# Assign specific role at resource group scope (least privilege)
az role assignment create \
  --assignee <service-principal-object-id> \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/<subscription-id>/resourceGroups/rg-prod-data
```

#### Common Role Assignments

**Subscription-Level**:
- `Contributor`: Full resource management (create/delete/modify), **cannot** grant permissions
- `Reader`: Read-only resource visibility
- `Owner`: Full access including permission management (⚠️ use sparingly)

**Resource Group-Level**:
- `Contributor`: Manage all resources within resource group
- `Reader`: View resources in resource group
- `Web Plan Contributor`: Manage App Service plans only

**Resource-Specific**:
- `Storage Blob Data Contributor`: Read/write blob storage
- `Key Vault Secrets User`: Read secrets from Key Vault
- `SQL DB Contributor`: Manage SQL databases

**Least-Privilege Principle**: Grant minimum necessary permissions
```
❌ Too Broad: Subscription Owner (full control)
✅ Appropriate: Resource Group Contributor (project-scoped)
✅ Best: Specific resource "Storage Blob Data Contributor" (least privilege)
```

---

### Step 4: Service Connection Requirements

**Azure Pipelines Service Connection Configuration**:

#### Required Values
```
Service Connection (Azure Resource Manager):
├── Tenant ID: tenant123-4567-8901-2345-678901234567
├── Subscription ID: sub456-7890-1234-5678-901234567890
├── Subscription Name: "Production"
├── Application (client) ID: 12345678-1234-1234-1234-123456789abc
├── Client Secret: "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz~"
└── Service Connection Name: "Azure-Production-ServicePrincipal"
```

#### Azure DevOps Configuration
```
Azure DevOps Organization → Project Settings → Service connections → New service connection

Service Connection Type: Azure Resource Manager
Authentication Method: Service principal (manual)

Configuration:
├── Environment: Azure Cloud
├── Scope Level: Subscription
├── Subscription ID: sub456-7890-1234-5678-901234567890
├── Subscription Name: Production
├── Service Principal ID: 12345678-1234-1234-1234-123456789abc
├── Service Principal Key: <paste client secret>
├── Tenant ID: tenant123-4567-8901-2345-678901234567
├── Service connection name: "Azure-Production-ServicePrincipal"
├── Description: "Production Azure subscription for pipeline deployments"
└── Security: ✅ Grant access permission to all pipelines (or restrict per-pipeline)

Verify Connection → [Save]
```

---

## Service Principal Usage in Pipelines

### Azure Pipelines YAML

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  azureServiceConnection: 'Azure-Production-ServicePrincipal'
  resourceGroupName: 'rg-prod-webapp'
  location: 'eastus'
  webAppName: 'webapp-prod-contoso'

stages:
- stage: Deploy
  displayName: 'Deploy to Production'
  jobs:
  - job: DeployWebApp
    displayName: 'Deploy Web Application'
    steps:
    - task: AzureResourceManagerTemplateDeployment@3
      displayName: 'Deploy ARM Template'
      inputs:
        azureResourceManagerConnection: $(azureServiceConnection)
        subscriptionId: 'sub456-7890-1234-5678-901234567890'
        resourceGroupName: $(resourceGroupName)
        location: $(location)
        csmFile: 'infrastructure/webapp.json'
        csmParametersFile: 'infrastructure/webapp.parameters.json'
        deploymentMode: 'Incremental'
    
    - task: AzureWebApp@1
      displayName: 'Deploy Application Code'
      inputs:
        azureSubscription: $(azureServiceConnection)
        appName: $(webAppName)
        package: '$(System.ArtifactsDirectory)/**/*.zip'
        deploymentMethod: 'auto'
```

**Authentication Flow**:
```
1. Pipeline Executes: Azure Pipelines agent starts
2. Retrieve Credentials: Fetch TenantID + AppID + Secret from service connection
3. Authenticate: Obtain OAuth 2.0 access token from Entra ID
   POST https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/token
   Body: client_id={appId}&client_secret={secret}&scope=https://management.azure.com/.default
4. Access Azure: Use token to call Azure Resource Manager APIs
   Authorization: Bearer <access_token>
5. Deploy Resources: Create/update Azure resources (web app, storage, etc.)
```

---

### GitHub Actions Workflow (Traditional Service Principal)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Azure

on:
  push:
    branches: [main]

env:
  AZURE_WEBAPP_NAME: webapp-prod-contoso
  RESOURCE_GROUP: rg-prod-webapp

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Azure Login (Service Principal)
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Deploy ARM Template
        uses: azure/arm-deploy@v1
        with:
          subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          resourceGroupName: ${{ env.RESOURCE_GROUP }}
          template: ./infrastructure/webapp.json
          parameters: ./infrastructure/webapp.parameters.json
      
      - name: Deploy Application
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          package: './app.zip'
      
      - name: Azure Logout
        run: az logout
```

**GitHub Secrets Configuration**:
```
Repository → Settings → Secrets and variables → Actions → New repository secret

AZURE_CREDENTIALS (JSON format):
{
  "clientId": "12345678-1234-1234-1234-123456789abc",
  "clientSecret": "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz~",
  "subscriptionId": "sub456-7890-1234-5678-901234567890",
  "tenantId": "tenant123-4567-8901-2345-678901234567"
}

AZURE_SUBSCRIPTION_ID:
sub456-7890-1234-5678-901234567890
```

---

## Service Principal Lifecycle Management

### Credential Rotation

**Challenge**: Client secrets expire (6/12/24 months)

**Rotation Workflow**:
```
Month 10 (2 months before 12-month expiration):
1. Generate new client secret:
   - Entra ID → App registrations → Your app → Certificates & secrets → New client secret
   - Description: "Azure Pipelines Production v2"
   - Expires: 12 months
   - Copy secret value immediately

2. Update service connections:
   - Azure DevOps → Project Settings → Service connections → Edit
   - Update "Service Principal Key" field
   - Verify connection → Save

3. Update GitHub secrets:
   - GitHub repository → Settings → Secrets → Edit AZURE_CREDENTIALS
   - Update "clientSecret" value
   - Save

4. Test deployments:
   - Trigger test pipeline run
   - Verify authentication success
   - Monitor for errors

5. Revoke old secret (Month 12):
   - Entra ID → App registrations → Certificates & secrets
   - Delete old secret (grace period complete)
```

**Automation Recommendations**:
- ✅ **Calendar reminders**: 30-day advance notice
- ✅ **Monitoring alerts**: Detect authentication failures
- ✅ **Multiple secrets**: Overlapping validity periods (blue-green rotation)
- ✅ **Key Vault integration**: Centralized secret management

---

### Security Audit

**Regular Reviews**:
```
Quarterly Service Principal Audit:
├── List all service principals: az ad sp list --all
├── Identify unused service principals: No sign-in > 90 days
├── Review role assignments: az role assignment list --assignee <sp-object-id>
├── Validate least-privilege: Downgrade excessive permissions
├── Check secret expiration: Alert for < 30 days remaining
└── Document usage: Purpose, owner, expiration calendar
```

---

## Service Principal vs Managed Identity Decision Tree

```
Start: What authentication scenario?

├── Azure resource authenticating to Azure service
│   ├── Single Azure resource (e.g., VM → Key Vault)
│   │   └── ✅ Use: System-Assigned Managed Identity
│   └── Multiple Azure resources (e.g., 5 VMs → Storage Account)
│       └── ✅ Use: User-Assigned Managed Identity
│
├── Azure Pipelines deploying Azure resources
│   ├── Modern approach (recommended)
│   │   └── ✅ Use: Workload Identity Federation (Managed Identity)
│   └── Traditional approach
│       └── ⚠️ Use: Service Principal (client secret)
│
├── GitHub Actions deploying Azure resources
│   ├── Modern approach (recommended)
│   │   └── ✅ Use: Workload Identity Federation (Managed Identity)
│   └── Traditional approach
│       └── ⚠️ Use: Service Principal (client secret)
│
├── Multi-tenant SaaS application
│   └── ⚠️ Use: Multi-tenant Application + Service Principals (per tenant)
│
└── Non-Azure automation (Jenkins, GitLab CI)
    └── ⚠️ Use: Service Principal (only option)
```

**Recommendation Priority**:
1. **Managed Identity** (passwordless, zero rotation overhead)
2. **Workload Identity Federation** (eliminates secrets for CI/CD)
3. **Service Principal with Certificates** (better than secrets)
4. **Service Principal with Client Secrets** (last resort, high overhead)

---

## Common Pitfalls

### 1. Excessive Permissions
```
❌ Anti-Pattern: Grant subscription-level Owner role to service principal
Risk: Full control including permission management

✅ Best Practice: Grant resource group-level Contributor role
Benefit: Scoped permissions, cannot escalate privileges
```

### 2. Hardcoded Secrets
```
❌ Anti-Pattern: Store client secret in pipeline YAML
Risk: Secret exposure in source control

✅ Best Practice: Store in Azure DevOps Library/GitHub Secrets
Benefit: Encrypted at rest, access logging
```

### 3. Forgotten Secret Expiration
```
❌ Anti-Pattern: No expiration tracking
Risk: Pipeline failures when secret expires

✅ Best Practice: 30-day advance alerts + rotation workflow
Benefit: Zero-downtime secret rotation
```

### 4. Single Service Principal for All Environments
```
❌ Anti-Pattern: Same service principal for dev/staging/production
Risk: Dev pipeline can accidentally modify production

✅ Best Practice: Separate service principals per environment
Benefit: Blast radius containment, principle of least privilege
```

---

## Quick Reference

### Service Principal Creation (Azure CLI)
```bash
# Create service principal
az ad sp create-for-rbac \
  --name "AzurePipelines-ServiceConnection" \
  --role Contributor \
  --scopes /subscriptions/<subscription-id>/resourceGroups/rg-prod \
  --sdk-auth

# Output (save securely):
{
  "clientId": "12345678-1234-1234-1234-123456789abc",
  "clientSecret": "abc123def456ghi789...",
  "subscriptionId": "sub456-7890-1234-5678-901234567890",
  "tenantId": "tenant123-4567-8901-2345-678901234567"
}
```

### Service Principal Authentication (PowerShell)
```powershell
# Authenticate as service principal
$clientSecret = ConvertTo-SecureString "abc123def456..." -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential("12345678-1234-1234-1234-123456789abc", $clientSecret)

Connect-AzAccount -ServicePrincipal -Credential $credential -Tenant "tenant123-4567-8901-2345-678901234567"

# Deploy resources
New-AzResourceGroupDeployment `
  -ResourceGroupName "rg-prod" `
  -TemplateFile "webapp.json"
```

---

## Next Steps

✅ **Completed**: Workload identities and service principal provisioning

**Next Unit**: Learn about managed identities for passwordless authentication →

---

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-identity-management-systems/5-explore-service-principals)
