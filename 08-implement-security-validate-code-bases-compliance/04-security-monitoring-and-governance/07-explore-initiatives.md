# Explore Initiatives

## Key Concepts
- **Initiatives** (policy sets): Bundle multiple related policies into logical groups
- **Simplified management**: Assign 100+ policies with single initiative assignment
- **Compliance frameworks**: Map to regulatory standards (PCI DSS, ISO 27001, HIPAA)
- **Centralized configuration**: Single parameter file for all policies in initiative
- **Unified tracking**: Aggregate compliance view instead of individual policy tracking

## Initiative Benefits

| Benefit | Description | Example |
|---------|-------------|---------|
| **Single assignment** | Assign entire framework at once | Azure Security Benchmark (100+ policies) with 1 assignment |
| **Centralized configuration** | One parameter file for all policies | Configure allowed locations once for all policies |
| **Unified compliance** | View framework compliance, not individual policies | PCI DSS compliance: 68% → 94% (aggregate score) |
| **Easier updates** | Add/remove policies without changing assignments | Add new security policy to existing initiative |
| **Framework representation** | Natural mapping to compliance standards | ISO 27001 controls → Azure Policy initiative |

## Built-in Initiatives

### Azure Security Benchmark Initiative
**Comprehensive security baseline** (100+ policies):

```bash
# Assign Azure Security Benchmark to subscription
az policy assignment create \
  --name azure-security-benchmark \
  --display-name 'Azure Security Benchmark' \
  --scope /subscriptions/{subscription-id} \
  --policy-set-definition '/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8'
```

**Coverage Areas**:
| Category | Policy Examples | Count |
|----------|-----------------|-------|
| **Identity & Access** | MFA, privileged accounts, RBAC | 15+ |
| **Network Security** | NSGs, private endpoints, DDoS, firewall | 20+ |
| **Data Protection** | Encryption at rest, encryption in transit, TLS 1.2 | 18+ |
| **Asset Management** | Resource inventory, approved types, locks | 10+ |
| **Logging & Monitoring** | Diagnostic settings, activity logs, security monitoring | 15+ |
| **Vulnerability Management** | Security assessments, patching, scanning | 12+ |
| **Endpoint Security** | Endpoint protection, malware detection | 8+ |
| **Backup & Recovery** | Backup policies, disaster recovery | 6+ |
| **Governance** | Policy compliance, security policies | 10+ |

### Common Compliance Initiatives
```bash
# PCI DSS 3.2.1 (Payment Card Industry)
az policy assignment create \
  --name pci-dss-compliance \
  --policy-set-definition '/providers/Microsoft.Authorization/policySetDefinitions/496eeda9-8f2f-4d5e-8dfd-204f0a92ed41' \
  --scope /subscriptions/{subscription-id}

# ISO 27001:2013 (Information Security)
az policy assignment create \
  --name iso-27001-compliance \
  --policy-set-definition '/providers/Microsoft.Authorization/policySetDefinitions/89c6cddc-1c73-4ac1-b19c-54d1a15a42f2' \
  --scope /subscriptions/{subscription-id}

# HIPAA/HITRUST (Healthcare)
az policy assignment create \
  --name hipaa-compliance \
  --policy-set-definition '/providers/Microsoft.Authorization/policySetDefinitions/a169a624-5599-4385-a696-c8d643089fab' \
  --scope /subscriptions/{subscription-id}

# NIST SP 800-53 Rev. 5 (Federal)
az policy assignment create \
  --name nist-compliance \
  --policy-set-definition '/providers/Microsoft.Authorization/policySetDefinitions/179d1daa-458f-4e47-8086-2a68d0d6c38f' \
  --scope /subscriptions/{subscription-id}
```

## Custom Initiative Definitions

