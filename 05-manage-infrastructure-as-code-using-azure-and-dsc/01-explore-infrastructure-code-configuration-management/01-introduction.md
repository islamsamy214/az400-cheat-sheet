# Introduction to Infrastructure as Code and Configuration Management

## Overview

Infrastructure as Code (IaC) represents a foundational DevOps methodology that enables teams to manage infrastructure through code rather than manual processes. This transformative approach treats infrastructure resources—networks, servers, storage, and services—as software artifacts that can be version controlled, tested, and deployed through automated pipelines. Many DevOps experts consider IaC a prerequisite for successful DevOps implementation, as it provides the automation, consistency, and reproducibility essential for modern software delivery.

The shift from manual infrastructure management to IaC mirrors the evolution of software development practices. Just as developers moved from manual compilation and deployment to automated CI/CD pipelines, infrastructure teams have transitioned from clicking through management portals and executing one-off scripts to defining infrastructure in version-controlled files. This transformation enables teams to apply software engineering best practices—code review, automated testing, continuous integration—to infrastructure management.

Modern cloud platforms like Azure, AWS, and Google Cloud provide elastic, on-demand resources that can scale dynamically based on application needs. However, without IaC practices, organizations struggle to leverage this flexibility effectively. Manual configuration leads to inconsistencies, "snowflake" environments where every server has unique configurations, and deployment processes that depend on tribal knowledge. IaC solves these challenges by treating infrastructure definitions as first-class code artifacts.

## Key Concepts

### Infrastructure as Code Fundamentals

**Definition and Scope**
- Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable definition files rather than physical hardware configuration or interactive configuration tools
- IaC applies software development practices to infrastructure management, including version control, testing, and continuous deployment
- Infrastructure definitions specify all resources needed for an application: networks, virtual machines, load balancers, databases, monitoring tools, and configuration settings

**Core Principles**
- **Version Control**: Infrastructure definitions stored in Git or other version control systems enable change tracking, code review, and rollback capabilities
- **Automated Testing**: Infrastructure changes validated before deployment through automated tests that verify resource configurations, security policies, and compliance requirements
- **Continuous Monitoring**: Infrastructure state continuously tracked and managed to detect drift (unauthorized changes) and ensure environments remain in desired states
- **Reproducibility**: Same configuration produces identical results every time, eliminating "works on my machine" problems and enabling reliable disaster recovery

### The Evolution of Infrastructure Management

**Traditional Manual Approach**
- Infrastructure engineers manually provision servers, configure networks, install software, and manage updates
- Configuration documented in spreadsheets, wikis, or runbooks that quickly become outdated
- Each environment (development, test, production) configured separately, leading to inconsistencies
- Deployment processes depend on specific individuals' knowledge ("key person risk")
- Changes made through interactive tools (portals, GUIs) leave no audit trail

**Infrastructure as Code Approach**
- Infrastructure defined in code files (JSON, YAML, domain-specific languages)
- All changes tracked in version control with full audit history
- Automated pipelines provision and configure resources consistently
- Self-documenting: code serves as always-current documentation
- Team collaboration through code review processes

### IaC in Modern Cloud Environments

**Cloud Platform Integration**
- Azure Resource Manager (ARM) templates and Bicep for Azure infrastructure
- AWS CloudFormation for Amazon Web Services
- Google Cloud Deployment Manager for Google Cloud Platform
- Multi-cloud tools like Terraform work across multiple cloud providers

**Infrastructure Patterns**
- **Immutable Infrastructure**: Deploy new resources rather than updating existing ones; if updates needed, create new resources and decommission old ones
- **Blue-Green Deployments**: Maintain two identical production environments; deploy changes to inactive environment, test thoroughly, then switch traffic
- **Canary Releases**: Gradually route traffic to new infrastructure versions while monitoring metrics; rollback if issues detected

## Learning Objectives

By completing this module, you'll be able to:

1. **Implement Environment Deployment Automation**: Use Infrastructure as Code to automate provisioning of networks, compute resources, storage, and services across cloud and hybrid environments

2. **Design Configuration Management Strategies**: Create version-controlled configuration management approaches that ensure consistency, enable drift detection, and support compliance requirements

3. **Choose Configuration Methodologies**: Differentiate between imperative (procedural, step-by-step) and declarative (desired state) configuration approaches and select the appropriate methodology for different scenarios

4. **Apply Idempotent Configuration Principles**: Design infrastructure and configuration scripts that produce consistent outcomes regardless of how many times they execute

## Prerequisites

