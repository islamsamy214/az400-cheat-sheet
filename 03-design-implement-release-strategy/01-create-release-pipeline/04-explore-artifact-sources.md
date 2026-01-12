# Explore Artifact Sources

Artifacts are the deployable components of your application—the packages produced by build pipelines and deployed through release pipelines. Understanding artifact sources and their characteristics is essential for designing reliable continuous delivery workflows.

## What Are Artifacts?

**Definition**: An artifact is a deployable component of your application that can be deployed to one or more environments.

```
Artifact Examples by Application Type

Web Application:
  └── webapp.zip
      ├── compiled binaries (DLLs, assemblies)
      ├── static content (HTML, CSS, JS, images)
      ├── configuration templates
      └── deployment scripts

Container Application:
  └── myapp:1.2.3 (Docker image)
      ├── application layers
      ├── dependencies
      └── runtime environment

Database:
  └── database-package.dacpac
      ├── schema definitions
      ├── migration scripts
      └── data seed scripts

Infrastructure:
  └── infrastructure.zip
      ├── ARM templates
      ├── Terraform files
      └── configuration scripts
```

## Build Once, Deploy Many

The fundamental principle of continuous delivery is **build once, deploy multiple times** with the same immutable artifact.

```
Correct Approach (Build Once):

┌───────────────────────────────┐
│   Build Pipeline (Once)       │
│   ├── Compile code            │
│   ├── Run tests               │
│   ├── Package → artifact      │
│   └── Publish artifact        │
└───────────────────────────────┘
              ↓ (Same artifact)
      ┌───────┼───────┬────────┐
      ↓       ↓       ↓        ↓
    Dev     QA    Staging   Prod
 (config1) (config2) (config3) (config4)

Benefits:
✅ Consistent deployments across environments
✅ What's tested in QA is what goes to production
✅ Faster deployments (no compilation)
✅ Full traceability (artifact → build → commit)
```

```
Incorrect Approach (Rebuild for Each Environment):

Build for Dev → Deploy to Dev
Build for QA  → Deploy to QA    ❌ Different binaries!
Build for Prod → Deploy to Prod ❌ Never tested!

Problems:
❌ Different binaries in each environment
❌ QA tests version A, production gets version B
❌ Compilation differences cause unexpected behavior
❌ No traceability (which code is in production?)
```

## Immutability Principle

**Artifacts must be immutable**—contents never change after creation. Only configuration should differ between environments.

### Configuration vs. Code Separation

```
Artifact (Immutable):
├── application.dll (v1.2.3)
├── libraries/*.dll
└── static-content/**/*

Configuration (Environment-Specific):
├── appsettings.dev.json       → Injected at deployment
├── appsettings.qa.json        → Injected at deployment
├── appsettings.production.json → Injected at deployment
```

**Example - ASP.NET Core**:
```csharp
// Artifact includes this code (immutable)
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Configuration loaded from environment-specific file
        var connectionString = Configuration.GetConnectionString("Database");
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(connectionString));
    }
}
```

```json
// appsettings.dev.json (injected at deployment)
{
  "ConnectionStrings": {
    "Database": "Server=dev-sql;Database=MyApp;..."
  }
}

// appsettings.production.json (injected at deployment)
{
  "ConnectionStrings": {
    "Database": "Server=prod-sql;Database=MyApp;..."
  }
}
```

## Artifact Source Types

Azure Pipelines supports multiple artifact sources, each suited to specific scenarios.

### 1. Build Artifacts (Recommended)

**Most common and recommended artifact source**—packages produced by build pipelines.

```
Build Pipeline → Artifact Repository → Release Pipeline

┌─────────────────────────────┐
│  Build Pipeline             │
│  ├── Compile                │
│  ├── Test                   │
│  ├── Package                │
│  └── Publish Artifact ───┐  │
└──────────────────────────│──┘
                           ↓
              ┌────────────────────────┐
              │  Artifact Repository   │
              │  (Secure Storage)      │
              │  - Azure Artifacts     │
              │  - Azure Storage       │
              │  - Build Drop Location │
              └────────────────────────┘
                           ↓
              ┌────────────────────────┐
              │  Release Pipeline      │
              │  Download Artifact ──→ │
              │  Deploy to Env         │
              └────────────────────────┘
```

**Characteristics**:
- ✅ **Immutable**: Package contents never change
- ✅ **Versioned**: Each build produces uniquely versioned artifact
- ✅ **Traceable**: Links back to specific build and source commit
- ✅ **Secure**: Stored in secure, access-controlled locations
- ✅ **Auditable**: Complete history of what was deployed when

**Example Configuration**:
```yaml
# Build pipeline publishes artifact
- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'drop'

# Release pipeline consumes artifact
resources:
  pipelines:
  - pipeline: buildPipeline
    source: MyApp-CI
    trigger:
      branches:
        include:
        - main

steps:
- download: buildPipeline
  artifact: drop
```

