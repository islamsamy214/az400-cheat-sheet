# Explore Azure Pipelines Integration with GitHub

Integrating GitHub repositories with Azure Pipelines enables CI/CD workflows from your GitHub projects. Understanding authentication methods and naming conventions ensures smooth setup.

## Connection Overview

**Azure Pipelines ↔ GitHub Integration**:
```
GitHub Repository → Azure Pipelines → Build/Deploy → Azure Resources
```

**Benefits**:
- Use GitHub for source control
- Leverage Azure Pipelines for CI/CD
- Single workflow from commit to production
- GitHub Actions alternative with more features

## Setting Up Integration

### Step 1: Create Pipeline from GitHub Repo

```
1. Azure DevOps → Pipelines → New Pipeline
2. Select "GitHub" as code location
3. Authorize Azure Pipelines app (OAuth)
4. Select repository
5. Configure pipeline (YAML template or existing)
6. Save and run
```

### Step 2: Authentication Methods

Azure Pipelines needs permission to access GitHub repo. Three methods available:

#### Method 1: GitHub App (Recommended ⭐)

**Benefits**:
- Fine-grained permissions
- Better security
- Explicit repo selection
- Easier revocation

**Setup**:
```
1. Install Azure Pipelines GitHub App
2. Select repositories to grant access
3. Azure Pipelines receives installation token
4. Token auto-refreshes
```

**Pipeline automatically connects**:
```yaml
# azure-pipelines.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- checkout: self  # Uses GitHub App auth
- script: npm install
```

#### Method 2: OAuth

**Use Case**: Quick setup, less granular control

**Process**:
```
1. User clicks "Authorize Azure Pipelines"
2. GitHub OAuth flow
3. Azure Pipelines receives OAuth token
4. Token stored in Azure DevOps
```

**Limitations**:
- Access to all user's repos
- Less secure than GitHub App
- Token tied to individual user

#### Method 3: Personal Access Token (PAT)

**Use Case**: Manual setup, service connections, older workflows

**Setup**:
```
1. GitHub → Settings → Developer settings → Personal access tokens
2. Generate token with repo scope
3. Azure DevOps → Project Settings → Service Connections
4. New service connection → GitHub → PAT authentication
5. Enter PAT
```

**Pipeline usage**:
```yaml
resources:
  repositories:
  - repository: MyGitHubRepo
    type: github
    endpoint: GitHubServiceConnection  # PAT-based connection
    name: myorg/myrepo
```

**PAT Scopes Required**:
- `repo` (full control of private repositories)
- `admin:repo_hook` (for webhook creation)

## Authentication Method Comparison

| Method | Security | Granularity | Token Management | Best For |
|--------|----------|-------------|------------------|----------|
| **GitHub App** | ⭐⭐⭐ | Per-repo | Auto-refresh | New pipelines, best practice |
| **OAuth** | ⭐⭐ | All user repos | User-tied | Quick setup, personal projects |
| **PAT** | ⭐ | All or none | Manual renewal | Service accounts, specific needs |

## Repository Naming Conventions

### GitHub Repository Reference

**Format**: `<organization-or-user>/<repository-name>`

**Examples**:
```yaml
# Organization repo
resources:
  repositories:
  - repository: ProductionCode
    type: github
    name: contoso/production-app

# Personal repo
resources:
  repositories:
  - repository: MyLib
    type: github
    name: johnsmith/utility-library
```

### Multiple Repository Checkout

**Scenario**: Monorepo build requires shared library

```yaml
resources:
  repositories:
  - repository: SharedLibrary
    type: github
    endpoint: GitHubConnection
    name: myorg/shared-lib
    ref: refs/heads/main

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- checkout: self  # Main repo
- checkout: SharedLibrary  # Shared library
- script: |
    # Both repos now available
    ls -la $(Build.SourcesDirectory)
    # Contains: main repo + SharedLibrary/
```

## Managing GitHub Users in Azure DevOps

### Connecting Identities

**Purpose**: Link GitHub identity with Azure DevOps account

```
Azure DevOps → User Settings → Profile → Security
→ Connect with GitHub
```

**Benefits**:
- Single sign-on
- Unified permissions
- Automatic repo access inheritance

### Organization-Level GitHub Connection

