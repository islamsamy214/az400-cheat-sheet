# Module Assessment

## Overview

This knowledge check validates your understanding of Azure CLI concepts, commands, scripting, and best practices covered throughout this module. Questions cover installation, authentication, command structure, scripting techniques, troubleshooting, and real-world application scenarios.

Review previous units if needed before attempting these questions. Explanations are provided for each answer to reinforce learning.

## Question 1: Azure CLI Installation Methods

**Scenario**: Your team needs to deploy Azure CLI across different environments: Windows desktops, Linux servers, and a CI/CD pipeline.

**Question**: Which installation method is MOST appropriate for each environment?

**A)** Windows: MSI installer, Linux: Docker container, CI/CD: Azure Cloud Shell  
**B)** Windows: MSI installer, Linux: Package manager (apt/yum), CI/CD: Docker container  
**C)** Windows: Azure Cloud Shell, Linux: Script installation, CI/CD: Package manager  
**D)** Windows: Docker container, Linux: MSI installer, CI/CD: Script installation

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: B**

**Explanation**:
- **Windows desktops**: MSI installer provides native Windows integration, GUI installation, and automatic PATH configuration
- **Linux servers**: Package manager (apt for Debian/Ubuntu, yum for RHEL/CentOS) enables automatic updates through system package management
- **CI/CD pipelines**: Docker containers provide isolated, version-pinned environments ensuring consistency across pipeline runs

**Why other options are incorrect**:
- **Option A**: Azure Cloud Shell requires internet and has session timeouts (20 minutes idle), unsuitable for automated CI/CD
- **Option C**: Azure Cloud Shell isn't an "installation"—it's a cloud-hosted environment
- **Option D**: Docker containers on Windows desktops add unnecessary complexity; MSI installer isn't available for Linux

**Key Takeaway**: Match installation method to use case—native installers for desktops, package managers for servers, containers for CI/CD.

</details>

---

## Question 2: Authentication Methods

**Scenario**: You're implementing Azure CLI automation across three scenarios:
1. Local development on your workstation
2. Automated nightly deployment script in Azure Pipelines
3. Azure VM running scheduled backup scripts

**Question**: Which authentication method is MOST appropriate for each scenario?

**A)** 1) Interactive login, 2) Service principal, 3) Managed identity  
**B)** 1) Service principal, 2) Interactive login, 3) Service principal  
**C)** 1) Managed identity, 2) Managed identity, 3) Interactive login  
**D)** 1) Interactive login, 2) Managed identity, 3) Service principal

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: A**

**Explanation**:
1. **Local development**: Interactive login (`az login`) with MFA provides secure, user-context authentication for developers
2. **Azure Pipelines**: Service principal with stored credentials (client ID, secret, tenant) enables unattended automation
3. **Azure VM**: Managed identity eliminates credential management—VM authenticates using Azure AD-assigned identity

**Why other options are incorrect**:
- **Option B**: Interactive login in CI/CD requires human intervention (impossible for automation)
- **Option C**: Managed identity doesn't work on local workstations (only Azure-hosted resources)
- **Option D**: Managed identity can't be used in Azure Pipelines (runs on hosted agents, not Azure VMs with managed identity)

**Key Takeaway**: Authentication method must match execution environment and automation requirements.

**Best Practices**:
```bash
# Local development
az login

# Azure Pipelines (service principal from environment variables)
az login --service-principal \
    --username $AZURE_CLIENT_ID \
    --password $AZURE_CLIENT_SECRET \
    --tenant $AZURE_TENANT_ID

# Azure VM (managed identity)
az login --identity
```

</details>

---

## Question 3: Command Structure

**Question**: Given the command `az storage account create --name mystorageacct --resource-group myrg --location eastus --sku Standard_LRS`, identify the components:

**A)** Reference group: `storage`, Subgroup: `account`, Command: `create`, Parameters: everything after `create`  
**B)** Reference group: `az`, Subgroup: `storage account`, Command: `create`, Parameters: `--name`, `--resource-group`, `--location`, `--sku`  
**C)** Reference group: `storage account`, Command: `create`, Parameters: `mystorageacct`, `myrg`, `eastus`, `Standard_LRS`  
**D)** Reference group: `az storage`, Subgroup: `account create`, Parameters: all `--` prefixed arguments

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: A**

**Explanation**:

**Command Structure**: `az <reference-group> <subgroup> <command> --<parameter> <value>`

**Breakdown**:
- **`az`**: Root command (invokes Azure CLI)
- **Reference group**: `storage` (top-level organizational unit for Azure Storage commands)
- **Subgroup**: `account` (nested unit for storage account operations)
- **Command**: `create` (action to perform)
- **Parameters**: `--name mystorageacct`, `--resource-group myrg`, `--location eastus`, `--sku Standard_LRS`

**Why other options are incorrect**:
- **Option B**: `az` is the root command, not a reference group
- **Option C**: `storage account` combined isn't a single reference group; parameters are key-value pairs, not just values
- **Option D**: Incorrect grouping of reference group and subgroup

**Key Takeaway**: Understanding command structure helps discover new commands using `az <group> <subgroup> --help`.

</details>

---

## Question 4: JMESPath Query Syntax

**Scenario**: You need to find all StorageV2 accounts in a resource group that have HTTPS-only enforcement enabled.

**Question**: Which Azure CLI command accomplishes this?

**A)** `az storage account list --resource-group myrg --query "[?kind=='StorageV2' && enableHttpsTrafficOnly==\`true\`].name"`  
**B)** `az storage account list --resource-group myrg --query "[?kind=='StorageV2' && enableHttpsTrafficOnly=='true'].name"`  
**C)** `az storage account list --resource-group myrg --query "[kind=='StorageV2' && enableHttpsTrafficOnly==true].name"`  
**D)** `az storage account list --resource-group myrg --filter "kind eq 'StorageV2' and enableHttpsTrafficOnly eq true"`

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: A**

**Explanation**:

**Correct syntax**:
```bash
az storage account list --resource-group myrg --query "[?kind=='StorageV2' && enableHttpsTrafficOnly==\`true\`].name"
```

**JMESPath Query Components**:
- **`[]`**: Array notation (list of storage accounts)
- **`?`**: Filter expression indicator
- **`kind=='StorageV2'`**: Filter condition (string comparison with single quotes)
- **`&&`**: Logical AND operator
- **`enableHttpsTrafficOnly==\`true\``**: Boolean comparison (backticks escaped for Bash)
- **`.name`**: Property projection (extract only name field)

**Why other options are incorrect**:
- **Option B**: Boolean value `'true'` as string instead of boolean literal (will always return no results)
- **Option C**: Missing `?` filter indicator and `[]` array notation
- **Option D**: `--filter` parameter doesn't exist in Azure CLI; uses JMESPath `--query`, not OData filters

**PowerShell Equivalent** (double backticks):
```powershell
az storage account list --resource-group myrg --query "[?kind=='StorageV2' && enableHttpsTrafficOnly==``true``].name"
```

**Key Takeaway**: JMESPath requires proper syntax: `[?condition]` for filtering, escaped backticks for boolean/number literals in Bash.

</details>

---

## Question 5: Scripting Best Practices

**Scenario**: You're writing a Bash script to deploy production infrastructure. Which approach follows best practices?

**A)** 
```bash
#!/bin/bash
az group create --name rg-prod --location eastus
az storage account create --name stprod --resource-group rg-prod --location eastus
```

**B)**
```bash
#!/bin/bash
set -e
if ! az account show &> /dev/null; then
    echo "Not authenticated"
    exit 1
fi
az group create --name rg-prod --location eastus || exit 1
az storage account create --name stprod --resource-group rg-prod --location eastus || exit 1
```

