# Execute Azure CLI Commands Interactively

## Overview

Interactive mode allows you to execute Azure CLI commands one at a time in a terminal session, seeing immediate results after each command. This approach is ideal for learning Azure CLI syntax, troubleshooting issues, exploring Azure services, and performing one-time administrative tasks. While not suitable for repetitive operations or automation, interactive mode provides the fastest path from idea to executed command.

This unit walks through a practical example: creating an Azure Storage account. You'll learn authentication, subscription management, resource group creation, resource deployment, result verification, and cleanup—all foundational skills for Azure CLI usage.

## When to Use Interactive Mode

### Ideal Scenarios

**Learning and Exploration**:
- Testing new Azure services without writing scripts
- Discovering available commands and parameters
- Understanding command output formats
- Experimenting with JMESPath queries

**Troubleshooting**:
- Investigating resource configuration issues
- Checking resource state during incidents
- Validating permissions and access
- Diagnosing deployment failures

**One-Time Tasks**:
- Creating single resources for testing
- Quick configuration changes
- Ad-hoc administrative operations
- Validating ARM template outputs

**Rapid Prototyping**:
- Testing resource configurations before scripting
- Verifying parameter combinations
- Exploring service-specific options
- Building command sequences for later automation

### When to Avoid Interactive Mode

- **Repetitive Tasks**: Creating multiple similar resources (use scripts instead)
- **Production Deployments**: Requires repeatability and version control
- **Complex Multi-Step Processes**: Error-prone without automation
- **CI/CD Integration**: Pipelines need executable scripts
- **Audit Requirements**: Interactive commands lack comprehensive logging

## Create a Storage Account with Azure CLI

### Step 1: Connect to Azure

Before executing any Azure commands, authenticate to your Azure subscription.

#### Interactive Login

```bash
# Sign in to Azure with web browser authentication
az login
```

**Authentication Process**:
1. Command opens default web browser
2. Browser navigates to https://microsoft.com/devicelogin
3. User enters provided device code
4. User signs in with Azure credentials (email/password)
5. User completes multi-factor authentication (required starting 2025)
6. Browser displays success confirmation
7. Terminal shows list of accessible subscriptions

**Example Output**:
```json
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e",
    "id": "11111111-1111-1111-1111-111111111111",
    "isDefault": true,
    "managedByTenants": [],
    "name": "My Production Subscription",
    "state": "Enabled",
    "tenantId": "aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e",
    "user": {
      "name": "user@company.com",
      "type": "user"
    }
  }
]
```

#### Alternative Login Methods

**Device Code Flow** (for SSH sessions, VMs without browsers):
```bash
az login --use-device-code

# Output:
# To sign in, use a web browser to open https://microsoft.com/devicelogin
# and enter the code AB12-CD34 to authenticate.
```

**Service Principal** (for automation - covered in scripting unit):
```bash
az login --service-principal \
    --username $AZURE_CLIENT_ID \
    --password $AZURE_CLIENT_SECRET \
    --tenant $AZURE_TENANT_ID
```

**Tenant-Specific Login**:
```bash
# Login to specific Azure AD tenant
az login --tenant contoso.onmicrosoft.com
```

#### Multi-Factor Authentication (MFA) Requirement

**Important**: Starting in 2025, Microsoft enforces mandatory MFA for Azure CLI interactive logins. The MFA requirement impacts Microsoft Entra ID user identities but does not affect workload identities like service principals and managed identities.

**Ensure MFA is configured**:
1. Azure Portal → Azure Active Directory
2. Users → Your user account
3. Security info → Authentication methods
4. Add phone, authenticator app, or security key

### Step 2: Verify Your Subscription

After login, verify which subscription will receive created resources.

#### Show Current Default Subscription

```bash
# Display detailed information about default subscription
az account show

# Formatted as table for readability
az account show --output table
```

**Example Output** (table format):
```
EnvironmentName    HomeTenantId                          IsDefault    Name                               State    TenantId
-----------------  ------------------------------------  -----------  ---------------------------------  -------  ------------------------------------
AzureCloud         aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e  True         My Production Subscription         Enabled  aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e
```

**Key Fields**:
- **IsDefault**: Indicates which subscription receives commands
- **Name**: Human-readable subscription name
- **State**: Enabled (active) or Disabled
- **TenantId**: Azure AD tenant governing subscription

#### List All Accessible Subscriptions

```bash
# List all subscriptions your account can access
az account list --output table
```