### Required Knowledge
- **DevOps Principles**: Understanding of DevOps culture, practices, and objectives including continuous integration, continuous delivery, and collaboration between development and operations teams
- **Version Control Systems**: Familiarity with Git workflows, branching strategies, pull requests, and code review processes (helpful but not required for beginners)
- **Cloud Computing Concepts**: Basic knowledge of cloud service models (IaaS, PaaS, SaaS), virtual machines, networks, storage, and cloud management portals

### Helpful Background
- Experience with Azure portal or Azure CLI for resource management
- Understanding of networking fundamentals (subnets, firewalls, load balancers)
- Basic scripting knowledge (PowerShell, Bash, or Python)
- Familiarity with JSON or YAML file formats

## IaC Tools and Technologies Overview

### Declarative Tools

**Azure Resource Manager (ARM) Templates**
- Native Azure IaC solution using JSON-based template files
- Supports all Azure resource types and properties
- Integrated with Azure portal, Azure CLI, and Azure DevOps
- Template validation before deployment to catch errors early
- Supports parameters, variables, and outputs for reusability

**Bicep**
- Domain-specific language (DSL) for Azure resource deployment
- Simpler, more readable syntax compared to ARM templates JSON
- Compiles to ARM templates, ensuring compatibility
- Built-in support in Azure CLI and Visual Studio Code
- Automatic dependency detection and management

**Terraform (HashiCorp)**
- Multi-cloud IaC tool supporting Azure, AWS, Google Cloud, and many other providers
- Uses HashiCorp Configuration Language (HCL) for resource definitions
- State management tracks actual resource configurations
- Plan/apply workflow shows changes before execution
- Large ecosystem of provider plugins and modules

**Kubernetes Manifests**
- YAML-based definitions for containerized workloads
- Declarative specification of pods, services, deployments, and configurations
- GitOps workflows for managing Kubernetes resources
- Integration with Helm for package management

### Imperative Tools

**Azure CLI**
- Command-line interface for managing Azure resources
- Bash-style syntax with commands like `az vm create`, `az network vnet create`
- Scriptable for automation in CI/CD pipelines
- Interactive mode for exploratory work
- Cross-platform support (Windows, Linux, macOS)

**PowerShell**
- Powerful scripting language with Azure PowerShell modules
- Object-oriented pipeline for complex automation
- Extensive .NET library access for advanced scenarios
- Azure Automation integration for scheduled tasks

**Ansible**
- Configuration management tool using YAML playbooks
- Can be used declaratively or imperatively depending on module design
- Agentless architecture using SSH or WinRM
- Large collection of modules for various platforms and services

**Python/Bash Scripts**
- General-purpose programming languages for custom automation
- Azure SDKs available for programmatic resource management
- Flexibility for complex logic, error handling, and integrations

### Configuration Management Tools

**Azure Automation State Configuration**
- PowerShell DSC (Desired State Configuration) hosted in Azure
- Continuous drift detection and automatic remediation
- Centralized configuration management for Windows and Linux VMs
- Integration with Azure Monitor for compliance reporting

**Ansible**
- Push-based configuration management
- YAML-based playbooks define desired states
- Extensive module library for application configuration
- Tower/AWX for enterprise orchestration and RBAC

**Chef**
- Ruby-based configuration management using "recipes" and "cookbooks"
- Pull-based agent architecture
- Test Kitchen for testing infrastructure code
- Chef Infra Server for centralized management

**Puppet**
- Declarative language for system configuration
- Catalog-based approach with Puppet manifests
- Facter gathers system information for conditional logic
- Puppet Enterprise for GUI management and reporting

## Real-World Scenarios

### Scenario 1: Multi-Environment Application Deployment

**Challenge**: A development team needs to maintain consistent environments across development, staging, and production. Manual configuration led to drift, causing bugs that only appeared in production.

**IaC Solution**:
1. Created ARM template defining web app, database, storage account, and networking
2. Template parameterized for environment-specific values (size, SKU, region)
3. Azure DevOps pipeline deploys to all environments using same template
4. Changes reviewed through pull requests before merging to main branch

**Benefits**:
- Development environments match production exactly
- New team members spin up full environment in minutes
- Compliance auditors review infrastructure definitions in Git
- Disaster recovery: recreate entire environment from code repository

### Scenario 2: Scaling E-Commerce Platform

**Challenge**: E-commerce company experiences traffic spikes during sales events. Manual scaling took hours, causing lost revenue during high-traffic periods.

**IaC Solution**:
1. Terraform modules define auto-scaling policies and metrics
2. Infrastructure code includes load balancers, VM scale sets, caching layers
3. CI/CD pipeline tests scaling behavior in staging before production deployment
4. Monitoring triggers automatic scale-out/scale-in based on CPU and request metrics

