# Publish Packages

## Create a Feed
First step is to create a feed where packages will be stored. Azure Artifacts allows multiple feeds as needed.

### Feed Creation Options
When creating a feed, specify:
- **Name**: Descriptive name (e.g., `MyCompany-Libraries`)
- **Visibility**: Organization-scoped or project-scoped
- **Upstream sources**: Include packages from common public sources (NuGet.org, npmjs.com)
- **Permissions**: Set who can contribute, read, or administer the feed

### Creating a Feed in Azure Artifacts
**Steps:**
1. Navigate to Azure DevOps → Artifacts
2. Click Create Feed
3. Specify feed name and visibility
4. Optionally enable upstream sources for public package sources
5. Click Create

## What are Feeds in Azure Artifacts?
Feeds can be scoped to project or made available across entire organization.

### Feed Characteristics

**Scope Options:**
- **Project-scoped feeds**: Available only within specific project
- **Organization-scoped feeds**: Shared across all projects in organization

**Package Type Support:**
Feeds can store different types:
- npm: JavaScript and Node.js packages
- NuGet: .NET packages
- Maven: Java packages
- Python: Python packages
- Universal Packages: Azure Artifacts-specific format

### Feed Features

**Upstream Sources:**
Each feed can include one or more upstream sources to automatically include packages from:
- Public sources (NuGet.org, npmjs.com, Maven Central, PyPI)
- Other private Azure Artifacts feeds

**Security Settings:**
Each feed has configurable security settings:
- **Readers**: Can view and restore packages
- **Contributors**: Can publish and restore packages
- **Collaborators**: Can save packages from upstream sources
- **Owners**: Full administrative control

**Access Control:**
Developers can control how packages are shared:
- Restricting access to specific users, teams, or organizations
- Making packages publicly available
- Controlling package promotion through feed views

## Publish Packages to a Feed

### General Publishing Workflow
1. **Create the package**: Build and package component using appropriate tools
2. **Authenticate**: Configure authentication to Azure Artifacts (PAT, Credential Provider)
3. **Configure feed**: Add Azure Artifacts feed to package manager configuration
4. **Publish**: Use package manager commands to push package to feed

### Finding the Right Workflow
To find the right workflow for your technology:
1. Sign in to Azure DevOps organization
2. Navigate to Artifacts
3. Select Connect to feed
4. From left panel, choose package type you're working with
5. Follow the Project setup instructions

### Publishing Examples

**NuGet:**
```bash
dotnet nuget push MyPackage.1.0.0.nupkg --source "MyFeed" --api-key az
```

**npm:**
```bash
npm publish --registry https://pkgs.dev.azure.com/myorg/_packaging/myfeed/npm/registry/
```

**Maven:**
```bash
mvn deploy
```

**Python:**
```bash
twine upload --repository-url https://pkgs.dev.azure.com/myorg/_packaging/myfeed/pypi/upload mypackage-1.0.0.tar.gz
```

## Updating Packages
Technically, updating a package is made by pushing a new version of the package to the feed.

### Versioning Requirements

**Common Versioning Approaches:**
- **Semantic Versioning (SemVer)**: `MAJOR.MINOR.PATCH` format (e.g., 1.2.3)
- **Date-based versioning**: Using dates in version numbers (e.g., 2025.10.09)
- **Build-based versioning**: Incorporating build numbers (e.g., 1.0.0-build.456)

### Immutability
⚠️ **Important**: Once a package version is published, it's typically immutable (cannot be changed).

- To update package, publish new version with incremented version number
- Don't overwrite existing versions, as this breaks reproducibility
- Use prerelease suffixes for unstable versions (e.g., 1.0.0-alpha, 1.0.0-beta)

### Best Practices for Package Updates

**Versioning:**
- Follow semantic versioning principles
- Increment MAJOR for breaking changes
- Increment MINOR for new features (backward compatible)
- Increment PATCH for bug fixes

**Release Notes:**
- Document changes in each version
- Include what's new, what's changed, and what's fixed
- Note any breaking changes clearly

**Deprecation:**
- Mark old versions as deprecated when appropriate
- Provide migration guides for breaking changes
- Communicate deprecation timeline to consumers

**Testing:**
- Test packages before publishing
- Use prerelease versions for beta testing
- Promote to stable versions only after validation

## Automation with CI/CD
Publishing packages can be automated using Azure Pipelines.

**Benefits of Automation:**
- ✅ **Consistency**: Same process for every package
- ✅ **Version management**: Automatic version incrementing
- ✅ **Quality gates**: Publish only after passing tests
- ✅ **Speed**: Fast and reliable publishing

**Example Pipeline Task:**
```yaml
- task: NuGetCommand@2
  inputs:
    command: "push"
    packagesToPush: "$(Build.ArtifactStagingDirectory)/**/*.nupkg"
    nuGetFeedType: "internal"
    publishVstsFeed: "MyFeed"
```

## Quick Reference

| Step | Action | Tool |
|------|--------|------|
| **Create Feed** | Set up package repository | Azure Artifacts |
| **Build Package** | Compile and package code | dotnet pack, npm pack, mvn package |
| **Authenticate** | Configure credentials | PAT, Credential Provider |
| **Publish** | Push package to feed | Package manager CLI |
| **Update** | Publish new version | Increment version number |

## Security Settings

| Role | Permissions |
|------|-------------|
| **Readers** | View and restore packages |
| **Contributors** | Publish and restore packages |
| **Collaborators** | Save packages from upstream |
| **Owners** | Full administrative control |

## Critical Notes
- 🎯 Feeds can be project-scoped or organization-scoped
- 💡 Package versions are immutable once published
- ⚠️ Use semantic versioning for clear version communication
- 📊 Automate publishing with Azure Pipelines for consistency
- 🔒 Configure appropriate permissions for feed security

[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-package-management/9-publish-packages)