**Example Output**:
```
Name                           SubscriptionId                        TenantId                              State    IsDefault
-----------------------------  ------------------------------------  ------------------------------------  -------  -----------
My Production Subscription     11111111-1111-1111-1111-111111111111  aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e  Enabled  True
My Development Subscription    22222222-2222-2222-2222-222222222222  aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e  Enabled  False
My Staging Subscription        33333333-3333-3333-3333-333333333333  aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e  Enabled  False
```

#### Change Default Subscription

```bash
# Set default subscription by name
az account set --subscription "My Development Subscription"

# Or by subscription ID
az account set --subscription "22222222-2222-2222-2222-222222222222"

# Verify change
az account show --query name --output tsv
# Output: My Development Subscription
```

**When to Change Subscriptions**:
- Deploying to different environments (dev vs. prod)
- Managing resources across multiple subscriptions
- Testing commands in non-production subscription first
- Following least-privilege access principles

### Step 3: Create a Resource Group

Resource groups are containers that organize and manage related Azure resources. All Azure resources must belong to a resource group.

#### List Available Azure Locations

Find which Azure regions are available to your subscription:

```bash
# List all locations with full details
az account list-locations --output table
```

**Example Output**:
```
DisplayName               Name                 RegionalDisplayName
------------------------  -------------------  -------------------------------------
East US                   eastus               (US) East US
South Central US          southcentralus       (US) South Central US
West US 2                 westus2              (US) West US 2
West US 3                 westus3              (US) West US 3
Australia East            australiaeast        (Asia Pacific) Australia East
Southeast Asia            southeastasia        (Asia Pacific) Southeast Asia
North Europe              northeurope          (Europe) North Europe
UK South                  uksouth              (Europe) UK South
West Europe               westeurope           (Europe) West Europe
Central US                centralus            (US) Central US
```

**Location Selection Considerations**:
- **Latency**: Choose regions close to users
- **Data Residency**: Compliance requirements for data location
- **Service Availability**: Not all Azure services available in all regions
- **Disaster Recovery**: Pair primary region with geographically distant secondary
- **Cost**: Pricing varies by region

#### Create Resource Group with Variables

Using variables and random identifiers enables testing scripts repeatedly without resource name conflicts:

```bash
# Variable block for reusability
let "randomIdentifier=$RANDOM*$RANDOM"
location="eastus"
resourceGroup="msdocs-rg-$randomIdentifier"

# Create resource group
az group create \
    --name $resourceGroup \
    --location $location \
    --output json
```

**Example Output**:
```json
{
  "id": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/msdocs-rg-1234567890",
  "location": "eastus",
  "managedBy": null,
  "name": "msdocs-rg-1234567890",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

**Key Fields**:
- **provisioningState**: "Succeeded" indicates successful creation
- **id**: Fully qualified Azure resource ID
- **location**: Azure region hosting metadata (resources can be in different regions)

#### Create Resource Group with Tags

Tags provide metadata for organization, cost tracking, and automation:

```bash
# Create resource group with organizational tags
az group create \
    --name rg-app-prod \
    --location eastus \
    --tags Environment=Production CostCenter=IT Owner=DevOpsTeam
```

**Common Tagging Strategies**:
```bash
# By environment
--tags Environment=Development

# By cost center
--tags CostCenter=Engineering Department=IT

# By ownership
--tags Owner=john.doe@company.com Team=Platform

# By lifecycle
--tags AutoShutdown=true BackupPolicy=Daily

# Combination
--tags Environment=Production CostCenter=Marketing Application=WebApp Tier=Frontend
```

#### Verify Resource Group Creation

```bash
# List all resource groups in subscription
az group list --output table

# Show specific resource group details
az group show --name $resourceGroup

# Check if resource group exists (useful in scripts)
if az group exists --name $resourceGroup; then
    echo "Resource group exists"
else
    echo "Resource group not found"
fi
```

### Step 4: Create a Storage Account

Storage accounts provide cloud storage for blobs, files, queues, and tables.

#### Storage Account Naming Requirements

- **Length**: 3-24 characters
- **Allowed Characters**: Lowercase letters and numbers only
- **Uniqueness**: Must be globally unique across all Azure
- **No Special Characters**: No hyphens, underscores, or uppercase letters

#### Generate Unique Storage Account Name

```bash
# Variable block
let "randomIdentifier=$RANDOM*$RANDOM"
location="eastus"
resourceGroup="msdocs-rg-0000000"  # Replace with your resource group
storageAccount="msdocssa$randomIdentifier"

# Create storage account
echo "Creating storage account $storageAccount in resource group $resourceGroup"

az storage account create \
    --name $storageAccount \
    --resource-group $resourceGroup \
    --location $location \
    --sku Standard_RAGRS \
    --kind StorageV2 \
    --output json
