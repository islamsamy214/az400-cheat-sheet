# Install Azure CLI

## Overview

Installing Azure CLI is the first step toward automating Azure resource management. This unit guides you through selecting the right environment, installing Azure CLI on various platforms, verifying your installation, and configuring authentication. By the end, you'll have a fully functional Azure CLI environment ready for interactive commands and automated scripts.

## Environment Selection

Before installing Azure CLI, consider which environment best fits your needs:

### Decision Factors

| Factor | Considerations | Recommended Environment |
|--------|---------------|------------------------|
| **Automation Needs** | Frequent deployments, CI/CD integration | Linux servers, Docker containers |
| **Learning Curve** | New to Azure CLI, exploring features | Azure Cloud Shell (no installation required) |
| **Team Skillset** | Existing Bash expertise | Linux, macOS, Azure Cloud Shell |
| **Team Skillset** | Existing PowerShell expertise | Windows, PowerShell 7 on any platform |
| **Development Platform** | Windows enterprise environment | Windows 10/11 with MSI installer |
| **Development Platform** | Mac developers | macOS with Homebrew |
| **Development Platform** | Linux workstations/servers | Package manager (apt, yum) or script |
| **Consistency Requirements** | Same environment across team | Docker container (version-pinned) |
| **Quick Access** | No installation permissions | Azure Cloud Shell |
| **Offline Requirements** | Work without internet | Local installation (Windows, Linux, macOS) |

### Environment Comparison

| Environment | Pros | Cons | Best For |
|-------------|------|------|----------|
| **Azure Cloud Shell** | No installation, always latest version, pre-authenticated | Requires internet, session timeout (20 min inactive), limited customization | Learning, quick tasks, mobile access |
| **Windows (Local)** | Native Windows integration, offline access, full customization | Manual updates, Windows-specific path issues | Enterprise Windows environments |
| **Linux (Local)** | Native bash scripting, server automation, package manager updates | Requires command-line comfort | Servers, CI/CD agents, Linux developers |
| **macOS (Local)** | Developer-friendly, Homebrew simplicity, local testing | macOS-only | Mac developers, local dev/test |
| **Docker** | Isolated, version-controlled, reproducible environments | Requires Docker knowledge, slight overhead | CI/CD pipelines, multi-project environments |

## Windows Installation

### Method 1: MSI Installer (Recommended for Windows)

#### Step 1: Download Installer

1. Navigate to https://aka.ms/installazurecliwindows
2. Choose installer type:
   - **64-bit**: For modern Windows systems (Windows 10/11, Server 2016+)
   - **32-bit**: For older 32-bit Windows systems (rare)

**Recommended**: 64-bit installer for all modern systems

#### Step 2: Run Installer

1. Double-click the downloaded `.msi` file
2. Accept User Account Control (UAC) prompt (requires administrator rights)
3. Follow installation wizard:
   - Accept license agreement
   - Choose installation directory (default: `C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2`)
   - Complete installation

#### Step 3: Verify Installation

Open **new** terminal (cmd.exe, PowerShell, or Windows Terminal):

```powershell
# Check Azure CLI version
az version

# Expected output
{
  "azure-cli": "2.56.0",
  "azure-cli-core": "2.56.0",
  "azure-cli-telemetry": "1.1.0",
  "extensions": {}
}
```

**Important**: Close and reopen terminal after installation for PATH updates to take effect.

#### Step 4: Update Azure CLI

```powershell
# Check for updates
az upgrade

# Suppress prompts for automation
az upgrade --yes
```

### Method 2: Windows Package Manager (winget)

For Windows 10 version 1809+ and Windows 11:

```powershell
# Install Azure CLI
winget install -e --id Microsoft.AzureCLI

# Verify installation
az version
```

### Method 3: PowerShell Script Installation

```powershell
# Download and run installation script
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
Start-Process msiexec.exe -ArgumentList '/I AzureCLI.msi /quiet' -Wait
Remove-Item .\AzureCLI.msi

# Verify installation (in new PowerShell session)
az version
```

### Windows Troubleshooting

**Issue**: `az` command not recognized  
**Solution**: 
1. Close and reopen terminal
2. Verify installation directory in PATH:
   ```powershell
   $env:PATH -split ';' | Select-String "Azure"
   ```
