# Explore Resource Locks

## Key Concepts
- **Resource locks**: Administrative restrictions preventing accidental deletion or modification
- **Override permissions**: Even Owners/Contributors cannot delete or modify locked resources
- **Lock types**: CanNotDelete (allow updates, block delete) vs ReadOnly (block all changes)
- **Inheritance**: Locks cascade from parent scopes to child resources
- **Final safeguard**: Protection against operational mistakes, unauthorized changes

## Lock Types

### CanNotDelete Lock
| Operation | Allowed? | Use Case |
|-----------|----------|----------|
| **Read** | ✅ Yes | View properties, monitor metrics, access data |
| **Update** | ✅ Yes | Modify configurations, change settings |
| **Delete** | ❌ No | Cannot delete resource |

**Use Cases**:
- 🗄️ **Production databases**: Prevent accidental deletion, allow schema updates
- 🖥️ **Critical VMs**: Protect infrastructure VMs (domain controllers, monitoring servers)
- 🌐 **Network resources**: Secure VNets, VPN gateways, ExpressRoute circuits
- 💾 **Storage accounts**: Protect backup storage, audit log storage

### ReadOnly Lock
| Operation | Allowed? | Use Case |
|-----------|----------|----------|
| **Read** | ✅ Yes | View properties, monitor metrics |
| **Update** | ❌ No | Cannot modify any settings |
| **Delete** | ❌ No | Cannot delete resource |

**Use Cases**:
- 📋 **Compliance-frozen resources**: Resources under audit or regulatory hold
- 📐 **Reference architectures**: Template resources for replication
- 🔒 **Change control periods**: Temporary restriction during change freezes
- 🛠️ **Shared services**: Core infrastructure requiring stability

## Lock Inheritance

### Inheritance Hierarchy
```
Subscription Lock (CanNotDelete)
├── ALL resource groups inherit CanNotDelete
│   ├── Resource Group A
│   │   ├── ALL resources inherit CanNotDelete
│   │   └── Resource Group Lock (ReadOnly) added
│   │       └── Resources have BOTH CanNotDelete AND ReadOnly
│   │           → Most restrictive wins: ReadOnly
│   └── Resource Group B
│       ├── ALL resources inherit CanNotDelete
│       └── Individual resource lock can add ReadOnly
```

### Lock Precedence
**Most restrictive lock wins**:
- Resource with CanNotDelete + ReadOnly = **ReadOnly** (more restrictive)
- Subscription CanNotDelete + Resource Group ReadOnly = **ReadOnly**
- Cannot remove inherited locks at child level (must remove at parent)

## Implementing Resource Locks

### Apply Lock via Azure Portal
1. Navigate to resource (subscription/resource group/individual resource)
2. Click **Locks** in Settings section
3. Click **Add** to create lock
4. Configure:
   - **Lock name**: Descriptive name (e.g., "ProductionRGLock")
   - **Lock type**: CanNotDelete or ReadOnly
   - **Notes**: Explanation of lock purpose (recommended)
5. Save to apply lock

### Apply Lock via Azure CLI
```bash
# Apply CanNotDelete lock to resource group
az lock create \
  --name ProductionRGLock \
  --resource-group production-rg \
  --lock-type CanNotDelete \
  --notes "Prevents accidental deletion of production resources"

# Apply ReadOnly lock to storage account
az lock create \
  --name AuditStorageLock \
  --resource-type Microsoft.Storage/storageAccounts \
  --resource-name auditlogstorage \
  --resource-group compliance-rg \
  --lock-type ReadOnly \
  --notes "Storage account under regulatory hold - no modifications allowed"

# Apply lock to subscription
az lock create \
  --name SubscriptionProtection \
  --lock-type CanNotDelete \
  --notes "Prevents deletion of critical subscription resources"
```

