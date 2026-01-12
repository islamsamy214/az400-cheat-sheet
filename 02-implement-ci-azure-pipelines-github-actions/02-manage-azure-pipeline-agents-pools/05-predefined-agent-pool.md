# Explore Predefined Agent Pool

## Key Concepts
- **Azure Pipelines pool** - Predefined pool with Microsoft-hosted agents
- **No configuration needed** - Ready to use out of the box
- **Multiple VM images** - Windows, Ubuntu, macOS options
- **Default permissions** - All contributors can use by default
- **Latest software** - Images updated regularly

## Predefined "Azure Pipelines" Pool

Azure Pipelines provides a **predefined agent pool named "Azure Pipelines"** with Microsoft-hosted agents. This pool offers an easy way to run jobs without configuring build infrastructure.

```yaml
# Use predefined Azure Pipelines pool
pool:
  vmImage: 'ubuntu-latest'  # Choose VM image

steps:
- script: echo "Running on Azure Pipelines"
```

## Available Virtual Machine Images

| Image | Description | Use For |
|-------|-------------|---------|
| **Windows Server 2022 with Visual Studio 2022** | Latest Windows, VS 2022 | .NET 6/7, modern Windows apps |
| **Windows Server 2019 with Visual Studio 2019** | Windows 2019, VS 2019 | .NET Framework, legacy Windows apps |
| **Ubuntu 20.04** | Ubuntu 20.04 LTS | Linux builds, containers |
| **Ubuntu 18.04** | Ubuntu 18.04 LTS | Older Linux applications |
| **macOS 11 Big Sur** | macOS Big Sur | iOS, macOS apps (newer) |
| **macOS X Catalina 10.15** | macOS Catalina | iOS, macOS apps (older) |

### Image Selection Examples
```yaml
# Windows with Visual Studio 2022
pool:
  vmImage: 'windows-2022'

# Ubuntu latest
pool:
  vmImage: 'ubuntu-latest'

# macOS latest
pool:
  vmImage: 'macOS-latest'

# Specific versions
pool:
  vmImage: 'ubuntu-20.04'
```

## Pre-Installed Software

Each image comes with **extensive pre-installed software**:

### Windows Images Include
- Visual Studio (Enterprise edition)
- .NET Framework and .NET Core SDKs
- PowerShell and Azure PowerShell
- Git, Node.js, Python, Java
- Docker, Kubernetes tools
- Azure CLI, Visual Studio Code

### Ubuntu Images Include
- Build essentials (gcc, make, etc.)
- .NET Core SDK
- Docker and Docker Compose
- Node.js, Python, Ruby, Java
- Azure CLI, kubectl, Helm
- Git, Subversion

### macOS Images Include
- Xcode (multiple versions)
- Swift, Objective-C compilers
- Node.js, Python, Ruby
- Homebrew package manager
- Git, Carthage, CocoaPods
- Azure CLI, Docker

## Default Permissions

### All Contributors Have Access
By default, **all contributors in a project are members of the User role** on each hosted pool.

```
Default Permissions:
└── User Role (All Contributors)
    ├── Can create pipelines
    ├── Can run build pipelines
    ├── Can run release pipelines
    └── Uses Microsoft-hosted pools
```

**What This Means**:
- ✅ Every contributor can create pipelines using Microsoft-hosted pools
- ✅ Every contributor can run build and release pipelines
- ✅ No additional configuration needed for basic access
- ✅ Simplifies getting started with Azure Pipelines

### Permission Roles
| Role | Permissions |
|------|-------------|
| **Reader** | View pool information only |
| **User** | Use pool in pipelines (default for contributors) |
| **Administrator** | Manage pool settings and permissions |

## Usage in Pipelines

### Basic Usage
```yaml
# Simplest form - use latest image
pool:
  vmImage: 'ubuntu-latest'

steps:
- script: npm install
- script: npm test
```

### Multi-Stage with Different Images
```yaml
stages:
- stage: Build
  jobs:
  - job: BuildOnLinux
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: ./build.sh
  
  - job: BuildOnWindows
    pool:
      vmImage: 'windows-latest'
    steps:
    - script: build.bat

- stage: Test
  jobs:
  - job: TestOnMac
    pool:
      vmImage: 'macOS-latest'
    steps:
    - script: ./run-tests.sh
```

### Specifying Image Capabilities
```yaml
# If you need specific tools
pool:
  vmImage: 'windows-2022'
  demands:
  - VisualStudio_17.0  # Requires VS 2022
  - msbuild
  - azureps
```

## Image Update Frequency

Microsoft updates these images regularly:
- **Security patches** - Applied as soon as available
- **Tool updates** - Monthly releases with latest versions
- **New versions** - Added for major platform releases

### Checking Installed Software
```bash
# View complete list of installed software
# Documentation available at:
# https://github.com/actions/virtual-environments

# Check versions in pipeline
- script: |
    node --version
    npm --version
    python --version
    dotnet --version
  displayName: 'Check tool versions'
```

## Benefits of Predefined Pool

| Benefit | Description |
|---------|-------------|
| **Zero setup** | No installation or configuration required |
| **Always updated** | Latest tools and security patches |
| **Clean environment** | Fresh VM for every run |
| **Multiple platforms** | Windows, Linux, macOS support |
| **Free for public projects** | Unlimited minutes for open source |
| **Generous free tier** | 1,800 minutes/month for private projects |

## Limitations

| Limitation | Impact | Solution |
|------------|--------|----------|
| **Job time limit** | 60 minutes for free tier, 6 hours for paid | Use self-hosted agents for longer jobs |
| **No customization** | Can't install permanent software | Use container jobs or self-hosted agents |
| **No state preservation** | Fresh VM each run | Use artifacts or external storage |
| **Network access** | Standard internet connectivity only | Use self-hosted agents for VPN/private access |

## When to Use Predefined Pool

✅ **Best For**:
- Standard builds (Node.js, .NET, Python, Java)
- Quick start without infrastructure
- Pull request validation
- Open source projects
- Teams without dedicated infrastructure

❌ **Not Ideal For**:
- Custom tool requirements
- Long-running jobs (>6 hours)
- On-premises deployments
- Specialized hardware needs
- High build volume (cost optimization)

## Cost Considerations

### Free Tier
- **Public projects** - Unlimited minutes
- **Private projects** - 1,800 minutes/month (1 concurrent job)

### Paid Tier
- **Per parallel job** - $40/month
- **Includes** - Unlimited minutes (6-hour job limit)
- **Additional jobs** - Scale by adding more parallel jobs

```yaml
# Cost optimization tip: Use shorter jobs
# Split long-running jobs into smaller parallel jobs
jobs:
- job: Test_Unit
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - script: npm run test:unit

- job: Test_Integration
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - script: npm run test:integration
```

## Critical Notes
- 🎯 "Azure Pipelines" is predefined pool name (no configuration needed)
- 💡 See official docs for most up-to-date list of images and software
- ⚠️ All contributors can use pool by default (User role)
- 📊 Images updated regularly with latest tools and security patches
- 🔄 Fresh VM for each run means no state preserved between builds
- ✨ Choose vmImage based on platform needs (windows/ubuntu/macOS)

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-azure-pipeline-agents-pools/5-explore-predefined-agent-pool)
