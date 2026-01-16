# Implement Pipeline Security

## Key Concepts
- **Secrets management**: Centralized storage in Azure Key Vault, no hardcoded credentials
- **Service connections**: Managed identities for passwordless Azure authentication
- **Pipeline permissions**: RBAC roles (Reader, Contributor, Administrator)
- **Agent security**: Isolated Microsoft-hosted vs hardened self-hosted agents
- **Deployment gates**: Approvals, business hours restrictions, compliance checks
- **Audit logging**: Azure Monitor integration for pipeline activity tracking
- **DAST integration**: OWASP ZAP for runtime vulnerability scanning

## Secrets Management

### Azure Key Vault Integration
```yaml
# Azure Pipelines - Retrieve secrets from Key Vault
steps:
  - task: AzureKeyVault@2
    inputs:
      azureSubscription: 'Production-Service-Connection'
      KeyVaultName: 'contoso-prod-kv'
      SecretsFilter: 'DatabasePassword,ApiKey,StorageConnectionString'
      RunAsPreJob: true

  - script: |
      echo "Connecting with secured credentials..."
      # Secrets available as variables: $(DatabasePassword), $(ApiKey)
    displayName: 'Use Key Vault secrets'
```

### GitHub Secrets (Repository/Environment/Organization Scope)
```yaml
# GitHub Actions - Use encrypted secrets
steps:
  - name: Deploy to Azure
    env:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
      DATABASE_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}
    run: |
      az login --service-principal-u $AZURE_CREDENTIALS
      az webapp config appsettings set --settings DB_PWD=$DATABASE_PASSWORD
```

### Pipeline Secure Variables
```yaml
# Azure Pipelines - Runtime secure variables
variables:
  - group: production-secrets  # Variable group from Azure DevOps Library
  - name: ApiKey
    value: ''  # Set as secret variable in pipeline UI
```

## Service Connection Security

### Service Principal Configuration (Least Privilege)
```bash
# Create service principal with minimal permissions
az ad sp create-for-rbac \
  --name "AzDevOps-Production-Deploy" \
  --role "Contributor" \
  --scopes /subscriptions/{subscription-id}/resourceGroups/production-rg

# Assign specific permissions only
az role assignment create \
  --assignee {service-principal-id} \
  --role "Website Contributor" \
  --scope /subscriptions/{subscription-id}/resourceGroups/web-app-rg
```

### Separate Connections Per Environment
- **Dev connection**: Contributor on dev resource groups
- **Staging connection**: Contributor on staging resource groups
- **Production connection**: Reader + specific deployment permissions only

## Pipeline Permissions (RBAC)

| Role | Permissions | Use Case |
|------|-------------|----------|
| **Reader** | View pipelines, runs, logs | Auditors, stakeholders |
| **Contributor** | Create/edit pipelines, queue builds | Developers |
| **Administrator** | All permissions + security settings | DevOps engineers, security team |

### Branch Protection Policies
```yaml
# Azure Repos - Branch policy enforcement
policies:
  - minimumApproverCount: 2
  - requireReviewers: true
  - buildValidationRequired: true
  - commentResolutionRequired: true
  - enforceLinkedWorkItems: true
```

## Agent Security

### Microsoft-Hosted Agents (Secure by Default)
- ✅ Clean environment per build
- ✅ Automatic patching
- ✅ Network isolation
- ✅ No persistent state

### Self-Hosted Agents (Requires Hardening)
```bash
# Harden self-hosted agent
# 1. Network restrictions
sudo ufw enable
sudo ufw allow 443/tcp  # HTTPS only

# 2. Regular patching
sudo apt update && sudo apt upgrade -y

# 3. Managed identity (Azure VMs)
az vm identity assign --name build-agent-vm --resource-group agents-rg

# 4. Minimal software installation (only required tools)
```

## Secure Pipeline Design Patterns

### Immutable Infrastructure (ARM/Bicep Templates)
```yaml
# Deploy infrastructure from code (no manual changes)
steps:
  - task: AzureResourceManagerTemplateDeployment@3
    inputs:
      deploymentScope: 'Resource Group'
      resourceGroupName: '$(ResourceGroupName)'
      location: '$(Location)'
      templateLocation: 'Linked artifact'
      csmFile: '$(Build.SourcesDirectory)/infrastructure/main.bicep'
      deploymentMode: 'Incremental'
```

