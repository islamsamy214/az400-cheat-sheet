# Tips to Use the Azure CLI Successfully

## Overview

Mastering Azure CLI requires more than knowing command syntax—it demands understanding how Azure CLI works behind the scenes, how to troubleshoot when things go wrong, and where to find help when stuck. This unit dives deep into Azure CLI internals, reveals powerful debugging techniques, explains scripting language differences that cause common errors, and provides comprehensive resources for ongoing learning.

Whether you're debugging a failing script, optimizing performance, or learning new commands, these tips transform you from a casual Azure CLI user into an expert who leverages the tool's full power.

## Understand Azure CLI API Calls

### What Happens Behind the Scenes

Every Azure CLI command translates to REST API calls to Azure Resource Manager. Understanding this translation helps debug issues, optimize performance, and troubleshoot unexpected behavior.

### Using `--debug` to Expose API Calls

The `--debug` parameter reveals detailed information about command execution:

```bash
az group create --location westus2 --name myResourceGroupName --debug
```

### Debug Output Analysis

**Example debug output** (abbreviated for readability):

```
cli.knack.cli: Command arguments: ['group', 'create', '--location', 'westus2', '--name', 'myResourceGroupName', '--debug']
cli.knack.cli: __init__ debug log:
...
cli.azure.cli.core: Modules found from index for 'group': ['azure.cli.command_modules.resource']
cli.azure.cli.core: Loading command modules:
...
cli.azure.cli.core: Loaded 53 groups, 233 commands.
cli.azure.cli.core: Found a match in the command table.
cli.azure.cli.core: Raw command  : group create
...
cli.azure.cli.core.azlogging: metadata file logging enabled - writing logs to '/home/myName/.azure/commands/2025-10-08.21-47-27.group_create.5217.log'.
...
cli.azure.cli.core.sdk.policies: Request URL: 'https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourcegroups/myResourceGroupName?api-version=2022-09-01'
cli.azure.cli.core.sdk.policies: Request method: 'PUT'
cli.azure.cli.core.sdk.policies: Request headers:
cli.azure.cli.core.sdk.policies:     'Content-Type': 'application/json'
cli.azure.cli.core.sdk.policies:     'Content-Length': '23'
cli.azure.cli.core.sdk.policies:     'Accept': 'application/json'
cli.azure.cli.core.sdk.policies:     'x-ms-client-request-id': 'c79caddc-ed78-11ef-8a83-00155dbc433c'
cli.azure.cli.core.sdk.policies:     'CommandName': 'group create'
cli.azure.cli.core.sdk.policies:     'ParameterSetName': '--location --name --debug'
...
cli.azure.cli.core.sdk.policies: Response content:
...
{
  "id": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/myResourceGroupName",
  "location": "westus2",
  "managedBy": null,
  "name": "myResourceGroupName",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
cli.knack.cli: Event: Cli.SuccessfulExecute []
cli.knack.cli: Event: Cli.PostExecute [<function AzCliLogging.deinit_cmd_metadata_logging at 0x7f98a6bc7820>]
az_command_data_logger: exit code: 0
...
```

### Key Insights from Debug Output

**1. Command Arguments Parsing**:
```
Command arguments: ['group', 'create', '--location', 'westus2', '--name', 'myResourceGroupName', '--debug']
```
- **Critical**: The scripting language (Bash, PowerShell, cmd.exe) parses arguments **before** Azure CLI receives them
- **Troubleshooting First Step**: If command fails, check this line to verify arguments parsed correctly
- **Common Issue**: Missing quotes, unescaped special characters, incorrect variable expansion

