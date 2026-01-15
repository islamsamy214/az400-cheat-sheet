# Understand Imperative versus Declarative Configuration

## Overview

One of the most fundamental decisions when implementing Infrastructure as Code and Configuration as Code is choosing between imperative and declarative approaches. This choice impacts not just how you write code, but how you think about infrastructure, how your team collaborates, and how reliably your systems behave. Understanding these two paradigms—their strengths, weaknesses, and appropriate use cases—enables you to select the right tool for each situation and avoid common pitfalls that plague infrastructure automation projects.

The distinction between imperative and declarative configuration parallels differences in programming paradigms. Imperative approaches (procedural programming) specify exact steps to achieve a goal: "do this, then do that, then do the other thing." Declarative approaches (functional programming) specify the desired end state: "I want this configuration to exist." The system figures out how to make it happen. Neither approach is inherently superior—each excels in different scenarios, and modern infrastructure management often combines both.

## Key Concepts

### Declarative Approach (Functional Programming Style)

**Core Principle**: Describe what you want, not how to get there.

Declarative configuration focuses on the desired end state. You specify resource properties and relationships, and the IaC tool determines what actions are necessary to achieve that state. The tool handles complexity like dependency ordering, error handling, and idempotency automatically.

**Real-World Analogy**: Ordering food at a restaurant

When you order a cheeseburger, you don't tell the kitchen: "First, take ground beef and form it into a patty. Then, heat a grill to 375°F. Place the patty on the grill for 4 minutes. Flip it. Cook for 3 more minutes..." You simply say "I want a cheeseburger" and the kitchen figures out how to make it. That's declarative.

**Code Example**:
```hcl
# Terraform (declarative)
resource "azurerm_virtual_machine" "web_server" {
  name                  = "web-vm-01"
  location              = "East US"
  resource_group_name   = azurerm_resource_group.main.name
  network_interface_ids = [azurerm_network_interface.main.id]
  size                  = "Standard_D2s_v3"
  
  storage_os_disk {
    name              = "web-vm-01-os-disk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Premium_LRS"
  }
  
  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
  
  os_profile {
    computer_name  = "web-vm-01"
    admin_username = "azureuser"
  }
  
  os_profile_linux_config {
    disable_password_authentication = true
    
    ssh_keys {
      path     = "/home/azureuser/.ssh/authorized_keys"
      key_data = file("~/.ssh/id_rsa.pub")
    }
  }
  
  tags = {
    Environment = "Production"
    ManagedBy   = "Terraform"
  }
}
```

**What This Code Declares**:
- "I want a virtual machine named web-vm-01"
- "It should be in East US region"
- "It should be size Standard_D2s_v3"
- "It should use Ubuntu 18.04 LTS image"
- "It should have these tags"

**What Terraform Handles Automatically**:
- Checking if VM already exists
- Creating VM if it doesn't exist
- Updating VM properties if they changed
- Ignoring request if VM exists with correct properties
- Managing dependencies (creates network interface before VM)
- Error handling and retries
- Reporting success or failure

**Visual Representation**:

```
[Desired State Code] → [IaC Tool] → [Actual Infrastructure]
         ↓               ↓                    ↓
   "I want this"    "I'll figure      "Resources match
    to exist"        out how to         desired state"
                    make it happen"
```

### Popular Declarative Tools

**Azure Resource Manager (ARM) Templates**
- JSON-based template files defining Azure resources
- Integrated natively with Azure platform
- Supports all Azure resource types
- Template validation before deployment

