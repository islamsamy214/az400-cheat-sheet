# What Is Azure CLI?

## Overview

Azure CLI (Command-Line Interface) is a cross-platform command-line tool that provides a unified interface for creating and managing Azure resources. Unlike graphical interfaces that require mouse clicks through multiple screens, Azure CLI enables you to execute Azure operations with simple, consistent commands that work identically across Windows, Linux, and macOS.

Azure CLI translates your commands into REST API calls to Azure Resource Manager, the same backend that powers the Azure portal. This means anything you can do in the portal, you can do with Azure CLI—often more efficiently and with the ability to automate through scripting.

## Cross-Platform Compatibility

### Supported Platforms

Azure CLI runs everywhere Azure customers work:

| Platform | Shells | Installation Method | Use Case |
|----------|--------|-------------------|----------|
| **Linux** | Bash, PowerShell 7 | Package manager (apt, yum), script | Servers, developer workstations, CI/CD agents |
| **macOS** | Bash, Zsh, PowerShell 7 | Homebrew, installer package | Developer workstations, local testing |
| **Windows** | cmd.exe, PowerShell, PowerShell 7 | MSI installer | Enterprise desktops, Windows servers |
| **Docker** | Bash | Pre-built container image | Containerized CI/CD, isolated environments |
| **Azure Cloud Shell** | Bash, PowerShell | Pre-installed (no setup) | Browser-based, learning, quick tasks |

### Platform Selection Guidance

**Use Linux When**:
- Running in CI/CD pipelines (GitHub Actions, Azure Pipelines with Linux agents)
- Managing Azure from existing Linux servers
- Leveraging Bash scripting expertise
- Building containerized automation workflows

**Use macOS When**:
- Developing locally on Mac hardware
- Testing scripts before deploying to Linux production environments
- Integrating with macOS development tools (Xcode, VS Code)

**Use Windows When**:
- Enterprise Windows environments with existing PowerShell expertise
- Integration with Windows-based automation (Task Scheduler, SCCM)
- Managing Azure from Windows Server infrastructure

**Use Docker When**:
- Need consistent, reproducible environments
- Running in Kubernetes-based CI/CD systems
- Isolating Azure CLI versions for different projects
- Avoiding system-wide installations

**Use Azure Cloud Shell When**:
- Learning Azure CLI without installation
- Quick administrative tasks from any browser
- Working from restricted environments (no local admin rights)
- Accessing Azure from mobile devices

## Command Structure

### Anatomy of an Azure CLI Command

Azure CLI uses a hierarchical, intuitive command structure:

```
az <reference-group> <subgroup> <command> --<parameter> <value>
```

#### Command Components

1. **`az`**: The root command that invokes Azure CLI
2. **Reference Group**: Top-level organizational unit (e.g., `account`, `vm`, `storage`, `network`)
3. **Subgroup** (optional): Nested organizational unit (e.g., `storage account`, `vm disk`)
4. **Command**: The action to perform (e.g., `create`, `list`, `delete`, `show`, `update`)
5. **Parameters**: Arguments that specify options and values (e.g., `--name`, `--resource-group`, `--location`)

### Command Examples with Breakdown

#### Example 1: Set Default Subscription
```bash
az account set --subscription "My Production Subscription"
```
- **Reference Group**: `account` (Azure subscription management)
- **Command**: `set` (change default subscription)
- **Parameter**: `--subscription` with value "My Production Subscription"

#### Example 2: Create Resource Group
```bash
az group create --name rg-app-prod --location eastus --tags Environment=Production CostCenter=IT
```
- **Reference Group**: `group` (resource group management)
- **Command**: `create` (create new resource group)
- **Parameters**: 
  - `--name`: rg-app-prod
  - `--location`: eastus
  - `--tags`: Environment=Production CostCenter=IT

#### Example 3: List Storage Accounts with Query
```bash
az storage account list --resource-group rg-app-prod --query "[?kind=='StorageV2'].name" --output table
```
- **Reference Group**: `storage`
- **Subgroup**: `account`
- **Command**: `list` (retrieve storage accounts)
- **Parameters**:
  - `--resource-group`: rg-app-prod (filter by resource group)
  - `--query`: JMESPath query to filter results
  - `--output`: Display format (table, json, tsv, yaml)