### Custom Initiative Structure
```json
{
  "properties": {
    "displayName": "Contoso Security and Compliance Framework",
    "description": "Comprehensive security and compliance policies for Contoso production",
    "metadata": {
      "category": "Security",
      "version": "1.0.0"
    },
    "parameters": {
      "effect": {
        "type": "String",
        "defaultValue": "Audit",
        "allowedValues": ["Audit", "Deny", "Disabled"]
      },
      "allowedLocations": {
        "type": "Array",
        "defaultValue": ["eastus", "westus", "centralus"]
      }
    },
    "policyDefinitions": [
      {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0a914e76-4921-4c19-b460-a2d36003525a",
        "policyDefinitionReferenceId": "RequireMFAforPrivilegedAccounts",
        "parameters": {
          "effect": {"value": "[parameters('effect')]"}
        }
      },
      {
        "policyDefinitionId": "/subscriptions/{id}/providers/Microsoft.Authorization/policyDefinitions/custom-encryption-policy",
        "policyDefinitionReferenceId": "RequireStorageEncryption",
        "parameters": {
          "effect": {"value": "Deny"}
        }
      },
      {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
        "policyDefinitionReferenceId": "AllowedLocations",
        "parameters": {
          "listOfAllowedLocations": {"value": "[parameters('allowedLocations')]"}
        }
      }
    ]
  }
}
```

### Create Custom Initiative
```bash
# Create initiative definition
az policy set-definition create \
  --name contoso-security-framework \
  --display-name 'Contoso Security and Compliance Framework' \
  --description 'Custom security policies for Contoso production environments' \
  --definitions '[
    {
      "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0a914e76-4921-4c19-b460-a2d36003525a",
      "parameters": {"effect": {"value": "Audit"}}
    },
    {
      "policyDefinitionId": "/subscriptions/{id}/providers/Microsoft.Authorization/policyDefinitions/custom-encryption-policy",
      "parameters": {"effect": {"value": "Deny"}}
    },
    {
      "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
      "parameters": {"listOfAllowedLocations": {"value": ["eastus", "westus"]}}
    }
  ]'

# Assign initiative to subscription
az policy assignment create \
  --name contoso-security-assignment \
  --display-name 'Contoso Security Framework Assignment' \
  --scope /subscriptions/{subscription-id} \
  --policy-set-definition contoso-security-framework
```

### Initiative Design Patterns

#### Security-Focused Initiative
```json
{
  "displayName": "Zero Trust Security Framework",
  "policyDefinitions": [
    {"policyDefinitionReferenceId": "RequireMFA"},
    {"policyDefinitionReferenceId": "RequirePrivateEndpoints"},
    {"policyDefinitionReferenceId": "DenyPublicIPsOnVMs"},
    {"policyDefinitionReferenceId": "RequireNetworkSegmentation"},
    {"policyDefinitionReferenceId": "RequireEncryption"},
    {"policyDefinitionReferenceId": "RequireJITVMAccess"}
  ]
}
```

#### Cost-Control Initiative
```json
{
  "displayName": "Cost Optimization Framework",
  "policyDefinitions": [
    {"policyDefinitionReferenceId": "AllowedVMSKUs"},
    {"policyDefinitionReferenceId": "AllowedStorageSKUs"},
    {"policyDefinitionReferenceId": "AllowedLocations"},
    {"policyDefinitionReferenceId": "DenyExpensiveResourceTypes"},
    {"policyDefinitionReferenceId": "RequireCostCenterTag"}
  ]
}
```

#### Operational Excellence Initiative
```json
{
  "displayName": "Operational Excellence Framework",
  "policyDefinitions": [
    {"policyDefinitionReferenceId": "RequireDiagnosticSettings"},
    {"policyDefinitionReferenceId": "RequireBackupForVMs"},
    {"policyDefinitionReferenceId": "RequireTagging"},
    {"policyDefinitionReferenceId": "RequireResourceLocks"},
    {"policyDefinitionReferenceId": "RequireMonitoringAgents"}
  ]
}
```

## Initiative Compliance Tracking

### View Compliance in Portal
1. Navigate to **Azure Policy** → **Compliance**
2. Select initiative assignment (e.g., "Azure Security Benchmark")
3. View compliance metrics:
   - **Overall compliance percentage**: 83/100
   - **Non-compliant policies**: 17 out of 102 policies
   - **Non-compliant resources**: 45 resources violating policies
4. Drill down to policy details:
   - Click individual policy to see affected resources
   - Review remediation guidance

