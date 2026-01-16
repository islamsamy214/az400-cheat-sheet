# Explore Microsoft Defender for Cloud

## Key Concepts
- **CSPM** (Cloud Security Posture Management): Configuration security, Secure Score, recommendations
- **CWP** (Cloud Workload Protection): Runtime threat detection, vulnerability management
- **DevSecOps Integration**: IaC scanning in pipelines, shift-left security
- **Multi-cloud support**: Azure, AWS, GCP, on-premises hybrid environments
- **Unified security management**: Single dashboard for entire cloud estate

## Core Capabilities

| Capability | Description | Use Case |
|------------|-------------|----------|
| **Foundational CSPM** | Free configuration assessments | Basic security posture visibility |
| **Defender CSPM** | Advanced threat protection, attack path analysis | Enterprise security operations |
| **Workload Protection** | Runtime detection for VMs, containers, databases | Production workload security |
| **DevSecOps** | IaC scanning, code-to-cloud security | Shift-left security automation |

## Defender Plans

### Available Plans
```bash
# Enable Defender plans
az security pricing create \
  --name VirtualMachines \
  --tier Standard

az security pricing create \
  --name Containers \
  --tier Standard

az security pricing create \
  --name SqlServers \
  --tier Standard
```

| Plan | Protection Scope | Key Features |
|------|------------------|--------------|
| **Servers** | VMs, Azure Arc servers | Vulnerability assessment, JIT VM access, file integrity monitoring |
| **Containers** | AKS, ACR, container registries | Image scanning, runtime threat detection, admission control |
| **App Service** | Web apps, APIs | Threat detection, vulnerability scanning |
| **Storage** | Blob, Files, Data Lake | Malware scanning, anomaly detection |
| **SQL** | Azure SQL, SQL on VMs | Vulnerability assessment, threat detection, data classification |
| **Key Vault** | Key vaults | Anomalous access patterns, certificate lifecycle monitoring |
| **Resource Manager** | ARM operations | Suspicious management activity, privilege escalation |
| **DNS** | DNS queries | DNS tunneling, malicious domain resolution |

## Architecture

### Azure Monitor Agent (AMA)
```bash
# Automatically provisioned by Defender for Cloud
# Collects security events, logs, performance data
az vm extension set \
  --publisher Microsoft.Azure.Monitor \
  --name AzureMonitorLinuxAgent \
  --vm-name production-vm \
  --resource-group production-rg
```

### Defender for Endpoint Integration
- **Unified endpoint protection**: Windows, Linux, macOS, mobile devices
- **EDR capabilities**: Advanced threat hunting, automated investigation
- **Attack surface reduction**: Exploit protection, network protection

### Agentless Scanning
```bash
# Enable agentless scanning (no agent installation required)
az security setting update \
  --name MCSB \
  --enabled true
```
- **Disk snapshots**: Read-only analysis of VM disks
- **No performance impact**: Scanning outside production VMs
- **Software inventory**: Installed applications, missing patches
- **Secret scanning**: Exposed credentials, certificates in disk

## Enabling Defender for Cloud

```bash
# Enable Foundational CSPM (free)
az security pricing create \
  --name CloudPosture \
  --tier Free

# Enable Defender plans (paid)
az security pricing create \
  --name VirtualMachines \
  --tier Standard

# Configure automatic agent provisioning
az security auto-provisioning-setting update \
  --name default \
  --auto-provision On
```

### Foundational CSPM (Free)
- ✅ Security recommendations
- ✅ Secure Score
- ✅ Asset inventory
- ❌ No threat protection
- ❌ No vulnerability scanning

### Defender Plans (Paid)
- ✅ All CSPM features
- ✅ Threat protection alerts
- ✅ Vulnerability assessment
- ✅ JIT VM access
- ✅ Regulatory compliance dashboard

## Secure Score

### Calculation Formula
```
Secure Score = (Healthy Resources / Total Resources) × Recommendation Points
```

### Score Range: 0-100
| Score Range | Security Posture | Action Required |
|-------------|------------------|-----------------|
| **80-100** | Excellent | Maintain current practices |
| **60-79** | Good | Address medium-priority findings |
| **40-59** | Fair | Remediate high-priority issues |
| **0-39** | Poor | Immediate action required |

### View Secure Score
```bash
# Get current Secure Score
az security secure-score show \
  --name ascScore \
  --query "score.current"

# List recommendations by potential score increase
az security assessment list \
  --query "sort_by([?status.code=='Unhealthy'], &properties.weight) | reverse(@)" \
  --output table
```

## Security Recommendations

### Severity Levels
- **High**: Critical misconfigurations (public databases, disabled encryption)
- **Medium**: Important security gaps (missing MFA, outdated software)
- **Low**: Minor improvements (missing tags, diagnostic settings)

### Recommendation Categories
| Category | Examples | Impact |
|----------|----------|--------|
| **Identity** | MFA, privileged access, conditional access | Prevent credential theft |
| **Data** | Encryption, TLS, secure transfer | Protect sensitive data |
| **Networking** | NSGs, private endpoints, DDoS protection | Network segmentation |
| **Compute** | Patching, disk encryption, endpoint protection | Workload hardening |
| **Application** | WAF, HTTPS only, secure headers | Application security |

### Remediation Approaches

#### Manual Remediation
```bash
# Example: Enable storage encryption
az storage account update \
  --name contosodata \
  --resource-group production-rg \
  --encryption-services blob file \
  --encryption-key-source Microsoft.Storage
```

