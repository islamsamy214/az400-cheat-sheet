# Summary

## Module Overview

Congratulations on completing the **Create Azure Resources Using Azure CLI** module! You've gained comprehensive knowledge of Azure CLI fundamentals, installation methods, command structure, interactive and scripted execution, troubleshooting techniques, and production best practices.

This module equipped you with practical skills to automate Azure resource management using a cross-platform, scriptable command-line interface that integrates seamlessly with modern DevOps workflows.

## Key Takeaways

### 1. **Automation at Scale**

Azure CLI transforms manual, error-prone portal operations into repeatable, version-controlled scripts:

**Manual Portal Approach** (Time-Consuming):
- Click through Azure Portal UI
- Prone to human error
- Difficult to replicate exactly
- No audit trail of changes
- Cannot integrate with CI/CD

**Azure CLI Approach** (Efficient):
```bash
#!/bin/bash
# Create 10 storage accounts in 30 seconds
for i in {1..10}; do
    az storage account create \
        --name "stapp$(date +%s)${RANDOM}" \
        --resource-group rg-prod \
        --location eastus \
        --sku Standard_LRS \
        --no-wait
done
```

**Benefits**:
- **Speed**: Deploy 100+ resources in minutes
- **Consistency**: Same script = identical results
- **Scalability**: Loop through configurations easily
- **Reproducibility**: Share scripts with team

### 2. **Version Control Integration**

Store Azure CLI scripts in Git alongside application code:

**Repository Structure**:
```
infrastructure/
├── environments/
│   ├── dev.sh
│   ├── staging.sh
│   └── prod.sh
├── modules/
│   ├── storage.sh
│   ├── networking.sh
│   └── compute.sh
└── README.md
```

**Benefits**:
- **Change Tracking**: Git history shows who changed what and when
- **Code Reviews**: Pull requests for infrastructure changes
- **Rollback**: `git revert` to previous infrastructure state
- **Branching**: Test infrastructure changes in feature branches
- **Collaboration**: Team members contribute and review

**Example Git Workflow**:
```bash
# Create feature branch for infrastructure change
git checkout -b feature/add-storage-redundancy

# Update deployment script
vim infrastructure/environments/prod.sh
# Change: --sku Standard_LRS → --sku Standard_GRS

# Commit and push
git add infrastructure/environments/prod.sh
git commit -m "Add geo-redundant storage for production"
git push origin feature/add-storage-redundancy

# Create pull request for review
# After approval, merge to main
# CI/CD pipeline deploys updated infrastructure
```

### 3. **CI/CD Pipeline Integration**

Azure CLI scripts integrate seamlessly with DevOps pipelines:

**GitHub Actions Example**:
```yaml
name: Deploy Infrastructure

on:
  push:
    branches: [main]
    paths:
      - 'infrastructure/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Deploy to Azure
        run: |
          az account set --subscription ${{ secrets.SUBSCRIPTION_ID }}
          ./infrastructure/deploy.sh
        env:
          RESOURCE_GROUP: ${{ secrets.RESOURCE_GROUP }}
          LOCATION: eastus
```

**Azure Pipelines Example**:
```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - infrastructure/*

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: AzureCLI@2
  displayName: 'Deploy Infrastructure'
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    scriptType: 'bash'
    scriptLocation: 'scriptPath'
    scriptPath: './infrastructure/deploy.sh'
    arguments: '$(Environment) $(ResourceGroup)'
```

**Benefits**:
- **Automated Deployments**: Push to main → infrastructure updates automatically
- **Environment Promotion**: Dev → Staging → Prod pipelines
- **Gated Approvals**: Require manual approval for production deployments
- **Rollback Capability**: Redeploy previous pipeline run
- **Audit Trail**: Pipeline logs show all infrastructure changes

### 4. **Cross-Platform Compatibility**

Azure CLI works consistently across all major platforms:

| Platform | Shell Options | Installation Method | Use Case |
|----------|---------------|---------------------|----------|
| **Windows** | PowerShell, cmd.exe, WSL | MSI installer, winget | Developer workstations |
| **Linux** | Bash, Zsh | apt, yum, script | Servers, CI/CD agents |
| **macOS** | Bash, Zsh | Homebrew, script | Developer workstations |
| **Docker** | Bash | `mcr.microsoft.com/azure-cli` | Isolated environments, CI/CD |
| **Azure Cloud Shell** | Bash, PowerShell | Pre-installed | Browser-based, no installation |

**Platform-Specific Script Considerations**:

**Bash (Linux, macOS, WSL)**:
```bash
#!/bin/bash
# Line continuation with backslash
az storage account create \
    --name mystorageacct \
    --resource-group myrg

# Variables with $ prefix
resourceGroup="myrg"
az group show --name $resourceGroup

# Boolean literals with escaped backticks
az storage account list --query "[?enableHttpsTrafficOnly==\`true\`]"
```

**PowerShell (Windows)**:
```powershell
# Line continuation with backtick
az storage account create `
    --name mystorageacct `
    --resource-group myrg

# Variables with $ prefix
$resourceGroup = "myrg"
az group show --name $resourceGroup

# Boolean literals with double backticks
az storage account list --query "[?enableHttpsTrafficOnly==``true``]"
```

**cmd.exe (Legacy Windows)**:
```batch
REM Line continuation with caret
az storage account create ^
    --name mystorageacct ^
    --resource-group myrg

REM Variables with % delimiters
set resourceGroup=myrg
az group show --name %resourceGroup%

REM Boolean literals without backticks
az storage account list --query "[?enableHttpsTrafficOnly=='true']"
```

### 5. **Infrastructure as Code (IaC) Best Practices**

Azure CLI scripts follow IaC principles for maintainable, reliable infrastructure:

**Idempotency** (Safe to Run Multiple Times):
```bash
# Check if resource exists before creating
if ! az group show --name rg-prod &> /dev/null; then
    az group create --name rg-prod --location eastus
fi
```

**Error Handling** (Fail Fast):
```bash
set -e          # Exit on error
set -u          # Exit on undefined variable
set -o pipefail # Fail if any command in pipe fails

if ! az account show &> /dev/null; then
    echo "Error: Not authenticated to Azure"
    exit 1
fi
```

**Parameterization** (Flexible Configuration):
```bash
environment=$1  # Accept environment as parameter
location=$2     # Accept location as parameter

az group create \
    --name "rg-${environment}" \
    --location "$location" \
    --tags Environment="$environment"
```

**Validation** (Verify Deployment):
```bash
# Verify storage account created
if az storage account show --name mystorageacct --resource-group myrg &> /dev/null; then
    echo "✓ Storage account deployed successfully"
else
    echo "✗ Storage account deployment failed"
    exit 1
fi
```

## Learning Objectives Recap

By completing this module, you achieved the following objectives:

### 1. **Describe Azure CLI Syntax Components**

You learned the hierarchical command structure:

```
az <reference-group> <subgroup> <command> --<parameter> <value>
```

**Example Breakdown**:
```bash
az storage account create --name mystorageacct --resource-group myrg
│  │       │       │       └─ Parameters (key-value pairs)
│  │       │       └───────── Command (action to perform)
│  │       └───────────────── Subgroup (nested organizational unit)
│  └───────────────────────── Reference group (top-level organization)
└──────────────────────────── Root command (invokes Azure CLI)
```

**Key Concepts**:
- **Reference groups**: Core references (built-in) vs extensions (installable)
- **Reference status**: GA (stable), Preview (testing), Experimental (early), Deprecated (outdated)
- **Global parameters**: Work with all commands (`--subscription`, `--resource-group`, `--output`)

### 2. **Use Azure CLI to Automate Azure Resource Management**

You practiced automation techniques:

**Interactive Execution** (Ad-Hoc Tasks):
```bash
# Authenticate
az login

# Create resource group
az group create --name rg-demo --location eastus

# Create storage account
az storage account create \
    --name stdemo$(date +%s) \
    --resource-group rg-demo \
    --location eastus \
    --sku Standard_LRS

# Query resources with JMESPath
az storage account list --query "[?location=='eastus'].name" -o table
```

