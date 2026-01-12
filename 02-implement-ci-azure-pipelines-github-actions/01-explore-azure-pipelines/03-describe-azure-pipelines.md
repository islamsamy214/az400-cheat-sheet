# Describe Azure Pipelines

## Key Concepts
- **Cloud service** - Automatically builds, tests, and deploys code
- **24/7 automation** - Personal DevOps assistant working continuously
- **Technology agnostic** - Works with any programming language or framework
- **Universal integration** - Connects with GitHub, Azure Repos, and most Git providers
- **Combines CI/CD** - Constantly tests, builds, and ships code reliably

## What Azure Pipelines Supports

### Programming Languages
| Category | Languages |
|----------|-----------|
| **Web development** | JavaScript, TypeScript, HTML, CSS |
| **Backend development** | Python, Java, C#, PHP, Ruby, Go |
| **Mobile development** | Swift, Kotlin, Xamarin |
| **Enterprise languages** | C++, .NET, PowerShell |

**Key Point**: If you can build it on a computer, Azure Pipelines can probably automate it

### Version Control Systems
```yaml
# Example: GitHub integration
trigger:
  branches:
    include:
      - main
      - develop

resources:
  repositories:
  - repository: self
    type: github
    endpoint: GitHubConnection
```

**Supported Systems**:
- **GitHub** - Most popular for open source and enterprise
- **Azure Repos** - Microsoft's Git service (part of Azure DevOps)
- **Bitbucket** - Atlassian's Git service

### Application Types
- Web applications (React, Angular, Vue, ASP.NET, Django, Rails)
- Mobile apps (iOS, Android, cross-platform)
- Desktop applications (Windows, macOS, Linux)
- APIs and microservices (REST APIs, GraphQL, serverless functions)
- Data applications (ML models, data pipelines)

### Deployment Destinations

**Cloud Platforms**:
| Platform | Services |
|----------|----------|
| **Microsoft Azure** | Full integration with all Azure services |
| **AWS** | EC2, Lambda, ECS, and more |
| **Google Cloud** | GKE, App Engine, Cloud Functions |

**Infrastructure Options**:
- Container registries (Azure Container Registry, Docker Hub, AWS ECR)
- Virtual machines (Windows and Linux VMs anywhere)
- Kubernetes clusters (any Kubernetes environment)
- On-premises servers (your own data center)

### Package Management
```bash
# Example: Publish to multiple registries
- task: NuGetCommand@2  # .NET packages
- task: Npm@1           # JavaScript packages
- task: Maven@3         # Java packages
- task: TwineAuthenticate@1  # Python packages
```

**Supported Registries**:
- NuGet (.NET packages)
- npm (JavaScript packages)
- Maven (Java packages)
- PyPI (Python packages)
- Docker registries (container images)
- Azure Artifacts (built-in package management)

## Benefits of CI/CD

### Continuous Integration (CI) Benefits
| Benefit | Description |
|---------|-------------|
| **Catch bugs early** | Problems found while code fresh in developers' minds |
| **Increase code coverage** | Automated tests run consistently, improving quality |
| **Build faster** | Split testing across multiple machines to speed up process |
| **Prevent broken releases** | Never ship code that doesn't build or pass tests |
| **Enable continuous testing** | Tests run automatically with every change |

### Continuous Delivery (CD) Benefits
| Benefit | Description |
|---------|-------------|
| **Deploy automatically** | Get new features to users faster with less manual work |
| **Ensure consistency** | Every environment gets exactly same tested code |
| **Reduce deployment risk** | Thoroughly tested code = more reliable deployments |
| **Enable rapid feedback** | Users can try new features quickly and provide feedback |

## Why Choose Azure Pipelines?

### Flexibility and Compatibility
- Work with any language or platform - No technology lock-in
- Deploy to multiple targets simultaneously - Cloud, on-premises, or hybrid
- Choose your development tools - Use whatever works best for team

### Seamless Azure Integration
```yaml
# Example: Easy Azure deployment
- task: AzureWebApp@1
  displayName: 'Deploy to Azure App Service'
  inputs:
    azureSubscription: 'MyAzureSubscription'
    appName: 'my-web-app'
    package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

- Native Azure support - Deploy to Azure services with minimal configuration
- Azure resource management - Automatically provision and manage resources
- Azure security integration - Leverage Azure AD and other security services

### Platform Choice
| Agent Type | Description | Best For |
|------------|-------------|----------|
| **Microsoft-hosted** | Microsoft provides and maintains | Quick start, minimal setup |
| **Self-hosted** | You provide and maintain | Custom tools, specific requirements |
| **Container builds** | Run builds in Docker containers | Consistency across environments |

### Collaboration Features
- **GitHub integration** - Built-in support for GitHub repositories and workflows
- **Open source friendly** - Free builds for public repositories
- **Team collaboration** - Built-in code reviews, work item tracking, reporting

### Enterprise Ready
- **Scalable** - Handles projects of any size automatically
- **Secure** - Enterprise-grade security and compliance features
- **Reliable** - Backed by Microsoft's global cloud infrastructure

## Getting Started Requirements
```yaml
# Minimal pipeline example
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'
```

**What You Need**:
1. Source code in Git - Project in any supported Git repository
2. Azure DevOps account - Free account includes generous build minutes
3. Simple pipeline definition - YAML file describing build and deployment process

## Critical Notes
- 🎯 Fully automated once set up - no manual intervention needed
- 💡 Technology agnostic - if you can build it, Azure Pipelines can automate it
- ⚠️ Free builds for public repositories (open source friendly)
- 📊 Cloud-based means no servers to maintain and auto-scaling
- 🔄 YAML pipeline definitions are version-controlled with your code
- ✨ Native Azure integration is a major differentiator for Azure-centric projects

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-pipelines/3-describe-azure-pipelines)