#### Example 4: Restart Virtual Machine
```bash
az vm restart --resource-group rg-app-prod --name vm-webserver-01
```
- **Reference Group**: `vm` (virtual machine management)
- **Command**: `restart` (reboot VM)
- **Parameters**:
  - `--resource-group`: rg-app-prod
  - `--name`: vm-webserver-01

#### Example 5: Complex Network Configuration
```bash
az network vnet create \
    --resource-group rg-network \
    --name vnet-prod \
    --address-prefix 10.0.0.0/16 \
    --subnet-name subnet-web \
    --subnet-prefix 10.0.1.0/24 \
    --location eastus \
    --tags Environment=Production Tier=Network
```
- **Reference Group**: `network`
- **Subgroup**: `vnet` (virtual network)
- **Command**: `create`
- **Parameters**: Multiple configuration values for VNet and subnet

### Command Parameter Types

#### Required Parameters
Most commands have required parameters that must be provided:
```bash
az group create --name <required> --location <required>
```

#### Optional Parameters
Enhance functionality or override defaults:
```bash
az storage account create \
    --name mystorageaccount \
    --resource-group myResourceGroup \
    --location eastus \
    --sku Standard_LRS \              # Optional: defaults to Standard_RAGRS
    --kind StorageV2 \                # Optional: defaults to StorageV2
    --tags Environment=Dev            # Optional: no tags by default
```

#### Boolean Parameters
Enable features without values:
```bash
az vm create \
    --name myVM \
    --resource-group myResourceGroup \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys               # Boolean flag (no value needed)
```

#### Global Parameters
Work with any command:
- `--output`, `-o`: Format output (json, jsonc, table, tsv, yaml, yamlc, none)
- `--query`: JMESPath query to filter results
- `--verbose`: Increase logging verbosity
- `--debug`: Show detailed debug output
- `--help`, `-h`: Display command help

## Core References vs. Extensions

### Core References (Built-In)

Core references are permanently included with Azure CLI and don't require additional installation. They cover the most commonly used Azure services:

| Reference Group | Purpose | Example Commands |
|-----------------|---------|------------------|
| `az account` | Subscription management | `az account list`, `az account set` |
| `az group` | Resource group operations | `az group create`, `az group delete` |
| `az storage` | Storage services | `az storage account create`, `az storage blob upload` |
| `az vm` | Virtual machine management | `az vm create`, `az vm start`, `az vm stop` |
| `az network` | Networking resources | `az network vnet create`, `az network nsg rule create` |
| `az webapp` | App Service applications | `az webapp create`, `az webapp config set` |
| `az sql` | Azure SQL resources | `az sql server create`, `az sql db create` |
| `az aks` | Azure Kubernetes Service | `az aks create`, `az aks get-credentials` |
| `az keyvault` | Key Vault operations | `az keyvault create`, `az keyvault secret set` |
| `az monitor` | Monitoring and diagnostics | `az monitor metrics list`, `az monitor log-analytics workspace create` |

**Core Reference Benefits**:
- Always available after Azure CLI installation
- Stable, well-tested functionality
- Optimized performance
- Comprehensive documentation
- Generally available (GA) status

### Extensions (Optional Add-Ons)

Extensions provide functionality for:
- **Preview Services**: New Azure services in public preview
- **Specialized Scenarios**: Advanced features for specific use cases
- **Third-Party Integration**: Tools that integrate with Azure services

#### Installing Extensions

Azure CLI automatically prompts to install extensions when needed:
```bash
# First use of an extension command prompts for installation
az aks-preview create --help
# Prompt: "The command requires the extension aks-preview. Do you want to install it? [Y/n]"
```

Manual extension installation:
```bash
# Install specific extension
az extension add --name azure-devops

# List installed extensions
az extension list --output table

# Update all extensions
az extension update --name azure-devops

# Remove extension
az extension remove --name azure-devops
```

