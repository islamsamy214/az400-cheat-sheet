# Why Use Azure Resource Manager Templates?

## Overview

Resource Manager templates (ARM templates) represent a fundamental shift in how organizations manage Azure infrastructure. Rather than treating infrastructure as manually configured systems that evolve over time, ARM templates enable infrastructure as code (IaC) where infrastructure is defined, versioned, tested, and deployed like application code.

This unit explores the compelling reasons to adopt ARM templates, examining the problems they solve, the benefits they provide, and the competitive advantages they offer organizations managing cloud infrastructure at scale.

## The Challenge with Manual Deployments

Traditional infrastructure provisioning methods—manual portal configuration and imperative scripting—create significant operational challenges that become more acute as organizations scale.

### Manual Portal Deployment Problems

**Scenario**: Operations engineer provisions production environment through Azure Portal.

**The Process**:
1. Open Azure Portal, navigate to resource creation
2. Click through wizard screens for Virtual Network creation
3. Manually enter CIDR blocks, subnet names, DNS settings
4. Click "Create," wait for deployment
5. Repeat process for Network Security Groups
6. Manually configure security rules one by one
7. Create Public IP address
8. Create Network Interface, manually select VNet and subnet
9. Create Virtual Machine, manually configure all settings
10. Realize typo in VM name, cannot change it
11. Delete VM, start over from step 9
12. 4 hours later: environment created

**Problems**:

**1. Human Error Risk**
- Typos in resource names (cannot be changed after creation for many resources)
- Wrong SKU selection (Standard_B2s when Standard_D2s_v3 intended)
- Incorrect region selection (East US instead of East US 2)
- Missed configuration settings (forgot to enable encryption)
- Wrong subnet selection (VM in DMZ subnet instead of app subnet)

**2. Inconsistency Across Environments**
- Development configured differently than staging
- Staging has different NSG rules than production
- "It works in dev but fails in production" mysteries
- Configuration drift accumulates over time
- No way to verify environments match

**3. Time Consumption**
- Simple environment: 2-4 hours
- Complex environment: 1-2 days
- Multi-region deployment: multiply time by number of regions
- Updates: repeat entire process for changes

**4. No Documentation**
- Configuration decisions lost to memory
- "Why is this subnet /24 instead of /25?" Unknown.
- Tribal knowledge: only senior engineer knows production setup
- New team members can't understand infrastructure
- Compliance audits require manual documentation efforts

**5. Non-Repeatable**
- Disaster recovery requires recreating environment from memory
- Scaling to new regions requires manual replication
- Testing changes requires separate test environment creation
- Development environments inconsistent across team

**6. No Version Control**
- Can't track what changed or when
- No rollback capability when something breaks
- No approval process for infrastructure changes
- Changes made directly in production without review

### Imperative Script Problems

**Scenario**: DevOps engineer writes PowerShell script to automate deployment.

**Script Approach**:
```powershell
# Create resource group
New-AzResourceGroup -Name "myapp-rg" -Location "eastus"

# Create virtual network
$vnet = New-AzVirtualNetwork `
    -Name "myapp-vnet" `
    -ResourceGroupName "myapp-rg" `
    -Location "eastus" `
    -AddressPrefix "10.0.0.0/16"

# Create subnet
$subnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "app-subnet" `
    -AddressPrefix "10.0.1.0/24" `
    -VirtualNetwork $vnet
$vnet | Set-AzVirtualNetwork

# Create public IP
$pip = New-AzPublicIpAddress `
    -Name "myapp-pip" `
    -ResourceGroupName "myapp-rg" `
    -Location "eastus" `
    -AllocationMethod Static

# Create network interface
$nic = New-AzNetworkInterface `
    -Name "myapp-nic" `
    -ResourceGroupName "myapp-rg" `
    -Location "eastus" `
    -SubnetId $vnet.Subnets[0].Id `
    -PublicIpAddressId $pip.Id

