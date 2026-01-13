# Introduction

## Module Overview

This module explores **task group creation**, **variable group management**, and **custom task development** for building modular, scalable Azure DevOps pipeline architectures. Learn to eliminate redundancy, centralize configuration, and standardize deployment workflows across your organization.

**Duration**: ~30 minutes  
**Level**: Intermediate

## Learning Objectives

By the end of this module, you'll be able to:

1. ✅ **Create and manage task groups** for reusable pipeline components
2. ✅ **Implement variable groups** for centralized configuration management
3. ✅ **Configure release and stage variables** for environment-specific deployments
4. ✅ **Develop custom build and release tasks** for specialized requirements

## Prerequisites

| Requirement | Description |
|------------|-------------|
| **DevOps Understanding** | Comprehensive understanding of DevOps principles and practices |
| **Version Control** | Experience with version control and collaborative workflows |
| **Azure Pipelines** | Practical knowledge of Azure Pipelines (build/release) |
| **Azure DevOps Access** | Active Azure DevOps organization with team project access |

**Setup Azure DevOps**: [Create an organization](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization)

## Module Scope

This module covers **6 units** focused on pipeline modularization and reusability:

### 📚 What You'll Learn

1. **Introduction** (Unit 1)
   - Module objectives and prerequisites

2. **Task Groups** (Unit 2)
   - Encapsulate task sequences
   - Create reusable components
   - Parameter extraction

3. **Variables in Release Pipelines** (Unit 3)
   - Predefined variables
   - Release pipeline variables
   - Stage variables
   - Variable groups
   - Secret variables

4. **Exercise: Variable Groups** (Unit 4)
   - Hands-on variable group creation
   - Link variable groups to pipelines
   - Scope variables to stages

5. **Knowledge Check** (Unit 5)
   - Assessment questions

6. **Summary** (Unit 6)
   - Key takeaways and next steps

## Why Modularization Matters

### 🎯 Key Benefits

| Benefit | Description |
|---------|-------------|
| **DRY Principle** | Don't Repeat Yourself - Define once, use everywhere |
| **Consistency** | Standardized workflows across all pipelines |
| **Maintainability** | Update once, propagate changes automatically |
| **Scalability** | Manage hundreds of pipelines efficiently |
| **Security** | Centralized secret management |
| **Velocity** | Faster pipeline creation with reusable components |

### 🏢 Real-World Scenarios

**Scenario 1: Multi-Application Organization**
- 50+ applications with similar deployment patterns
- Task group for Azure App Service deployment
- Variable group for shared configuration
- Result: 90% reduction in pipeline maintenance time

**Scenario 2: Compliance Requirements**
- Security scans required before every production deployment
- Task group encapsulates: OWASP ZAP + SonarQube + dependency scan
- Applied to all pipelines via inheritance
- Result: 100% compliance coverage, zero manual configuration

**Scenario 3: Multi-Environment Deployment**
- Dev, QA, Staging, Production environments
- Variable groups per environment (connection strings, API keys)
- Scope variables to appropriate stages
- Result: Zero hardcoded secrets, environment parity guaranteed

## Problem: Pipeline Proliferation Without Modularization

### ❌ Before Modularization

**Pipeline 1** (App A - Production):
```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Production-Subscription'
    appName: 'app-a-prod'
    package: '$(System.DefaultWorkingDirectory)/**/*.zip'
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Production-Subscription'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: 'az webapp restart --name app-a-prod --resource-group rg-prod'
```

**Pipeline 2** (App B - Production):
```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Production-Subscription'  # Duplicated
    appName: 'app-b-prod'
    package: '$(System.DefaultWorkingDirectory)/**/*.zip'
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Production-Subscription'  # Duplicated
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: 'az webapp restart --name app-b-prod --resource-group rg-prod'
```

**Problems**:
- 🔴 **Duplication**: Same tasks copied across 50+ pipelines
- 🔴 **Maintenance Nightmare**: Update subscription? Modify 50+ pipelines
- 🔴 **Inconsistency**: Some pipelines have restart step, others don't
- 🔴 **Error-Prone**: Copy-paste mistakes lead to production issues

### ✅ After Modularization

**Task Group** (Azure Web App Deployment):
```
Task Group: "Deploy to Azure Web App"
Parameters:
  - subscriptionParameter: $(azureSubscription)
  - appNameParameter: $(appName)
  
Tasks:
  1. Azure Web App Deploy
  2. Azure CLI (restart)
```