**2. Log File Location**:
```
metadata file logging enabled - writing logs to '/home/myName/.azure/commands/2025-10-08.21-47-27.group_create.5217.log'
```
- **Purpose**: Detailed logs stored for post-mortem analysis
- **Location**: `~/.azure/commands/` (Linux/macOS) or `%USERPROFILE%\.azure\commands\` (Windows)
- **Use Case**: Share logs with Microsoft support or review complex failures

**3. Request URL and Method**:
```
Request URL: 'https://management.azure.com/subscriptions/.../resourcegroups/myResourceGroupName?api-version=2022-09-01'
Request method: 'PUT'
```
- **Shows**: Actual REST API endpoint called
- **HTTP Method**: PUT (create/update), GET (retrieve), DELETE (remove), POST (actions)
- **API Version**: Azure service API version used
- **Troubleshooting**: Verify correct subscription ID, resource name, API version

**4. Request Headers**:
```
'CommandName': 'group create'
'ParameterSetName': '--location --name --debug'
```
- **Telemetry**: Microsoft tracks command usage to improve Azure CLI
- **Privacy**: No sensitive data (passwords, keys) sent in headers
- **Debugging**: Confirms parameters Azure CLI received

**5. Response Content**:
```json
{
  "provisioningState": "Succeeded"
}
```
- **Success Indicator**: "Succeeded" confirms resource created
- **Error Details**: If failed, response includes error code and message
- **Full Response**: Complete resource properties returned

**6. Exit Code**:
```
exit code: 0
```
- **0**: Success
- **Non-zero**: Error occurred
- **CI/CD Integration**: Pipelines check exit codes to determine job success/failure

### When to Use `--debug`

**Ideal Scenarios**:
1. **Script Behaves Unexpectedly**: See how arguments are actually parsed
2. **Authentication Failures**: Verify subscription, tenant, credentials used
3. **Permissions Issues**: Check exact API endpoint accessed
4. **Intermittent Failures**: Capture full request/response for analysis
5. **Learning**: Understand what Azure CLI does behind the scenes
6. **Performance**: Identify slow API calls

**Example Troubleshooting Workflow**:
```bash
# Command fails mysteriously
az storage account create --name myaccount --resource-group myrg --location eastus
# Error: invalid jmespath_type value

# Add --debug to investigate
az storage account create --name myaccount --resource-group myrg --location eastus --debug | grep "Command arguments"
# Output shows how Bash parsed the command

# Identify issue: Bash expanded variable incorrectly
# Fix and retry
```

## Troubleshooting with `--debug`

### Real-World Troubleshooting Example

**Scenario**: Finding storage accounts with blob public access enabled

**Incorrect Script** (Bash):
```bash
resourceGroup="msdocs-rg-00000000"
az storage account list --resource-group $resourceGroup --query "[?allowBlobPublicAccess == `true`].id"
```

**Error Encountered**:
```
argument --query: invalid jmespath_type value: '[?allowBlobPublicAccess == ].id'
```

**Problem**: Where did the value `true` go?

### Debugging the Issue

Add `--debug` to see argument parsing:

```bash
az storage account list --resource-group $resourceGroup --query "[?allowBlobPublicAccess == `true`].id" --debug
```

**Debug Output**:
```
Command arguments: ['storage', 'account', 'list', '--resource-group', 'msdocs-rg-00000000', '--query', '[?allowBlobPublicAccess == ].id', '--debug']
```

**Analysis**: The `--query` value shows `[?allowBlobPublicAccess == ].id` with **missing boolean value**. Bash interpreted backticks (`) as command substitution and removed `true`.

### Solution: Escape Backticks in Bash

**Correct Bash Script**:
```bash
resourceGroup="msdocs-rg-00000000"
az storage account list --resource-group $resourceGroup --query "[?allowBlobPublicAccess == \`true\`].id" --debug
```

**Debug Output** (corrected):
```
Command arguments: ['storage', 'account', 'list', '--resource-group', 'msdocs-rg-00000000', '--query', '[?allowBlobPublicAccess == `true`].id', '--debug']
```

**Result**: Now backticks are preserved, JMESPath query executes correctly.

**Key Lesson**: The scripting environment (Bash, PowerShell, cmd.exe) parses command strings **before** Azure CLI sees them. Debug output shows what Azure CLI actually received.

## Scripting Language Syntax Differences

### Why Syntax Differences Matter

Azure CLI documentation is primarily written for Bash (tested in Azure Cloud Shell). Running examples in PowerShell or cmd.exe often requires syntax modifications.

### Common Syntax Differences

| Construct | Bash | PowerShell | cmd.exe |
|-----------|------|------------|---------|
| **Line continuation** | Backslash (`\`) | Backtick (`` ` ``) | Caret (`^`) |
| **Variable naming** | `variableName=varValue` | `$variableName="varValue"` | `set variableName=varValue` |
| **Number as string** | `\`50\`` | `` `50` `` | `"50"` |
| **Boolean as string** | `\`true\`` | `` `false` `` | `"true"` |
| **Random ID** | `let "randomIdentifier=$RANDOM*$RANDOM"` | `$randomIdentifier = (New-Guid).ToString().Substring(0,8)` | `set randomIdentifier=%RANDOM%` |
| **Looping** | `for`, `while`, `until` | `for`, `foreach`, `while`, `do-while`, `do-until` | `for` |
| **Write to console** | `echo` | `Write-Host` (preferred) or `echo` | `echo` |