# Create VM configuration
$vmConfig = New-AzVMConfig -VMName "myapp-vm" -VMSize "Standard_D2s_v3"
# ... many more lines of configuration
```

**Problems**:

**1. Order Dependency**
- Commands must execute in precise sequence
- Cannot create NIC before VNet exists
- Cannot create VM before NIC exists
- Parallel execution impossible (some resources could create simultaneously)

**2. Non-Idempotent**
- Running script twice fails at first command (resource group exists)
- Requires extensive error handling:
```powershell
# Check if resource group exists
$rg = Get-AzResourceGroup -Name "myapp-rg" -ErrorAction SilentlyContinue
if ($null -eq $rg) {
    New-AzResourceGroup -Name "myapp-rg" -Location "eastus"
}

# Check if VNet exists
$vnet = Get-AzVirtualNetwork -Name "myapp-vnet" -ResourceGroupName "myapp-rg" -ErrorAction SilentlyContinue
if ($null -eq $vnet) {
    # Create VNet
}
# Repeat for every resource...
```

**3. Fragile to Failures**
- Network timeout after creating VNet but before NIC creation
- Script state: VNet created, nothing else
- Re-running script: Fails at VNet creation (already exists)
- Manual intervention required to continue
- Partial deployments leave inconsistent state

**4. Complex Error Handling**
```powershell
try {
    $vnet = New-AzVirtualNetwork ...
} catch {
    if ($_.Exception.Message -like "*already exists*") {
        # Get existing VNet
        $vnet = Get-AzVirtualNetwork ...
    } else {
        # Handle other errors
        Write-Error "Failed to create VNet: $_"
        # Cleanup? Retry? Fail?
    }
}
# Repeat for every resource...
```

**5. Hard to Maintain**
- Adding new resource requires updating multiple sections
- Changing resource name requires finding all references
- Script grows to hundreds of lines
- Logic mixed with infrastructure definition
- Difficult to understand what infrastructure will be created

**6. Environment-Specific Logic**
```powershell
if ($Environment -eq "production") {
    $vmSize = "Standard_D4s_v3"
    $enableBackup = $true
} elseif ($Environment -eq "staging") {
    $vmSize = "Standard_D2s_v3"
    $enableBackup = $true
} else {
    $vmSize = "Standard_B2s"
    $enableBackup = $false
}
# Logic scattered throughout script
```

## ARM Templates Solution

ARM templates solve these problems through declarative infrastructure definition:

### Declarative Approach

**What You Write** (template.json):
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D2s_v3"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2023-04-01",
      "name": "myapp-vnet",
      "location": "eastus",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        },
        "subnets": [
          {
            "name": "app-subnet",
            "properties": {
              "addressPrefix": "10.0.1.0/24"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2023-04-01",
      "name": "myapp-pip",
      "location": "eastus",
      "properties": {
        "publicIPAllocationMethod": "Static"
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2023-04-01",
      "name": "myapp-nic",
      "location": "eastus",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', 'myapp-vnet')]",
        "[resourceId('Microsoft.Network/publicIPAddresses', 'myapp-pip')]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'myapp-vnet', 'app-subnet')]"
              },
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', 'myapp-pip')]"
              }
            }
          }
        ]
      }
    }
  ]
}
```

**What Resource Manager Does**:
1. Validates template syntax and structure
2. Analyzes all resources and dependencies
3. Creates deployment plan with correct ordering
4. Deploys independent resources (VNet, Public IP) in parallel
5. Waits for dependencies before creating NIC
6. Handles errors and provides detailed logs
7. **Idempotent**: Running again updates only changed resources

### Key Differences

| Aspect | Manual/Scripts | ARM Templates |
|--------|---------------|---------------|
| **Approach** | Imperative (how) | Declarative (what) |
| **Ordering** | Manual | Automatic |
| **Idempotency** | Must code explicitly | Built-in |
| **Parallelism** | Sequential | Automatic parallel |
| **Dependencies** | Manual management | Automatic resolution |
| **Validation** | At runtime | Before deployment |
| **Rollback** | Manual cleanup | Automated |
| **Versioning** | Script versions | Template versions |
| **Testing** | Run and verify | Validate before deploy |

