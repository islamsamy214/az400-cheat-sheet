# Explore YAML Resources

## Key Concepts

YAML resources enable pipelines to consume artifacts from other pipelines, use external repositories for templates, and run jobs in containers. Resources provide a structured way to declare dependencies and integrate with external systems.

### Resource Types

| Type | Purpose | Trigger Support | Use Case |
|------|---------|-----------------|----------|
| **pipelines** | Consume artifacts from other pipelines | Yes | Artifact promotion, multi-stage releases |
| **repositories** | Reference external Git repos | No | Shared templates, monorepo builds |
| **containers** | Define container images for jobs | No | Consistent build environments |
| **packages** | NuGet/npm/Maven packages | No | Package versioning (limited support) |

## Pipeline Resources

Pipeline resources allow you to consume artifacts from other pipelines and optionally trigger on their completion.

### Basic Pipeline Resource

```yaml
resources:
  pipelines:
  - pipeline: SecurityScan  # Alias for this resource
    source: SecurityPipeline  # Source pipeline name
    trigger: true  # Trigger this pipeline when source completes
```

### Complete Pipeline Resource Configuration

```yaml
resources:
  pipelines:
  - pipeline: BuildPipeline
    source: MyApp-CI  # Pipeline name in Azure DevOps
    project: MyProject  # Optional: project name (if different)
    branch: main  # Trigger only on specific branch
    trigger:
      branches:
        include:
        - main
        - releases/*
      stages:
        - Build
        - Test  # Trigger only when these stages complete
    version: latest  # Use specific version: 'latest', 'latestFromBranch', 'specific'
    tags:
    - Production  # Only trigger on builds with specific tags
```

### Downloading Pipeline Artifacts

```yaml
resources:
  pipelines:
  - pipeline: BuildArtifacts
    source: MyApp-Build
    trigger: false

jobs:
- job: Deploy
  steps:
  # Download all artifacts from pipeline resource
  - download: BuildArtifacts
    artifact: drop
  
  # Use downloaded artifacts
  - script: |
      echo "Artifacts location: $(Pipeline.Workspace)/BuildArtifacts/drop"
      ls -la $(Pipeline.Workspace)/BuildArtifacts/drop
    displayName: 'List artifacts'
```

### Multi-Pipeline Dependencies

```yaml
# Deployment pipeline that consumes from multiple build pipelines
resources:
  pipelines:
  - pipeline: FrontendBuild
    source: Frontend-CI
    trigger: true
  
  - pipeline: BackendBuild
    source: Backend-CI
    trigger: true
  
  - pipeline: DatabaseBuild
    source: Database-CI
    trigger: true

jobs:
- job: DeployFullStack
  steps:
  - download: FrontendBuild
    artifact: frontend-drop
  
  - download: BackendBuild
    artifact: backend-drop
  
  - download: DatabaseBuild
    artifact: db-scripts
  
  - script: |
      echo "Deploying frontend from $(Pipeline.Workspace)/FrontendBuild/frontend-drop"
      echo "Deploying backend from $(Pipeline.Workspace)/BackendBuild/backend-drop"
      echo "Running DB scripts from $(Pipeline.Workspace)/DatabaseBuild/db-scripts"
    displayName: 'Deploy full stack'
```

## Repository Resources

Repository resources enable pipelines to check out multiple repositories or consume templates from external sources.

### Basic Repository Resource

```yaml
resources:
  repositories:
  - repository: templates  # Alias
    type: git  # git, github, githubenterprise, bitbucket
    name: SharedTemplates  # Repo name
    ref: refs/heads/main  # Branch, tag, or commit
```

### Repository Types

#### Azure Repos Git

```yaml
resources:
  repositories:
  - repository: SharedLibrary
    type: git
    name: MyProject/SharedLibrary  # Project/RepoName
    ref: refs/heads/main
```

#### GitHub

```yaml
resources:
  repositories:
  - repository: GitHubTemplates
    type: github
    name: myorg/pipeline-templates  # org/repo
    ref: refs/heads/main
    endpoint: GitHubConnection  # Service connection name
```

#### Bitbucket Cloud

```yaml
resources:
  repositories:
  - repository: BitbucketTemplates
    type: bitbucket
    name: myworkspace/my-repo
    ref: refs/heads/main
    endpoint: BitbucketConnection
```

### Using Repository Resources

#### Template Consumption