**Storage Locations**:

| Location | Use Case | Retention |
|----------|----------|-----------|
| **Azure Pipelines** (default) | Small-medium artifacts (<100 MB) | 30 days default |
| **Azure Artifacts** | NuGet, npm, Maven packages | Unlimited |
| **Azure Blob Storage** | Large artifacts, long-term storage | Configurable |
| **File Share** | On-premises scenarios | Manual management |

### 2. Container Registry Artifacts

**For containerized applications**—Docker images stored in container registries.

```
Container Build → Registry → Deployment

┌─────────────────────────────┐
│  Build Pipeline             │
│  ├── docker build           │
│  ├── docker tag             │
│  └── docker push ────────┐  │
└──────────────────────────│──┘
                           ↓
              ┌────────────────────────┐
              │  Container Registry    │
              │  - Azure Container Reg │
              │  - Docker Hub          │
              │  - GitHub Container Reg│
              └────────────────────────┘
                           ↓
              ┌────────────────────────┐
              │  Release Pipeline      │
              │  docker pull ────────→ │
              │  Deploy to AKS/ACI     │
              └────────────────────────┘
```

**Example**:
```yaml
# Build and push to Azure Container Registry
- task: Docker@2
  inputs:
    command: buildAndPush
    repository: myapp
    dockerfile: Dockerfile
    containerRegistry: myACR
    tags: |
      $(Build.BuildId)
      latest

# Release pipeline deploys from ACR
- task: KubernetesManifest@0
  inputs:
    action: deploy
    containers: myacr.azurecr.io/myapp:$(Build.BuildId)
```

**Benefits**:
- ✅ Optimized for container deployments
- ✅ Layer caching for faster pulls
- ✅ Geo-replication for multi-region deployments
- ✅ Security scanning integrated

### 3. Version Control (Git) Artifacts

**For script-based deployments**—directly from source repository.

```
Use Cases:
✅ PowerShell/Bash deployment scripts
✅ ARM/Terraform templates
✅ Configuration files
✅ Database migration scripts (SQL files)
❌ Compiled applications (use build artifacts instead)
```

**Example**:
```yaml
# Release pipeline references Git repository
resources:
  repositories:
  - repository: scripts
    type: git
    name: DeploymentScripts
    ref: refs/heads/main

steps:
- checkout: scripts
- task: PowerShell@2
  inputs:
    filePath: '$(Build.SourcesDirectory)/deploy.ps1'
```

**When to Use**:
- Simple script deployments (no compilation needed)
- Infrastructure as Code (Terraform, ARM templates)
- Configuration management (no binaries)
- Single-file deployments

**Limitations**:
- ❌ No immutable versioning (commit can be overwritten)
- ❌ No dedicated artifact storage
- ❌ Not suitable for compiled applications

### 4. Package Management Feeds

**For library/package deployments**—NuGet, npm, Maven, Python packages.

```
Package Types:

NuGet (.nupkg):
  └── MyLibrary.1.2.3.nupkg
      └── Deployed to: Azure Functions, libraries

npm (package.json):
  └── my-package@1.2.3.tgz
      └── Deployed to: Node.js applications

Maven (JAR/WAR):
  └── myapp-1.2.3.jar
      └── Deployed to: Java application servers

Python (wheel):
  └── mypackage-1.2.3-py3-none-any.whl
      └── Deployed to: Python environments
```

**Example - NuGet Package Deployment**:
```yaml
# Build pipeline publishes NuGet package
- task: NuGetCommand@2
  inputs:
    command: pack
    packagesToPack: 'src/**/*.csproj'
    versioningScheme: byBuildNumber

- task: NuGetCommand@2
  inputs:
    command: push
    packagesToPush: '$(Build.ArtifactStagingDirectory)/**/*.nupkg'
    nuGetFeedType: internal
    publishVstsFeed: 'MyFeed'

# Release pipeline installs package
- task: NuGetCommand@2
  inputs:
    command: install
    packageName: 'MyLibrary'
    version: '1.2.3'
```

### 5. Azure Artifacts

**Unified package management** for multiple package types in single feed.

```
Azure Artifacts Capabilities:

┌─────────────────────────────┐
│      Azure Artifacts        │
│  ┌─────────┬─────────────┐  │
│  │ NuGet   │ npm         │  │
│  ├─────────┼─────────────┤  │
│  │ Maven   │ Python      │  │
│  ├─────────┼─────────────┤  │
│  │ Universal Packages    │  │
│  └───────────────────────┘  │
│  Features:                  │
│  ✅ Versioning              │
│  ✅ Access control          │
│  ✅ Upstream sources        │
│  ✅ Retention policies      │
└─────────────────────────────┘
```