Example:
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "web-vm-01",
      "location": "East US",
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_D2s_v3"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS",
            "version": "latest"
          }
        }
      }
    }
  ]
}
```

**Bicep**
- Domain-specific language (DSL) for Azure resources
- Simpler syntax than ARM templates JSON
- Compiles to ARM templates for deployment
- Strong type checking and IntelliSense support

Example:
```bicep
resource webVm 'Microsoft.Compute/virtualMachines@2021-03-01' = {
  name: 'web-vm-01'
  location: 'East US'
  properties: {
    hardwareProfile: {
      vmSize: 'Standard_D2s_v3'
    }
    storageProfile: {
      imageReference: {
        publisher: 'Canonical'
        offer: 'UbuntuServer'
        sku: '18.04-LTS'
        version: 'latest'
      }
    }
  }
}
```

**Terraform (HashiCorp)**
- Multi-cloud IaC tool (Azure, AWS, GCP, and 100+ providers)
- Uses HashiCorp Configuration Language (HCL)
- State management tracks actual resource configurations
- Plan/apply workflow shows changes before execution

**Kubernetes Manifests**
- YAML-based definitions for containerized workloads
- Declarative specification of desired cluster state
- Kubernetes controllers continuously reconcile actual state to desired state
- GitOps workflows for managing Kubernetes resources

Example:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

### Imperative Approach (Procedural Programming Style)

**Core Principle**: Describe how to achieve the desired state step by step.

Imperative configuration specifies explicit instructions the system should execute in order. You control the exact sequence of operations, conditionals, loops, and error handling. This provides fine-grained control but requires more detailed code.

**Real-World Analogy**: Following a recipe

A recipe is imperative: "First, preheat oven to 350°F. Then, mix flour and sugar. Next, add eggs and beat for 2 minutes. Pour into pan. Bake for 35 minutes." Every step is explicit and ordered.

**Code Example**:
```bash
# Azure CLI (imperative)
# Step 1: Create resource group
az group create \
  --name my-resource-group \
  --location eastus

# Step 2: Create virtual network
az network vnet create \
  --resource-group my-resource-group \
  --name my-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name my-subnet \
  --subnet-prefix 10.0.1.0/24

# Step 3: Create network security group
az network nsg create \
  --resource-group my-resource-group \
  --name my-nsg

# Step 4: Create NSG rule allowing HTTP
az network nsg rule create \
  --resource-group my-resource-group \
  --nsg-name my-nsg \
  --name AllowHTTP \
  --priority 100 \
  --destination-port-ranges 80 \
  --protocol Tcp \
  --access Allow

# Step 5: Create public IP address
az network public-ip create \
  --resource-group my-resource-group \
  --name my-public-ip \
  --allocation-method Static

# Step 6: Create network interface
az network nic create \
  --resource-group my-resource-group \
  --name my-nic \
  --vnet-name my-vnet \
  --subnet my-subnet \
  --network-security-group my-nsg \
  --public-ip-address my-public-ip

# Step 7: Create virtual machine
az vm create \
  --resource-group my-resource-group \
  --name web-vm-01 \
  --location eastus \
  --size Standard_D2s_v3 \
  --image UbuntuLTS \
  --nics my-nic \
  --admin-username azureuser \
  --generate-ssh-keys

# Step 8: Install web server extension
az vm extension set \
  --resource-group my-resource-group \
  --vm-name web-vm-01 \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"apt-get update && apt-get install -y nginx"}'
```

**What This Code Specifies**:
- Exact sequence of operations
- Which resources to create and in what order
- Explicit error handling (none in this example - would fail at first error)
- No automatic dependency management (you must order correctly)

**Visual Representation**:

```
[Script] → [Step 1] → [Step 2] → [Step 3] → ... → [Final State]
    ↓         ↓          ↓          ↓               ↓
"Do this, Execute,  Execute,  Execute,        "State after
then this, then     then       then            all steps
then that"         continue   continue         complete"
```

### Imperative Characteristics

**Procedural Control Flow**

Imperative code includes programming constructs:

**Loops**:
```powershell
# PowerShell - Create 5 VMs
for ($i=1; $i -le 5; $i++) {
    $vmName = "web-vm-0$i"
    New-AzVM `
        -ResourceGroupName "my-rg" `
        -Name $vmName `
        -Location "East US" `
        -Size "Standard_B2s"
    
    Write-Host "Created $vmName"
}
```

**Conditionals**:
```python
# Python - Check if resource exists before creating
import subprocess

result = subprocess.run(
    ["az", "vm", "show", "--name", "web-vm-01", "--resource-group", "my-rg"],
    capture_output=True
)