3. Manually add to PATH if missing:
   ```powershell
   [Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\wbin", [EnvironmentVariableTarget]::Machine)
   ```

**Issue**: Permission denied during installation  
**Solution**: Run installer as Administrator (right-click → "Run as administrator")

**Issue**: Installation fails with error code 1603  
**Solution**: 
1. Uninstall existing Azure CLI versions
2. Clear Windows Installer cache: `%TEMP%`
3. Restart computer
4. Reinstall with administrator rights

## Linux Installation

### Method 1: Package Manager (Debian/Ubuntu)

#### One-Step Installation Script

```bash
# Download and run Microsoft's installation script
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

This script:
1. Configures Microsoft's package repository
2. Installs dependencies
3. Installs Azure CLI via `apt`
4. Configures automatic updates

#### Manual Step-by-Step Installation

```bash
# Step 1: Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl apt-transport-https lsb-release gnupg

# Step 2: Download and install Microsoft signing key
sudo mkdir -p /etc/apt/keyrings
curl -sLS https://packages.microsoft.com/keys/microsoft.asc | \
    gpg --dearmor | \
    sudo tee /etc/apt/keyrings/microsoft.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/microsoft.gpg

# Step 3: Add Azure CLI repository
AZ_DIST=$(lsb_release -cs)
echo "deb [arch=`dpkg --print-architecture` signed-by=/etc/apt/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/azure-cli/ $AZ_DIST main" | \
    sudo tee /etc/apt/sources.list.d/azure-cli.list

# Step 4: Install Azure CLI
sudo apt-get update
sudo apt-get install azure-cli

# Step 5: Verify installation
az version
```

#### Update Azure CLI

```bash
# Update all packages including Azure CLI
sudo apt-get update
sudo apt-get upgrade

# Update only Azure CLI
sudo apt-get update && sudo apt-get install --only-upgrade azure-cli
```

### Method 2: Package Manager (RHEL/CentOS/Fedora)

```bash
# Step 1: Import Microsoft repository key
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc

# Step 2: Add Azure CLI repository
echo -e "[azure-cli]
name=Azure CLI
baseurl=https://packages.microsoft.com/yumrepos/azure-cli
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc" | sudo tee /etc/yum.repos.d/azure-cli.repo

# Step 3: Install Azure CLI
sudo yum install azure-cli

# Step 4: Verify installation
az version
```

### Method 3: Script Installation (Distribution-Agnostic)

For distributions without package manager support:

```bash
# Download installation script
curl -L https://aka.ms/InstallAzureCli | bash

# Script installs to ~/.local/bin/az
# Add to PATH if not already present
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
source ~/.bashrc

# Verify installation
az version
```

### Linux Troubleshooting

**Issue**: `az: command not found`  
**Solution**: 
```bash
# Check if az is installed
which az

# If installed but not in PATH
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
source ~/.bashrc

# Or create symlink
sudo ln -s /opt/az/bin/az /usr/local/bin/az
```

**Issue**: Permission denied  
**Solution**: Run installation with `sudo` for system-wide installation, or use script method for user-only installation

**Issue**: Conflicting packages  
**Solution**: 
```bash
# Remove old installations
sudo apt-get remove azure-cli
sudo apt-get autoremove

# Clean package cache
sudo apt-get clean

# Reinstall
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

## macOS Installation

### Method 1: Homebrew (Recommended)

Homebrew is the preferred installation method for macOS:

```bash
# Step 1: Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Step 2: Update Homebrew
brew update

# Step 3: Install Azure CLI
brew install azure-cli

# Step 4: Verify installation
az version
```

#### Update Azure CLI

```bash
# Update Homebrew package list
brew update

# Upgrade Azure CLI
brew upgrade azure-cli

# Or update all packages
brew upgrade
```

### Method 2: Installation Script

```bash
# Download and run installation script
curl -L https://aka.ms/InstallAzureCli | bash

# Restart terminal to update PATH
exec bash -l

# Verify installation
az version
```

### macOS Troubleshooting

**Issue**: `brew: command not found`  
**Solution**: Install Homebrew first:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Issue**: Python version conflicts  
**Solution**: Azure CLI bundles its own Python runtime; no system Python configuration needed

**Issue**: macOS Gatekeeper blocks installation  
**Solution**: 
1. System Preferences → Security & Privacy
2. Allow applications downloaded from identified developers
3. Click "Open Anyway" if prompted