### Security Scanning Integration
```yaml
# Multi-stage security scanning
stages:
  - stage: SecurityScan
    jobs:
      - job: SAST
        steps:
          - task: SonarQubePrepare@5
          - task: DotNetCoreCLI@2
            inputs:
              command: 'build'
          - task: SonarQubeAnalyze@5
      
      - job: SCA
        steps:
          - task: WhiteSource@21
            inputs:
              cwd: '$(Build.SourcesDirectory)'
              projectName: 'ContosoApp'
      
      - job: ContainerScan
        steps:
          - task: AzureContainerRegistry@0
            inputs:
              command: 'scan'
              repository: '$(ImageRepository)'
```

### Deployment Gates with Approvals
```yaml
# Azure Pipelines - Manual approval + compliance check
stages:
  - stage: Production
    jobs:
      - deployment: DeployToProduction
        environment: 'Production'  # Requires manual approval in UI
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Deploying to production..."
```

### Business Hours Gate
```yaml
# Azure DevOps - Restrict deployments to business hours
# Configure in Environment > Approvals and checks > Business hours
businessHours:
  - days: Monday-Friday
  - hours: 09:00-17:00
  - timezone: 'Eastern Standard Time'
```

## Audit Logging and Monitoring

### Azure DevOps Audit Logs
```bash
# Query audit logs via Azure DevOps REST API
az devops invoke \
  --area audit \
  --resource log \
  --route-parameters org=contoso \
  --query "decoratedAuditLogEntries[?category=='Pipeline']"
```

### Azure Monitor Integration
```bash
# Send pipeline logs to Log Analytics
az monitor diagnostic-settings create \
  --name pipeline-audit-logs \
  --resource /subscriptions/{id}/resourceGroups/{rg}/providers/Microsoft.DevOps/organizations/{org} \
  --workspace /subscriptions/{id}/resourceGroups/monitoring-rg/providers/Microsoft.OperationalInsights/workspaces/devops-logs \
  --logs '[{"category":"Pipelines","enabled":true,"retentionPolicy":{"days":90,"enabled":true}}]'
```

### Monitoring Alerts
```kql
// KQL query - Detect failed production deployments
AzureDevOpsAuditLogs
| where TimeGenerated > ago(1h)
| where Category == "Release" and Result == "Failed" and Environment == "Production"
| project TimeGenerated, User, PipelineName, Result, Details
```

## Authentication and Authorization

### MFA Enforcement (Conditional Access)
```powershell
# PowerShell - Require MFA for DevOps admins
$conditions = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessConditionSet
$conditions.Applications = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessApplicationConditionSet
$conditions.Applications.IncludeApplications = "499b84ac-1321-427f-aa17-267ca6975798"  # Azure DevOps

$controls = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessGrantControls
$controls.BuiltInControls = "mfa"

New-AzureADMSConditionalAccessPolicy `
  -DisplayName "Require MFA for Azure DevOps" `
  -State "Enabled" `
  -Conditions $conditions `
  -GrantControls $controls
```

### JIT Administration (Privileged Identity Management)
- ⏰ Time-limited admin access (1-8 hours)
- ✅ Approval workflow for elevation
- 📝 Justification required
- 🔔 Audit trail of all privilege escalations

## DAST with OWASP ZAP

```yaml
# Azure Pipelines - OWASP ZAP dynamic scanning
steps:
  - task: Docker@2
    inputs:
      command: 'run'
      arguments: |
        --rm -v $(Build.SourcesDirectory)/report:/zap/wrk/:rw \
        owasp/zap2docker-stable:latest \
        zap-baseline.py \
        -t https://$(WebAppUrl) \
        -r zap-report.html \
        -J zap-report.json
    displayName: 'OWASP ZAP Baseline Scan'
  
  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: '$(Build.SourcesDirectory)/report'
      ArtifactName: 'zap-reports'
```

## Production Monitoring

### Defender for Cloud Integration
```yaml
# Query Defender recommendations for deployed resources
az security assessment list \
  --resource-group production-rg \
  --query "[?status.code=='Unhealthy'].{Name:displayName, Severity:severity}"
```

### Application Insights
```yaml
# Inject App Insights instrumentation key from Key Vault
- task: AzureWebApp@1
  inputs:
    appSettings: '-APPINSIGHTS_INSTRUMENTATIONKEY $(AppInsightsKey)'
```

## Critical Notes
- 🔐 **Never commit secrets**: Always use Key Vault or pipeline variables
- 🎯 **Principle of least privilege**: Grant minimum required permissions
- ✅ **Security gates mandatory**: Block deployments failing security scans
- 📊 **Monitor everything**: Audit logs + application telemetry + security alerts
- 🔄 **Rotate credentials**: Regular rotation of service principal secrets
- ⚠️ **Self-hosted agent risks**: Requires additional hardening (patching, network isolation, minimal software)
- 💡 **Managed identities preferred**: Eliminate password-based authentication where possible

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/2-implement-pipeline-security)
