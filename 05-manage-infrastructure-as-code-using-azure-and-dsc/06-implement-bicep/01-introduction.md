# Introduction

This module explains what Bicep is and how it integrates with tools like Azure CLI, Visual Studio Code, Azure Pipelines, and GitHub workflows for infrastructure as code deployment.

## Module Overview

**Bicep** is a domain-specific language (DSL) for deploying Azure resources declaratively. It provides a simpler syntax than JSON templates, automatic dependency management, and type safety. Bicep transpiles to Azure Resource Manager (ARM) templates, providing all the benefits of infrastructure as code with an improved authoring experience.

This module covers:
- Bicep fundamentals and benefits
- Installation and configuration
- Template creation and syntax
- Deployment through various automation tools

## Why Bicep?

**Problem with ARM templates**: JSON-based ARM templates are verbose, difficult to read, and require manual dependency management. Complex deployments become hard to maintain.

**Bicep solution**: Clean syntax, automatic dependencies, type validation, VS Code integration, and module reusability—all while maintaining 100% ARM template compatibility.

## Learning Objectives

After completing this module, you'll be able to:

1. **Understand what Bicep is**: Learn about Bicep as a domain-specific language for Azure infrastructure as code
2. **Install Bicep**: Install Bicep CLI and configure Visual Studio Code with the Bicep extension for enhanced authoring
3. **Create Bicep templates**: Write Bicep files to define Azure resources with parameters, variables, and modules
4. **Understand Bicep syntax**: Learn file structure including parameters, variables, resources, outputs, and modules
5. **Deploy resources to Azure**: Use Azure CLI and Cloud Shell to deploy Bicep templates
6. **Deploy with Azure Pipelines**: Integrate Bicep deployments into Azure Pipelines for continuous deployment
7. **Deploy with GitHub workflows**: Use GitHub Actions to automate Bicep template deployments
8. **Understand ARM integration**: Learn how Bicep transpiles to Azure Resource Manager templates

## Prerequisites

Before starting this module, you should have:

- **Infrastructure as code concepts**: Understanding of IaC principles and benefits
- **Azure resources knowledge**: Basic familiarity with Azure resources and resource groups
- **Azure CLI or PowerShell**: Experience with Azure CLI or Azure PowerShell commands
- **CI/CD experience** (helpful): Azure Pipelines or GitHub workflows knowledge beneficial but not required

## Bicep vs ARM Templates Quick Comparison

| Aspect | ARM Templates (JSON) | Bicep |
|--------|---------------------|-------|
| **Syntax** | Verbose JSON | Concise, clean DSL |
| **Lines of code** | ~100 lines | ~30 lines (typical) |
| **Dependencies** | Manual (`dependsOn`) | Automatic inference |
| **Type safety** | Runtime errors | Design-time validation |
| **Tooling** | Basic | Excellent VS Code support |
| **Reusability** | Linked templates | Native modules |
| **Learning curve** | Steep | Gentle |

**Example**: Creating a storage account

**ARM Template (JSON) - 45 lines**:
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2"
    }
  ]
}
```

**Bicep - 12 lines**:
```bicep
@minLength(3)
@maxLength(24)
param storageAccountName string
param location string = resourceGroup().location

resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}
```

**Result**: Same infrastructure, 73% less code, much easier to read and maintain.

## Module Structure

This module consists of 10 units:

1. **Introduction** ← You are here
2. **What is Bicep?** - Benefits and capabilities
3. **Install Bicep** - CLI and VS Code setup
4. **Exercise: Create Bicep Templates** - Hands-on template creation
5. **Understand Bicep File Structure and Syntax** - Deep dive into syntax
6. **Exercise: Deploy Bicep from Azure Pipelines** - CI/CD integration
7. **Exercise: Deploy Bicep from GitHub Workflows** - GitHub Actions
8. **Deployments using Azure Bicep Templates** - Real-world labs
9. **Module Assessment** - Knowledge check
10. **Summary** - Recap and next steps

## Real-World Scenario

**Contoso Corp Challenge**: 
- Infrastructure team manages 500+ Azure resources across dev/staging/prod
- Current ARM templates: 15,000+ lines of JSON, difficult to maintain
- Frequent deployment errors due to manual dependency management
- Developers struggle with ARM template syntax

**Bicep Solution**:
- Reduced template code by 70% (4,500 lines of Bicep)
- Automatic dependency detection eliminated ordering errors
- VS Code IntelliSense reduced authoring time by 50%
- Modular design enabled code reuse across environments
- Deployment success rate improved from 82% to 98%

## What You'll Build

Throughout this module, you'll create:
- **Simple storage account** template
- **Web app with App Service Plan** using modules
- **Multi-resource environment** with networking, compute, and storage
- **CI/CD pipelines** for automated deployments
- **Reusable modules** for common patterns

## Key Terminology

- **Bicep**: Domain-specific language for Azure IaC
- **Transpile**: Convert Bicep to ARM template JSON
- **Resource**: Azure service (VM, storage, network, etc.)
- **Parameter**: Input value for template reusability
- **Variable**: Calculated value used in template
- **Module**: Reusable Bicep file
- **Output**: Value returned after deployment
- **Decorator**: Metadata for parameters (@minLength, @secure, etc.)

## Success Criteria

By the end of this module, you should be able to:
- ✅ Explain Bicep benefits over ARM templates
- ✅ Install and configure Bicep authoring tools
- ✅ Write Bicep templates with proper syntax
- ✅ Use parameters, variables, and modules effectively
- ✅ Deploy Bicep templates via CLI, Pipelines, and GitHub Actions
- ✅ Understand Bicep-to-ARM transpilation
- ✅ Create reusable, modular infrastructure code

---

**Module**: Implement Bicep  
**Unit**: 1 of 10  
**Duration**: 2 minutes  
**Next**: [Unit 2: What is Bicep?](#)  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/1-introduction