## Docker Installation

### Running Azure CLI in Docker

#### Interactive Session

```bash
# Run Azure CLI container interactively
docker run -it mcr.microsoft.com/azure-cli

# Inside container
az login
az account show
```

#### Mount Local Directory

```bash
# Mount current directory to container
docker run -it \
    -v ${PWD}:/workspace \
    -w /workspace \
    mcr.microsoft.com/azure-cli \
    bash

# Inside container, scripts in current directory are accessible
ls /workspace
./deploy.sh
```

#### Run Single Command

```bash
# Execute single command without interactive session
docker run --rm \
    mcr.microsoft.com/azure-cli \
    az account list --output table
```

#### Using in CI/CD (GitHub Actions Example)

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    container: mcr.microsoft.com/azure-cli
    steps:
      - name: Login to Azure
        run: |
          az login --service-principal \
            -u ${{ secrets.AZURE_CLIENT_ID }} \
            -p ${{ secrets.AZURE_CLIENT_SECRET }} \
            --tenant ${{ secrets.AZURE_TENANT_ID }}
      
      - name: Deploy infrastructure
        run: ./infrastructure/deploy.sh
```

#### Pinning Azure CLI Version

```bash
# Use specific version for reproducibility
docker run -it mcr.microsoft.com/azure-cli:2.56.0

# In Dockerfile
FROM mcr.microsoft.com/azure-cli:2.56.0
COPY scripts/ /scripts/
WORKDIR /scripts
```

### Docker Benefits for Azure CLI

1. **Isolated Environments**: Different projects can use different CLI versions
2. **No System Pollution**: Azure CLI contained in container
3. **Reproducible Builds**: Version-pinned containers ensure consistency
4. **Easy Cleanup**: Remove container to completely uninstall
5. **CI/CD Integration**: Standard container works across CI systems

## Azure Cloud Shell

### Accessing Cloud Shell

Azure Cloud Shell provides a pre-configured environment with Azure CLI already installed:

**Access Methods**:
1. **Azure Portal**: Click Cloud Shell icon (>_) in top navigation bar
2. **Direct URL**: https://shell.azure.com
3. **Mobile App**: Azure mobile app for iOS/Android
4. **VS Code**: Azure Account extension provides integrated terminal

### Cloud Shell Features

**Pre-Installed Tools**:
- Azure CLI (always latest version)
- Azure PowerShell
- Git
- Docker
- kubectl
- Terraform
- Ansible
- jq
- Editors: vim, nano, code (VS Code editor)

**Storage**:
- Persistent home directory in Azure Files
- 5 GB storage included
- Files persist across sessions
- Accessible from any location

**Authentication**:
- Automatically authenticated to Azure
- No `az login` required
- Uses your portal identity
- Subscription context automatically set

### When to Use Cloud Shell

**Ideal For**:
- Learning Azure CLI without installation
- Quick administrative tasks
- Accessing Azure from public computers
- Mobile device management
- Sharing commands in documentation (consistent environment)

**Not Ideal For**:
- Long-running scripts (20-minute idle timeout)
- Offline work
- Custom tool installations (reset on new sessions)
- Extensive customization

### Cloud Shell Example

```bash
# No installation needed - start using immediately

# Check Azure CLI version
az version

# List subscriptions (already authenticated)
az account list --output table

# Create resource group
az group create --name rg-cloudshell-demo --location eastus

# Upload files from local machine
# Use "Upload/Download files" button in Cloud Shell toolbar

# Edit scripts with built-in editor
code deploy.sh

# Execute scripts
chmod +x deploy.sh
./deploy.sh
```

## Verification

After installation on any platform, verify Azure CLI is working correctly:

### Check Version

```bash
# Display Azure CLI version and components
az version
```

Expected output:
```json
{
  "azure-cli": "2.56.0",
  "azure-cli-core": "2.56.0",
  "azure-cli-telemetry": "1.1.0",
  "extensions": {}
}
```

### Check Basic Functionality

```bash
# Test command execution (doesn't require authentication)
az --help

# List available reference groups
az --help | grep "^  [a-z]"
```

### Verify PATH Configuration

```bash
# Linux/macOS: Check az location
which az

# Windows (PowerShell): Check az location
Get-Command az

