# Understand Microsoft Defender for Identity

## Key Concepts
- **Identity threat detection**: Cloud-based solution for on-premises Active Directory
- **Lightweight sensors**: Installed on domain controllers, no port mirroring required
- **Behavioral analytics**: Machine learning detects credential theft, lateral movement
- **MITRE ATT&CK mapping**: Alerts aligned with framework tactics/techniques
- **Microsoft Defender XDR integration**: Unified security operations across identities, endpoints, email, cloud

## Architecture and Deployment

### Components
| Component | Description | Location |
|-----------|-------------|----------|
| **Defender for Identity portal** | Management portal for alerts, investigations, configuration | https://security.microsoft.com/ |
| **Sensors** | Lightweight agents monitoring authentication traffic | Domain controllers |
| **Microsoft Defender XDR** | Unified security operations platform | Cloud |
| **Cloud service** | Analytics engine, ML models, threat intelligence | Azure |

### Deployment Prerequisites
- ✅ **Active Directory**: On-premises AD DS
- ✅ **Domain controllers**: Windows Server 2012 or later
- ✅ **Network connectivity**: Outbound HTTPS (port 443) to Defender cloud service
- ✅ **Licensing**: Microsoft 365 E5, EMS E5, or standalone licenses
- ✅ **Permissions**: Domain Admin or Enterprise Admin for sensor installation

### Install Defender for Identity Sensor
```bash
# Installation steps:
# 1. Download sensor installer from Defender for Identity portal
# 2. Copy installer to domain controller
# 3. Run installer as administrator with Access Key from portal
# 4. Sensor registers with cloud service and begins monitoring
# 5. Repeat on ALL domain controllers for comprehensive coverage
```

**Best practice**: Install on **all domain controllers** for complete visibility into authentication traffic

## Threat Detection Capabilities

### Credential Theft Detection
| Attack Type | Detection Method | MITRE ATT&CK |
|-------------|------------------|--------------|
| **Pass-the-Hash** | NTLM hash reuse patterns | T1550.002 |
| **Pass-the-Ticket** | Kerberos ticket theft and reuse | T1550.003 |
| **Over-pass-the-Hash** | NTLM to Kerberos ticket conversion | T1550.002 |
| **Golden Ticket** | Forged Kerberos TGTs (domain-wide access) | T1558.001 |
| **Silver Ticket** | Forged service tickets (specific resources) | T1558.002 |
| **Skeleton Key** | Backdoor passwords for any account | T1556.004 |

### Reconnaissance Activities
- 🔍 **Account enumeration**: Probing for valid usernames
- 🗺️ **Network mapping**: SMB session enumeration
- 📡 **DNS reconnaissance**: Suspicious DNS queries mapping infrastructure
- 📂 **Directory services queries**: Unusual LDAP queries gathering domain info

### Lateral Movement
- 🖥️ **Remote execution**: PsExec, WMI, PowerShell remoting, scheduled tasks
- 🛠️ **Suspicious service creation**: Services created for persistence
- 🔑 **Overpass-the-hash**: Credential reuse across systems

### Domain Dominance
- 👑 **DCSync attacks**: Replication requests extracting all domain credentials
- 👻 **DCShadow attacks**: Unauthorized domain controller registration
- ⚠️ **Suspicious DC promotions**: Unauthorized DC installations
- 🦴 **Skeleton key malware**: Domain controller backdoors

### Compromised Accounts
- 📊 **Abnormal behavior**: Unusual auth patterns, working hours violations, geographic anomalies
- 🔒 **Brute force attacks**: Password spraying, credential stuffing
- ⚠️ **Suspicious authentications**: Broken trust, expired accounts, unusual protocols
- 🌐 **VPN anomalies**: Suspicious remote access patterns

## Alert Investigation