**C)**
```bash
#!/bin/bash
set -e
set -u
set -o pipefail

if ! az account show &> /dev/null; then
    echo "Error: Not authenticated to Azure"
    exit 1
fi

if ! az group show --name rg-prod &> /dev/null; then
    az group create --name rg-prod --location eastus
fi

az storage account create --name stprod --resource-group rg-prod --location eastus || {
    echo "Error: Failed to create storage account"
    exit 1
}
```

**D)**
```bash
az group create --name rg-prod --location eastus
az storage account create --name stprod --resource-group rg-prod --location eastus
echo "Done"
```

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: C**

**Explanation**:

**Option C demonstrates production best practices**:

1. **Strict Error Handling**:
   ```bash
   set -e          # Exit on error
   set -u          # Exit on undefined variable
   set -o pipefail # Fail if any command in pipe fails
   ```

2. **Authentication Validation**:
   ```bash
   if ! az account show &> /dev/null; then
       echo "Error: Not authenticated to Azure"
       exit 1
   fi
   ```
   - Prevents script execution without Azure authentication
   - Fails fast with clear error message

3. **Idempotency Check**:
   ```bash
   if ! az group show --name rg-prod &> /dev/null; then
       az group create --name rg-prod --location eastus
   fi
   ```
   - Checks if resource group exists before creating
   - Script can run multiple times safely

4. **Explicit Error Handling**:
   ```bash
   az storage account create ... || {
       echo "Error: Failed to create storage account"
       exit 1
   }
   ```
   - Provides context-specific error messages
   - Ensures script exits on failure

**Why other options are insufficient**:
- **Option A**: No error handling, authentication check, or idempotency—fails silently
- **Option B**: Has error handling but lacks idempotency checks; `set -u` and `set -o pipefail` missing
- **Option D**: No shebang, error handling, or authentication check—not a proper script

**Additional Production Enhancements**:
- Logging to file
- Rollback on failure
- Parameter validation
- Service principal authentication for automation

**Key Takeaway**: Production scripts require comprehensive error handling, authentication validation, and idempotency.

</details>

---

## Question 6: Debugging with `--debug`

**Scenario**: This Bash command fails with "invalid jmespath_type value":

```bash
resourceGroup="myrg"
az storage account list --resource-group $resourceGroup --query "[?kind==`StorageV2`].name"
```

**Question**: Using `--debug`, you discover the command arguments show: `--query '[?kind==].name'`. What's the problem and solution?

**A)** Problem: Variable $resourceGroup not expanding. Solution: Use double quotes around query  
**B)** Problem: Bash interpreting backticks as command substitution. Solution: Escape backticks: `\`StorageV2\``  
**C)** Problem: JMESPath syntax error. Solution: Use double quotes: `"StorageV2"`  
**D)** Problem: Wrong operator. Solution: Use `!=` instead of `==`

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: B**

**Explanation**:

**Problem Analysis**:
- Debug output shows: `--query '[?kind==].name'`
- The value `StorageV2` disappeared
- Bash interpreted backticks (`) as command substitution operators
- Bash tried to execute `StorageV2` as a command, which failed/returned nothing

**Solution** - Escape Backticks in Bash:
```bash
az storage account list --resource-group $resourceGroup --query "[?kind==\`StorageV2\`].name"
```

**How Escaping Works**:
- Backslash (`\`) tells Bash to treat next character literally
- `\`` prevents Bash from interpreting ` as command substitution
- Azure CLI receives intact query: `[?kind==`StorageV2`].name`

**Platform Variations**:

**Bash**:
```bash
--query "[?kind==\`StorageV2\`].name"
```

**PowerShell** (double backticks):
```powershell
--query "[?kind==``StorageV2``].name"
```

**cmd.exe** (no backticks needed):
```batch
--query "[?kind=='StorageV2'].name"
```

**Why other options are incorrect**:
- **Option A**: Variable expansion works fine; problem is backtick interpretation
- **Option C**: Double quotes for string values in JMESPath use single quotes: `'StorageV2'`, but backticks indicate boolean/number literals
- **Option D**: Operator is correct; problem is value disappeared

**Key Takeaway**: `--debug` reveals how the scripting language parsed command arguments before Azure CLI received them. Always check "Command arguments" line when troubleshooting.

</details>

---

## Question 7: Performance Optimization

**Scenario**: You need to get names of all Standard_LRS storage accounts across 50 resource groups (500+ storage accounts total).

**Question**: Which approach is MOST efficient?

**A)** 
```bash
for rg in $(az group list --query "[].name" -o tsv); do
    az storage account list --resource-group $rg --output json | jq '.[] | select(.sku.name=="Standard_LRS") | .name'
done
```

**B)**
```bash
for rg in $(az group list --query "[].name" -o tsv); do
    az storage account list --resource-group $rg --query "[?sku.name=='Standard_LRS'].name" --output tsv
done
```

**C)**
```bash
az storage account list --query "[?sku.name=='Standard_LRS'].name" --output tsv
```

**D)**
```bash
az resource list --resource-type "Microsoft.Storage/storageAccounts" --query "[?sku.name=='Standard_LRS'].name" --output table
```

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: C**

**Explanation**:

**Option C** is most efficient:
```bash
az storage account list --query "[?sku.name=='Standard_LRS'].name" --output tsv
```

**Efficiency Analysis**:
- **Single API call**: Lists all storage accounts across subscription
- **Server-side filtering**: `--query` filters at Azure Resource Manager level
- **Minimal data transfer**: Returns only matching names (TSV format)
- **Fast execution**: ~2-3 seconds for 500 accounts

**Why other options are less efficient**:

**Option A** (Least Efficient):
- **51 API calls**: 1 for resource groups + 50 for storage accounts per group
- **Client-side filtering**: Downloads full JSON, filters with jq locally
- **Maximum data transfer**: Full resource properties downloaded
- **Slow execution**: ~30-60 seconds
- **Rate limiting risk**: Many rapid API calls

**Option B** (Moderately Efficient):
- **51 API calls**: Still loops through each resource group
- **Server-side filtering**: Better than Option A (--query instead of jq)
- **Moderate data transfer**: Returns only names, but across 50 calls
- **Moderate execution**: ~15-20 seconds

**Option D** (Almost Optimal):
- **Single API call**: Good performance
- **Generic resource API**: Slightly slower than specialized `az storage account list`
- **Table output**: Adds formatting overhead vs. TSV

**Performance Comparison**:
| Option | API Calls | Execution Time | Data Transfer |
|--------|-----------|---------------|---------------|
| A | 51 | 60s | High (full JSON × 50) |
| B | 51 | 20s | Medium (TSV × 50) |
| **C** | **1** | **3s** | **Low (TSV)** |
| D | 1 | 5s | Low-Medium (table) |

**Key Takeaways**:
1. **Minimize API calls**: One call across subscription beats 50 per-resource-group calls
2. **Server-side filtering**: `--query` parameter filters at Azure, reducing data transfer
3. **Use TSV for scripting**: Fastest parsing for simple value extraction
4. **Avoid loops when possible**: Use subscription-wide queries instead

</details>

---

## Question 8: Cross-Platform Scripts

**Question**: You need to create an Azure CLI script that works on Windows (PowerShell), Linux (Bash), and macOS (Zsh). Which strategy is BEST?

**A)** Write one script with conditional logic detecting platform and adjusting syntax  
**B)** Write separate scripts for each platform optimized for that environment  
**C)** Write all commands in Azure Cloud Shell Bash and document platform conversion steps  
**D)** Use only single-line commands without variables or line continuations

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: B**

**Explanation**:

**Option B** (Best Practice):
- **Platform-specific optimization**: Each script uses native syntax (Bash backslash, PowerShell backtick)
- **Clear documentation**: README explains which script for which platform
- **Maintainability**: Changes in one script don't affect others
- **Testing**: Each script tested in target environment
- **No runtime detection overhead**: No conditional logic complexity

**Example Structure**:
```
infrastructure/
├── deploy.sh           # Bash (Linux, macOS, WSL)
├── deploy.ps1          # PowerShell (Windows, cross-platform)
├── deploy.bat          # cmd.exe (legacy Windows)
└── README.md           # Platform selection guidance
```

**Why other options are suboptimal**:

**Option A** (Overly Complex):
```bash
# Complex, error-prone platform detection
if [[ "$OSTYPE" == "linux-gnu"* ]]; then
    # Linux syntax
elif [[ "$OSTYPE" == "darwin"* ]]; then
    # macOS syntax
elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "win32" ]]; then
    # Windows syntax
