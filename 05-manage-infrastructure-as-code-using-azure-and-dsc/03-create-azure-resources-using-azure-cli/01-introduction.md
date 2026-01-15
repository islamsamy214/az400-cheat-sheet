# Introduction

## Overview

Azure CLI (Command-Line Interface) is Microsoft's cross-platform command-line tool for managing Azure resources. It provides a consistent, powerful interface that works across Windows, macOS, and Linux, enabling developers and DevOps professionals to automate infrastructure management, integrate with CI/CD pipelines, and maintain version-controlled infrastructure as code.

In modern cloud environments, managing resources through graphical interfaces becomes impractical at scale. Azure CLI bridges the gap between manual portal operations and full infrastructure as code (IaC) solutions, offering flexibility for interactive troubleshooting, scripted automation, and integration with existing DevOps workflows.

## Why Azure CLI Matters

### Automation at Scale
Manual resource creation through the Azure portal doesn't scale when managing multiple environments or dozens of resources. Azure CLI enables:
- **Batch Operations**: Create multiple resources with loops and conditionals
- **Consistent Deployments**: Same commands work across dev, staging, and production
- **Rapid Provisioning**: Execute complex multi-resource deployments in seconds
- **Repeatable Processes**: Eliminate human error through scripted procedures

### Version Control and Collaboration
Azure CLI scripts are text files that integrate seamlessly with Git:
- **Track Changes**: See who changed what infrastructure and when
- **Code Reviews**: Apply software engineering practices to infrastructure
- **Rollback Capability**: Revert to previous infrastructure states
- **Team Collaboration**: Share and improve infrastructure scripts across teams

### CI/CD Integration
Modern DevOps pipelines require programmatic infrastructure management:
- **Pipeline Integration**: Azure CLI works in Azure Pipelines, GitHub Actions, Jenkins
- **Automated Testing**: Create test environments, run tests, destroy resources automatically
- **Infrastructure Gates**: Validate infrastructure state before deployments
- **Cross-Platform Compatibility**: Same scripts run on Linux build agents, Windows runners, or macOS developers' machines

### Flexibility and Power
Azure CLI complements other infrastructure tools:
- **Interactive Troubleshooting**: Quickly investigate issues without writing full ARM templates
- **Hybrid Approaches**: Combine CLI scripts with ARM templates, Terraform, or Bicep
- **Service Principal Integration**: Secure authentication for automated workflows
- **Query and Filter**: JMESPath queries extract specific data from complex JSON responses

## Learning Objectives

After completing this module, you'll be able to:

1. **Understand Azure CLI Architecture**
   - Explain command structure: `az <group> <subgroup> <command> --parameter value`
   - Differentiate between core references and extensions
   - Choose between interactive and scripted execution modes
   - Understand how Azure CLI translates commands to REST API calls

2. **Install and Configure Azure CLI**
   - Select the appropriate installation method for your platform (Windows, Linux, macOS)
   - Verify successful installation with `az version`
   - Authenticate using interactive login, service principals, or managed identities
   - Configure default settings with `az configure`

3. **Execute Azure CLI Commands Interactively**
   - Sign in to Azure with `az login`
   - Manage subscriptions with `az account`
   - Create and manage resource groups
   - Deploy Azure resources (storage accounts, virtual machines, networks)
   - Query results using JMESPath syntax
   - Format output as JSON, table, TSV, or YAML

4. **Create Azure CLI Scripts for Bash**
   - Structure scripts with shebang, variables, and error handling
   - Implement loops for bulk resource creation
   - Use conditionals for environment-specific configurations
   - Generate unique resource names programmatically
   - Create parameterized scripts accepting command-line arguments
   - Develop configuration-driven deployments with JSON files

5. **Apply Best Practices for Production**
   - Implement comprehensive error handling and logging
   - Create idempotent scripts that can run multiple times safely
   - Use authentication methods appropriate for automation (service principals)
   - Organize scripts into reusable function libraries
   - Test scripts safely with `--debug` and dry-run modes

6. **Troubleshoot Azure CLI Issues**
   - Use `--debug` to see detailed API calls and command parsing
   - Understand scripting language syntax differences (Bash, PowerShell, cmd.exe)
   - Find help with `az find`, `--help`, and documentation indexes
   - Interpret error messages and resolve common issues

## Prerequisites

To get the most from this module, you should have:

### Command-Line Experience
- **Basic Terminal Usage**: Navigate directories, edit files, run commands
- **Scripting Fundamentals**: Variables, loops, conditionals, functions (Bash or PowerShell)
- **Text Editors**: Comfort using vim, nano, VS Code, or other editors for script development

