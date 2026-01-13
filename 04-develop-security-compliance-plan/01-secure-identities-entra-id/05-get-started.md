# Get Started with Microsoft Entra ID

## Phased Deployment Strategy

### Stage 1: Build a Secure Foundation

**Timeline**: 1-2 weeks  
**Goal**: Establish baseline security before adding users

| Task | Description | License |
|------|-------------|---------|
| **Assign multiple Global Admins** | At least 2 accounts, long complex passwords, not used daily | Free |
| **Use regular admin roles** | Principle of least privilege, avoid Global Admin overuse | Free |
| **Configure PIM** | Monitor admin role usage, improve governance | P2 |
| **Enable self-service password reset** | Reduce helpdesk tickets | Free |
| **Create banned password list** | Block common phrases (company name, location) | P1 |
| **Warn against credential reuse** | Prevent password reuse across platforms | Free |
| **Set passwords to never expire** | Avoid incremental password changes (R4ndom1Strong → R4ndom2Strong) | Free |
| **Enforce MFA via Conditional Access** | Require multiple authentication challenges | P1 |
| **Configure ID Protection** | Auto-trigger MFA/password reset based on risk | P2 |

**Critical Actions**:
```yaml
Priority 1 (Day 1):
  - Create 2+ Global Admin accounts (emergency access)
  - Enable MFA for all admins
  - Configure self-service password reset

Priority 2 (Week 1):
  - Implement Conditional Access (require MFA)
  - Create banned password list
  - Enable audit logging
```

---

### Stage 2: Add Users & Configure Synchronization

**Timeline**: 2-4 weeks  
**Goal**: Migrate users, configure hybrid identity, manage devices

| Task | Description | License |
|------|-------------|---------|
| **Install Azure AD Connect** | Sync on-prem AD users to Entra ID | Free |
| **Use password hash sync** | Sync password changes, detect leaked credentials | Free |
| **Enable password writeback** | Cloud password changes write to on-prem AD | P1 |
| **Use Azure AD Connect Health** | Monitor sync health, get alerts | P1 |
| **Group-based licensing** | Assign licenses at group level (not individual) | Free |
| **Enable B2B Collaboration** | External partners use own credentials | Varies |
| **Define device strategy** | BYOD vs corporate devices | Free |
| **Passwordless authentication** | Windows Hello, FIDO2 keys, Microsoft Authenticator | P1 |

**Azure AD Connect Setup**:
```bash
# 1. Download Azure AD Connect from portal
# 2. Install on domain-joined server (on-premises)
# 3. Run configuration wizard
#    - Select sync method (password hash sync recommended)
#    - Provide on-prem AD credentials
#    - Provide Entra ID Global Admin credentials
# 4. Initial sync (can take hours for large directories)
# 5. Configure sync schedule (default: 30 min)
```

**Sync Options**:
| Method | Description | Pros | Cons |
|--------|-------------|------|------|
| **Password Hash Sync** | Hashes synced to cloud | Fast, cloud auth works during on-prem outage | Hashes in cloud |
| **Pass-through Auth** | Auth request sent to on-prem | No passwords in cloud | On-prem dependency |
| **Federation (ADFS)** | Federated trust with on-prem ADFS | Full control, compliance | Complex, high availability needed |

---

### Stage 3: Manage Applications

**Timeline**: 2-4 weeks  
**Goal**: Integrate apps with Entra ID for SSO

| Task | Description | License |
|------|-------------|---------|
| **Identify applications** | Audit all SaaS apps, on-prem apps, custom apps | N/A |
| **Use Entra Gallery apps** | Leverage 3,000+ pre-integrated SaaS apps | Free |
| **Configure Application Proxy** | Secure remote access to on-prem apps | Free |

**App Integration Workflow**:
```yaml
# Entra Gallery App (e.g., Salesforce)
1. Navigate to Entra ID → Enterprise Applications → New Application
2. Search gallery for "Salesforce"
3. Click "Create"
4. Configure SSO (SAML/OIDC)
5. Assign users/groups
6. Test SSO

# On-Premises App via Application Proxy
1. Download Application Proxy connector
2. Install on on-prem server (outbound HTTPS to Azure)
3. Entra ID → App Proxy → Create new application
4. Internal URL: http://internal-app.contoso.com
5. External URL: https://internal-app-contoso.msappproxy.net
6. Configure SSO (Kerberos/header-based)
7. Assign users
```

