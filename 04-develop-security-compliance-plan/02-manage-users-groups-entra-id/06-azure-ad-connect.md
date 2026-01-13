# Connect Active Directory to Microsoft Entra ID with Microsoft Entra Connect

## Key Concepts
- **Azure AD Connect**: Free tool to synchronize on-premises AD with Entra ID
- **Hybrid Identity**: Same user credentials for on-premises and cloud resources
- **Common Identity**: Single identity for Microsoft 365, Azure, SaaS apps
- **SSO (Single Sign-On)**: Users authenticate once, access all resources

## What's Included in Microsoft Entra Connect?

### 1. Sync Services
**Purpose**: Core synchronization engine

**Syncs**:
- Users
- Groups
- Passwords (hash or pass-through)
- Contacts
- Device objects

**Frequency**: Every 30 minutes (default, configurable)

---

### 2. Health Monitoring
**Purpose**: Microsoft Entra Connect Health provides monitoring and alerts

**Features**:
- Real-time sync status
- Performance metrics
- Alert notifications (email, portal)
- Historical sync reports
- Centralized view in Azure Portal

**Access**: Azure Portal → Microsoft Entra Connect Health

---

### 3. Active Directory Federation Services (AD FS) - Optional
**Purpose**: Federation for complex scenarios

**Use Cases**:
- Domain-join SSO (Kerberos/NTLM)
- Enforce on-premises sign-in policies (IP restrictions, device compliance)
- Smart card authentication
- Third-party MFA integration
- Legacy apps requiring federation

**Architecture**:
```
User → AD FS (on-prem) → Entra ID → Cloud resources
```

---

### 4. Password Hash Synchronization
**Purpose**: Sync password hashes to cloud

**How It Works**:
```
On-Prem AD Password Hash → One-way hash → Azure AD Connect → Entra ID
```

**Benefits**:
- Cloud authentication (no on-prem dependency for sign-in)
- Leaked credential detection (ID Protection)
- Fast, simple, resilient

**Security**: Original password never leaves on-prem (only hash of hash sent)

---

### 5. Pass-Through Authentication
**Purpose**: Authenticate against on-premises AD directly

**How It Works**:
```
User → Entra ID → Auth request → On-Prem Agent → On-Prem AD → Result
```

**Benefits**:
- No passwords/hashes in cloud
- Enforce on-premises password policies
- Immediate password expiration/account lockout

**Trade-off**: On-premises dependency (if AD unavailable, cloud auth fails)

---

## Benefits of Integration

| Benefit | Description |
|---------|-------------|
| **Common Identity** | Single identity for on-premises apps, Microsoft 365, Azure, SaaS |
| **SSO** | Authenticate once, access all resources (no repeated logins) |
| **Productivity** | Users don't forget multiple passwords, fewer helpdesk tickets |
| **Easy Deployment** | Single tool provides sync, authentication, health monitoring |
| **Modern Auth** | MFA, Conditional Access, passwordless (Windows Hello) |
| **Replaces Legacy Tools** | Supersedes DirSync, Azure AD Sync |

---

## Installation Overview

### Prerequisites
- On-premises Active Directory (Windows Server 2016 or later)
- Azure subscription with Global Administrator access
- Domain-joined Windows Server (for Azure AD Connect)
- Network connectivity to Azure (outbound HTTPS)
- SQL Server (Express included, Enterprise optional for large deployments)

### High-Level Steps
1. **Download Azure AD Connect** (from Azure Portal)
2. **Run installation wizard**
   - Express Settings (recommended for most)
   - Custom Settings (advanced scenarios: multi-forest, AD FS)
3. **Select authentication method**:
   - Password Hash Sync (recommended)
   - Pass-Through Authentication
   - Federation (AD FS)
4. **Provide credentials**:
   - On-prem AD credentials (Enterprise Admin)
   - Entra ID credentials (Global Administrator)
5. **Select objects to sync**:
   - All OUs (default)
   - Selected OUs (optional)
6. **Configure optional features**:
   - Password writeback
   - Group writeback
   - Device writeback
   - Exchange hybrid deployment
7. **Start initial sync**:
   - Can take minutes to hours (depends on object count)
   - Monitor progress in Azure Portal

---

## Authentication Methods Comparison

| Method | Password Location | On-Prem Dependency | Complexity | Use Case |
|--------|-------------------|---------------------|------------|----------|
| **Password Hash Sync** | Hashes in cloud | No | Low | Recommended for most |
| **Pass-Through Auth** | On-premises only | Yes | Medium | Compliance requirements (no cloud passwords) |
| **Federation (AD FS)** | On-premises only | Yes | High | Complex SSO, smart cards, third-party MFA |

### Decision Guide
```
Need simple, resilient auth?
  → Password Hash Sync

Compliance prohibits passwords in cloud?
  → Pass-Through Authentication

Need smart cards or legacy federation?
  → Federation (AD FS)
```

---

## Quick Reference

### Architecture Diagram
```
┌─────────────────────────────────────────────┐
│ On-Premises Environment                     │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │ Windows Server Active Directory      │  │
│  │  - Users: alice, bob, charlie        │  │
│  │  - Groups: Sales, Engineering        │  │
│  └──────────────────────────────────────┘  │
│                 ↓ syncs                     │
│  ┌──────────────────────────────────────┐  │
│  │ Azure AD Connect                     │  │
│  │  - Installed on domain-joined server │  │
│  │  - Sync every 30 min                 │  │
│  └──────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
                  ↓ HTTPS
┌─────────────────────────────────────────────┐
│ Azure Cloud (Microsoft Entra ID)            │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │ Microsoft Entra ID                   │  │
│  │  - Users: alice@contoso.com          │  │
│  │  - Groups: Sales, Engineering        │  │
│  │  - Synced from on-prem AD            │  │
│  └──────────────────────────────────────┘  │
│                 ↓ provides access           │
│  Microsoft 365, Azure, SaaS Apps            │
└─────────────────────────────────────────────┘
```

### Sync Behavior
| Object Type | Sync Direction | Notes |
|-------------|----------------|-------|
| **Users** | On-prem → Cloud | Source of truth = on-prem AD |
| **Groups** | On-prem → Cloud | Cloud-created groups not synced back |
| **Passwords** | Bidirectional (if writeback enabled) | Users can reset password in cloud → writes to AD |
| **Devices** | Bidirectional (if writeback enabled) | Azure AD-joined devices write to on-prem |

### Common PowerShell Commands
```powershell
# Trigger manual sync
Start-ADSyncSyncCycle -PolicyType Delta      # Incremental sync
Start-ADSyncSyncCycle -PolicyType Initial    # Full sync

# Check sync status
Get-ADSyncScheduler

# View last sync time
Get-ADSyncScheduler | Select-Object LastSyncTime

# Disable sync schedule (maintenance)
Set-ADSyncScheduler -SyncCycleEnabled $false

# Re-enable sync schedule
Set-ADSyncScheduler -SyncCycleEnabled $true
```

---

## Critical Notes
- ⚠️ **Not trivial installation**: Requires planning, testing, rollback strategy
- 💡 **Password Hash Sync recommended**: Best balance of security, resilience, simplicity
- 🎯 **Follow installation guide**: [Microsoft Entra Connect Installation Roadmap](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-install-roadmap)
- 📊 **Initial sync time**: 50,000 objects ≈ 2-4 hours (depends on network, server specs)
- 🔄 **Sync frequency**: Default 30 min (don't change unless needed, increases server load)
- ✨ **Staging mode**: Test config without syncing (dry run mode)

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-users-and-groups-in-aad/6-azure-ad-connect)