### Security Alert Properties
| Property | Description | Example |
|----------|-------------|---------|
| **Severity** | High, Medium, Low | High: DCSync attack |
| **Category** | Threat type | Credential theft, lateral movement |
| **MITRE ATT&CK** | Framework alignment | T1003.006 (DCSync) |
| **Affected entities** | Users, computers, DCs involved | CORP\admin, DC01.corp.local |
| **Evidence** | Timeline, IPs, auth attempts | 2024-01-15 14:32:18 UTC, IP: 10.0.1.50 |

### Investigation Workflow
```
1. Review Alert in Defender Portal
   ↓
2. Examine Alert Details
   - Timeline of suspicious activities
   - Affected accounts and computers
   - Source/destination IP addresses
   - Authentication methods used
   ↓
3. Check User Risk Level
   - User's risk score
   - Previous suspicious activities
   ↓
4. Investigate Related Alerts
   - Correlated alerts (multi-stage attack)
   ↓
5. Review User Activity Timeline
   - All activities around alert timestamp
   ↓
6. Take Response Actions
   - Disable compromised account
   - Reset password
   - Require MFA re-enrollment
   - Isolate affected systems
```

### Response Actions
```powershell
# Disable compromised account
Disable-ADAccount -Identity "compromised.user"

# Reset password
Set-ADAccountPassword -Identity "compromised.user" -Reset

# Force re-enrollment of MFA
Set-MsolUser -UserPrincipalName "compromised.user@corp.com" -StrongAuthenticationMethods @()

# Remove user from privileged groups
Remove-ADGroupMember -Identity "Domain Admins" -Members "compromised.user"

# Isolate affected computer
Disable-ADAccount -Identity "WORKSTATION01$"
```

### Automated Investigation and Response
**Microsoft Defender XDR capabilities**:
- 🔄 **Automatic alert correlation**: Groups related alerts into incidents
- 🔍 **Automated investigation**: Analyzes alerts, examines entities, identifies scope
- 📋 **Recommended actions**: Remediation recommendations based on findings
- ⚡ **Automated remediation**: Optional auto-execution of response actions

## Integration with Microsoft Defender XDR

### Unified Incident Management
**Cross-domain incidents**:
- 🆔 Defender for Identity (identity threats)
- 💻 Defender for Endpoint (endpoint threats)
- 📧 Defender for Office 365 (email threats)
- ☁️ Defender for Cloud Apps (cloud app threats)

**Benefits**:
- ✅ Complete attack story across entire attack surface
- ✅ Single investigation interface
- ✅ Full timeline spanning identity → endpoint → email → cloud

### Advanced Hunting (KQL)
```kql
// Find all Pass-the-Hash attacks in last 7 days
IdentityLogonEvents
| where Timestamp > ago(7d)
| where ActionType == "PassTheHashDetection"
| project Timestamp, AccountName, DeviceName, LogonType, IPAddress
| order by Timestamp desc

// Detect lateral movement from compromised account
IdentityLogonEvents
| where Timestamp > ago(24h)
| where AccountName == "compromised.user"
| where LogonType == "RemoteInteractive"
| summarize DestinationCount = dcount(DeviceName) by AccountName, IPAddress
| where DestinationCount > 5  // More than 5 unique destinations

// Find DCSync attempts
IdentityDirectoryEvents
| where Timestamp > ago(7d)
| where ActionType == "Replication request"
| where AccountName !startswith "DC"  // Exclude legitimate DCs
| project Timestamp, AccountName, TargetDeviceName, AdditionalFields
```

### Automated Response Configuration
**Logic Apps integration**:
```json
{
  "trigger": {
    "type": "Defender for Identity High Severity Alert"
  },
  "actions": [
    {
      "type": "Disable user account",
      "account": "[alert.affectedUser]"
    },
    {
      "type": "Create ServiceNow incident",
      "severity": "P1",
      "description": "[alert.description]"
    },
    {
      "type": "Send email to SOC",
      "recipients": "soc@corp.com",
      "subject": "Critical Identity Alert: [alert.title]"
    }
  ]
}
```

