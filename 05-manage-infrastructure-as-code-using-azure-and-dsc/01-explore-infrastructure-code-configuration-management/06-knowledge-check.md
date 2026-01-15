# Module Assessment - Knowledge Check

## Overview

This knowledge check evaluates your understanding of the core concepts covered in this module: Infrastructure as Code (IaC), Configuration Management, Declarative vs. Imperative approaches, and Idempotency. These assessment questions test both conceptual understanding and practical application of IaC principles critical for the AZ-400 exam.

Take time to consider each question carefully. The detailed explanations provided will reinforce key concepts and clarify common misconceptions.

## Assessment Questions

### Question 1: Infrastructure as Code Fundamentals

**You need to deploy consistent development, staging, and production environments in Azure. The environments must be identical except for VM sizes and instance counts. Manual deployments have resulted in configuration drift between environments.**

**What should you do?**

A) Create detailed documentation specifying exact configuration steps for each environment  
B) Take snapshots of production environment and restore to dev/staging  
C) Define infrastructure in code using ARM templates or Terraform, parameterize environment-specific values  
D) Create a PowerShell script that prompts for environment name and manually adjusts settings

**Correct Answer**: C

**Explanation**:

**Why C is correct**: Infrastructure as Code (IaC) solves configuration drift by defining infrastructure in version-controlled code rather than manual processes. By using ARM templates, Terraform, or Bicep:
- Infrastructure is defined declaratively in code
- Environment-specific values (VM sizes, counts) are parameterized
- Same code deploys all environments with different parameters
- Version control tracks all changes
- Deployments are automated and repeatable
- Configuration drift eliminated (environments always match code)

Example:
```hcl
# Terraform with parameterized environments
resource "azurerm_linux_virtual_machine" "app" {
  name                = "${var.environment}-app-vm"
  size                = var.vm_size  # Different per environment
  resource_group_name = azurerm_resource_group.main.name
  # ... configuration
}

# Variables per environment:
# Dev:        vm_size = "Standard_B2s"
# Staging:    vm_size = "Standard_D2s_v3"
# Production: vm_size = "Standard_D4s_v3"
```

**Why other answers are wrong**:

**A (Documentation)**: Documentation-based processes are prone to:
- Human error during manual execution
- Documentation becoming outdated
- Inconsistent interpretation of steps
- Configuration drift over time
- No automation or enforcement

**B (Snapshots)**: Using snapshots:
- Doesn't capture infrastructure (networking, storage, services)
- Requires manual restoration process
- Snapshot creation itself is manual
- Doesn't parameterize environment differences
- Not version controlled or repeatable

**D (PowerShell script with prompts)**: Interactive scripts:
- Still require manual input (prone to errors)
- Don't enforce consistency
- Hard to integrate with automation pipelines
- Not truly IaC (not declarative or version controlled)
- Manual adjustments defeat purpose of automation

### Question 2: Configuration Management vs. Provisioning

**Your team manages 50 Azure VMs running a web application. You need to:**
- Ensure specific application dependencies are installed
- Maintain correct application configuration files
- Automatically remediate configuration drift
- Update configurations without recreating VMs

**Which approach best meets these requirements?**

A) Use Azure Resource Manager templates to manage VM configuration  
B) Implement configuration management with Ansible or PowerShell DSC  
C) Create custom VM images with all configurations pre-installed  
D) Use Azure CLI scripts to check and update VM settings

**Correct Answer**: B

**Explanation**:

**Why B is correct**: Configuration management tools (Ansible, PowerShell DSC, Chef, Puppet) are designed specifically for this scenario:

