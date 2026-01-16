# Explore Azure Policy

## Key Concepts
- **Policy-as-code**: JSON-based governance stored in version control, deployed via CI/CD
- **Effects**: Deny (block), Audit (log), DeployIfNotExists (auto-deploy), Modify (auto-fix), Append
- **Evaluation**: Deployment-time + periodic compliance scans (~24 hours) + on-demand
- **Scope hierarchy**: Management Group → Subscription → Resource Group → Resource
- **Inheritance**: Policies cascade from parent to child scopes

## Policy Definition Structure

```json
{
  "properties": {
    "displayName": "Require encryption for storage accounts",
    "description": "Enforces encryption in transit for all storage accounts",
    "mode": "All",  // "All" or "Indexed" (tags/location resources)
    "metadata": {
      "category": "Storage",
      "version": "1.0.0"
    },
    "parameters": {
      "effect": {
        "type": "String",
        "defaultValue": "Deny",
        "allowedValues": ["Audit", "Deny", "Disabled"]
      }
    },
    "policyRule": {
      "if": {
        "allOf": [
          {"field": "type", "equals": "Microsoft.Storage/storageAccounts"},
          {"field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly", "notEquals": "true"}
        ]
      },
      "then": {"effect": "[parameters('effect')]"}
    }
  }
}
```

### Policy Definition Components
| Component | Purpose | Example |
|-----------|---------|---------|
| **DisplayName** | Human-readable name | "Require encryption for storage accounts" |
| **Mode** | Evaluation scope | "All" (all resources), "Indexed" (tags/location only) |
| **Parameters** | Configurable values | Effect (Audit/Deny), allowed locations, SKU sizes |
| **PolicyRule** | Conditional logic | If storage account AND HTTPS=false, Then Deny |
| **Metadata** | Categorization | Category: Storage, Version: 1.0.0 |

## Policy Effects

### Enforcement Effects

#### Deny (Block Deployment)
```bash
# Blocks non-compliant resource creation
# Example: Prevent storage accounts without HTTPS
"then": {"effect": "deny"}
```
- ❌ Deployment fails with error message
- ✅ Prevents non-compliant resources from being created

#### DeployIfNotExists (Auto-Deploy)
```json
{
  "effect": "deployIfNotExists",
  "details": {
    "type": "Microsoft.Insights/diagnosticSettings",
    "deployment": {
      "properties": {
        "mode": "incremental",
        "template": {
          "resources": [
            {
              "type": "Microsoft.Insights/diagnosticSettings",
              "name": "audit-logging",
              "properties": {
                "workspaceId": "[parameters('workspaceId')]",
                "logs": [{"category": "Administrative", "enabled": true}]
              }
            }
          ]
        }
      }
    }
  }
}
```
- ✅ Automatically deploys missing resources (diagnostic settings, agents, encryption)
- 🔄 Applied during deployment + remediation tasks

#### Modify (Auto-Fix Properties)
```json
{
  "effect": "modify",
  "details": {
    "roleDefinitionIds": ["/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"],
    "operations": [
      {
        "operation": "addOrReplace",
        "field": "tags['Environment']",
        "value": "[parameters('environmentTag')]"
      }
    ]
  }
}
```
- 🔧 Modifies resource properties during deployment
- 🏷️ Common use: Enforce required tags

### Assessment Effects

#### Audit (Log Violations)
```bash
# Logs non-compliant resources without blocking
"then": {"effect": "audit"}
```
- 📝 Compliance dashboard shows violations
- ✅ Deployment proceeds (no blocking)

#### AuditIfNotExists (Detect Missing Resources)
```json
{
  "effect": "auditIfNotExists",
  "details": {
    "type": "Microsoft.Security/complianceResults",
    "existenceCondition": {
      "field": "Microsoft.Security/complianceResults/resourceStatus",
      "equals": "Compliant"
    }
  }
}
```
- 🔍 Detects missing configurations (backup policies, diagnostic settings)

### Control Effects

#### Disabled
```bash
# Temporarily disables policy without removing assignment
"then": {"effect": "disabled"}
```

#### Append
```json
{
  "effect": "append",
  "details": [
    {
      "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
      "value": [{"action": "Allow", "value": "10.0.0.0/24"}]
    }
  ]
}
```

## Policy Evaluation