**Benefits**:
- Automatic scaling handles traffic surges without manual intervention
- Cost optimization: scale down during low-traffic periods
- Predictable scaling behavior validated in staging
- Infrastructure changes deployed through same pipeline as application code

### Scenario 3: Compliance and Security Hardening

**Challenge**: Financial services organization must meet strict regulatory requirements. Manual security configurations varied across servers, creating audit findings.

**IaC Solution**:
1. Azure Policy definitions codified as ARM templates
2. Desired State Configuration (DSC) ensures consistent security settings
3. Automated compliance scanning integrated into deployment pipeline
4. Git commits provide audit trail for all security configuration changes

**Benefits**:
- Consistent security posture across all resources
- Automated compliance reporting for auditors
- Configuration drift detected and remediated automatically
- Security updates deployed simultaneously to all resources

## Best Practices

### Version Control and Collaboration

**Repository Structure**
- Organize IaC code in dedicated repositories separate from application code (or monorepo with clear directory structure)
- Use meaningful directory names: `environments/`, `modules/`, `policies/`, `scripts/`
- Include README files explaining purpose, usage, and prerequisites
- Maintain separate branches for development, testing, and production changes

**Code Review Practices**
- Require pull requests for all infrastructure changes
- Automated tests run on pull requests (linting, security scanning, plan/dry-run)
- At least one reviewer approves changes before merging
- Document rationale for changes in commit messages and PR descriptions

**Git Workflow**
- Use feature branches for infrastructure changes
- Tag releases for production deployments
- Protect main branch: no direct commits, require PR approval
- Maintain CHANGELOG documenting infrastructure changes

### Testing and Validation

**Pre-Deployment Testing**
- **Linting**: Validate syntax and style (ARM template test toolkit, terraform validate)
- **Security Scanning**: Check for exposed secrets, insecure configurations (Azure Security Center, tfsec)
- **Policy Compliance**: Verify resources comply with organizational policies
- **Dry Run**: Generate plan showing changes before applying (Terraform plan, ARM template what-if)

**Post-Deployment Validation**
- **Smoke Tests**: Verify basic functionality after deployment
- **Integration Tests**: Confirm resources communicate correctly
- **Compliance Checks**: Validate security configurations applied successfully
- **Performance Tests**: Ensure infrastructure meets performance requirements

### Security Considerations

**Secrets Management**
- Never commit secrets (passwords, API keys, certificates) to version control
- Use Azure Key Vault, AWS Secrets Manager, or HashiCorp Vault for secret storage
- Reference secrets in IaC code without embedding values
- Rotate secrets regularly and audit secret access

**Access Control**
- Apply principle of least privilege to service principals running IaC deployments
- Use managed identities where possible instead of service principals
- Implement role-based access control (RBAC) for IaC pipelines
- Audit who can approve and deploy infrastructure changes

**Network Security**
- Default deny network policies with explicit allow rules
- Segment networks using subnets and network security groups
- Use private endpoints for Azure services to avoid public internet exposure
- Implement network monitoring and logging

### Documentation and Maintenance

**Code Documentation**
- Comment complex logic in IaC files
- Include descriptions for parameters and variables
- Document dependencies between modules
- Maintain architecture diagrams showing infrastructure topology

**Operations Documentation**
- Deployment runbooks for manual intervention scenarios
- Troubleshooting guides for common issues
- Rollback procedures if deployment fails
- Contact information for infrastructure teams

**Regular Maintenance**
- Update IaC tool versions regularly
- Review and update modules for deprecated resource types
- Refactor code to eliminate duplication
- Archive unused infrastructure definitions

## Common Pitfalls and Solutions

### Pitfall 1: State Management Issues

**Problem**: Terraform state files get out of sync with actual resources, causing errors or duplicates.

**Solution**:
- Use remote state storage (Azure Storage, AWS S3, Terraform Cloud)
- Enable state locking to prevent concurrent modifications
- Never edit state files manually
- Use `terraform refresh` to sync state with actual resources
- Implement state backup and recovery procedures

### Pitfall 2: Credential Exposure

**Problem**: Accidentally committing secrets to version control, exposing credentials.

**Solution**:
- Configure `.gitignore` to exclude files containing secrets
- Use pre-commit hooks to scan for secrets before commit
- Implement secret scanning in CI/CD pipelines (GitHub secret scanning, GitGuardian)
- If secrets committed, rotate immediately and use tools like BFG Repo-Cleaner to remove from history
- Educate team on secure credential handling

### Pitfall 3: Over-Complicated Modules

**Problem**: Creating overly complex, tightly-coupled modules that are difficult to understand and maintain.