## Complete List of ARM Template Advantages

### 1. Improved Consistency

**The Problem**: Manual deployments create configuration drift. Development has 3 VMs, staging has 5, production has 6—but they're configured differently.

**ARM Templates Solution**: Common language for infrastructure across all tools.

**Benefits**:

**Cross-Team Collaboration**
- **Developers**: Define infrastructure requirements in templates
- **Operations**: Review and approve infrastructure changes via PR
- **Security**: Add security policies to templates
- **All teams**: Work with same infrastructure definitions

Example workflow:
```bash
# Developer defines infrastructure need
git checkout -b feature/add-redis-cache
# Edit template to add Azure Cache for Redis
git commit -m "Add Redis cache for session management"
git push

# Pull request created
# Operations reviews infrastructure changes
# Security reviews network configuration
# Automated tests validate template
# Approved and merged

# CI/CD deploys to dev, then staging, then production
# Same template used for all environments (different parameters)
```

**Multi-Environment Reliability**
- Development, staging, production use identical template structure
- Only parameters differ (VM sizes, SKUs, region)
- Guaranteed consistency: if it works in dev, it works in prod
- Configuration drift eliminated

**Tool Independence**
- Deploy via Azure Portal: Upload template, fill parameters
- Deploy via Azure CLI: `az deployment group create`
- Deploy via PowerShell: `New-AzResourceGroupDeployment`
- Deploy via Azure DevOps Pipelines: Built-in tasks
- Deploy via GitHub Actions: Azure CLI action
- Deploy via Terraform: Azure provider can import ARM templates

### 2. Express Complex Deployments Clearly

**The Problem**: 50+ resource deployment requires understanding creation order, dependencies, and timing.

**ARM Templates Solution**: Declare all resources, Resource Manager handles complexity.

**Benefits**:

**Dependency Awareness**
```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "dependsOn": [
    "[resourceId('Microsoft.Network/networkInterfaces', 'mynic')]",
    "[resourceId('Microsoft.Storage/storageAccounts', 'mystorage')]"
  ]
}
```
- Resource Manager won't create VM before NIC and storage exist
- Explicit dependencies clearly documented in template
- Changes to dependencies automatically handled

**Automatic Ordering**
Resource Manager creates deployment graph:
```
Virtual Network ──┬──> Network Interface ──> Virtual Machine
                  │
Public IP ────────┘
Storage Account ──────────────────────────> Virtual Machine

Parallel: VNet, Public IP, Storage Account (all independent)
Sequential: NIC waits for VNet and Public IP
Sequential: VM waits for NIC and Storage Account
```

**Visual Representation**
- All resources visible in single file
- Relationships clear from dependsOn declarations
- Easy to understand complete infrastructure at a glance
- Documentation embedded in infrastructure definition

### 3. Infrastructure as Code Paradigm

**The Problem**: Infrastructure changes made directly in production, no history, no review, no rollback.

**ARM Templates Solution**: Infrastructure is code, managed like application code.

**Benefits**:

**Version Control**
```bash
# Infrastructure repository
git clone https://github.com/company/infrastructure.git

# View infrastructure history
git log --oneline
# a1b2c3d Add Redis cache for session management
# e4f5g6h Increase VM size for production workload
# h7i8j9k Add Azure SQL database with geo-replication

# View specific change
git show a1b2c3d
# Shows exactly what infrastructure changed

# Rollback to previous version
git revert a1b2c3d
git push
# CI/CD deploys previous infrastructure configuration
```

**Code Reviews**
```yaml
# Pull Request Process
1. Developer proposes infrastructure change
2. Automated validation runs:
   - Template syntax validation
   - Security scanning (no hardcoded secrets)
   - Cost estimation (Azure Pricing API)
   - terraform plan preview
3. Team reviews changes:
   - Security team: "NSG rules too permissive"
   - Operations: "VM size appropriate for workload?"
   - Cost management: "Estimated $500/month increase"
4. Changes approved after discussion
5. Merged to main branch
6. Deployed via CI/CD
```