if result.returncode != 0:
    # VM doesn't exist, create it
    subprocess.run([
        "az", "vm", "create",
        "--resource-group", "my-rg",
        "--name", "web-vm-01",
        "--image", "UbuntuLTS"
    ])
    print("Created VM")
else:
    print("VM already exists, skipping creation")
```

**Error Handling**:
```bash
# Bash - Handle errors and retry
for i in {1..3}; do
    if az vm create --name web-vm-01 --resource-group my-rg --image UbuntuLTS; then
        echo "VM created successfully"
        break
    else
        echo "Attempt $i failed, retrying..."
        sleep 10
    fi
done
```

**Explicit Dependency Ordering**

You must manually sequence operations based on dependencies:
```bash
# Must create these resources in correct order
# 1. Resource group (no dependencies)
az group create --name my-rg --location eastus

# 2. VNet (depends on resource group)
az network vnet create --resource-group my-rg --name my-vnet

# 3. Subnet (depends on VNet)
az network vnet subnet create --resource-group my-rg --vnet-name my-vnet --name my-subnet

# 4. NIC (depends on subnet)
az network nic create --resource-group my-rg --name my-nic --vnet-name my-vnet --subnet my-subnet

# 5. VM (depends on NIC)
az vm create --resource-group my-rg --name my-vm --nics my-nic
```

If you run these out of order, they fail. Declarative tools handle dependency ordering automatically.

### Popular Imperative Tools

**PowerShell with Azure PowerShell Modules**
- Powerful scripting language with object-oriented pipeline
- Rich Azure management capabilities through Az modules
- Excellent for complex automation logic
- Azure Automation integration for scheduled tasks

Example:
```powershell
# Complex conditional logic
$vmConfig = Get-AzVM -ResourceGroupName "my-rg" -Name "web-vm-01" -ErrorAction SilentlyContinue

