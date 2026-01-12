# Explore Runners

Runners are the compute infrastructure where your GitHub Actions workflows execute. Understanding runner types and their characteristics is crucial for optimizing workflow performance, cost, and security.

## Runner Types Overview

| Runner Type | Hosted By | Cost | Setup | Use Case |
|-------------|-----------|------|-------|----------|
| **GitHub-hosted** | GitHub | Included minutes* | Zero setup | Most workflows |
| **Self-hosted** | You | Infrastructure cost | Manual setup | Special requirements |

*Free tier: 2,000 minutes/month (Linux), varies by plan

## GitHub-Hosted Runners

GitHub provides virtual machines with pre-installed software for common development tasks.

### Available Runners

```yaml
jobs:
  build-linux:
    runs-on: ubuntu-latest  # Most common, fastest
  
  build-windows:
    runs-on: windows-latest
  
  build-macos:
    runs-on: macos-latest  # Most expensive minutes
  
  build-macos-arm:
    runs-on: macos-14  # Apple Silicon M1
```

### Runner Specifications

| Runner | Virtual Environment | Hardware | Minute Multiplier |
|--------|---------------------|----------|-------------------|
| **ubuntu-latest** | Ubuntu 22.04 | 4-core CPU, 16 GB RAM, 14 GB SSD | **1x** |
| **ubuntu-22.04** | Ubuntu 22.04 | 4-core CPU, 16 GB RAM, 14 GB SSD | 1x |
| **ubuntu-20.04** | Ubuntu 20.04 | 4-core CPU, 16 GB RAM, 14 GB SSD | 1x |
| **windows-latest** | Windows Server 2022 | 4-core CPU, 16 GB RAM, 14 GB SSD | **2x** |
| **windows-2022** | Windows Server 2022 | 4-core CPU, 16 GB RAM, 14 GB SSD | 2x |
| **windows-2019** | Windows Server 2019 | 4-core CPU, 16 GB RAM, 14 GB SSD | 2x |
| **macos-latest** | macOS 14 (Sonoma) | 4-core CPU, 14 GB RAM, 14 GB SSD | **10x** |
| **macos-14** | macOS 14 (Sonoma) - M1 | 3-core CPU, 7 GB RAM, 14 GB SSD | 10x |
| **macos-13** | macOS 13 (Ventura) | 4-core CPU, 14 GB RAM, 14 GB SSD | 10x |
| **macos-12** | macOS 12 (Monterey) | 4-core CPU, 14 GB RAM, 14 GB SSD | 10x |

**Minute Multiplier**: A 10-minute job on `macos-latest` consumes 100 minutes from your quota.

### Pre-Installed Software

GitHub-hosted runners come with extensive tooling pre-installed:

**Linux (Ubuntu)**:
- Languages: Node.js, Python, Ruby, Go, PHP, Java, .NET
- Tools: Docker, Kubernetes tools, Azure CLI, AWS CLI, Git, Homebrew
- Databases: PostgreSQL, MySQL, SQLite
- Build tools: CMake, Maven, Gradle, npm, yarn

**Windows**:
- Languages: Node.js, Python, Ruby, Go, PHP, Java, .NET Framework/.NET Core
- Tools: Visual Studio Build Tools, Docker, Git, Chocolatey, Azure CLI, AWS CLI
- Databases: PostgreSQL, MySQL, SQLite
- Build tools: MSBuild, Maven, Gradle, npm, yarn

**macOS**:
- Languages: Node.js, Python, Ruby, Go, PHP, Java, .NET
- Tools: Xcode, Docker, Git, Homebrew, Azure CLI, AWS CLI, CocoaPods
- Build tools: CMake, Maven, Gradle, npm, yarn
- iOS/macOS: Xcode Command Line Tools, iOS/macOS simulators