**Scripted Automation** (Repeatable Deployments):
```bash
#!/bin/bash
# deploy-infrastructure.sh

for environment in dev staging prod; do
    echo "Deploying $environment environment..."
    
    az group create \
        --name "rg-app-$environment" \
        --location eastus \
        --tags Environment=$environment
    
    az storage account create \
        --name "stapp${environment}$(date +%s)" \
        --resource-group "rg-app-$environment" \
        --location eastus \
        --sku Standard_LRS \
        --tags Environment=$environment
done
```

### 3. **Run Azure CLI Commands Interactively and Through Scripts**

You mastered both execution modes:

**Interactive Mode** (Best for):
- Learning and experimentation
- Ad-hoc administrative tasks
- Quick troubleshooting
- Verifying resource state

**Scripted Mode** (Best for):
- Automated deployments
- Repeatable workflows
- CI/CD integration
- Complex multi-step processes

**Comparison**:

| Aspect | Interactive | Scripted |
|--------|-------------|----------|
| **Execution** | One command at a time | Sequential batch |
| **Feedback** | Immediate results | Logged to file/console |
| **Error Handling** | Manual retry | Automated with `set -e` |
| **Reproducibility** | Manual recreation | Automated re-run |
| **Documentation** | History command | Script itself |

### 4. **Get Help with Azure CLI Commands**

You discovered multiple help resources:

**Built-in Help**:
```bash
# Get help for any command
az storage account create --help

# Find commands by example
az find "create storage account"
az find "blob"

# Search within command group
az storage --help
```

**Help Output Sections**:
- **Command**: Brief description
- **Arguments**: Required and optional parameters
- **Global Arguments**: Parameters available for all commands
- **Examples**: Common usage patterns