if ($null -eq $vmConfig) {
    # VM doesn't exist, create it
    New-AzVM `
        -ResourceGroupName "my-rg" `
        -Name "web-vm-01" `
        -Location "East US" `
        -Size "Standard_D2s_v3" `
        -Image "UbuntuLTS"
} elseif ($vmConfig.HardwareProfile.VmSize -ne "Standard_D2s_v3") {
    # VM exists but wrong size, resize it
    $vmConfig.HardwareProfile.VmSize = "Standard_D2s_v3"
    Update-AzVM -ResourceGroupName "my-rg" -VM $vmConfig
    Restart-AzVM -ResourceGroupName "my-rg" -Name "web-vm-01"
} else {
    Write-Host "VM already exists with correct size"
}
```

**Azure CLI Scripts**
- Command-line interface for managing Azure resources
- Bash or PowerShell script automation
- Cross-platform support (Windows, Linux, macOS)
- Scriptable commands with JSON output

Example:
```bash
#!/bin/bash
# Script with error handling and validation

set -e  # Exit on first error

RESOURCE_GROUP="my-rg"
LOCATION="eastus"
VM_NAME="web-vm-01"

# Check if resource group exists
if ! az group show --name "$RESOURCE_GROUP" &>/dev/null; then
    echo "Creating resource group..."
    az group create --name "$RESOURCE_GROUP" --location "$LOCATION"
fi

# Create VM with retry logic
MAX_RETRIES=3
for i in $(seq 1 $MAX_RETRIES); do
    if az vm create \
        --resource-group "$RESOURCE_GROUP" \
        --name "$VM_NAME" \
        --image UbuntuLTS \
        --size Standard_D2s_v3 \
        --generate-ssh-keys; then
        echo "VM created successfully"
        break
    else
        if [ $i -eq $MAX_RETRIES ]; then
            echo "Failed to create VM after $MAX_RETRIES attempts"
            exit 1
        fi
        echo "Attempt $i failed, retrying in 10 seconds..."
        sleep 10
    fi
done
```

**Ansible Playbooks**
- Configuration management tool that can be used declaratively or imperatively
- YAML-based playbooks define tasks to execute
- Agentless architecture using SSH or WinRM
- Large module library for various platforms

Example (imperative style):
```yaml
---
- name: Deploy web application
  hosts: web_servers
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes
    
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    - name: Copy SSL certificate
      copy:
        src: files/ssl-cert.pem
        dest: /etc/nginx/ssl/cert.pem
        mode: '0600'
    
    - name: Copy Nginx configuration
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: reload nginx
    
    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
  
  handlers:
    - name: reload nginx
      service:
        name: nginx
        state: reloaded
```

**Python/Bash Scripts**
- General-purpose programming languages
- Azure SDKs (Python: azure-mgmt-* packages)
- Complete flexibility for complex logic
- Integration with external systems and APIs

Example (Python):
```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.network import NetworkManagementClient
from azure.mgmt.resource import ResourceManagementClient

credential = DefaultAzureCredential()
subscription_id = "your-subscription-id"

# Initialize clients
compute_client = ComputeManagementClient(credential, subscription_id)
network_client = NetworkManagementClient(credential, subscription_id)
resource_client = ResourceManagementClient(credential, subscription_id)

# Step 1: Create resource group
resource_client.resource_groups.create_or_update(
    "my-rg",
    {"location": "eastus"}
)

# Step 2: Create VNet
vnet_params = {
    "location": "eastus",
    "address_space": {
        "address_prefixes": ["10.0.0.0/16"]
    }
}
network_client.virtual_networks.begin_create_or_update(
    "my-rg",
    "my-vnet",
    vnet_params
).result()

# Step 3: Create subnet
subnet_params = {
    "address_prefix": "10.0.1.0/24"
}
network_client.subnets.begin_create_or_update(
    "my-rg",
    "my-vnet",
    "my-subnet",
    subnet_params
).result()

# ... additional steps to create NIC, VM, etc.
```

## Choosing the Right Approach

### When to Use Declarative

**Advantages**:

1. **Easier to Read and Understand**
   - See what infrastructure exists, not implementation details
   - Clear resource definitions without procedural noise
   - New team members understand infrastructure quickly

   Example comparison:
   ```hcl
   # Declarative - easy to understand
   resource "azurerm_virtual_machine" "web" {
     count = 5
     name  = "web-vm-${count.index}"
     size  = "Standard_D2s_v3"
     # ...
   }
   ```
   
   vs.
   
   ```bash
   # Imperative - procedural details
   for i in {0..4}; do
     az vm create \
       --name "web-vm-$i" \
       --size Standard_D2s_v3 \
       # ...
   done
   ```

2. **Idempotent by Design**
   - Run same template repeatedly without errors
   - Tool automatically checks if resources exist
   - Updates only what changed
   - Safe to re-run after failures

3. **Order-Independent**
   - Tool figures out dependencies automatically
   - No need to manually sequence operations
   - Parallel resource creation where possible
   - Less error-prone for complex infrastructures

4. **Less Code to Maintain**
   - Shorter, more concise definitions
   - No explicit error handling needed (built into tool)
   - No loops or conditionals for common patterns
   - Framework handles complexity

5. **Better for Team Collaboration**
   - Easier for team members to review pull requests
   - Clear intent without implementation details
   - Standardized patterns across team
   - Less room for creative (but inconsistent) solutions

**Best For**:

- **Provisioning Cloud Infrastructure**: Creating VMs, networks, storage, databases
- **Deploying Containerized Applications**: Kubernetes manifests, Docker Compose files
- **Teams New to IaC**: Simpler learning curve, fewer concepts
- **Infrastructure That Changes Infrequently**: Stable environments benefit from declarative simplicity

**Example Use Case**: Deploy web application infrastructure (VNet, load balancer, VM scale set, database)

```hcl
# Terraform - declarative approach
module "web_application" {
  source = "./modules/web-app"
  
  environment         = "production"
  instance_count      = 5
  vm_size             = "Standard_D2s_v3"
  database_tier       = "BusinessCritical"
  enable_auto_scaling = true
}
```

Tool handles all details: creates resources in correct order, configures auto-scaling, sets up load balancer, creates database with specified tier.

### When to Use Imperative

**Advantages**:

1. **Fine-Grained Control**
   - Specify exactly how things should happen
   - Control execution order explicitly
   - Custom error handling logic
   - Retry mechanisms tailored to your needs

2. **Easier for Complex Logic**
   - Conditional branching based on runtime checks
   - Loops for dynamic scenarios
   - Integration with external systems
   - Data transformations and processing

3. **Familiar to Developers**
   - Looks like regular programming
   - Use familiar constructs (if/else, for loops, functions)
   - Leverage programming skills
   - Rich debugging tools available

4. **Better for Dynamic Scenarios**
   - Make decisions based on current state
   - Query resources and act on results
   - Complex workflows with multiple decision points
   - Integration with APIs and external systems

**Best For**:

- **Complex Configuration Workflows**: Multi-step processes with conditional logic
- **Migrating Existing Scripts**: Gradual transition from manual scripts to automation
- **Precise Control Requirements**: Situations demanding exact execution order
- **External System Integration**: Workflows involving non-cloud systems, databases, third-party APIs

**Example Use Case**: Database migration with validation and rollback

```python
# Python - imperative approach for complex workflow
def migrate_database():
    # Step 1: Backup current database
    backup_name = f"backup-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
    create_backup(backup_name)
    
    # Step 2: Verify backup completed successfully
    if not verify_backup(backup_name):
        raise Exception("Backup verification failed")
    
    # Step 3: Put application in maintenance mode
    set_maintenance_mode(True)
    
    try:
        # Step 4: Run migration scripts
        for script in get_migration_scripts():
            execute_sql_script(script)
            
            # Verify each script's result
            if not verify_migration_step(script):
                raise Exception(f"Migration script {script} failed")
        
        # Step 5: Run validation tests
        if not run_validation_tests():
            raise Exception("Post-migration validation failed")
        
        # Step 6: Take application out of maintenance mode
        set_maintenance_mode(False)
        
        print("Migration completed successfully")
    
    except Exception as e:
        # Rollback on any error
        print(f"Migration failed: {e}")
        print("Rolling back to backup...")
        restore_backup(backup_name)
        set_maintenance_mode(False)
        raise
```

This workflow requires imperative approach because:
- Conditional logic at each step
- Complex error handling and rollback
- External system interaction (database, application)
- Sequential dependencies that can't be parallelized

### Hybrid Approach

Most real-world infrastructure management combines both approaches, using each where it excels:

**Common Pattern**: Declarative for provisioning, imperative for configuration

```hcl
# Step 1: Provision infrastructure (Terraform - declarative)
resource "azurerm_virtual_machine" "web" {
  count = 3
  name  = "web-vm-${count.index}"
  # ... VM configuration
}

# Output VM IP addresses for next step
output "vm_ips" {
  value = azurerm_virtual_machine.web[*].private_ip_address
}
```

```yaml
# Step 2: Configure VMs (Ansible - imperative)
---
- name: Configure web servers
  hosts: "{{ lookup('terraform', 'vm_ips') }}"
  tasks:
    - name: Update system packages
      apt:
        update_cache: yes
        upgrade: dist
    
    - name: Install application dependencies
      apt:
        name:
          - nginx
          - python3
          - python3-pip
        state: present
    
    - name: Deploy application code
      synchronize:
        src: /local/app/
        dest: /var/www/app/
    
    - name: Configure and start application
      template:
        src: app-config.j2
        dest: /etc/app/config.yaml
      notify: restart application
```

**Benefits of Hybrid Approach**:
- Leverage strengths of each paradigm
- Declarative creates resources reliably
- Imperative handles complex configuration logic
- Clear separation of concerns
- Team specialization (infrastructure team vs. application team)

**Example Architecture**:
```
┌─────────────────────────────────────────────────────┐
│               CI/CD Pipeline                         │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Stage 1: Infrastructure (Declarative)               │
│  ├─ Terraform provisions VMs, networks, databases   │
│  └─ Outputs: IP addresses, connection strings       │
│                                                      │
│  Stage 2: Configuration (Imperative)                 │
│  ├─ Ansible installs software on VMs                │
│  └─ Uses Terraform outputs as inventory              │
│                                                      │
│  Stage 3: Application Deployment (Mix)               │
│  ├─ Helm charts deploy containers (declarative)     │
│  └─ Post-deployment scripts run tests (imperative)  │
│                                                      │
└─────────────────────────────────────────────────────┘
```

## Real-World Scenarios

### Scenario 1: Multi-Region Deployment (Declarative Wins)

**Requirement**: Deploy identical infrastructure across 5 Azure regions.

**Declarative Approach (Terraform)**:
```hcl
variable "regions" {
  default = ["eastus", "westus", "centralus", "northeurope", "southeastasia"]
}

module "regional_deployment" {
  source = "./modules/regional-infrastructure"
  
  for_each = toset(var.regions)
  region   = each.value
  
  # Same configuration deployed to all regions
  vm_count      = 3
  vm_size       = "Standard_D2s_v3"
  database_tier = "GeneralPurpose"
}
```

**Why Declarative Is Better**:
- Single module defines infrastructure
- `for_each` deploys to all regions automatically
- Terraform manages dependencies across all regions
- Easy to add/remove regions
- Consistent configuration guaranteed

**Imperative Approach Would Require**:
- Loop through regions array
- Manually create each resource in sequence
- Handle errors for each region separately
- Complex dependency management
- Much more code and potential for errors

### Scenario 2: Conditional Database Migration (Imperative Wins)

**Requirement**: Migrate database only if schema version is specific version, with rollback capability.

**Imperative Approach (PowerShell)**:
```powershell
# Check current database schema version
$currentVersion = (Invoke-Sqlcmd -Query "SELECT version FROM schema_info" -ServerInstance $dbServer -Database $dbName).version

if ($currentVersion -eq "2.5.0") {
    Write-Host "Current version is 2.5.0, proceeding with migration"
    
    # Backup database first
    $backupFile = "C:\backups\db-backup-$(Get-Date -Format 'yyyyMMdd-HHmmss').bak"
    Backup-SqlDatabase -ServerInstance $dbServer -Database $dbName -BackupFile $backupFile
    
    try {
        # Run migration scripts
        Invoke-Sqlcmd -InputFile "migrations\2.5.0-to-2.6.0.sql" -ServerInstance $dbServer -Database $dbName
        
        # Verify migration
        $newVersion = (Invoke-Sqlcmd -Query "SELECT version FROM schema_info" -ServerInstance $dbServer -Database $dbName).version
        
        if ($newVersion -ne "2.6.0") {
            throw "Migration completed but version not updated correctly"
        }
        
        Write-Host "Migration successful"
    }
    catch {
        Write-Host "Migration failed: $_"
        Write-Host "Restoring from backup..."
        Restore-SqlDatabase -ServerInstance $dbServer -Database $dbName -BackupFile $backupFile -ReplaceDatabase
        throw
    }
} else {
    Write-Host "Current version is $currentVersion, not 2.5.0. Skipping migration."
}
```

**Why Imperative Is Better**:
- Complex conditional logic (check version before proceeding)
- Runtime decision based on database state
- Custom error handling with rollback
- Step-by-step validation at each stage

**Declarative Approach Would Struggle With**:
- Conditional logic based on runtime state
- Multi-step workflows with dependencies
- Custom error handling and rollback logic

### Scenario 3: Hybrid Approach for Complete Application Stack

**Requirement**: Deploy infrastructure, configure servers, and deploy application with health checks.

**Stage 1: Infrastructure (Terraform - Declarative)**
```hcl
# infrastructure/main.tf
module "networking" {
  source = "./modules/networking"
  vnet_address_space = "10.0.0.0/16"
}

module "compute" {
  source = "./modules/compute"
  vm_count = 3
  subnet_id = module.networking.subnet_id
}

module "database" {
  source = "./modules/database"
  subnet_id = module.networking.subnet_id
  tier = "GeneralPurpose"
}

output "vm_ips" {
  value = module.compute.private_ips
}

output "database_connection_string" {
  value = module.database.connection_string
  sensitive = true
}
```

**Stage 2: Configuration (Ansible - Imperative)**
```yaml
# configuration/playbook.yml
---
- name: Configure application servers
  hosts: "{{ vm_ips }}"
  vars:
    db_connection: "{{ database_connection_string }}"
  tasks:
    - name: Install prerequisites
      apt:
        name: ["nginx", "python3", "python3-pip"]
        state: present
    
    - name: Deploy application code
      synchronize:
        src: ../application/
        dest: /var/www/app/
    
    - name: Configure application
      template:
        src: app-config.j2
        dest: /etc/app/config.yaml
      vars:
        database_connection: "{{ db_connection }}"
    
    - name: Start application service
      systemd:
        name: myapp
        state: started
        enabled: yes
```

**Stage 3: Validation (Python - Imperative)**
```python
# validation/health_check.py
import requests
import time
import sys

def wait_for_healthy(endpoints, timeout=300, interval=10):
    """Wait for all endpoints to return 200 OK"""
    start_time = time.time()
    
    while time.time() - start_time < timeout:
        all_healthy = True
        
        for endpoint in endpoints:
            try:
                response = requests.get(f"{endpoint}/health", timeout=5)
                if response.status_code != 200:
                    print(f"❌ {endpoint} returned {response.status_code}")
                    all_healthy = False
                else:
                    print(f"✓ {endpoint} is healthy")
            except requests.RequestException as e:
                print(f"❌ {endpoint} not reachable: {e}")
                all_healthy = False
        
        if all_healthy:
            print("✓ All endpoints healthy")
            return True
        
        time.sleep(interval)
    
    print(f"✗ Timeout waiting for healthy endpoints")
    return False

if __name__ == "__main__":
    endpoints = sys.argv[1:]  # Passed from Terraform outputs
    if not wait_for_healthy(endpoints):
        sys.exit(1)
```

**CI/CD Pipeline Tying It All Together**:
```yaml
# azure-pipelines.yml
stages:
  - stage: Infrastructure
    jobs:
      - job: Terraform
        steps:
          - script: terraform init
          - script: terraform plan -out=tfplan
          - script: terraform apply tfplan
          - script: |
              echo "##vso[task.setvariable variable=vm_ips;isOutput=true]$(terraform output -json vm_ips)"

  - stage: Configuration
    dependsOn: Infrastructure
    variables:
      vm_ips: $[ stageDependencies.Infrastructure.Terraform.outputs['vm_ips'] ]
    jobs:
      - job: Ansible
        steps:
          - script: ansible-playbook -i "$(vm_ips)" playbook.yml

  - stage: Validation
    dependsOn: Configuration
    variables:
      vm_ips: $[ stageDependencies.Infrastructure.Terraform.outputs['vm_ips'] ]
    jobs:
      - job: HealthCheck
        steps:
          - script: python health_check.py $(vm_ips)
```

**Benefits of This Hybrid Approach**:
- Terraform handles infrastructure (its strength - declarative provisioning)
- Ansible handles configuration (good at both declarative and imperative)
- Python handles validation (requires imperative logic for retries, timeouts)
- Each tool used where it excels
- Clear separation of concerns
- Pipeline orchestrates entire workflow

## Assessment Questions

### Question 1: Declarative vs. Imperative Identification

**Question**: Which of the following code snippets represents a declarative approach?

A)
```bash
az group create --name my-rg --location eastus
az vm create --resource-group my-rg --name my-vm --image UbuntuLTS
```