**Software Versions**: [View complete list](https://github.com/actions/runner-images/tree/main/images)

### Usage Limits

| Account Type | Concurrent Jobs | Storage | Included Minutes |
|--------------|-----------------|---------|------------------|
| **Free** | 20 | 500 MB | 2,000/month (Linux) |
| **Pro** | 40 | 1 GB | 3,000/month |
| **Team** | 60 | 2 GB | 10,000/month |
| **Enterprise** | 180 | 50 GB | 50,000/month |

**Important**:
- ⚠️ Minutes reset monthly
- 💰 Additional minutes available for purchase
- 🕐 Windows = 2x minutes, macOS = 10x minutes

### When to Use GitHub-Hosted Runners

✅ **Perfect For**:
- Standard CI/CD workflows
- Open-source projects (free for public repos)
- Testing across multiple OS platforms
- Short to medium-duration jobs
- Teams without infrastructure resources

❌ **Not Ideal For**:
- Jobs requiring > 6 hours (360-minute timeout)
- Jobs needing specialized hardware (GPUs, specific CPUs)
- Jobs requiring access to internal networks
- High-frequency workflows (cost concerns)
- Jobs requiring persistent state between runs

## Self-Hosted Runners

Self-hosted runners give you complete control over the execution environment.

### Deployment Options

```yaml
jobs:
  deploy:
    runs-on: self-hosted  # Any self-hosted runner
  
  deploy-specific:
    runs-on: [self-hosted, linux, x64, gpu]  # Specific labels
  
  deploy-windows:
    runs-on: [self-hosted, windows, production]
```

### Setting Up Self-Hosted Runners

#### Repository Level

1. Go to Repository → Settings → Actions → Runners
2. Click "New self-hosted runner"
3. Follow OS-specific download/configuration instructions

```bash
# Example: Linux setup
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L   https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# Configure runner (interactive)
./config.sh --url https://github.com/ORG/REPO --token TOKEN

# Run as service
sudo ./svc.sh install
sudo ./svc.sh start
```

#### Organization Level

More efficient for multiple repositories:

1. Go to Organization → Settings → Actions → Runners
2. Click "New runner"
3. Configure once, use across all repos

#### Enterprise Level

Centralized runner management for all organizations.

### Custom Labels

Add labels to target specific runners:

```bash
# During configuration
./config.sh --url ... --token ... --labels gpu,ubuntu-22.04,production

# Or after configuration (edit .runner file)
```

Using labels in workflows:

```yaml
jobs:
  train-model:
    runs-on: [self-hosted, linux, gpu, cuda-12]
    steps:
    - run: python train_model.py
  
  deploy-prod:
    runs-on: [self-hosted, production, kubernetes]
    steps:
    - run: kubectl apply -f deployment.yaml
```

### Advantages of Self-Hosted Runners

✅ **Cost**: No minute charges (only infrastructure)

✅ **Performance**: Choose exact hardware (GPUs, high-memory, fast storage)

✅ **Network Access**: Connect to internal resources (databases, APIs, VPNs)

✅ **Persistent Cache**: Maintain caches between runs for faster builds

✅ **Custom Software**: Pre-install proprietary tools, licenses, configurations

✅ **Compliance**: Keep code and data within your infrastructure

✅ **Long Jobs**: No 6-hour timeout limit

### Disadvantages of Self-Hosted Runners

❌ **Management**: You maintain OS, software, security patches

❌ **Scaling**: Manual capacity planning and provisioning

❌ **Security**: Higher risk with public repositories

❌ **Availability**: You ensure uptime and monitoring

❌ **Cost**: Infrastructure costs even when idle

### When to Use Self-Hosted Runners

✅ **Use Self-Hosted When**:
- Need specialized hardware (GPUs, TPUs, high-memory machines)
- Require access to internal networks/resources
- Long-running jobs (> 6 hours)
- High workflow volume (cost savings)
- Need persistent cache/state
- Compliance requirements (data residency)
- Custom software/licenses required

❌ **Avoid Self-Hosted When**:
- Running workflows from public repositories
- No infrastructure team to manage runners
- Low workflow volume (GitHub-hosted more cost-effective)
- Standard tooling sufficient

## Security Considerations

### ⚠️ Critical: Public Repositories

**NEVER use self-hosted runners with public repositories.**

Why? Anyone can fork your repo and submit a malicious PR that:
- Mines cryptocurrency on your infrastructure
- Steals environment secrets
- Installs backdoors
- Attacks your network

**Public repos → MUST use GitHub-hosted runners only**

### Self-Hosted Security Best Practices

```yaml
# ✅ Good: Restrict to private repos
# Repository Settings → Actions → General → Runner groups
# Set to "Selected repositories" only

# ✅ Good: Limit permissions
permissions:
  contents: read

# ✅ Good: Review PR workflows
# Require approval for first-time contributors
```

**Security Checklist**:
- ☑️ Use private repositories only
- ☑️ Keep runner software updated
- ☑️ Isolate runners (containers, VMs)
- ☑️ Use ephemeral runners (auto-scale, destroy after each job)
- ☑️ Implement network segmentation
- ☑️ Monitor runner activity
- ☑️ Rotate runner tokens regularly
- ☑️ Apply least-privilege access

### Ephemeral Self-Hosted Runners

For maximum security, use runners that are created and destroyed for each job:

```bash
# Run once and exit
./run.sh --once

# Or with auto-scaling solutions:
# - Azure Container Instances
# - AWS ECS/Fargate
# - Kubernetes with actions-runner-controller
```

## Runner Maintenance

### Updating Runners

```bash
# Stop runner
sudo ./svc.sh stop

# Update runner software
./config.sh --upgrade

# Start runner
sudo ./svc.sh start
```

### Monitoring Runners

```bash
# Check runner status
sudo ./svc.sh status

# View runner logs
journalctl -u actions.runner.*
```

### Removing Runners

```bash
# Stop and uninstall service
sudo ./svc.sh stop
sudo ./svc.sh uninstall

# Remove runner from GitHub
./config.sh remove --token TOKEN
```

## Cost Comparison Example

**Scenario**: 1,000 minutes of workflow execution per month

| Runner Type | Cost Calculation | Monthly Cost |
|-------------|------------------|--------------|
| **GitHub-hosted (Linux)** | 1,000 min included (Free tier) | $0 |
| **GitHub-hosted (Windows)** | 2,000 min consumed (2x multiplier) | $0 (within free tier) |
| **GitHub-hosted (macOS)** | 10,000 min consumed (10x multiplier) | ~$400* (8,000 overage × $0.05) |
| **Self-hosted (t3.medium)** | AWS EC2: $30/month | $30 |
| **Self-hosted (dedicated)** | Physical server: $100-500/month | $100-500 |

*Assuming $0.05/minute overage

**Break-Even Analysis**:
- Low volume (< 2,000 min/month): GitHub-hosted
- Medium volume (2,000-10,000 min/month): GitHub-hosted or small self-hosted
- High volume (> 10,000 min/month): Self-hosted often cheaper
- Specialized hardware: Self-hosted required

## Best Practices

### 1. Start with GitHub-Hosted

Default to GitHub-hosted runners unless you have specific requirements:

```yaml
# ✅ Good: Use GitHub-hosted for standard workflows
jobs:
  build:
    runs-on: ubuntu-latest  # Fast, free, zero maintenance
```

### 2. Use Specific Runner Versions

```yaml
# ❌ Bad: "latest" changes over time
runs-on: ubuntu-latest

# ✅ Good: Pin to specific version for stability
runs-on: ubuntu-22.04
```

### 3. Choose Appropriate OS

```yaml
# Linux for most tasks (fastest, cheapest)
build:
  runs-on: ubuntu-latest

# macOS only when necessary (iOS, macOS builds)
build-ios:
  runs-on: macos-latest

# Windows only when required (.NET Framework, Windows APIs)
build-windows:
  runs-on: windows-latest
```

### 4. Optimize Self-Hosted Runner Labels

```yaml
# ✅ Good: Descriptive labels
runs-on: [self-hosted, linux, x64, 32gb-ram, gpu-enabled]

# ❌ Bad: Generic labels
runs-on: [self-hosted, server1]
```

### 5. Implement Auto-Scaling

For self-hosted runners, use auto-scaling to match demand:

- **Kubernetes**: actions-runner-controller
- **AWS**: EC2 Auto Scaling + Lambda
- **Azure**: Container Instances + Functions
- **GCP**: Cloud Run + Cloud Functions

## Critical Notes

🎯 **Default Choice**: Use GitHub-hosted runners unless you have specific requirements (GPU, internal network, etc.).

💡 **macOS Cost**: macOS runners consume minutes 10x faster than Linux. Use sparingly.

⚠️ **Public Repo Security**: NEVER use self-hosted runners with public repositories - major security risk.

📊 **Concurrent Limits**: Monitor your concurrent job usage. 20 concurrent jobs on Free tier fills quickly with matrix strategies.

🔄 **Ephemeral Runners**: For self-hosted, prefer ephemeral runners (create, run, destroy) over persistent runners for security.

✨ **Runner Images**: Check [GitHub's runner images](https://github.com/actions/runner-images) for exact software versions and updates.

## Quick Reference

### Runner Selection Decision Tree

```
Need specialized hardware (GPU, high-memory)?
├─ YES → Self-hosted
└─ NO
    └─ Need internal network access?
        ├─ YES → Self-hosted (private repos only)
        └─ NO
            └─ Job > 6 hours?
                ├─ YES → Self-hosted
                └─ NO → GitHub-hosted (recommended)
```

### Common Runner Configurations

```yaml
# Standard builds (most common)
runs-on: ubuntu-latest

# Cross-platform testing
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
runs-on: ${{ matrix.os }}

# Self-hosted with specific requirements
runs-on: [self-hosted, linux, gpu, production]

# Specific Ubuntu version
runs-on: ubuntu-22.04
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/8-explore-runners)
