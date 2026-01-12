# Explore Source Control Integration

Azure Pipelines supports multiple source control systems, each with unique integration patterns. Understanding the differences helps you choose the right approach for your workflow.

## Supported Source Control Systems

### Tier 1: Full Azure DevOps Integration

**Azure Repos Git**:
- ✅ Native integration (same platform)
- ✅ Built-in authentication
- ✅ Branch policies
- ✅ Pull request builds
- ✅ Work item linking
- ✅ No external service connection needed

**GitHub**:
- ✅ GitHub App authentication (recommended)
- ✅ OAuth fallback
- ✅ PR validation builds
- ✅ Status checks
- ✅ GitHub Actions alternative

**GitHub Enterprise Server**:
- ✅ Self-hosted GitHub
- ✅ Same features as GitHub
- ✅ Requires service connection
- ✅ On-premises compliance

### Tier 2: Supported with Service Connections

**Bitbucket Cloud**:
- ✅ OAuth authentication
- ✅ PR triggers
- ✅ Webhook integration
- ⚠️ Limited compared to GitHub/Azure Repos

**Bitbucket Server**:
- ✅ Self-hosted Bitbucket
- ✅ Service connection required
- ⚠️ Manual webhook setup may be needed

### Other Git Providers

**Generic Git** (GitLab, Gitea, etc.):
- ✅ Clone via HTTPS/SSH
- ⚠️ Manual triggers (no webhook automation)
- ⚠️ No PR validation
- ⚠️ Manual authentication setup

**TFVC (Team Foundation Version Control)**:
- ✅ Azure DevOps legacy system
- ⚠️ Centralized, not distributed
- ⚠️ Limited modern CI/CD features

## Azure Repos Git Integration

### Advantages

**Seamless Integration**:
- No authentication setup required
- Automatic webhook creation
- Built-in work item tracking
- Branch policies enforce PR requirements

**Pipeline YAML**:
```yaml
# Automatically uses Azure Repos from same project
trigger:
  branches:
    include:
    - main
    - develop
  paths:
    exclude:
    - docs/*

pr:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- checkout: self
- script: npm install
- script: npm test
```

**No explicit repository reference needed** - Azure Pipelines assumes current project's repo

### Branch Policies

**Require Build Validation**:
```
Project Settings → Repositories → Policies → Branch Policies (main)
→ Build validation
→ Add build policy
→ Select pipeline
→ Trigger: Automatic
→ Policy requirement: Required
```

**Result**: PRs cannot merge until pipeline succeeds

## GitHub Integration

### Setup Options

#### Option 1: GitHub App (Recommended)

**Installation**:
```
1. Azure DevOps → New Pipeline → GitHub
2. Authorize Azure Pipelines GitHub App
3. Select repositories
4. App installed to GitHub org
```

**Benefits**:
- Fine-grained permissions (per-repo)
- Automatic token refresh
- Better security
- Status checks to PRs

**Pipeline YAML**:
```yaml
# azure-pipelines.yml in GitHub repo

trigger:
- main

pr:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- checkout: self
- script: npm ci
- script: npm test
```

#### Option 2: Service Connection (PAT/OAuth)

**When to Use**:
- Multiple repos from same connection
- Shared service account
- Specific authentication requirements

**Setup**:
```
Azure DevOps → Project Settings → Service Connections
→ New service connection → GitHub
→ Personal Access Token or OAuth
→ Name: GitHubConnection
```

**Pipeline YAML**:
```yaml
resources:
  repositories:
  - repository: MyGitHubRepo
    type: github
    endpoint: GitHubConnection
    name: myorg/myrepo

trigger:
  branches:
    include:
    - main

steps:
- checkout: MyGitHubRepo
- script: npm install
```

### GitHub Status Checks

**Automatic Reporting**:
```
GitHub PR → Checks tab
→ Shows Azure Pipeline build status
→ ✅ All checks have passed
→ ❌ Some checks were not successful
```

**Branch Protection**:
```
GitHub → Settings → Branches → Branch protection rules
→ Require status checks to pass before merging
→ Select: Azure Pipelines - <pipeline-name>
```

## Bitbucket Integration

### Bitbucket Cloud

**Service Connection Setup**:
```
Azure DevOps → Project Settings → Service Connections
→ New service connection → Bitbucket Cloud
→ OAuth or Username/Password
→ Authorize Azure Pipelines
```

**Pipeline YAML**:
```yaml
resources:
  repositories:
  - repository: BitbucketRepo
    type: bitbucket
    endpoint: BitbucketConnection
    name: myworkspace/myrepo

trigger:
  branches:
    include:
    - main

steps:
- checkout: BitbucketRepo
- script: mvn clean install
```

### Bitbucket Server (Self-Hosted)

**Connection Requirements**:
- Network access to Bitbucket Server
- Personal Access Token or SSH key
- Manual webhook configuration (often)

**Service Connection**:
```
Type: Bitbucket Server
Server URL: https://bitbucket.company.com
Username: <service-account>
Password/PAT: <token>
```

## Comparison Matrix

| Feature | Azure Repos | GitHub | GitHub Enterprise | Bitbucket Cloud | Bitbucket Server |
|---------|-------------|--------|-------------------|-----------------|------------------|
| **Authentication** | Automatic | GitHub App/OAuth | GitHub App | OAuth | PAT/SSH |
| **PR Builds** | ✅ Native | ✅ Native | ✅ Native | ✅ Supported | ⚠️ Manual |
| **Status Checks** | ✅ Branch Policies | ✅ Status API | ✅ Status API | ✅ Build Status | ⚠️ Limited |
| **Work Item Linking** | ✅ Built-in | ⚠️ Via #123 | ⚠️ Via #123 | ❌ | ❌ |
| **Webhook Setup** | ✅ Automatic | ✅ Automatic | ✅ Automatic | ✅ Automatic | ⚠️ Manual |
| **YAML Templates** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Multi-Repo** | ✅ | ✅ | ✅ | ✅ | ✅ |

