# Explore Variables in Release Pipelines

⏱️ **Duration**: ~4 minutes | 📚 **Type**: Conceptual

## Overview

Variables provide **configuration abstraction** for release pipelines, enabling environment-specific deployments without code duplication. Master predefined system variables, release/stage scoping, variable groups, and secret management for secure, flexible pipeline configuration.

## Learning Objectives

After completing this unit, you'll be able to:
- ✅ Understand variable types and scoping rules
- ✅ Use predefined system variables in pipelines
- ✅ Configure release and stage variables
- ✅ Implement variable groups for cross-pipeline sharing
- ✅ Secure sensitive data with secret variables
- ✅ Apply variable precedence hierarchy

---

## What Are Variables?

**Definition**: Variables are **name-value pairs** that store configuration data for use across pipeline tasks, enabling parameterization and environment-specific behavior.

### Why Use Variables?

**Without Variables** (hardcoded):
```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Production-Subscription'  # Hardcoded
    appName: 'myapp-prod'                          # Hardcoded
    resourceGroup: 'prod-rg'                       # Hardcoded
```

**With Variables** (flexible):
```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: '$(azureSubscription)'  # Variable reference
    appName: '$(appName)'                      # Variable reference
    resourceGroup: '$(resourceGroup)'          # Variable reference
```

**Benefits**:
- ✅ **Reusability**: Same pipeline for dev, test, prod
- ✅ **Security**: Separate secrets from code
- ✅ **Maintainability**: Update values without editing pipeline YAML
- ✅ **Flexibility**: Override values at runtime

---

## Variable Types

### 1. Predefined Variables (System Variables)

**Definition**: Built-in variables automatically provided by Azure DevOps for every pipeline run.

#### Common System Variables

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `$(Build.BuildId)` | Unique build identifier | `12345` |
| `$(Build.SourceBranch)` | Source branch path | `refs/heads/main` |
| `$(Build.SourceBranchName)` | Branch name only | `main` |
| `$(Build.Repository.Name)` | Repository name | `MyApp` |
| `$(Build.ArtifactStagingDirectory)` | Artifact staging folder | `/home/vsts/work/1/a` |
| `$(Release.ReleaseName)` | Release name | `Release-123` |
| `$(Release.ReleaseId)` | Unique release identifier | `789` |
| `$(Release.EnvironmentName)` | Current stage name | `Production` |
| `$(Agent.WorkFolder)` | Agent working directory | `/home/vsts/work` |
| `$(System.DefaultWorkingDirectory)` | Default working directory | `/home/vsts/work/1/s` |

#### Usage Example

**Accessing Build Artifacts**:
```yaml
steps:
- task: DownloadBuildArtifacts@0
  inputs:
    buildType: 'current'
    downloadType: 'single'
    artifactName: 'drop'
    downloadPath: '$(System.ArtifactsDirectory)'
```

**Dynamic Naming**:
```yaml
steps:
- task: AzureWebApp@1
  inputs:
    appName: 'myapp-$(Build.SourceBranchName)'  # myapp-main
```

**Full List**: [Predefined variables documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables)

---

### 2. Release Pipeline Variables (Global Scope)

**Definition**: Custom variables defined at the **release pipeline level**, available to all stages.

#### Creating Pipeline Variables

**Azure DevOps UI**:
```
Pipelines → Releases → [Your Release] → Edit
→ Variables tab → Pipeline variables section
→ + Add

Name: azureSubscription
Value: Production-Subscription
Scope: Release
```

**YAML Equivalent**:
```yaml
variables:
  azureSubscription: 'Production-Subscription'
  appName: 'myapp'
  resourceGroup: 'prod-rg'

stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - task: AzureWebApp@1
      inputs:
        azureSubscription: '$(azureSubscription)'  # References pipeline variable
        appName: '$(appName)'
```

**Characteristics**:
- ✅ **Scope**: All stages in the release pipeline
- ✅ **Override**: Can be overridden by stage variables
- ✅ **Persistence**: Stored in pipeline definition
- ✅ **Visibility**: Available in all tasks and scripts

---

### 3. Stage Variables (Stage Scope)

**Definition**: Variables scoped to a **specific stage**, overriding pipeline-level variables with the same name.

#### Creating Stage Variables

**Azure DevOps UI**:
```
Pipelines → Releases → [Your Release] → Edit
→ Stage → [Select Stage] → Variables tab
→ + Add

Name: appName
Value: myapp-dev
Scope: Development (stage)
```