### Examples by Language

#### Bash Example

```bash
#!/bin/bash
# Bash line continuation with backslash
az storage account create \
    --name mystorageacct \
    --resource-group myrg \
    --location eastus \
    --sku Standard_LRS

# Bash variables (no $)
storageAccount="mystorageacct"

# Bash JMESPath with escaped backticks
az storage account list --query "[?kind==\`StorageV2\`].name"

# Bash random identifier
let "randomIdentifier=$RANDOM*$RANDOM"
```

#### PowerShell Example

```powershell
# PowerShell line continuation with backtick
az storage account create `
    --name mystorageacct `
    --resource-group myrg `
    --location eastus `
    --sku Standard_LRS

# PowerShell variables (with $)
$storageAccount="mystorageacct"

# PowerShell JMESPath with backtick escaping
az storage account list --query "[?kind==``StorageV2``].name"

# PowerShell random identifier
$randomIdentifier = (New-Guid).ToString().Substring(0,8)
```

#### cmd.exe Example

```batch
REM cmd.exe line continuation with caret
az storage account create ^
    --name mystorageacct ^
    --resource-group myrg ^
    --location eastus ^
    --sku Standard_LRS

REM cmd.exe variables (no $)
set storageAccount=mystorageacct

REM cmd.exe JMESPath with quotes
az storage account list --query "[?kind=='StorageV2'].name"

REM cmd.exe random identifier
set randomIdentifier=%RANDOM%
```

### Common Pitfalls

**1. Copying Bash Examples to PowerShell**:
```bash
# Bash example (from documentation)
az vm create \
    --name myVM \
    --resource-group myRG

# PowerShell error: Backslash not recognized
# Fix: Replace \ with `
az vm create `
    --name myVM `
    --resource-group myRG
```

**2. Unescaped Backticks in JMESPath**:
```bash
# Wrong in Bash (backticks interpreted as command substitution)
az storage account list --query "[?kind==`StorageV2`].name"

# Correct in Bash (escaped backticks)
az storage account list --query "[?kind==\`StorageV2\`].name"

# PowerShell (double backticks)
az storage account list --query "[?kind==``StorageV2``].name"
```

**3. Variable Expansion**:
```bash
# Bash: No $ needed for assignment
resourceGroup="myRG"
az group create --name $resourceGroup --location eastus

# PowerShell: $ required
$resourceGroup="myRG"
az group create --name $resourceGroup --location eastus

# cmd.exe: % delimiters
set resourceGroup=myRG
az group create --name %resourceGroup% --location eastus
```

### Cross-Platform Script Strategy

**Option 1: Write Platform-Specific Scripts**
- Separate scripts for Bash, PowerShell, cmd.exe
- Optimal syntax for each platform
- Clear documentation of platform requirements

**Option 2: Test in Multiple Environments**
- Write in preferred language (e.g., Bash)
- Test in Azure Cloud Shell (Bash)
- Document conversion steps for other platforms

**Option 3: Use Azure Cloud Shell**
- Consistent Bash environment
- No platform syntax issues
- Pre-authenticated, pre-configured
- Ideal for learning and documentation

## More Ways to Get Help

### Use `az find`

AI-powered command search helps discover commands and usage patterns:

**Find Commands Related to a Topic**:
```bash
# Find blob-related commands
az find blob
```

**Output Example**:
```
Most popular blob-related commands:
  az storage blob upload
  az storage blob download
  az storage blob list
  az storage blob delete
  az storage account blob-service-properties update

Related searches:
  azure blob storage
  blob container
  blob SAS token
```

**Find Popular Commands for Service**:
```bash
# Most popular commands for Azure Storage
az find "az storage"

# Popular commands for specific resource type
az find "az storage account create"
```

**Example Output**:
```
Popular commands for "az storage account create":

az storage account create --name <name> --resource-group <rg> --location <location>
Creates a new storage account.