# Windows (cmd.exe): Check az location
where az
```

## Authentication Methods

After installation, authenticate to Azure using one of these methods:

### Method 1: Interactive Login (Recommended for Development)

**Use Case**: Local development, learning, individual user access

```bash
# Sign in with web browser
az login

# Process:
# 1. Opens browser to https://microsoft.com/devicelogin
# 2. Enter provided code
# 3. Sign in with Azure credentials
# 4. Complete MFA if required
# 5. Browser confirms successful authentication
# 6. Terminal displays available subscriptions
```

**Multi-Factor Authentication (MFA) Requirement**:
Starting in 2025, Microsoft enforces mandatory MFA for interactive logins. Ensure MFA is configured on your account before using `az login`.

**Specify Tenant**:
```bash
# Login to specific Azure AD tenant
az login --tenant <tenant-id>

# Login with specific subscription
az login
az account set --subscription "My Production Subscription"
```

**Browser-less Login** (for SSH sessions, VMs):
```bash
# Device code authentication (manual browser step)
az login --use-device-code

# Output provides URL and code to enter in browser
```

### Method 2: Service Principal (Recommended for Automation)

**Use Case**: CI/CD pipelines, automation scripts, application authentication

#### Create Service Principal

```bash
# Create service principal with Contributor role
az ad sp create-for-rbac --name "sp-azure-cli-automation" --role Contributor --scopes /subscriptions/{subscription-id}

# Output (save securely - password shown only once):
{
  "appId": "00000000-0000-0000-0000-000000000000",
  "displayName": "sp-azure-cli-automation",
  "password": "your-generated-password",
  "tenant": "11111111-1111-1111-1111-111111111111"
}
```

#### Authenticate with Service Principal

```bash
# Login using service principal credentials
az login --service-principal \
    --username <appId> \
    --password <password> \
    --tenant <tenant>

# Using environment variables (recommended for CI/CD)
az login --service-principal \
    --username $AZURE_CLIENT_ID \
    --password $AZURE_CLIENT_SECRET \
    --tenant $AZURE_TENANT_ID
```

#### CI/CD Pipeline Example (GitHub Actions)

```yaml
- name: Azure Login
  uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}

- name: Deploy Infrastructure
  run: |
    az group create --name rg-prod --location eastus
    az deployment group create --resource-group rg-prod --template-file main.bicep
```

### Method 3: Managed Identity (Azure-Hosted Applications)

**Use Case**: Azure VMs, App Services, Azure Functions running Azure CLI

```bash
# Authenticate using system-assigned managed identity
az login --identity

# Authenticate using user-assigned managed identity
az login --identity --username <client-id>
```

**Benefits**:
- No credentials to manage
- Automatic credential rotation
- Secure identity for Azure resources
- Ideal for VMs executing Azure CLI scripts

**Example**: VM running daily backup script
```bash
#!/bin/bash
# backup-resources.sh (runs on Azure VM with managed identity)

# Authenticate with managed identity
az login --identity

# Execute backup operations
az backup protection backup-now \
    --resource-group rg-backups \
    --vault-name vault-prod \
    --item-name vm-web-01 \
    --retain-until 30-12-2026
```

### Method 4: Azure Cloud Shell (Automatic)

**Use Case**: Quick tasks, learning, no local installation

Cloud Shell automatically authenticates using your portal identity:
```bash
# In Cloud Shell, already authenticated
az account show  # Displays current subscription
```

No `az login` needed in Cloud Shell.

### Authentication Persistence

Azure CLI caches authentication tokens locally:

**Token Location**:
- Linux/macOS: `~/.azure/`
- Windows: `%USERPROFILE%\.azure\`

**Token Lifetime**: 
- Interactive login: 90 days (reauth required after expiration)
- Service principal: Until credential expiration

**Clear Cached Authentication**:
```bash
# Logout (clears cached tokens)
az logout

# Clear all cached credentials
az account clear
```

### Verify Authentication

```bash
# Show currently authenticated account
az account show

# Output displays:
# - Subscription name and ID
# - User principal name (UPN)
# - Tenant ID
# - Cloud environment

# List all accessible subscriptions
az account list --output table
```

## Configuration

### Set Default Subscription

```bash
# Set default subscription for all commands
az account set --subscription "My Production Subscription"

# Or by subscription ID
az account set --subscription "00000000-0000-0000-0000-000000000000"