**Universal Packages** (any file type):
```bash
# Publish universal package
az artifacts universal publish \
  --organization https://dev.azure.com/myorg \
  --feed MyFeed \
  --name my-app \
  --version 1.2.3 \
  --path ./artifact-folder

# Download in release pipeline
az artifacts universal download \
  --organization https://dev.azure.com/myorg \
  --feed MyFeed \
  --name my-app \
  --version 1.2.3 \
  --path $(Pipeline.Workspace)
```

### 6. Network Shares / File Shares

**Legacy artifact source**—network file shares (UNC paths).

```
⚠️ Not Recommended for Production

Problems:
❌ Security risks (unauthorized access)
❌ No immutability guarantees
❌ Poor auditability
❌ No version control
❌ Manual management required
❌ Compliance issues

Only Use For:
✅ Legacy on-premises scenarios
✅ Temporary development environments
✅ Migration from legacy systems
```

**Example** (if absolutely necessary):
```yaml
# Download from file share
- task: CopyFiles@2
  inputs:
    sourceFolder: '\\fileserver\artifacts\myapp'
    contents: '**\*'
    targetFolder: '$(Pipeline.Workspace)/artifact'
```

## Artifact Source Comparison

| Source | Immutable | Versioned | Traceable | Secure | Best For |
|--------|-----------|-----------|-----------|--------|----------|
| **Build Artifacts** | ✅ | ✅ | ✅ | ✅ | Compiled apps, web apps |
| **Container Registry** | ✅ | ✅ | ✅ | ✅ | Docker/K8s deployments |
| **Git Repository** | ⚠️ | ⚠️ | ✅ | ✅ | Scripts, IaC templates |
| **Package Feeds** | ✅ | ✅ | ✅ | ✅ | Libraries, packages |
| **Azure Artifacts** | ✅ | ✅ | ✅ | ✅ | Multi-type packages |
| **File Shares** | ❌ | ❌ | ❌ | ❌ | Legacy systems only |

## Artifact Metadata and Traceability

**Every artifact should include metadata** for complete traceability:

```
Artifact Metadata Example:

MyApp-1.2.3.zip
├── Build Information:
│   ├── Build ID: 1234
│   ├── Build Number: 20260112.1
│   ├── Build Definition: MyApp-CI
│   └── Build Status: Succeeded
│
├── Source Information:
│   ├── Repository: MyApp
│   ├── Branch: refs/heads/main
│   ├── Commit: abc123def456789
│   ├── Author: john.doe@company.com
│   └── Commit Message: "Fix critical bug"
│
├── Version Information:
│   ├── Semantic Version: 1.2.3
│   ├── Assembly Version: 1.2.3.1234
│   └── File Version: 1.2.3.1234
│
└── Quality Information:
    ├── Unit Test Results: Passed (152/152)
    ├── Code Coverage: 85%
    ├── Security Scan: No vulnerabilities
    └── Quality Gate: Passed
```

**How to Include Metadata**:
```yaml
# Build pipeline sets build number with version
name: $(MajorVersion).$(MinorVersion).$(PatchVersion)$(Rev:.r)

variables:
  MajorVersion: 1
  MinorVersion: 2
  PatchVersion: 3

# Write metadata file
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      $metadata = @{
        BuildId = "$(Build.BuildId)"
        BuildNumber = "$(Build.BuildNumber)"
        SourceVersion = "$(Build.SourceVersion)"
        SourceBranch = "$(Build.SourceBranch)"
        Repository = "$(Build.Repository.Name)"
      }
      $metadata | ConvertTo-Json | Out-File metadata.json

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'drop'
```

## Critical Notes

🎯 **Build Once**: Never rebuild artifacts for different environments—use the same binary with different configurations.

💡 **Immutability**: Artifact contents must never change—only configuration should differ between environments.

⚠️ **Traceability**: Every artifact must link to source commit and build—required for auditing and compliance.

📊 **Secure Storage**: Store artifacts in access-controlled, secure locations—no network shares in production.

🔄 **Versioning**: Use semantic versioning (major.minor.patch) for clear version identification and dependency management.

✨ **Retention**: Configure retention policies—keep production artifacts longer, clean up dev/test artifacts.

## Quick Reference

### Artifact Source Selection Guide

```
What to Deploy?

Compiled Application (C#, Java, Go)
  → Build Artifacts

Container Application (Docker)
  → Container Registry

Script-Based Deployment (PowerShell, Bash)
  → Git Repository

Library/Package (NuGet, npm, Maven)
  → Package Feed / Azure Artifacts

Infrastructure as Code (Terraform, ARM)
  → Git Repository or Build Artifacts

Database (DACPAC, migration scripts)
  → Build Artifacts
```

### Configuration Management Patterns

```
Development:
  Artifact: myapp-1.2.3.zip (immutable)
  Config:  dev.config (connection strings, API keys)
  
QA:
  Artifact: myapp-1.2.3.zip (same!)
  Config:  qa.config (different values)
  
Production:
  Artifact: myapp-1.2.3.zip (same!)
  Config:  prod.config (production values)
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline-devops/4-explore-artifact-sources)
