# Create an Azure CLI Script for Bash

## Overview

Azure CLI scripting transforms interactive commands into automated, repeatable infrastructure deployment workflows. While executing commands one-by-one teaches syntax and explores services, production environments demand consistency, error handling, and version control. Bash scripts combine Azure CLI commands with programming constructs—variables, loops, conditionals, error handling—to create sophisticated automation suitable for CI/CD pipelines and enterprise infrastructure management.

This unit progresses from basic script structure through production-ready patterns, demonstrating resource deployment scripts, parameterized configurations, configuration-driven automation, resource cleanup, and enterprise best practices.

## Understanding Azure CLI Scripts

### Why Script Azure CLI Commands?

**Automation Benefits**:
- **Consistency**: Identical deployments across dev, staging, production
- **Speed**: Minutes instead of hours for complex infrastructure
- **Repeatability**: Same script produces identical results
- **Error Reduction**: Eliminates manual typos and configuration mistakes
- **Scale**: Deploy 100 resources as easily as deploying 1

**Infrastructure as Code (IaC) Advantages**:
- **Version Control**: Track infrastructure changes in Git
- **Code Reviews**: Apply software engineering practices to infrastructure
- **Audit Trail**: Complete history of who changed what and when
- **Collaboration**: Teams share and improve infrastructure code
- **Rollback**: Revert to previous infrastructure states

**DevOps Integration**:
- **CI/CD Pipelines**: Automated deployment in Azure Pipelines, GitHub Actions, Jenkins
- **Environment Parity**: Dev environments identical to production
- **Testing**: Create test resources, run tests, destroy automatically
- **Documentation**: Scripts serve as executable documentation

### When to Use Scripts vs. Templates

| Scenario | Azure CLI Scripts | ARM/Bicep Templates |
|----------|------------------|-------------------|
| **Quick prototyping** | ✅ Excellent | ❌ Too verbose |
| **Imperative control** | ✅ Full control over order | ❌ Declarative only |
| **Conditional logic** | ✅ Bash if/else/case | ⚠️ Limited (ARM functions) |
| **Loops for bulk creation** | ✅ Native Bash loops | ⚠️ Copy resource with count |
| **Error handling** | ✅ Custom logic | ⚠️ Limited |
| **Idempotency** | ⚠️ Manual implementation | ✅ Built-in |
| **Azure portal integration** | ❌ No GUI | ✅ Deploy button support |
| **Declarative infra** | ❌ Not declarative | ✅ Ideal |
| **Complex dependencies** | ⚠️ Manual orchestration | ✅ Automatic dependency resolution |

**Best Practice**: Use both strategically:
- CLI scripts for operational tasks, conditional deployment, environment-specific logic
- ARM/Bicep templates for complex infrastructure with dependencies
- Hybrid: CLI scripts deploying ARM/Bicep templates

## Script Structure Basics

### Minimal Script Structure

```bash
#!/bin/bash
# Script: minimal-example.sh
# Description: Basic Azure CLI script structure

# Exit on error
set -e

# Variables
resourceGroup="rg-demo"
location="eastus"

# Authentication check
if ! az account show &> /dev/null; then
    echo "Error: Not authenticated. Run 'az login' first."
    exit 1
fi

# Create resource group
echo "Creating resource group $resourceGroup..."
az group create --name $resourceGroup --location $location

echo "Script complete!"
```

### Well-Structured Production Script

