# Understand Policies

## Key Concepts
- **Custom policies**: Organization-specific governance beyond built-in policies
- **Authoring workflow**: Identify → Design → Author → Test → Deploy → Monitor → Iterate
- **Testing approach**: Isolated environment + Audit mode + validation before production
- **Parameterization**: Reusable policies with configurable values
- **Remediation tasks**: Bring existing non-compliant resources into compliance
- **Version control**: Store policies in Git for CI/CD deployment

## Policy Authoring Workflow

```
1. Identify Requirement
   ↓
2. Design Policy Logic (conditions + effect)
   ↓
3. Author JSON Definition
   ↓
4. Test in Non-Production (Audit mode)
   ↓
5. Deploy to Production
   ↓
6. Monitor Compliance
   ↓
7. Iterate and Improve
```

## Custom Policy Structure

### Complete Policy Example
```json
{
  "properties": {
    "mode": "all",
    "parameters": {
      "allowedLocations": {
        "type": "array",
        "metadata": {
          "description": "The list of locations that can be specified when deploying resources",
          "strongType": "location",
          "displayName": "Allowed locations"
        }
      }
    },
    "displayName": "Allowed locations",
    "description": "Restricts locations your organization can specify when deploying resources",
    "policyRule": {
      "if": {
        "not": {
          "field": "location",
          "in": "[parameters('allowedLocations')]"
        }
      },
      "then": {"effect": "deny"}
    }
  }
}
```

### Common Custom Policy Scenarios
| Scenario | Effect | Use Case |
|----------|--------|----------|
| **Allowed Storage SKUs** | Deny | Prevent expensive storage tiers (Premium, Ultra) |
| **Allowed Resource Types** | Deny | Block HDInsight, ExpressRoute (cost control) |
| **Allowed VM SKUs** | Deny | Restrict to Standard_D2s_v3, Standard_D4s_v3 |
| **Add Tags to Resources** | Modify | Auto-apply Environment, CostCenter tags |
| **Required Diagnostic Settings** | DeployIfNotExists | Ensure all resources send logs to Log Analytics |

## Testing Custom Policies

### Test Workflow
```bash
# 1. Create test resource group
az group create --name policy-test-rg --location eastus

# 2. Assign policy in AUDIT mode (non-blocking)
az policy assignment create \
  --name test-vm-size-policy \
  --display-name 'Test: Allowed VM Sizes (Audit Mode)' \
  --scope /subscriptions/{id}/resourceGroups/policy-test-rg \
  --policy custom-vm-size-policy \
  --params '{"effect": {"value": "Audit"}, "allowedSKUs": {"value": ["Standard_D2s_v3"]}}'

# 3. Deploy COMPLIANT resource
az vm create \
  --resource-group policy-test-rg \
  --name compliant-vm \
  --image UbuntuLTS \
  --size Standard_D2s_v3  # Should pass

# 4. Deploy NON-COMPLIANT resource
az vm create \
  --resource-group policy-test-rg \
  --name non-compliant-vm \
  --image UbuntuLTS \
  --size Standard_B2ms  # Should trigger violation

# 5. Check compliance
az policy state list \
  --resource-group policy-test-rg \
  --query "[?complianceState=='NonCompliant'].{Resource:resourceId, Policy:policyDefinitionName}"

# 6. If tests pass, promote to production with DENY effect
az policy assignment create \
  --name production-vm-size-policy \
  --display-name 'Production: Allowed VM Sizes' \
  --scope /subscriptions/{id}/resourceGroups/production-rg \
  --policy custom-vm-size-policy \
  --params '{"effect": {"value": "Deny"}, "allowedSKUs": {"value": ["Standard_D2s_v3", "Standard_D4s_v3"]}}'
```

### Test Environment Best Practices
- ✅ **Isolated test subscription/RG**: Avoid affecting production
- ✅ **Audit mode first**: Observe without blocking
- ✅ **Test positive and negative cases**: Compliant + non-compliant resources
- ✅ **Review compliance dashboard**: Verify correct identification
- ✅ **Adjust logic as needed**: Refine conditions based on results

## Policy Parameters