**Solution**:
- Follow single responsibility principle: each module does one thing well
- Limit module nesting depth
- Provide clear input variables and output values
- Include examples showing how to use modules
- Balance DRY (Don't Repeat Yourself) with readability

### Pitfall 4: Lack of Testing

**Problem**: Deploying untested infrastructure changes directly to production, causing outages.

**Solution**:
- Implement test environments that mirror production
- Use staging environment for final validation
- Automate testing in CI/CD pipelines
- Test rollback procedures before production deployment
- Maintain test coverage metrics for infrastructure code

### Pitfall 5: Configuration Drift

**Problem**: Manual changes made directly to resources, causing infrastructure to diverge from code definitions.

**Solution**:
- Implement Azure Policy or AWS Config to prevent manual changes
- Use drift detection tools (Terraform plan, Azure Resource Manager what-if)
- Configure alerts when drift detected
- Educate teams that all changes must go through IaC pipelines
- Regular drift remediation: reapply IaC to correct unauthorized changes

## Tools and Commands Quick Reference

### Azure Resource Manager

```bash
# Create resource group and deploy ARM template
az group create --name myResourceGroup --location eastus
az deployment group create \
  --resource-group myResourceGroup \
  --template-file template.json \
  --parameters parameters.json

# Validate template before deployment
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file template.json

# Show what would change (what-if operation)
az deployment group what-if \
  --resource-group myResourceGroup \
  --template-file template.json
```

### Bicep

```bash
# Build Bicep file to ARM template JSON
az bicep build --file main.bicep

# Deploy Bicep template
az deployment group create \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters environment=production

# Decompile ARM template to Bicep
az bicep decompile --file template.json
```

### Terraform

```bash
# Initialize Terraform working directory
terraform init

# Validate configuration syntax
terraform validate

# Create execution plan showing changes
terraform plan -out=tfplan

# Apply changes to infrastructure
terraform apply tfplan

# Destroy managed infrastructure
terraform destroy

# Show current state
terraform show

# Import existing resource into Terraform state
terraform import azurerm_resource_group.example /subscriptions/{subscription-id}/resourceGroups/myResourceGroup
```

### Azure CLI for IaC

```bash
# Create virtual network
az network vnet create \
  --resource-group myResourceGroup \
  --name myVnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.1.0/24

# Create virtual machine
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys

# Export resource group as ARM template
az group export \
  --name myResourceGroup \
  --include-comments \
  --include-parameter-default-value > template.json
```

## Assessment Questions

### Question 1: Infrastructure as Code Benefits
**Scenario**: Your organization currently manages infrastructure manually through the Azure portal. Management asks you to justify investing in Infrastructure as Code practices.

**Question**: Which benefits would you highlight to demonstrate IaC value? (Select all that apply)

A) Complete audit trail showing who changed what and when  
B) Ability to recreate entire environments quickly for disaster recovery  
C) Elimination of configuration drift through automated compliance checks  
D) Reduced infrastructure costs through resource deletion  
E) Consistent environments eliminating "works on my machine" issues

**Answer**: A, B, C, E

**Explanation**:
- **A (Correct)**: Version control provides complete audit trail of infrastructure changes
- **B (Correct)**: IaC enables rapid environment recreation from code definitions
- **C (Correct)**: Automated tools detect and remediate configuration drift
- **D (Incorrect)**: IaC improves management efficiency but doesn't inherently reduce costs; cost optimization requires deliberate resource sizing decisions
- **E (Correct)**: Consistent infrastructure definitions ensure environment parity

### Question 2: IaC Tools Selection
**Scenario**: You're building a new application that will deploy to both Azure and AWS. The team has experience with JSON and YAML but limited programming knowledge.

**Question**: Which IaC tool would be most appropriate for this scenario?

A) Azure Resource Manager templates  
B) Terraform  
C) PowerShell scripts  
D) Ansible playbooks

**Answer**: B) Terraform

**Explanation**:
- **B (Correct)**: Terraform supports multi-cloud deployments with a single tool and uses HCL (similar to JSON/YAML). It provides declarative syntax familiar to teams without deep programming skills.
- **A (Incorrect)**: ARM templates only work with Azure, not AWS
- **C (Incorrect)**: PowerShell scripts require programming knowledge and aren't truly multi-cloud without significant abstraction
- **D (Incorrect)**: While Ansible works cross-cloud, it's primarily a configuration management tool rather than infrastructure provisioning tool

### Question 3: Version Control for Infrastructure
**Scenario**: Your team stores IaC templates in a shared network drive. Developers sometimes overwrite each other's changes, and there's no history of what changed or why.