**External Resources**:
- **Documentation**: [https://docs.microsoft.com/cli/azure](https://docs.microsoft.com/cli/azure)
- **A-Z Index**: Alphabetical command reference
- **JMESPath Tutorial**: Query syntax guide
- **GitHub Copilot**: AI-powered command suggestions
- **Stack Overflow**: Community troubleshooting (tag: `azure-cli`)
- **Microsoft Q&A**: Official support forum

### 5. **Troubleshoot Azure CLI Commands with `--debug`**

You learned advanced debugging techniques:

**Enable Debug Output**:
```bash
az storage account create --name mystorageacct --resource-group myrg --debug
```

**Debug Output Sections**:
```
Command arguments: ['storage', 'account', 'create', '--name', 'mystorageacct', ...]
cli.knack.cli: Command arguments: ...
cli.azure.cli.core.sdk.policies: Request URL: 'https://management.azure.com/...'
cli.azure.cli.core.sdk.policies: Request method: 'PUT'
cli.azure.cli.core.sdk.policies: Request headers: { ... }
cli.azure.cli.core.sdk.policies: Response status: 201
cli.azure.cli.core.sdk.policies: Response content: { ... }
```

**Troubleshooting Workflow**:
1. **Run command with `--debug`**
2. **Check "Command arguments"**: Verify arguments parsed correctly
3. **Check "Request URL"**: Verify correct API endpoint
4. **Check "Request method"**: Verify HTTP method (GET, PUT, POST, DELETE)
5. **Check "Response status"**: HTTP status code (200=success, 400=bad request, 500=server error)
6. **Check "Response content"**: Error message from Azure Resource Manager

**Common Issues**:
- **Backtick escaping**: Bash interprets backticks as command substitution
- **Variable expansion**: Use double quotes around queries with variables
- **JMESPath syntax**: Check filter expressions with `?` indicator

## Azure CLI vs ARM Templates

Azure CLI and ARM templates serve complementary roles in IaC:

| Aspect | Azure CLI Scripts | ARM Templates |
|--------|-------------------|---------------|
| **Format** | Bash/PowerShell | JSON (declarative) |
| **Learning Curve** | Lower (imperative) | Higher (declarative) |
| **Logic** | Full programming (if/for/while) | Limited (copy loops, conditions) |
| **Error Handling** | Explicit (try/catch, exit codes) | Implicit (rollback on failure) |
| **Idempotency** | Manual (check existence) | Automatic (desired state) |
| **Validation** | Runtime (during execution) | Pre-deployment (`az deployment validate`) |
| **Readability** | Sequential steps | Resource dependencies |
| **Best For** | Ad-hoc tasks, complex logic | Declarative infrastructure |

**When to Use Each**:

**Azure CLI** (Imperative):
- Complex conditional logic (if-else branches)
- Dynamic resource naming/counting
- Integration with external systems (APIs, databases)
- Ad-hoc administrative tasks
- Quick prototyping and experimentation

**ARM Templates** (Declarative):
- Large-scale infrastructure (50+ resources)
- Complex resource dependencies
- Built-in rollback on failure
- What-if analysis before deployment
- Template specs and versioning

**Hybrid Approach** (Best Practice):
```bash
#!/bin/bash
# Use Azure CLI for orchestration
environment=$1

# Use ARM template for declarative resource deployment
az deployment group create \
    --resource-group "rg-app-$environment" \
    --template-file infrastructure/main.json \
    --parameters "environment=$environment"

# Use Azure CLI for post-deployment tasks
storageKey=$(az storage account keys list \
    --resource-group "rg-app-$environment" \
    --account-name "stapp${environment}" \
    --query "[0].value" -o tsv)

# Configure application with storage key
echo "STORAGE_KEY=$storageKey" >> .env
```

## Real-World Application Scenarios

### Scenario 1: Multi-Environment Deployment Pipeline

**Requirement**: Deploy application to Dev, Staging, and Production with environment-specific configurations.

**Solution**:
```bash
#!/bin/bash
# deploy-multi-environment.sh

environment=$1

case $environment in
    dev)
        subscription="Dev-Subscription-ID"
        location="eastus"
        sku="Standard_LRS"
        ;;
    staging)
        subscription="Staging-Subscription-ID"
        location="centralus"
        sku="Standard_GRS"
        ;;
    prod)
        subscription="Prod-Subscription-ID"
        location="eastus2"
        sku="Premium_ZRS"
        ;;
    *)
        echo "Error: Environment must be dev, staging, or prod"
        exit 1
        ;;
esac

az account set --subscription "$subscription"

az group create \
    --name "rg-app-$environment" \
    --location "$location" \
    --tags Environment=$environment

az storage account create \
    --name "stapp${environment}$(date +%s)" \
    --resource-group "rg-app-$environment" \
    --location "$location" \
    --sku "$sku" \
    --tags Environment=$environment
```

### Scenario 2: Automated Testing Infrastructure

**Requirement**: Create temporary test environments before each deployment, run tests, then cleanup.

**Solution**:
```bash
#!/bin/bash
# test-deployment.sh

timestamp=$(date +%s)
testRg="rg-test-$timestamp"

# Create test environment
az group create --name "$testRg" --location eastus --tags Purpose=Testing

az storage account create \
    --name "sttest${timestamp}" \
    --resource-group "$testRg" \
    --location eastus \
    --sku Standard_LRS

# Run tests
./run-integration-tests.sh "$testRg"
testResult=$?

# Cleanup
az group delete --name "$testRg" --yes --no-wait

exit $testResult
```

### Scenario 3: Disaster Recovery Automation

**Requirement**: Regularly backup configurations and enable quick restoration.

**Solution**:
```bash
#!/bin/bash
# backup-configurations.sh

backupDir="backups/$(date +%Y%m%d)"
mkdir -p "$backupDir"

# Export resource groups
az group list --query "[].{name:name,location:location}" > "$backupDir/resource-groups.json"

# Export all storage accounts
az storage account list --query "[].{name:name,resourceGroup:resourceGroup,location:location,sku:sku.name}" > "$backupDir/storage-accounts.json"

# Export network configurations
az network vnet list --query "[].{name:name,resourceGroup:resourceGroup,addressSpace:addressSpace.addressPrefixes}" > "$backupDir/vnets.json"

echo "Backup completed: $backupDir"
```

**Restore Script**:
```bash
#!/bin/bash
# restore-configurations.sh

backupFile=$1

for account in $(jq -r '.[] | @base64' "$backupFile"); do
    _jq() {
        echo "$account" | base64 --decode | jq -r "$1"
    }
    
    az storage account create \
        --name "$(_jq '.name')" \
        --resource-group "$(_jq '.resourceGroup')" \
        --location "$(_jq '.location')" \
        --sku "$(_jq '.sku')"
done
```

### Scenario 4: Cost Optimization with Scheduled Resources

**Requirement**: Stop/start non-production VMs outside business hours to save costs.

**Solution**:
```bash
#!/bin/bash
# scheduled-vm-management.sh

action=$1  # start or stop
environment="dev"

vmList=$(az vm list --resource-group "rg-app-$environment" --query "[].name" -o tsv)

for vmName in $vmList; do
    echo "${action^}ing VM: $vmName"
    az vm $action --name "$vmName" --resource-group "rg-app-$environment" --no-wait
done

echo "All VMs ${action} initiated"
```

**Azure Automation Runbook** (scheduled):
```powershell
# Stop VMs at 6 PM
Schedule: Daily at 18:00
Script: ./scheduled-vm-management.sh stop

# Start VMs at 8 AM
Schedule: Daily at 08:00
Script: ./scheduled-vm-management.sh start
```

## Performance and Optimization

### Best Practices for Fast Scripts

1. **Minimize API Calls**:
```bash
# BAD: 50+ API calls
for rg in $(az group list --query "[].name" -o tsv); do
    az storage account list --resource-group $rg
done

# GOOD: 1 API call
az storage account list
```

2. **Use Server-Side Filtering** (JMESPath):
```bash
# BAD: Client-side filtering with jq
az storage account list -o json | jq '.[] | select(.location=="eastus")'

# GOOD: Server-side filtering with --query
az storage account list --query "[?location=='eastus']"
```

3. **Async Operations** (--no-wait):
```bash
# BAD: Wait for each deletion (10+ minutes)
for rg in $testResourceGroups; do
    az group delete --name $rg --yes  # Blocks until complete
done

# GOOD: Initiate all deletions (returns immediately)
for rg in $testResourceGroups; do
    az group delete --name $rg --yes --no-wait
done
```

4. **Output Format Optimization**:
```bash
# BAD: JSON output for simple value extraction
names=$(az storage account list -o json | jq -r '.[].name')

# GOOD: TSV output for scripting
names=$(az storage account list --query "[].name" -o tsv)
```

5. **Cache Configuration**:
```bash
# Enable caching (speeds up repeated queries)
az config set core.cache_enabled=true

# Clear cache when needed
az cache purge
```

## Security Best Practices

1. **Use Service Principals for Automation**:
```bash
# Never use interactive login in CI/CD
az login --service-principal \
    --username $AZURE_CLIENT_ID \
    --password $AZURE_CLIENT_SECRET \
    --tenant $AZURE_TENANT_ID
```

2. **Use Managed Identity on Azure VMs**:
```bash
# No credentials needed
az login --identity
```

3. **Avoid Hardcoding Credentials**:
```bash
# BAD: Hardcoded in script
subscriptionId="12345678-1234-1234-1234-123456789012"

# GOOD: Environment variable
subscriptionId=${AZURE_SUBSCRIPTION_ID}
```

4. **Principle of Least Privilege**:
- Grant service principals minimum permissions
- Use role-based access control (RBAC)
- Scope permissions to specific resource groups

5. **Secure Secrets Management**:
```bash
# Store secrets in Azure Key Vault
az keyvault secret set \
    --vault-name myKeyVault \
    --name storageAccountKey \
    --value "$storageKey"

# Retrieve in scripts
storageKey=$(az keyvault secret show \
    --vault-name myKeyVault \
    --name storageAccountKey \
    --query value -o tsv)
```

## Next Steps

### Continue Learning

Now that you've mastered Azure CLI fundamentals, explore advanced topics:

1. **Module 4: Explore Azure Automation with DevOps** (next in learning path)
   - Azure Automation accounts
   - PowerShell and Python runbooks
   - Hybrid Runbook Workers
   - Source control integration
   - Webhooks for automation

2. **Module 5: Implement Desired State Configuration (DSC)**
   - Configuration as code
   - DSC resources and modules
   - Compliance monitoring
   - Azure Automation DSC integration

3. **Module 6: Implement Bicep**
   - Modern ARM template language
   - Strongly-typed parameters
   - Modular infrastructure code
   - Bicep CLI tooling

### Additional Resources

**Official Documentation**:
- **Azure CLI Reference**: [https://docs.microsoft.com/cli/azure](https://docs.microsoft.com/cli/azure)
- **Azure CLI GitHub Repository**: [https://github.com/Azure/azure-cli](https://github.com/Azure/azure-cli)
- **Azure CLI Extensions**: [https://docs.microsoft.com/cli/azure/azure-cli-extensions-list](https://docs.microsoft.com/cli/azure/azure-cli-extensions-list)

**Quick References**:
- **Azure CLI Onboarding Cheat Sheet**: [Download PDF](https://docs.microsoft.com/cli/azure/cheat-sheet)
- **JMESPath Tutorial**: [https://jmespath.org/tutorial.html](https://jmespath.org/tutorial.html)
- **Azure CLI Samples**: [https://github.com/Azure-Samples/azure-cli-samples](https://github.com/Azure-Samples/azure-cli-samples)

**Tutorials**:
- **Learn to Use Azure CLI**: [https://docs.microsoft.com/cli/azure/get-started-with-azure-cli](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli)
- **Azure CLI Interactive Tutorial**: `az interactive` (install with `az extension add --name interactive`)
- **Query Results with JMESPath**: [https://docs.microsoft.com/cli/azure/query-azure-cli](https://docs.microsoft.com/cli/azure/query-azure-cli)

**Community Resources**:
- **Stack Overflow**: [https://stackoverflow.com/questions/tagged/azure-cli](https://stackoverflow.com/questions/tagged/azure-cli)
- **Microsoft Q&A**: [https://docs.microsoft.com/answers/topics/azure-cli.html](https://docs.microsoft.com/answers/topics/azure-cli.html)
- **GitHub Issues**: [https://github.com/Azure/azure-cli/issues](https://github.com/Azure/azure-cli/issues)
- **Azure CLI Twitter**: [@azurecli](https://twitter.com/azurecli)

**Certification Preparation**:
- **AZ-400: DevOps Engineer Expert**: Azure CLI is heavily tested
- **AZ-104: Azure Administrator**: CLI administration skills
- **AZ-305: Azure Solutions Architect**: Architecture automation with CLI

### Practice Exercises

**Exercise 1: Multi-Region Deployment**
Create a script that deploys identical resources to three Azure regions (East US, West Europe, Southeast Asia).

**Exercise 2: Resource Tagging Automation**
Write a script that applies consistent tags (Owner, CostCenter, Environment, Project) to all resources in a resource group.

**Exercise 3: Compliance Reporting**
Create a report of all storage accounts without HTTPS-only enforcement enabled, then enable it for those accounts.

**Exercise 4: Backup and Restore**
Implement a complete backup/restore solution for storage account configurations using JSON exports.

**Exercise 5: CI/CD Integration**
Set up a GitHub Actions or Azure Pipelines workflow that deploys infrastructure changes on push to main branch.

## Final Thoughts

Azure CLI is a powerful, flexible tool that bridges the gap between manual portal operations and fully declarative infrastructure as code. By mastering Azure CLI:

✅ **You can automate repetitive tasks**, freeing time for strategic work  
✅ **You enable version control for infrastructure**, bringing DevOps practices to resource management  
✅ **You integrate infrastructure deployment into CI/CD pipelines**, achieving true continuous delivery  
✅ **You work across all platforms**, from Windows desktops to Linux servers to containerized environments  
✅ **You troubleshoot issues effectively** using `--debug` and comprehensive help resources

The skills you've gained in this module form the foundation for advanced Infrastructure as Code practices with Azure Automation, DSC, and Bicep in subsequent modules.

**Remember**: The best way to master Azure CLI is through practice. Start small with simple scripts, gradually add error handling and parameterization, and evolve toward production-ready automation.

Happy scripting, and welcome to the world of Infrastructure as Code with Azure CLI! 🚀

---

**Module**: Create Azure Resources Using Azure CLI  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/create-azure-resources-by-using-azure-cli/8-summary
