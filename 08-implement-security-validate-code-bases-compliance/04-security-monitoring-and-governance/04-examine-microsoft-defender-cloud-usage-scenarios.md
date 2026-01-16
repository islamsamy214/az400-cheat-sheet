# Examine Microsoft Defender for Cloud Usage Scenarios

## Key Scenarios
- **Incident response**: Detection → Investigation → Containment → Remediation → Post-incident review
- **Regulatory compliance**: Automated PCI DSS/ISO 27001/HIPAA monitoring
- **Secure Score campaigns**: Systematic security posture improvement (62 → 83/100)
- **Container security**: Image scanning, runtime protection, Kubernetes admission control
- **DevSecOps integration**: IaC scanning in CI/CD pipelines
- **Vulnerability management**: Continuous scanning across 200+ VMs

## Incident Response and Threat Detection

### Detection Phase - SQL Injection Example
```bash
# Alert: "Potential SQL injection attack detected on SQL Database"
# Severity: High
# Resource: contoso-prod-sqldb
# MITRE ATT&CK: T1190 (Initial Access)
# Details: SQL query with '; DROP TABLE' from IP 203.0.113.45
```

### Investigation Steps
1. **Navigate** to Security Alerts in Defender for Cloud
2. **Review** alert details (source IP, attack time, affected database)
3. **Check** threat intelligence (IP reputation, known attack patterns)
4. **Review** entity timeline (database activity leading to alert)
5. **Assess** impact (data exfiltration, modification, account compromise)

### Containment Actions
```bash
# Block malicious IP at network level
az network nsg rule create \
  --resource-group production-rg \
  --nsg-name sql-nsg \
  --name BlockMaliciousIP \
  --priority 100 \
  --direction Inbound \
  --access Deny \
  --source-address-prefixes 203.0.113.45 \
  --destination-port-ranges 1433 \
  --protocol Tcp

# Disable compromised SQL login
az sql server ad-admin delete \
  --resource-group production-rg \
  --server-name contoso-sql-server

# Enable Advanced Threat Protection
az sql db threat-policy update \
  --resource-group production-rg \
  --server contoso-sql-server \
  --database contoso-prod-sqldb \
  --state Enabled \
  --email-account-admins Enabled
```

### Post-Incident Review
- **Root cause**: Publicly exposed SQL Database without IP firewall
- **Detection time**: Attack detected within minutes
- **Response time**: Attack contained within 15 minutes
- **Improvements**:
  - ✅ Implement SQL Database firewall (restrict to known IPs)
  - ✅ Enable Azure Private Link (eliminate public exposure)
  - ✅ Implement WAF to filter SQL injection attempts
  - ✅ Configure workflow automation (auto-block malicious IPs)

## Regulatory Compliance Monitoring

### PCI DSS Compliance Scenario
**Initial compliance**: 68% → **Target**: 94%

```bash
# Enable PCI DSS compliance standard
az security regulatory-compliance-standard show \
  --name PCI-DSS-v3.2.1

# Review failed controls
az security regulatory-compliance-assessment list \
  --standard-name PCI-DSS-v3.2.1 \
  --filter "properties/state eq 'Failed'" \
  --query "[].{Control:name, Description:properties.description}"
```

### Failed Controls Remediation

#### Control 2.2.4: Remove Unnecessary Services
```json
// Azure Policy - Audit VMs with unnecessary services
{
  "policyRule": {
    "if": {
      "allOf": [
        {"field": "type", "equals": "Microsoft.Compute/virtualMachines"},
        {"field": "tags['PCI-DSS-Scope']", "equals": "In-Scope"}
      ]
    },
    "then": {
      "effect": "auditIfNotExists",
      "details": {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "HardeningExtension"
      }
    }
  }
}
```

#### Control 8.2.3: Enforce MFA
```powershell
# Require MFA for payment system administrators
New-AzureADMSConditionalAccessPolicy `
  -DisplayName "PCI DSS - Require MFA for Payment System Admins" `
  -State "Enabled" `
  -Conditions $conditions `
  -GrantControls $controls
```

#### Control 10.2.1: Enable Audit Logging
```bash
# Enable diagnostic logging for storage accounts
for storage_account in $(az storage account list --query "[?tags.'PCI-DSS-Scope'=='In-Scope'].name" -o tsv); do
  az monitor diagnostic-settings create \
    --name PCI-DSS-Audit-Logging \
    --resource "/subscriptions/{id}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/${storage_account}" \
    --workspace /subscriptions/{id}/resourceGroups/security-rg/providers/Microsoft.OperationalInsights/workspaces/pci-audit-logs \
    --logs '[{"category":"StorageRead","enabled":true},{"category":"StorageWrite","enabled":true},{"category":"StorageDelete","enabled":true}]'
done
```