**Configuration Management Strengths**:
- Manages software and configuration on existing VMs (doesn't recreate VMs)
- Ensures packages installed correctly
- Maintains configuration files in desired state
- Detects and remediates configuration drift automatically
- Runs continuously or on schedule to enforce compliance
- Idempotent operations safe to run repeatedly

**PowerShell DSC Example**:
```powershell
Configuration WebServerConfig {
    Node "web-*" {
        # Ensure IIS installed
        WindowsFeature IIS {
            Name = "Web-Server"
            Ensure = "Present"
        }
        
        # Ensure app deployed
        File ApplicationFiles {
            SourcePath = "\\fileserver\apps\webapp"
            DestinationPath = "C:\inetpub\wwwroot"
            Recurse = $true
            Ensure = "Present"
        }
        
        # Ensure config file correct
        Script ConfigFile {
            GetScript = { ... }
            SetScript = { ... }
            TestScript = { ... }
        }
    }
}

# DSC continuously monitors and corrects drift
```

**Drift Remediation**: DSC/Ansible agents check configuration every 15-30 minutes. If drift detected (manual change, unauthorized edit), they automatically correct it to match desired state.

**Why other answers are wrong**:

**A (ARM templates)**: ARM templates are for infrastructure provisioning, not configuration management:
- ARM creates/updates infrastructure (VMs, networks, storage)
- Doesn't manage software installation or configuration files on VMs
- Doesn't detect or remediate drift in VM configuration
- Changes typically require VM recreation or redeployment
- Not designed for ongoing configuration management

**C (Custom VM images)**: Pre-configured images:
- Configuration "baked" into image at build time
- Updating configuration requires new image creation and VM redeployment
- No drift remediation (changes persist until VM replaced)
- Inflexible for frequent configuration changes
- Large image management overhead
- Best for immutable infrastructure, not configuration management

**D (Azure CLI scripts)**: Azure CLI manages Azure resources:
- Controls Azure services (VMs, networks, storage accounts)
- Doesn't execute code inside VMs
- No built-in drift detection or remediation
- Requires manual scheduling and execution
- Not designed for configuration management

### Question 3: Declarative vs. Imperative Configuration

**Your organization needs to deploy Azure infrastructure for a new application. The infrastructure team has experience with both Terraform (declarative) and Azure CLI (imperative). The infrastructure will be:**
- Complex (multiple VNets, subnets, NSGs, VMs, databases)
- Updated frequently (weekly changes)
- Managed by multiple team members
- Deployed to multiple environments (dev, staging, production)

**Which approach should you recommend and why?**

A) Azure CLI because it provides more control over execution order  
B) Terraform because declarative configuration is idempotent and manages state  
C) Azure CLI because it has better Azure integration  
D) Terraform because it's faster to execute

**Correct Answer**: B

**Explanation**:

**Why B is correct**: Declarative tools (Terraform, ARM templates, Bicep) are superior for complex infrastructure management:

**Declarative Advantages for Complex Infrastructure**:

1. **State Management**: Terraform maintains state file tracking all resources
   - Knows what exists, what changed, what to update
   - Multiple team members work safely (state prevents conflicts)
   - Can detect drift (manual changes outside Terraform)

2. **Dependency Management**: Automatically handles resource dependencies
   ```hcl
   # Terraform determines creation order automatically
   resource "azurerm_resource_group" "main" { ... }
   resource "azurerm_virtual_network" "main" {
     resource_group_name = azurerm_resource_group.main.name  # Dependency
   }
   resource "azurerm_subnet" "app" {
     virtual_network_name = azurerm_virtual_network.main.name  # Dependency
   }
   # Terraform creates in correct order: RG → VNet → Subnet
   ```

3. **Idempotent by Design**: Running `terraform apply` repeatedly:
   - First run: Creates all resources
   - Second run: No changes (already exists)
   - After manual change: Detects drift, corrects it
   - After code update: Updates only changed resources

4. **Change Planning**: `terraform plan` shows exactly what will change before applying
   - Prevents surprises
   - Enables code review of infrastructure changes
   - Shows diff between current and desired state

5. **Multi-Environment Management**:
   ```hcl
   # Same code, different workspaces
   terraform workspace select dev
   terraform apply -var-file="dev.tfvars"
   
   terraform workspace select production
   terraform apply -var-file="production.tfvars"
   ```

**Real-World Example**:
```hcl
# Declarative: Define desired state
resource "azurerm_network_security_group" "web" {
  name                = "web-nsg"
  location            = "East US"
  resource_group_name = "production-rg"
  
  security_rule {
    name                       = "AllowHTTPS"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "443"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

# Run 1: Creates NSG with rule
# Run 2: No changes (already correct)
# Manual change (admin adds port 80 rule): Run 3 removes it (not in code)
# Code change (add port 22 rule): Run 4 adds new rule, keeps existing
```

