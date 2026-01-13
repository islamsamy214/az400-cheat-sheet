# Create and Manage Groups

## Group Types

| Type | Purpose | Members | Use Case | License |
|------|---------|---------|----------|---------|
| **Security Group** | Manage access to shared resources | Users, devices, other groups, service principals | Resource permissions, policy application | Free |
| **Microsoft 365 Group** | Collaboration (mailbox, calendar, files, SharePoint) | Users only | Team collaboration, project workspaces | Free |

## Membership Types

### 1. Assigned (Static)
**Definition**: Manually select specific users or groups

**Example**:
```
Group: "Sales Team"
Members: alice@contoso.com, bob@contoso.com, charlie@contoso.com
```

**Best For**: Small groups with stable membership (~5-50 members)

---

### 2. Dynamic User
**Definition**: Auto-assign users based on attributes (rules)

**Requirements**: Premium P1 or P2 license

**Example Rule**:
```yaml
Group: "Sales-US"
Rule: (user.department -eq "Sales") -and (user.country -eq "United States")
```

**Result**: Users automatically added when department = Sales AND country = US

**Common Attributes**:
- `user.department`
- `user.jobTitle`
- `user.country`
- `user.city`
- `user.manager`
- `user.companyName`
- `user.accountEnabled`

**Advanced Rule**:
```
(user.department -eq "Sales" -or user.department -eq "Marketing") 
-and user.country -eq "United States" 
-and user.accountEnabled -eq true
```

**Use Case**: Large organizations (100+ users), frequent department changes

---

### 3. Dynamic Device
**Definition**: Auto-assign devices based on device attributes

**Requirements**: Premium P1 or P2 license

**Example Rule**:
```yaml
Group: "Service-Department-Devices"
Rule: (device.department -eq "Service")
```

**Common Device Attributes**:
- `device.department`
- `device.deviceOSType` (Windows, iOS, Android)
- `device.deviceOSVersion`
- `device.deviceManagementAppId`
- `device.displayName`
- `device.deviceTrustType` (ServerAd, Workplace)

**Use Case**: Device compliance policies, Intune management

---

## Create Groups

### Method 1: Microsoft Entra Admin Center

**Steps**:
1. Navigate to Microsoft Entra admin center
2. Select **Groups** → **All groups**
3. Click **New group**
4. Fill form:

| Field | Options | Description |
|-------|---------|-------------|
| **Group type** | Security, Microsoft 365 | Choose based on use case |
| **Group name** | Unique name | E.g., "Sales-US" |
| **Group description** | Optional | Purpose/scope of group |
| **Membership type** | Assigned, Dynamic User, Dynamic Device | See membership types above |
| **Owners** | Select users/groups | Can manage group membership |
| **Members** | Select users/groups (if Assigned) | Initial members |

5. If Dynamic: Click **Add dynamic query** → Build rule
6. Click **Create**

---

### Method 2: PowerShell Script

#### Create Security Group (Assigned)
```powershell
New-MgGroup -DisplayName "Marketing" `
            -Description "Marketing Department" `
            -MailNickName "Marketing" `
            -SecurityEnabled `
            -MailEnabled:$False
```

#### Create Dynamic Group
```powershell
$rule = '(user.department -eq "Sales") -and (user.country -eq "United States")'

New-MgGroup -DisplayName "Sales-US" `
            -Description "US Sales Team" `
            -MailNickName "SalesUS" `
            -SecurityEnabled `
            -MailEnabled:$False `
            -GroupTypes "DynamicMembership" `
            -MembershipRule $rule `
            -MembershipRuleProcessingState "On"
```

#### Create Microsoft 365 Group
```powershell
New-MgGroup -DisplayName "Project Phoenix" `
            -Description "Project Phoenix Collaboration" `
            -MailNickName "ProjectPhoenix" `
            -SecurityEnabled:$False `
            -MailEnabled `
            -GroupTypes "Unified"
```

---

## Manage Group Membership

### View Group Members
```powershell
# PowerShell
Get-MgGroupMember -GroupId <group-id>

# Azure CLI
az ad group member list --group "Sales-US"
```

### Add Members (Assigned Groups)
**Portal**: Groups → [Select Group] → Members → Add members

**PowerShell**:
```powershell
# Add user to group
New-MgGroupMember -GroupId <group-id> -DirectoryObjectId <user-id>

# Add multiple users
$userIds = @("user-id-1", "user-id-2", "user-id-3")
foreach ($userId in $userIds) {
    New-MgGroupMember -GroupId <group-id> -DirectoryObjectId $userId
}
```

### Remove Members
```powershell
Remove-MgGroupMember -GroupId <group-id> -DirectoryObjectId <user-id>
```

### Manage Group Owners
```powershell
# Add owner
New-MgGroupOwner -GroupId <group-id> -DirectoryObjectId <user-id>

# Remove owner
Remove-MgGroupOwner -GroupId <group-id> -DirectoryObjectId <user-id>
```

---

## Quick Reference

### Group Type Selection Guide
```
Need collaboration tools (mailbox, calendar)? 
  → Microsoft 365 Group

Just access control? 
  → Security Group

Device-based policies?
  → Security Group (Dynamic Device)

Large org with frequent changes?
  → Security Group (Dynamic User)

Small stable team (<50 users)?
  → Security Group (Assigned)
```

### Dynamic Rule Examples
```powershell
# All Sales department
(user.department -eq "Sales")

# Sales OR Marketing
(user.department -eq "Sales") -or (user.department -eq "Marketing")

# Sales AND US
(user.department -eq "Sales") -and (user.country -eq "United States")

# Managers only
user.jobTitle -contains "Manager"

# Exclude contractors
(user.userType -eq "Member") -and (user.accountEnabled -eq true)

# Devices running Windows 10
(device.deviceOSType -eq "Windows") -and (device.deviceOSVersion -startsWith "10")
```

## Critical Notes
- ⚠️ **Dynamic groups require P1**: Free tier only supports Assigned membership
- 💡 **Nested groups**: Groups can contain other groups (inheritance)
- 🎯 **Owner permissions**: Owners can add/remove members, update group properties (not delete)
- 📊 **Dynamic rule testing**: Use "Validate Rules" in portal before creating (preview results)
- 🔄 **Dynamic processing time**: Can take 5-15 minutes for rule to apply after creation
- ✨ **M365 Groups auto-create**: SharePoint site, mailbox, Planner, OneNote, Teams (if enabled)

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-users-and-groups-in-aad/4-groups)