```yaml
resources:
  repositories:
  - repository: templates
    type: git
    name: SharedTemplates
    ref: refs/heads/v2.0

stages:
- template: stages/build-stage.yml@templates  # @alias syntax
  parameters:
    buildConfiguration: 'Release'

- template: stages/deploy-stage.yml@templates
  parameters:
    environment: 'production'
```

#### Multi-Repository Checkout

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
- job: Build
  steps:
  - checkout: self  # Checkout triggering repository
  - checkout: SharedLibrary  # Checkout SharedLibrary
  - checkout: Tools  # Checkout BuildTools
  
  - script: |
      echo "Self repo: $(Build.SourcesDirectory)"
      echo "Shared library: $(Build.SourcesDirectory)/SharedLibrary"
      echo "Tools: $(Build.SourcesDirectory)/Tools"
    displayName: 'Show repository locations'
```

## Container Resources

Container resources define Docker images for running jobs in consistent, isolated environments.

### Basic Container Resource

```yaml
resources:
  containers:
  - container: linux  # Alias
    image: ubuntu:20.04
```

### Container with Registry Authentication

```yaml
resources:
  containers:
  - container: build
    image: myregistry.azurecr.io/build-image:latest
    endpoint: MyACRConnection  # Service connection to Azure Container Registry
    
  - container: test
    image: mcr.microsoft.com/dotnet/sdk:8.0
    options: --cpus 2 --memory 4g  # Docker run options
```

### Using Containers in Jobs

#### Single Container Job

```yaml
resources:
  containers:
  - container: dotnet
    image: mcr.microsoft.com/dotnet/sdk:8.0

jobs:
- job: BuildInContainer
  container: dotnet  # Run entire job in container
  steps:
  - script: dotnet --version
    displayName: 'Check .NET version'
  
  - script: |
      dotnet restore
      dotnet build --configuration Release
    displayName: 'Build application'
```

#### Service Containers

```yaml
resources:
  containers:
  - container: postgres
    image: postgres:15
    env:
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
      POSTGRES_DB: testdb
    ports:
    - 5432:5432

jobs:
- job: IntegrationTests
  services:
    postgres: postgres  # Run postgres as service container
  steps:
  - script: |
      echo "Waiting for PostgreSQL to be ready"
      sleep 10
    displayName: 'Wait for database'
  
  - script: |
      # Run tests against PostgreSQL service
      dotnet test --configuration Release --filter Category=Integration
    displayName: 'Run integration tests'
    env:
      ConnectionString: 'Host=postgres;Port=5432;Database=testdb;Username=testuser;Password=testpass'
```

### Multi-Container Scenario

```yaml
resources:
  containers:
  - container: build
    image: node:18
  
  - container: redis
    image: redis:7
    ports:
    - 6379:6379
  
  - container: mongo
    image: mongo:6
    ports:
    - 27017:27017
    env:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret

jobs:
- job: TestWithServices
  container: build  # Build container for job
  services:
    redis: redis  # Redis service container
    mongo: mongo  # MongoDB service container
  
  steps:
  - script: |
      echo "Installing dependencies"
      npm install
    displayName: 'Install dependencies'
  
  - script: |
      echo "Running tests with Redis and MongoDB"
      npm test
    displayName: 'Run tests'
    env:
      REDIS_HOST: redis
      REDIS_PORT: 6379
      MONGO_HOST: mongo
      MONGO_PORT: 27017
```

## Complete Resources Example

### Multi-Resource Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main

resources:
  # Pipeline resources for artifact consumption
  pipelines:
  - pipeline: SecurityScanResults
    source: SecurityScan-Pipeline
    trigger:
      branches:
        include:
        - main
      stages:
        - Scan
  
  # Repository resources for templates and shared code
  repositories:
  - repository: templates
    type: git
    name: SharedTemplates
    ref: refs/heads/v2.0
  
  - repository: SharedLibrary
    type: github
    name: myorg/shared-library
    ref: refs/heads/main
    endpoint: GitHubConnection
  
  # Container resources for consistent environments
  containers:
  - container: buildImage
    image: mcr.microsoft.com/dotnet/sdk:8.0
  
  - container: testDb
    image: postgres:15
    env:
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
      POSTGRES_DB: testdb
    ports:
    - 5432:5432

variables:
- template: variables/common.yml@templates

stages:
# Build stage using templates from repository resource
- template: stages/build-stage.yml@templates
  parameters:
    container: buildImage
    buildConfiguration: 'Release'

# Test stage with service container
- stage: Test
  dependsOn: Build
  jobs:
  - job: IntegrationTests
    container: buildImage
    services:
      postgres: testDb
    steps:
    - checkout: self
    - checkout: SharedLibrary
    
    - script: |
        echo "Running integration tests"
        dotnet test --configuration Release
      displayName: 'Run tests'
      env:
        DB_CONNECTION: 'Host=postgres;Port=5432;Database=testdb;Username=testuser;Password=testpass'

# Deploy stage consuming security scan results
- stage: Deploy
  dependsOn: Test
  jobs:
  - job: DeployApp
    steps:
    - download: SecurityScanResults
      artifact: scan-results
    
    - script: |
        echo "Checking security scan results"
        cat $(Pipeline.Workspace)/SecurityScanResults/scan-results/report.json
      displayName: 'Review security scan'
    
    - script: echo "Deploying application"
      displayName: 'Deploy'
```