#### Popular Extensions

| Extension | Purpose | Use Case |
|-----------|---------|----------|
| `azure-devops` | Azure DevOps integration | Manage pipelines, repos, work items from CLI |
| `aks-preview` | AKS preview features | Test upcoming Kubernetes features |
| `application-insights` | Application Insights | Query telemetry, manage components |
| `containerapp` | Azure Container Apps | Deploy containerized applications |
| `datafactory` | Data Factory | Manage data integration pipelines |

#### Extension Management Best Practices

1. **Install Only What You Need**: Extensions increase CLI size and startup time
2. **Keep Extensions Updated**: `az extension update --name <extension-name>`
3. **Review Before Installing**: Check extension description and publisher
4. **Use Version Pinning in CI/CD**: Ensure consistent behavior across pipeline runs

## Reference Status Levels

Azure CLI commands have status indicators showing their maturity:

### General Availability (GA)
- **Production-Ready**: Fully tested, supported by Microsoft
- **Stable API**: Commands won't break in minor version updates
- **SLA Coverage**: Covered by Azure service SLAs
- **Example**: `az vm create`, `az storage account create`

**Indicator**: No status label (GA is default)

### Public Preview
- **Testing Phase**: New features available for customer feedback
- **Breaking Changes Possible**: API may change before GA
- **No SLA**: Not recommended for production workloads
- **Example**: `az ml model create` (some ML features)

**Indicator**: `[Preview]` label in help text

### Experimental
- **Proof of Concept**: Very early stage, subject to significant changes
- **Limited Support**: May be removed without notice
- **Feedback Welcome**: Microsoft seeks input on design
- **Example**: Certain AI service integrations

**Indicator**: `[Experimental]` label in help text

### Deprecated
- **Phased Out**: Being replaced by newer commands
- **Grace Period**: Usually 6-12 months before removal
- **Migration Path**: Documentation shows replacement commands
- **Example**: `az functionapp devops-build` (replaced by `az functionapp deployment`)

**Indicator**: `[Deprecated]` warning message

### Checking Command Status

```bash
# View command help to see status
az aks create --help
# Output shows: [GA] or [Preview] or [Deprecated]

# Check specific reference group
az aks --help
# Lists all commands with status indicators
```

## Execution Modes

Azure CLI supports two primary execution modes, each suited for different workflows:

### Interactive Mode

Execute commands one at a time in a terminal session.

**Characteristics**:
- Immediate feedback after each command
- Results displayed in real-time
- Manual command entry
- Session context preserved (authenticated user, default subscription)

**Best For**:
- **Learning**: Explore Azure services and command syntax
- **Troubleshooting**: Investigate issues interactively
- **One-Time Tasks**: Create single resources, investigate configurations
- **Exploration**: Discover resource properties with `show` and `list` commands

**Example Workflow**:
```bash
# Interactive troubleshooting session
az login
az account show  # Verify subscription

# Investigate storage account issues
az storage account list --resource-group rg-prod --output table
az storage account show --name stproddata001 --query 'primaryEndpoints'
az storage account keys list --account-name stproddata001

# Check network connectivity
az network vnet show --name vnet-prod --resource-group rg-network
az network vnet subnet list --vnet-name vnet-prod --resource-group rg-network
```

**Advantages**:
- Immediate visual feedback
- Easy to experiment and learn
- No script development needed
- Can pivot based on results

**Disadvantages**:
- Not repeatable without retyping commands
- Prone to typos and manual errors
- Difficult to share with team members
- No version control

### Scripted Mode

Combine multiple Azure CLI commands in script files (`.sh`, `.ps1`, `.bat`).

**Characteristics**:
- Multiple commands executed sequentially
- Variables, loops, and conditionals for logic
- Error handling and validation
- Reusable across projects and teams

**Best For**:
- **Automation**: Deploy infrastructure without manual intervention
- **Repetitive Tasks**: Create multiple similar resources
- **Multi-Step Operations**: Complex deployments requiring multiple commands
- **CI/CD Integration**: Embed in Azure Pipelines, GitHub Actions, Jenkins
- **Documentation**: Scripts serve as executable documentation

