# Introduction to Azure Artifacts

## Azure DevOps
Azure DevOps provides various features for application lifecycle management:
- **Work item tracking**: Plan and track work with Agile tools
- **Source code repositories**: Git and TFVC version control
- **Build and release pipelines**: CI/CD automation with Azure Pipelines
- **Artifact management**: Package management with Azure Artifacts
- **Test plans**: Manual and exploratory testing tools

## What is Azure Artifacts?
Fully integrated package management solution within Azure DevOps that enables teams to:
- **Create and share packages**: Publish packages to feeds for consumption across projects
- **Manage dependencies**: Control package versions and dependencies centrally
- **Secure packages**: Use permissions and scopes to control access
- **Integrate with CI/CD**: Seamlessly use packages in build and release pipelines
- **Support multiple formats**: Work with various package types in unified platform

## Types of Packages Supported

Azure Artifacts currently supports five different package types:
- **NuGet packages**: For .NET libraries and tools
- **npm packages**: For JavaScript and Node.js libraries
- **Maven packages**: For Java libraries and applications
- **Python packages**: For Python modules and packages
- **Universal Packages**: Azure Artifacts-specific format for any file type

### Universal Packages
Azure Artifacts-specific package type for versioned packages containing multiple files and folders.

**Use Cases:**
- **Build artifacts**: Store compiled applications or libraries
- **Configuration files**: Version control for configuration sets
- **Documentation**: Package documentation for distribution
- **Any file type**: Use for any content that doesn't fit standard package formats

**Benefits:**
- Simple format, easy to create and consume
- Versioning like other package types
- Integration with Azure Pipelines and other tools

### Mixed Package Types
A single Azure Artifacts feed can contain any combination of package types.

**Examples:**
- Feed containing both NuGet and npm packages for full-stack project
- Feed with Maven, npm, and Universal Packages for Java web application

**Tooling Support:**
- For Maven packages, Gradle build tool also works
- Each package type has its own endpoint within the feed

## Selecting Package Sources

### Public Sources
Publicly available packages found in public package sources:
- **NuGet.org**: For .NET packages
- **npmjs.com**: For JavaScript/Node.js packages
- **Maven Central**: For Java packages
- **PyPI.org**: For Python packages

### Private Sources
Whenever solution has private packages that can't be available on public sources, use a private feed.

**Reasons for Private Feeds:**
- Proprietary code: Internal libraries and business logic
- Security: Sensitive code that shouldn't be public
- Pre-release packages: Beta versions not ready for public consumption

## Upstream Sources in Azure Artifacts
Azure Artifacts allows feeds to specify one or more upstream sources (public or other private feeds).

### Configuring Upstream Sources

**Benefits:**
- **Unified configuration**: Developers configure only your private feed
- **Automatic fallback**: Packages not in feed are fetched from upstream sources
- **Caching**: Downloaded upstream packages are cached in your feed
- **Governance**: Control which external packages teams can use

**Supported Upstream Sources:**
- Public sources: NuGet.org, npmjs.com, Maven Central, PyPI.org
- Other Azure Artifacts feeds: Within your organization or other organizations

### Example Scenario
1. Configure Azure Artifacts feed with NuGet.org as upstream source
2. Developers reference only your feed
3. When package requested, Azure Artifacts checks your feed first
4. If not found, fetches from NuGet.org and caches it
5. Subsequent requests use the cached version

## Feed Views
Azure Artifacts supports feed views to manage package promotion and lifecycle.

**What are Feed Views:**
- **Filtered subsets**: Views show specific subsets of packages in feed
- **Promotion**: Move packages between views as they mature (e.g., Development → Release)
- **Access control**: Different views can have different permissions

**Common Views:**
- **@Local**: Packages published directly to your feed
- **@Prerelease**: Packages tagged as prerelease versions
- **@Release**: Stable packages promoted for production use

**Use Cases:**
- Quality gates: Promote packages through views as they pass testing
- Environment separation: Different environments consume different views
- Version control: Control which versions are available in each view

## Integration with Azure Pipelines
Azure Artifacts integrates seamlessly with Azure Pipelines:
- **Restore packages**: Automatically restore packages during builds
- **Publish packages**: Publish build outputs as packages to feeds
- **Version management**: Automatically version packages based on build numbers
- **Authentication**: Built-in authentication for pipeline tasks

## Quick Reference

| Feature | Purpose | Benefit |
|---------|---------|---------|
| **Multi-format support** | NuGet, npm, Maven, Python, Universal | Single platform for all package types |
| **Upstream sources** | Reference public/private feeds | Unified configuration, caching |
| **Feed views** | Package promotion workflow | Quality gates, environment separation |
| **Azure Pipelines integration** | CI/CD package management | Automated publishing and consumption |
| **Access control** | Permissions and scopes | Secure package management |

## Critical Notes
- 🎯 Azure Artifacts is fully integrated with Azure DevOps
- 💡 Supports 5 package types in single unified platform
- ⚠️ Upstream sources provide caching and governance benefits
- 📊 Feed views enable package promotion and lifecycle management
- 🔒 Integrated authentication with Azure DevOps

[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-package-management/8-introduction-to-azure-artifacts)