B)
```hcl
resource "azurerm_virtual_machine" "my_vm" {
  name                = "my-vm"
  location            = "eastus"
  resource_group_name = azurerm_resource_group.my_rg.name
  size                = "Standard_B2s"
}
```

C)
```python
for i in range(5):
    create_vm(f"vm-{i}", "eastus", "Standard_B2s")
```

D)
```powershell
if (Get-AzVM -Name "my-vm" -ErrorAction SilentlyContinue) {
    Write-Host "VM exists"
} else {
    New-AzVM -Name "my-vm" -Location "eastus"
}
```

**Answer**: B

**Explanation**: Option B (Terraform/HCL) is declarative—it declares desired state (resource should exist with these properties) without specifying how to create it. Options A, C, D are imperative—they specify step-by-step instructions. A uses sequential CLI commands. C uses a loop. D uses conditional logic. All are procedural approaches.

### Question 2: Tool Selection

**Question**: Your team needs to deploy identical infrastructure across 10 Azure regions. Most team members are new to IaC. Which approach and tool combination is most appropriate?

A) Imperative - PowerShell scripts with loops  
B) Declarative - Terraform with modules and for_each  
C) Imperative - Azure CLI scripts  
D) Declarative - Manually clicking through portal 10 times

**Answer**: B

**Explanation**: Declarative Terraform with modules excels at multi-region deployment. Single module deployed to multiple regions using `for_each`. Easier for teams new to IaC (simpler than scripting). Options A and C (imperative scripts) require more code and are error-prone. Option D isn't IaC at all—manual portal clicking defeats the purpose.