### Parameterized Policy Definition
```json
{
  "properties": {
    "displayName": "Require specific tags on resources",
    "parameters": {
      "tagName": {
        "type": "String",
        "metadata": {
          "displayName": "Tag Name",
          "description": "Name of the tag to require"
        }
      },
      "tagValue": {
        "type": "String",
        "metadata": {
          "displayName": "Tag Value",
          "description": "Required value for the tag"
        }
      },
      "effect": {
        "type": "String",
        "defaultValue": "Deny",
        "allowedValues": ["Audit", "Deny", "Disabled"],
        "metadata": {
          "displayName": "Effect",
          "description": "Policy enforcement effect"
        }
      }
    },
    "policyRule": {
      "if": {
        "not": {
          "field": "[concat('tags[', parameters('tagName'), ']')]",
          "equals": "[parameters('tagValue')]"
        }
      },
      "then": {"effect": "[parameters('effect')]"}
    }
  }
}
```

### Parameter Usage in Assignments
```bash
# Production environment - DENY effect
az policy assignment create \
  --name production-tagging \
  --scope /subscriptions/{id}/resourceGroups/production-rg \
  --policy custom-tag-requirement \
  --params '{
    "tagName": {"value": "Environment"},
    "tagValue": {"value": "Production"},
    "effect": {"value": "Deny"}
  }'

# Development environment - AUDIT effect (less strict)
az policy assignment create \
  --name development-tagging \
  --scope /subscriptions/{id}/resourceGroups/development-rg \
  --policy custom-tag-requirement \
  --params '{
    "tagName": {"value": "Environment"},
    "tagValue": {"value": "Development"},
    "effect": {"value": "Audit"}
  }'
```

**Benefits of Parameters**:
- 🔄 **Reusability**: One policy, multiple configurations
- 🎯 **Flexibility**: Adjust enforcement per environment
- 📝 **Maintainability**: Update assignments without changing policy definition

## Policy Remediation

### Remediation Scenarios
| Scenario | Resource State | Action |
|----------|----------------|--------|
| **Missing diagnostic settings** | Existing resources lack logging | Deploy diagnostic settings via DINE |
| **Missing tags** | Resources deployed before tagging policy | Add tags via Modify effect |
| **Missing security agents** | VMs without monitoring agents | Deploy agents via DINE |
| **Unencrypted storage** | Existing storage without encryption | Enable encryption via remediation task |

### Create Remediation Task (Portal)
1. Navigate to **Azure Policy** → **Compliance**
2. Select non-compliant policy (must have DINE or Modify effect)
3. Click **Create remediation task**
4. Configure:
   - Policy assignment
   - Resources to remediate (all or specific)
   - Re-evaluate compliance before remediation (optional)
5. Create task

### Create Remediation Task (Azure CLI)
```bash
# Create remediation for non-compliant resources
az policy remediation create \
  --name remediate-diagnostic-settings \
  --policy-assignment /subscriptions/{id}/providers/Microsoft.Authorization/policyAssignments/require-diagnostic-settings \
  --resource-group production-rg

# Monitor remediation progress
az policy remediation show \
  --name remediate-diagnostic-settings \
  --resource-group production-rg

# List remediation deployments
az policy remediation deployment list \
  --name remediate-diagnostic-settings \
  --resource-group production-rg
```

### Bulk Remediation (PowerShell)
```powershell
# Remediate all non-compliant VMs (enable backup)
$policyAssignment = Get-AzPolicyAssignment -Name "require-backup-for-vms"
$nonCompliantResources = Get-AzPolicyState -PolicyAssignmentName $policyAssignment.Name | 
  Where-Object {$_.ComplianceState -eq "NonCompliant"}

foreach ($resource in $nonCompliantResources) {
    Write-Host "Remediating resource: $($resource.ResourceId)"
    
    Start-AzPolicyRemediation `
        -Name "remediation-$(Get-Date -Format 'yyyyMMddHHmmss')" `
        -PolicyAssignmentId $policyAssignment.Id `
        -ResourceId $resource.ResourceId
}