**Why other answers are wrong**:

**A (CLI for control over execution order)**:
- Manual execution order management is error-prone
- Declarative tools handle dependencies automatically
- Explicit ordering doesn't scale for complex infrastructure
- Creates maintenance burden as infrastructure grows

**C (CLI for better Azure integration)**:
- Terraform has excellent Azure provider (azurerm)
- Terraform actually provides better multi-cloud support
- CLI integration doesn't address state management, idempotency
- Azure also provides declarative ARM templates/Bicep for complex scenarios

**D (Terraform for speed)**:
- Speed is not the primary advantage
- Both can be fast depending on operations
- Real advantages: idempotency, state management, change planning
- Execution speed shouldn't drive architectural decisions

### Question 4: Idempotency in Practice

**You're writing a deployment script that creates Azure resources. The script must:**
- Work correctly when run for the first time
- Work correctly when run repeatedly
- Recover from partial failures (network interruption, timeout)
- Not create duplicate resources

**Which script design achieves these requirements?**

A)
```bash
az group create --name my-rg --location eastus
az vm create --name vm-01 --resource-group my-rg --image UbuntuLTS
az vm create --name vm-02 --resource-group my-rg --image UbuntuLTS
```

B)
```bash
az group create --name my-rg --location eastus || true
az vm create --name vm-01 --resource-group my-rg --image UbuntuLTS || true
az vm create --name vm-02 --resource-group my-rg --image UbuntuLTS || true
```

C)
```bash
if ! az group show --name my-rg &>/dev/null; then
    az group create --name my-rg --location eastus
fi

if ! az vm show --name vm-01 --resource-group my-rg &>/dev/null; then
    az vm create --name vm-01 --resource-group my-rg --image UbuntuLTS
fi

if ! az vm show --name vm-02 --resource-group my-rg &>/dev/null; then
    az vm create --name vm-02 --resource-group my-rg --image UbuntuLTS
fi
```

D) All approaches work equally well

**Correct Answer**: C

**Explanation**:

**Why C is correct**: This script implements idempotency through explicit state checking:

**How It Works**:
```bash
# First run (fresh environment):
if ! az group show --name my-rg &>/dev/null; then  # Check: RG doesn't exist
    az group create --name my-rg --location eastus  # Action: Create RG ✓
fi

if ! az vm show --name vm-01 --resource-group my-rg &>/dev/null; then  # Check: VM doesn't exist
    az vm create --name vm-01 --resource-group my-rg --image UbuntuLTS  # Action: Create VM ✓
fi

# Second run (everything exists):
if ! az group show --name my-rg &>/dev/null; then  # Check: RG exists
    # Skipped - RG already exists
fi

if ! az vm show --name vm-01 --resource-group my-rg &>/dev/null; then  # Check: VM exists
    # Skipped - VM already exists
fi

# Partial failure recovery:
# Network fails after creating vm-01 but before vm-02
# Re-run script:
# - Checks RG: exists, skip creation
# - Checks vm-01: exists, skip creation
# - Checks vm-02: doesn't exist, creates it ✓
# Successfully completes deployment!
```

**Idempotency Properties**:
- ✅ Works on first run (creates everything)
- ✅ Works on repeated runs (skips existing resources)
- ✅ Recovers from partial failures (creates only missing resources)
- ✅ No duplicate resources (checks before creating)
- ✅ No errors from resources already existing

**Why other answers are wrong**:

**A (No error handling)**:
```bash
# First run: ✓ Succeeds (creates all resources)
# Second run: ✗ FAILS at first command
az group create --name my-rg --location eastus
# Error: Resource group 'my-rg' already exists
# Script aborts, never creates VMs

# Partial failure:
# Network fails after creating RG
# Re-run: ✗ FAILS immediately (RG exists)
# Cannot recover from partial failure
```

**Not idempotent**: Fails when resources already exist.

**B (Suppressing all errors)**:
```bash
az group create --name my-rg --location eastus || true  # Ignores all errors
az vm create --name vm-01 --resource-group my-rg --image UbuntuLTS || true
az vm create --name vm-02 --resource-group my-rg --image UbuntuLTS || true
```

