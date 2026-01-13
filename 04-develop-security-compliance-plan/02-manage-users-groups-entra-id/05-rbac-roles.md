# Use Roles to Control Resource Access

## Key Concepts
- **RBAC (Role-Based Access Control)**: Assign permissions based on roles, not individual users
- **Azure Roles**: Control access to Azure resources (VMs, storage, networks)
- **Microsoft Entra Roles**: Control access to Entra ID resources (users, groups, applications)
- **Principle of Least Privilege**: Grant minimum permissions needed for job function

## Microsoft Entra ID Roles

### Built-In Roles (Common)

| Role | Description | Permissions | Use Case |
|------|-------------|-------------|----------|
| **Global Administrator** | Full access to all Entra ID features | Everything | Emergency access only (2+ accounts) |
| **User Administrator** | Create/manage users and groups | User/group CRUD, reset passwords | HR, IT helpdesk |
| **Application Administrator** | Manage enterprise apps and registrations | App registration, SSO config | DevOps, app owners |
| **Security Administrator** | Read security info, manage security features | Conditional Access, ID Protection | Security team |
| **Billing Administrator** | Manage subscriptions, support tickets | Billing, purchasing | Finance team |
| **Helpdesk Administrator** | Reset passwords, limited user management | Password reset, user properties | IT support |
| **Groups Administrator** | Create/manage groups, group settings | Group CRUD, group policies | Team leads |

### Global Administrator (Most Powerful)
**Permissions**: 
- Manage all aspects of Entra ID and Microsoft 365 services
- Assign admin roles to other users
- Reset passwords for all users (including other Global Admins)
- Manage all subscriptions and resources

**Best Practices**:
- Create 2-3 emergency access accounts (break-glass)
- Use MFA on all Global Admin accounts
- Store credentials in secure location (vault)
- Don't use for daily tasks (use lower-privilege roles)
- Enable PIM (Privileged Identity Management) for just-in-time access

---

## Azure RBAC (Resource-Level)

### Built-In Azure Roles (Common)

| Role | Scope | Permissions | Use Case |
|------|-------|-------------|----------|
| **Owner** | Full control | Manage resources + assign roles | Subscription owner |
| **Contributor** | Manage resources | Create/modify/delete resources (no role assignment) | DevOps engineer |
| **Reader** | View only | Read resources, no changes | Auditors, reporting |
| **User Access Administrator** | Manage access | Assign Azure roles (no resource management) | IAM team |

### Scope Levels
```
Management Group (highest)
  ↓
Subscription
  ↓
Resource Group
  ↓
Resource (lowest)
```

**Inheritance**: Permissions assigned at higher scope inherit to lower scopes

**Example**:
```yaml
# User assigned "Contributor" at Subscription level
Result:
  - Can manage all resource groups in subscription
  - Can manage all resources in all resource groups
  - Cannot assign roles (requires Owner)
```

---

## Assign Roles

### Entra ID Roles (Portal)
**Steps**:
1. Microsoft Entra admin center → **Roles and administrators**
2. Select role (e.g., "User Administrator")
3. Click **Add assignments**
4. Select users/groups
5. Click **Add**

**PowerShell**:
```powershell
# Get role definition
$role = Get-MgDirectoryRole -Filter "DisplayName eq 'User Administrator'"

# Assign role to user
New-MgDirectoryRoleMemberByRef -DirectoryRoleId $role.Id -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/users/alice@contoso.com"
}
```

---

### Azure RBAC Roles (Portal)
**Steps**:
1. Navigate to resource (subscription, resource group, or individual resource)
2. Select **Access control (IAM)**
3. Click **Add role assignment**
4. Select role (e.g., "Contributor")
5. Select members (users/groups/service principals)
6. Click **Review + assign**

**PowerShell**:
```powershell
# Assign Contributor at resource group level
New-AzRoleAssignment -ObjectId <user-object-id> `
                     -RoleDefinitionName "Contributor" `
                     -ResourceGroupName "MyResourceGroup"

# Assign Reader at subscription level
New-AzRoleAssignment -SignInName "alice@contoso.com" `
                     -RoleDefinitionName "Reader" `
                     -Scope "/subscriptions/<subscription-id>"
```

**Azure CLI**:
```bash
# Assign Contributor
az role assignment create \
  --assignee alice@contoso.com \
  --role "Contributor" \
  --resource-group MyResourceGroup
```

---

## Custom Roles

### When to Use
- Built-in roles too broad (violate least privilege)
- Need very specific permission set
- Compliance requirements (audit-specific actions)

### Create Custom Azure Role
```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can start, stop, and monitor VMs",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/<subscription-id>"
  ]
}
```

**Create via PowerShell**:
```powershell
New-AzRoleDefinition -InputFile "VMOperator.json"
```

---

## Quick Reference

### Role Selection Flowchart
```
Need to manage Entra ID objects (users, groups, apps)?
  → Microsoft Entra ID Roles
  
Need to manage Azure resources (VMs, storage, networks)?
  → Azure RBAC Roles

Need full control?
  → Owner (Azure) or Global Administrator (Entra ID)

Need to create/modify but not assign roles?
  → Contributor (Azure) or User Administrator (Entra ID)

Need read-only access?
  → Reader (Azure) or Directory Readers (Entra ID)
```

### Common PowerShell Commands
```powershell
# List all Entra ID roles
Get-MgDirectoryRoleTemplate

# List Azure roles at subscription
Get-AzRoleDefinition

# List role assignments for user
Get-AzRoleAssignment -SignInName alice@contoso.com

# Remove role assignment
Remove-AzRoleAssignment -ObjectId <user-id> -RoleDefinitionName "Contributor"
```

### Entra ID vs Azure RBAC
| Aspect | Microsoft Entra ID Roles | Azure RBAC |
|--------|-------------------------|------------|
| **Scope** | Entra ID resources | Azure resources |
| **Examples** | Users, groups, apps, licenses | VMs, storage, networks |
| **Management** | Entra admin center | Azure Portal (IAM) |
| **Granularity** | Directory-wide or admin unit | Subscription/RG/Resource |
| **Custom Roles** | Limited | Fully supported |

## Critical Notes
- ⚠️ **Global Admin = dangerous**: Only 2-3 emergency accounts, never daily use
- 💡 **Separation of concerns**: Entra ID roles ≠ Azure roles (assign both if needed)
- 🎯 **Least privilege**: Start with Reader, add permissions as needed
- 📊 **PIM recommended**: Just-in-time admin access reduces attack surface (requires P2)
- 🔄 **Role inheritance**: Azure RBAC inherits from higher scopes (subscription → RG → resource)
- ✨ **Custom roles**: Use sparingly, prefer built-in roles (easier to maintain)

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-users-and-groups-in-aad/5-manage-aad-roles)