fi
```
- Hard to maintain
- Difficult to test all branches
- Single point of failure

**Option C** (Documentation Burden):
- Requires manual conversion by users
- Error-prone (users forget to convert syntax)
- Poor user experience
- Not suitable for automation

**Option D** (Severely Limited):
- No variables = constant repetition
- No line continuations = unreadable long commands
- No flexibility for complex deployments
- Not practical for real-world use

**When to Use Each Strategy**:
- **Separate scripts (B)**: Default choice for production
- **Single Bash script (C)**: Team standardized on Linux/Azure Cloud Shell
- **Platform detection (A)**: Advanced teams with unified tooling
- **Simple commands (D)**: Only for trivial, one-off tasks

**Key Takeaway**: Platform-specific scripts provide best balance of maintainability, performance, and user experience.

</details>

---

## Question 9: Resource Cleanup

**Scenario**: Your CI/CD pipeline creates temporary test resource groups (named `rg-test-<timestamp>`). After tests complete, cleanup should delete these resource groups.

**Question**: Which cleanup script follows best practices?

**A)**
```bash
az group list --query "[?starts_with(name, 'rg-test-')].name" -o tsv | xargs -I {} az group delete --name {} --yes
```

**B)**
```bash
for rg in $(az group list --query "[?starts_with(name, 'rg-test-')].name" -o tsv); do
    az group delete --name $rg --yes --no-wait
done
```

**C)**
```bash
matchingGroups=$(az group list --query "[?starts_with(name, 'rg-test-')].name" -o tsv)
if [ -z "$matchingGroups" ]; then
    echo "No test resource groups found"
    exit 0
fi

echo "Resource groups to delete:"
echo "$matchingGroups"

for rgName in $matchingGroups; do
    echo "Deleting $rgName..."
    az group delete --name $rgName --yes --no-wait
done

echo "Deletion initiated for all test resource groups"
```

**D)**
```bash
az group delete --name rg-test-* --yes
```

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: C**

**Explanation**:

**Option C demonstrates cleanup best practices**:

1. **Check for Matching Resources**:
   ```bash
   matchingGroups=$(az group list --query "[?starts_with(name, 'rg-test-')].name" -o tsv)
   if [ -z "$matchingGroups" ]; then
       echo "No test resource groups found"
       exit 0
   fi
   ```
   - Gracefully handles case where no resources match
   - Avoids errors in CI/CD pipeline when no cleanup needed

2. **Show What Will Be Deleted**:
   ```bash
   echo "Resource groups to delete:"
   echo "$matchingGroups"
   ```
   - Provides audit trail in pipeline logs
   - Helps troubleshoot if wrong resources deleted

3. **Async Deletion**:
   ```bash
   az group delete --name $rgName --yes --no-wait
   ```
   - `--yes`: Skips confirmation prompts (required for automation)
   - `--no-wait`: Returns immediately, deletion runs in background
   - Pipeline doesn't wait 10+ minutes for deletion

4. **Clear Logging**:
   - Progress messages for each deletion
   - Final confirmation message

**Why other options are suboptimal**:

**Option A**:
```bash
az group list ... | xargs -I {} az group delete --name {} --yes
```
**Problems**:
- No check if resources exist (errors on empty result)
- Synchronous deletion (blocks pipeline)
- No logging of what's being deleted
- `xargs` less readable than loops

**Option B**:
- Better than A (uses `--no-wait`)
- Missing: empty result check, progress logging
- Acceptable for simple scripts, insufficient for production

**Option D**:
```bash
az group delete --name rg-test-* --yes
```
**Problems**:
- Azure CLI doesn't support wildcards in `--name`
- Command will fail
- Not valid syntax

**Enhanced Version with Retry Logic**:
```bash
matchingGroups=$(az group list --query "[?starts_with(name, 'rg-test-')].name" -o tsv)