**Problems**:
- Masks real errors: `|| true` hides actual failures
- If RG creation fails due to auth issue, script continues silently
- VM creation fails (no RG), but `|| true` hides failure
- Script reports success even when nothing was created
- No visibility into what succeeded vs. failed

**Example Failure**:
```bash
# User has no permissions
az group create --name my-rg --location eastus || true
# Fails silently (permission denied), continues

az vm create --name vm-01 --resource-group my-rg --image UbuntuLTS || true
# Fails silently (no resource group), continues

# Script exits with success code
# User believes deployment succeeded
# Actually: nothing was created!
```

**D (All equally good)**: Clearly false because:
- A fails on second run
- B masks real errors
- Only C is truly idempotent and reliable

### Question 5: IaC Best Practices

**Your organization is adopting Infrastructure as Code. You need to establish practices that ensure:**
- Infrastructure changes are reviewed before deployment
- Previous infrastructure versions can be restored if needed
- Multiple team members can collaborate without conflicts
- Audit trail of all infrastructure changes exists

**Which practices should you implement?**

A) Store IaC code in version control, use pull requests for reviews, implement CI/CD pipelines for deployment  
B) Store IaC code on shared network drive, email code to team lead for review before running  
C) Keep IaC code on local machines, use screen sharing for reviews, manually run scripts  
D) Use cloud shell to write and execute IaC code, document changes in wiki

**Correct Answer**: A

**Explanation**:

**Why A is correct**: This implements infrastructure as code best practices comprehensively:

**1. Version Control (Git)**:
```bash
# Store all IaC code in Git repository
git clone https://github.com/company/infrastructure.git
cd infrastructure/

# Create feature branch for changes
git checkout -b feature/add-production-database

# Make infrastructure changes
vim terraform/database.tf

# Commit with descriptive message
git add terraform/database.tf
git commit -m "Add production PostgreSQL database with HA configuration"

# Push for review
git push origin feature/add-production-database
```

**Benefits**:
- Complete change history (who changed what, when, why)
- Can revert to any previous version instantly
- Branching enables parallel development
- Audit trail for compliance

**2. Pull Request Reviews**:
```yaml
# GitHub Pull Request process
# 1. Developer creates PR from feature branch
# 2. Automated checks run (linting, security scanning, terraform plan)
# 3. Team reviews changes
# 4. Terraform plan output visible in PR
# 5. Requires 2 approvals before merge
# 6. Merge to main triggers deployment
```

**Review Process**:
```
Developer submits PR:
├── Automated Checks
│   ├── terraform fmt -check (formatting)
│   ├── terraform validate (syntax)
│   ├── tfsec (security scanning)
│   └── terraform plan (preview changes)
│
├── Peer Review
│   ├── Review code quality
│   ├── Verify plan output matches intent
│   ├── Check for security issues
│   └── Approve or request changes
│
└── Deployment
    ├── Merge to main branch
    ├── CI/CD pipeline triggered
    └── terraform apply runs automatically
```

**Benefits**:
- Prevents bad changes (caught in review)
- Knowledge sharing (team learns from reviews)
- Enforces standards (automated checks)
- Approval workflow (controlled deployments)

**3. CI/CD Pipeline**:
```yaml
# Azure DevOps Pipeline
trigger:
  branches:
    include:
      - main

stages:
  - stage: Plan
    jobs:
      - job: TerraformPlan
        steps:
          - task: TerraformInstaller@0
          - task: TerraformTaskV2@2
            inputs:
              command: 'init'
          - task: TerraformTaskV2@2
            inputs:
              command: 'plan'
              commandOptions: '-out=tfplan'
          - publish: tfplan  # Save plan for apply stage

  - stage: Apply
    dependsOn: Plan
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: TerraformApply
        environment: 'Production'  # Requires approval
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: tfplan
                - task: TerraformTaskV2@2
                  inputs:
                    command: 'apply'
                    commandOptions: 'tfplan'
```

**Benefits**:
- Consistent deployment process
- Automated testing before deployment
- Approval gates for production
- Deployment logs and audit trail
- Rollback capability