**YAML Equivalent**:
```yaml
stages:
- stage: Development
  variables:
    appName: 'myapp-dev'
    resourceGroup: 'dev-rg'
  jobs:
  - job: DeployJob
    steps:
    - script: echo "Deploying to $(appName)"  # Uses stage variable

- stage: Production
  variables:
    appName: 'myapp-prod'
    resourceGroup: 'prod-rg'
  jobs:
  - job: DeployJob
    steps:
    - script: echo "Deploying to $(appName)"  # Uses different stage variable
```

**Output**:
```
Development stage: Deploying to myapp-dev
Production stage: Deploying to myapp-prod
```

---

### 4. Variable Groups

**Definition**: **Shared collections of variables** stored in Azure DevOps Library, reusable across multiple pipelines.

#### Why Use Variable Groups?

**Scenario**: 50 pipelines all need the same Azure subscription connection details.

**Without Variable Groups** (duplication):
```
Pipeline 1: azureSubscription = 'Production-Sub'
Pipeline 2: azureSubscription = 'Production-Sub' (duplicated)
Pipeline 3: azureSubscription = 'Production-Sub' (duplicated)
... (50 times)
```

**Problem**: Subscription name changes → update 50 pipelines! 🚨

**With Variable Groups** (centralized):
```
Variable Group: "Azure-Production"
├── azureSubscription: 'Production-Sub'
├── resourceGroup: 'prod-rg'
└── location: 'eastus'

Pipeline 1: Links to "Azure-Production" group
Pipeline 2: Links to "Azure-Production" group
Pipeline 3: Links to "Azure-Production" group
... (all link to same group)
```

**Benefit**: Update subscription once → propagates to all 50 pipelines! ✅

#### Creating Variable Groups

**Azure DevOps UI**:
```
Pipelines → Library → + Variable group

Name: Azure-Production
Description: Production Azure configuration

Variables:
  azureSubscription: Production-Subscription
  resourceGroup: prod-rg
  location: eastus
  appServicePlan: prod-asp

Save
```

#### Using Variable Groups in Pipelines

**Classic Pipelines**:
```
Pipelines → Releases → [Your Release] → Edit
→ Variables tab → Variable groups section
→ Link variable group → Select "Azure-Production"
```

**YAML Pipelines**:
```yaml
variables:
- group: Azure-Production  # Links variable group

stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - task: AzureWebApp@1
      inputs:
        azureSubscription: '$(azureSubscription)'  # From variable group
        resourceGroup: '$(resourceGroup)'          # From variable group
```

**Multiple Groups**:
```yaml
variables:
- group: Azure-Production
- group: Database-Connection-Strings
- group: API-Keys

stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - script: |
        echo "Azure Sub: $(azureSubscription)"        # From Azure-Production
        echo "DB Connection: $(databaseConnection)"   # From Database-Connection-Strings
        echo "API Key: $(apiKey)"                     # From API-Keys
```

---

## Normal vs. Secret Variables

### Normal Variables

**Definition**: Standard variables with **visible values** in logs and UI.

**Creation**:
```
Variable name: appName
Value: myapp-prod
Type: Normal (default)
```

**Log Output**:
```
Deploying to myapp-prod  ✅ Value visible
```

**Use Cases**:
- Application names
- Resource group names
- Regions/locations
- Non-sensitive configuration

---

### Secret Variables

**Definition**: Encrypted variables with **masked values** in logs to protect sensitive data.

**Creation**:
```
Variable name: apiKey
Value: abc123xyz789  ← Entered but not shown
Type: Secret (toggle padlock icon 🔒)
```

**Log Output**:
```
Using API Key: ***  ✅ Value masked
```

**Characteristics**:
- 🔒 **Encrypted at rest**: Stored encrypted in Azure DevOps
- 👁️ **Hidden in UI**: Value shows as dots or asterisks
- 📝 **Masked in logs**: Automatically replaced with `***`
- ⚠️ **Not accessible via scripts**: Cannot echo or print to logs

#### Secret Variable Limitations

**Problem**: Cannot directly access secret variable values in scripts

**Example - Won't Work**:
```yaml
variables:
  apiKey: 'abc123xyz789'  # Marked as secret

steps:
- script: |
    echo "API Key: $(apiKey)"  # Output: API Key: ***
    curl -H "Authorization: $(apiKey)" https://api.example.com  # Fails!
```

**Workaround - Use Environment Variables**:
```yaml
variables:
  apiKey: 'abc123xyz789'  # Marked as secret

steps:
- script: |
    curl -H "Authorization: $API_KEY" https://api.example.com  # Works!
  env:
    API_KEY: $(apiKey)  # Secret passed as environment variable
```

**Why This Works**: Environment variables are not logged by default

---

## Azure Key Vault Integration

### What Is Azure Key Vault Integration?

