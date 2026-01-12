# Use Multiple Repositories in Pipeline

## Key Concepts

Azure Pipelines can check out and build code from multiple repositories in a single pipeline run. This enables monorepo patterns, shared library integration, microservices builds, and template consumption from centralized repositories.

### Multi-Repository Scenarios

| Scenario | Use Case | Checkout Pattern |
|----------|----------|------------------|
| **Monorepo** | Multiple projects in one repo | Single checkout with path filtering |
| **Shared Libraries** | Common code across projects | Multiple checkouts (main + library) |
| **Microservices** | Independent services in separate repos | Multiple checkouts (one per service) |
| **Templates** | Centralized pipeline templates | Repository resource (no checkout) |
| **Multi-Platform** | Platform-specific code in separate repos | Selective checkout based on platform |

### Supported Repository Types

- Azure Repos Git
- GitHub
- GitHub Enterprise Server
- Bitbucket Cloud
- Other Git repositories

## Checkout Behavior

### Default Checkout Behavior

**Regular Jobs**:
- Automatically check out the triggering repository (`self`)
- Checkout happens at the start of each job
- Default location: `$(Build.SourcesDirectory)`

**Deployment Jobs**:
- Do NOT automatically check out any repository
- Must explicitly specify `checkout: self` or other repositories
- Prevents accidental source code access in deployment scenarios

### Controlling Checkout

```yaml
jobs:
- job: Build
  steps:
  - checkout: self  # Explicit checkout (default for regular jobs)
    clean: true
    fetchDepth: 1
```

```yaml
jobs:
- job: NoBuild
  steps:
  - checkout: none  # Skip checkout entirely
  - script: echo "No source code needed"
```

## Repository Resource Definition

### Basic Repository Resource

```yaml
resources:
  repositories:
  - repository: SharedLibrary  # Alias for reference
    type: git  # git, github, githubenterprise, bitbucket
    name: MyProject/SharedLibrary  # Project/Repo for Azure Repos
    ref: refs/heads/main  # Branch, tag, or commit SHA
```

### GitHub Repository

```yaml
resources:
  repositories:
  - repository: GitHubRepo
    type: github
    name: myorg/my-repo  # org/repo
    ref: refs/heads/main
    endpoint: GitHubConnection  # Service connection name (required for GitHub)
```

### Bitbucket Repository

```yaml
resources:
  repositories:
  - repository: BitbucketRepo
    type: bitbucket
    name: myworkspace/my-repo
    ref: refs/heads/main
    endpoint: BitbucketConnection
```

## Multiple Repository Checkout

### Checkout Multiple Repositories

```yaml
resources:
  repositories:
  - repository: SharedLibrary
    type: git
    name: MyProject/SharedLibrary
  
  - repository: Tools
    type: git
    name: MyProject/BuildTools

jobs:
- job: BuildWithDependencies
  steps:
  # Checkout triggering repository
  - checkout: self
  
  # Checkout additional repositories
  - checkout: SharedLibrary
  - checkout: Tools
  
  - script: |
      echo "Main repo: $(Build.SourcesDirectory)"
      echo "Shared library: $(Build.SourcesDirectory)/SharedLibrary"
      echo "Tools: $(Build.SourcesDirectory)/Tools"
    displayName: 'Show checkout locations'
  
  - script: |
      # Build main application with shared library
      dotnet build $(Build.SourcesDirectory)/MyApp.sln
    displayName: 'Build application'
```

### Checkout Order and Directory Structure

When checking out multiple repositories:

```
$(Build.SourcesDirectory)/
├── <main-repo-files>        # self repository (root)
├── SharedLibrary/           # Additional repo 1
│   └── <library-files>
└── Tools/                   # Additional repo 2
    └── <tool-files>
```

**Important**: The `self` repository is checked out to the root of `$(Build.SourcesDirectory)`, while other repositories are checked out to subdirectories named after their alias.

### Checkout with Custom Path

```yaml
resources:
  repositories:
  - repository: Docs
    type: git
    name: MyProject/Documentation

jobs:
- job: BuildDocs
  steps:
  - checkout: self
    path: src  # Custom path: $(Build.SourcesDirectory)/src
  
  - checkout: Docs
    path: docs  # Custom path: $(Build.SourcesDirectory)/docs
  
  - script: |
      echo "Source: $(Build.SourcesDirectory)/src"
      echo "Docs: $(Build.SourcesDirectory)/docs"
```

## Inline Repository Syntax

For simple scenarios, you can check out repositories without defining them in the `resources` section.

### Inline Checkout (Azure Repos)

```yaml
jobs:
- job: Build
  steps:
  - checkout: git://MyProject/SharedLibrary@refs/heads/main
  - checkout: self
  
  - script: echo "Building with shared library"
```

### Inline Checkout (GitHub)

```yaml
jobs:
- job: Build
  steps:
  - checkout: github://myorg/my-repo@main
    endpoint: GitHubConnection  # Service connection required
  
  - checkout: self
```

## Multi-Repository Patterns

### Pattern 1: Monorepo with Shared Library