**4. Collaboration Without Conflicts**:
```hcl
# Terraform backend with state locking
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    # State locking prevents concurrent modifications
  }
}

# Team member 1 runs: terraform apply
# - Acquires state lock
# - Makes changes
# - Releases lock

# Team member 2 tries concurrent apply:
# - Terraform waits for lock
# - Prevents conflicting changes
# - Applies after lock released
```

**Complete Workflow Example**:
```
1. Developer clones repository
   git clone https://github.com/company/infrastructure.git

2. Creates feature branch
   git checkout -b feature/add-monitoring

3. Makes infrastructure changes
   vim terraform/monitoring.tf

4. Tests locally
   terraform plan -out=tfplan

5. Commits and pushes
   git commit -m "Add Azure Monitor and Log Analytics"
   git push origin feature/add-monitoring

6. Creates pull request
   - Automated checks run (validation, security, plan)
   - terraform plan output shown in PR
   - Team reviews changes

7. PR approved and merged
   - Merge to main branch
   - CI/CD pipeline triggered

8. Automated deployment
   - Staging: terraform apply (auto)
   - Production: terraform apply (manual approval gate)

9. Audit trail captured
   - Git: who changed what
   - Azure DevOps: deployment logs
   - Terraform state: infrastructure changes
   - Azure Activity Log: Azure resource changes
```

**Why other answers are wrong**:

**B (Shared network drive, email reviews)**:
- No version control (can't revert changes)
- No change history (who made what changes?)
- Email reviews are unstructured and lost
- Manual file locking prone to conflicts
- No automated testing or validation
- No audit trail for compliance

**C (Local machines, screen sharing)**:
- No centralization (code scattered across machines)
- Screen sharing doesn't scale (time-consuming)
- No change history or audit trail
- Single point of failure (code on one machine)
- No collaboration tools (branching, merging)
- Manual script execution (error-prone, no consistency)

**D (Cloud shell, wiki documentation)**:
- Cloud shell is ephemeral (code can be lost)
- No version control (can't track changes or revert)
- Wiki documentation separate from code (becomes outdated)
- No code review process
- No automated deployment pipeline
- No state management or locking

## Module Completion

### What You've Learned

This module covered Infrastructure as Code and Configuration Management fundamentals:

✅ **Infrastructure as Code**: Define infrastructure in version-controlled code rather than manual processes

✅ **Environment Deployment**: Automated deployment strategies (pets vs. cattle, immutable infrastructure)

✅ **Configuration Management**: Manage software and configuration on existing infrastructure (Ansible, DSC, Chef, Puppet)

✅ **Declarative vs. Imperative**: Declarative defines desired state (Terraform, ARM), imperative defines steps (PowerShell, Azure CLI)

✅ **Idempotency**: Operations produce same result when run multiple times, enabling safe retries and self-healing infrastructure

### Key Principles

1. **Treat Infrastructure as Code**: Version control, code review, automated testing, CI/CD
2. **Declarative Configuration**: Define what you want, not how to do it
3. **Idempotent Operations**: Safe to run repeatedly, recovers from failures
4. **Configuration Management**: Ongoing enforcement of desired state, drift remediation
5. **Immutable Infrastructure**: Replace rather than update for predictable deployments

### Exam Relevance (AZ-400)

These concepts are foundational for AZ-400 DevOps Engineer certification:

- **Implement IaC**: ARM templates, Terraform, Bicep
- **Manage configuration**: Azure Automation DSC, PowerShell DSC
- **Implement CI/CD**: Automated infrastructure deployment pipelines
- **Ensure quality**: Idempotent deployments, automated testing
- **Enable collaboration**: Version control, code reviews, approval workflows

### Next Steps

Continue to the next module to learn about specific IaC tools and implementation:

- **Module 2**: Create Azure Resources by using Azure Resource Manager templates
- **Module 3**: Create Azure Resources Using Azure CLI
- **Module 4**: Explore Azure Automation with DevOps
- **Module 5**: Implement Desired State Configuration (DSC)
- **Module 6**: Implement Bicep

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-infrastructure-code-configuration-management/6-knowledge-check)