```

**Parameter Explanations**:

**`--name`**: Storage account name (must be globally unique)

**`--resource-group`**: Parent resource group

**`--location`**: Azure region (must be available for storage accounts)

**`--sku`**: Performance and replication tier
- `Standard_LRS`: Locally redundant storage (lowest cost)
- `Standard_GRS`: Geo-redundant storage (replicates to paired region)
- `Standard_RAGRS`: Read-access geo-redundant (GRS + read access to secondary)
- `Premium_LRS`: SSD-based, low-latency storage

**`--kind`**: Storage account type
- `StorageV2`: General-purpose v2 (recommended, supports all features)
- `Storage`: General-purpose v1 (legacy)
- `BlobStorage`: Blob-only storage
- `BlockBlobStorage`: Premium block blob storage
- `FileStorage`: Premium file storage

#### Multi-Line Commands with Backslash

The backslash (`\`) is the line continuation character in Bash, allowing long commands to span multiple lines for readability:

```bash
az storage account create \
    --name $storageAccount \           # Line 1: name parameter
    --resource-group $resourceGroup \  # Line 2: resource group
    --location $location \             # Line 3: location
    --sku Standard_RAGRS \             # Line 4: SKU
    --kind StorageV2 \                 # Line 5: kind
    --tags Environment=Dev \           # Line 6: tags
    --output json                      # Line 7: output format (no backslash on last line)
```

**Important**: No backslash on the final line.

#### Storage Account with Advanced Configuration

```bash
az storage account create \
    --name $storageAccount \
    --resource-group $resourceGroup \
    --location $location \
    --sku Standard_RAGRS \
    --kind StorageV2 \
    --access-tier Hot \                                # Hot or Cool access tier
    --https-only true \                                 # Require HTTPS
    --allow-blob-public-access false \                  # Disable anonymous blob access
    --min-tls-version TLS1_2 \                          # Minimum TLS version
    --tags Environment=Production ManagedBy=AzureCLI \
    --output json
```

**Security Best Practices**:
- `--https-only true`: Enforce encrypted connections
- `--allow-blob-public-access false`: Prevent anonymous access
- `--min-tls-version TLS1_2`: Use modern encryption protocols
- `--default-action Deny`: Restrict network access by default

#### Example Output

```json
{
  "accessTier": "Hot",
  "allowBlobPublicAccess": false,
  "creationTime": "2026-01-14T10:30:00.000000+00:00",
  "enableHttpsTrafficOnly": true,
  "id": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/msdocs-rg-1234567890/providers/Microsoft.Storage/storageAccounts/msdocssa1234567890",
  "kind": "StorageV2",
  "location": "eastus",
  "name": "msdocssa1234567890",
  "primaryEndpoints": {
    "blob": "https://msdocssa1234567890.blob.core.windows.net/",
    "file": "https://msdocssa1234567890.file.core.windows.net/",
    "queue": "https://msdocssa1234567890.queue.core.windows.net/",
    "table": "https://msdocssa1234567890.table.core.windows.net/"
  },
  "primaryLocation": "eastus",
  "provisioningState": "Succeeded",
  "resourceGroup": "msdocs-rg-1234567890",
  "sku": {
    "name": "Standard_RAGRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "tags": {
    "Environment": "Production",
    "ManagedBy": "AzureCLI"
  }
}
```

### Step 5: Verify Resource Creation

Azure CLI provides `list` and `show` commands for most resource types to verify successful deployment.

#### List All Storage Accounts

```bash
# List all storage accounts in subscription
az storage account list

# Table format for better readability
az storage account list --output table
```

**Example Output** (table format):
```
Name                  ResourceGroup          Location    Kind         ProvisioningState
--------------------  ---------------------  ----------  -----------  -------------------
msdocssa1234567890    msdocs-rg-1234567890   eastus      StorageV2    Succeeded
prodstorageacct001    rg-app-prod            westus2     StorageV2    Succeeded
devstorageacct001     rg-app-dev             centralus   StorageV2    Succeeded
```

#### Show Specific Storage Account

```bash
# Display full details of specific storage account
az storage account show \
    --name $storageAccount \
    --resource-group $resourceGroup
```

#### Filter Results with JMESPath Queries

JMESPath is a query language for JSON that enables filtering and transforming Azure CLI output.

**Example 1: Filter by Creation Date**

```bash
# Get storage accounts created in the last 30 days
saDate=$(date +%F -d "-30days")

az storage account list \
    --resource-group $resourceGroup \
    --query "[?creationTime >='$saDate'].{Name:name, ID:id, SKU:sku.name}" \
    --output table