```bash
#!/bin/bash
# Script: production-deployment.sh
# Description: Deploy application infrastructure with error handling
# Usage: ./production-deployment.sh <environment> <location>
# Example: ./production-deployment.sh prod eastus

#===========================================
# CONFIGURATION
#===========================================

# Strict error handling
set -e          # Exit on error
set -u          # Exit on undefined variable
set -o pipefail # Exit on pipe failure

# Script metadata
readonly SCRIPT_NAME=$(basename "$0")
readonly SCRIPT_DIR=$(cd "$(dirname "$0")" && pwd)
readonly TIMESTAMP=$(date +"%Y%m%d-%H%M%S")
readonly LOG_FILE="deployment-${TIMESTAMP}.log"

#===========================================
# FUNCTIONS
#===========================================

# Logging function
log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    echo "[$timestamp] [$level] $message" | tee -a "$LOG_FILE"
}

# Error handler
error_exit() {
    log "ERROR" "$1"
    exit 1
}

# Cleanup function
cleanup() {
    log "INFO" "Performing cleanup..."
    # Add cleanup logic here
}

# Trap errors and exit signals
trap cleanup EXIT
trap 'error_exit "Script interrupted"' INT TERM

#===========================================
# PARAMETER VALIDATION
#===========================================

if [ $# -ne 2 ]; then
    error_exit "Usage: $SCRIPT_NAME <environment> <location>"
fi

environment=$1
location=$2

# Validate environment
case $environment in
    dev|staging|prod)
        log "INFO" "Environment: $environment"
        ;;
    *)
        error_exit "Invalid environment. Use: dev, staging, or prod"
        ;;
esac

#===========================================
# AZURE AUTHENTICATION
#===========================================

log "INFO" "Checking Azure authentication..."
if ! az account show &> /dev/null; then
    error_exit "Not authenticated to Azure. Run 'az login'."
fi

subscription=$(az account show --query name -o tsv)
log "INFO" "Using subscription: $subscription"

#===========================================
# DEPLOYMENT
#===========================================

resourceGroup="rg-app-${environment}"

log "INFO" "Creating resource group: $resourceGroup"
if az group create --name "$resourceGroup" --location "$location" &>> "$LOG_FILE"; then
    log "SUCCESS" "Resource group created"
else
    error_exit "Failed to create resource group"
fi

log "INFO" "Deployment complete!"
```

### Key Script Components

**1. Shebang (`#!/bin/bash`)**:
- Specifies Bash interpreter
- Must be first line
- Alternative: `#!/usr/bin/env bash` (more portable)

**2. Error Handling**:
```bash
set -e          # Exit immediately if command fails
set -u          # Treat unset variables as errors
set -o pipefail # Pipeline fails if any command fails
```

**3. Variables**:
```bash
# Lowercase for local variables
resourceGroup="rg-demo"
location="eastus"

# Uppercase for constants
readonly MAX_RETRIES=3
readonly TIMEOUT=300
```

**4. Comments**:
```bash
# Single-line comment
: << 'END_COMMENT'
Multi-line comment block
spanning multiple lines
END_COMMENT
```

**5. Functions**:
```bash
create_resource_group() {
    local name=$1
    local location=$2
    
    az group create --name "$name" --location "$location"
}
```

**6. Error Handling**:
```bash
if az group create --name "$resourceGroup" --location "$location"; then
    echo "✓ Success"
else
    echo "✗ Failed"
    exit 1
fi
```

## Create a Resource Deployment Script

### Scenario: Deploy Multiple Storage Accounts

```bash
#!/bin/bash
# Script: deploy-storage-accounts.sh
# Description: Automates storage account creation with consistent naming and configuration

#===========================================
# CONFIGURATION
#===========================================

# Deployment parameters
resourceGroupName="rg-storage-prod-eastus"
saCount=3
saLocation="eastus"
saNamePrefix="stprod"
saSku="Standard_GRS"
saKind="StorageV2"

#===========================================
# PRE-FLIGHT CHECKS
#===========================================

# Validate Azure CLI is installed
if ! command -v az &> /dev/null; then
    echo "Error: Azure CLI is not installed. Please install Azure CLI first."
    exit 1
fi

# Validate authentication
if ! az account show &> /dev/null; then
    echo "Error: Not authenticated to Azure. Please run 'az login' first."
    exit 1
fi

echo "Starting storage account deployment..."
echo "Resource Group: $resourceGroupName"
echo "Location: $saLocation"
echo "Count: $saCount"
echo ""

#===========================================
# RESOURCE GROUP CREATION
#===========================================

# Create resource group if it doesn't exist (idempotent)
if ! az group show --name $resourceGroupName &> /dev/null; then
    echo "Creating resource group $resourceGroupName..."
    az group create --name $resourceGroupName --location $saLocation
else
    echo "Resource group $resourceGroupName already exists."
fi

#===========================================
# STORAGE ACCOUNT DEPLOYMENT
#===========================================

# Loop to create multiple storage accounts
for i in $(seq 1 $saCount)
do
    # Generate unique identifier using timestamp and random number
    timestamp=$(date +%s)
    let "randomIdentifier=$RANDOM"
    saName="${saNamePrefix}${timestamp}${randomIdentifier}"

    # Ensure name is lowercase and within 24 character limit
    saName=$(echo $saName | tr '[:upper:]' '[:lower:]' | cut -c1-24)

    echo "Creating storage account $saName..."

    # Create storage account with error handling
    if az storage account create \
        --name $saName \
        --resource-group $resourceGroupName \
        --location $saLocation \
        --sku $saSku \
        --kind $saKind \
        --tags Environment=Production ManagedBy=AzureCLI \
        --output none; then
        echo "✓ Successfully created storage account $saName"
    else
        echo "✗ Failed to create storage account $saName"
    fi
done

echo ""
echo "Deployment complete. Verifying results..."
echo ""

#===========================================
# VERIFICATION
#===========================================

# Verify results with formatted output
az storage account list \
    --resource-group $resourceGroupName \
    --query "[].{Name:name, Location:location, SKU:sku.name, Status:statusOfPrimary}" \
    --output table
```

