# Summary

## Module Recap
**Comprehensive DevSecOps security** spanning:
- Pipeline security (Azure DevOps & GitHub Actions)
- Cloud security posture management (Microsoft Defender for Cloud)
- Governance automation (Azure Policy & Initiatives)
- Resource protection (Resource Locks)
- Identity threat detection (Defender for Identity)
- Code security (GitHub Advanced Security integration)

**Duration**: 2 hours | **Units**: 13 (11 content, 1 knowledge check, 1 summary)

## Key Takeaways

### Pipeline Security
**Multilayered protection**:
- 🔐 **Secrets management**: Azure Key Vault integration (no hardcoded credentials)
- 🔑 **Service connections**: Managed identities (passwordless auth)
- 👥 **RBAC**: Pipeline permissions (Reader, Contributor, Administrator)
- 🖥️ **Agent security**: Microsoft-hosted (secure by default) vs self-hosted (requires hardening)
- ✅ **Deployment gates**: Approvals, business hours restrictions, security scans
- 📊 **Audit logging**: Azure Monitor integration, Log Analytics
- 🔒 **MFA + Conditional Access + PIM**: Secure administrator access
- 🔍 **DAST integration**: OWASP ZAP for runtime vulnerability detection

**Critical commands**:
```yaml
# Azure Key Vault secret retrieval
- task: AzureKeyVault@2
  inputs:
    KeyVaultName: 'contoso-prod-kv'
    SecretsFilter: 'DatabasePassword,ApiKey'

# OWASP ZAP scanning
- task: Docker@2
  inputs:
    command: 'run'
    arguments: 'owasp/zap2docker-stable zap-baseline.py -t $(WebAppUrl)'
```

### Microsoft Defender for Cloud
**Unified security management**:
- ☁️ **CSPM** (Cloud Security Posture Management): Free config assessments, Secure Score
- 🛡️ **CWP** (Cloud Workload Protection): Runtime threat detection, vulnerability scanning
- 📦 **Defender plans**: Servers, Containers, App Service, Storage, SQL, Key Vault
- 📊 **Secure Score**: 0-100 quantified security posture (Formula: Healthy Resources / Total × Points)
- 🔧 **Remediation**: Manual fixes, Quick Fix (bulk), Azure Policy enforcement
- 🚨 **Security alerts**: MITRE ATT&CK mapped, investigation workflows, automated response
- 🔐 **JIT VM access**: Dynamic port opening (RDP/SSH), reduced attack surface
- 📋 **Regulatory compliance**: PCI DSS, ISO 27001, HIPAA, SOC 2, NIST dashboards

**Critical commands**:
```bash
# Enable Defender plans
az security pricing create --name VirtualMachines --tier Standard
az security pricing create --name Containers --tier Standard

# Get Secure Score
az security secure-score show --name ascScore --query "score.current"

# Enable JIT VM access
az security jit-policy create --location eastus --name jit-policy-vm1 \
  --ports '[{"number":22,"protocol":"TCP","maxRequestAccessDuration":"PT3H"}]'
```

### Azure Policy Governance
**Policy-as-code automation**:
- 📝 **Policy effects**: Deny (block), Audit (log), DeployIfNotExists (auto-deploy), Modify (auto-fix)
- 📐 **Evaluation**: Deployment-time + periodic scans (~24h) + on-demand
- 🎯 **Scope hierarchy**: Management Group → Subscription → Resource Group → Resource
- 🔄 **Built-in policies**: Region restrictions, required tagging, VM SKU limits, encryption
- 📊 **Compliance dashboard**: States (Compliant, Non-compliant, Conflict, Not Started, Exempt)
- 🔧 **Remediation tasks**: Bring existing non-compliant resources into compliance
- 🚀 **CI/CD integration**: Validate IaC templates before deployment (shift-left)

**Critical commands**:
```bash
# Assign policy
az policy assignment create \
  --name 'RequireEncryption' \
  --scope '/subscriptions/{id}' \
  --policy '404c3081-a854-4457-ae30-26a93ef643f9' \
  --params '{"effect": {"value": "Deny"}}'

# Check compliance
az policy state list --filter "complianceState eq 'NonCompliant'"

# Create remediation task
az policy remediation create \
  --name remediate-diagnostics \
  --policy-assignment require-diagnostic-settings
```