**Testing**
```yaml
# Automated Infrastructure Testing
stages:
  - name: Validate
    steps:
      - script: az deployment group validate --template-file template.json
  
  - name: SecurityScan
    steps:
      - script: checkov --file template.json
  
  - name: DeployDev
    steps:
      - script: az deployment group create --resource-group dev-rg --template-file template.json
  
  - name: IntegrationTests
    steps:
      - script: pytest tests/infrastructure_tests.py
  
  - name: DeployProduction
    condition: Manual approval
    steps:
      - script: az deployment group create --resource-group prod-rg --template-file template.json
```

**Audit Trail**
- Every infrastructure change documented in Git
- Who changed what, when, and why (commit messages)
- Pull request discussions captured
- Compliance requirements met automatically

### 4. Promote Reuse Through Parameters

**The Problem**: Maintain separate templates for dev, staging, production—90% identical, 10% different.

**ARM Templates Solution**: Single template with parameters for environment-specific values.

**Example**:
```json
{
  "parameters": {
    "environmentName": {
      "type": "string",
      "allowedValues": ["dev", "staging", "production"],
      "metadata": {
        "description": "Environment name determines VM sizes and redundancy"
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_B2s"
    },
    "vmCount": {
      "type": "int",
      "defaultValue": 1,
      "minValue": 1,
      "maxValue": 100
    },
    "enableBackup": {
      "type": "bool",
      "defaultValue": false
    }
  },
  "variables": {
    "vmSizeMap": {
      "dev": "Standard_B2s",
      "staging": "Standard_D2s_v3",
      "production": "Standard_D4s_v3"
    },
    "selectedVmSize": "[variables('vmSizeMap')[parameters('environmentName')]]",
    "backupEnabled": "[if(equals(parameters('environmentName'), 'production'), true(), parameters('enableBackup'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[concat(parameters('environmentName'), '-vm-', copyIndex())]",
      "copy": {
        "name": "vmCopy",
        "count": "[parameters('vmCount')]"
      },
      "properties": {
        "hardwareProfile": {
          "vmSize": "[variables('selectedVmSize')]"
        }
      }
    }
  ]
}
```

**Usage**:
```bash
# Development
az deployment group create \
  --template-file app.json \
  --parameters environmentName=dev vmCount=1

# Staging
az deployment group create \
  --template-file app.json \
  --parameters environmentName=staging vmCount=2

# Production
az deployment group create \
  --template-file app.json \
  --parameters environmentName=production vmCount=5
```

**Benefits**:
- **Environment Flexibility**: Same template for all environments
- **Cost Optimization**: Dev uses B2s, production uses D4s_v3
- **Reduced Duplication**: One template instead of three
- **Consistent Updates**: Change template once, affects all environments

### 5. Modular and Linkable Templates

**The Problem**: Monolithic 5,000-line template hard to understand, test, and maintain.

**ARM Templates Solution**: Break into small, focused modules that link together.

**Example Structure**:
```
infrastructure/
├── main.json                  # Master template
├── modules/
│   ├── networking.json        # VNet, subnets, NSGs
│   ├── storage.json           # Storage accounts
│   ├── compute.json           # Virtual machines
│   ├── database.json          # Azure SQL
│   └── monitoring.json        # Log Analytics, Application Insights
└── parameters/
    ├── dev.parameters.json
    ├── staging.parameters.json
    └── prod.parameters.json
```

**Main Template** (main.json):
```json
{
  "resources": [
    {
      "name": "networkingDeployment",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2022-09-01",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[uri(deployment().properties.templateLink.uri, 'modules/networking.json')]"
        },
        "parameters": {
          "vnetName": { "value": "[parameters('vnetName')]" }
        }
      }
    },
    {
      "name": "computeDeployment",
      "type": "Microsoft.Resources/deployments",
      "dependsOn": ["networkingDeployment"],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[uri(deployment().properties.templateLink.uri, 'modules/compute.json')]"
        }
      }
    }
  ]
}
```