### Script Breakdown

**Authentication Validation**:
```bash
if ! az account show &> /dev/null; then
    echo "Error: Not authenticated to Azure. Please run 'az login' first."
    exit 1
fi
```
- Prevents script execution without authentication
- `&> /dev/null` suppresses output
- Fails fast with clear error message

**Idempotent Resource Group Creation**:
```bash
if ! az group show --name $resourceGroupName &> /dev/null; then
    az group create --name $resourceGroupName --location $saLocation
fi
```
- Checks existence before creating
- Script can run multiple times safely
- No error if resource already exists

**Unique Name Generation**:
```bash
timestamp=$(date +%s)
let "randomIdentifier=$RANDOM"
saName="${saNamePrefix}${timestamp}${randomIdentifier}"
saName=$(echo $saName | tr '[:upper:]' '[:lower:]' | cut -c1-24)
```
- Combines timestamp and random number for uniqueness
- Converts to lowercase (Azure requirement)
- Limits to 24 characters (Azure requirement)

**Error Handling Per Resource**:
```bash
if az storage account create ...; then
    echo "✓ Successfully created storage account $saName"
else
    echo "✗ Failed to create storage account $saName"
fi
```
- Validates each creation individually
- Continues processing remaining accounts if one fails
- Clear success/failure indicators

## Create a Parameterized Script

### Accept Command-Line Arguments

```bash
#!/bin/bash
# Script: create-storage-accounts.sh
# Usage: ./create-storage-accounts.sh <resource-group> <location> <count> <environment>
# Example: ./create-storage-accounts.sh rg-storage-dev eastus 3 Development

#===========================================
# PARAMETER PARSING
#===========================================

# Accept command-line parameters
resourceGroupName=$1
saLocation=$2
saCount=$3
environment=$4

# Validate parameters
if [ -z "$resourceGroupName" ] || [ -z "$saLocation" ] || [ -z "$saCount" ] || [ -z "$environment" ]; then
    echo "Usage: $0 <resource-group> <location> <count> <environment>"
    echo "Example: $0 rg-storage-dev eastus 3 Development"
    exit 1
fi

# Validate count is a number
if ! [[ "$saCount" =~ ^[0-9]+$ ]]; then
    echo "Error: Count must be a number"
    exit 1
fi

echo "Parameters received:"
echo "  Resource Group: $resourceGroupName"
echo "  Location: $saLocation"
echo "  Count: $saCount"
echo "  Environment: $environment"
echo ""

#===========================================
# ENVIRONMENT-SPECIFIC CONFIGURATION
#===========================================

# Set environment-specific configurations
case $environment in
    Development)
        saSku="Standard_LRS"
        saKind="StorageV2"
        ;;
    Staging)
        saSku="Standard_GRS"
        saKind="StorageV2"
        ;;
    Production)
        saSku="Standard_RAGRS"
        saKind="StorageV2"
        ;;
    *)
        echo "Error: Environment must be Development, Staging, or Production"
        exit 1
        ;;
esac

echo "Using SKU: $saSku for $environment environment"
echo ""

# Continue with resource creation...
```

### Execute Parameterized Script

```bash
# Make script executable
chmod +x create-storage-accounts.sh

# Execute with parameters
./create-storage-accounts.sh rg-storage-dev eastus 3 Development

# Different environment, different SKU
./create-storage-accounts.sh rg-storage-prod westus2 5 Production
```

### Advanced Parameter Handling