```yaml
# Main application repo consumes shared library
resources:
  repositories:
  - repository: SharedLib
    type: git
    name: MyOrg/SharedLibrary
    ref: refs/tags/v2.1.0  # Pin to specific version

jobs:
- job: Build
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - checkout: self
  - checkout: SharedLib
  
  - script: |
      echo "Building application with shared library v2.1.0"
      dotnet build --configuration Release
    displayName: 'Build application'
    workingDirectory: $(Build.SourcesDirectory)
```

### Pattern 2: Microservices Multi-Repo Build

```yaml
# Build multiple microservices from separate repositories
resources:
  repositories:
  - repository: AuthService
    type: git
    name: MyProject/auth-service
  
  - repository: ApiService
    type: git
    name: MyProject/api-service
  
  - repository: WebService
    type: git
    name: MyProject/web-service

jobs:
- job: BuildAuthService
  steps:
  - checkout: AuthService
  - script: dotnet build
    workingDirectory: $(Build.SourcesDirectory)/AuthService

- job: BuildApiService
  steps:
  - checkout: ApiService
  - script: dotnet build
    workingDirectory: $(Build.SourcesDirectory)/ApiService

- job: BuildWebService
  steps:
  - checkout: WebService
  - script: npm install && npm run build
    workingDirectory: $(Build.SourcesDirectory)/WebService
```

### Pattern 3: Shared Pipeline Templates

```yaml
# Consume pipeline templates from centralized repository
resources:
  repositories:
  - repository: templates
    type: git
    name: PipelineTemplates
    ref: refs/heads/v2.0

# Use templates from repository resource
stages:
- template: stages/build-stage.yml@templates
  parameters:
    buildConfiguration: 'Release'

- template: stages/test-stage.yml@templates
  parameters:
    testFramework: 'xUnit'

- template: stages/deploy-stage.yml@templates
  parameters:
    environment: 'production'
```

### Pattern 4: Multi-Repo Integration Tests

```yaml
# Integration tests requiring multiple components
resources:
  repositories:
  - repository: Frontend
    type: git
    name: MyProject/Frontend
  
  - repository: Backend
    type: git
    name: MyProject/Backend
  
  - repository: TestSuite
    type: git
    name: MyProject/IntegrationTests

jobs:
- job: IntegrationTests
  steps:
  - checkout: Frontend
  - checkout: Backend
  - checkout: TestSuite
  
  - script: |
      # Start backend service
      cd $(Build.SourcesDirectory)/Backend
      dotnet run &
      sleep 10
    displayName: 'Start backend'
  
  - script: |
      # Start frontend service
      cd $(Build.SourcesDirectory)/Frontend
      npm start &
      sleep 10
    displayName: 'Start frontend'
  
  - script: |
      # Run integration tests
      cd $(Build.SourcesDirectory)/TestSuite
      npm test
    displayName: 'Run integration tests'
```

## Triggers with Multiple Repositories

### Triggering from Multiple Repositories

```yaml
# Trigger pipeline when any of multiple repos change
trigger:
  branches:
    include:
    - main

resources:
  repositories:
  - repository: SharedLibrary
    type: git
    name: MyProject/SharedLibrary
    trigger:  # Trigger this pipeline when SharedLibrary changes
      branches:
        include:
        - main
        - releases/*

jobs:
- job: Build
  steps:
  - checkout: self
  - checkout: SharedLibrary
  - script: dotnet build
```

### Conditional Checkout Based on Changes

```yaml
resources:
  repositories:
  - repository: Docs
    type: git
    name: MyProject/Documentation

jobs:
- job: Build
  steps:
  - checkout: self
  
  # Conditionally checkout docs if docs repo triggered the build
  - ${{ if eq(variables['Build.Repository.Name'], 'MyProject/Documentation') }}:
    - checkout: Docs
    - script: echo "Building documentation"
  
  - script: echo "Building application"
```

## Checkout Options

### Advanced Checkout Configuration

```yaml
jobs:
- job: Build
  steps:
  - checkout: self
    clean: true  # Delete repo before checkout
    fetchDepth: 1  # Shallow clone (single commit)
    lfs: true  # Enable Git LFS
    submodules: true  # Checkout Git submodules
    persistCredentials: true  # Keep credentials for subsequent Git operations
    path: custom/path  # Custom checkout path
```

### Checkout Options Reference

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `clean` | boolean | `false` | Clean workspace before checkout |
| `fetchDepth` | number | `0` (full) | Shallow clone depth (performance) |
| `lfs` | boolean | `false` | Enable Git Large File Storage |
| `submodules` | boolean/string | `false` | Checkout submodules (`true`, `recursive`) |
| `persistCredentials` | boolean | `true` | Keep credentials for later steps |
| `path` | string | (varies) | Custom checkout directory |

## Multi-Repo Build Script Example

### Complete Multi-Repo CI Pipeline