## Best Practices

### Comprehensive Sensor Deployment
```bash
# Install sensors on ALL domain controllers
# Coverage checklist:
✅ Primary domain controller
✅ Additional domain controllers
✅ Read-only domain controllers (RODCs)
✅ Branch office domain controllers
```

### Enable Audit Policies
```powershell
# Configure Advanced Audit Policy on domain controllers
auditpol /set /category:"Account Logon" /success:enable /failure:enable
auditpol /set /category:"Account Management" /success:enable /failure:enable
auditpol /set /category:"DS Access" /success:enable /failure:enable
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable

# Verify configuration
auditpol /get /category:*
```

### Regular Alert Review
**SOC workflow**:
1. ✅ Review Defender for Identity alerts daily
2. ✅ Prioritize High and Medium severity alerts
3. ✅ Investigate within 4 hours (High), 24 hours (Medium)
4. ✅ Document investigation findings
5. ✅ Track remediation actions to completion

### Tune Alert Sensitivity
**Learning period**:
- 📊 Defender for Identity baselines normal user behaviors
- ⏱️ Learning period: 30 days recommended
- 🎯 Reduces false positives by understanding organizational patterns

**Tuning actions**:
```bash
# Exclude known administrative activities
# Portal: Settings → Detection tuning → Exclude entities
# Example: Exclude service accounts from VPN anomaly detection
```

### Integrate with SIEM
```bash
# Export Defender for Identity alerts to Azure Sentinel
# Portal: Settings → Integrations → Microsoft Sentinel
# Correlation with:
# - Firewall logs
# - VPN logs
# - Application logs
# - Other security data sources
```

## Detection Examples

### Pass-the-Hash Detection
```
Alert: "Suspected Pass-the-Hash attack"
Severity: High
Account: CORP\user01
Source IP: 10.0.2.50
Target: DC01.corp.local
Description: NTLM hash used for authentication without password
Evidence: NTLM authentication from non-domain-joined device
MITRE: T1550.002
```

### Golden Ticket Detection
```
Alert: "Suspected Golden Ticket usage (encryption downgrade)"
Severity: High
Account: CORP\admin
Source IP: 10.0.3.75
Description: Forged Kerberos TGT with unusual encryption
Evidence: TGT with downgraded encryption, lifetime of 10 years
MITRE: T1558.001
```

### DCSync Attack Detection
```
Alert: "Suspected DCSync attack (replication of directory services)"
Severity: High
Account: CORP\serviceaccount
Target DC: DC01.corp.local
Description: Non-DC account requesting AD replication (credential extraction)
Evidence: Directory Replication Get Changes All requested
MITRE: T1003.006
```

## Integration Architecture

```
On-Premises Active Directory
    ↓ (Authentication traffic)
Domain Controllers with Sensors
    ↓ (HTTPS 443)
Defender for Identity Cloud Service
    ↓ (Integration)
Microsoft Defender XDR Portal
    ├── Defender for Endpoint
    ├── Defender for Office 365
    ├── Defender for Cloud Apps
    └── Azure Sentinel (SIEM)
```

## Critical Notes
- 🎯 **Install on ALL DCs**: Comprehensive coverage essential for complete visibility
- 📊 **30-day learning period**: Allows behavioral baseline establishment
- 🔔 **Daily alert review**: High/Medium severity within 4/24 hours
- 🔄 **Enable Advanced Audit Policies**: Captures detailed security events
- 💡 **MITRE ATT&CK mapping**: Provides structured investigation framework
- ⚡ **Automated response**: Configure Logic Apps for immediate response
- 🛡️ **Defense in depth**: Combine with Defender for Endpoint, MFA, Conditional Access
- 📝 **SIEM integration**: Correlate with other security data sources

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/9-understand-microsoft-defender-identity)
