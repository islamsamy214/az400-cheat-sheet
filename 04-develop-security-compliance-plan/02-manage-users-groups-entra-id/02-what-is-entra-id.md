# What is Microsoft Entra ID?

## Key Concepts
- **Not Cloud AD**: Microsoft Entra ID ≠ cloud version of Windows Server AD (complement, not replacement)
- **Hybrid Identity**: Extend on-premises AD to Azure, use same credentials locally and in cloud
- **Independent Usage**: Can use Entra ID without Windows AD (smaller companies)
- **Tenant**: Dedicated Entra ID instance representing organization
- **Subscription**: Billing entity + security boundary associated with tenant

## Microsoft Cloud Services Using Entra ID
- Microsoft Azure
- Microsoft 365
- Microsoft Intune
- Microsoft Dynamics 365

## Tenants, Subscriptions, and Users

### Relationship Model
```
Tenant (Organization) → Default Directory (Entra ID instance)
  ↓
Multiple Subscriptions (billing containers)
  ↓
Resources (VMs, websites, databases)
  ↓
Users & Groups (access control)
```

### Key Rules
- **Tenant**: Created when signing up for Azure/M365/Dynamics 365
- **Directory**: Each tenant has one directory holding users, groups, apps
- **Subscription**: 
  - Associated with single Entra ID directory
  - Multiple subscriptions can trust same directory
  - Subscription can only trust one directory
- **Users**: Can belong to multiple directories (switch via Directory + Subscription button)

### Visual Architecture
```
┌─────────────────────────────────────────────┐
│ Tenant (contoso.onmicrosoft.com)           │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │ Directory (Users, Groups, Apps)      │  │
│  │  - alice@contoso.com                 │  │
│  │  - bob@contoso.com                   │  │
│  │  - Sales Group                       │  │
│  └──────────────────────────────────────┘  │
│                 ↓ trusts                    │
│  ┌──────────────────────────────────────┐  │
│  │ Subscription 1 (Development)         │  │
│  │  - VMs, Websites, Databases          │  │
│  └──────────────────────────────────────┘  │
│  ┌──────────────────────────────────────┐  │
│  │ Subscription 2 (Production)          │  │
│  │  - VMs, Websites, Databases          │  │
│  └──────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

## Create a New Directory

### Steps
1. Sign in to Azure Portal
2. Azure home page → Create a resource
3. Left menu → Identity → Search "Microsoft Entra ID"
4. Click Create
5. Select tenant type: **Microsoft Entra ID**
6. Click Next: Configuration

### Configuration Form
| Field | Description | Example |
|-------|-------------|---------|
| **Organization name** | Display name for directory (changeable) | Contoso Corporation |
| **Initial domain name** | Default domain (permanent) | contoso.onmicrosoft.com |
| **Country/Region** | Datacenter location (permanent, not changeable) | United States |

### Result
- Free tier directory created
- Can add users, create roles, register apps, control licenses
- Access via: "Click here to manage your new tenant"

## Quick Reference

### Tenant vs Directory vs Subscription
| Concept | Definition | Example |
|---------|------------|---------|
| **Tenant** | Organization representation | Contoso Corporation |
| **Directory** | Container for users/groups/apps | contoso.onmicrosoft.com directory |
| **Subscription** | Billing entity + resource container | Dev Subscription ($500/month) |

### Knowledge Check Answers
**Q1**: An Azure subscription is a _______________?  
✅ **Billing entity and security boundary**

**Q2**: Which best describes subscription-directory relationship?  
✅ **A directory can be associated with multiple subscriptions, but a subscription is tied to a single directory**

**Q3**: Can an organization have more than one Microsoft Entra directory?  
✅ **True** (for dev/test, multi-forest sync, separate environments)

## Critical Notes
- ⚠️ **Country/Region permanent**: Cannot change datacenter location after creation
- 💡 **Hybrid identity pattern**: Extend on-prem AD to cloud with Azure AD Connect
- 🎯 **One default domain**: Always ends with `.onmicrosoft.com` (add custom domains later)
- 📊 **Multi-directory support**: Organizations can have multiple tenants (dev/test/prod)
- 🔄 **Directory switching**: Users in multiple directories switch via Portal header
- ✨ **Free tier includes**: User/group management, SSO (10 apps), basic reports, self-service password reset

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-users-and-groups-in-aad/2-create-aad)