**Azure Key Vault** is Microsoft's cloud-based secret management service. Azure DevOps can **automatically sync secrets** from Key Vault into variable groups.

**Benefits**:
- 🔐 **Enterprise-grade security**: Hardware Security Modules (HSMs)
- 🔄 **Automatic sync**: Secrets updated in Key Vault → reflected in pipelines
- 🎯 **Centralized management**: One source of truth for all secrets
- 📊 **Audit logging**: Track who accessed secrets and when
- ✅ **Compliance**: Meet regulatory requirements (HIPAA, PCI-DSS)

### How It Works

#### Step 1: Store Secrets in Azure Key Vault

**Azure CLI**:
```bash
# Create Key Vault
az keyvault create \
  --name myapp-keyvault \
  --resource-group prod-rg \
  --location eastus

# Add secrets
az keyvault secret set \
  --vault-name myapp-keyvault \
  --name DatabaseConnectionString \
  --value "Server=prod-db.database.windows.net;Database=mydb;User Id=admin;Password=***"

az keyvault secret set \
  --vault-name myapp-keyvault \
  --name ApiKey \
  --value "abc123xyz789"
```

#### Step 2: Create Azure DevOps Service Connection

**Azure DevOps UI**:
```
Project Settings → Service connections → New service connection
→ Azure Resource Manager → Service principal (automatic)

Connection name: KeyVault-Connection
Subscription: Production-Subscription
Resource group: prod-rg
```

#### Step 3: Link Key Vault to Variable Group

**Azure DevOps UI**:
```
Pipelines → Library → + Variable group

Name: Production-Secrets
Description: Production secrets from Azure Key Vault

☑️ Link secrets from an Azure key vault as variables
  Azure subscription: KeyVault-Connection
  Key vault name: myapp-keyvault

Add secrets:
  ☑️ DatabaseConnectionString
  ☑️ ApiKey

Save
```

#### Step 4: Use in Pipeline

**YAML**:
```yaml
variables:
- group: Production-Secrets  # Links Key Vault-backed variable group

stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'Production-Subscription'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          # Secrets automatically available as environment variables
          echo "Connecting to database..."
          sqlcmd -S prod-db.database.windows.net -d mydb -U admin -P $DB_PASSWORD
      env:
        DB_PASSWORD: $(DatabaseConnectionString)  # From Key Vault
        API_KEY: $(ApiKey)                        # From Key Vault
```

**Automatic Sync**: Changes in Key Vault propagate to pipelines (may take a few minutes)

---

## Variable Scoping and Precedence

### Variable Hierarchy

When multiple variables have the **same name**, Azure DevOps uses this precedence order (highest to lowest):

```
1. Job-level variables (highest priority)
   ↓ Overrides ↓
2. Stage-level variables
   ↓ Overrides ↓
3. Pipeline-level variables
   ↓ Overrides ↓
4. Variable group variables
   ↓ Overrides ↓
5. Predefined system variables (lowest priority)
```

### Precedence Example

**Scenario**: Variable `appName` defined at multiple levels

```yaml
# Pipeline-level variable
variables:
  appName: 'myapp-pipeline'  # Priority: 3

# Variable group
- group: AzureConfig  # Contains: appName = 'myapp-group' (Priority: 4)

stages:
- stage: Development
  variables:
    appName: 'myapp-dev'  # Priority: 2 (overrides pipeline + group)
  jobs:
  - job: DeployJob
    variables:
      appName: 'myapp-job'  # Priority: 1 (overrides all)
    steps:
    - script: echo "Deploying to $(appName)"  # Output: myapp-job
```

**Output**: `Deploying to myapp-job` (job-level wins)

### Scoping Best Practices

| Scope Level | Best For | Examples |
|-------------|----------|----------|
| **Job** | Task-specific overrides | Temporary test values |
| **Stage** | Environment-specific config | Dev/Test/Prod app names |
| **Pipeline** | Pipeline-wide defaults | Build configuration |
| **Variable Group** | Cross-pipeline shared values | Azure subscriptions, DB connections |
| **System** | Metadata, artifact paths | Build IDs, source branches |

---

## Variable Lifecycle

### 1. Definition

**Where Variables Are Created**:
- Pipeline YAML (`variables:` section)
- Azure DevOps UI (Variables tab)
- Variable groups (Library)
- Runtime (script output variables)

### 2. Resolution

**When Variables Are Resolved**:
```
Pipeline Start
  ↓
Load predefined variables (system)
  ↓
Load variable groups
  ↓
Load pipeline variables
  ↓
Load stage variables (when stage starts)
  ↓
Load job variables (when job starts)
  ↓
Resolve $(variableName) references in tasks
  ↓
Execute tasks
```