### Apply Lock via Azure PowerShell
```powershell
# Apply CanNotDelete lock to VM
New-AzResourceLock `
  -LockName "DomainControllerLock" `
  -LockLevel CanNotDelete `
  -ResourceName "dc01" `
  -ResourceType "Microsoft.Compute/virtualMachines" `
  -ResourceGroupName "infrastructure-rg" `
  -LockNotes "Domain controller - critical infrastructure"

# Apply ReadOnly lock to SQL Server
New-AzResourceLock `
  -LockName "ComplianceDBLock" `
  -LockLevel ReadOnly `
  -ResourceName "compliance-sql" `
  -ResourceType "Microsoft.Sql/servers" `
  -ResourceGroupName "compliance-rg" `
  -LockNotes "Compliance database - read-only during audit period"
```

## Lock Operations

### View Existing Locks
```bash
# List locks on resource group
az lock list --resource-group production-rg --output table

# List all locks in subscription
az lock list --subscription {subscription-id} --output table

# Get lock details
az lock show \
  --name ProductionRGLock \
  --resource-group production-rg
```

### Remove Locks
```bash
# Delete lock by name
az lock delete \
  --name ProductionRGLock \
  --resource-group production-rg

# Delete lock by ID
az lock delete \
  --ids /subscriptions/{id}/resourceGroups/production-rg/providers/Microsoft.Authorization/locks/ProductionRGLock
```

### Lock Removal Requirements
**Required permissions**:
- ✅ `Microsoft.Authorization/locks/delete` permission
- ✅ **Owner** role (has lock delete permission)
- ✅ **User Access Administrator** role (has lock delete permission)
- ❌ **Contributor** role (does NOT have lock delete permission)

**Security benefit**: Separate permission from resource deletion provides additional protection layer

## Lock Management Best Practices

### Lock During Deployment (IaC)
```bicep
// Bicep template - Deploy resource with lock
resource sqlServer 'Microsoft.Sql/servers@2021-11-01' = {
  name: 'contoso-prod-sql'
  location: location
  properties: {
    administratorLogin: adminUsername
    administratorLoginPassword: adminPassword
  }
}

// Apply CanNotDelete lock to SQL Server
resource sqlServerLock 'Microsoft.Authorization/locks@2020-05-01' = {
  name: 'sql-server-lock'
  scope: sqlServer
  properties: {
    level: 'CanNotDelete'
    notes: 'Production SQL Server - prevents accidental deletion'
  }
}
```

```terraform
# Terraform - Deploy resource with lock
resource "azurerm_resource_group" "production" {
  name     = "production-rg"
  location = "East US"
}

resource "azurerm_management_lock" "resource-group-level" {
  name       = "production-rg-lock"
  scope      = azurerm_resource_group.production.id
  lock_level = "CanNotDelete"
  notes      = "Prevents deletion of production resource group"
}
```

### Document Lock Purposes
**Good lock notes examples**:
- ✅ "Production database - requires VP approval for deletion"
- ✅ "Under regulatory audit until 2024-12-31 - read-only"
- ✅ "Core networking infrastructure - protected by change control process"
- ❌ "Do not delete" (too vague)
- ❌ No notes (missing context)

### Regular Lock Reviews
**Review checklist**:
1. ✅ Are locks still necessary?
2. ✅ Do lock notes accurately reflect current purpose?
3. ✅ Are decommissioned resources still locked?
4. ✅ Do new critical resources need locks?
5. ✅ Are lock types appropriate (CanNotDelete vs ReadOnly)?

### Balance Protection with Flexibility
| Environment | Lock Type | Rationale |
|-------------|-----------|-----------|
| **Production** | CanNotDelete | Allow necessary updates, prevent deletion |
| **Change freeze** | ReadOnly (temporary) | Block all changes during high-risk periods |
| **Under audit** | ReadOnly | Ensure immutability for compliance |
| **Development** | None | Maximum flexibility for development |

### Emergency Procedures
**Document emergency lock removal**:
1. ✅ Who can authorize emergency removal?
2. ✅ What justification is required?
3. ✅ How is removal logged and audited?
4. ✅ When should lock be reapplied?