### Compliance Achievement
- **Initial score**: 68/100
- **Final score**: 94/100
- **Remediated controls**: 156/203
- **High-severity findings**: Reduced from 24 to 3
- **Critical gaps**: Eliminated completely

## Secure Score Improvement Campaign

### 90-Day Campaign: 62 → 80 (Target) → 83 (Achieved)

#### Phase 1: Prioritization (Weeks 1-2)
```bash
# Identify high-impact, low-effort recommendations
az security assessment list \
  --query "sort_by([?status.code=='Unhealthy'], &properties.weight) | reverse(@)[0:10]" \
  --output table
```

| Recommendation | Score Impact | Effort | Priority |
|----------------|--------------|--------|----------|
| Enable MFA for privileged accounts | +8 points | Low | 🔥 High |
| Enable storage encryption | +6 points | Low | 🔥 High |
| Apply system updates to VMs | +5 points | Medium | 🔥 High |
| Enable Azure Backup for VMs | +4 points | Low | Medium |

#### Phase 2: Bulk Remediation (Weeks 3-8)

**Enable MFA via Azure Policy**:
```json
{
  "displayName": "Require MFA for all administrators",
  "policyRule": {
    "if": {
      "allOf": [
        {"field": "type", "equals": "Microsoft.Authorization/roleAssignments"},
        {"field": "Microsoft.Authorization/roleAssignments/roleDefinitionId", "in": [
          "/subscriptions/{id}/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
          "/subscriptions/{id}/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
        ]}
      ]
    },
    "then": {"effect": "audit"}
  }
}
```

**Enable Storage Encryption (Quick Fix)**:
1. Click recommendation: "Storage accounts should use customer-managed key"
2. Select all 45 affected storage accounts
3. Click **Quick fix** → Configure Key Vault + encryption key
4. Apply remediation automatically

**Automate VM Patching**:
```yaml
# Azure Pipelines - Weekly automated patching
trigger:
  schedules:
    - cron: "0 2 * * 0"  # Every Sunday at 2 AM

steps:
  - task: AzureCLI@2
    inputs:
      scriptType: bash
      inlineScript: |
        vm_list=$(az vm list --query "[?tags.'Auto-Patch'=='Enabled']" -o json)
        echo "$vm_list" | jq -c '.[]' | while read vm; do
          vm_name=$(echo $vm | jq -r '.name')
          rg=$(echo $vm | jq -r '.resourceGroup')
          az vm run-command invoke \
            --resource-group $rg \
            --name $vm_name \
            --command-id RunShellScript \
            --scripts "sudo apt-get update && sudo apt-get upgrade -y"
        done
```

#### Phase 3: Validation (Weeks 9-12)
- ✅ Monitor Secure Score weekly
- ✅ Review newly created resources for compliance
- ✅ Conduct security review of all remediations
- ✅ Document exceptions (accepted risks with business justification)

**Campaign Results**:
- **Initial**: 62/100
- **Target**: 80/100
- **Achieved**: 83/100 (exceeding target)
- **Recommendations remediated**: 156/203
- **High-severity**: 24 → 3
- **Critical**: Eliminated

## Container Security

### Enable Defender for Containers
```bash
az security pricing create \
  --name Containers \
  --tier Standard

# Configure features
# - Image scanning: ACR vulnerability scanning
# - Runtime threat detection: Suspicious container activities
# - Kubernetes admission control: Policy enforcement at deployment
```

### Container Image Scanning
**Vulnerability Findings**:
- Image: `contosoapp:v1.2.3`
- Vulnerabilities: 12 (3 High, 6 Medium, 3 Low)
- **High-severity**:
  - CVE-2024-12345: OpenSSL buffer overflow (CVSS 8.9)
  - CVE-2024-67890: Node.js remote code execution (CVSS 9.1)
  - CVE-2024-11111: Apache Tomcat path traversal (CVSS 7.5)

**Remediation (Dockerfile)**:
```dockerfile
# Update base image to patched version
FROM node:18-alpine3.18  # Updated from node:16-alpine3.15

WORKDIR /app

# Update system packages
RUN apk update && apk upgrade

COPY package*.json ./
RUN npm ci --only=production

COPY . .

USER node
EXPOSE 3000
CMD ["node", "server.js"]
```

### Kubernetes Runtime Protection
**Alert**: "Cryptocurrency mining activity detected"
- **Severity**: High
- **Resource**: Pod `payment-service-7d8f9c-abcd` in namespace `production`
- **Details**: Process `xmrig` (crypto miner) with elevated CPU usage