```bash
#!/bin/bash
# Script with named parameters and defaults

# Default values
RESOURCE_GROUP=""
LOCATION="eastus"
COUNT=1
ENVIRONMENT="Development"
VERBOSE=false

# Parse named parameters
while [[ $# -gt 0 ]]; do
    case $1 in
        -g|--resource-group)
            RESOURCE_GROUP="$2"
            shift 2
            ;;
        -l|--location)
            LOCATION="$2"
            shift 2
            ;;
        -c|--count)
            COUNT="$2"
            shift 2
            ;;
        -e|--environment)
            ENVIRONMENT="$2"
            shift 2
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        -h|--help)
            echo "Usage: $0 -g <resource-group> [-l <location>] [-c <count>] [-e <environment>] [-v]"
            exit 0
            ;;
        *)
            echo "Unknown parameter: $1"
            exit 1
            ;;
    esac
done

# Validate required parameters
if [ -z "$RESOURCE_GROUP" ]; then
    echo "Error: Resource group is required (-g)"
    exit 1
fi

# Usage examples:
# ./script.sh -g rg-demo -l westus2 -c 3 -e Production -v
# ./script.sh --resource-group rg-demo --count 5
```

## Configuration-Driven Deployments

### JSON Configuration File

**config.json**:
```json
{
  "resourceGroup": {
    "name": "rg-app-prod-eastus",
    "location": "eastus"
  },
  "storageAccounts": [
    {
      "namePrefix": "stappdata",
      "sku": "Standard_RAGRS",
      "kind": "StorageV2",
      "count": 2
    },
    {
      "namePrefix": "stappbackup",
      "sku": "Standard_GRS",
      "kind": "StorageV2",
      "count": 1
    }
  ],
  "tags": {
    "Environment": "Production",
    "CostCenter": "Engineering",
    "ManagedBy": "AzureCLI"
  }
}
```

### Configuration-Driven Script

**deploy-from-config.sh**:
```bash
#!/bin/bash
# Script: deploy-from-config.sh
# Description: Deploy resources from JSON configuration file

# Configuration file
configFile="config.json"

if [ ! -f "$configFile" ]; then
    echo "Error: Configuration file $configFile not found"
    exit 1
fi

# Parse configuration using jq (JSON processor)
resourceGroupName=$(jq -r '.resourceGroup.name' $configFile)
resourceGroupLocation=$(jq -r '.resourceGroup.location' $configFile)
tagsJson=$(jq -r '.tags | to_entries | map("\(.key)=\(.value)") | join(" ")' $configFile)

echo "Deploying resources from configuration file..."
echo "Resource Group: $resourceGroupName"
echo "Location: $resourceGroupLocation"
echo ""

# Create resource group
az group create --name $resourceGroupName --location $resourceGroupLocation

# Read storage account configurations
storageConfigCount=$(jq '.storageAccounts | length' $configFile)

for i in $(seq 0 $(($storageConfigCount - 1)))
do
    namePrefix=$(jq -r ".storageAccounts[$i].namePrefix" $configFile)
    sku=$(jq -r ".storageAccounts[$i].sku" $configFile)
    kind=$(jq -r ".storageAccounts[$i].kind" $configFile)
    count=$(jq -r ".storageAccounts[$i].count" $configFile)

    echo "Creating $count storage account(s) with prefix: $namePrefix"

    for j in $(seq 1 $count)
    do
        timestamp=$(date +%s)
        saName="${namePrefix}${timestamp}${RANDOM}"
        saName=$(echo $saName | tr '[:upper:]' '[:lower:]' | cut -c1-24)

        az storage account create \
            --name $saName \
            --resource-group $resourceGroupName \
            --location $resourceGroupLocation \
            --sku $sku \
            --kind $kind \
            --tags $tagsJson \
            --output none

        echo "  ✓ Created: $saName"
    done
done

echo ""
echo "Deployment complete."
```

### Configuration-Driven Benefits

1. **Separation of Concerns**: Configuration separate from deployment logic
2. **Environment Promotion**: Same script, different config files (dev.json, staging.json, prod.json)
3. **Version Control**: Track configuration changes alongside code
4. **Validation**: Validate JSON structure before deployment
5. **Team Collaboration**: Non-developers can modify config files

### Multiple Environment Configurations