Most used parameters:
  --name: Storage account name (required)
  --resource-group: Resource group (required)
  --location: Azure region (required)
  --sku: Performance tier (default: Standard_RAGRS)
  --kind: Account type (default: StorageV2)

Examples:
  az storage account create --name mystorageacct --resource-group myrg --location eastus --sku Standard_LRS
```

### Use `--help`

Every command and subgroup supports `--help` for detailed documentation:

**Reference Group Help**:
```bash
az storage --help
```

**Subgroup Help**:
```bash
az storage account --help
```

**Command Help**:
```bash
az storage account create --help
```

**Help Output Includes**:
- Command description
- Required and optional parameters
- Parameter data types and defaults
- Usage examples
- Related commands
- Links to online documentation

**Example Help Output**:
```bash
az storage account create --help

Command
    az storage account create : Create a storage account.

Arguments
    --name -n           [Required] : Storage account name.
    --resource-group -g [Required] : Resource group name.
    --location -l       [Required] : Location.
    --sku                          : The storage account SKU.  Allowed values: Standard_LRS,
                                     Standard_GRS, Standard_RAGRS, Premium_LRS.
                                     Default: Standard_RAGRS.
    --kind                         : The storage account type.  Allowed values: Storage,
                                     StorageV2, BlobStorage.  Default: StorageV2.

Examples
    Create a storage account with the specified SKU.
        az storage account create -n mystorageaccount -g myresourcegroup -l eastus --sku Standard_LRS
```

### A to Z Documentation Indexes

#### 1. Reference Index

**URL**: https://learn.microsoft.com/en-us/cli/azure/reference-index

**Content**: A to Z list of all Azure CLI reference groups
- Expand left navigation for subgroups
- Complete command syntax
- Parameter descriptions
- Code examples

**Use When**: Need complete API reference for a command

#### 2. Conceptual Article List

**URL**: https://learn.microsoft.com/en-us/cli/azure/reference-docs-index

**Content**: A to Z list of tutorials, how-to guides, quickstarts
- Grouped by Azure CLI command group
- Real-world scenarios
- Best practices
- Step-by-step instructions

**Use When**: Learning how to accomplish specific tasks

**Search Tip**: Use Ctrl+F (Windows) or Cmd+F (macOS) to jump to command group

#### 3. Azure CLI Sample Scripts

**URL**: https://learn.microsoft.com/en-us/cli/azure/samples-index

**Three Tabs**:

**a) List by Subject Area**:
- Find samples by Azure service (Compute, Storage, Networking, etc.)
- Browse by solution area (DevOps, Security, Monitoring, etc.)

**b) List by Reference Group**:
- Find samples by Azure CLI command group
- Example: All samples using `az vm`, `az storage`, `az network`

**c) GitHub Repository**:
- Direct link to Azure CLI samples repository
- Full working scripts
- README with explanations
- Clone repository for offline access

**Example Use Case**:
```bash
# Clone samples repository
git clone https://github.com/Azure-Samples/azure-cli-samples.git
cd azure-cli-samples

# Browse samples by service
ls -l storage/
ls -l compute/

# Run example script
cd storage/create-storage-account
cat create-storage-account.sh
./create-storage-account.sh
```

### Use Copilot

**Azure Portal Copilot**:
- Click Copilot icon in Azure portal
- Ask questions like "How do I create a storage account with Azure CLI?"
- Get command syntax, examples, and related documentation links

**Microsoft Edge Copilot**:
- Open Edge browser sidebar
- Ask Azure CLI questions
- Get aggregated information from Microsoft Learn, Stack Overflow, GitHub

**GitHub Copilot** (in VS Code):
- Install GitHub Copilot extension
- Write comments describing desired Azure CLI functionality
- Copilot suggests complete scripts

**Example GitHub Copilot Usage**:
```bash
# Comment describing intent
# Create a Bash script that deploys 5 storage accounts in East US with Standard_LRS SKU

# GitHub Copilot suggests:
#!/bin/bash
resourceGroup="rg-storage-demo"
location="eastus"
storageCount=5

for i in $(seq 1 $storageCount); do
    saName="storageacct${RANDOM}"
    az storage account create \
        --name $saName \
        --resource-group $resourceGroup \
        --location $location \
        --sku Standard_LRS