### Evaluation Timeline
| Trigger | When | Use Case |
|---------|------|----------|
| **Deployment-time** | During ARM deployment | Prevent non-compliant deployments |
| **Compliance scan** | Every ~24 hours | Detect configuration drift |
| **On-demand** | Manual trigger | Immediate compliance check |

### Evaluation Flow
```
Resource Deployment Request
    ↓
Azure Resource Manager
    ↓
Policy Engine Evaluation
    ↓
┌───────────────────────────┐
│ Deny → Deployment FAILS   │
│ DINE → Additional Deploy  │
│ Modify → Properties Fixed │
│ Audit → Logged, Continues │
└───────────────────────────┘
    ↓
Compliance State Updated
```

## Policy Assignment

### Assignment Scopes
```
Management Group (Root)
├── Management Group (Production)
│   ├── Subscription (Prod-01)  ← Assign here: affects all RGs/resources
│   │   ├── Resource Group (Web-App-RG)  ← Assign here: affects resources only
│   │   └── Resource Group (Database-RG)
└── Management Group (Development)
```

### Assign Policy via Azure CLI
```bash
# Assign to subscription
az policy assignment create \
  --name 'RequireEncryptionForStorage' \
  --display-name 'Require encryption for all storage accounts' \
  --scope '/subscriptions/{subscription-id}' \
  --policy '/providers/Microsoft.Authorization/policyDefinitions/404c3081-a854-4457-ae30-26a93ef643f9' \
  --params '{"effect": {"value": "Deny"}}'

# Assign to resource group
az policy assignment create \
  --name 'EnforceVMSizes' \
  --scope '/subscriptions/{id}/resourceGroups/production-rg' \
  --policy 'custom-vm-size-policy' \
  --params '{"allowedSKUs": {"value": ["Standard_D2s_v3", "Standard_D4s_v3"]}}'
```

### Exemptions
```bash
# Create policy exemption
az policy exemption create \
  --name 'dev-environment-exemption' \
  --policy-assignment '/subscriptions/{id}/providers/Microsoft.Authorization/policyAssignments/require-backup' \
  --exemption-category 'Mitigated' \
  --expires-on '2024-12-31T23:59:59Z' \
  --description 'Dev environment excluded from backup requirement during migration'
```

**Exemption Reasons**:
- **Mitigation**: Temporary exemption while remediation is planned
- **Waiver**: Permanent exemption with business justification

## Built-in Policies

### Security Policies
```bash
# Require secure transfer (HTTPS) for storage accounts
/providers/Microsoft.Authorization/policyDefinitions/404c3081-a854-4457-ae30-26a93ef643f9

# Deploy Advanced Threat Protection on storage accounts
/providers/Microsoft.Authorization/policyDefinitions/361c2074-3595-4e5d-8cab-4f21dffc835c

# Require TDE for SQL databases
/providers/Microsoft.Authorization/policyDefinitions/17k78e20-9358-41c9-923c-fb736d382a12
```

### Cost Management Policies
```bash
# Allowed VM SKUs
/providers/Microsoft.Authorization/policyDefinitions/cccc23c7-8427-4f53-ad12-b6a63eb452b3

# Allowed locations for resources
/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c
```

### Browse Built-in Policies
```bash
# List all built-in policies
az policy definition list --query "[?policyType=='BuiltIn']" --output table

# Search by category
az policy definition list \
  --query "[?policyType=='BuiltIn' && metadata.category=='Security Center']" \
  --output table
```

## Policy Compliance

### Compliance Dashboard
```bash
# Get overall compliance
az policy state summarize \
  --subscription {subscription-id}

# List non-compliant resources
az policy state list \
  --filter "complianceState eq 'NonCompliant'" \
  --query "[].{Resource:resourceId, Policy:policyDefinitionName}"
```

### Compliance States
| State | Meaning | Action Required |
|-------|---------|-----------------|
| **Compliant** | Meets all requirements | None - maintain |
| **Non-compliant** | Violates policy | Remediate |
| **Conflict** | Conflicting policies (one allows, one denies) | Resolve conflict |
| **Not started** | Evaluation pending | Wait for scan |
| **Exempt** | Explicitly exempted | Review exemption validity |

## CI/CD Pipeline Integration

