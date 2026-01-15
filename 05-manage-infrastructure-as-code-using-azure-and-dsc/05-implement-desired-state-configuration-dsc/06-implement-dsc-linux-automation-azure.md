# Implement DSC and Linux Automation on Azure

While DSC originated on Windows, it fully supports Linux environments. This unit covers implementing DSC for Linux nodes in Azure using Azure Automation State Configuration.

## Why DSC for Linux?

**Cross-platform management**: Manage both Windows and Linux from a single DSC solution, reducing operational complexity.

**Benefits**:
- **Unified approach**: Same DSC concepts (configurations, resources, LCM) work on Linux
- **Centralized management**: Azure Automation State Configuration manages Windows + Linux nodes from one dashboard
- **Compliance**: Ensure Linux servers maintain desired state just like Windows
- **Automation**: Reduce manual configuration tasks on Linux fleets

## Supported Linux Operating Systems

DSC for Linux supports these distributions:

| Distribution | Supported Versions | Architecture |
|-------------|-------------------|--------------|
| **CentOS** | 6, 7, 8 | x64 |
| **Debian GNU/Linux** | 8, 9, 10 | x64 |
| **Oracle Linux** | 6, 7 | x64 |
| **Red Hat Enterprise Linux Server** | 6, 7, 8 | x64 |
| **SUSE Linux Enterprise Server** | 12, 15 | x64 |
| **Ubuntu Server** | 14.04 LTS, 16.04 LTS, 18.04 LTS, 20.04 LTS | x64 |

**Note**: Only x64 architecture is supported. ARM-based Linux systems are not currently supported.

## DSC for Linux Resources

DSC for Linux includes built-in resources for common Linux administration tasks:

### nxFile
Manage files and directories on Linux systems.

```powershell
nxFile ExampleFile {
    DestinationPath = "/etc/myapp/config.json"
    Ensure = "Present"
    Type = "File"
    Contents = '{"apiEndpoint": "https://api.contoso.com"}'
    Mode = "644"
    Owner = "root"
    Group = "root"
}
```

**Common parameters**:
- `DestinationPath`: Full path to file or directory
- `Ensure`: "Present" or "Absent"
- `Type`: "File" or "Directory"
- `Contents`: File contents (for small files)
- `SourcePath`: Copy from source
- `Mode`: Permissions (e.g., "755", "644")
- `Owner`, `Group`: User and group ownership

### nxScript
Run custom bash scripts to configure resources.

```powershell
nxScript InstallNginx {
    GetScript = @"
        #!/bin/bash
        if [ -x "$(command -v nginx)" ]; then
            echo "Installed"
        else
            echo "NotInstalled"
        fi
"@
    SetScript = @"
        #!/bin/bash
        apt-get update
        apt-get install -y nginx
        systemctl enable nginx
"@
    TestScript = @"
        #!/bin/bash
        command -v nginx > /dev/null 2>&1
"@
    User = "root"
}
```

**Scripts**:
- `GetScript`: Returns current state
- `SetScript`: Applies configuration
- `TestScript`: Tests if in desired state (exit 0 = compliant)

### nxUser
Manage user accounts on Linux systems.

```powershell
nxUser AppUser {
    UserName = "appuser"
    Ensure = "Present"
    FullName = "Application Service Account"
    Description = "Service account for web application"
    HomeDirectory = "/home/appuser"
    GroupID = "appgroup"
    Disabled = $false
}
```

### nxGroup
Manage user groups on Linux systems.

```powershell
nxGroup AppGroup {
    GroupName = "appgroup"
    Ensure = "Present"
    Members = "appuser", "admin"
    MembersToInclude = "developer"
}
```

### nxService
Manage services (systemd, upstart, System V).

```powershell
nxService NginxService {
    Name = "nginx"
    Controller = "systemd"
    Enabled = $true
    State = "Running"
}
```

**Controllers**:
- `systemd`: Modern Linux distributions (CentOS 7+, Ubuntu 16.04+)
- `upstart`: Older Ubuntu (14.04)
- `systemv`: Legacy systems