### Policy Initiatives
**Bundled compliance frameworks**:
- 📦 **Simplified management**: Assign 100+ policies with single initiative
- 🏛️ **Built-in initiatives**: Azure Security Benchmark, PCI DSS, ISO 27001, HIPAA, NIST
- 📊 **Aggregate compliance**: Framework-level view (82% compliance) vs individual policies
- 🎨 **Custom initiatives**: Organization-specific policy bundles
- 🌐 **Management group assignment**: Cascade to all subscriptions

**Critical commands**:
```bash
# Assign Azure Security Benchmark
az policy assignment create \
  --name azure-security-benchmark \
  --scope /subscriptions/{id} \
  --policy-set-definition '1f3afdf9-d0c9-4c3d-847f-89da613e70a8'

# Create custom initiative
az policy set-definition create \
  --name contoso-security-framework \
  --definitions '[{...}]'
```

### Resource Locks
**Protection against accidental changes**:
- 🔒 **CanNotDelete**: Allow read/update, block delete (production databases, VMs, networks)
- 📖 **ReadOnly**: Allow read only, block update/delete (audit hold, change freeze)
- ⬇️ **Inheritance**: Locks cascade from parent to child (most restrictive wins)
- 👑 **Override permissions**: Even Owners cannot delete locked resources
- 🔐 **Lock removal**: Requires `Microsoft.Authorization/locks/delete` permission

**Critical commands**:
```bash
# Apply CanNotDelete lock
az lock create \
  --name ProductionRGLock \
  --resource-group production-rg \
  --lock-type CanNotDelete

# Apply ReadOnly lock
az lock create \
  --name AuditStorageLock \
  --resource-type Microsoft.Storage/storageAccounts \
  --resource-name auditlogstorage \
  --lock-type ReadOnly

# Remove lock
az lock delete --name ProductionRGLock --resource-group production-rg
```

### Microsoft Defender for Identity
**Identity threat detection**:
- 🎯 **Lightweight sensors**: Installed on domain controllers (no port mirroring)
- 🔍 **Threat detection**: Pass-the-Hash, Golden Ticket, DCSync, lateral movement
- 📊 **MITRE ATT&CK**: Framework-aligned tactics/techniques
- 🔄 **Microsoft Defender XDR**: Unified incidents across identity, endpoint, email, cloud
- 🔎 **Advanced hunting**: KQL queries for proactive threat hunting

**Threat categories**:
| Category | Examples | MITRE ATT&CK |
|----------|----------|--------------|
| **Credential theft** | Pass-the-Hash, Golden Ticket | T1550, T1558 |
| **Reconnaissance** | Account enumeration, DNS recon | T1087, T1018 |
| **Lateral movement** | PsExec, WMI, PowerShell remoting | T1021 |
| **Domain dominance** | DCSync, DCShadow | T1003.006 |

**Critical setup**:
```powershell
# Enable Advanced Audit Policies
auditpol /set /category:"Account Logon" /success:enable /failure:enable
auditpol /set /category:"DS Access" /success:enable /failure:enable

# KQL query example
IdentityLogonEvents
| where ActionType == "PassTheHashDetection"
| project Timestamp, AccountName, DeviceName, IPAddress
```

### GitHub Advanced Security Integration
**Code-to-cloud security** (from summary content):
- 🔍 **CodeQL scanning**: Semantic analysis detecting SQL injection, XSS, deserialization
- 🔑 **Secret scanning**: 200+ credential patterns, push protection blocks commits
- 📦 **Dependabot**: Vulnerability scanning, automatic update PRs
- 🔄 **Defender for DevOps**: Synchronizes GitHub findings to Defender for Cloud
- 🚀 **CI/CD integration**: Security gates in pull requests

## Integration and Automation

**Synergistic capabilities**:
```
Azure Policy
    ↓ (auto-deploys)
Defender for Cloud Agents + Diagnostic Settings
    ↓ (generates)
Security Recommendations
    ↓ (triggers)
Remediation Tasks
```

