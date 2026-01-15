# Identify Existing Artifact Repositories

## Key Concepts
- An artifact is a deployable component of your application
- Azure Pipelines can work with wide variety of artifact sources and repositories
- Understanding current artifact landscape is first step in migration

## Understanding Artifacts in Release Pipelines

### Current Artifact Storage Locations
Artifacts might be stored in various locations:
- **Source control**: Git, Team Foundation Version Control (TFVC), Subversion
- **File shares**: Network drives or shared folders
- **Package management tools**: NuGet Server, npm registries, Maven repositories
- **Container registries**: Docker Hub, Azure Container Registry
- **Cloud storage**: Azure Blob Storage, Amazon S3

### Version Specification
When creating a release, specify which version of artifacts is required:
- **Specific branch**: Choose artifacts from particular branch (main, develop, release/1.0)
- **Specific build version**: Select specific build number or ID
- **Tags**: Use Git tags or build tags to identify versions
- **Latest successful**: Most recent successful build
- **Latest with specific label**: Artifacts with particular labels or metadata

## Why Consolidate to Azure Artifacts?

### Benefits of Azure Artifacts

**Centralized Management:**
- Store Maven, npm, NuGet, Python, and Universal Packages together in one place
- Cloud-hosted: No infrastructure to maintain
- Indexed and searchable: Easily find packages
- Integrated with Azure DevOps: Seamless access from pipelines and projects

**Eliminate Binary Storage in Source Control:**

Why avoid binaries in source control:
- ❌ **Repository bloat**: Large files make repositories slow to clone and pull
- ❌ **Version control overhead**: Git isn't optimized for binary files
- ❌ **Merge conflicts**: Binary files can't be easily merged
- ❌ **Security**: Binaries in source control are harder to secure and scan

Universal Packages as alternative:
- ✅ **Versioned storage**: Store any file type with version tracking
- ✅ **Secure**: Control access with permissions
- ✅ **Efficient**: Optimized for binary storage
- ✅ **Integrated**: Works seamlessly with Azure Pipelines

### Comprehensive Package Management

**Package Types Supported:**
- NuGet: .NET packages
- npm: JavaScript/Node.js packages
- Maven: Java packages
- Python: PyPI packages
- Universal Packages: Any file type

**Integration Benefits:**
- Build integration: Reference packages during builds
- Release integration: Deploy packages as part of releases
- Versioning: Track package versions across environments
- Testing: Test with specific package versions

## Assessing Your Current Artifact Landscape

### Inventory Checklist

**Package Repositories:**
- Self-hosted: NuGet Server, npm registries, Nexus, Artifactory, Archiva
- Third-party SaaS: MyGet, JFrog Cloud, GitHub Packages
- File shares: Network drives containing packages or binaries

**Container Registries:**
- Self-hosted: Docker Registry, Harbor
- Cloud-based: Docker Hub, Azure Container Registry, Amazon ECR

**Source Control:**
- Binary files: Large files stored in Git or TFVC
- Build outputs: Compiled binaries committed to repositories

### Assessment Questions
For each repository, consider:
- **Usage**: Who uses this repository? Which projects depend on it?
- **Size**: How many packages? What's the total storage size?
- **Criticality**: Is this repository critical to operations?
- **Access control**: Who has permissions? How is access managed?
- **Integration**: What tools and pipelines reference this repository?
- **Maintenance**: Who maintains the infrastructure?

### Migration Priorities
Prioritize migration based on:
- **High maintenance burden**: Repositories requiring significant operational overhead
- **Security concerns**: Repositories with weak access controls or vulnerabilities
- **Integration opportunities**: Repositories that would benefit from Azure DevOps integration
- **Cost savings**: Self-hosted solutions with high infrastructure costs

## Quick Reference

| Current Location | Issues | Azure Artifacts Benefit |
|------------------|--------|------------------------|
| **File shares** | VPN required, no versioning | Cloud-hosted, versioned |
| **Source control** | Repository bloat, slow clones | Optimized for binaries |
| **Self-hosted servers** | Maintenance overhead | Managed service |
| **Multiple registries** | Fragmented management | Centralized |

## Critical Notes
- 🎯 Assess all artifact locations before migration planning
- 💡 Universal Packages provide alternative to binaries in source control
- ⚠️ Prioritize migration based on maintenance burden and security concerns
- 📊 Azure Artifacts supports 5 package types in unified platform
- 🔄 Consider integration opportunities with Azure DevOps

[Learn More](https://learn.microsoft.com/en-us/training/modules/migrate-consolidating-secure-artifacts/2-identify-existing-artifact-repositories)