### nxPackage
Manage packages with apt, yum, or zypper.

```powershell
nxPackage InstallGit {
    Name = "git"
    Ensure = "Present"
    PackageManager = "apt"
}

nxPackage InstallDocker {
    Name = "docker-ce"
    Ensure = "Present"
    PackageManager = "apt"
    RequireSource = $true
    PackageGroup = $false
}
```

**Package managers**:
- `apt`: Debian, Ubuntu
- `yum`: CentOS, RHEL, Oracle Linux
- `zypper`: SUSE

## Implementing DSC for Linux on Azure

### Step 1: Create DSC Configuration for Linux

```powershell
Configuration LinuxWebServer {
    Import-DscResource -ModuleName nx

    Node "UbuntuWebServer" {
        # Install required packages
        nxPackage InstallNginx {
            Name = "nginx"
            Ensure = "Present"
            PackageManager = "apt"
        }

        nxPackage InstallGit {
            Name = "git"
            Ensure = "Present"
            PackageManager = "apt"
        }

        # Create application user
        nxGroup AppGroup {
            GroupName = "webappgroup"
            Ensure = "Present"
        }

        nxUser AppUser {
            UserName = "webappuser"
            Ensure = "Present"
            FullName = "Web App Service Account"
            HomeDirectory = "/home/webappuser"
            GroupID = "webappgroup"
        }

        # Create application directory
        nxFile AppDirectory {
            DestinationPath = "/var/www/myapp"
            Ensure = "Present"
            Type = "Directory"
            Mode = "755"
            Owner = "webappuser"
            Group = "webappgroup"
        }

        # Configure nginx
        nxFile NginxConfig {
            DestinationPath = "/etc/nginx/sites-available/myapp"
            Ensure = "Present"
            Type = "File"
            Contents = @"
server {
    listen 80;
    server_name myapp.contoso.com;
    root /var/www/myapp;
    index index.html;
}
"@
            Mode = "644"
        }

        # Enable and start nginx
        nxService NginxService {
            Name = "nginx"
            Controller = "systemd"
            Enabled = $true
            State = "Running"
            DependsOn = "[nxPackage]InstallNginx"
        }
    }
}
```

### Step 2: Upload and Compile Configuration

```powershell
# Connect to Azure
Connect-AzAccount

# Import configuration to Azure Automation
Import-AzAutomationDscConfiguration `
    -SourcePath ".\LinuxWebServer.ps1" `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "aa-contoso-prod" `
    -Published

# Compile configuration
Start-AzAutomationDscCompilationJob `
    -ConfigurationName "LinuxWebServer" `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "aa-contoso-prod"
```

### Step 3: Onboard Linux VMs to Azure Automation State Configuration

**For Azure Linux VMs**:

```powershell
# Register Azure VM
Register-AzAutomationDscNode `
    -ResourceGroupName "rg-web" `
    -AutomationAccountName "aa-contoso-prod" `
    -AzureVMName "ubuntu-web-01" `
    -ConfigurationMode "ApplyAndAutoCorrect" `
    -ConfigurationModeFrequencyMins 15 `
    -RefreshFrequencyMins 30 `
    -RebootNodeIfNeeded $true `
    -AllowModuleOverwrite $true `
    -ActionAfterReboot "ContinueConfiguration"
```

**For on-premises or other cloud Linux VMs**:

1. **Install OMI and DSC for Linux**:
```bash
# Download and install OMI
wget https://github.com/Microsoft/omi/releases/download/v1.6.8-1/omi-1.6.8-1.ssl_110.ulinux.x64.deb
sudo dpkg -i omi-1.6.8-1.ssl_110.ulinux.x64.deb

# Download and install DSC
wget https://github.com/Microsoft/PowerShell-DSC-for-Linux/releases/download/v1.2.1-0/dsc-1.2.1-0.ssl_110.x64.deb
sudo dpkg -i dsc-1.2.1-0.ssl_110.x64.deb
```

2. **Get registration key**:
```powershell
$RegistrationInfo = Get-AzAutomationRegistrationInfo `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "aa-contoso-prod"