### Azure Fundamentals
- **Core Concepts**: Subscriptions, resource groups, regions, resources
- **Resource Types**: Basic understanding of VMs, storage accounts, networks, databases
- **Azure Portal**: Familiarity with creating resources through the GUI
- **RBAC Basics**: Understanding permissions, roles, and access control

### Azure Portal Experience
While this module focuses on CLI, understanding portal equivalents helps:
- Know how to create a resource group manually
- Understand resource properties and configuration options
- Familiarity with Azure Cloud Shell

### DevOps Awareness
- **CI/CD Concepts**: Pipelines, automated testing, deployment stages
- **Infrastructure as Code**: Benefits of declarative vs. imperative approaches
- **Version Control**: Git basics for managing infrastructure scripts

## Real-World Scenarios

### Scenario 1: Multi-Environment Resource Provisioning
**Challenge**: A software company maintains development, staging, and production environments. Manual portal creation is time-consuming and inconsistent.

**Azure CLI Solution**:
```bash
#!/bin/bash
# Parameterized script accepts environment name
environment=$1  # dev, staging, prod

case $environment in
    dev)
        sku="Standard_LRS"
        vmSize="Standard_B2s"
        ;;
    staging)
        sku="Standard_GRS"
        vmSize="Standard_D2s_v3"
        ;;
    prod)
        sku="Standard_RAGRS"
        vmSize="Standard_D4s_v3"
        ;;
esac

# Create resources with environment-specific configurations
az group create --name "rg-app-$environment" --location eastus
az storage account create --name "stapp$environment$RANDOM" \
    --resource-group "rg-app-$environment" --sku $sku
```

**Outcome**: Same script deploys all three environments with appropriate SKUs. Version-controlled script ensures consistency. Developers run `./deploy.sh dev` to create personal test environments.

### Scenario 2: Automated Testing Pipeline
**Challenge**: Integration tests require fresh Azure resources before each test run. Manual creation delays testing by hours.

**Azure CLI Solution**:
```bash
#!/bin/bash
# CI/CD pipeline script
echo "Creating test environment..."
testId=$(date +%s)
rgName="rg-test-$testId"

# Create isolated test environment
az group create --name $rgName --location westus2
az sql server create --name "sql-test-$testId" --resource-group $rgName \
    --admin-user testadmin --admin-password $(generate_password)

# Run tests
./run-integration-tests.sh

# Cleanup test resources
az group delete --name $rgName --yes --no-wait
```

**Outcome**: Each pipeline run gets isolated resources. Parallel test runs don't interfere. Automatic cleanup prevents cost accumulation. Test cycle time reduced from 3 hours to 15 minutes.

### Scenario 3: Disaster Recovery Automation
**Challenge**: During regional outage, manually recreating 50+ resources in secondary region takes 8+ hours.

**Azure CLI Solution**:
```bash
#!/bin/bash
# Disaster recovery failover script
primaryRegion="eastus"
secondaryRegion="westus2"

# Get list of resources from primary region
resourceList=$(az resource list --location $primaryRegion --output json)

# Recreate in secondary region
for resource in $(echo $resourceList | jq -r '.[] | @base64'); do
    _jq() {
        echo ${resource} | base64 --decode | jq -r ${1}
    }
    
    resourceType=$(_jq '.type')
    resourceName=$(_jq '.name')
    
    # Recreate resource in secondary region
    recreate_resource "$resourceType" "$resourceName" "$secondaryRegion"
done
```

**Outcome**: Disaster recovery time reduced from 8 hours to 45 minutes. Script tested quarterly during DR drills. RTO (Recovery Time Objective) improved by 90%.

### Scenario 4: Cost Optimization with Scheduled Resources
**Challenge**: Development environments run 24/7 but only needed during business hours. Annual waste: $50,000+.

**Azure CLI Solution**:
```bash
#!/bin/bash
# Scheduled startup (cron: 0 8 * * 1-5)
az vm start --ids $(az vm list --resource-group rg-dev --query "[].id" -o tsv)

# Scheduled shutdown (cron: 0 18 * * 1-5)
az vm deallocate --ids $(az vm list --resource-group rg-dev --query "[].id" -o tsv)
```

**Outcome**: VMs run only during business hours (8 AM - 6 PM, Monday-Friday). Annual cost reduced by $35,000 (70% savings). Script expanded to staging environments for additional savings.

## Module Structure

This module follows a progressive learning path:

### Unit 1: Introduction (Current Unit)
- Module objectives and prerequisites
- Why Azure CLI matters
- Real-world use cases
- Module roadmap