**Benefits**:
- **Small Focused Templates**: Networking team owns networking.json
- **Composition**: Combine modules for complete solutions
- **Reusability**: Use networking.json in multiple projects
- **Maintainability**: Update compute.json, all solutions benefit
- **Testing**: Test each module independently

### 6. Simplified Orchestration

**The Problem**: Deploying 50 resources requires running 50 commands in correct order, handling dependencies manually.

**ARM Templates Solution**: Single command deploys all resources with automatic orchestration.

**Deployment**:
```bash
# Deploy complete infrastructure
az deployment group create \
  --resource-group myapp-rg \
  --template-file main.json \
  --parameters @prod.parameters.json
```

**What Happens** (automatic):
1. **Validation Phase**
   - Validates JSON syntax
   - Checks API versions exist
   - Verifies parameters meet constraints
   - Identifies circular dependencies
   - Fails before any resources created if errors exist

2. **Planning Phase**
   - Analyzes all resources and dependencies
   - Creates deployment graph
   - Identifies resources that can deploy in parallel
   - Calculates deployment order

3. **Execution Phase**
   - Deploys independent resources simultaneously:
     - Virtual Network (no dependencies)
     - Storage Account (no dependencies)
     - Public IP (no dependencies)
   - Waits for dependencies:
     - Network Interface waits for VNet and Public IP
   - Continues deployment:
     - Virtual Machine waits for NIC and Storage Account

4. **Monitoring Phase**
   - Tracks deployment progress
   - Captures detailed logs
   - Reports errors with specific resource failures
   - Provides deployment status in real-time

5. **Completion Phase**
   - Returns deployment outputs
   - Provides deployment ID for future reference
   - Logs deployment in Azure Activity Log

**Deployment Timeline Example**:
```
Time | Action
-----|-------
0:00 | Start deployment
0:10 | Validation complete (no errors)
0:15 | Create VNet, Storage, Public IP (parallel)
0:45 | VNet, Storage, Public IP complete
0:50 | Create NIC (waited for VNet and Public IP)
1:00 | NIC complete
1:05 | Create VM (waited for NIC and Storage)
3:00 | VM complete
3:05 | Deployment complete (total: 3 minutes 5 seconds)
```

**vs. Manual Process** (30-45 minutes with sequential operations)

### 7. Built-In Validation

**The Problem**: Deploy infrastructure, discover error 30 minutes into deployment, start over.

**ARM Templates Solution**: Validate before deploying, catch errors early.

**Validation Command**:
```bash
az deployment group validate \
  --resource-group myapp-rg \
  --template-file template.json \
  --parameters @parameters.json
```

**What Gets Validated**:
- JSON syntax correctness
- Template schema compliance
- Parameter constraints (minValue, maxValue, allowedValues)
- Resource types and API versions exist
- Required properties present
- Circular dependencies
- Resource naming rules
- Quota limits

**Example Validation Errors**:
```json
{
  "error": {
    "code": "InvalidTemplate",
    "message": "Deployment template validation failed: 'The template parameter 'vmSize' has an invalid value 'Standard_D2s_v5'. The value must be one of 'Standard_B2s,Standard_D2s_v3,Standard_D4s_v3'."
  }
}
```

**Benefits**:
- Catch errors in seconds instead of minutes
- Fix issues before deployment starts
- No partial deployments from validation errors
- Integrate into CI/CD for automated validation

### 8. Reduced Errors and Increased Reliability

**Comparison**:

| Error Type | Manual | Imperative Script | ARM Template |
|------------|--------|-------------------|--------------|
| Typos in names | ✗ High risk | ⚠️ Possible | ✓ Caught in review |
| Wrong SKU | ✗ High risk | ⚠️ Possible | ✓ allowedValues prevents |
| Wrong region | ✗ High risk | ⚠️ Possible | ✓ Parameter validation |
| Dependency issues | ✗ Deployment fails | ⚠️ Manual ordering | ✓ Automatic |
| Duplicate resources | ✗ Manual tracking | ⚠️ Must code checks | ✓ Idempotent |
| Partial failures | ✗ Inconsistent state | ✗ Requires cleanup | ✓ Automatic retry |
| Configuration drift | ✗ Inevitable | ⚠️ Possible | ✓ Complete mode fixes |