# Verify default subscription
az account show --query name -o tsv
```

### Configure CLI Settings

```bash
# Interactive configuration
az configure

# Set default output format
az configure --defaults output=table

# Set default resource group and location
az configure --defaults group=rg-prod location=eastus

# View current configuration
az config get
```

**Common Configuration Options**:
```ini
[core]
output = table          # Default output format
collect_telemetry = no  # Disable telemetry

[defaults]
group = rg-prod        # Default resource group
location = eastus      # Default location
```

### Enable Tab Completion

**Bash**:
```bash
# Add to ~/.bashrc
echo "source /etc/bash_completion.d/azure-cli" >> ~/.bashrc
source ~/.bashrc
```

**Zsh** (macOS default):
```bash
# Add to ~/.zshrc
autoload -U bashcompinit && bashcompinit
source /usr/local/etc/bash_completion.d/az
```

**PowerShell**:
```powershell
# Enable Azure CLI completion
Register-ArgumentCompleter -Native -CommandName az -ScriptBlock {
    param($wordToComplete, $commandAst, $cursorPosition)
    $completion_file = New-TemporaryFile
    $env:ARGCOMPLETE_USE_TEMPFILES = 1
    $env:_ARGCOMPLETE_STDOUT_FILENAME = $completion_file
    $env:COMP_LINE = $commandAst
    $env:COMP_POINT = $cursorPosition
    az 2>&1 | Out-Null
    Get-Content $completion_file | ForEach-Object {
        [System.Management.Automation.CompletionResult]::new($_, $_, "ParameterValue", $_)
    }
    Remove-Item $completion_file
}
```

## Installation Comparison

| Platform | Installation Method | Update Method | Best For |
|----------|-------------------|---------------|----------|
| **Windows** | MSI installer | `az upgrade` | Enterprise Windows, desktop users |
| **Windows** | winget | `winget upgrade --id Microsoft.AzureCLI` | Modern Windows systems |
| **Linux (Debian/Ubuntu)** | apt package manager | `sudo apt-get update && sudo apt-get upgrade azure-cli` | Linux servers, developers |
| **Linux (RHEL/CentOS)** | yum package manager | `sudo yum update azure-cli` | Red Hat environments |
| **macOS** | Homebrew | `brew upgrade azure-cli` | Mac developers |
| **Any** | Docker | Pull new image tag | CI/CD, isolated environments |
| **Any** | Azure Cloud Shell | Always latest (automatic) | Learning, quick tasks |

## Troubleshooting Installation Issues

### Universal Troubleshooting Steps

1. **Verify Installation**:
   ```bash
   az version
   az --help
   ```

2. **Check PATH**:
   ```bash
   # Linux/macOS
   echo $PATH | tr ':' '\n' | grep az
   
   # Windows
   $env:PATH -split ';' | Select-String az
   ```

3. **Reinstall**:
   - Completely uninstall existing installation
   - Clear cache/temporary files
   - Install latest version

4. **Check Permissions**:
   - Ensure write permissions to installation directory
   - Run installers with appropriate privileges

5. **Update Dependencies**:
   - Python (Linux/macOS script installations)
   - .NET Framework (Windows)
   - Package manager indexes

### Getting Help

```bash
# Azure CLI built-in help
az --help
az <command> --help

# Community support
# - Stack Overflow: [azure-cli] tag
# - GitHub Issues: https://github.com/Azure/azure-cli/issues
# - Microsoft Q&A: https://learn.microsoft.com/answers/
```

## Key Takeaways

1. **Multiple Installation Options**: Choose method appropriate for platform and use case
2. **Azure Cloud Shell**: Zero-installation option for learning and quick tasks
3. **Docker Isolation**: Ideal for reproducible CI/CD environments
4. **Authentication Methods**: Interactive (dev), service principal (automation), managed identity (Azure resources)
5. **MFA Required**: Starting 2025, interactive login requires multi-factor authentication
6. **Configuration**: Set defaults for output format, subscription, resource group, location
7. **Verification**: Always test installation with `az version` and `az login`

## Next Steps

With Azure CLI installed and authenticated, proceed to **Unit 4: Execute Azure CLI Commands Interactively** to start creating Azure resources and exploring interactive command execution.

---

**Module**: Create Azure Resources Using Azure CLI  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/create-azure-resources-by-using-azure-cli/3-install-azure-cli