### Question 3: Idempotency

**Question**: You run this Azure CLI script twice:
```bash
az group create --name my-rg --location eastus
az vm create --resource-group my-rg --name my-vm --image UbuntuLTS
```

What happens on the second execution?

A) Script succeeds without changes (idempotent)  
B) Script fails because resource group already exists  
C) Script creates duplicate resources  
D) Script fails because VM already exists

**Answer**: D

**Explanation**: Imperative Azure CLI commands are not automatically idempotent. Second execution attempts to create VM that already exists, causing error. Declarative tools (Terraform, ARM) handle this automatically—check if resource exists and skip creation if it does. To make imperative scripts idempotent requires explicit checks (like in Question 1, Option D).

### Question 4: Complex Logic

**Question**: You need to migrate a database with these requirements:
- Check current schema version
- Backup before migration
- Run migration scripts in specific order
- Verify each step
- Rollback on any failure
- Custom logging to external system

Which approach is most appropriate?

A) Declarative - Terraform  
B) Declarative - ARM templates  
C) Imperative - PowerShell or Python script  
D) Declarative - Bicep

**Answer**: C

**Explanation**: This scenario requires complex conditional logic, error handling, rollback capability, and external system integration—all strengths of imperative approaches. PowerShell or Python provide necessary control flow, exception handling, and API integration. Declarative tools (A, B, D) aren't designed for complex multi-step workflows with runtime decisions.