```bash
# Project structure
.
├── deploy.sh
├── config/
│   ├── dev.json
│   ├── staging.json
│   └── prod.json

# Deploy to different environments
./deploy.sh config/dev.json
./deploy.sh config/staging.json
./deploy.sh config/prod.json
```

## Delete Azure Resources with Scripts

### Delete Storage Accounts by Creation Date

```bash
#!/bin/bash
# Script: cleanup-old-storage-accounts.sh
# Description: Delete storage accounts created on or after a specific date

# Define cleanup parameters
cutoffDate="2025-10-08T00:00:00.000000+00:00"
resourceGroup="rg-storage-dev-eastus"

echo "Finding storage accounts created on or after $cutoffDate..."

# Get list of storage accounts matching criteria
saList=$(az storage account list \
    --resource-group $resourceGroup \
    --query "[?creationTime >='$cutoffDate'].{Name:name, Created:creationTime}" \
    --output table)

echo "$saList"
echo ""

# Confirm deletion
read -p "Delete these storage accounts? (yes/no): " confirm

if [ "$confirm" == "yes" ]; then
    for saId in $(az storage account list \
        --resource-group $resourceGroup \
        --query "[?creationTime >='$cutoffDate'].id" \
        --output tsv); do
        echo "Deleting storage account: $saId"
        az storage account delete --ids $saId --yes
    done
    echo "Cleanup complete."
else
    echo "Deletion cancelled."
fi
```

### Delete Resource Groups with Logging

```bash
#!/bin/bash
# Script: cleanup-resource-groups.sh
# Description: Delete resource groups matching a naming pattern with comprehensive logging

# Define cleanup parameters
namePattern="rg-dev-*"
logFileLocation="cleanup-$(date +%Y%m%d-%H%M%S).log"

echo "Resource Group Cleanup" > $logFileLocation
echo "Started: $(date)" >> $logFileLocation
echo "Pattern: $namePattern" >> $logFileLocation
echo "----------------------------------------" >> $logFileLocation

# Find matching resource groups
echo "Finding resource groups matching pattern: $namePattern"
matchingGroups=$(az group list \
    --query "[?starts_with(name, 'rg-dev-')].name" \
    --output tsv)

if [ -z "$matchingGroups" ]; then
    echo "No resource groups found matching pattern: $namePattern"
    echo "No resource groups found" >> $logFileLocation
    exit 0
fi

# Display matches
echo "Found resource groups:"
echo "$matchingGroups"
echo ""

# Log matches
echo "Resource groups found:" >> $logFileLocation
echo "$matchingGroups" >> $logFileLocation
echo "" >> $logFileLocation

# Confirm deletion
read -p "Delete these resource groups? (yes/no): " confirm

if [ "$confirm" == "yes" ]; then
    echo "Starting deletion..." >> $logFileLocation

    for rgName in $matchingGroups
    do
        echo "Deleting resource group: $rgName"
        echo "$(date): Deleting $rgName" >> $logFileLocation

        # Delete with --no-wait for background execution
        if az group delete --name $rgName --yes --no-wait; then
            echo "  ✓ Deletion initiated for $rgName"
            echo "$(date): ✓ Deletion initiated for $rgName" >> $logFileLocation
        else
            echo "  ✗ Failed to initiate deletion for $rgName"
            echo "$(date): ✗ Failed to delete $rgName" >> $logFileLocation
        fi
    done

    echo ""
    echo "Deletion operations initiated. Resources will be removed in the background."
    echo "Check deletion status with: az group list --query \"[?starts_with(name, 'rg-dev-')].name\""

    echo "" >> $logFileLocation
    echo "Completed: $(date)" >> $logFileLocation
    echo "Log saved to: $logFileLocation"

    # Display log
    echo ""
    echo "=== Cleanup Log ==="
    cat $logFileLocation
else
    echo "Deletion cancelled by user" >> $logFileLocation
    echo "Deletion cancelled."
fi
```

### Safe Deletion with Multiple Confirmations