$RegistrationKey = $RegistrationInfo.PrimaryKey
$RegistrationUrl = $RegistrationInfo.Endpoint
```

3. **Register node**:
```bash
# Register with Azure Automation
sudo /opt/microsoft/dsc/Scripts/Register.py \
    --RegistrationUrl $RegistrationUrl \
    --RegistrationKey $RegistrationKey \
    --ConfigurationMode ApplyAndAutoCorrect
```

### Step 4: Assign Configuration to Nodes

```powershell
# Assign configuration to specific node
Set-AzAutomationDscNode `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "aa-contoso-prod" `
    -NodeConfigurationName "LinuxWebServer.UbuntuWebServer" `
    -NodeName "ubuntu-web-01"
```

### Step 5: Monitor Compliance

```powershell
# Check node compliance status
Get-AzAutomationDscNode `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "aa-contoso-prod" `
    | Select-Object NodeName, Status, LastSeen

# Get detailed node report
Get-AzAutomationDscNodeReport `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "aa-contoso-prod" `
    -NodeId "ubuntu-web-01" `
    -Latest
```

## Complete Example: Multi-OS Environment

Manage both Windows and Linux from one configuration:

```powershell
Configuration HybridEnvironment {
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    Import-DscResource -ModuleName nx

    Node $AllNodes.Where{$_.OS -eq "Windows"}.NodeName {
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
        }

        File WebRoot {
            DestinationPath = "C:\inetpub\wwwroot"
            Ensure = "Present"
            Type = "Directory"
        }
    }

    Node $AllNodes.Where{$_.OS -eq "Linux"}.NodeName {
        nxPackage InstallNginx {
            Name = "nginx"
            Ensure = "Present"
            PackageManager = "apt"
        }

        nxFile WebRoot {
            DestinationPath = "/var/www/html"
            Ensure = "Present"
            Type = "Directory"
        }
    }
}

# Configuration data
$ConfigData = @{
    AllNodes = @(
        @{ NodeName = "WinWeb01"; OS = "Windows" },
        @{ NodeName = "WinWeb02"; OS = "Windows" },
        @{ NodeName = "UbuntuWeb01"; OS = "Linux" },
        @{ NodeName = "UbuntuWeb02"; OS = "Linux" }
    )
}

HybridEnvironment -ConfigurationData $ConfigData
```

## Best Practices

1. **Test locally first**: Use local DSC before deploying to Azure Automation
2. **Use configuration data**: Separate OS-specific logic with configuration data
3. **Monitor OMI service**: Ensure OMI service is running on Linux nodes
4. **Handle dependencies**: Use `DependsOn` for resource ordering
5. **Version control**: Track DSC configurations in Git
6. **Use nxScript sparingly**: Prefer native resources over scripts when possible
7. **Security**: Store credentials in Azure Automation credentials/variables

## Troubleshooting

**Check LCM status on Linux**:
```bash
sudo /opt/microsoft/dsc/Scripts/GetDscLocalConfigurationManager.py
```

**Check configuration status**:
```bash
sudo /opt/microsoft/dsc/Scripts/GetDscConfiguration.py
```

**View DSC logs**:
```bash
sudo tail -f /var/opt/omi/log/dsc.log
```

## Key Takeaways

- **Cross-platform**: DSC manages both Windows and Linux from one solution
- **Native resources**: 6 built-in Linux resources (nxFile, nxScript, nxUser, nxGroup, nxService, nxPackage)
- **Supported distributions**: CentOS, Debian, Oracle Linux, RHEL, SUSE, Ubuntu
- **Azure integration**: Azure Automation State Configuration centrally manages Linux + Windows
- **Unified dashboard**: Monitor compliance for all OS types in one place

---

**Module**: Implement Desired State Configuration (DSC)  
**Unit**: 6 of 8  
**Next**: [Unit 7: Module Assessment](#)  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-desired-state-configuration-dsc/6-implement-dsc-linux-automation-azure  
**Reference**: https://learn.microsoft.com/en-us/powershell/scripting/dsc/getting-started/lnxgettingstarted