done
```

### Stack Overflow and Community

**Stack Overflow**:
- Tag: `[azure-cli]`
- Search existing questions before posting
- Include `--debug` output in questions
- Provide minimal reproducible example

**Microsoft Q&A**:
- URL: https://learn.microsoft.com/answers/
- Tag: `azure-cli`
- Official Microsoft engineer responses
- Community contributions

**GitHub Issues**:
- URL: https://github.com/Azure/azure-cli/issues
- Report bugs
- Request features
- Search existing issues first

## Performance Optimization Tips

### 1. Query Efficiently with JMESPath

**Bad** (retrieves all data, filters client-side):
```bash
# Retrieves all storage accounts, filters in terminal
az storage account list --output json | jq '.[] | select(.kind=="StorageV2") | .name'
```

**Good** (filters server-side with --query):
```bash
# Filters on Azure side, returns only matching results
az storage account list --query "[?kind=='StorageV2'].name" --output tsv
```

**Performance Impact**: Server-side filtering reduces data transferred over network by 90%+ in large environments.

### 2. Use `--output tsv` for Scripting

**Bad** (JSON requires parsing):
```bash
# Returns JSON, requires jq to extract values
storageNames=$(az storage account list --output json | jq -r '.[].name')
```

**Good** (TSV directly usable):
```bash
# Returns tab-separated values, no parsing needed
storageNames=$(az storage account list --query "[].name" --output tsv)
```

**Performance Impact**: TSV parsing 10x faster than JSON for simple value extraction.

### 3. Avoid Redundant API Calls

**Bad** (repeated calls):
```bash
for i in {1..10}; do
    # Calls API 10 times
    az storage account show --name "account$i" --resource-group myrg
done
```

**Good** (single call with filter):
```bash
# Single API call, filter results
az storage account list --resource-group myrg --query "[?starts_with(name, 'account')].{Name:name, Status:statusOfPrimary}"
```

**Performance Impact**: Reduces API calls by 90%, avoids rate limiting.

### 4. Use `--no-wait` for Long-Running Operations

**Synchronous** (blocks terminal):
```bash
# Waits for deletion to complete (can take 10+ minutes)
az group delete --name myResourceGroup --yes
```

**Asynchronous** (returns immediately):
```bash
# Returns immediately, deletion runs in background
az group delete --name myResourceGroup --yes --no-wait

# Check status later
az group show --name myResourceGroup
# Returns error if deletion completed
```

**Performance Impact**: Allows running multiple operations in parallel.

### 5. Cache Configuration

**Set Default Values** (avoids repeating parameters):
```bash
# Configure defaults
az configure --defaults group=myResourceGroup location=eastus

# Now commands use defaults
az storage account create --name mystorageacct --sku Standard_LRS
# Implicitly uses: --resource-group myResourceGroup --location eastus
```

## Integration with CI/CD Pipelines

### GitHub Actions Example

```yaml
name: Deploy Infrastructure
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Create Resource Group
        run: |
          az group create \
            --name rg-app-prod \
            --location eastus
      
      - name: Deploy Resources
        run: |
          ./infrastructure/deploy.sh
```

### Azure Pipelines Example

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: AzureCLI@2
    displayName: 'Deploy Infrastructure'
    inputs:
      azureSubscription: 'My Azure Subscription'
      scriptType: 'bash'
      scriptLocation: 'inlineScript'
      inlineScript: |
        az group create --name rg-app-prod --location eastus
        az deployment group create \
          --resource-group rg-app-prod \
          --template-file infrastructure/main.bicep
```

## Key Takeaways

1. **`--debug` Reveals All**: See command parsing, API calls, request/response details
2. **Scripting Language Matters**: Bash, PowerShell, cmd.exe have different syntax for continuation, variables, escaping
3. **Backtick Escaping**: Critical for JMESPath queries in Bash and PowerShell
4. **Multiple Help Resources**: `az find`, `--help`, documentation indexes, Copilot, Stack Overflow
5. **Performance Optimization**: Server-side filtering, TSV output, avoid redundant calls, async operations
6. **CI/CD Integration**: Native support in GitHub Actions, Azure Pipelines, Jenkins

## Next Steps

With comprehensive Azure CLI knowledge—from installation through scripting to troubleshooting—you're ready to validate your understanding. Proceed to **Unit 7: Module Assessment** to test your knowledge with scenario-based questions covering all module concepts.

---

**Module**: Create Azure Resources Using Azure CLI  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/create-azure-resources-by-using-azure-cli/6-use-azure-cli-successfully