**Investigation & Response**:
```bash
# Investigate suspicious pod
kubectl describe pod payment-service-7d8f9c-abcd -n production

# Check container processes
kubectl exec payment-service-7d8f9c-abcd -n production -- ps aux

# Review logs
kubectl logs payment-service-7d8f9c-abcd -n production

# Terminate compromised pod
kubectl delete pod payment-service-7d8f9c-abcd -n production

# Scan container image
az acr repository show-tags \
  --name contosoregistry \
  --repository payment-service \
  --detail
```

### Kubernetes Admission Control
```yaml
# Deny deployment of images with high-severity vulnerabilities
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAzureContainerAllowedImages
metadata:
  name: deny-vulnerable-images
spec:
  enforcementAction: deny
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    namespaces:
      - production
  parameters:
    allowedRegistries:
      - contosoregistry.azurecr.io
    maxSeverity: "Medium"  # Block High/Critical vulnerabilities
```

## DevSecOps Pipeline Integration

### IaC Scanning in Azure Pipelines
```yaml
trigger:
  - main

steps:
  - task: AzureCLI@2
    displayName: "Install Microsoft Security DevOps"
    inputs:
      scriptType: bash
      inlineScript: |
        dotnet tool install -g microsoft.security.devops.cli

  - task: MicrosoftSecurityDevOps@1
    displayName: "Run Security DevOps Analysis"
    inputs:
      categories: "IaC,secrets,dependencies"

  - task: PublishSecurityAnalysisLogs@3
    inputs:
      ArtifactName: "CodeAnalysisLogs"

  - task: PostAnalysis@2
    inputs:
      GdnBreakGdnToolMicrosoftSecurityDevOps: true  # Fail build on issues

  - task: AzureResourceManagerTemplateDeployment@3
    condition: succeeded()  # Only deploy if security scan passes
    inputs:
      templateLocation: "Linked artifact"
      csmFile: "$(Build.SourcesDirectory)/infrastructure/main.bicep"
```

**Security Findings**:
- ❌ **Finding 1**: Storage account without HTTPS enforcement (High)
- ❌ **Finding 2**: SQL firewall allows all Azure IPs (Medium)
- ❌ **Finding 3**: Key Vault soft delete not enabled (Medium)

**Result**: Build fails, preventing insecure infrastructure deployment

**Remediation (Bicep)**:
```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: { name: 'Standard_GRS' }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true  // ✅ Fixed: Enforce HTTPS
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
    networkAcls: {
      defaultAction: 'Deny'
      bypass: 'AzureServices'
    }
  }
}
```

## Vulnerability Management

### Continuous Scanning (200 VMs)
```bash
# Defender for Cloud automatically deploys vulnerability assessment agent
# Results:
# - Critical: 5 VMs (remotely exploitable)
# - High: 23 VMs (privilege escalation)
# - Medium: 87 VMs (information disclosure)
# - Low: 143 VMs (minor weaknesses)
```

### Critical Vulnerability Example
- **CVE-2024-98765**: Windows RDS Remote Code Execution (CVSS 9.8)
- **Affected**: 5 Windows Server 2019 VMs
- **Remediation**: Install KB5044553

**Automated Patching (Azure Automation)**:
```powershell
# Automated vulnerability patching workflow
workflow AutomatedVulnerabilityPatching {
  param([string]$ResourceGroupName, [string]$VMName, [string]$UpdateKB)

  $connection = Get-AutomationConnection -Name AzureRunAsConnection
  Connect-AzAccount -ServicePrincipal `
    -Tenant $connection.TenantID `
    -ApplicationId $connection.ApplicationID `
    -CertificateThumbprint $connection.CertificateThumbprint

  Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroupName `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString "Install-WindowsUpdate -KBArticleID $UpdateKB -AcceptAll -AutoReboot"

  Write-Output "Successfully installed update $UpdateKB on VM $VMName"
}
```

## Critical Notes
- 🚨 **Incident response speed**: Detection within minutes, containment within 15 minutes critical
- 📊 **Compliance automation**: 68% → 94% achievable in weeks with systematic approach
- 🎯 **Secure Score campaigns**: Prioritize high-impact, low-effort wins first
- 🐳 **Container security**: Scan images before deployment + runtime monitoring essential
- 🔄 **DevSecOps integration**: Fail builds on security findings (shift-left)
- 🔧 **Automated remediation**: Quick Fix for bulk operations, Azure Automation for patching
- 💡 **Document learnings**: Post-incident reviews prevent future occurrences

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/4-examine-microsoft-defender-cloud-usage-scenarios)