#### Quick Fix (Automated Remediation)
1. Navigate to recommendation in Defender for Cloud
2. Click **Quick fix** button
3. Select affected resources (bulk selection supported)
4. Click **Remediate** - Defender executes remediation automatically

#### Azure Policy Enforcement (Preventive)
```json
{
  "policyRule": {
    "if": {
      "allOf": [
        {"field": "type", "equals": "Microsoft.Storage/storageAccounts"},
        {"field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly", "notEquals": "true"}
      ]
    },
    "then": {"effect": "deny"}
  }
}
```

## Security Alerts

### Alert Types
- **Suspicious activity**: Unusual authentication patterns, unexpected process execution
- **Threat intelligence**: Known malicious IPs, domains, file hashes
- **Anomaly detection**: Machine learning-based behavioral analysis
- **Brute force attacks**: Password spraying, credential stuffing
- **Malware detection**: Virus signatures, ransomware patterns

### MITRE ATT&CK Mapping
```bash
# Query alerts by MITRE technique
az security alert list \
  --query "[?properties.techniques[?contains(@, 'T1078')]]" \
  --output table
```

| Tactic | Examples | Detection Methods |
|--------|----------|-------------------|
| **Initial Access** | T1190 (Exploit Public-Facing App) | Web application firewall logs |
| **Credential Access** | T1110 (Brute Force) | Failed login pattern analysis |
| **Lateral Movement** | T1021 (Remote Services) | Network traffic anomalies |
| **Exfiltration** | T1048 (Exfiltration Over Alternative Protocol) | Unusual data transfer volumes |

### Investigation Workflow
```bash
# List high-severity alerts
az security alert list \
  --filter "properties/severity eq 'High'" \
  --query "[].{Name:name, Resource:compromisedEntity, Time:startTimeUtc}"

# Get alert details
az security alert show \
  --name {alert-id} \
  --location {location} \
  --query "{Description:description, Entities:entities, RemediationSteps:remediationSteps}"
```

**Investigation Steps**:
1. ✅ Review alert timeline and affected resources
2. ✅ Check entity risk score and historical activity
3. ✅ Identify related alerts (multi-stage attacks)
4. ✅ Examine network traffic and authentication logs
5. ✅ Execute recommended remediation actions

## Just-in-Time VM Access

### How JIT Works
```bash
# Enable JIT VM access
az security jit-policy create \
  --location eastus \
  --name jit-policy-vm1 \
  --resource-group production-rg \
  --virtual-machines "/subscriptions/{id}/resourceGroups/production-rg/providers/Microsoft.Compute/virtualMachines/vm1" \
  --ports '[{"number":22,"protocol":"TCP","allowedSourceAddresses":["*"],"maxRequestAccessDuration":"PT3H"}]'

# Request JIT access
az security jit-policy request \
  --name jit-policy-vm1 \
  --resource-group production-rg \
  --virtual-machines '[{"id":"/subscriptions/{id}/resourceGroups/production-rg/providers/Microsoft.Compute/virtualMachines/vm1","ports":[{"number":22,"duration":"PT2H","sourceAddressPrefix":"203.0.113.10"}]}]'
```

### Benefits
- 🔒 **Reduced attack surface**: Ports closed by default (RDP 3389, SSH 22)
- ⏰ **Time-limited access**: Automatic port closure after duration expires
- 📝 **Audit trail**: All access requests logged
- ✅ **Automatic NSG management**: Dynamic rule creation/deletion

### Access Request Flow
1. User requests JIT access (duration, source IP)
2. Azure validates user permissions
3. NSG rule created temporarily (auto-expires)
4. User connects to VM
5. NSG rule automatically removed after duration

## Regulatory Compliance

### Compliance Dashboard
```bash
# Get compliance assessment
az security assessment list \
  --query "[?metadata.policyDefinitionId contains 'regulatory']" \
  --output table

# Get PCI DSS compliance score
az security regulatory-compliance-standard show \
  --name PCI-DSS-v3.2.1 \
  --query "{Score:properties.complianceScore, PassedControls:properties.passedAssessments, FailedControls:properties.failedAssessments}"
```

### Supported Standards
| Standard | Industry | Key Controls |
|----------|----------|--------------|
| **PCI DSS 3.2.1** | Payment processing | Encryption, access control, monitoring |
| **ISO 27001:2013** | Information security | Risk management, asset protection |
| **HIPAA/HITRUST** | Healthcare | Patient data protection, audit logging |
| **SOC 2** | Service providers | Security, availability, confidentiality |
| **NIST SP 800-53 Rev. 5** | Federal systems | Comprehensive security controls |

### Compliance Report Export
```bash
# Download compliance report
az security regulatory-compliance-standard show \
  --name PCI-DSS-v3.2.1 \
  --query "properties" > pci-compliance-report.json

# PDF export available through Azure Portal only
```

## Critical Notes
- 💡 **Foundational CSPM is free**: Enable on all subscriptions immediately
- 🎯 **Prioritize by Secure Score impact**: Remediate high-score-increase recommendations first
- ⚠️ **Quick Fix for bulk remediation**: Select multiple resources for simultaneous remediation
- 🔐 **JIT reduces brute-force attacks**: Ports closed 99% of the time
- 📊 **Compliance dashboard auto-updates**: Real-time compliance tracking
- 🚀 **Integrate with pipelines**: Scan IaC templates before deployment (prevents violations)
- 🔔 **Configure alert automation**: Auto-response with Logic Apps, Azure Functions

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/3-explore-microsoft-defender-cloud)
