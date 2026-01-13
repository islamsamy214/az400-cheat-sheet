# Examine Task Groups

⏱️ **Duration**: ~3 minutes | 📚 **Type**: Conceptual

## Overview

Task groups provide **task sequence encapsulation**, enabling the transformation of multiple pipeline tasks into single, reusable components for build and release pipeline integration. Learn how to create, manage, and version task groups for standardized deployment workflows.

## Learning Objectives

After completing this unit, you'll be able to:
- ✅ Understand task group architecture and benefits
- ✅ Create task groups from existing pipeline tasks
- ✅ Extract parameters for configuration flexibility
- ✅ Manage task group versions and updates
- ✅ Apply task groups across multiple pipelines

---

## What Are Task Groups?

**Definition**: A task group is an **encapsulated sequence of tasks** that can be added to build and release pipelines as a single reusable unit.

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Encapsulation** | Multiple tasks treated as one logical unit |
| **Parameterization** | Extract configuration as variables |
| **Reusability** | Use across unlimited pipelines |
| **Versioning** | Track changes and roll back if needed |
| **Auto-Propagation** | Updates apply to all consuming pipelines |

---

## How Task Groups Work

### Before Task Groups (Duplication)

**Pipeline 1**:
```
Tasks:
  1. Azure Web App Deploy
     - Azure Subscription: Production-Sub
     - App Name: app1-prod
  2. Azure CLI
     - Script: az webapp restart
  3. Azure App Service Manage
     - Action: Swap slots
```

**Pipeline 2**:
```
Tasks:
  1. Azure Web App Deploy (duplicated)
  2. Azure CLI (duplicated)
  3. Azure App Service Manage (duplicated)
```

**Problem**: Same 3 tasks copied to 50+ pipelines!

### After Task Groups (Reuse)

**Task Group**: "Deploy and Swap Azure Web App"
```
Parameters:
  - azureSubscription
  - appName
  - deploymentSlot

Tasks:
  1. Azure Web App Deploy
  2. Azure CLI (restart)
  3. Azure App Service Manage (swap)
```

**Pipeline 1**:
```
Tasks:
  1. Deploy and Swap Azure Web App
     - azureSubscription: $(productionSubscription)
     - appName: app1-prod
     - deploymentSlot: staging
```

**Pipeline 2**:
```
Tasks:
  1. Deploy and Swap Azure Web App (same task group)
     - azureSubscription: $(productionSubscription)
     - appName: app2-prod
     - deploymentSlot: staging
```

**Benefit**: Update task group once → propagates to all 50+ pipelines automatically!

---

## Creating a Task Group

### Method 1: From Existing Pipeline Tasks

#### Step 1: Select Tasks in Existing Pipeline

**Azure DevOps UI**:
```
Pipelines → Releases → [Your Pipeline] → Edit

In the stage tasks:
1. Select the tasks you want to group (Ctrl+Click for multiple)
2. Right-click → Create task group
```

#### Step 2: Configure Task Group

**Task Group Configuration Dialog**:
```
Name: Deploy to Azure Web App
Description: Deploys application to Azure App Service with slot swap
Category: Deploy

Parameters (auto-detected from tasks):
✓ azureSubscription (string)
✓ appName (string)
✓ packagePath (string)

Tasks included (read-only preview):
1. Azure Web App Deploy
2. Azure CLI (restart webapp)
3. Azure App Service Manage (swap slots)
```

#### Step 3: Save Task Group

**Result**: Task group created and available in:
- Current pipeline (tasks replaced with task group reference)
- Task catalog for other pipelines
- Organization library

### Method 2: Create from Scratch

**Azure DevOps UI**:
```
Pipelines → Task groups → New task group

1. Name: "Database Migration"
2. Add tasks:
   - Azure SQL Database Deployment
   - Run SQL Scripts
   - Backup Database
3. Extract parameters (connection strings, database names)
4. Save
```

---

## Parameter Extraction

### What Are Parameters?

**Parameters** are placeholders that enable **configuration variable abstraction**, separating task logic from environment-specific values.

### Example: Before Parameter Extraction

**Task Group** (hardcoded values):
```yaml
tasks:
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Production-Subscription'  # Hardcoded!
    appName: 'myapp-prod'                          # Hardcoded!
    package: '$(System.DefaultWorkingDirectory)/**/*.zip'
```

**Problem**: Can't reuse for different subscriptions or app names!

### Example: After Parameter Extraction