---

### Stage 4: Monitor Admins & Automate Lifecycle

**Timeline**: 2-3 weeks  
**Goal**: Implement governance, access reviews, automated provisioning

| Task | Description | License |
|------|-------------|---------|
| **Use PIM for admin access** | Just-in-time admin access, MFA required | P2 |
| **Complete access reviews** | Periodic review of admin roles | P2 |
| **Configure dynamic groups** | Auto-add users based on attributes (department, location) | P1/P2 |
| **Group-based app assignment** | Assign app access to groups (auto-remove when left) | Free |
| **Automate user provisioning** | Auto-create/deprovision SaaS app accounts | Free |

**Dynamic Group Example**:
```yaml
Group: "Sales-US"
Rule: 
  (user.department -eq "Sales") -and (user.country -eq "United States")
Result:
  - Auto-adds users in Sales department in US
  - Auto-removes when department/country changes
License: P1 or P2
```

**Provisioning Automation**:
```yaml
# Automated Provisioning Example (Salesforce)
1. Entra ID → Enterprise Apps → Salesforce → Provisioning
2. Enable "Automatic" provisioning
3. Configure API credentials (Salesforce admin token)
4. Attribute mappings:
   - Entra ID email → Salesforce username
   - Entra ID department → Salesforce profile
5. Scope: "All users" or "Assigned users/groups"
6. Start provisioning (sync every 40 min)

Result:
  - New user added to "Sales" group → Auto-created in Salesforce
  - User leaves org → Auto-deactivated in Salesforce
```

---

## Tenant Creation

### Create Tenant
**Path**: Microsoft Entra admin center → Create a tenant

**Form Fields**:
- Organization name: `Contoso`
- Initial domain: `contoso.onmicrosoft.com`
- Country/Region: `United States`

**Result**: New tenant with Global Admin access

### Associate Subscription
**Path**: Azure Portal → Subscriptions → [Your Subscription] → Change directory

**Steps**:
1. Select subscription
2. Click "Change directory"
3. Select new Entra ID tenant
4. Confirm change

**Result**: Subscription now trusts new tenant (resources accessible by tenant users)

---

## Quick Reference

### Deployment Timeline
| Stage | Timeline | Key Deliverables |
|-------|----------|------------------|
| **Stage 1** | 1-2 weeks | MFA enabled, Conditional Access, self-service password reset |
| **Stage 2** | 2-4 weeks | Azure AD Connect syncing, B2B enabled, passwordless auth |
| **Stage 3** | 2-4 weeks | 10+ apps integrated, Application Proxy configured |
| **Stage 4** | 2-3 weeks | PIM enabled, access reviews running, dynamic groups |
| **Total** | 7-13 weeks | Full Entra ID deployment |

### Essential Commands
```bash
# Azure AD Connect Sync (PowerShell)
Start-ADSyncSyncCycle -PolicyType Delta  # Incremental sync
Start-ADSyncSyncCycle -PolicyType Initial # Full sync

# Check last sync time
Get-ADSyncScheduler

# Entra ID Module (PowerShell)
Connect-AzureAD
Get-AzureADUser -Filter "department eq 'Sales'"
Get-AzureADGroup -Filter "displayName eq 'Sales-US'"
```

## Critical Notes
- ⚠️ **Emergency access accounts**: Create 2+ cloud-only Global Admin accounts (break-glass)
- 💡 **Phased approach reduces risk**: Don't enable all features at once
- 🎯 **Password hash sync recommended**: Fastest, most resilient hybrid identity option
- 📊 **Dynamic groups save time**: Auto-manage group membership (requires P1)
- 🔄 **Provisioning = lifecycle automation**: Create/update/delete accounts automatically
- ✨ **PIM = just-in-time admin access**: Activate admin roles only when needed (requires P2)

[Learn More](https://learn.microsoft.com/en-us/training/modules/intro-to-azure-ad/5-get-started)