### Query Initiative Compliance (Azure CLI)
```bash
# Get initiative compliance state
az policy state list \
  --policy-set-definition contoso-security-framework \
  --query "[].{Resource:resourceId, Policy:policyDefinitionName, Compliance:complianceState}" \
  --output table

# Summary of non-compliant resources by policy
az policy state summarize \
  --policy-set-definition contoso-security-framework \
  --filter "complianceState eq 'NonCompliant'" \
  --query "policyAssignments[].{Policy:policyDefinitionName, NonCompliant:results.nonCompliantResources}"

# Get compliance percentage
az policy state summarize \
  --policy-set-definition azure-security-benchmark \
  --query "results.policyDetails[].{Policy:policyDefinitionName, Compliance:compliancePercentage}"
```

### Compliance Dashboard Example
| Policy | Compliant Resources | Non-Compliant | Compliance % |
|--------|---------------------|---------------|--------------|
| **MFA for admins** | 18/20 | 2 | 90% |
| **Storage encryption** | 85/90 | 5 | 94% |
| **VM patching** | 140/200 | 60 | 70% |
| **Diagnostic settings** | 450/500 | 50 | 90% |
| **Resource tagging** | 380/500 | 120 | 76% |
| **Overall Framework** | 1073/1310 | 237 | **82%** |

### Export Compliance Reports
```bash
# Generate compliance report (JSON)
az policy state list \
  --policy-set-definition contoso-security-framework \
  --output json > compliance-report-$(date +%Y%m%d).json

# Query for executive summary
az policy state summarize \
  --policy-set-definition azure-security-benchmark \
  --query "{
    TotalPolicies: length(policyAssignments),
    CompliantResources: results.resourceDetails[0].compliantResources,
    NonCompliantResources: results.resourceDetails[0].nonCompliantResources,
    CompliancePercentage: results.resourceDetails[0].compliancePercentage
  }"
```

## Initiative Assignment at Scale

### Management Group Hierarchy
```
Root Management Group
├── Production MG (Assign: Azure Security Benchmark + PCI DSS)
│   ├── Subscription: Prod-US-East
│   ├── Subscription: Prod-US-West
│   └── Subscription: Prod-Europe
└── Development MG (Assign: Azure Security Benchmark only)
    ├── Subscription: Dev-Shared
    └── Subscription: Dev-Team-A
```

### Assign to Management Group
```bash
# Assign initiative to management group (inherits to all subscriptions)
az policy assignment create \
  --name security-benchmark-production \
  --display-name 'Azure Security Benchmark - Production' \
  --scope /providers/Microsoft.Management/managementGroups/production-mg \
  --policy-set-definition azure-security-benchmark \
  --params '{
    "effect": {"value": "Deny"},
    "listOfAllowedLocations": {"value": ["eastus", "westus", "northeurope"]}
  }'
```

### Environment-Specific Configuration
```bash
# Production: Strict enforcement (Deny)
az policy assignment create \
  --name prod-governance \
  --scope /providers/Microsoft.Management/managementGroups/production-mg \
  --policy-set-definition contoso-security-framework \
  --params '{"effect": {"value": "Deny"}}'

# Development: Audit only (non-blocking)
az policy assignment create \
  --name dev-governance \
  --scope /providers/Microsoft.Management/managementGroups/development-mg \
  --policy-set-definition contoso-security-framework \
  --params '{"effect": {"value": "Audit"}}'
```

## Critical Notes
- 🎯 **Single assignment > individual policies**: Assign Azure Security Benchmark (100+ policies) once instead of 100 individual assignments
- 📊 **Aggregate compliance view**: Framework-level compliance (82%) more meaningful than individual policy tracking
- 🔄 **Built-in initiatives for standards**: PCI DSS, ISO 27001, HIPAA, NIST pre-configured
- 💡 **Custom initiatives for org requirements**: Bundle organization-specific policies (e.g., "Contoso Security Framework")
- 🚀 **Management group assignment**: Assign once at MG level, applies to all child subscriptions
- ⚠️ **Parameter inheritance**: Set parameters at initiative level, applies to all contained policies
- 📝 **Document policy reference IDs**: Use descriptive IDs (e.g., "RequireMFAforPrivilegedAccounts") for clarity
- ✅ **Start with built-in initiatives**: Customize existing frameworks rather than building from scratch

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/7-explore-initiatives)
