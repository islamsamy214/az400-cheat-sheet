# Create and Manage Users

## User Types in Microsoft Entra ID

| User Type | Source | Description | Lifecycle |
|-----------|--------|-------------|-----------|
| **Cloud identities** | Microsoft Entra ID | Exist only in Entra ID (admin accounts, self-managed users) | Deleted when removed from directory |
| **Directory-synchronized** | Windows Server AD | Synced via Azure AD Connect from on-premises AD | Managed in source AD |
| **Guest users** | External/Invited | Accounts from other cloud providers, Microsoft accounts (Xbox, Outlook.com) | External lifecycle |

## Add Users - Methods

### Method 1: Sync On-Premises Active Directory
**Tool**: Azure AD Connect (separate service)

**Benefits**:
- Users can use SSO (single sign-on) for local and cloud resources
- Automatic sync of users, groups, passwords
- Most common for enterprise customers

**Requirements**:
- On-premises Windows Server Active Directory
- Azure AD Connect installed on domain-joined server
- Network connectivity to Azure

---

### Method 2: Microsoft Entra Admin Center (Manual)
**Best For**: Small set of users (1-20)

**Requirements**: User Administrator role

**Steps**:
1. Navigate to Microsoft Entra admin center
2. Select **Users** → **New user** → **Create new user**
3. Fill form:
   - **Name**: Display name (e.g., Alice Smith)
   - **User name**: UPN (e.g., alice@contoso.onmicrosoft.com)
   - **Properties** (optional): Job Title, Department, Manager
4. Click Create

**Result**: User created with default domain (e.g., `alice@contoso.onmicrosoft.com`)

**Alternative: Invite External User**
1. Select **New user** → **Invite external user**
2. Enter email address (e.g., bob@partner.com)
3. System sends invitation email
4. User accepts invitation → Guest account created
5. Guest user must create/link Microsoft account if email not associated

---

### Method 3: Command Line (PowerShell or Azure CLI)
**Best For**: Bulk operations, automation, 20+ users

#### PowerShell Example
```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Create password profile
$PasswordProfile = @{ 
    Password = "P@ssw0rd123!" 
}

# Create new user
New-MgUser -DisplayName "Abby Brown" `
           -PasswordProfile $PasswordProfile `
           -MailNickName "AbbyB" `
           -UserPrincipalName "AbbyB@contoso.com" `
           -AccountEnabled
```

**Output**:
```
DisplayName | Id                                    | UserPrincipalName
Abby Brown  | f36634c8-8a93-4909-9248-0845548bc515 | AbbyB@contoso.com
```

#### Azure CLI Example
```bash
az ad user create \
  --display-name "Abby Brown" \
  --password "P@ssw0rd123!" \
  --user-principal-name "AbbyB@contoso.com" \
  --force-change-password-next-login true \
  --mail-nickname "AbbyB"
```

---

### Method 4: Bulk Import (CSV File)
**Best For**: 50+ users, migration projects

#### CSV Format Example
```csv
DisplayName,UserPrincipalName,MailNickname,Department,JobTitle,Password
John Smith,john.smith@contoso.com,JohnS,Sales,Account Manager,R4ndomStr0ng!
Jane Doe,jane.doe@contoso.com,JaneD,Engineering,Developer,R4ndomStr0ng2!
```

#### PowerShell Bulk Import Script
```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Import CSV
$users = Import-Csv -Path "C:\users\newusers.csv"

# Loop through users
foreach ($user in $users) {
    $PasswordProfile = @{ 
        Password = $user.Password 
    }
    
    New-MgUser -DisplayName $user.DisplayName `
               -UserPrincipalName $user.UserPrincipalName `
               -MailNickName $user.MailNickname `
               -Department $user.Department `
               -JobTitle $user.JobTitle `
               -PasswordProfile $PasswordProfile `
               -AccountEnabled
    
    Write-Host "Created user: $($user.DisplayName)"
}
```

#### Best Practices for CSV Import
1. **Naming Conventions**: Establish standard format (e.g., `firstname.lastname@domain.com`)
2. **Password Policy**: Generate random passwords, conform to complexity rules
3. **Secure Delivery**: Email passwords to users or managers securely (encrypted email, separate channel)
4. **Validation**: Test import with 2-3 users before bulk upload
5. **Error Handling**: Log errors, retry failed imports

---

### Method 5: Other Options
- **Microsoft Graph API**: Programmatic user creation for custom apps
- **Microsoft 365 Admin Center**: If using M365 (shares same directory)
- **Microsoft Intune Admin Console**: For device-focused environments

## Quick Reference

### User Creation Methods Comparison
| Method | Use Case | Speed | Skill Level | Best For |
|--------|----------|-------|-------------|----------|
| **Azure AD Connect** | Hybrid identity | Auto | Advanced | Enterprise (1,000+ users) |
| **Admin Center** | Few users | Slow | Beginner | 1-20 users |
| **PowerShell** | Bulk operations | Fast | Intermediate | 20-500 users |
| **CSV Import** | Mass migration | Fast | Intermediate | 500+ users |
| **Graph API** | App integration | Fast | Advanced | Automated workflows |

### Common PowerShell Commands
```powershell
# List all users
Get-MgUser

# Filter users by department
Get-MgUser -Filter "department eq 'Sales'"

# Update user property
Update-MgUser -UserId "alice@contoso.com" -Department "Marketing"

# Delete user
Remove-MgUser -UserId "bob@contoso.com"

# Restore deleted user (within 30 days)
Restore-MgUser -DirectoryObjectId <user-id>
```

## Critical Notes
- ⚠️ **Password complexity**: Default rules require uppercase, lowercase, numbers, symbols (min 8 characters)
- 💡 **UPN format**: Must be `username@domain` (domain = `*.onmicrosoft.com` or custom verified domain)
- 🎯 **Guest user limits**: External users consume licenses at 1:5 ratio (5 guests = 1 license)
- 📊 **Bulk operations limit**: PowerShell recommended for 20+ users (Portal too slow)
- 🔄 **Sync latency**: Azure AD Connect syncs every 30 minutes (can trigger manual sync)
- ✨ **Deleted user retention**: 30-day soft delete (can restore within this period)

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-users-and-groups-in-aad/3-users)
