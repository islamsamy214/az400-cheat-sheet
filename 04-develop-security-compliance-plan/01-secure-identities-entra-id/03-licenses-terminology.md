# Understand Microsoft Entra ID Licenses and Terminology

## Licensing Tiers

| License | Monthly Cost | Key Features | Best For |
|---------|--------------|--------------|----------|
| **Free** | $0 | User/group management, SSO (10 apps), basic reports, self-service password reset (cloud users), Azure AD Connect sync | Small organizations, basic SSO |
| **Microsoft 365 Apps** | Included | All Free features + MFA, self-service password reset (on-prem writeback), group-based access management | Microsoft 365 subscribers |
| **Premium P1** | ~$6/user/month | All M365 features + dynamic groups, self-service group management, Microsoft Identity Manager (on-prem suite), password writeback, Conditional Access policies | Mid-size enterprises |
| **Premium P2** | ~$9/user/month | All P1 features + Microsoft Entra ID Protection (risk-based policies), Privileged Identity Management (PIM), access reviews | Large enterprises, compliance-heavy industries |

## Core Terminology

### 1. Identity
**Definition**: Object that can be authenticated (user, application, service)

**Types**:
- **User identity**: Person (employee, contractor, customer)
- **Application identity**: Service principal for apps
- **Managed identity**: Auto-managed Azure resource identity

### 2. Account
**Definition**: Identity with associated data (profile, permissions)

**Types**:
- **Work/School account**: Organizational Entra ID account
- **Personal account**: Microsoft account (Outlook.com, Xbox, etc.)
- **Guest account**: B2B external user

### 3. Tenant
**Definition**: Dedicated Entra ID instance
- Created when signing up for Azure, Microsoft 365, or Dynamics 365
- Represented by unique domain (e.g., `contoso.onmicrosoft.com`)
- Can add custom domains (e.g., `contoso.com`)

### 4. Directory
**Definition**: Container within tenant holding users, groups, applications
- Each tenant has one directory
- Also called "directory service"

### 5. Subscription
**Definition**: Container for Azure resources + billing
- Associated with single Entra ID tenant
- Multiple subscriptions can trust same tenant
- Used for resource access control + billing

### 6. Multi-tenancy
**Definition**: Single app instance serves multiple organizations
- Each organization gets isolated tenant
- Common for SaaS applications

## Quick Reference

### License Feature Comparison
| Feature | Free | M365 Apps | P1 | P2 |
|---------|------|-----------|----|----|
| Users/Groups | ✅ | ✅ | ✅ | ✅ |
| SSO (10 apps) | ✅ | ✅ | ✅ (unlimited) | ✅ (unlimited) |
| MFA | ❌ | ✅ | ✅ | ✅ |
| Conditional Access | ❌ | ❌ | ✅ | ✅ |
| Dynamic Groups | ❌ | ❌ | ✅ | ✅ |
| PIM | ❌ | ❌ | ❌ | ✅ |
| Identity Protection | ❌ | ❌ | ❌ | ✅ |
| Access Reviews | ❌ | ❌ | ❌ | ✅ |

### Common Abbreviations
- **Entra ID**: Microsoft Entra ID (formerly Azure AD)
- **SSO**: Single Sign-On
- **MFA**: Multi-Factor Authentication
- **PIM**: Privileged Identity Management
- **RBAC**: Role-Based Access Control
- **B2B**: Business-to-Business (partner collaboration)
- **B2C**: Business-to-Consumer (customer identity)

## Critical Notes
- ⚠️ **Free tier limits**: Max 10 SSO apps, no MFA, no conditional access
- 💡 **P1 sweet spot**: Best value for mid-size enterprises (conditional access + dynamic groups)
- 🎯 **P2 for compliance**: Required for PIM, identity protection, access reviews
- 📊 **Trial available**: 30-day free trial of P2 for testing
- 🔄 **License assignment**: Assign at user or group level (group-based requires P1+)
- ✨ **Guest licensing**: External B2B users don't consume licenses (1:5 ratio)

[Learn More](https://learn.microsoft.com/en-us/training/modules/intro-to-azure-ad/3-understand-licenses-terminology)