## Resource Variables

Resources expose predefined variables for accessing resource metadata.

### Pipeline Resource Variables

```yaml
resources:
  pipelines:
  - pipeline: BuildArtifacts
    source: MyApp-Build

jobs:
- job: Deploy
  steps:
  - script: |
      echo "Pipeline: $(resources.pipeline.BuildArtifacts.pipelineName)"
      echo "Run ID: $(resources.pipeline.BuildArtifacts.runID)"
      echo "Run Name: $(resources.pipeline.BuildArtifacts.runName)"
      echo "Source Branch: $(resources.pipeline.BuildArtifacts.sourceBranch)"
      echo "Source Version: $(resources.pipeline.BuildArtifacts.sourceCommit)"
    displayName: 'Pipeline resource info'
```

### Repository Resource Variables

```yaml
resources:
  repositories:
  - repository: SharedLibrary
    type: git
    name: MyProject/SharedLibrary

jobs:
- job: Build
  steps:
  - checkout: SharedLibrary
  
  - script: |
      echo "Repo ID: $(resources.repository.SharedLibrary.id)"
      echo "Repo Name: $(resources.repository.SharedLibrary.name)"
      echo "Repo Type: $(resources.repository.SharedLibrary.type)"
      echo "Repo Ref: $(resources.repository.SharedLibrary.ref)"
    displayName: 'Repository resource info'
```

## Critical Notes

🎯 **Pipeline Triggers**: Use `trigger: true` on pipeline resources to automatically start deployment pipelines when build pipelines complete (CD pattern).

💡 **Container Performance**: Containers add 30-60s startup time but provide consistent environments. Use for complex dependencies or multi-service testing.

⚠️ **Repository Checkout**: Checking out multiple repositories places them in separate directories under `$(Build.SourcesDirectory)`. Default is `$(Build.SourcesDirectory)/<alias>`.

📊 **Service Containers**: Service containers are long-running containers (databases, caches) accessible by hostname matching their alias. Use for integration tests.

🔄 **Template Versioning**: Pin template repository resources to specific tags or commits (`ref: refs/tags/v2.0.1`) for production pipelines to ensure stability.

✨ **Resource Downloads**: Pipeline artifacts are downloaded to `$(Pipeline.Workspace)/<alias>/<artifact>`. Use `download` step to explicitly control artifact download.

## Quick Reference

### Resource Syntax

```yaml
resources:
  pipelines:
  - pipeline: <alias>
    source: <pipelineName>
    trigger: true|false

  repositories:
  - repository: <alias>
    type: git|github|bitbucket
    name: <repoName>
    ref: <branch|tag|commit>

  containers:
  - container: <alias>
    image: <imageName>
    endpoint: <serviceConnection>
```

### Common Container Images

| Purpose | Image | Use Case |
|---------|-------|----------|
| .NET | `mcr.microsoft.com/dotnet/sdk:8.0` | .NET builds |
| Node.js | `node:18` | Node.js builds |
| Python | `python:3.11` | Python builds |
| PostgreSQL | `postgres:15` | Database tests |
| Redis | `redis:7` | Cache tests |
| MongoDB | `mongo:6` | NoSQL tests |

### Download Locations

```bash
# Pipeline artifacts
$(Pipeline.Workspace)/<pipelineAlias>/<artifactName>

# Repositories
$(Build.SourcesDirectory)  # Self (triggering repo)
$(Build.SourcesDirectory)/<repoAlias>  # Other repos
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-azure-pipelines/explore-yaml-resources/)