**Example process**:
```bash
# Emergency lock removal (requires approval)
# 1. Document incident ticket: INC-12345
# 2. Get VP approval
# 3. Remove lock with audit trail
az lock delete \
  --name ProductionSQLLock \
  --resource-group production-rg \
  --verbose

# 4. Perform emergency change
az sql server update ...

# 5. Reapply lock immediately after change
az lock create \
  --name ProductionSQLLock \
  --resource-group production-rg \
  --lock-type CanNotDelete \
  --notes "Reapplied after emergency change INC-12345 on $(date)"
```

## Combine with Azure Policy

### Auto-Apply Locks with Policy
```json
{
  "displayName": "Auto-apply locks to production SQL Servers",
  "policyRule": {
    "if": {
      "allOf": [
        {"field": "type", "equals": "Microsoft.Sql/servers"},
        {"field": "tags['Environment']", "equals": "Production"}
      ]
    },
    "then": {
      "effect": "deployIfNotExists",
      "details": {
        "type": "Microsoft.Authorization/locks",
        "existenceCondition": {
          "field": "Microsoft.Authorization/locks/level",
          "equals": "CanNotDelete"
        },
        "deployment": {
          "properties": {
            "mode": "incremental",
            "template": {
              "resources": [
                {
                  "type": "Microsoft.Authorization/locks",
                  "apiVersion": "2020-05-01",
                  "name": "AutoAppliedProductionLock",
                  "properties": {
                    "level": "CanNotDelete",
                    "notes": "Automatically applied to production SQL Servers via Azure Policy"
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

### Policy Benefits
- 🚀 **Automatic lock application**: New production resources automatically locked
- ✅ **Consistent protection**: No manual steps required
- 📊 **Compliance tracking**: Policy dashboard shows lock coverage

## Lock Scenarios

### Scenario 1: Multi-Tier Application Protection
```bash
# Lock entire production resource group
az lock create \
  --name ProductionAppLock \
  --resource-group production-app-rg \
  --lock-type CanNotDelete

# Additional ReadOnly lock on production database
az lock create \
  --name ProductionDBReadOnly \
  --resource-type Microsoft.Sql/servers \
  --resource-name prod-sql \
  --resource-group production-app-rg \
  --lock-type ReadOnly
```
**Result**: 
- All resources protected from deletion (CanNotDelete from RG)
- Database additionally protected from modification (ReadOnly on resource)

### Scenario 2: Change Freeze Period
```bash
# Apply temporary ReadOnly lock during change freeze
az lock create \
  --name ChangeFreezeLock \
  --resource-group production-rg \
  --lock-type ReadOnly \
  --notes "Change freeze: 2024-12-15 to 2024-12-31 for holiday season"

# Remove after freeze period
az lock delete \
  --name ChangeFreezeLock \
  --resource-group production-rg
```

### Scenario 3: Compliance Audit
```bash
# Lock resources under audit
az lock create \
  --name ComplianceAuditLock \
  --resource-group audit-scope-rg \
  --lock-type ReadOnly \
  --notes "Under PCI DSS audit - no changes allowed until 2024-06-30"
```

## Critical Notes
- 🔒 **Locks override all permissions**: Even Owners cannot delete locked resources
- 🎯 **CanNotDelete for production**: Allow updates, prevent deletion
- 📋 **ReadOnly for audits**: Complete immutability during compliance reviews
- 🔄 **Lock critical resources immediately**: Apply during initial deployment
- 📝 **Document lock purposes**: Clear notes explaining why lock exists
- ⚠️ **Inheritance caveat**: Cannot remove inherited locks at child level
- 💡 **Combine with Azure Policy**: Auto-apply locks to new resources
- 🔧 **Emergency procedures**: Document approval process for lock removal
- ✅ **Regular reviews**: Ensure locks remain appropriate and necessary

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/8-explore-resource-locks)