if [ -z "$matchingGroups" ]; then
    echo "No test resource groups found"
    exit 0
fi

for rgName in $matchingGroups; do
    echo "Deleting $rgName..."
    retries=3
    count=0
    
    while [ $count -lt $retries ]; do
        if az group delete --name $rgName --yes --no-wait 2>/dev/null; then
            echo "✓ Deletion initiated for $rgName"
            break
        else
            ((count++))
            echo "Retry $count/$retries for $rgName"
            sleep 5
        fi
    done
done
```

**Key Takeaway**: Production cleanup scripts need error handling, logging, and async operations for CI/CD integration.

</details>

---

## Question 10: Real-World Scenario

**Scenario**: Your company deploys applications to three Azure subscriptions (Dev, Staging, Prod). Each deployment:
1. Creates a resource group
2. Deploys 5 storage accounts with unique names
3. Tags resources with environment and cost center
4. Validates deployment succeeded

The deployment runs in Azure Pipelines using a service principal with different permissions in each subscription.

**Question**: Which script structure BEST meets these requirements?

**A)** Single script with subscription ID as parameter, uses interactive login  
**B)** Separate scripts per environment, uses environment-specific credentials, validates each step  
**C)** Single parameterized script accepting environment name, uses service principal from environment variables, comprehensive error handling  
**D)** Three identical scripts with hardcoded values, no error handling

<details>
<summary><strong>Click to reveal answer</strong></summary>

**Correct Answer: C**

**Explanation**:

**Option C** (Production-Ready Pattern):

```bash
#!/bin/bash
# deploy-application.sh
# Usage: ./deploy-application.sh <environment>

set -e
set -u
set -o pipefail

# Accept environment parameter
environment=$1

# Validate environment
case $environment in
    dev|staging|prod)
        echo "Deploying to $environment"
        ;;
    *)
        echo "Error: Environment must be dev, staging, or prod"
        exit 1
        ;;
esac

# Authenticate with service principal (from pipeline environment variables)
echo "Authenticating to Azure..."
az login --service-principal \
    --username $AZURE_CLIENT_ID \
    --password $AZURE_CLIENT_SECRET \
    --tenant $AZURE_TENANT_ID

# Set subscription based on environment
case $environment in
    dev)
        az account set --subscription $DEV_SUBSCRIPTION_ID
        costCenter="Engineering"
        ;;
    staging)
        az account set --subscription $STAGING_SUBSCRIPTION_ID
        costCenter="Engineering"
        ;;
    prod)
        az account set --subscription $PROD_SUBSCRIPTION_ID
        costCenter="Operations"
        ;;
esac

# Verify subscription
currentSub=$(az account show --query name -o tsv)
echo "Using subscription: $currentSub"

# Create resource group
resourceGroup="rg-app-$environment"
location="eastus"

echo "Creating resource group $resourceGroup..."
if az group create --name $resourceGroup --location $location \
    --tags Environment=$environment CostCenter=$costCenter; then
    echo "✓ Resource group created"
else
    echo "✗ Failed to create resource group"
    exit 1
fi

# Deploy storage accounts
echo "Deploying storage accounts..."
for i in {1..5}; do
    timestamp=$(date +%s)
    saName="stapp${environment}${timestamp}${RANDOM}"
    saName=$(echo $saName | tr '[:upper:]' '[:lower:]' | cut -c1-24)
    
    echo "Creating storage account $saName..."
    if az storage account create \
        --name $saName \
        --resource-group $resourceGroup \
        --location $location \
        --sku Standard_LRS \
        --tags Environment=$environment CostCenter=$costCenter \
        --output none; then
        echo "✓ Created $saName"
    else
        echo "✗ Failed to create $saName"
        exit 1
    fi