Write-Host "Remediation tasks created for $($nonCompliantResources.Count) resources"
```

## Policy Versioning and Lifecycle

### Repository Structure
```
policies/
├── custom/
│   ├── compute/
│   │   ├── allowed-vm-sizes.json
│   │   └── require-vm-backup.json
│   ├── storage/
│   │   ├── require-encryption.json
│   │   └── require-private-endpoints.json
│   └── networking/
│       ├── require-nsg.json
│       └── deny-public-ip.json
├── assignments/
│   ├── production.parameters.json
│   └── development.parameters.json
└── README.md
```

### Policy Deployment Pipeline (Azure DevOps)
```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - policies/**

pool:
  vmImage: "ubuntu-latest"

steps:
  # Validate JSON syntax
  - task: AzureCLI@2
    displayName: "Validate policy definitions"
    inputs:
      azureSubscription: "Policy-Management-Connection"
      scriptType: bash
      inlineScript: |
        for policy in policies/custom/**/*.json; do
          echo "Validating $policy..."
          # Basic JSON syntax check
          jq empty $policy || exit 1
        done

  # Deploy custom policies
  - task: AzureCLI@2
    displayName: "Deploy policy definitions"
    inputs:
      azureSubscription: "Policy-Management-Connection"
      scriptType: bash
      inlineScript: |
        for policy in policies/custom/**/*.json; do
          policy_name=$(basename $policy .json)
          echo "Deploying policy: $policy_name..."
          
          az policy definition create \
            --name $policy_name \
            --display-name "$(jq -r '.properties.displayName' $policy)" \
            --description "$(jq -r '.properties.description' $policy)" \
            --rules $policy \
            --mode $(jq -r '.properties.mode' $policy)
        done

  # Assign policies to subscriptions
  - task: AzureCLI@2
    displayName: "Assign policies"
    inputs:
      azureSubscription: "Policy-Management-Connection"
      scriptType: bash
      inlineScript: |
        az policy assignment create \
          --name production-governance \
          --scope /subscriptions/{prod-subscription-id} \
          --policy-set-definition production-governance-initiative \
          --params @policies/assignments/production.parameters.json
```

### Version Control Benefits
| Benefit | Description |
|---------|-------------|
| **Version tracking** | Git commit history of policy changes |
| **Code review** | Pull requests for policy changes before deployment |
| **Rollback capability** | Revert to previous policy versions if issues arise |
| **CI/CD deployment** | Automated policy deployment through pipelines |
| **Audit trail** | Complete history of who changed what and when |

## Advanced Policy Patterns

### Conditional Logic (Multiple Conditions)
```json
{
  "policyRule": {
    "if": {
      "allOf": [
        {"field": "type", "equals": "Microsoft.Storage/storageAccounts"},
        {"field": "location", "equals": "eastus"},
        {
          "anyOf": [
            {"field": "Microsoft.Storage/storageAccounts/sku.name", "equals": "Premium_LRS"},
            {"field": "Microsoft.Storage/storageAccounts/sku.name", "equals": "Premium_ZRS"}
          ]
        }
      ]
    },
    "then": {"effect": "deny"}
  }
}
```

### Field Functions
```json
{
  "if": {
    "field": "[concat('tags[', parameters('tagName'), ']')]",  // Dynamic field
    "notLike": "[parameters('tagPattern')]"  // Pattern matching
  }
}
```

### Count Operator (Array Validation)
```json
{
  "policyRule": {
    "if": {
      "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
          "allOf": [
            {"field": "Microsoft.Network/networkSecurityGroups/securityRules[*].access", "equals": "Allow"},
            {"field": "Microsoft.Network/networkSecurityGroups/securityRules[*].direction", "equals": "Inbound"},
            {"field": "Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRange", "equals": "*"}
          ]
        }
      },
      "greater": 0
    },
    "then": {"effect": "deny"}
  }
}
```

## Critical Notes
- ✅ **Start with Audit mode**: Understand impact before enforcement
- 🎯 **Parameterize for flexibility**: One policy definition, multiple configurations
- 🔄 **Test thoroughly**: Isolated environment + positive/negative cases
- 📝 **Document policies**: Clear descriptions in displayName and description fields
- 🚀 **Version control**: Store in Git for history, review, rollback
- 💡 **Bulk remediation**: Use remediation tasks for existing non-compliant resources
- ⚠️ **Monitor compliance**: Regular reviews to ensure policies remain effective
- 🔧 **CI/CD deployment**: Automate policy deployment for consistency

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/6-understand-policies)