## Azure Quickstart Templates

**Repository**: https://github.com/Azure/azure-quickstart-templates

Microsoft maintains 400+ community-contributed templates for common scenarios.

**Categories**:
- **Compute**: Virtual machines, scale sets, AKS clusters
- **Networking**: Hub-spoke topologies, VPN gateways, load balancers
- **Storage**: Blob storage, file shares, data lakes
- **Databases**: SQL Server, PostgreSQL, Cosmos DB
- **Web**: App Service, Static Web Apps, Function Apps
- **Analytics**: Synapse, Data Factory, HDInsight
- **AI/ML**: Machine Learning, Cognitive Services
- **Security**: Key Vault, Security Center, Sentinel

**Example: WordPress on App Service**
```bash
# Clone repository
git clone https://github.com/Azure/azure-quickstart-templates.git

# Navigate to WordPress template
cd azure-quickstart-templates/quickstarts/microsoft.web/wordpress-app-service-mysql-inapp

# View README for template description
cat README.md

# Deploy
az deployment group create \
  --resource-group wordpress-rg \
  --template-file azuredeploy.json \
  --parameters siteName=mywordpresssite
```

**Benefits**:
- **Learning**: Study well-structured templates
- **Best Practices**: Follow Azure recommendations
- **Time Savings**: Don't start from scratch
- **Customization**: Fork and modify for your needs

## Bicep: Modern ARM Template Alternative

**Bicep** is a domain-specific language that compiles to ARM templates with cleaner syntax.

### Comparison

**JSON ARM Template**:
```json
{
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "2021-02-01",
  "name": "mystorageaccount",
  "location": "eastus",
  "sku": {
    "name": "Standard_LRS"
  },
  "kind": "StorageV2",
  "properties": {
    "supportsHttpsTrafficOnly": true,
    "minimumTlsVersion": "TLS1_2"
  }
}
```

**Bicep (Equivalent)**:
```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'mystorageaccount'
  location: 'eastus'
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
  }
}
```

**Bicep Advantages**:
- **Cleaner Syntax**: No quotes on property names, less verbose
- **Better IntelliSense**: IDE support excellent
- **Simpler Expressions**: No `[concat()]` needed
- **Type Safety**: Compile-time type checking
- **Same Power**: Compiles to ARM, all features available

**When to Use Bicep**:
- New projects (recommended by Microsoft)
- Teams preferring cleaner syntax
- Projects benefiting from enhanced IDE support

**When to Use ARM Templates**:
- Existing projects (continue using them)
- Integration with tools expecting JSON
- Generated templates from Azure Portal export

**Module 6** of this learning path covers Bicep in detail.

## Summary

ARM templates solve fundamental infrastructure management problems:

**Problems Solved**:
- ✗ Manual errors → ✓ Automated, validated deployments
- ✗ Configuration drift → ✓ Consistent, repeatable infrastructure
- ✗ Slow provisioning → ✓ Fast, parallel deployment
- ✗ No documentation → ✓ Self-documenting code
- ✗ No version control → ✓ Infrastructure as code
- ✗ Difficult rollback → ✓ Git revert to previous version

**Key Advantages**:
1. **Consistency**: Same template, all environments
2. **Complexity Management**: Automatic dependency resolution
3. **Infrastructure as Code**: Version control, review, test
4. **Reusability**: Parameters enable multi-environment templates
5. **Modularity**: Link small templates into complex solutions
6. **Simplified Deployment**: One command deploys everything
7. **Validation**: Catch errors before deployment
8. **Reliability**: Reduced errors, increased confidence

**Next Unit**: Explore template components—learn JSON structure, parameters, variables, functions, resources, and outputs that make ARM templates powerful and flexible.

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-azure-resources-using-azure-resource-manager-templates/2-why-use-azure-resource-manager-templates)