**Example Workflow**:
```bash
#!/bin/bash
# deploy-app-infrastructure.sh
# Automated deployment of application infrastructure

# Configuration
resourceGroup="rg-app-prod"
location="eastus"
appName="mywebapp"
dbName="sqldb-app"

# Create resource group
echo "Creating resource group $resourceGroup..."
az group create --name $resourceGroup --location $location

# Create App Service plan
echo "Creating App Service plan..."
az appservice plan create \
    --name "plan-$appName" \
    --resource-group $resourceGroup \
    --sku S1 \
    --is-linux

# Create web app
echo "Creating web application..."
az webapp create \
    --name $appName \
    --resource-group $resourceGroup \
    --plan "plan-$appName" \
    --runtime "NODE|18-lts"

# Create SQL server and database
echo "Creating SQL database..."
az sql server create \
    --name "sql-$appName" \
    --resource-group $resourceGroup \
    --admin-user sqladmin \
    --admin-password $(generate_secure_password)

az sql db create \
    --server "sql-$appName" \
    --resource-group $resourceGroup \
    --name $dbName \
    --service-objective S0

echo "Deployment complete!"
```

**Advantages**:
- **Repeatable**: Same script deploys consistently
- **Version Controlled**: Track changes with Git
- **Testable**: Validate scripts in dev before production
- **Shareable**: Team members use identical processes
- **Auditable**: Script history shows who changed what

**Disadvantages**:
- Requires scripting knowledge
- Debugging can be complex
- Initial development time investment

### Mode Comparison Matrix

| Aspect | Interactive Mode | Scripted Mode |
|--------|-----------------|---------------|
| **Learning Curve** | Low (type and see results) | Medium (requires scripting knowledge) |
| **Speed for Single Task** | Fast (no script needed) | Slower (must write script first) |
| **Speed for Repeated Tasks** | Slow (retype each time) | Very Fast (run script) |
| **Consistency** | Low (manual errors) | High (automated) |
| **Documentation** | Poor (commands not saved) | Excellent (script is documentation) |
| **Collaboration** | Difficult (share command history?) | Easy (share script file) |
| **CI/CD Integration** | Not possible | Native support |
| **Error Handling** | Manual intervention | Automated retry/rollback |
| **Version Control** | Not applicable | Full Git integration |

### Hybrid Approach

Most teams use both modes strategically:

1. **Develop Interactively**: Test commands in interactive mode to verify syntax
2. **Formalize as Script**: Once commands work, combine into script
3. **Test Script**: Validate in dev/test environment
4. **Deploy to Production**: Use script for consistent production deployments
5. **Troubleshoot Interactively**: When issues arise, investigate interactively, then update script

**Example Workflow**:
```bash
# Step 1: Interactive testing
az vm create --name testvm --resource-group rg-test --image UbuntuLTS --admin-username azureuser --generate-ssh-keys
# Success! Now capture in script...

# Step 2: Create script with tested command
cat > create-vm.sh << 'EOF'
#!/bin/bash
vmName=$1
rgName=$2
az vm create --name $vmName --resource-group $rgName --image UbuntuLTS --admin-username azureuser --generate-ssh-keys
EOF

# Step 3: Test script
chmod +x create-vm.sh
./create-vm.sh dev-vm-01 rg-dev

# Step 4: Deploy to production
./create-vm.sh prod-vm-01 rg-prod

# Step 5: Commit to version control
git add create-vm.sh
git commit -m "Add VM creation script"
git push
```

## Command Discovery and Help

### Finding Commands with `az find`

Azure CLI includes AI-powered command search:

```bash
# Find commands related to "blob"
az find blob

# Find commands for specific service
az find "az storage"

# Find parameters for command
az find "az storage account create"
```

Output includes:
- Most popular commands matching search term
- Command descriptions
- Usage examples
- Related commands

### Getting Help with `--help`

Every command and subgroup supports `--help`:

```bash
# Reference group help
az storage --help

# Subgroup help
az storage account --help

# Command help
az storage account create --help
```

Help output includes:
- Command description
- Required and optional parameters
- Parameter data types and defaults
- Usage examples
- Related commands

### Quick Reference: Essential Commands

#### Account Management
```bash
az login                           # Authenticate to Azure
az account list                    # List subscriptions
az account show                    # Show current subscription
az account set --subscription <id> # Change default subscription
```

#### Resource Groups
```bash
az group create --name <name> --location <location>
az group list --output table
az group show --name <name>
az group delete --name <name> --yes
```

#### Storage Accounts
```bash
az storage account create --name <name> --resource-group <rg> --location <location>
az storage account list --resource-group <rg>
az storage account show --name <name>
az storage account keys list --account-name <name>
az storage account delete --name <name>
```

#### Virtual Machines
```bash
az vm create --name <name> --resource-group <rg> --image <image>
az vm list --resource-group <rg> --output table
az vm start --name <name> --resource-group <rg>
az vm stop --name <name> --resource-group <rg>
az vm deallocate --name <name> --resource-group <rg>
az vm delete --name <name> --resource-group <rg> --yes
```

## Real-World Usage Patterns

### Pattern 1: Environment-Specific Configuration
```bash
#!/bin/bash
# Environment-aware deployment
case $ENVIRONMENT in
    dev)
        location="eastus"
        sku="Basic"
        ;;
    prod)
        location="westus2"
        sku="Standard"
        ;;
esac

az appservice plan create --name "plan-app-$ENVIRONMENT" --resource-group "rg-$ENVIRONMENT" --location $location --sku $sku
```

### Pattern 2: Bulk Operations with Queries
```bash
# Stop all VMs in resource group with specific tag
vmIds=$(az vm list --resource-group rg-dev --query "[?tags.AutoShutdown=='true'].id" -o tsv)
for vmId in $vmIds; do
    az vm deallocate --ids $vmId
done
```

### Pattern 3: CI/CD Integration
```yaml
# GitHub Actions workflow
- name: Deploy Infrastructure
  run: |
    az login --service-principal -u ${{ secrets.AZURE_CLIENT_ID }} -p ${{ secrets.AZURE_CLIENT_SECRET }} --tenant ${{ secrets.AZURE_TENANT_ID }}
    az group create --name rg-app-prod --location eastus
    az deployment group create --resource-group rg-app-prod --template-file infrastructure.json
```

### Pattern 4: Infrastructure Validation
```bash
#!/bin/bash
# Verify infrastructure deployment
echo "Checking resource group exists..."
if ! az group exists --name rg-prod; then
    echo "ERROR: Resource group not found"
    exit 1
fi

echo "Checking App Service is running..."
state=$(az webapp show --name myapp --resource-group rg-prod --query "state" -o tsv)
if [ "$state" != "Running" ]; then
    echo "ERROR: App Service not running (state: $state)"
    exit 1
fi

echo "Infrastructure validation passed"
```

## Key Takeaways

Azure CLI provides:
1. **Unified Cross-Platform Interface**: Same commands work on Windows, Linux, macOS, Docker, and Cloud Shell
2. **Hierarchical Command Structure**: Intuitive `az <group> <subgroup> <command>` pattern
3. **Flexible Execution Modes**: Interactive for learning, scripted for automation
4. **Extensible Architecture**: Core references plus optional extensions
5. **Maturity Indicators**: GA, Preview, Experimental, and Deprecated status levels
6. **Powerful Query Language**: JMESPath filtering for complex result processing
7. **CI/CD Ready**: Native integration with Azure Pipelines, GitHub Actions, Jenkins

Whether managing a single resource group or orchestrating complex multi-service deployments, Azure CLI provides the tools needed for efficient, automated Azure management.

## Next Steps

Now that you understand what Azure CLI is and how it works, proceed to **Unit 3: Install Azure CLI** to set up your environment and authenticate to Azure.

---

**Module**: Create Azure Resources Using Azure CLI  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/create-azure-resources-by-using-azure-cli/2-what-is-the-azure-cli