**Question**: What changes should you implement to address these issues?

A) Create separate folders for each developer  
B) Store templates in Git with branch protection and pull request workflow  
C) Lock files when developers edit them  
D) Maintain a change log document listing all modifications

**Answer**: B

**Explanation**:
- **B (Correct)**: Git provides version control, change tracking, code review through pull requests, and prevents conflicts through merge workflows. Branch protection ensures changes reviewed before production deployment.
- **A (Incorrect)**: Separate folders cause fragmentation and don't solve version control issues
- **C (Incorrect)**: File locking prevents collaboration and doesn't provide change history
- **D (Incorrect)**: Manual change logs become outdated and don't prevent conflicts

### Question 4: Testing Infrastructure Code
**Scenario**: Your team deploys infrastructure changes directly to production. Recently, a malformed template caused a production outage requiring emergency rollback.

**Question**: Which testing practices would prevent this issue? (Select all that apply)

A) Template validation to check syntax before deployment  
B) Deploy to staging environment first for validation  
C) Use what-if or plan operations to preview changes  
D) Implement automated security scanning  
E) Require management approval for all changes

**Answer**: A, B, C, D

**Explanation**:
- **A (Correct)**: Template validation catches syntax errors before deployment
- **B (Correct)**: Staging environments allow testing changes before production impact
- **C (Correct)**: What-if/plan operations show exactly what will change
- **D (Correct)**: Security scanning detects misconfigurations that could cause issues
- **E (Incorrect)**: Management approval is a governance control but doesn't prevent technical issues; automated testing is more effective

### Question 5: Secrets Management
**Scenario**: Your infrastructure templates contain database passwords and API keys hardcoded in parameter files. These files are committed to your Git repository.

**Question**: What is the security risk and how should you address it?

A) Risk: Exposed credentials; Solution: Encrypt parameter files  
B) Risk: Exposed credentials; Solution: Use Azure Key Vault and reference secrets  
C) Risk: Version conflicts; Solution: Use separate parameter files per environment  
D) Risk: Template complexity; Solution: Simplify templates by removing parameters

**Answer**: B

**Explanation**:
- **B (Correct)**: Hardcoded secrets in Git are exposed to anyone with repository access, including potentially malicious actors if repository public or compromised. Azure Key Vault stores secrets securely and templates reference them without embedding values.
- **A (Incorrect)**: Encrypting files in Git still exposes secrets to anyone who can decrypt them; proper solution is to never store secrets in version control
- **C (Incorrect)**: The issue is security, not version conflicts
- **D (Incorrect)**: Removing parameters reduces flexibility; the problem is secret storage method, not template structure

## Summary

Infrastructure as Code (IaC) transforms infrastructure management from manual, error-prone processes to automated, consistent, and reproducible workflows. By treating infrastructure definitions as code, teams gain version control, automated testing, and continuous deployment capabilities previously available only for application code.

**Key Takeaways**:

1. **IaC Enables DevOps Success**: Infrastructure automation is a prerequisite for successful DevOps implementation, enabling rapid iteration and continuous delivery

2. **Version Control is Essential**: Storing infrastructure definitions in Git provides audit trails, change tracking, collaboration through pull requests, and rollback capabilities

3. **Multiple Tools Available**: Choose tools based on team skills, cloud platform requirements, and complexity needs (declarative like Terraform/ARM vs. imperative like Azure CLI/PowerShell)

4. **Testing Prevents Outages**: Validate templates, test in staging, use dry-run operations, and implement security scanning before production deployment

5. **Security First**: Never commit secrets to version control; use dedicated secret management services like Azure Key Vault

6. **Consistency Across Environments**: IaC ensures development, testing, and production environments configured identically, eliminating "works on my machine" problems

7. **Collaboration Through Code Review**: Pull request workflows enable team review of infrastructure changes before deployment, catching errors and knowledge sharing

8. **Documentation Through Code**: Well-written IaC serves as always-current documentation, supplemented by README files and architecture diagrams

**Next Steps**:

This introductory unit established IaC foundations. Subsequent units dive deeper into specific topics:
- **Environment Deployment**: Strategies for provisioning infrastructure resources
- **Configuration Management**: Approaches for configuring resources after provisioning
- **Imperative vs. Declarative**: Detailed comparison of configuration methodologies
- **Idempotent Operations**: Ensuring consistent outcomes through idempotent design

By mastering IaC principles and practices, you'll enable your organization to deploy infrastructure reliably, quickly, and securely—essential capabilities for modern cloud-native applications.

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-infrastructure-code-configuration-management/1-introduction)