**Defense in depth**:
- 🔐 Resource Locks protect policy assignments
- 📊 Defender for Cloud recommendations trigger Azure Policy remediation
- 🆔 Defender for Identity alerts correlate with Defender for Cloud threats in XDR
- 💻 GitHub Advanced Security findings sync to Defender for Cloud
- 🔄 IaC templates define policies, locks, Defender configs

**Automation example (IaC)**:
```bicep
// Define resource with lock, policy, and Defender config
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'prodstorageacct'
  properties: {
    supportsHttpsTrafficOnly: true  // Policy compliant
    encryption: { keySource: 'Microsoft.Storage' }
  }
}

resource lock 'Microsoft.Authorization/locks@2020-05-01' = {
  name: 'storage-lock'
  scope: storageAccount
  properties: { level: 'CanNotDelete' }
}
```

## Next Steps

**Implementation roadmap**:
1. ✅ **Security assessment**: Identify gaps in pipeline, cloud, policy, identity, code security
2. ✅ **Prioritize critical assets**: Apply locks, Defender plans, enhanced monitoring
3. ✅ **Implement policy-as-code**: Define security/compliance as Azure Policy via CI/CD
4. ✅ **Enable Defender for Cloud**: Foundational CSPM (all subscriptions) + Defender plans (critical workloads)
5. ✅ **Deploy Defender for Identity**: Install sensors on all domain controllers
6. ✅ **Integrate GitHub Advanced Security**: Onboard GitHub orgs to Defender for DevOps
7. ✅ **Establish workflows**: Alert triage, incident response, vulnerability remediation
8. ✅ **Train teams**: Security tools, secure coding, DevSecOps practices
9. ✅ **Measure & improve**: Track Secure Score, policy compliance, remediation time

**Success metrics**:
| Metric | Target | Measurement |
|--------|--------|-------------|
| **Secure Score** | 80+ | Defender for Cloud dashboard |
| **Policy compliance** | 90%+ | Azure Policy compliance dashboard |
| **Vulnerability remediation** | <7 days | Defender vulnerability management |
| **High-severity alerts** | <10 | Defender XDR security alerts |
| **Lock coverage** | 100% critical resources | Azure Resource Graph query |

## Additional Resources

**Documentation**:
- [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/)
- [Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/)
- [Microsoft Defender for Identity](https://learn.microsoft.com/en-us/defender-for-identity/)
- [GitHub Advanced Security](https://docs.github.com/en/code-security)
- [Microsoft Security DevOps](https://github.com/microsoft/security-devops-action)
- [Azure Well-Architected Framework - Security](https://learn.microsoft.com/en-us/azure/well-architected/security/)
- [DevSecOps on Azure](https://learn.microsoft.com/en-us/azure/architecture/solution-ideas/articles/devsecops-in-azure)

## Critical Exam Focus Areas

**For AZ-400 certification**:
- 🎯 **Pipeline security**: Key Vault integration, managed identities, RBAC, deployment gates
- 📊 **Defender for Cloud**: CSPM vs CWP, Secure Score calculation, JIT VM access
- 📝 **Azure Policy**: Effects (Deny, DINE, Modify), evaluation timing, remediation tasks
- 🔒 **Resource locks**: CanNotDelete vs ReadOnly, inheritance, lock removal permissions
- 🆔 **Defender for Identity**: Sensor deployment, threat categories, MITRE ATT&CK mapping
- 🔄 **Integration patterns**: Policy → Defender → XDR correlation, IaC scanning in CI/CD

**Hands-on scenarios to practice**:
1. ✅ Create Azure Pipeline with Key Vault integration and security scanning
2. ✅ Configure Defender for Cloud Secure Score improvement campaign
3. ✅ Implement custom Azure Policy with DINE effect and remediation task
4. ✅ Design policy initiative mapping to compliance framework (PCI DSS/ISO 27001)
5. ✅ Apply resource locks with IaC (Bicep/Terraform)
6. ✅ Investigate Defender for Identity alert with KQL queries
7. ✅ Configure GitHub Advanced Security with branch protection

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/)