### Question 5: Hybrid Approach

**Question**: Your organization wants to provision Azure infrastructure and configure applications on VMs. Which combination makes most sense?

A) Use only Terraform for everything  
B) Use only Ansible for everything  
C) Use Terraform for infrastructure provisioning, Ansible for application configuration  
D) Use ARM templates for infrastructure, PowerShell for configuration

**Answer**: C

**Explanation**: Terraform (declarative) excels at infrastructure provisioning—creating VMs, networks, storage. Ansible (can be declarative or imperative) excels at configuration management—installing software, configuring applications. This hybrid approach leverages each tool's strengths. Options A and B (using single tool for everything) don't optimize for strengths. Option D works but Terraform is more popular and multi-cloud capable.

## Summary

Choosing between imperative and declarative approaches fundamentally shapes how you implement Infrastructure as Code and Configuration as Code. Declarative approaches excel at provisioning standardized infrastructure, offering simplicity, idempotency by design, and automatic dependency management. Imperative approaches provide fine-grained control for complex workflows requiring conditional logic, custom error handling, and runtime decisions.

**Key Takeaways**:

1. **Declarative**: Specify what you want (desired state), tool figures out how. Best for infrastructure provisioning, standardized configurations, teams new to IaC.

2. **Imperative**: Specify how to achieve result (step-by-step). Best for complex workflows, conditional logic, external system integration.

3. **Tool Selection**: Choose based on task requirements—Terraform/ARM/Bicep for infrastructure, PowerShell/Python/Ansible for complex configuration.

4. **Idempotency**: Declarative tools are idempotent by default. Imperative scripts require explicit checks to be idempotent.

5. **Hybrid Approach**: Most organizations use both—declarative for provisioning, imperative for complex configuration and workflows.

6. **No Single "Best" Approach**: Choose based on team skills, task complexity, and organizational needs. Modern infrastructure management often combines both paradigms.

Understanding these approaches and their trade-offs enables you to select the right tool for each scenario, avoiding common pitfalls and leveraging each paradigm's strengths.

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-infrastructure-code-configuration-management/4-understand-imperative-versus-declarative-configuration)