```

**Example 2: Filter by Name Pattern**

```bash
# Get storage accounts containing "msdocs"
az storage account list \
    --resource-group $resourceGroup \
    --query "[?contains(name, 'msdocs')].{Name:name, Kind:kind, Location:primaryLocation, Created:creationTime}" \
    --output table
```

**Example Output**:
```
Name                Kind       Location    Created
------------------  ---------  ----------  --------------------------------
msdocssa1234567890  StorageV2  eastus      2026-01-14T10:30:00.000000+00:00
msdocssa9876543210  StorageV2  westus2     2026-01-10T14:22:15.000000+00:00
```

**Example 3: Extract Specific Properties**

```bash
# Get only names of storage accounts
az storage account list \
    --resource-group $resourceGroup \
    --query "[].name" \
    --output tsv

# Output (one per line):
# msdocssa1234567890
# prodstorageacct001
```

**Example 4: Complex Filtering**

```bash
# Get StorageV2 accounts with RAGRS SKU, show specific properties
az storage account list \
    --resource-group $resourceGroup \
    --query "[?kind=='StorageV2' && sku.name=='Standard_RAGRS'].{Name:name, AccessTier:accessTier, HTTPSOnly:enableHttpsTrafficOnly}" \
    --output table
```

**JMESPath Query Components**:
- `[]`: Array notation
- `?<condition>`: Filter expression
- `{}`: Object projection (select specific properties)
- `.<property>`: Property access
- `==`, `!=`, `>`, `<`, `>=`, `<=`: Comparison operators
- `&&`, `||`: Logical operators
- `contains()`, `starts_with()`, `ends_with()`: String functions

**JMESPath Learning Resources**:
- Official Tutorial: https://jmespath.org/tutorial.html
- Azure CLI Query Documentation: https://learn.microsoft.com/cli/azure/query-azure-cli

#### Check Storage Account Keys

```bash
# List access keys for storage account
az storage account keys list \
    --account-name $storageAccount \
    --resource-group $resourceGroup \
    --output table
```

**Example Output**:
```
KeyName    Permissions    Value
---------  -------------  ----------------------------------------------------------------
key1       Full           ABCDEFGabcdefg1234567890HIJKLMN+opqrstuvwxyzABCDEFG==
key2       Full           XYZabcdefg9876543210HIJKLMNopqrstuvwxyzABCDEFGHIJKLMN==
```

**Security Note**: Storage account keys provide full access. Treat them as passwords. Prefer using Managed Identity or SAS tokens for application access.

#### Verify Network and Security Settings

```bash
# Check HTTPS-only enforcement
az storage account show \
    --name $storageAccount \
    --resource-group $resourceGroup \
    --query "enableHttpsTrafficOnly"

# Check blob public access setting
az storage account show \
    --name $storageAccount \
    --resource-group $resourceGroup \
    --query "allowBlobPublicAccess"

# Check minimum TLS version
az storage account show \
    --name $storageAccount \
    --resource-group $resourceGroup \
    --query "minimumTlsVersion"
```

## Clean Up Resources

Using random identifiers creates test resources that should be cleaned up to avoid unnecessary costs.

### Delete Storage Account

```bash
# Delete specific storage account
az storage account delete \
    --name $storageAccount \
    --resource-group $resourceGroup \
    --yes  # Skip confirmation prompt
```

### Delete Resource Group (Deletes All Contained Resources)

```bash
# List resource groups to verify correct name
az group list --output table

# Delete resource group and all resources inside
az group delete \
    --name $resourceGroup \
    --yes \       # Skip confirmation prompt
    --no-wait     # Don't wait for operation to complete (runs in background)
```

**Important**: Deleting a resource group permanently removes all resources inside it. This action cannot be undone. Always verify the resource group name before deletion.

### Verify Deletion

```bash
# Check if resource group still exists
az group exists --name $resourceGroup
# Output: false (if deleted successfully)

# Attempt to show deleted resource group (should fail)
az group show --name $resourceGroup
# Error: ResourceGroupNotFound
```

### Background Deletion with `--no-wait`

The `--no-wait` parameter allows deletion operations to run in the background, freeing up the terminal for other tasks:

```bash
# Start deletion in background
az group delete --name $resourceGroup --yes --no-wait

# Continue using terminal immediately
az account show
az group list