done

# Validate deployment
echo "Validating deployment..."
deployedCount=$(az storage account list --resource-group $resourceGroup --query "length([])" -o tsv)

if [ "$deployedCount" -eq 5 ]; then
    echo "✓ Deployment validation passed: $deployedCount storage accounts"
else
    echo "✗ Deployment validation failed: Expected 5, found $deployedCount"
    exit 1
fi

echo "Deployment to $environment completed successfully"
```

**Why Option C is Best**:

1. **Single Parameterized Script**:
   - One script handles all environments
   - Reduces maintenance burden
   - Consistent logic across environments
   - DRY (Don't Repeat Yourself) principle

2. **Service Principal Authentication**:
   - Uses environment variables from Azure Pipelines
   - No hardcoded credentials
   - Secure authentication for automation
   - Different permissions per subscription

3. **Environment-Specific Configuration**:
   - `case` statement maps environment to subscription
   - Environment-specific cost centers
   - Flexible for future additions (e.g., test environment)

4. **Comprehensive Error Handling**:
   - `set -e`: Exit on error
   - `set -u`: Exit on undefined variable
   - `set -o pipefail`: Catch pipe failures
   - Validates each step (resource group, storage accounts)
   - Final deployment validation

5. **Unique Name Generation**:
   - Timestamp + random number ensures uniqueness
   - Lowercase conversion (Azure requirement)
   - Character limit enforcement (24 chars)

6. **Pipeline Integration**:
   ```yaml
   # Azure Pipelines YAML
   - task: AzureCLI@2
     displayName: 'Deploy to Production'
     inputs:
       azureSubscription: 'Production Service Principal'
       scriptType: 'bash'
       scriptLocation: 'scriptPath'
       scriptPath: './deploy-application.sh'
       arguments: 'prod'
     env:
       AZURE_CLIENT_ID: $(PROD_CLIENT_ID)
       AZURE_CLIENT_SECRET: $(PROD_CLIENT_SECRET)
       AZURE_TENANT_ID: $(TENANT_ID)
       PROD_SUBSCRIPTION_ID: $(PROD_SUB_ID)
   ```

**Why other options are incorrect**:

**Option A**:
- Interactive login doesn't work in automated pipelines
- Requires human intervention
- Not suitable for CI/CD

**Option B**:
- Three separate scripts = tripled maintenance
- Code duplication
- Risk of environment drift (scripts become inconsistent)
- Acceptable only if environments drastically differ

**Option D**:
- Hardcoded values = security risk
- No error handling = silent failures
- Not production-ready
- Maintenance nightmare

**Key Takeaway**: Production deployment scripts should be parameterized, use secure authentication (service principals), validate each step, and handle errors gracefully.

</details>

---

## Summary

This assessment covered critical Azure CLI concepts:

1. **Installation**: Platform-appropriate methods (MSI, package managers, Docker, Cloud Shell)
2. **Authentication**: Interactive login, service principals, managed identity based on use case
3. **Command Structure**: Understanding reference groups, subgroups, commands, parameters
4. **JMESPath Queries**: Server-side filtering with proper syntax and escaping
5. **Scripting Best Practices**: Error handling, idempotency, authentication validation
6. **Debugging**: Using `--debug` to troubleshoot argument parsing and API calls
7. **Performance**: Minimizing API calls, server-side filtering, async operations
8. **Cross-Platform**: Platform-specific scripts for best user experience
9. **Resource Cleanup**: Safe, logged, async deletion with error handling
10. **Real-World Integration**: Production-ready scripts with service principal auth, validation, CI/CD integration

## Next Steps

Congratulations on completing the assessment! Proceed to **Unit 8: Summary** to review key takeaways and explore additional learning resources for continued Azure CLI mastery.

---

**Module**: Create Azure Resources Using Azure CLI  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/create-azure-resources-by-using-azure-cli/7-knowledge-check