**Task Group Definition** (parameterized):
```yaml
parameters:
- name: azureSubscription
  type: string
  displayName: 'Azure Subscription'
- name: appName
  type: string
  displayName: 'Application Name'
- name: packagePath
  type: string
  displayName: 'Package Path'
  default: '$(System.DefaultWorkingDirectory)/**/*.zip'

tasks:
- task: AzureWebApp@1
  inputs:
    azureSubscription: '$(azureSubscription)'  # Parameter reference
    appName: '$(appName)'                      # Parameter reference
    package: '$(packagePath)'                  # Parameter reference
```

**Pipeline Usage** (provides parameter values):
```yaml
- task: DeployToAzureWebApp@1  # Task group reference
  displayName: 'Deploy to Production'
  inputs:
    azureSubscription: '$(productionSubscription)'
    appName: 'myapp-prod'
    packagePath: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

**Benefit**: Same task group works for:
- Different subscriptions
- Different app names
- Different package locations
- Different environments

---

## Task Group Versioning

### Why Versioning Matters

**Scenario**: You update a task group used by 50 pipelines. What if the change breaks something?

**Without Versioning**: All 50 pipelines immediately use the new (broken) version → 50 failures!

**With Versioning**: Pipelines pin to specific versions → gradual rollout possible

### Version Management

**Task Group Versions**:
```
Version 1.0.0 (Initial release)
├── Tasks:
│   ├── Azure Web App Deploy
│   └── Azure CLI (restart)

Version 1.1.0 (Added slot swap)
├── Tasks:
│   ├── Azure Web App Deploy
│   ├── Azure CLI (restart)
│   └── Azure App Service Manage (swap slots)  ← New task

Version 2.0.0 (Breaking change: new parameter)
├── Parameters:
│   ├── azureSubscription
│   ├── appName
│   └── deploymentSlot  ← New required parameter
├── Tasks:
│   ├── Azure Web App Deploy
│   ├── Azure CLI (restart)
│   └── Azure App Service Manage (swap slots)
```

### Version Pinning in Pipelines

**Option 1: Use Latest Version** (default)
```yaml
- task: DeployToAzureWebApp@*  # Always uses latest
  inputs:
    azureSubscription: '$(subscription)'
```

**Option 2: Pin to Specific Version**
```yaml
- task: DeployToAzureWebApp@1  # Always uses v1.x.x
  inputs:
    azureSubscription: '$(subscription)'
```

**Option 3: Pin to Exact Version**
```yaml
- task: DeployToAzureWebApp@1.1.0  # Exact version
  inputs:
    azureSubscription: '$(subscription)'
```

**Best Practice**: Use latest in dev/test, pin to specific major version in production

---

## Auto-Propagation of Changes

### How Auto-Propagation Works

**Scenario**: You have 50 pipelines using "Deploy to Azure Web App" task group (v1.0.0)

**Update**: Add health check task to task group → save as v1.1.0

**Result**:
- Pipelines using `@*` (latest): Immediately use v1.1.0 (health check included)
- Pipelines using `@1`: Automatically upgrade to v1.1.0 (minor version change)
- Pipelines using `@1.0.0`: Stay on v1.0.0 (explicit pin)

**Propagation Matrix**:

| Pipeline Version Ref | Task Group v1.0.0 | Task Group v1.1.0 | Task Group v2.0.0 |
|---------------------|-------------------|-------------------|-------------------|
| `@*` (latest) | Uses 1.0.0 | ✅ **Auto-upgrades** to 1.1.0 | ✅ **Auto-upgrades** to 2.0.0 |
| `@1` (major) | Uses 1.0.0 | ✅ **Auto-upgrades** to 1.1.0 | ❌ Stays on 1.1.0 |
| `@1.0.0` (exact) | Uses 1.0.0 | ❌ Stays on 1.0.0 | ❌ Stays on 1.0.0 |

---

## Task Group Best Practices

### 1. Naming Conventions

**Format**: `[Category] - [Action] - [Target]`

**Examples**:
- ✅ `Deploy - Azure Web App - Blue-Green`
- ✅ `Test - Run Selenium - Chrome`
- ✅ `Build - .NET Core - Multi-Target`
- ❌ `MyTaskGroup` (too generic)
- ❌ `Deploy` (not descriptive)

### 2. Parameter Design

**Good Parameters** (flexible, reusable):
```yaml
parameters:
- name: azureSubscription  # Connection to Azure
  type: string
- name: resourceGroup  # Scoping
  type: string
- name: appName  # Specific resource
  type: string
- name: deploymentSlot  # Configuration
  type: string
  default: 'production'
- name: enableHealthCheck  # Feature flag
  type: boolean
  default: true
```

**Bad Parameters** (hardcoded, inflexible):
```yaml
parameters:
- name: deployToProduction  # Boolean limits flexibility
  type: boolean
- name: environmentName  # "dev"|"prod" - what about QA, staging?
  type: string
  values: ['dev', 'prod']