### 3. Output Variables (Runtime Creation)

**Create Variables During Pipeline Execution**:

**Example - Set Variable from Script**:
```yaml
steps:
- script: |
    echo "Computing version number..."
    VERSION="1.2.$(date +%Y%m%d)"
    echo "##vso[task.setvariable variable=appVersion]$VERSION"
  displayName: 'Generate version number'

- script: |
    echo "Deploying version $(appVersion)"  # Uses runtime-generated variable
  displayName: 'Display version'
```

**Output**:
```
Computing version number...
Deploying version 1.2.20240115
```

**Cross-Job Variables** (requires `isOutput=true`):
```yaml
jobs:
- job: BuildJob
  steps:
  - script: |
      echo "##vso[task.setvariable variable=buildNumber;isOutput=true]12345"
    name: setBuildNumber

- job: DeployJob
  dependsOn: BuildJob
  variables:
    buildNum: $[ dependencies.BuildJob.outputs['setBuildNumber.buildNumber'] ]
  steps:
  - script: echo "Deploying build $(buildNum)"  # Uses output from BuildJob
```

---

## Variable Security Best Practices

### 1. Never Store Secrets in Code

**❌ Bad Practice**:
```yaml
variables:
  databasePassword: 'MyP@ssw0rd123'  # Secret in YAML = security risk!
```

**✅ Good Practice**:
```yaml
variables:
- group: Database-Secrets  # Secrets in variable group (encrypted)
```

### 2. Use Azure Key Vault for Production

**❌ Bad Practice**: Store secrets directly in Azure DevOps variable groups

**✅ Good Practice**: Store secrets in Azure Key Vault, link to variable groups

### 3. Limit Variable Group Access

**Azure DevOps UI**:
```
Library → [Variable Group] → Security

Users/Groups:
  ✅ Developers: Reader (can use, cannot edit)
  ✅ DevOps Admins: Administrator (can edit)
  ❌ Public: No access
```

### 4. Audit Secret Access

**Azure Key Vault Diagnostic Settings**:
```
Enable logging for:
  ☑️ AuditEvent
  ☑️ AllMetrics

Send to:
  ☑️ Log Analytics workspace
  ☑️ Storage account (90-day retention)
```

**Query Logs**:
```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.KEYVAULT"
| where OperationName == "SecretGet"
| project TimeGenerated, CallerIPAddress, ResultSignature, Resource
| order by TimeGenerated desc
```

---

## Quick Reference

### Variable Syntax

| Syntax | Context | Example |
|--------|---------|---------|
| `$(variableName)` | Tasks, scripts (macro syntax) | `$(appName)` |
| `$[variables.variableName]` | YAML (runtime expression) | `$[variables.buildId]` |
| `$VARIABLE_NAME` | Bash scripts (environment variable) | `$APP_NAME` |
| `%VARIABLE_NAME%` | Windows CMD scripts | `%APP_NAME%` |
| `$env:VARIABLE_NAME` | PowerShell scripts | `$env:APP_NAME` |

### Common Predefined Variables

| Category | Variable | Use Case |
|----------|----------|----------|
| **Build** | `$(Build.BuildId)` | Unique build identifier |
| | `$(Build.SourceBranch)` | Branch being built |
| | `$(Build.Repository.Name)` | Repository name |
| **Release** | `$(Release.ReleaseName)` | Release identifier |
| | `$(Release.EnvironmentName)` | Current stage name |
| **Agent** | `$(Agent.WorkFolder)` | Agent working directory |
| | `$(Pipeline.Workspace)` | Pipeline workspace path |

---

## Key Takeaways

- 📝 **Variables** abstract configuration from pipeline logic
- 🌍 **Scoping** determines variable availability (pipeline, stage, job)
- 🔢 **Precedence** resolves conflicts (job > stage > pipeline > variable group > system)
- 🔒 **Secret variables** mask sensitive values in logs
- 🔐 **Azure Key Vault** provides enterprise secret management
- 🔄 **Variable groups** enable cross-pipeline sharing
- 🎯 **Predefined variables** provide build/release metadata

---

## Next Steps

✅ **Completed**: Variable types, scoping, and security

**Continue to**: Unit 4 - Exercise: Create and manage variable groups (hands-on lab)

---

## Additional Resources

- [Define variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables)
- [Predefined variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables)
- [Variable groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups)
- [Azure Key Vault integration](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-key-vault)

[↩️ Back to Module Overview](01-introduction.md) | [⬅️ Previous: Task Groups](02-examine-task-groups.md) | [➡️ Next: Exercise - Variable Groups](04-exercise-create-manage-variable-groups.md)