```yaml
# azure-pipelines-multi-repo.yml
trigger:
  branches:
    include:
    - main
    - develop

resources:
  repositories:
  # Shared libraries
  - repository: CoreLib
    type: git
    name: Libraries/Core
    ref: refs/tags/v3.0.0
  
  - repository: UtilsLib
    type: git
    name: Libraries/Utils
    ref: refs/tags/v2.5.1
  
  # Build tools from GitHub
  - repository: BuildScripts
    type: github
    name: myorg/build-scripts
    ref: refs/heads/main
    endpoint: GitHubConnection

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '8.x'

stages:
- stage: Build
  displayName: 'Build Application'
  jobs:
  - job: BuildJob
    displayName: 'Build with Dependencies'
    pool:
      vmImage: 'ubuntu-latest'
    
    steps:
    # Checkout all required repositories
    - checkout: self
      displayName: 'Checkout main application'
      clean: true
      fetchDepth: 1
    
    - checkout: CoreLib
      displayName: 'Checkout Core Library'
      fetchDepth: 1
    
    - checkout: UtilsLib
      displayName: 'Checkout Utils Library'
      fetchDepth: 1
    
    - checkout: BuildScripts
      displayName: 'Checkout Build Scripts'
      fetchDepth: 1
    
    # Install .NET SDK
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        version: $(dotnetVersion)
    
    # Build libraries first
    - script: |
        echo "Building Core Library"
        cd $(Build.SourcesDirectory)/CoreLib
        dotnet build --configuration $(buildConfiguration)
      displayName: 'Build Core Library'
    
    - script: |
        echo "Building Utils Library"
        cd $(Build.SourcesDirectory)/UtilsLib
        dotnet build --configuration $(buildConfiguration)
      displayName: 'Build Utils Library'
    
    # Copy library outputs to main application
    - script: |
        echo "Copying library binaries"
        mkdir -p $(Build.SourcesDirectory)/lib
        cp -r $(Build.SourcesDirectory)/CoreLib/bin/$(buildConfiguration)/* $(Build.SourcesDirectory)/lib/
        cp -r $(Build.SourcesDirectory)/UtilsLib/bin/$(buildConfiguration)/* $(Build.SourcesDirectory)/lib/
      displayName: 'Copy library outputs'
    
    # Build main application
    - script: |
        echo "Building main application"
        cd $(Build.SourcesDirectory)
        dotnet restore
        dotnet build --configuration $(buildConfiguration) --no-restore
      displayName: 'Build application'
    
    # Run build validation script from GitHub repo
    - script: |
        echo "Running build validation"
        bash $(Build.SourcesDirectory)/BuildScripts/validate-build.sh
      displayName: 'Validate build'
    
    # Run tests
    - script: |
        cd $(Build.SourcesDirectory)
        dotnet test --configuration $(buildConfiguration) --no-build --logger trx
      displayName: 'Run tests'
    
    # Publish test results
    - task: PublishTestResults@2
      displayName: 'Publish test results'
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/*.trx'
        failTaskOnFailedTests: true
    
    # Publish artifacts
    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'

- stage: Test
  displayName: 'Integration Tests'
  dependsOn: Build
  jobs:
  - job: IntegrationTest
    steps:
    - checkout: self
    - checkout: CoreLib
    - checkout: UtilsLib
    
    - script: |
        echo "Running integration tests across repositories"
        dotnet test --filter Category=Integration
      displayName: 'Integration tests'
```

## Critical Notes

🎯 **Default Behavior**: Regular jobs automatically checkout `self`. Deployment jobs do NOT checkout any repository by default (security).

💡 **Directory Structure**: `self` checks out to root of `$(Build.SourcesDirectory)`, other repos to subdirectories named after their alias.

⚠️ **Performance**: Each checkout adds 10-30 seconds. Use `fetchDepth: 1` for shallow clones to improve performance (50-90% faster).

📊 **Template Repositories**: Template-only repositories don't need checkout; they're accessed via `@alias` syntax in template references.

🔄 **Version Pinning**: Pin shared library repositories to specific tags (`refs/tags/v2.1.0`) for stable builds. Use branches for development pipelines.

✨ **Trigger Combinations**: You can trigger a pipeline from changes in the main repo OR any repository resource by setting `trigger:` on the resource.

## Quick Reference

### Checkout Patterns

```yaml
# Default (self only)
- checkout: self

# Skip checkout
- checkout: none

# Multiple repositories
- checkout: self
- checkout: RepoAlias

# Inline Azure Repos
- checkout: git://Project/Repo@branch

# Inline GitHub
- checkout: github://org/repo@branch
  endpoint: GitHubConnection
```

### Directory Variables

```bash
# Main repository root
$(Build.SourcesDirectory)

# Additional repository
$(Build.SourcesDirectory)/<RepoAlias>

# Custom path checkout
$(Build.SourcesDirectory)/<CustomPath>
```

### Common Multi-Repo Scenarios

| Scenario | Pattern | Checkout Count |
|----------|---------|----------------|
| Shared library | `self + library` | 2 |
| Microservices | `service1 + service2 + service3` | 3+ |
| Templates only | No checkout (resource only) | 0 |
| Monorepo split | `frontend + backend + shared` | 3 |

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-azure-pipelines/use-multiple-repositories-pipeline/)