```

### 3. Documentation

**Include in Description**:
- Purpose of the task group
- Required parameters
- Optional parameters and defaults
- Prerequisites (e.g., service connections)
- Usage examples
- Version history (what changed)

**Example**:
```
Name: Deploy to Azure Web App

Description:
Deploys application to Azure App Service with blue-green deployment strategy.

Prerequisites:
- Azure Resource Manager service connection
- App Service with deployment slots configured

Required Parameters:
- azureSubscription: Azure RM connection name
- appName: Name of the App Service
- packagePath: Path to deployment package (*.zip)

Optional Parameters:
- deploymentSlot: Target slot (default: 'staging')
- enableHealthCheck: Run health check after deployment (default: true)

Version History:
- v1.0.0: Initial release (deploy + restart)
- v1.1.0: Added slot swap functionality
- v1.2.0: Added health check step
- v2.0.0: Breaking change - removed appSettings parameter (use variable groups instead)
```

### 4. Testing Strategy

**Always Test Before Rollout**:
```
1. Create new version of task group (v1.2.0-beta)
2. Clone existing pipeline → test pipeline
3. Update test pipeline to use v1.2.0-beta
4. Run test pipeline against Dev environment
5. If successful:
   - Promote v1.2.0-beta → v1.2.0
   - Gradually roll out to production pipelines
6. If failed:
   - Fix issues
   - Repeat from step 1
```

### 5. Versioning Strategy

**Semantic Versioning** (MAJOR.MINOR.PATCH):
```
MAJOR: Breaking changes (new required parameters, removed tasks)
MINOR: Backward-compatible additions (new optional parameters, new tasks)
PATCH: Bug fixes (no functional changes)

Examples:
- v1.0.0 → v1.0.1: Fixed script typo (PATCH)
- v1.0.1 → v1.1.0: Added health check (MINOR)
- v1.1.0 → v2.0.0: Removed appSettings parameter (MAJOR)
```

---

## Limitations and Workarounds

### ❌ Limitation 1: Task Groups Not Supported in YAML

**Problem**: Task groups only work with **Classic Pipelines** (UI-based)

**Workaround**: Use **YAML Templates** for YAML pipelines

**YAML Template Example**:
```yaml
# File: templates/deploy-webapp.yml
parameters:
- name: azureSubscription
  type: string
- name: appName
  type: string
- name: packagePath
  type: string

steps:
- task: AzureWebApp@1
  inputs:
    azureSubscription: '${{ parameters.azureSubscription }}'
    appName: '${{ parameters.appName }}'
    package: '${{ parameters.packagePath }}'

- task: AzureCLI@2
  inputs:
    azureSubscription: '${{ parameters.azureSubscription }}'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az webapp restart --name ${{ parameters.appName }}
```

**Usage in Pipeline**:
```yaml
# azure-pipelines.yml
stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - template: templates/deploy-webapp.yml
      parameters:
        azureSubscription: '$(azureSubscription)'
        appName: 'myapp-prod'
        packagePath: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

### ❌ Limitation 2: Cannot Nest Task Groups

**Problem**: Task groups cannot contain other task groups

**Workaround**: Flatten task groups or use YAML templates with includes

---

## Quick Reference

| Feature | Classic Pipelines | YAML Pipelines |
|---------|------------------|----------------|
| **Task Groups** | ✅ Supported | ❌ Not supported |
| **Templates** | ❌ Not available | ✅ Supported |
| **Versioning** | ✅ Built-in | ⚠️ File-based (Git) |
| **UI Creation** | ✅ Yes | ❌ No (code only) |
| **Parameter Types** | String, Boolean, PickList | String, Boolean, Object, Array |

**Migration Path**:
- Classic Pipeline + Task Groups → YAML Pipeline + Templates
- Export task group logic → convert to YAML template

---

## Key Takeaways

- 🔄 **Task groups** encapsulate multiple tasks into reusable components
- 📝 **Parameters** enable configuration flexibility and reusability
- 🔢 **Versioning** allows safe updates with gradual rollout
- ⚡ **Auto-propagation** applies changes across all consuming pipelines
- 🎯 **Classic Pipelines** only - use YAML templates for YAML pipelines
- 🏗️ **Standardization** eliminates duplication and ensures consistency
- 🛠️ **Centralized management** reduces maintenance overhead

---

## Next Steps

✅ **Completed**: Task group concepts and architecture

**Continue to**: Unit 3 - Explore variables in release pipelines

---

## Additional Resources

- [Task groups for builds and releases](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/task-groups)
- [YAML templates](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/template)
- [Template references](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates)

[↩️ Back to Module Overview](01-introduction.md) | [➡️ Next: Variables in Release Pipelines](03-explore-variables-release-pipelines.md)