**Variable Group** (Production Environment):
```
Variable Group: "Production-Config"
Variables:
  - azureSubscription: "Production-Subscription"
  - resourceGroup: "rg-prod"
  - environment: "production"
```

**Simplified Pipeline 1** (App A):
```yaml
variables:
- group: Production-Config
- name: appName
  value: 'app-a-prod'

stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - task: DeployToAzureWebApp@1  # Task group reference
      inputs:
        subscriptionParameter: $(azureSubscription)
        appNameParameter: $(appName)
```

**Simplified Pipeline 2** (App B):
```yaml
variables:
- group: Production-Config  # Same variable group
- name: appName
  value: 'app-b-prod'

stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - task: DeployToAzureWebApp@1  # Same task group
      inputs:
        subscriptionParameter: $(azureSubscription)
        appNameParameter: $(appName)
```

**Benefits**:
- ✅ **Single Source of Truth**: Update subscription once in variable group
- ✅ **Consistency**: All pipelines use identical deployment logic
- ✅ **Easy Updates**: Modify task group → automatically updates all pipelines
- ✅ **Fewer Errors**: Less copy-paste, more reuse

## Module Learning Path

```
Introduction
    ↓
Examine Task Groups
    ↓
Explore Variables in Release Pipelines
    ↓
Exercise: Create and Manage Variable Groups
    ↓
Knowledge Check
    ↓
Summary
```

## Key Concepts

### Task Groups

**Definition**: Encapsulated sequence of tasks that can be reused across multiple pipelines

**Use Cases**:
- Deployment patterns (Azure App Service, AKS, VMs)
- Security scanning workflows
- Database migration steps
- Infrastructure provisioning sequences

**Important Note**: Task groups are for **Classic Pipelines** only. For YAML pipelines, use **templates** instead.

### Variable Groups

**Definition**: Named collection of variables that can be shared across multiple pipelines

**Types**:
- **Standard Variables**: Plain-text configuration values
- **Secret Variables**: Masked values for passwords, API keys, tokens
- **Azure Key Vault Integration**: Enterprise-grade secret management

**Scoping**:
- **Release Scope**: Available to all stages in a release
- **Stage Scope**: Limited to specific stages (Dev, QA, Prod)

### Variable Hierarchy

**Scope Precedence** (most specific wins):
```
1. Job-level variables (highest precedence)
2. Stage variables
3. Pipeline variables
4. Variable group variables
5. Predefined system variables (lowest precedence)
```

## Success Criteria

By module completion, you should be able to:

- [ ] Create task groups from existing pipeline tasks
- [ ] Extract parameters from task groups for reusability
- [ ] Create variable groups with standard and secret variables
- [ ] Link variable groups to release pipelines
- [ ] Scope variables to specific stages
- [ ] Integrate Azure Key Vault with variable groups
- [ ] Understand variable precedence and inheritance

## Critical Notes

⚠️ **Important Considerations**:

1. **Task Groups vs. Templates**: Task groups are for Classic Pipelines; use YAML templates for YAML pipelines
2. **Variable Overriding**: More specific scopes override less specific (job > stage > pipeline)
3. **Secret Variables**: Must be explicitly passed to tasks (not automatically inherited)
4. **Azure Key Vault**: Requires service connection and appropriate permissions
5. **Variable Groups Are Linked**: Changes propagate to all consuming pipelines
6. **Versioning**: Task groups support versioning for safe updates
7. **Testing**: Always test task group changes in non-production first

## Quick Reference

| Concept | Description | Example |
|---------|-------------|---------|
| **Task Group** | Reusable task sequence | Deploy to Azure Web App (deploy + restart) |
| **Variable Group** | Shared variable collection | Production-Config (subscription, RG, env) |
| **Release Variable** | Pipeline-level variable | `$(Build.BuildId)` |
| **Stage Variable** | Stage-specific variable | `targetEnvironment = 'staging'` |
| **Secret Variable** | Masked sensitive value | API key, password, connection string |
| **Predefined Variable** | System-generated | `$(Agent.BuildDirectory)` |

## What's Next?

**Unit 2**: Examine task groups - Learn how to create and manage reusable task sequences

---

## Additional Resources

- [Task groups for builds and releases](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/task-groups)
- [Variable groups for Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups)
- [Define variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables)
- [Use Azure Key Vault secrets](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-key-vault)

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/manage-modularize-tasks-templates/1-introduction)