## Multi-Repository Pipelines

### Checkout Multiple Repos

```yaml
resources:
  repositories:
  - repository: WebApp
    type: git
    name: MyProject/WebApp
  
  - repository: SharedLibrary
    type: github
    endpoint: GitHubConnection
    name: myorg/shared-lib

trigger:
  branches:
    include:
    - main

steps:
- checkout: self  # Main repo (where YAML is)
- checkout: WebApp  # Azure Repos repo
- checkout: SharedLibrary  # GitHub repo

- script: |
    # All repos now available
    ls -la $(Build.SourcesDirectory)
    # WebApp/
    # SharedLibrary/
    # (current repo)/
```

### Trigger from Multiple Repos

```yaml
resources:
  repositories:
  - repository: BackendAPI
    type: git
    name: MyProject/BackendAPI
    trigger:
      branches:
        include:
        - main

# Pipeline triggers when BackendAPI changes
steps:
- checkout: BackendAPI
- script: dotnet build
```

## YAML Benefits Across All Platforms

### Version-Controlled Pipelines

**Advantages**:
1. **Code Review**: Pipeline changes reviewed in PRs
2. **History**: Track pipeline evolution with git log
3. **Branching**: Different pipeline configs per branch
4. **Rollback**: Revert pipeline changes easily

**Example - Feature Branch Pipeline**:
```yaml
# feature-branch: azure-pipelines.yml
trigger:
- feature/*

pool:
  vmImage: 'ubuntu-latest'

steps:
- script: npm install
- script: npm test  # No deployment for feature branches

---

# main branch: azure-pipelines.yml
trigger:
- main

stages:
- stage: Build
  jobs:
  - job: BuildAndTest
    steps:
    - script: npm install
    - script: npm test

- stage: Deploy
  jobs:
  - deployment: DeployProd
    environment: 'Production'
    steps:
    - script: kubectl apply -f k8s/
```

### Template Reusability

**Shared Template** (in any repo):
```yaml
# templates/dotnet-build.yml
parameters:
- name: buildConfiguration
  default: 'Release'

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'restore'

- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    arguments: '--configuration ${{ parameters.buildConfiguration }}'

- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    arguments: '--configuration ${{ parameters.buildConfiguration }}'
```

**Usage in Pipeline**:
```yaml
# azure-pipelines.yml
trigger:
- main

steps:
- template: templates/dotnet-build.yml
  parameters:
    buildConfiguration: 'Release'
```

## Best Practices

### Source Control Selection

| Scenario | Recommended | Reason |
|----------|-------------|--------|
| **All-in Azure DevOps** | Azure Repos | Seamless integration, no auth setup |
| **Open Source Project** | GitHub | Community, visibility, Actions |
| **Enterprise with GitHub** | GitHub Enterprise | Security, compliance, on-prem |
| **Existing Bitbucket Users** | Bitbucket | Maintain current workflow |
| **Multi-Platform Team** | GitHub | Broadest integration support |

### Authentication

1. **Azure Repos**: Use built-in auth (no setup needed)
2. **GitHub**: Use GitHub App (better than OAuth/PAT)
3. **Bitbucket**: Use OAuth (avoid username/password)
4. **Self-Hosted**: Use service accounts with PATs

### Branch Protection

**All Platforms**:
- Require PR before merge
- Require build success
- Require code review (1-2 reviewers)
- Enforce for administrators (no bypass)

### YAML Organization

```
.azure-pipelines/
├── azure-pipelines.yml          # Main pipeline
├── templates/
│   ├── build-template.yml
│   ├── test-template.yml
│   └── deploy-template.yml
└── variables/
    ├── dev.yml
    └── prod.yml
```

## Troubleshooting

### Issue: "Repository not found"

**Solutions**:
1. Verify service connection has access
2. Check repository name format: `org/repo` (GitHub), `project/repo` (Azure Repos)
3. Confirm PAT/OAuth hasn't expired

### Issue: "PR build not triggering"

**Check**:
1. Webhook exists (repo settings)
2. PR trigger defined in YAML
3. Branch name matches trigger pattern
4. Service connection authorized

### Issue: "Authentication failed"

**Solutions**:
1. Regenerate PAT/token
2. Re-authorize OAuth
3. Check service connection status
4. Verify network access (self-hosted agents)

## Critical Notes

- 🎯 **Azure Repos = zero-config** - Same platform as pipelines; automatic auth, webhooks, branch policies; no service connection needed
- 💡 **GitHub App > OAuth/PAT** - Fine-grained permissions, auto-refresh, better security; recommended for GitHub integration
- ⚠️ **YAML = version-controlled** - Pipeline changes reviewed in PRs, tracked in git; branching enables different configs per environment
- 📊 **Multi-repo checkout supported** - Single pipeline can checkout from Azure Repos, GitHub, Bitbucket simultaneously; shared libraries, monorepo builds
- 🔄 **Branch protection for quality** - Require pipeline success before merge on all platforms; prevents breaking main branch
- ✨ **Templates enable reuse** - Share build/test/deploy logic across pipelines; store in separate repo, reference via resources.repositories

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-pipeline-strategy/8-explore-source-control-types)
