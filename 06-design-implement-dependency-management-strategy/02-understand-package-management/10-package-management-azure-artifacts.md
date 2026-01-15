# Package Management with Azure Artifacts

## Scenario
Azure Artifacts facilitate discovery, installation, and publishing of NuGet, npm, and Maven packages in Azure DevOps. Deeply integrated with other Azure DevOps features such as Build, making package management a seamless part of existing workflows.

## Lab Objectives
After completing this lab, you'll be able to:
- ✅ Create and connect to a feed
- ✅ Create and publish a NuGet package
- ✅ Import a NuGet package
- ✅ Update a NuGet package

## Requirements

### Prerequisites
- **Browser**: Microsoft Edge or Azure DevOps-supported browser
- **Azure DevOps organization**: If you don't have one, create by following [Create an organization or project collection](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization)
- **Visual Studio 2022 Community Edition**: Available from [Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/)

### Visual Studio 2022 Workloads
Installation should include:
- ASP.NET and web development
- Azure development
- .NET Core cross-platform development

## Lab Exercises

### Exercise 0: Configure Lab Prerequisites
Set up the required environment and tools.

### Exercise 1: Working with Azure Artifacts
Hands-on practice with:
- Creating feeds
- Publishing packages
- Consuming packages
- Updating packages

## Key Concepts Covered

### Feed Creation
- Create feeds in Azure Artifacts
- Configure feed visibility and permissions
- Set up upstream sources

### Package Publishing
- Build NuGet packages from code
- Publish packages to Azure Artifacts feeds
- Version packages appropriately

### Package Consumption
- Connect to Azure Artifacts feeds
- Install packages from feeds
- Configure package sources in projects

### Package Management
- Import existing packages
- Update package versions
- Manage package dependencies

## Hands-On Skills
This lab provides practical experience with:
- **Azure Artifacts**: Creating and managing feeds
- **NuGet packages**: Building, publishing, and consuming
- **Visual Studio integration**: Using Azure Artifacts within Visual Studio
- **Version management**: Handling package updates and versions

## Integration Points
- **Azure Pipelines**: Automate package publishing
- **Visual Studio**: Integrated package management
- **Azure DevOps**: Seamless integration with other features
- **Build automation**: Package creation during CI builds

## Best Practices Demonstrated
- Proper feed configuration
- Semantic versioning for packages
- Upstream source usage
- Package restoration in builds
- Access control for feeds

## Estimated Time
⏱️ **40 minutes** to complete all exercises

## Quick Reference

| Task | Tool/Feature | Purpose |
|------|--------------|---------|
| **Create Feed** | Azure Artifacts | Package repository |
| **Build Package** | Visual Studio / dotnet CLI | Create NuGet package |
| **Publish Package** | nuget push / dotnet push | Upload to feed |
| **Install Package** | NuGet Package Manager | Consume in projects |
| **Update Package** | Version increment | New package version |

## Learning Outcomes
After this lab, you'll understand:
- How to create and manage Azure Artifacts feeds
- The process of building and publishing packages
- How to consume packages from private feeds
- Package versioning and update workflows
- Integration between Azure Artifacts and Visual Studio

## Critical Notes
- 🎯 Hands-on lab provides practical experience with Azure Artifacts
- 💡 Covers full package lifecycle: create, publish, consume, update
- ⚠️ Requires Visual Studio 2022 with specific workloads
- 📊 Estimated 40 minutes for completion
- 🔄 Demonstrates integration with Azure DevOps features

## Lab Access
[Launch Lab](https://go.microsoft.com/fwlink/?linkid=2270305)

[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-package-management/10-package-management-azure-artifacts)