### Azure Pipelines - Pre-Deployment Validation
```yaml
trigger:
  - main

steps:
  # Test deployment (what-if)
  - task: AzureCLI@2
    displayName: "Validate ARM template against policies"
    inputs:
      azureSubscription: "Production-Service-Connection"
      scriptType: bash
      inlineScript: |
        az deployment group what-if \
          --resource-group production-rg \
          --template-file infrastructure/main.bicep

  # Check policy compliance
  - task: AzurePolicyCheckGate@0
    displayName: "Check Azure Policy compliance"
    inputs:
      azureSubscription: "Production-Service-Connection"
      ResourceGroupName: "production-rg"
      Resources: "StorageAccount,VirtualMachine,SqlDatabase"

  # Deploy only if checks pass
  - task: AzureResourceManagerTemplateDeployment@3
    condition: succeeded()
    displayName: "Deploy infrastructure"
    inputs:
      templateLocation: "Linked artifact"
      csmFile: "$(Build.SourcesDirectory)/infrastructure/main.bicep"
```

### GitHub Actions - Policy Validation
```yaml
name: Infrastructure Deployment with Policy Validation

on:
  push:
    branches: [main]

jobs:
  validate-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Validate against policies
        uses: azure/arm-deploy@v1
        with:
          scope: resourcegroup
          resourceGroupName: production-rg
          template: ./infrastructure/main.bicep
          deploymentMode: Validate  # Validation only

      - name: Check compliance
        run: |
          az policy state trigger-scan --resource-group production-rg
          non_compliant=$(az policy state list \
            --resource-group production-rg \
            --filter "complianceState eq 'NonCompliant'" \
            --query "length(@)")
          
          if [ $non_compliant -gt 0 ]; then
            echo "::error::Found $non_compliant non-compliant resources"
            exit 1
          fi

      - name: Deploy infrastructure
        if: success()
        uses: azure/arm-deploy@v1
        with:
          scope: resourcegroup
          resourceGroupName: production-rg
          template: ./infrastructure/main.bicep
          deploymentMode: Incremental
```

## Policy-Driven Governance Scenarios

### Scenario 1: Enforce Region Restrictions (Data Residency)
```json
{
  "displayName": "Allowed locations for resources",
  "description": "EU data residency compliance - restrict to EU regions",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "location",
          "notIn": ["northeurope", "westeurope", "francecentral", "germanywestcentral"]
        },
        {"field": "type", "notEquals": "Microsoft.Resources/resourceGroups"}
      ]
    },
    "then": {"effect": "deny"}
  }
}
```

### Scenario 2: Enforce Required Tagging
```json
{
  "displayName": "Require CostCenter and Owner tags",
  "policyRule": {
    "if": {
      "anyOf": [
        {"field": "tags['CostCenter']", "exists": "false"},
        {"field": "tags['Owner']", "exists": "false"}
      ]
    },
    "then": {"effect": "deny"}
  }
}
```

### Scenario 3: Auto-Enable Diagnostic Logging
```json
{
  "displayName": "Deploy diagnostic settings for storage accounts",
  "policyRule": {
    "if": {"field": "type", "equals": "Microsoft.Storage/storageAccounts"},
    "then": {
      "effect": "deployIfNotExists",
      "details": {
        "type": "Microsoft.Insights/diagnosticSettings",
        "deployment": {
          "properties": {
            "mode": "incremental",
            "template": {
              "resources": [
                {
                  "type": "Microsoft.Storage/storageAccounts/providers/diagnosticSettings",
                  "properties": {
                    "workspaceId": "[parameters('logAnalyticsWorkspaceId')]",
                    "logs": [
                      {"category": "StorageRead", "enabled": true},
                      {"category": "StorageWrite", "enabled": true}
                    ]
                  }
                }
              ]
            }
          }
        }
      }
    }
  }
}
```

## Critical Notes
- 🎯 **Policy-as-code**: Store definitions in Git for version control, review, rollback
- 🔒 **Deny for security**: Use Deny effect for critical security requirements
- 📝 **Audit for discovery**: Start with Audit mode to understand impact before enforcement
- 🔄 **DeployIfNotExists for automation**: Auto-configure diagnostic settings, agents, encryption
- ⚠️ **Test in non-production first**: Validate policy behavior before production assignment
- 💡 **Parameters for reusability**: Single policy, multiple configurations
- 🚀 **CI/CD integration**: Validate IaC templates before deployment (shift-left governance)
- 📊 **Monitor compliance daily**: Track compliance dashboard, remediate violations promptly

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/5-explore-azure-policy)