```bash
#!/bin/bash
# Script: safe-delete-resource-group.sh
# Description: Delete resources with multiple confirmations

resourceGroup="rg-storage-test-eastus"

# Check if resource group exists
if ! az group show --name $resourceGroup &> /dev/null; then
    echo "Error: Resource group '$resourceGroup' not found"
    exit 1
fi

# Display resources that will be deleted
echo "Resources in resource group '$resourceGroup':"
az resource list --resource-group $resourceGroup \
    --query "[].{Name:name, Type:type, Location:location}" \
    --output table

echo ""
echo "⚠️  WARNING: This will delete ALL resources in '$resourceGroup'"
echo ""

# First confirmation
read -p "Are you sure you want to delete '$resourceGroup'? (yes/no): " confirm1

if [ "$confirm1" != "yes" ]; then
    echo "Deletion cancelled."
    exit 0
fi

# Second confirmation with exact name
read -p "Type the resource group name to confirm: " confirm2

if [ "$confirm2" != "$resourceGroup" ]; then
    echo "Resource group name does not match. Deletion cancelled."
    exit 1
fi

# Final countdown
echo "Deleting in 5 seconds. Press Ctrl+C to cancel."
sleep 5

echo "Deleting resource group '$resourceGroup'..."
az group delete --name $resourceGroup --yes

echo "Resource group deleted successfully."
```

## Best Practices for Production Scripts

### Essential Production Script Components

**1. Authentication and Authorization (Service Principal)**:

```bash
#!/bin/bash
# Authenticate with service principal (for automation)
if [ -n "$AZURE_CLIENT_ID" ] && [ -n "$AZURE_CLIENT_SECRET" ] && [ -n "$AZURE_TENANT_ID" ]; then
    echo "Authenticating with service principal..."
    az login --service-principal \
        --username $AZURE_CLIENT_ID \
        --password $AZURE_CLIENT_SECRET \
        --tenant $AZURE_TENANT_ID
else
    echo "Error: Service principal credentials not found"
    exit 1
fi

# Set subscription
az account set --subscription $AZURE_SUBSCRIPTION_ID

# Verify authentication
if ! az account show &> /dev/null; then
    echo "Error: Authentication failed"
    exit 1
fi
```

**2. Comprehensive Error Handling**:

```bash
#!/bin/bash
# Exit immediately if command fails
set -e

# Trap errors and perform cleanup
trap 'cleanup_on_error' ERR

cleanup_on_error() {
    echo "Error occurred on line $1"
    echo "Performing cleanup..."
    # Add cleanup logic here
    exit 1
}

# Enable error trapping with line numbers
trap 'cleanup_on_error $LINENO' ERR
```

**3. Idempotency Checks**:

```bash
# Check if resource group exists before creation
create_resource_group() {
    local rgName=$1
    local location=$2

    if az group show --name $rgName &> /dev/null; then
        echo "Resource group $rgName already exists. Skipping creation."
    else
        echo "Creating resource group $rgName..."
        az group create --name $rgName --location $location
    fi
}

# Check if storage account exists
create_storage_account() {
    local saName=$1
    local rgName=$2

    if az storage account show --name $saName --resource-group $rgName &> /dev/null; then
        echo "Storage account $saName already exists. Skipping creation."
        return 0
    else
        echo "Creating storage account $saName..."
        az storage account create --name $saName --resource-group $rgName --location eastus
    fi
}
```

**4. Structured Logging**:

```bash
# Define log file with timestamp
logFile="deployment-$(date +%Y%m%d-%H%M%S).log"
errorLog="errors-$(date +%Y%m%d-%H%M%S).log"

# Logging function
log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")

    echo "[$timestamp] [$level] $message" | tee -a $logFile

    if [ "$level" == "ERROR" ]; then
        echo "[$timestamp] $message" >> $errorLog
    fi
}

# Usage
log "INFO" "Starting deployment..."
log "ERROR" "Failed to create resource"
log "SUCCESS" "Deployment complete"
```

**5. Configuration Validation**:

```bash
# Validate required configuration
validate_config() {
    local errors=0

    if [ -z "$RESOURCE_GROUP" ]; then
        echo "Error: RESOURCE_GROUP not defined"
        ((errors++))
    fi

    if [ -z "$LOCATION" ]; then
        echo "Error: LOCATION not defined"
        ((errors++))
    fi

    # Validate location exists
    if ! az account list-locations --query "[?name=='$LOCATION']" | grep -q "$LOCATION"; then
        echo "Error: Invalid location: $LOCATION"
        ((errors++))
    fi

    if [ $errors -gt 0 ]; then
        echo "Configuration validation failed with $errors error(s)"
        exit 1
    fi

    echo "Configuration validation passed"
}
```

**6. Tagging Strategy**:

```bash
# Define standard tags
environment=$1  # Development, Staging, Production
costCenter=$2
owner=$3

# Create tags string
tags="Environment=$environment CostCenter=$costCenter Owner=$owner \
      CreatedBy=AzureCLI CreatedDate=$(date +%Y-%m-%d) \
      ManagedBy=Infrastructure-Team Project=WebApp"

# Apply tags to resources
az group create \
    --name $resourceGroup \
    --location $location \
    --tags $tags

az storage account create \
    --name $saName \
    --resource-group $resourceGroup \
    --location $location \
    --tags $tags
```

**7. Rollback Capability**:

```bash
# Track created resources for rollback
createdResources=()

create_with_tracking() {
    local resourceType=$1
    local resourceName=$2

    # Attempt resource creation
    if create_resource "$resourceType" "$resourceName"; then
        createdResources+=("$resourceType:$resourceName")
        echo "✓ Created: $resourceType - $resourceName"
        return 0
    else
        echo "✗ Failed: $resourceType - $resourceName"
        return 1
    fi
}

rollback() {
    echo "Rolling back created resources..."

    for resource in "${createdResources[@]}"; do
        IFS=':' read -r type name <<< "$resource"
        echo "Deleting $type: $name"
        delete_resource "$type" "$name"
    done

    echo "Rollback complete"
}

# Use trap to rollback on error
trap rollback ERR
```

### Script Organization Structure

```
azure-infrastructure/
├── config/
│   ├── dev.json
│   ├── staging.json
│   └── prod.json
├── scripts/
│   ├── deploy-infrastructure.sh
│   ├── deploy-storage.sh
│   └── cleanup.sh
├── lib/
│   ├── common-functions.sh
│   ├── logging.sh
│   └── validation.sh
├── logs/
└── README.md
```

### Reusable Functions Library

**lib/common-functions.sh**:

```bash
#!/bin/bash
# Load this file in other scripts: source ./lib/common-functions.sh

# Check if Azure CLI is installed
check_azure_cli() {
    if ! command -v az &> /dev/null; then
        echo "Error: Azure CLI not installed"
        exit 1
    fi
}

# Wait for resource to be ready
wait_for_resource() {
    local resourceId=$1
    local maxAttempts=30
    local attempt=1

    echo "Waiting for resource to be ready..."

    while [ $attempt -le $maxAttempts ]; do
        if az resource show --ids $resourceId &> /dev/null; then
            echo "Resource is ready"
            return 0
        fi

        echo "Attempt $attempt/$maxAttempts - waiting..."
        sleep 10
        ((attempt++))
    done

    echo "Timeout waiting for resource"
    return 1
}

# Generate unique name
generate_unique_name() {
    local prefix=$1
    local timestamp=$(date +%s)
    local random=$RANDOM
    echo "${prefix}${timestamp}${random}" | tr '[:upper:]' '[:lower:]' | cut -c1-24
}
```

### Testing Scripts Safely

```bash
#!/bin/bash
# Test mode flag
TEST_MODE=${TEST_MODE:-false}

execute_command() {
    local command=$1

    if [ "$TEST_MODE" == "true" ]; then
        echo "[TEST MODE] Would execute: $command"
    else
        echo "Executing: $command"
        eval $command
    fi
}

# Usage
execute_command "az group create --name rg-test --location eastus"
```

Run in test mode:
```bash
TEST_MODE=true ./deploy-infrastructure.sh
```

## Key Takeaways

1. **Scripts Enable Automation**: Consistent, repeatable infrastructure deployment
2. **Structure Matters**: Shebang, error handling, functions, logging, cleanup
3. **Idempotency**: Scripts should be safe to run multiple times
4. **Error Handling**: Trap errors, log failures, rollback on failure
5. **Parameterization**: Accept command-line arguments for flexibility
6. **Configuration-Driven**: Separate configuration from deployment logic
7. **Version Control**: Store scripts in Git for collaboration and audit trail
8. **Production Readiness**: Service principal auth, comprehensive logging, validation

## Next Steps

With scripting skills established, proceed to **Unit 6: Tips to Use the Azure CLI Successfully** to master troubleshooting, debugging with `--debug`, understanding scripting language differences, and leveraging Azure CLI documentation resources.

---

**Module**: Create Azure Resources Using Azure CLI  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/create-azure-resources-by-using-azure-cli/5-create-an-azure-cli-script