```
Azure DevOps → Organization Settings → GitHub connections
→ Connect GitHub organization
```

**Enables**:
- All users can access GitHub repos
- Centralized authentication
- Easier team management

## Triggering Builds from GitHub

### CI Triggers

```yaml
# Trigger on push to main or develop
trigger:
  branches:
    include:
    - main
    - develop
  paths:
    include:
    - src/*
    exclude:
    - docs/*
```

### PR Triggers

```yaml
# Validate pull requests
pr:
  branches:
    include:
    - main
  paths:
    exclude:
    - '*.md'
```

**Result**: Azure Pipeline runs on every PR, status reported back to GitHub

### Webhook Configuration

**Automatic Setup**: Azure Pipelines creates webhooks when pipeline is created

**Manual Verification**:
```
GitHub Repo → Settings → Webhooks
→ Should see Azure Pipelines webhook
→ Payload URL: https://dev.azure.com/...
→ Events: Pushes, Pull requests
```

## GitHub Status Checks

**Azure Pipelines reports build status to GitHub**:

```yaml
# Pipeline runs on PR
# Result posted to GitHub as status check
```

**GitHub PR displays**:
- ✅ Build succeeded
- ❌ Build failed
- ⏳ Build in progress

**Enforce in Branch Protection**:
```
GitHub Repo → Settings → Branches → Branch protection rules
→ Require status checks to pass before merging
→ Select Azure Pipelines build
```

## Example: Complete Integration

```yaml
# azure-pipelines.yml in GitHub repo

trigger:
  branches:
    include:
    - main
    - release/*

pr:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  displayName: 'Use .NET 8 SDK'
  inputs:
    version: '8.x'

- script: dotnet restore
  displayName: 'Restore dependencies'

- script: dotnet build --configuration $(buildConfiguration)
  displayName: 'Build project'

- script: dotnet test --no-build --configuration $(buildConfiguration)
  displayName: 'Run tests'

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'drop'
```

**Workflow**:
1. Developer pushes to feature branch
2. Creates PR to main
3. Azure Pipeline triggered automatically
4. Build status reported to GitHub PR
5. Merge blocked until build succeeds

## Troubleshooting

### Issue: "Could not access GitHub repository"

**Solutions**:
1. Check GitHub App installation (Settings → Installed GitHub Apps)
2. Verify PAT hasn't expired
3. Confirm repo permissions (OAuth/PAT has repo access)
4. Re-authorize Azure Pipelines app

### Issue: "Pipeline not triggering on push"

**Check**:
1. Webhook exists in GitHub repo settings
2. Trigger branches match push branch
3. Path filters not excluding all changes
4. Webhook deliveries show success (GitHub Settings → Webhooks → Recent Deliveries)

### Issue: "Permission denied during checkout"

**Solution**: Service connection credentials insufficient

```yaml
# Explicitly specify connection
resources:
  repositories:
  - repository: MyRepo
    type: github
    endpoint: GitHubServiceConnection  # Define this
    name: org/repo
```

## Best Practices

1. **Use GitHub App**: Most secure, easiest to manage
2. **Limit PAT Scopes**: Only grant necessary permissions if using PAT
3. **Branch Protection**: Require pipeline success before merge
4. **Path Filters**: Optimize triggers to relevant code changes
5. **Multiple Repos**: Use explicit checkout when referencing external repos

## Critical Notes

- 🎯 **GitHub App = recommended auth** - Fine-grained permissions, auto-refresh tokens, explicit repo selection; better than OAuth/PAT
- 💡 **Repository format = org/repo** - Always use `organization/repository` or `user/repository` format in YAML; case-sensitive
- ⚠️ **PAT expiration risk** - Personal Access Tokens expire; builds fail when expired; use GitHub App to avoid manual renewal
- 📊 **Status checks enable gating** - Azure Pipelines reports to GitHub PRs; enable "Require status checks" in branch protection for quality gates
- 🔄 **Webhooks auto-created** - Azure Pipelines installs webhook automatically; verify in GitHub repo Settings → Webhooks if triggers fail
- ✨ **Multiple repo checkout supported** - Use `resources.repositories` + `checkout` for monorepo scenarios; both repos available in workspace

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-pipeline-strategy/4-integrate-github)