### Unit 2: What Is Azure CLI?
- Azure CLI architecture and design
- Command structure and syntax
- Platform support (Windows, Linux, macOS, Docker, Cloud Shell)
- Core references vs. extensions
- Reference status levels (GA, Preview, Deprecated)
- Interactive vs. scripted execution modes

### Unit 3: Install Azure CLI
- Environment selection considerations
- Windows installation (MSI installer)
- Linux installation (package manager, script)
- macOS installation (Homebrew)
- Docker container usage
- Azure Cloud Shell (pre-installed option)
- Verification with `az version`
- Authentication methods (interactive login, service principal, managed identity)

### Unit 4: Execute Azure CLI Commands Interactively
- Sign in with `az login`
- Manage subscriptions
- Create resource groups
- Deploy storage accounts
- Query results with JMESPath
- Format output (JSON, table, TSV, YAML)
- Verify resource creation with `list` and `show` commands

### Unit 5: Create an Azure CLI Script for Bash
- Bash script structure (shebang, variables, comments)
- Resource deployment scripts
- Error handling and validation
- Loops for bulk operations
- Conditionals for environment-specific logic
- Parameterized scripts
- Configuration-driven deployments
- Resource cleanup scripts
- Production script best practices

### Unit 6: Tips to Use the Azure CLI Successfully
- Understanding API calls with `--debug`
- Troubleshooting scripting language differences
- Getting help with `az find` and `--help`
- Documentation resources (reference index, samples)
- Using Copilot for assistance
- Performance optimization
- Integration with CI/CD pipelines

### Unit 7: Module Assessment
- Knowledge check questions
- Scenario-based assessments
- Best practice validation

### Unit 8: Summary
- Key takeaways
- Next steps
- Additional resources

## Getting Started

Azure CLI represents a critical skill for modern DevOps professionals. While ARM templates and Terraform provide declarative infrastructure as code, Azure CLI offers imperative control for:
- **Quick Prototyping**: Test resource configurations before committing to templates
- **Troubleshooting**: Investigate issues without modifying production templates
- **Hybrid Solutions**: Combine CLI scripts with template-based deployments
- **Learning**: Understand Azure services interactively before automating

This module emphasizes practical, production-ready patterns. Examples progress from simple interactive commands to complex, error-handled scripts suitable for enterprise CI/CD pipelines.

### Your Learning Path

1. **Understand the Tool** (Units 2-3): Learn architecture, install Azure CLI, authenticate
2. **Execute Commands** (Unit 4): Practice interactive commands hands-on
3. **Build Scripts** (Unit 5): Create automated, repeatable deployment scripts
4. **Master the Tool** (Unit 6): Apply best practices, troubleshooting, optimization
5. **Validate Knowledge** (Units 7-8): Assess understanding, plan next steps

## Key Takeaways

Azure CLI empowers DevOps teams to:
- **Automate Consistently**: Deploy resources programmatically across environments
- **Collaborate Effectively**: Version control infrastructure scripts with Git
- **Integrate Seamlessly**: Embed in CI/CD pipelines (Azure Pipelines, GitHub Actions, Jenkins)
- **Troubleshoot Efficiently**: Interactively investigate issues without GUI overhead
- **Scale Operations**: Manage hundreds of resources with bulk operations
- **Reduce Costs**: Automate resource lifecycle (create, test, destroy)

Whether you're provisioning development environments, automating disaster recovery, or building CI/CD pipelines, Azure CLI provides the flexibility and power needed for modern cloud operations.

## Next Steps

Ready to start? Proceed to **Unit 2: What Is Azure CLI?** to understand the architecture, command structure, and execution modes that make Azure CLI a powerful tool for Azure resource management.

For hands-on practice, have an Azure subscription ready (free trial available at azure.microsoft.com/free). If you prefer not to install locally, Azure Cloud Shell provides a browser-based environment with Azure CLI pre-installed.

---

## Additional Resources

- **Azure CLI Documentation**: https://learn.microsoft.com/en-us/cli/azure/
- **Azure CLI Reference**: https://learn.microsoft.com/en-us/cli/azure/reference-index
- **Azure CLI Samples**: https://github.com/Azure-Samples/azure-cli-samples
- **Azure Cloud Shell**: https://shell.azure.com
- **JMESPath Tutorial**: https://jmespath.org/tutorial.html
- **Azure Free Trial**: https://azure.microsoft.com/free

---

**Module**: Create Azure Resources Using Azure CLI  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/create-azure-resources-by-using-azure-cli/1-introduction