# Check deletion status later
az group show --name $resourceGroup
# Error indicates deletion completed
```

## Output Format Options

Azure CLI supports multiple output formats for different use cases.

### JSON (Default)

Comprehensive, machine-readable format:

```bash
az storage account show --name $storageAccount --resource-group $resourceGroup --output json
```

**Use When**:
- Parsing output with jq or Python
- Debugging (shows all properties)
- Saving complete resource information

### Table

Human-readable columnar format:

```bash
az storage account list --output table
```

**Use When**:
- Quick visual inspection
- Terminal display for humans
- Comparing multiple resources

### TSV (Tab-Separated Values)

Minimal format for scripting:

```bash
az storage account list --query "[].name" --output tsv
```

**Use When**:
- Piping to other commands
- Bash loops
- Extracting single values

### YAML

Human-readable structured format:

```bash
az storage account show --name $storageAccount --resource-group $resourceGroup --output yaml
```

**Use When**:
- Readable structured data
- Configuration file generation
- Documentation

### Setting Default Output Format

```bash
# Set default output to table
az configure --defaults output=table

# All commands now default to table format
az storage account list
```

## Interactive Command Best Practices

### 1. Use Variables for Repeatability

```bash
# Define once, reuse multiple times
storageAccount="mystorageacct001"
resourceGroup="rg-app-prod"

az storage account show --name $storageAccount --resource-group $resourceGroup
az storage account keys list --account-name $storageAccount --resource-group $resourceGroup
az storage blob list --account-name $storageAccount --container-name mycontainer
```

### 2. Test in Non-Production First

```bash
# Test command in dev subscription
az account set --subscription "My Development Subscription"
az storage account create --name teststorageacct --resource-group rg-dev --location eastus --sku Standard_LRS

# After verifying success, run in production
az account set --subscription "My Production Subscription"
az storage account create --name prodstorageacct --resource-group rg-prod --location eastus --sku Standard_RAGRS
```

### 3. Use `--help` Liberally

```bash
# Check available parameters
az storage account create --help

# Understand command behavior
az storage account delete --help
```

### 4. Verify Before Deleting

```bash
# Always list resources before deletion
az resource list --resource-group $resourceGroup --output table

# Confirm resource group name
echo "About to delete: $resourceGroup"
az group delete --name $resourceGroup --yes
```

### 5. Save Command History

```bash
# Bash history saved to ~/.bash_history
history | grep "az storage"

# Search history interactively
Ctrl+R, then type "az group create"
```

## Real-World Interactive Scenarios

### Scenario 1: Troubleshooting Application Connectivity

```bash
# Check if storage account exists
az storage account show --name prodstorageacct --resource-group rg-app-prod

# Verify network rules
az storage account show --name prodstorageacct --resource-group rg-app-prod --query "networkRuleSet"

# Check if application's IP is allowed
az storage account network-rule list --account-name prodstorageacct --resource-group rg-app-prod

# Temporarily allow all networks for testing
az storage account update --name prodstorageacct --resource-group rg-app-prod --default-action Allow
```

### Scenario 2: Validating Resource Configuration

```bash
# Verify all production resources have correct tags
az resource list --resource-group rg-app-prod --query "[?tags.Environment != 'Production'].{Name:name, Tags:tags}" --output table

# Check if HTTPS-only is enforced on all storage accounts
az storage account list --resource-group rg-app-prod --query "[?enableHttpsTrafficOnly == \`false\`].name"
```

### Scenario 3: Quick Cost Investigation

```bash
# List expensive SKUs in use
az storage account list --query "[?sku.name == 'Premium_LRS'].{Name:name, ResourceGroup:resourceGroup, SKU:sku.name}" --output table

# Find large storage accounts
az storage account list --query "[].{Name:name, Location:location}" --output tsv | while read name location; do
    echo "$name: $(az storage account show --name $name --query 'primaryEndpoints' -o json)"
done
```

## Key Takeaways

1. **Interactive Mode Strengths**: Learning, troubleshooting, one-time tasks, exploration
2. **Authentication Required**: Always start with `az login` (MFA required starting 2025)
3. **Subscription Management**: Verify and set default subscription with `az account`
4. **Resource Groups**: Containers for organizing related resources
5. **Unique Naming**: Storage accounts require globally unique names (lowercase, alphanumeric, 3-24 chars)
6. **Verification**: Use `list` and `show` commands to confirm resource creation
7. **JMESPath Queries**: Powerful filtering for complex output
8. **Cleanup**: Delete resource groups to remove all contained resources
9. **Output Formats**: JSON, table, TSV, YAML for different use cases

## Next Steps

Interactive commands are perfect for learning and exploration, but repetitive tasks require automation. Proceed to **Unit 5: Create an Azure CLI Script for Bash** to learn how to combine multiple commands into reusable, error-handled scripts for production deployments.

---

**Module**: Create Azure Resources Using Azure CLI  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/create-azure-resources-by-using-azure-cli/4-execute-azure-cli-interactively
