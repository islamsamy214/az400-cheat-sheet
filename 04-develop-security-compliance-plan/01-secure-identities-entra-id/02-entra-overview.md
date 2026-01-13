# Microsoft Entra Overview

## Key Concepts
- **Microsoft Entra ID** (formerly Azure Active Directory): Cloud identity platform
- **Identity Provider (IdP)**: Centralized authentication and authorization service
- **Difference from AD**: Cloud-native vs on-premises, REST API vs LDAP, flat structure vs hierarchical OU
- **Hybrid Identity**: Synchronize on-premises AD with Entra ID via Azure AD Connect

## Core Components

### 1. Tenants
- **Definition**: Dedicated Entra ID instance representing organization
- **Isolation**: Each tenant isolated and separate
- **Multi-tenant**: Organizations can have multiple tenants

### 2. Identities
| Type | Description | Use Case |
|------|-------------|----------|
| **Users** | Internal employees, external partners | Staff access |
| **Service Principals** | Application identities | App-to-app auth |
| **Managed Identities** | Auto-managed Azure service identities | Eliminate hardcoded credentials |
| **Devices** | Registered/joined devices | Device-based conditional access |

### 3. Groups
- **Security Groups**: Manage access to resources
- **Microsoft 365 Groups**: Shared mailbox, calendar, files, SharePoint site
- **Dynamic Groups** (P1+): Auto-membership based on user attributes

## Quick Reference

### On-Premises AD vs Microsoft Entra ID
| Feature | On-Premises AD | Microsoft Entra ID |
|---------|----------------|-------------------|
| **Protocol** | LDAP, Kerberos | OAuth 2.0, SAML, OpenID Connect |
| **Structure** | Hierarchical OUs | Flat structure |
| **Query** | LDAP queries | REST API (Microsoft Graph) |
| **Authentication** | NTLM, Kerberos | Modern auth (MFA, passwordless) |
| **Location** | On-premises DC | Cloud (global) |
| **Management** | Group Policy | Conditional Access, Intune |

### Key Services
1. **Authentication**: SSO, MFA, passwordless (biometric, FIDO2)
2. **Authorization**: RBAC, conditional access policies
3. **Application Management**: 3,000+ pre-integrated SaaS apps
4. **Device Management**: Register, join, manage access
5. **Identity Protection**: Risk detection, automated remediation

## Critical Notes
- ⚠️ **Not a replacement for AD**: Complement, not full replacement (some services need on-premises AD)
- 💡 **Microsoft Graph API**: Unified endpoint for Entra ID queries (replaces LDAP)
- 🎯 **Global availability**: Microsoft runs Entra ID in 40+ datacenters worldwide
- 📊 **Built for cloud**: Designed for internet-scale identity management
- 🔄 **Hybrid identity**: Azure AD Connect syncs users, groups, passwords bidirectionally
- ✨ **Zero Trust ready**: Identity-based security, verify explicitly, least privilege

[Learn More](https://learn.microsoft.com/en-us/training/modules/intro-to-azure-ad/2-overview)
