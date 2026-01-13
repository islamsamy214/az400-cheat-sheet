# Essential Features of Microsoft Entra ID

## Key Features

### 1. Microsoft Entra B2B (Business-to-Business)
**Purpose**: Invite external partners to collaborate while maintaining security

**Workflow**:
```
1. Invite external user → Email with redemption link
2. User clicks link → Redirected to authentication
3. MFA challenge (if enabled) → Verification code
4. User provides code → Access granted to invited resources
```

**Key Benefits**:
- External partners use their own credentials (no password management)
- Control access per resource
- Revoke access anytime
- Available in **all license tiers**

**Use Case**: Healthcare org invites external partner doctors for project collaboration

---

### 2. Azure AD B2C (Business-to-Consumer)
**Purpose**: Manage customer identities for public-facing applications

**Workflow**:
```
1. Customer accesses app → Redirected to sign-in
2. Customer completes sign-in form → Credentials verified
3. MFA (if enabled) → Verification code
4. Customer provides code → Access granted
```

**Key Capabilities**:
- Social identity providers (Google, Facebook, Microsoft)
- Custom branding and user flows
- Threat protection (brute force, DDoS detection)
- Self-service password reset
- **Pricing**: Pay-as-you-go (separate from Entra ID licenses)

**Use Case**: Doctor portal for patients (50,000+ users)

---

### 3. Microsoft Entra Domain Services
**Purpose**: Managed domain services (domain join, LDAP, Kerberos) without domain controllers

**Architecture**:
```
On-Premises AD → Azure AD Connect Sync → Entra ID → Entra Domain Services
                                           ↓
                                    Azure VMs (domain-joined)
```

**Key Features**:
- Domain join for Azure VMs
- LDAP read support
- Kerberos/NTLM authentication
- No need to manage domain controllers
- **Pricing**: Pay-as-you-go (based on total objects)

**Use Case**: Lift-and-shift legacy apps requiring domain authentication

---

### 4. Application Management

#### App Categories
| Category | Description | Example |
|----------|-------------|---------|
| **Entra Gallery Apps** | 3,000+ pre-integrated SaaS apps | Salesforce, GitHub, Slack |
| **Non-Gallery Apps** | Manually add custom SaaS apps | Internal partner portals |
| **On-Premises Apps** | Via Application Proxy | Internal legacy web apps |
| **Custom Apps** | Company-built applications | In-house ERP system |

#### Application Proxy
**Purpose**: Secure remote access to on-premises web apps

**Setup**:
```bash
# 1. Download connector
# 2. Install on-premises server
# 3. Connector creates outbound HTTPS connection to Azure
# 4. Users access via external URL (no VPN needed)
```

**Benefits**:
- No inbound firewall rules
- No VPN required
- Pre-authentication via Entra ID
- Conditional Access support

---

### 5. Conditional Access
**Purpose**: Enforce access policies based on conditions

**Example Policy**:
```yaml
Policy: "Require MFA for High-Risk Sign-Ins"
Conditions:
  - User: All users
  - Cloud apps: Microsoft 365
  - Sign-in risk: High
Access Controls:
  - Require MFA
  - Block legacy authentication
```

**Common Conditions**:
- User/group membership
- Location (trusted IPs)
- Device compliance (managed devices)
- Sign-in risk level
- Client app type

**License**: Premium P1 required

---

### 6. Monitoring & Reports

**Available Reports**:
| Report Type | Description | License |
|-------------|-------------|---------|
| **Sign-in logs** | Who, what, when, where | All tiers |
| **Audit logs** | Changes to users, groups, policies | All tiers |
| **Risky sign-ins** | Anomalous sign-in attempts | P2 |
| **Risky users** | Users with compromised credentials | P2 |
| **Usage & insights** | App usage patterns | P1 |

**Access**:
- Azure Portal → Entra ID → Monitoring & Health → Reports
- Microsoft Graph API (programmatic)
- Microsoft Entra admin center

---

### 7. Microsoft Entra ID Protection
**Purpose**: Automated identity risk detection and remediation

**Risk Types**:
- **Sign-in risk**: Atypical travel, anonymous IP, malware-linked IP, unfamiliar properties
- **User risk**: Leaked credentials, anomalous behavior

**Risk Policies**:
```yaml
# Example: User Risk Policy
Policy: "Require password change for high-risk users"
Conditions:
  - User risk level: High
Actions:
  - Require password change
  - Require MFA
  - Block access (if critical)
```

**Workflow**:
```
1. Admin configures risk policies → Automated monitoring
2. Risk detected (e.g., leaked credentials) → Policy triggered
3. User required to reset password → Self-remediation
4. Password reset completed → Risk cleared
```

**License**: Premium P2 required

---

## Quick Reference

### Feature-to-License Mapping
| Feature | Free | M365 | P1 | P2 |
|---------|------|------|----|----|
| B2B Collaboration | ✅ | ✅ | ✅ | ✅ |
| B2C (pay-as-you-go) | ✅ | ✅ | ✅ | ✅ |
| Domain Services (pay-as-you-go) | ✅ | ✅ | ✅ | ✅ |
| App Management | ✅ (10 apps) | ✅ | ✅ (unlimited) | ✅ |
| Conditional Access | ❌ | ❌ | ✅ | ✅ |
| Sign-in Reports | ✅ | ✅ | ✅ | ✅ |
| Identity Protection | ❌ | ❌ | ❌ | ✅ |

### Knowledge Check Answers
**Q1**: What does Microsoft Entra B2B provide?  
✅ **Allows you to invite external users to your tenant for collaboration**

**Q2**: What does Microsoft Entra Application Proxy do?  
✅ **Adds on-premises applications to Entra ID for secure remote access**

## Critical Notes
- ⚠️ **B2B vs B2C**: B2B = partner collaboration, B2C = customer identity (separate services)
- 💡 **Application Proxy**: No VPN or inbound firewall rules needed
- 🎯 **Conditional Access**: Most powerful security feature (requires P1)
- 📊 **Identity Protection**: AI-powered risk detection (requires P2)
- 🔄 **Domain Services**: For legacy app migration (not needed for cloud-native apps)
- ✨ **Self-service**: Policies enable users to self-remediate (password reset, MFA registration)

[Learn More](https://learn.microsoft.com/en-us/training/modules/intro-to-azure-ad/4-essential-features)
